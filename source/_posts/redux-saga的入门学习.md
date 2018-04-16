---
title: redux-saga的入门学习
date: 2018-04-03 22:10:26
categories: React
tags: Redux
---
## redux-saga的入门学习

redux-saga 是一个用于管理 Redux 应用异步操作（Side Effects。译注：直译成 “副作用” 不太通顺，所以这里译为 “异步操作” 更好理解）的中间件（又称异步 action）。 redux-saga 通过创建 Sagas 将所有的异步操作逻辑收集在一个地方集中处理，可以用来代替 redux-thunk 中间件。

##### 这意味着应用的逻辑会存在两个地方：

1. Reducers 负责处理 action 的 state 更新。

2. Sagas 负责协调那些复杂或异步的操作。

<!--more-->

Sagas 是通过 Generator 函数来创建的。如果你还不熟悉 Generator，可以在这里找到 [一些有用的链接](http://superraytin.github.io/redux-saga-in-chinese/docs/ExternalResources.html)。

Sagas 不同于 Thunks，Thunks 是在 action 被创建时调用，而 Sagas 只会在应用启动时调用（但初始启动的 Sagas 可能会动态调用其他 Sagas）。 Sagas 可以被看作是在后台运行的进程。Sagas 监听发起的 action，然后决定基于这个 action 来做什么：是发起一个异步调用（比如一个 Ajax 请求），还是发起其他的 action 到 Store，甚至是调用其他的 Sagas。

在 redux-saga 的世界里，所有的任务都通用 yield Effects 来完成（译注：Effect 可以看作是 redux-saga 的任务单元）。Effects 都是简单的 Javascript 对象，包含了要被 Saga middleware 执行的信息（打个比方，你可以看到 Redux action 其实是一个个包含执行信息的对象）。 redux-saga 为各项任务提供了各种 Effect 创建器，比如调用一个异步函数，发起一个 action 到 Store，启动一个后台任务或者等待一个满足某些条件的未来的 action。

因为使用了 Generator，redux-saga 让你可以用同步的方式写异步代码。就像你可以使用 async/await 函数所能做的一样。但 Generator 可以让你做一些 async 函数做不到的事情。

事实上 Sagas yield 普通对象的方式让你能容易地测试 Generator 里所有的业务逻辑，可以通过简单地迭代 yield 过的对象进行简单的单元测试。

此外，redux-saga 启动的任务可以在任何时候通过手动取消，也可以把任务和其他的 Effects 放到 race 方法里以自动取消。

##### 安装

```
npm install --save redux-saga
```

你也可以直接在 HTML 页面中通过 `&lt;script&gt`; 标签使用提供的 UMD 构建版本，看 [这里](https://github.com/superRaytin/redux-saga-in-chinese#using-umd-build-in-the-browser)。

##### 使用示例
假设我们有一个 UI 界面，在单击按钮时从远程服务器获取一些用户数据（为简单起见，我们只列出 action 触发代码）。

```
class UserComponent extends React.Component {
  ...
  onSomeButtonClicked() {
    const { userId, dispatch } = this.props
    dispatch({type: 'USER_FETCH_REQUESTED', payload: {userId}})
  }
  ...
}
```

这个组件发起一个普通对象格式的 action 到 Store。我们将创建一个 Saga 来监听所有的 `USER_FETCH_REQUESTED` action，并触发一个 API 调用以获取用户数据。

###### sagas.js

```
import { takeEvery, takeLatest } from 'redux-saga'
import { call, put } from 'redux-saga/effects'
import Api from '...'

// workder Saga : 将在 USER_FETCH_REQUESTED action 被发起时调用
function* fetchUser(action) {
   try {
      const user = yield call(Api.fetchUser, action.payload.userId);
      yield put({type: "USER_FETCH_SUCCEEDED", user: user});
   } catch (e) {
      yield put({type: "USER_FETCH_FAILED", message: e.message});
   }
}

/*
  在每个 `USER_FETCH_REQUESTED` action 被发起时调用 fetchUser
  允许并发（译注：即同时处理多个相同的 action）
*/
function* mySaga() {
  yield* takeEvery("USER_FETCH_REQUESTED", fetchUser);
}

/*
  也可以使用 takeLatest

  不允许并发，发起一个 `USER_FETCH_REQUESTED` action 时，
  如果在这之前已经有一个 `USER_FETCH_REQUESTED` action 在处理中，
  那么处理中的 action 会被取消，只会执行当前的
*/
function* mySaga() {
  yield* takeLatest("USER_FETCH_REQUESTED", fetchUser);
}
```

为了能跑起 Saga，我们需要使用 redux-saga 中间件将 Saga 与 Redux Store 建立连接。

##### main.js

```
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'

import reducer from './reducers'
import mySaga from './sagas'

const sagaMiddleware = createSagaMiddleware(mySaga)
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)

// render the application
```


[Redux-saga 中文文档](https://github.com/superRaytin/redux-saga-in-chinese)