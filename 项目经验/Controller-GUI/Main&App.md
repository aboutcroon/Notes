## Main.js



```js
// Main.js

import Vue from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'
import ViewUI from 'view-design'
import moment from 'moment'
import 'moment/locale/zh-cn'
// import cat from 'virtai-monitor-sdk'
import messageTip from '@/common/message'
import component from '@/common/plugins/component'
import directives from '@/common/directives'
// import '@/common/dt'
import '@/common/plugins/antd.js'
import '@/assets/style/common.less'
import '@/assets/theme/iview.less'
import '@/assets/theme/antd.less'
import '@/assets/font/iconfont.css'
console.log('%c 当前环境：%s', 'color: red', process.env.NODE_ENV)
console.log('%c 当前版本：%s', 'color: orange', process.env.VUE_APP_BUILD_VERSION)

// 右下角浮层
const NODE_ENV = process.env.NODE_ENV
if (NODE_ENV !== 'production') {
  const buildInfo = document.createElement('div')
  buildInfo.style = 'position: fixed; z-index: 99999; bottom: 0; right: 0; padding: 5px 15px; background: green; color: white;'
  buildInfo.textContent = '构建时间：' + process.env.VUE_APP_BUILD_VERSION
  document.body.appendChild(buildInfo)
}

// Mock 数据
if (NODE_ENV !== 'production') {
  const modules = require.context('../mock', true, /\.js$/)
  modules.keys().forEach(m => modules(m))
}

moment.locale('zh-cn')
Vue.config.productionTip = false
Vue.use(ViewUI)
Vue.use(messageTip)
Vue.use(component)
Vue.use(directives)
const vueInstance = new Vue({
  router,
  store,
  render: h => h(App)
}).$mount('#app')

// 可能是为了在控制台便于调试
window.vueInstance = vueInstance

```



### 浮层

右下角的 `构建时间：xxx` 的浮层，可以在这里初始化

```js
const NODE_ENV = process.env.NODE_ENV
if (NODE_ENV !== 'production') {
  const buildInfo = document.createElement('div')
  buildInfo.style = 'position: fixed; z-index: 99999; bottom: 0; right: 0; padding: 5px 15px; background: green; color: white;'
  buildInfo.textContent = '构建时间：' + process.env.VUE_APP_BUILD_VERSION
  document.body.appendChild(buildInfo)
}
```

在 `vue.config.js` 中添加环境变量 `VUE_APP_BUILD_VERSION`

```js
// vue.config.js

process.env.VUE_APP_BUILD_VERSION = new Date().toLocaleString()
```



### Mock 数据

将 Mock 文件夹下的文件全部引入

```js
if (NODE_ENV !== 'production') {
  const modules = require.context('../mock', true, /\.js$/)
  modules.keys().forEach(m => modules(m))
}
```

这里用到了 `require.context` 方法

如果要在一个目录下引入多个文件，可以使用这个方法来减少 import 语句

`modules.keys()` 是引入的文件组成的数组

https://webpack.js.org/guides/dependency-management/#requirecontext

https://zhuanlan.zhihu.com/p/59564277



## App.vue



```vue
<template>
  <router-view v-if="display" />
</template>

<script>
// init root-service
import { sendMessage, EVENT, initSocket } from '@/common/socket'
import request from '@/common/request'
export default {
  data () {
    return {
      display: false
    }
  },
  created () {
    // 获取配置信息
    request.config().then(async res => {
      this.$store.commit('setConfig', res.data)
      this.display = true
      initSocket()
      const rootService = res.data.rootService
      const data = `${window.location.protocol}//${rootService.host}:${rootService.socketPort}`
      // 发送消息给 socket
      sendMessage({
        event: EVENT.CONNECT_ROOT_SERVICE,
        data
      })
    })
  }
}
</script>

```

先获取配置信息 config

获取 `root-service` 的 ip， port 和 socketPort

初始化 socket



```js
// src/common/socket/index.js

import io from 'socket.io-client'
import store from '@/store'
let socket = null
let socketReady = false
let socketReadyResolve
const socketReadyPromise = new Promise((resolve) => {
  socketReadyResolve = resolve
})
export const EVENT = {
  CONNECT: 'connect',
  DISCONNECT: 'disconnect',
  CONNECT_ROOT_SERVICE: 'CONNECT_ROOT_SERVICE',
  CONNECT_CONTROLLER: 'CONNECT_CONTROLLER',
  CONTROLLER_UPDATE: 'CONTROLLER_UPDATE',
  SERVER_UPDATE: 'SERVER_UPDATE',
  REVEIVE_MESSAGE: 'REVEIVE_MESSAGE'
}

// 发送消息给 socket
// 由于发送消息是根据不同业务来发送不同类型的消息，所以在外面定义，可以 export 出去使用
// 例如在 App.vue 中，我们先获取 config，config 中有 rootService 的 ip 和 port，然后再使用 sendMessage 传递 rootService 的 ip 和 port 给服务端来初始化 rootService
export async function sendMessage ({ event, data }) {
  if (!socketReady) {
    // 若 socket 连接未成功，则直到等再次 socketReadyResolve() 之后，再操作
    await socketReadyPromise
  }
  socket.emit(event, data)
}

/**
 * 处理 socket 通信传递的消息
 * 全局事件可以在初始化后处理，例如 CONTROLLER_UPDATE
 * 业务内的事件在业务内处理，例如 REVEIVE_MESSAGE，通过 getSocket 获取 socket
 */
function handleEvent () {
  socket.on(EVENT.CONTROLLER_UPDATE, function (data) {
    console.log(data)
    store.commit('setControllerList', typeof data === 'string' ? JSON.parse(data) : data)
  })
}

// 传递出 socket 实例，用于业务内的事件，例如 REVEIVE_MESSAGE，需要通过 getSocket 获取 socket
export async function getSocket () {
  if (!socketReady) {
    // 若 socket 连接未成功，则直到等再次 socketReadyResolve() 之后，再操作
    await socketReadyPromise
  }
  return socket
}

// 初始化 socket
export function initSocket () {
  const config = store.state.config
  const target = process.env.NODE_ENV === 'development' ? `http://localhost:${config.server.port}` : location.origin
  socket = io(target)
  socket.on(EVENT.CONNECT, function () {
    console.log('socket connect success')
    socketReady = true
    handleEvent()
    socketReadyResolve()
  })
  socket.on(EVENT.DISCONNECT, function () {})
}

```



RECEIVE_MESSAGE 的例子：

```js
// src/components/Header/message.vue

mounted () {
  this.getMessageList(true)
  this.getMessageUnreadCount()

  // 获取 socket 实例，然后监听 REVEIVE_MESSAGE 事件
  getSocket().then(socket => {
    socket.on(EVENT.REVEIVE_MESSAGE, (data) => {
      this.messageUnreadCount++
      data = typeof data === 'string' ? JSON.parse(data) : data
      data.read = false
      this.messageList.unshift(data)
    })
  })
}
```



