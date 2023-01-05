

## Router



```js
// src/router/index.js

import Vue from 'vue'
import VueRouter from 'vue-router'
import store from '../store'
import get from 'lodash/get'

import ControllerLayout from '@/components/Layout/controllerLayout'
import Unauthorized from '@/views/unauthorized'
import NotFound from '@/views/notFound'
import Login from '@/views/login'
import Summary from '@/views/controller/summary'
import controller from './controller'
import user from './user'
import message from './message'
import manage from './manage'

Vue.use(VueRouter)

const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes: [
    {
      path: '',
      redirect: {
        name: 'Summaries'
      }
    },
    // 全部资源池
    {
      path: '/controllers',
      name: 'Controllers',
      component: ControllerLayout,
      redirect: {
        name: 'Summaries'
      },
      meta: {
        requiresAuth: true
      },
      children: [
        {
          path: 'Summaries',
          name: 'Summaries',
          meta: {
            title: '摘要',
            menuName: 'Summary',
            requiresAuth: true
          },
          isMenu: true,
          component: Summary
        }
      ]
    },
    // 单一资源池
    {
      path: '/controller/:controller',
      name: 'Controller',
      component: ControllerLayout,
      meta: {
        requiresAuth: true
      },
      redirect: {
        name: 'Summary'
      },
      children: controller.map((item) => {
        return {
          ...item,
          meta: {
            ...item.meta
          }
        }
      })
    },
    // 个人中心/活动记录
    {
      path: '/user',
      name: 'User',
      component: ControllerLayout,
      meta: {
        requiresAuth: true
      },
      redirect: {
        name: 'Personal'
      },
      children: user.map((item) => {
        return {
          ...item,
          meta: {
            ...item.meta
          }
        }
      })
    },
    // 消息列表
    {
      path: '/message',
      name: 'Message',
      component: ControllerLayout,
      meta: {
        requiresAuth: true
      },
      redirect: {
        name: 'MessageList'
      },
      children: message.map((item) => {
        return {
          ...item,
          meta: {
            ...item.meta
          }
        }
      })
    },
    // 权限管理
    {
      path: '/manage',
      name: 'Manage',
      component: ControllerLayout,
      meta: {
        requiresAuth: true
      },
      redirect: {
        name: 'Account'
      },
      children: manage.map((item) => {
        return {
          ...item,
          meta: {
            ...item.meta
          }
        }
      })
    },
    // 登录页
    {
      path: '/login',
      name: 'Login',
      meta: {
        title: '登录'
      },
      component: Login
    },
    { path: '/unauthorized', name: 'Unauthorized', component: Unauthorized },
    { path: '*', component: NotFound }
  ]
})

router.beforeEach(async (to, from, next) => {
  // 登录判断
  const requiresAuth = to.matched.some(record => record.meta.requiresAuth)

  if (requiresAuth) {
    const userInfo = JSON.parse(localStorage.getItem('userInfo'))
    const tokenInfo = JSON.parse(localStorage.getItem('tokenInfo'))
    if (tokenInfo && tokenInfo.expireTime > Date.now()) {
      // 已登录，在登录页已经将 userInfo tokenInfo 存储到 localStorage
      // 每进入一个页面都更新 store 的 userInfo
      store.commit('saveUserInfo', userInfo)
      // 判断当前用户是否有进入该页面的权限
      const permissions = get(store, ['state', 'userInfo', 'permissions'], []).map(i => i.id)
      if (to.meta.permission && !permissions.includes(to.meta.permission)) {
        // 无权限时
        return next({
          name: 'Unauthorized'
        })
      }
      next()
    } else {
      // 未登录时
      console.log('authenticating a protected url:' + to.path)
      store.commit('saveUserInfo', null)
      next({
        name: 'Login'
      })
    }
  } else {
    next()
  }
})

export default router

```

需要登录才能访问的页面会在 meta 中添加 `requireAuth`

路由的 meta 中的 `requireAuth` 用于在路由前置守卫中判断是否登录（`使用登录后存储在 localStorage 中的 token 来判断是否登录`），不需要登录的则可以直接进入

而 meta 中的 `permission` 用于权限控制，userInfo 中的 permission 是一个权限的数组，表示当前用户拥有哪些权限



子路由文件：

```js
// src/router/controller.js

import Summary from '@/views/controller/summary'
import SNode from '@/views/controller/server'
import ServerDevice from '@/views/controller/device'
import Task from '@/views/controller/task'
import TaskDetail from '@/views/controller/task/detail'
import Log from '@/views/controller/log'
import LicenseList from '@/views/controller/activate'
import LicenseDetail from '@/views/controller/activate/detail'
import Schedule from '@/views/controller/schedule'
import Server from '@/views/controller/server/detail'
import ServerOverview from '@/views/controller/server/overview'
import Monitor from '@/views/controller/monitor'

// 二级路由下面的三级路由需要 render 渲染出一个 router-view 放置子路由
const Skeleton = { render: h => <router-view></router-view> }

export default [
  {
    path: 'summary',
    name: 'Summary',
    meta: {
      title: '摘要',
      icon: 'summary',
      menuName: 'Summary'
    },
    isMenu: true,
    component: Summary
  },
  {
    path: 'snode',
    name: 'ServerNode',
    meta: {
      title: '节点',
      icon: 'resource',
      menuName: 'ServerNode'
    },
    isMenu: true,
    component: SNode
  },
  {
    path: 'device',
    name: 'ServerDevice',
    meta: {
      title: '设备',
      icon: 'equipment',
      menuName: 'ServerDevice'
    },
    isMenu: true,
    component: ServerDevice
  },
  {
    path: 'server/:server',
    name: 'Server',
    meta: {
      title: '服务节点',
      icon: 'ios-home'
    },
    redirect: {
      name: 'ServerOverview'
    },
    component: Server,
    children: [
      {
        path: 'overview',
        name: 'ServerOverview',
        meta: {
          menuName: 'ServerNode'
        },
        component: ServerOverview
      },
      {
        path: 'task',
        name: 'ServerTask',
        meta: {
          menuName: 'ServerNode'
        },
        component: Task
      },
      {
        path: 'task/:id',
        name: 'ServerTaskDetail',
        meta: {
          title: '任务详情',
          menuName: 'ServerNode'
        },
        component: TaskDetail
      },
      {
        path: 'log',
        name: 'ServerLog',
        meta: {
          menuName: 'ServerNode'
        },
        component: Log
      }
    ]
  },
  {
    path: 'task',
    name: 'Task',
    meta: {
      title: '任务',
      icon: 'task',
      menuName: 'Task'
    },
    isMenu: true,
    component: Skeleton,
    redirect: {
      name: 'TaskList'
    },
    children: [
      {
        path: '',
        name: 'TaskList',
        meta: {
          menuName: 'Task'
        },
        component: Task
      },
      {
        path: ':id',
        name: 'TaskDetail',
        meta: {
          title: '任务详情',
          menuName: 'Task'
        },
        component: TaskDetail
      }
    ]
  },
  {
    path: 'monitor',
    name: 'Monitor',
    meta: {
      title: '监控',
      icon: 'monitor',
      menuName: 'Monitor'
    },
    isMenu: true,
    component: Monitor
  },
  {
    path: 'log',
    name: 'Log',
    meta: {
      title: '日志',
      icon: 'log',
      menuName: 'Log'
    },
    isMenu: true,
    component: Log
  },
  {
    path: 'schedule',
    name: 'Schedule',
    meta: {
      title: '调度',
      icon: 'schedule',
      menuName: 'Schedule'
    },
    isMenu: true,
    component: Schedule
  },
  {
    path: 'activate',
    name: 'Activate',
    meta: {
      title: '激活',
      icon: 'activate',
      menuName: 'Activate'
    },
    isMenu: true,
    component: Skeleton,
    redirect: {
      name: 'LicenseList'
    },
    children: [
      {
        path: '',
        name: 'LicenseList',
        meta: {
          menuName: 'Activate'
        },
        component: LicenseList
      },
      {
        path: ':id',
        name: 'LicenseDetail',
        meta: {
          title: 'license详情',
          menuName: 'Activate'
        },
        component: LicenseDetail
      }
    ]
  }
]

```



```js
// src/router/user.js

import Personal from '@/views/user/personal'
import OperationRecord from '@/views/user/operationRecord'

export default [
  {
    path: 'personal',
    name: 'Personal',
    meta: {
      title: '个人中心'
    },
    isMenu: true,
    component: Personal
  },
  {
    path: 'operationRecord',
    name: 'OperationRecord',
    meta: {
      title: '活动记录'
    },
    isMenu: true,
    component: OperationRecord
  }
]

```



```js
// src/router/message.js

import MessageList from '@/views/message/index'
import MessageExplain from '@/views/message/explain'

export default [
  {
    path: '',
    name: 'MessageList',
    meta: {
      title: '消息列表'
    },
    component: MessageList
  },
  {
    path: 'explain',
    name: 'MessageExplain',
    meta: {
      title: '消息说明'
    },
    component: MessageExplain
  }
]

```



```js
// src/router/manage.js

import Account from '@/views/manage/account'
import UserList from '@/views/manage/account/user'
import Role from '@/views/manage/account/role'
import messageManage from '@/views/manage/message'

export default [
  {
    path: 'account',
    name: 'Account',
    meta: {
      title: '权限管理',
      permission: 100000
    },
    isMenu: true,
    redirect: {
      name: 'AccountList'
    },
    component: Account,
    children: [
      {
        path: 'list',
        name: 'AccountList',
        meta: {
          permission: 100000 // 账号管理权限
        },
        component: UserList
      },
      {
        path: 'role',
        name: 'AccountRole',
        meta: {
          permission: 100000 // 账号管理权限
        },
        component: Role
      }
    ]
  },
  {
    path: 'message',
    name: 'MessageManage',
    meta: {
      title: '消息管理',
      permission: 100000
    },
    isMenu: true,
    component: messageManage
  }
]

```

