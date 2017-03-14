Redux Flux

    镇楼
    /*
                     _________               ____________               ___________
                    |         |             |            |             |           |
                    | Action  |------------▶| Dispatcher |------------▶| callbacks |
                    |_________|             |____________|             |___________|
                         ▲                                                   |
                         |                                                   |
                         |                                                   |
     _________       ____|_____                                          ____▼____
    |         |◀----|  Action  |                                        |         |
    | Web API |     | Creators |                                        |  Store  |
    |_________|----▶|__________|                                        |_________|
                         ▲                                                   |
                         |                                                   |
                     ____|________           ____________                ____▼____
                    |   User       |         |   React   |              | Change  |
                    | interactions |◀--------|   Views   |◀-------------| events  |
                    |______________|         |___________|              |_________|
    
    */

第一章 simple-action-creator

- action creator
  -     var actionCreator = function() {
          //...负责构建action
          return {
            type:'AN_ACTION'
          }
        }
        action 是一个拥有type属性的对象。按type决定如何处理action,action也可以拥有其他属性,存放任意想要的数据。
        actionCreator 可以返回action以外的其他东西，比如一个函数。(参考dispatch-async-action.js)
        console.log(actionCreator());
  - 分发action (Dispatching an action)
  - 处理器--store
  - ActionCreator---> Action
  - http://rackt.org/redux/docs/recipes/ReducingBoilerplate.html

第二章 about-state-and-meet-redux

- 相应的问题：
  - 在应用程序的整个生命周期内维持所有的数据？
  - 如何修改数据？
  - 如何包数据变更传播到整个应用程序？
- Redux 是一个“可预测化状态的Java Script 容器”
  - 一次性回答上面问题：
    - 1.将状态信息交给Redux.
    - 2.使用reducer函数修改数据，是action的订阅者。作用：接收应用程序的当前状态以及发生的action.然后返回修改后的新状态(归并的状态)。
    - 3.使用订阅者来监听状态的变更
  - 1) 存放应用程序状态的容器
  - 2)一种把 action 分发到状态修改器的机制，也就是 reducer 函数
  - 3) 监听状态变化的机制
- Redux实例---store
      import {createStore} from 'redux';
      var store = createStore(() => {});
  

