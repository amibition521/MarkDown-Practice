### Redux 语法

设计思想：

* web应用是一个状态机，试图与状态是一一对应的。
* 所有的状态，保存在一个对象里面。

基本概念和API

1.Store

​	保存数据，容器，整个应用只能有一个store。

​	Redux提供了`createStore`函数，用来生成Store.

```
import { createStore } from 'redux'
const store = createStore(fn);
```

`createStore`函数接受另一个函数作为参数，返回新生成的Store对象。

2.State

​	Store 对象包含所有数据，如果想得到某个时点的数据，就要对Store生成快照，这种时点的数据集合就叫做State.

当前时刻的State，可以通过`store.getState()`拿到。

```
import { createStore } from 'redux';
const store = createStore(fn);

const state = store.getState();
```

Redux 规定， 一个 State 对应一个 View。只要 State 相同，View 就相同。你知道 State，就知道 View 是什么样，反之亦然。

3.Action

​	Action 是View发出的通知，表示State应该发生变化。

​	Action 是一个对象，其中type属性是必须的，表示action的名称，其他属性可以自由设置。

```
//Action 的名称是ADD_TODO，它携带的信息是字符串Learn Reduxk
const action = {
  type: 'ADD_TODO',
  payload: 'Learn Redux'
};
```

4.Action Creator

​	定义一个函数来生成action,这个函数就是action creator;

```
const ADD_TODO = '添加 TODO';

function addTodo(text) {
  return {
    type : ADD_TODO,
    text
  }
}
const action = addTodo ('Learn Redux');
```

5.store.dispatch()

​	`store.dispathc()` 是View发出action的唯一方法。

```
import { createStore } from 'redux';
const store = createStore(fn);

store.dispatch({
  type: 'ADD_TODO',
  payload: 'Learn Redux'
});

//store.dispatch(addTodo('Learn Redux'));
```

6.Reducer

​	接收action和当前state作为参数，返回一个新的state。

```
const reducer = function (state, action) {
  // ...
  return new_state;
};
```

```
const defaultState = 0;
const reducer = (state = defaultState, action) => {
  switch (action.type) {
    case 'ADD':
      return state + action.payload;
    default: 
      return state;
  }
};

const state = reducer(1, {
  type: 'ADD',
  payload: 2
});
```

store.dispath()会触发Reducer的自动执行。

```
import { createStore } from 'redux';
const store = createStore(reducer);
```

createStore接受 Reducer 作为参数，生成一个新的 Store。以后每当store.dispatch发送过来一个新的 Action，就会自动调用 Reducer，得到新的 State。

7.纯函数

​	是函数式编程的概念

*   不得改写参数
*   不能调用系统 I／O的API
*   不能调用Date.now()或Math.random()等不纯的方法，因为每次会得到不一致的结果

Reducer 是一个纯函数，里面不能改变state，必须返回一个全新的对象。

```
// State 是一个对象
function reducer(state, action) {
  return Object.assign({}, state, { thingToChange });
  // 或者
  return { ...state, ...newState };
}

// State 是一个数组
function reducer(state, action) {
  return [...state, newItem];
}
```

8.store.subscribe()

​	Store 允许使用store.subscribe方法设置监听函数，

​	一旦 State 发生变化，就自动执行这个函数。

```
import { createStore } from 'redux';
const store = createStore(reducer);

store.subscribe(listener);
```

组件的render方法或者setstate方法放入listen 就会实现view的自动渲染。

该方法返回一个函数，调用这个函数可以解除监听。

```
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
);

unsubscribe();
```

### Store的实现

```
import { createStore } from 'redux';
let { subscribe, dispatch, getState } = createStore(reducer);
```

```
let store = createStore(todoApp,window.STATE_FROM_SERVER);
```

window.STATE_FROM_SERVER就是整个应用的状态初始值。注意，如果提供了这个参数，它会覆盖 Reducer 函数的默认初始值

下面是一个createStore方法的简单实现：

```
const createStore = (reducer) => {
  let state;
  let listeners = [];

  const getState = () => state;

  const dispatch = (action) => {
    state = reducer(state, action);
    listeners.forEach(listener => listener());
  };

  const subscribe = (listener) => {
    listeners.push(listener);
    return () => {
      listeners = listeners.filter(l => l !== listener);
    }
  };

  dispatch({});

  return { getState, dispatch, subscribe };
};
```

### Reducer 的拆分

reducer函数负责生成State

```
const chatReducer = (state = defaultState,action = {}) => {
  const {type,payload} = action;
  switch (type) {
    case ADD_CHAT:
    	return Object.assign ({},state, {
          chatlog : state.chatlog.concat(payload);
    	});
    case CHANGE_STATUS:
    	return Object.assign({},state, {
          statusMessage:palyload
    	});
    case CHANGE_USERNAME:
    	return Object.assign ({},state, {
          username:palyload
    	});
    default :return state;
  } 
};
```

三个action分别代表改变State的三个属性。由于这三个属性之间没有必然的联系，因此将Reducer函数拆分。

```
const chatReducer = (state = defaultState,action = {}) => {
  return {
    chatLog:chatLog (state.chatLog,action),
    statusMessage :statusMessage(state.statusMessage,action),
    userName : userName (state.userName,action),
  }
};
```

Redux 提供了combineReducers方法，用于Reducer的拆分。

```
import { combineReducers} from 'redux'
const chatReducer = combineReducers ({
  chatLog,
  statusMessage,
  userName,
});
```

这么写的前提是State的属性·名必须与子Reducer同名。如果不同名，就采用下面的写法：

```
const reducer = combineReducers({
  a: doSomethingWithA,
  b: processB,
  c: c
})

// 等同于
function reducer(state = {}, action) {
  return {
    a: doSomethingWithA(state.a, action),
    b: processB(state.b, action),
    c: c(state.c, action)
  }
}
```

combineReducer的简单实现

```
const combineRecuders = recedues => {
  return (state = {},action) => {
    return Object.keys(reducers).reduce(
    (nextState,key) => {
      nextState[key] = reducers[key](state[key],action);
      return nextState;
    },
    {}
    );
  };
};
```

比较好的方式是把所有的子Reducer放在一个文件里面，统一引入。

```
import { combineReducers } from 'redux'
import * as reducers from './reducers'

const reducer = combineReducers(reducers)
```

