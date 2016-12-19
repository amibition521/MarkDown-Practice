* UI 组件 

  * 负责UI的呈现

* 容器组件

  * 负责管理数据和逻辑

  如果一个组件既有 UI 又有业务逻辑，那怎么办？回答是，将它拆分成下面的结构：外面是一个容器组件，里面包了一个UI 组件。前者负责与外部的通信，将数据传给后者，由后者渲染出视图。

  React-Redux 规定，所有的 UI 组件都由用户提供，容器组件则是由 React-Redux 自动生成。也就是说，用户负责视觉层，状态管理则是全部交给它。

* connect()

  * 用于从UI组件生成容器组件。

    ```
    import {connect } from 'react-redux'
    const VisibleTodoList = connect()(TodoList);
    ```

    `TodoList`是 UI 组件，`VisibleTodoList`就是由 React-Redux 通过`connect`方法自动生成的容器组件。

    业务逻辑需要满足下面两方面的信息

    > （1）输入逻辑：外部的数据（即`state`对象）如何转换为 UI 组件的参数
    >
    > （2）输出逻辑：用户发出的动作如何变为 Action 对象，从 UI 组件传出去。

  connect方法的完整API如下：

  ```
  import { connect } from 'reac-redux'

  const VisibleTodoList = connect (
  	mapStateToProps,
  	mapDispatchToProps,
  )(TodoList)
  ```

  上面代码中，`connect`方法接受两个参数：`mapStateToProps`和`mapDispatchToProps`。它们定义了 UI 组件的业务逻辑。前者负责输入逻辑，即将`state`映射到 UI 组件的参数（`props`），后者负责输出逻辑，即将用户对 UI 组件的操作映射成 Action。

* mapStateToProps()

  * 建立一个从（外部的）state对象到（UI组件的）props对象的映射关系

  * 返回一个对象，每一个键值对就是一个映射

    ```
    const mapStateToProps = (state) => {
     return {
       todos: 									         getVisibleTodos(state.todos,state.visibilityFilter)
     } 
    }
    ```

    `mapStateToProps`是一个函数，它接受`state`作为参数，返回一个对象。这个对象有一个`todos`属性，代表 UI 组件的同名参数，后面的`getVisibleTodos`也是一个函数，可以从`state`算出 `todos` 的值。

    下面就是`getVisibleTodos`的一个例子，用来算出`todos`。

    ```
    const getVisibleTodos = (todos, filter) => {
      switch (filter) {
        case 'SHOW_ALL':
          return todos
        case 'SHOW_COMPLETED':
          return todos.filter(t => t.completed)
        case 'SHOW_ACTIVE':
          return todos.filter(t => !t.completed)
        default:
          throw new Error('Unknown filter: ' + filter)
      }
    }
    ```

    `mapStateToProps`会订阅Store，每当state更新的时候，就会自动执行，重新计算UI组件的参数，从而触发UI组件重新渲染。

    `mapStateToProps`的第一个参数总是state，第二个参数代表容器组件的`props`对象。

     ```
    // 容器组件的代码
    //    <FilterLink filter="SHOW_ALL">
    //      All
    //    </FilterLink>

    const mapStateToProps = (state, ownProps) => {
      return {
        active: ownProps.filter === state.visibilityFilter
      }
    }
     ```

    使用`ownProps`作为参数后，如果容器组件的参数发生变化，也会引发 UI 组件重新渲染。

    `connect`方法可以省略`mapStateToProps`参数，那样的话，UI 组件就不会订阅Store，就是说 Store 的更新不会引起 UI 组件的更新。

* mapDispatchToProps()

  * 用来建立UI组件的参数到store.dispatch方法的映射，

  * 定义了哪些用户的操作应该当作Action传给Store，可以是一个函数，也可以是一个对象。

  * mapDispatchToProps是一个函数，会得到dispatch 和ownProps（容器组件的props对象）两个参数。

    ```

    const mapDispatchToProps = (
      dispatch,
      ownProps
    ) => {
      return {
        onClick: () => {
          dispatch({
            type: 'SET_VISIBILITY_FILTER',
            filter: ownProps.filter
          });
        }
      };
    }
    ```

    从上面代码可以看到，`mapDispatchToProps`作为函数，应该返回一个对象，该对象的每个键值对都是一个映射，定义了 UI 组件的参数怎样发出 Action。

    如果`mapDispatchToProps`是一个对象，它的每个键名也是对应 UI 组件的同名参数，键值应该是一个函数，会被当作 Action creator ，返回的 Action 会由 Redux 自动发出。举例来说，上面的`mapDispatchToProps`写成对象就是下面这样。

    ```
    const mapDispatchToProps = {
      onClick :(filter) => {
        type : 'SET_VISIBILITY_FILTER',
        filter :filter
      };
    }
    ```

* <Provider> 组件

  * 让容器组件得到state。

    ```
    import { Provider } from 'react-redux'
    import { createStore } from 'redux'
    import todoApp from './reducers'
    import App from './components/App'

    let store = createStore(todoApp);

    render(
      <Provider store={store}>
        <App />
      </Provider>,
      document.getElementById('root')
    )
    ```

    `Provider`在根组件外面包了一层，这样以来，App的所有子组件就默认都可以拿到state了，

* React-Router路由库

  ```

  const Root = ({ store }) => (
    <Provider store={store}>
      <Router>
        <Route path="/" component={App} />
      </Router>
    </Provider>
  );
  ```

  ​
