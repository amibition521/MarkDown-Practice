### Middlewares

* 中间件

  一个函数，对store.dispatch方法进行了改造，在发出action和执行Reducer着两步之间，添加了其他功能。

  ```
  let next = store.dispatch;
  store.dispatch = function dispatchAndLog(action) {
    console.log('dispatching',action);
    next(action);
    console.log('next state',store.getState());
    }
  ```

* 中间件的用法

  * 日志中间件     redux_logger

    ```
    import { applyMiddleware,creatStore} from 'redux'
    import createLogger from 'redux_logger'
    const logger = createLogger();

    const store = creatStore(
    	reducer,
    	applyMiddleware(logger)
    );
    ```

    `redux-logger`提供一个生成器`createLogger`，可以生成日志中间件`logger`。然后，将它放在`applyMiddleware`方法之中，传入`createStore`方法，就完成了`store.dispatch()`的功能增强。

    两点需要注意：

    1.creatStore方法可以接受整个应用的初始状态作为参数，

    ```

    const store = createStore(
      reducer,
      initial_state,
      applyMiddleware(logger)
    );
    ```

    2.中间件的次序有讲究

    ```
    const store = createStore(
    	reducer,
    	appleyMiddleware(thunk,promise,logger)
    );
    ```

* applyMiddlewares()

  * 作用是将所有中间件组成一个数组，依次执行。源码如下：

    ```
    export default function applyMiddleware(...middlewares) {
      return (createStore) => (reducer, preloadedState, enhancer) => {
        var store = createStore(reducer, preloadedState, enhancer);
        var dispatch = store.dispatch;
        var chain = [];

        var middlewareAPI = {
          getState: store.getState,
          dispatch: (action) => dispatch(action)
        };
        chain = middlewares.map(middleware => middleware(middlewareAPI));
        dispatch = compose(...chain)(store.dispatch);

        return {...store, dispatch}
      }
    }
    ```

    所有中间件被放进了一个数组`chain`，然后嵌套执行，最后执行`store.dispatch`。可以看到，中间件内部（`middlewareAPI`）可以拿到`getState`和`dispatch`这两个方法。

* 异步操作的基本思路

  * 同步操作只要发出一种action，异步操作需要发出三种action

    ```
    * 操作发起时的 Action
    * 操作成功时的 Action
    * 操作失败时的 Action
    ```

    ```
    // 写法一：名称相同，参数不同
    { type: 'FETCH_POSTS' }
    { type: 'FETCH_POSTS', status: 'error', error: 'Oops' }
    { type: 'FETCH_POSTS', status: 'success', response: { ... } }

    // 写法二：名称不同
    { type: 'FETCH_POSTS_REQUEST' }
    { type: 'FETCH_POSTS_FAILURE', error: 'Oops' }
    { type: 'FETCH_POSTS_SUCCESS', response: { ... } }

    ```

    action种类不同，异步操作的state也要进行改造

    ```
    let state = {
      //.....
      isFetching:true,
      didInvalidate : true,
      lastUpdated:'xxxxxx',
    }
    ```

    State 的属性`isFetching`表示是否在抓取数据。`didInvalidate`表示数据是否过时，`lastUpdated`表示上一次更新时间。

    整体思路如下：

    * 操作开始时，送出一个 aciton，触发 state 更新为“正在操作”状态，View重新渲染。
    * 操作结束后，再送出一个 action，触发state更新为“操作结束”状态，View再一次重新渲染。

* redux-thunk中间件

  * 目的：改造store.dispatch,使得后者可以接受函数作为参数

  异步操作的第一种解决方案，写出一个返回函数的 Action Creator，然后使用redux-thunk中间件改造store.dispatch.

  ```
  import { createStore, applyMiddleware } from 'redux';
  import thunk from 'redux-thunk';
  import reducer from './reducers';

  // Note: this API requires redux@>=3.1.0
  const store = createStore(
    reducer,
    applyMiddleware(thunk)
  );

  ```

  ​

* redux-promise中间件

  * 异步操作的解决方案：让Action Creator返回一个Promise对象。

    ```

    import { createStore, applyMiddleware } from 'redux';
    import promiseMiddleware from 'redux-promise';
    import reducer from './reducers';

    const store = createStore(
      reducer,
      applyMiddleware(promiseMiddleware)
    );
    ```

    这个中间件使得`store.dispatch`方法可以接受 Promise 对象作为参数。这时，Action Creator 有两种写法。写法一，返回值是一个 Promise 对象。

    ```
    const fetchPosts = 
      (dispatch, postTitle) => new Promise(function (resolve, reject) {
         dispatch(requestPosts(postTitle));
         return fetch(`/some/API/${postTitle}.json`)
           .then(response => {
             type: 'FETCH_POSTS',
             payload: response.json()
           });
    });

    ```

    写法二，Action 对象的`payload`属性是一个 Promise 对象。这需要从[`redux-actions`](https://github.com/acdlite/redux-actions)模块引入`createAction`方法，并且写法也要变成下面这样。

    ```
    import { createAction } from 'redux-actions';

    class AsyncApp extends Component {
      componentDidMount() {
        const { dispatch, selectedPost } = this.props
        // 发出同步 Action
        dispatch(requestPosts(selectedPost));
        // 发出异步 Action
        dispatch(createAction(
          'FETCH_POSTS', 
          fetch(`/some/API/${postTitle}.json`)
            .then(response => response.json())
        ));
      }
    ```

    源码如下：

    ```

    export default function promiseMiddleware({ dispatch }) {
      return next => action => {
        if (!isFSA(action)) {
          return isPromise(action)
            ? action.then(dispatch)
            : next(action);
        }

        return isPromise(action.payload)
          ? action.payload.then(
              result => dispatch({ ...action, payload: result }),
              error => {
                dispatch({ ...action, payload: error, error: true });
                return Promise.reject(error);
              }
            )
          : next(action);
      };
    }
    ```

    从上面代码可以看出，如果 Action 本身是一个 Promise，它 resolve 以后的值应该是一个 Action 对象，会被`dispatch`方法送出（`action.then(dispatch)`），但 reject 以后不会有任何动作；如果 Action 对象的`payload`属性是一个 Promise 对象，那么无论 resolve 和 reject，`dispatch`方法都会发出 Action。

    ​
