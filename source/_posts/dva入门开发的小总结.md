---
title: dva入门开发的小总结
date: 2018-01-05 19:53:05
categories: React
tags: Dva
---
## dva入门开发的小总结
`dva 是对 redux 的一层浅封装，基于 react 语言，实现前端代码的分层`

##### 一般会分为3层：
1. component：组件渲染、展示页面
2. models：数据组装、处理
3. services：接口调用、拿数据

##### 框架主要构成模块：

<!--more-->

**1. `router.js` 文件中配置路由，指定具体路径所要加载的 views 和 models**
```
{
    path: '/security/users',
    models: () => [import('./models/security/user')],
    component: () => import('./views/security/user/'),
}
```
**2. services**

`.roadhogrc.js`文件中配置好代理请求地址
```
proxy: {
    "/api/v1/weather": {
      "target": "https://api.seniverse.com/",
      "changeOrigin": true,
      "pathRewrite": { "^/api/v1/weather" : "/v3/weather" }
    },
    "/services": {
        "target": "http://ip地址:8080/",
        "changeOrigin": true,
        "pathRewrite": { "^/services" : "/services" }
      },
  },
```
`serviecs/user.js`中定义好接口，获取原始数据。

对于自定义方法，提供 url、method、data（可选），返回 request 封装好的 http 请求。
```
export async function query({page = 1 , pageSize = 10 , ...qs}) {
    return user.listPage(page,pageSize,qs)
}

export async function get({id}) {
    return user.get(id)
}

export async function create (params) {
    return user.create(params);
}

export async function update (params) {
    return user.update(params);
}

// 用户设置 自定义方法
//修改用户
export async function modify(payload) {
    return request({
        url: `${apiPrefix}/security/user/modify`,
        method: 'put',
        data: payload
    })
}
```

**3. models**
```
import pathToRegexp from 'path-to-regexp'
import modelExtend from 'dva-model-extend'
import * as user from 'services/security/user'
import { pageModel } from 'models/common'
import { message } from 'antd'

export default modelExtend(pageModel, {

  namespace: 'secUser',

  state: {
   currentItem: {},
   modalVisible: false,
   modalType: 'create',
   selectedRowKeys: [],
  },

  // 添加一个监听器，当pathname === '/security/users'时执行dispatch
  subscriptions: {
    setup ({ dispatch, history }) {
      history.listen((location) => {
        if (location.pathname === '/security/users') {
          dispatch({
              type: 'query',
              payload: location.query || {},
          })
        }
      })
    },
  },

//
  effects: {
    * query ({
      payload,
    }, { call, put }) {
      const result = yield call(user.query, payload)
      const { success, message, status, data } = result
      if (success) {
        yield put({
          type: 'querySuccess',
          payload: data,
        })
      } else {
        throw result
      }
    },

    * create ({ payload }, { call, put,select }) {
      const {data} = yield call(user.create, payload)
      message.info("新增成功");
      yield put({ type: 'hideModal' })
      const { pagination: { pageSize,current } } = yield select(_ => _.secUser)
      yield put({ type: 'query', payload: {pageSize, page:current } })
    },

    * update ({ payload }, { select, call, put }) {
      const id = yield select(({ secUser }) => secUser.currentItem.id)
      const newUser = { ...payload, id }
      const {data} = yield call(user.update, newUser)
      message.info("修改成功");
      yield put({ type: 'hideModal' })
      const { pagination: { pageSize,current } } = yield select(_ => _.secUser)
      yield put({ type: 'query', payload: {pageSize, page:current } })
    },
  },

  reducers: {
     showModal (state, { payload }) {
      return { ...state, ...payload, modalVisible: true }
    },

    hideModal (state) {
      return { ...state, modalVisible: false }
    },
  },
})
```
- **query、update 前面的 * 号，表示这个方法是一个 Generator函数**

- **subscriptions：用于添加一个监听器**

- **effects：异步执行 dispatch，用于发起异步请求**
```
call 是调用执行一个函数 (调用service里面的方法)
put 则是相当于 dispatch 执行一个 action
select 则可以用来访问其它 model
```
- **reducers：同步请求，主要用来改变state**

**4.component**

==注意：组件入口文件一定要命名为 index.js ，否则会找不到==
```
import React from 'react'
import PropTypes from 'prop-types'
import { routerRedux } from 'dva/router'
import { connect } from 'dva'
import { Row, Col, Button, Popconfirm } from 'antd'
import List from './List'
import Filter from './Filter'
import Modal from './Modal'
import { Page } from 'components'
import { i18n }  from 'utils'

const User = ({ location, dispatch, secUser, loading }) => {
  const { dataSource, pagination, currentItem, modalVisible, modalType, selectedRowKeys } = secUser
  const { pageSize } = 10

  const modalProps = {
    item: modalType === 'create' ? {} : currentItem,
    visible: modalVisible,
    maskClosable: false,
    confirmLoading: loading.effects['secUser/update'],
    title: modalType === 'create' ? i18n('lab.user.create') : i18n('lab.user.update'),
    wrapClassName: 'vertical-center-modal',
    onOk (data) {
      dispatch({
        type: `secUser/${modalType}`,
        payload: data,
      })
    },
    onCancel () {
      dispatch({
        type: 'secUser/hideModal',
      })
    },
  }

  const listProps = {
    dataSource,
    loading: loading.effects['secUser/query'],
    pagination,
    location,
    onChange (page) {
      const { query, pathname } = location
      dispatch(routerRedux.push({
        pathname,
        query: {
          ...query,
          page: page.current,
          pageSize: page.pageSize,
        },
      }))
    },
    onDeleteItem (id) {
      dispatch({
        type: 'secUser/delete',
        payload: id,
      })
    },
    onEditItem (item) {
      dispatch({
        type: 'secUser/showModal',
        payload: {
          modalType: 'update',
          currentItem: item,
        },
      })
    },
    rowSelection: {
      selectedRowKeys,
      onChange: (keys) => {
        dispatch({
          type: 'secUser/updateState',
          payload: {
            selectedRowKeys: keys,
          },
        })
      },
    },
  }

  const filterProps = {
    ...
  }

  const handleDeleteItems = () => {
    dispatch({
      type: 'secUser/multiDelete',
      payload: {
        ids: selectedRowKeys,
      },
    })
  }

  return (
    <Page inner>
      <Filter {...filterProps} />
      <List {...listProps} />
      {modalVisible && <Modal {...modalProps} />}
    </Page>
  )
}

User.propTypes = {
  secUser: PropTypes.object,
  location: PropTypes.object,
  dispatch: PropTypes.func,
  loading: PropTypes.object,
}

export default connect(({ secUser }) => ({ secUser}))(User)
```
- **connect方法将组件与数据关联在一起**
`export default connect(({ secUser }) => ({ secUser}))(User)`
- **secUser 必须与对应 models 里面定义的 namespace 字段一致**

- **组件中发起请求调用 dispatch**
```
type 字段对应 models 中 effects 和 reducers 对应方法
payload 为传参对象
```

[尊重原创，感谢原创分享](https://www.jianshu.com/p/6d4c70ef63be)