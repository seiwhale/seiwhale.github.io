---
title: Redux-saga使用说明
date: 2019-8-6 15:14:44
tags:
  - javascript
  - react
  - redux
  - react-redux
  - redux-saga
categories:
  - WIKI
keywords: 'ES6,react,javascript,react-redux,redux,react,redux-saga'
description: redux-saga就是一个用于管理redux应用的异步操作中间件，redux-saga通过创建sagas将所有异步操作的逻辑都收集在一个位置集中处理。
cover: https://cdn.jsdelivr.net/gh/seiwhale/assets/img/react_redux.png
---

### 概述
在 `redux` 一文中我们有说过处理异步我们应该放在 `reducer` 之前，所以我们需要中间件来处理我们的异步操作。`redux-saga` 就是一个用于管理 `redux` 应用的异步操作中间件，`redux-saga` 通过创建 `sagas` 将所有异步操作的逻辑都收集在一个位置集中处理。

简单的来说就是两步：
-  `sagas` 负责协调那些复杂或者异步的操作
-  `reducer` 负责处理 `action` 的 `stage` 更新

### 数据流
我们先看一张图来大致了解下整个过程：

![6493119-f4c44eb340874014.png](https://upload-images.jianshu.io/upload_images/14272773-b350a23f458d6312.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


有了之前 `redux` 的基础，这张图其实还是比较容易理解的，下面我们来简单说下：

1. `React` 组件由用户触发事件，通过 `action creator` 发起 `action`
2. `sagas` 监听发起的 `action`，然后决定基于这个 `action` 做什么
3. 紧接着根据上步所做处理将 `effects`(sage的任务单元，简单的 `JavaScript` 对象)传给 `reducer`
4. 最后由 `reducer` 返回新的 `state`

### 安装

``` sh
yarn add redux-saga 
// npm install redux-saga -S
```

### 使用实例
#### 配置

``` js
import { createStore, combineReducers, applyMiddleware } from 'redux';
import global from './reducers/global';
import login from './reducers/login';
import createSagaMiddleware from 'redux-saga';       // 引入redux-saga中的createSagaMiddleware函数
import rootSaga from '../sagas';                    // 引入saga.js

const sagaMiddleware = createSagaMiddleware()        // 执行

const reducerAll = {
  global,
  login
}


export const store = createStore(
    combineReducers({...reducerAll}),               // 合并reducer
    applyMiddleware(sagaMiddleware)                 // 中间件，加载sagaMiddleware
)

sagaMiddleware.run(rootSaga)

```

#### ui组件触发action创建函数

``` js
handleSubmit = (e) => {
    this.props.form.validateFields((err, values) => {
      if (!err) {
        console.log('Received values of form: ', values);
        this.props.onSubmit(values);
      }
    });
  }

<Form onSubmit={this.handleSubmit} className="login-form">
    ...
</Form>
```
#### 将action传入saga

``` js
const mapDispatchToProps = dispatch => {
  return {
    onSubmit: (payload) => {
      dispatch({ type: LOGIN_REQUEST, ...payload })
    }
  }
}
```

#### saga捕获action创建函数返回的action(effects)

``` js
function* fetchLogin(userName, password) {
  try {
    console.log("saga:");
    const token = yield call(axios.get({ url: '' }))
    yield put({ type: 'LOGIN_SUCCESS', token })
  } catch(error) {
    yield put({ type: 'LOGIN_ERROR', error })
  }
}

export default function* watchIsLogin(dispatch) {
  while(true) {
    const { userName, password } = yield take('LOGIN_REQUEST');
    yield fork(fetchLogin, userName, password)
  }
}
```

#### reducer接受并返回新的state

``` js
const initState = {
  userName: 'Cola'
}

export default function(state = initState, action){
  switch (action.type) {
    case LOGIN_IN:
      return {...state, ...action.payload};
    default:
      return state;
  }
}
```
看了上面的步骤是不是觉得整体逻辑还是非常清晰明了的，下面我们把上面代码中未讲解到的代码名词解释下。

### 名词解释
#### Effect
一个 `effect` 就是一个纯文本 `javascript` 对象，包含一些将被 `saga middleware` 执行的指令。那么如何创建 `effect` 呢？使用下面我们马上要讲到API中 `redux-saga` 提供的工厂函数来创建 `effect`

#### Task
一个 `task` 就像是一个在后台运行的进程。在基于 `redux-saga` 的应用程序中，可以同时运行多个 `task`。通过 `fork` 函数来创建 `task`：

#### 阻塞调用/非阻塞调用
阻塞调用的意思是，`Saga` 在 `yield Effect` 之后会等待其执行结果返回，结果返回后才会恢复执行 `Generator` 中的下一个指令。

非阻塞调用的意思是，`Saga` 会在 `yield Effect` 之后立即恢复执行。

``` js
function* saga() {
  yield take(ACTION)              // 阻塞: 将等待 action
  yield call(ApiFn, ...args)      // 阻塞: 将等待 ApiFn (如果 ApiFn 返回一个 Promise 的话)
  yield call(otherSaga, ...args)  // 阻塞: 将等待 otherSaga 结束

  yield put(...)                   // 阻塞: 将同步发起 action (使用 Promise.then)

  const task = yield fork(otherSaga, ...args)  // 非阻塞: 将不会等待 otherSaga
  yield cancel(task)                           // 非阻塞: 将立即恢复执行
  // or
  yield join(task)                             // 阻塞: 将等待 task 结束
}
```

#### Watcher/Worker
指的是一种使用两个单独的 `Saga` 来组织控制流的方式。

- Watcher: 监听发起的 `action` 并在每次接收到 `action` 时 `fork` 一个 `worker`。
- Worker: 处理 `action` 并结束它。

``` js
function* watcher() {
  while(true) {
    const action = yield take(ACTION)
    yield fork(worker, action.payload)
  }
}

function* worker(payload) {
  // ... do some stuff
}
```

### Middleware API
#### createSagaMiddleware(options)
创建一个 `Redux middleware`，并将 `Sagas` 连接到 `Redux Store`。

`options` : `Object` - 传递给 `middleware` 的选项列表。

#### middleware.run(saga, ...args)
动态执行 `saga`。用于 `applyMiddleware` 阶段之后执行 `Sagas`。这个方法返回一个
`Task` 描述对象。

### Saga辅助函数
#### takeEvery
在发起（`dispatch`）到 `Store` 并且匹配 `pattern` 的每一个 `action` 上派生一个 `saga`。简单来说就是监听所有的匹配到的 `action`。

``` js
import { takeEvery } from `redux-saga/effects`

function* fetchUser(action) {
  ...
}

function* watchFetchUser() {
  yield takeEvery('USER_REQUESTED', fetchUser)
}
```

#### takeLatest
在发起到 `Store` 并且匹配 `pattern` 的每一个 `action` 上派生一个 `saga`。并自动取消之前所有已经启动但仍在执行中的 `saga` 任务。

``` js
import { takeLatest } from `redux-saga/effects`

function* fetchUser(action) {
  ...
}

function* watchLastFetchUser() {
  yield takeLatest('USER_REQUESTED', fetchUser)
}
```
这样便可以保证：即使用户以极快的速度连续多次触发 `USER_REQUESTED` `action`，我们都只会以最后的一个结束。

#### takeLeading
在发起到 `Store` 并且匹配 `pattern` 的每一个 `action` 上派生一个 `saga`。 它将在派生一次任务之后阻塞，直到派生的 `saga` 完成，然后又再次开始监听指定的 `pattern`。

``` js
import { takeLeading } from `redux-saga/effects`

function* fetchUser(action) {
  ...
}

function* watchLastFetchUser() {
  yield takeLeading('USER_REQUESTED', fetchUser)
}
```
由于 `takeLeading` 在其开始之后便无视所有新传入的任务，我们便可以保证：如果用户以极快的速度连续多次触发 `USER_REQUESTED action`，我们都只会保持以第一个 `action` 运行。

#### throttle
它在派生一次任务之后，仍然将新传入的 `action` 接收到底层的 `buffer` 中，至多保留（最近的）一个。但与此同时，它在 `ms` 毫秒内将暂停派生新的任务 —— 这也就是它被命名为节流阀（`throttle`）的原因。其用途，是在处理任务时，无视给定的时长内新传入的 `action`。

``` js
import { call, put, throttle } from `redux-saga/effects`

function* fetchAutocomplete(action) {
  const autocompleteProposals = yield call(Api.fetchAutocomplete, action.text)
  yield put({type: 'FETCHED_AUTOCOMPLETE_PROPOSALS', proposals: autocompleteProposals})
}

function* throttleAutocomplete() {
  yield throttle(1000, 'FETCH_AUTOCOMPLETE', fetchAutocomplete)
}
```
上面的代码我们通过 `throttle` 无视了一段时间内连续的 `FETCH_AUTOCOMPLETE`，我们便可以确保用户不会因此向我们的服务器发起大量请求。

### Effect 创建器
> 注意:
> - 以下每个函数都会返回一个普通 Javascript 对象（plain JavaScript > > object），并且不会执行任何其它操作。
> - 执行是由 middleware 在上述迭代过程中进行的。
> - middleware 会检查每个 Effect 的描述信息，并进行相应的操作

#### take
创建一个 `Effect` 描述信息，用来命令 `middleware` 在 `Store` 上等待指定的 `action`。 在发起匹配的 `action` 之前，`Generator` 将暂停。

#### put
创建一个 `Effect` 描述信息，用来命令 `middleware` 向 `Store` 发起一个 `action`。 这个 `effect` 是非阻塞型的，并且所有向下游抛出的错误（例如在 `reducer` 中），都不会冒泡回到 `saga` 当中。

#### call(fn, ...args)
创建一个 `Effect` 描述信息，用来命令 `middleware` 以参数 `args` 调用函数 `fn` 。

#### fork(fn, ...args)
创建一个 `Effect` 描述信息，用来命令 `middleware` 以 **非阻塞调用** 的形式执行 `fn`。

#### join(task)
创建一个 `Effect` 描述信息，用来命令 `middleware` 等待之前的一个分叉任务的结果。

#### cancel(task)
创建一个 `Effect` 描述信息，用来命令 `middleware` 取消之前的一个分叉任务。

#### select(selector, ...args)
创建一个 `Effect`，用来命令 `middleware` 在当前 `Store` 的 `state` 上调用指定的选择器（即返回 `selector(getState(), ...args)` 的结果）

例如，假设我们在应用程序中有这样结构的一份 `state`：

``` js
state = {
  cart: {...}
}
```
我们创建一个 **选择器（selector）**，即一个知道如果从 `State` 中提取 `cart` 数据的函数：

``` js
// selectors.js
export const getCart = state => state.cart
```
然后，我们可以使用 `select Effect` 从 `Saga` 的内部使用该选择器：

``` js
import { take, fork, select } from 'redux-saga/effects'
import { getCart } from './selectors'

function* checkout() {
  // 使用被导出的选择器查询 state
  const cart = yield select(getCart)

  // ... 调用某些 API，然后发起一个 success/error action
}

export default function* rootSaga() {
  while (true) {
    yield take('CHECKOUT_REQUEST')
    yield fork(checkout)
  }
}
```

#### actionChannel(pattern, [buffer])
创建一个 `Effect`，用来命令 `middleware` 通过一个事件 `channel` 对匹配 `pattern` 的 `action` 进行排序。 作为可选项，你也可以提供一个 `buffer` 来控制如何缓存排序的 `actions`。

``` js
import { actionChannel, call } from 'redux-saga/effects'
import api from '...'

function* takeOneAtMost() {
  const chan = yield actionChannel('USER_REQUEST')
  while (true) {
    const {payload} = yield take(chan)
    yield call(api.getUser, payload)
  }
}
```

#### flush(channel)
创建一个 `Effect`，用来命令 `middleware` 从 `channel` 中冲除所有被缓存的数据。被冲除的数据会返回至 `saga`，这样便可以在需要的时候再次被利用。

``` js
function* saga() {
  const chan = yield actionChannel('ACTION')

  try {
    while (true) {
      const action = yield take(chan)
      // ...
    }
  } finally {
    const actions = yield flush(chan)
    // ...
  }

}
```

#### cancelled()
创建一个 `Effect`，用来命令 `middleware` 返回该 `generator` 是否已经被取消。通常你会在 `finally` 区块中使用这个 `Effect` 来运行取消时专用的代码。

``` js
function* saga() {
  try {
    // ...
  } finally {
    if (yield cancelled()) {
      // 只应在取消时执行的逻辑
    }
    // 应在所有情况下都执行的逻辑（例如关闭一个 channel）
  }
}
```

#### setContext(props)
创建一个 `effect`，用来命令 `middleware` 更新其自身的上下文。这个 `effect` 扩展了 `saga` 的上下文，而不是代替。

#### getContext(prop)
创建一个 `effect`，用来命令 `middleware` 返回 `saga` 的上下文中的一个特定属性。

### Effect 组合器
#### race(effects)
创建一个 `Effect` 描述信息，用来命令 `middleware` 在多个 `Effect` 间运行 竞赛（`Race`）（与 `Promise.race([...])` 的行为类似）。

``` js
import { take, call, race } from `redux-saga/effects`
import fetchUsers from './path/to/fetchUsers'

function* fetchUsersSaga {
  const { response, cancel } = yield race({
    response: call(fetchUsers),
    cancel: take(CANCEL_FETCH)
  })
}
```

#### all(effects)
创建一个 `Effect` 描述信息，用来命令 `middleware` 并行地运行多个 `Effect`，并等待它们全部完成。这是与标准的 `Promise#all` 相当对应的 `API`。

``` js
import { fetchCustomers, fetchProducts } from './path/to/api'
import { all, call } from `redux-saga/effects`

function* mySaga() {
  const { customers, products } = yield all({
    customers: call(fetchCustomers),
    products: call(fetchProducts)
  })
}
```
### 接口
#### Task

方法 | 返回值
---|---
task.isRunning()        | 若任务还未返回或抛出了一个错误则为 true
task.isCancelled()      | 若任务已被取消则为 true
task.result()           | 任务的返回值。若任务仍在运行中则为 `undefined`
task.error()            | 任务抛出的错误。若任务仍在执行中则为 `undefined`
task.done               | 一个 Promise
task.cancel()           | 取消任务（如果任务仍在执行中）

#### Channel

方法 | 返回值
---|---
Channel.take(callback)        | 用于注册一个 taker
Channel.put(message)    | 用于在 buffer 上放入消息
Channel.flush(callback)           | 用于从 channel 中提取所有被缓存的消息
Channel.close()            | 关闭 channel，意味着不再允许做放入操作

#### Buffer

方法 | 返回值
---|---
Buffer.isEmpty()        | 如果缓存中没有消息则返回
Buffer.put(message)    | 用于往缓存中放入新的消息
Buffer.take()            | 用于检索任何被缓存的消息

### SagaMonitor
用于由 `middleware` 发起监视（`monitor`）事件。实际上，`middleware` 发起 5 个事件：
- 当一个 `effect` 被触发时（通过 `yield someEffect`），`middleware` 调用 `sagaMonitor.effectTriggered(options)` : `options` 包括:
    - `effectId` : `Number` - 分配给 `yielded effect` 的唯一 `ID`
    
    - `parentEffectId` : `Number` - 父级 `Effect` 的 `ID`。在 `race` 或 `parallel` `effect` 的情况下，所有在内部 `yield` 的 `effect` 都将有一个直接 `race/parallel` 的父级 `effect`。在最顶级的 `effect` 的情况下，父级是包裹它的 `Saga`。
    
    - `label` : `String` - 在 `race effect` 的情况下，所有子 `effect` 都将被指定为传递给 `race` 的对象中对应键的标签。
    
    - `effect` : `Object` - `yielded effect` 其自身

- 如果该 `effect` 成功地被 `resolve`，则 `middleware` 调用 `sagaMonitor.effectResolved(effectId, result)`
    - `effectId` : `Number` - `yielded effect` 的 `ID`

    - `result` : `any` - 该 `effect` 成功 `resolve` 的结果。在 `fork` 或 `spawn` 的情况下，结果将是一个 `Task` 对象。

- 如果该 `effect` 因一个错误被 `reject`，则 `middleware` 调用 `sagaMonitor.effectRejected`
    - `effectId` : `Number` - `yielded effect` 的 `ID`

    - `error` : `any` - 该 `effect reject` 的错误

- 如果该 `effect` 被取消，则 `middleware` 调用 `sagaMonitor.effectCancelled`
    - `effectId` : `Number` - `yielded effect` 的 `ID`

- 最后，当 `Redux action` 被发起时，`middleware` 调用 `sagaMonitor.actionDispatched`
    - `action` : `Object` - 被发起的 `Redux action`。如果该 `action` 是由一个 `Saga` 发起的，那么该 `action` 将拥有一个属性 `SAGA_ACTION` 并被设为 `true`（你可以从 `redux-saga/utils` 中导入 `SAGA_ACTION`）。

### 外部API
不做赘述，可根据自己需要使用






