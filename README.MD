# Redex
Redux is a predictable state container for JavaScript apps  
Redux就是一个你能预定义的状态管理器  

> 对于复杂的单页面应用，状态（state）管理非常重要。state 可能包括：服务端的响应数据、本地对响应数据的缓存、本地创建的数据（比如，表单数据）以及一些 UI 的状态信息（比如，路由、选中的 tab、是否显示下拉列表、页码控制等等）。如果 state 变化不可预测，就会难于调试（state 不易重现，很难复现一些 bug）和不易于扩展（比如，优化更新渲染、服务端渲染、路由切换时获取数据等等  

- [Redux Github](https://github.com/reactjs/redux)
- [Redux 中文](http://cn.redux.js.org/index.html)
- [segmentfault](https://segmentfault.com/a/1190000003503338?_ea=323420)
- [阮一峰 Redux](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html)

Redux 就是用来确保`state`变化的可预测性，主要的约束有：
* state 以`单一 树形 对象`存储在`store`对象中
* state 只读
* 使用纯函数(pure function) `reducer`执行 state 更新

> state 为`单一对象`，因此Redux 只需要维护`一棵 状态树`，服务端很容易初始化状态，易于服务器渲染。  
state 只能通过 `dispatch`方法来触发`action` *dispatch(action)*，借此更新state，更新逻辑由`reducer`来执行

```js
// 导入redux库，并获取其createStore方法
import { createStore } from 'redux'

/**
* 这就是一个拥有 state 和 action 的单一 reducer函数
 * This is a reducer, a pure function with (state, action) => state signature.
 * 它具体说明了action怎样将一个state变换到另一个state
 * It describes how an action transforms the state into the next state.
 *
 * state的类型由你自定义
 * The shape of the state is up to you: it can be a primitive, an array, an object,
 * or even an Immutable.js data structure. The only important part is that you should
 * 唯一要注意的是 state不能修改的，只读；因此你需要返回一个新的state对象
 * not mutate the state object, but return a new object if the state changes.
 *
 * In this example, we use a `switch` statement and strings, but you can use a helper that
 * follows a different convention (such as function maps) if it makes sense for your
 * project.
 */
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}

// 创建一个 全局的 state的redux对象，这个对象有如下方法：subscribe, dispatch, getState
// Create a Redux store holding the state of your app.
// Its API is { subscribe, dispatch, getState }.
let store = createStore(counter)

// subscribe 用于监听state改变，在监听函数中可以对界面做一些修改
// You can use subscribe() to update the UI in response to state changes.
// Normally you'd use a view binding library (e.g. React Redux) rather than subscribe() directly.
// However it can also be handy to persist the current state in the localStorage.

store.subscribe(() =>
  console.log(store.getState())
)

// 要更改state只能通过dispatch方法来进行
// The only way to mutate the internal state is to dispatch an action.
// The actions can be serialized, logged or stored and later replayed.
store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
```

## action

* action就是你`预定义`的将会触发的各种`可能`，他是一个`普通的对象`
* action中type属性是必需的，用于描述具体类型的action，其值是字符串常量（Types should typically be defined as string constants），其他属性值可自定义

## reducer

*Actions describe the fact that something happened, but don't specify how the application's state changes in response. This is the job of a reducer*

* reducer就是你针对触发每一个action时，需进行具体`逻辑处理`的`函数`
* Reducer 函数最重要的特征是，它是一个纯函数。也就是说，只要是同样的输入，必定得到同样的输出。

> * 一个应用中`有且只有一个store`,也只有一个root reducer
* root reducer 可包含`多个子reducer`，对应处理具体某一部分state
* 当触发action时，会调用`每一个reducer`并返回最终结果

## store

* `store.getState()` 获取当前state
* `store.dispatch(action)` 来触发 state 更新
* `store.subscribe(listener)` 来注册 state 变化监听器
* `createStore(reducer, [initialState])`创建store
* `store.unsbuscribe(listener)` 取消监听

## Redux异步操纵
```
redux中dispatch触发action，在reducer中执行，并返回新的state，这个过程是一个同步的操作，
那么要实现异步就需要中间件（middleware）
```
* 中间件的运用就是在`dispath`的时候封装一层其他操纵

### 中间件的使用

```js
// 引入applyMiddleware方法
import { applyMiddleware, createStore } from 'redux';
// 引入要使用的插件，如：日志
import createLogger from 'redux-logger';
const logger = createLogger();

// 创建store时就需要添加中间件的使用，调用applyMiddleware(middleware)
const store = createStore(
  reducer,
  [initialState,]
  applyMiddleware(logger)
);
```
* 中间件的次序有讲究

```js
const store = createStore(
  reducer,
  applyMiddleware(thunk, promise, logger)
);

```
*上面代码中，applyMiddleware方法的三个参数，就是三个中间件。有的中间件有次序要求，使用前要查一下文档。比如，logger就一定要放在最后，否则输出结果会不正确*

#### redux-thunk
*由于默认的dispatch(action)触发的action只能是一个object对象*
* redux-thunk 的作用是在`dispatch(action)`时，action可以传递`function`,即`dispatch(function(){})`

#### redux-promise
* redux-promise 的作用是可以传递`promise`作为dispatch的参数，即：`dispatch(promise)`

## redux & react

* React-Redux连接redux和react的天使

React-Redux 将所有组件分成两大类：UI 组件（presentational component）和容器组件（container component）

* UI组件只负责组件渲染
 * 只负责UI的呈现，不带有任何业务逻辑
 * 没有状态（即不使用this.state这个变量）
 * 所有数据都由参数（this.props）提供
 * 不使用任何Redux的 API

* 容器组件负责状态管理
 * 负责管理数据和业务逻辑，不负责UI的呈现
 * 带有内部状态
 * 可调用Redux的API

React-Redux 提供`connect`方法，用于从`UI组件`生成`容器组件`
```
import { connect } from 'react-redux';
const VisibleTodoList = connect(mapStateToProps,mapDispatchToProps)(TodoList);
```
*TodoList是 UI 组件，VisibleTodoList就是由 React-Redux 通过connect方法自动生成的容器组件*
* mapStateToProps 用于映射state到UI组件的props
* mapDispatchToProps 用于指定UI组件触发事件时将进行怎样的dispatch(action)
