---
title: Redux使用说明
date: 2019-8-6 14:57:32
tags:
  - javascript
  - react
  - redux
categories:
  - WIKI
keywords: 'ES6,react,javascript,react-redux,redux,react'
description: Redux是JavaScript状态管理容器，提供可预测的状态管理。redux可以让你构建一致化的应用，运行于不同的环境（客户端、服务端、原生应用），且易于测试。
cover: https://cdn.jsdelivr.net/gh/LishiJ/assets/img/redux.png
---

> `Redux` 是 `JavaScript` 状态管理容器，提供可预测的状态管理。`redux` 可以让你构建一致化的应用，运行于不同的环境（客户端、服务端、原生应用），且易于测试。

`Redux` 由 [`Flux`](http://facebook.github.io/flux/) 演变而来，但避开了 `Flux` 的复杂性。

### 使用动机及场景
我们应该什么时候使用状态管理容器呢？

不是所有的应用都需要用到 `Redux` 状态管理容器。随着单页面的日趋发展，`Javascript`需要处理更多更复杂的 `state(状态)`，这些状态包括缓存数据、服务器的响应、本地生成或处理UI、路由等，管理这些状态的变化变得非常困难。

同时，由于不同的 `model` 变化都可能造成 `state` 的变化，从而导致状态变化不可控，即什么时候由于什么原因导致的变化不受控制。

总而言之，最大的难点在于变化和异步，所以 `Redux` 基于这个思想，将两者分开，使状态可预测。

### 安装

``` sh
# npm
npm install --save redux

# yarn
yarn add redux
```

在使用React进行开发时，你还需要使用 [`React Redux`](https://github.com/reduxjs/react-redux) 和[开发者工具](https://github.com/reduxjs/redux-devtools)。

``` sh
npm install --save react-redux
npm install --save-dev redux-devtools
```

### 基本原则

`Redux` 有三个基本原则，通过这三个基本原则，我们可以很好地将变化和异步分开，同时通过这三个基本原则，我们也可以清楚的知道 `Redux` 数据流，先来看一张图:

![Redux数据流图](https://upload-images.jianshu.io/upload_images/14272773-9130e2bfc73f7f1d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 单一数据源
> 整个应用的 `state` 被储存在一棵 `object tree` 中，并且这个 `object tree` 只存在于唯一一个 `store` 中。

#### State 是只读的
> 唯一改变 `state` 的方法就是触发 `action`，`action` 是一个用于描述已发生事件的普通对象。

#### 使用纯函数来执行修改
> 为了描述 `action` 如何改变 `state tree`，你需要编写 `reducers`。

如果你了解过其他的状态管理容器可能一下就能明白上面的含义（中心思想还是一样的），不懂得我们等看过下面的介绍再回过头来看就非常清楚了。

### 基础
#### State
`state` 不用多说，简单来说就是用于存储各状态的变量。

#### Action
`action` 本质上是 `Javascript` 对象，约定内部必须使用一个字符串类型的 `type` 字段来表示将要执行的动作，整体作为从应用传到 `store` 的有效载荷。一般来说你会通过 `store.dispatch()` 将 `action` 传到 `store`。多数情况线下，`type` 会被定义成字符串常量，当应用规模越来越大时，建议使用单独的模块或文件来存储 `action`。 

除了 `type` 字段外，`action` 对象的结构完全由你自己决定。`action` 的格式应该是这样的：

``` js
const ADD_TODO = 'ADD_TODO'
```
```
{
  type: ADD_TODO,
  text: 'Build my first Redux app'
}
```
##### Action 创建函数
顾名思义就是生成 `action` 的方法。在 `Redux` 中 `action` 创建函数只是简单的返回一个 `action`：

``` js
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}
```

然后通过 `dispatch()` 方法发起一次 `dispatch` 过程。

``` js
dispatch(addTodo(text))
```

或者创建一个被绑定的 `action` 创建函数来自动 `dispatch`：

``` js
const boundAddTodo = text => dispatch(addTodo(text))
```

然后直接调用它们：

``` js
boundAddTodo(text);
```

##### actions.js

``` js
/*
 * action 类型
 */

export const ADD_TODO = 'ADD_TODO';
export const TOGGLE_TODO = 'TOGGLE_TODO'
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER'

/*
 * 其它的常量
 */

export const VisibilityFilters = {
  SHOW_ALL: 'SHOW_ALL',
  SHOW_COMPLETED: 'SHOW_COMPLETED',
  SHOW_ACTIVE: 'SHOW_ACTIVE'
}

/*
 * action 创建函数
 */

export function addTodo(text) {
  return { type: ADD_TODO, text }
}

export function toggleTodo(index) {
  return { type: TOGGLE_TODO, index }
}

export function setVisibilityFilter(filter) {
  return { type: SET_VISIBILITY_FILTER, filter }
}
```

#### Reducer
`reducer` 指定了应用状态的变化如何响应 `actions` 并发送到 `store` 的，记住 `actions` 只是描述了有事情发生了这一事实，并没有描述应用如何更新 `state`。

`reducer` 就是一个纯函数，接收旧的 `state` 和 `action`，返回新的 `state`。

``` js
import { VisibilityFilters } from './actions'

const initialState = {
  visibilityFilter: VisibilityFilters.SHOW_ALL,
  todos: []
};

function todoApp(state, action) {
  if (typeof state === 'undefined') {
    return initialState
  }

  // 这里暂不处理任何 action，
  // 仅返回传入的 state。
  return state
}
```
这里一个技巧是使用 ES6 参数默认值语法 来精简代码。

``` js
import { VisibilityFilters } from './actions'

const initialState = {
  visibilityFilter: VisibilityFilters.SHOW_ALL,
  todos: []
};

function todoApp(state = initialState, action) {
  return state;
}
```
现在可以处理 `SET_VISIBILITY_FILTER`。需要做的只是改变 `state` 中的 `visibilityFilter`。

``` js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    default:
      return state
  }
}
```
或者

``` js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return {...state, ...{
        visibilityFilter: action.filter
      })
    default:
      return state
  }
}
```

#### Store
在前面的章节中，我们学会了使用 `action` 来描述“发生了什么”，和使用 `reducers` 来根据 `action` 更新 `state` 的用法。

`Store` 就是把它们联系到一起的对象。`Store` 有以下职责：
- 维持应用的 `state`；
- 提供 `getState()` 方法获取 state；
- 提供 `dispatch(action)` 方法更新 state；
- 通过 `subscribe(listener)` 注册监听器;
- 通过 `subscribe(listener)` 返回的函数注销监听器。

再次强调一下 `Redux` 应用只有一个单一的 `store`。当需要拆分数据处理逻辑时，你应该使用 `reducer` 组合 而不是创建多个 `store`。

```
import { createStore } from 'redux'
import todoApp from './reducers'
let store = createStore(todoApp)
```
现在我们已经创建好了 `store` ，让我们来验证一下！虽然还没有界面，我们已经可以测试数据处理逻辑了。

``` js
import {
  addTodo,
  toggleTodo,
  setVisibilityFilter,
  VisibilityFilters
} from './actions'

// 打印初始状态
console.log(store.getState())

// 每次 state 更新时，打印日志
// 注意 subscribe() 返回一个函数用来注销监听器
const unsubscribe = store.subscribe(() =>
  console.log(store.getState())
)

// 发起一系列 action
store.dispatch(addTodo('Learn about actions'))
store.dispatch(addTodo('Learn about reducers'))
store.dispatch(addTodo('Learn about store'))
store.dispatch(toggleTodo(0))
store.dispatch(toggleTodo(1))
store.dispatch(setVisibilityFilter(VisibilityFilters.SHOW_COMPLETED))

// 停止监听 state 更新
unsubscribe();
```

#### 数据流
在刚开始之前我们有提到过 `redux` 是单向数据流，通过上面的基础讲解，大致上我们知道了数据的流动，下面我们总结一下 `Redux` 应用中数据的流动步骤：

1. 调用 `store.dispatch(action)`。

`Action` 就是一个描述发生了什么的普通对象，比如 `{ type: 'LIKE_ARTICLE', articleId: 42 }`，我们可以在任意地方调用 `store.dispatch(action)`。

2. `Redux store` 调用传入的 `reducer` 函数。

`Reducer` 简单的说就是纯函数，仅仅用于计算下一个 `state`，它应该是可预测的。它不应做有副作用的操作，如 `API` 调用或路由跳转。这些应该在 `dispatch action` 前发生。

3. 根 `reducer` 应该把多个子 `reducer` 输出合并成一个单一的 `state` 树。

``` js
function todos(state = [], action) {
   // 省略处理逻辑...
   return nextState
 }

 function visibleTodoFilter(state = 'SHOW_ALL', action) {
   // 省略处理逻辑...
   return nextState
 }

 let todoApp = combineReducers({
   todos,
   visibleTodoFilter
 })
```

4. `Redux store` 保存了根 `reducer` 返回的完整 `state` 树。

这个新的树就是应用的下一个 `state`！所有订阅 `store.subscribe(listener)` 的监听器都将被调用；监听器里可以调用 `store.getState()` 获得当前 `state`。

### React中使用Redux
这里需要再强调一下：`Redux` 和 `React` 之间没有关系。`Redux` 支持 `React`、`Angular`、`Ember`、`jQuery` 甚至纯 `JavaScript`。
#### 安装 React Redux

``` sh
npm install --save react-redux
```

#### 如何使用
对于 `React` 相关知识点这里就不多做赘述，我们主要针对于 `React` 开发中如何使用 `Redux` 做一些介绍。

使用 `connect` 方法将 `React` 和 `Redux` 关联起来。

使用 `connect()` 前，需要先定义 `mapStateToProps` 这个函数来指定如何把当前 `Redux store state` 映射到展示组件的 `props` 中。例如，`VisibleTodoList` 需要计算传到 `TodoList` 中的 `todos`，所以定义了根据 `state.visibilityFilter` 来过滤 `state.todos` 的方法，并在 `mapStateToProps` 中使用。

``` js
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
    case 'SHOW_ALL':
    default:
      return todos
  }
}

const mapStateToProps = state => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}
```
另外，还需要能分发的 `action`，可以定义 `mapDispatchToProps()` 方法接收 `dispatch()` 方法并返回期望注入到展示组件的 `props` 中的回调方法。

``` js
const mapDispatchToProps = dispatch => {
  return {
    onTodoClick: id => {
      dispatch(toggleTodo(id))
    }
  }
}
```
最后，使用 `connect` 关联起来即可


``` js
import { connect } from 'react-redux'

class TodoList extends Component {
    // ...
}


const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```

### 进阶
本文主要讲述基础，如需更好的优化代码 ，拆分逻辑结构等，请自行根据项目或官方推荐文档进行操作。




