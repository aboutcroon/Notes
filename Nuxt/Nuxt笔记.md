## 1. 生命周期

> #### nuxtServerInit

服务器初始化时触发，放于`store/index.js`目录下的`actions`操作中，用于`store`状态树的初始操作

```js
export const actions = {
  nuxtServerInit(store, context) {  // 可以拿到store的实例，context上下文
    // 在这个钩子里，初始化东西到store中
    store.commit('xxx', xxxx)   // 将cookie中的字段同步到Vuex
  },
}
```



> #### middleware

中间件运行时触发，可放于nuxt.config.js中，layouts中，pages页面中（执行顺序为nuxt.config.js --> layouts --> pages）。可用于写一些自定义的函数，运行在全局，或某个页面前。一般可用于做前置守卫

```js
export default ({store, route, redirect, params, query, req, res}) => {
    // 在这里可以写全局守卫的逻辑
    // context是服务端上下文，包括store, route, redirect, params, query, req, res等
    // store 状态树信息
    // route 一条目标路由信息
    // redirect 强制跳转
    // params query 校验参数合理性
    // 通过上述参数来写全局前置守卫
    console.log('middleware')
}
```



> #### validate

在页面中触发，只放于pages页面中，用于进入页面前的参数校验（此时页面和组件还未初始化，用于进入页面前的拦截）。可对页面和组件进行一些动态参数的有效性的请求，校验客户端的一些数据到了服务端之后是否正确，校验成功则进入页面，校验失败则返回404页面

```js
export default {
  name: 'detail',
  validate({params, query}) {   // 若父级目录下默认展示第一个子路由，则第一个子路由未通过的话，会直接是错误页面，所以可以加一个index.vue使得默认展示index页面
    return typeof params.id === 'number'	// 类型是number时才进入此页面
  }
}
```



> #### asyncData() & fetch()

在页面中触发，只放于pages页面中，数据都可以在这两个中去读取

```js
export default {
  name: 'index',
  async asyncData({ $axios }) {
    // context中包括$axios，可以直接提取出$axios
    let res = await $axios({ url: '/data/list.json' })

    return {
      // 将抓取到的数据合并到组件内部，此时template中的title则会展示成相应的数据
      title: res.data.title,
    }
  },
}
```

由于`asyncData`方法是在组件 **初始化** 前被调用的，所以在方法内是没有办法通过 `this` 来引用组件的实例对象。

`fetch`方法用于在渲染页面前填充应用的状态树（store）数据， 与 `asyncData` 方法类似，不同的是它不会设置组件的数据。



> #### render

即将做服务端渲染时触发`render`，我们在这之前要把数据准备好，通过`render`将渲染后的页面返回给浏览器，`render`钩子只能定义渲染的一些配置，不能写业务逻辑，写了也不执行。

当用户再次请求新的页面时，会再次跳转到middleware这个钩子（只有在强制刷新时才会重新触发nuxtServerInit），然后再往后面的生命周期继续执行。只要数据发生了变化，render在服务端都会重新渲染



> #### beforeCreate和created

`beforeCreate`和`created`在客户端加载完之前是运行在服务端的，所以在这两个钩子中取不到`window`对象



## 2. 路由

若不自行配置的话，Nuxt.js路由默认采用的是约定式路由

> #### 展示区

有`<nuxt-link></nuxt-link>`时，需要有子路由的展示区，展示区是`<nuxt/>`



> #### 声明式跳转

name: 路由名，格式为目录名-其他目录-文件名

params: key值，要对等文件名

可以以字符串或者以对象的形式来传递参数

```vue
// 以字符串形式传递参数，这样子params的id则是字符串形式
<nuxt-link to="/goods/1?a=1&b=2">商品01</nuxt-link>
<nuxt-link to="/goods/2?a=11&b=22">商品02</nuxt-link>

// 以对象形式传递参数，这样子params的id是数字形式
<nuxt-link :to="{name: 'goods-id', params: {id: 3}, query: {a: 111, b: 222}}">商品03</nuxt-link>
```



> #### 子路由

目录代表子路由，子路由内部同级的文件，代表的是同级一级路由



> #### 展示区层级

当我们希望展示区的子路由展示在外面一层级时，我们则需要将外层的goods.vue命名为index.vue，并添加到goods文件夹中（向下移一个层级），与_id.vue同级

当一个目录下只有子文件而没有index.vue，切换到该目录时会默认展示第一个子文件，若不希望如此，则可以在目录下添加一个空的index.vue



> #### 扩展路由

在nuxt.config.js的router下配置extendRoutes

```js
router: {
    extendRoutes(routes, resolve) {   // 扩展路由的配置
      console.log(routes)
      routes.push({
        name: 'home',
        path: '/index',
        component: resolve(__dirname, 'pages/index.vue')
      })
    }
  }
```



> #### 参数校验

当跳转到某个子路由，路由名为id，传递过来的params中携带了id，此时可能要校验该id，这时就要用到validate钩子



> #### 定制错误页面

在`layouts`下新建`error.vue`

```vue
<template>
  <div>
    <!-- 展示错误信息 -->
    <h1 v-if="error.statusCode">{{ error.message }}</h1>
    <h1 v-else>应用发送异常</h1>
    <button @click="$router.replace('/index')"></button>
  </div>
</template>

<script>
export default {
  props: ['error'], // 接收错误信息 error: {statusCode, message}
}
</script>

<style>
</style>
```



> #### 统一动效，独享动效

- 统一动效

```js
// Global CSS (https://go.nuxtjs.dev/config-css)
  css: [
    'assets/xxx.css'	// 在此引入全局效果
  ],
```

- 独立动效

若想要有独立的效果，可在相对应的页面中添加transition选项

```vue
<template>
  <div class="detail">
    <h3>详情</h3>
  </div>
</template>

<script>
export default {
  name: 'detail',
  transition: 'test'  // 单独的transition动态效果
}
</script>

<style scoped>
  /*名为test的过渡效果，也可以将以下代码添加到一个独立css文件中，然后引入到nuxt.config.js文件中*/
  .test-enter-active, .test-leave-active {
    transition: .5s ease all;
  }
  .test-enter, .test-leave-active {
    margin-left: -1000px;
  }
</style>
```



> #### 路由守卫

**前置守卫**

- 依赖中间件`middleware`和插件

- 全局守卫：`middleware`定义在`nuxt.config.js`中或`layouts`中。在`layouts`的`default.vue`中定义`middleware`时，凡是在该文件中布局的页面都会有效果

```js
// middleware/auth.js

export default ({store, route, redirect, params, query, req, res}) => {
    // 在这里可以写全局守卫的逻辑
    // context是服务端上下文，包括store, route, redirect, params, query, req, res等
    // store 状态树信息
    // route 一条目标路由信息
    // redirect 强制跳转
    // params query 校验参数合理性
    // 通过上述参数来写全局前置守卫
    console.log('middleware outside')   // 定义在外部
    // middleware还可以定义在layout布局中
}
```

- 组件独享守卫：`middleware`定义在组件中

- 插件全局守卫

在`plugins`文件夹中创建一个文件

```js
// plugins/router.js

export default ({app, redirect, params, query, store}) => {
  // 插件全局前置守卫
  // app 路由实例
  // redirect 跳转函数
  app.router.beforeEach((to, from, next) => {
    if (to.name === 'login') {
      next()
    } else {
      redirect({name: 'login'})
    }
  })
}
```

然后在`nuxt.config.js`中引入该插件

```js
// nuxt.config.js

// Plugins to run before rendering page (https://go.nuxtjs.dev/config-plugins)
  plugins: [
    '~/plugins/router'
  ],
```



**后置守卫**

- 插件全局后置守卫

```js
// plugins/router.js

export default ({app, redirect, params, query, store}) => {
  // 插件全局前置守卫
  // app 路由实例
  // redirect 跳转函数
  app.router.afterEach((to, from) => {
    // 插件全局后置守卫
  })
}
```

- 在组件中使用vue的beforeRouteLeave钩子，路由独享守卫

```js
// 某组件页面中

export default {
  beforeRouteLeave(to, from, next) {
    let b = window.confirm('是否要离开')
    next(b)
  }
}
```



## 3. 数据交互

> #### axios配置

Nuxt.js下`axios`安装

```
npm i @nuxtjs/axios @nuxtjs/proxy --save
```

然后在`nuxt.config.j`s的`modules`中添加进来，nuxt自身携带的模块都需要在此添加进来

```js
// nuxt.config.js

// Modules (https://go.nuxtjs.dev/config-modules)
  modules: [
    '@nuxtjs/axios'   // 添加axios
  ]
```

然后在pages页面中，用`asyncData`或`fetch`钩子获取

```vue
<template>
  <div class="container">
    <h3>首页</h3>
    <div>{{ title }}</div>
  </div>
</template>

<script>
export default {
  name: 'index',
  async asyncData({ $axios }) {
    // context中包括$axios，可以直接提取出$axios
    // 访问静态资源时，直接/就好
    let res = await $axios({ url: '/data/list.json' })
    return {
      // 将抓取到的数据合并到组件内部，此时上面的title则会展示成相应的数据
      title: res.data.title,
    }
  },
}
</script>

<style>
</style>
```

发送跨域请求时，需要在`nuxt.config.js`中配置

```js
axios: {
    proxy: true,  // 开启axios跨域
    // prefix: '/api'  // baseUrl，前缀
    // credentials: true // 证书
  },

proxy: {
  '/api': {
    target: 'http://xxx:8000',  // 代理转发的地址
    changeOrigin: true,
    pathRewrite: {
      '^/api': '' // 将转发地址中的api字符串置为空
    }
  }
}
```



> #### axios拦截器配置

在`plugins`文件夹中添加`axios.js`文件，然后在`nuxt.config.js`中配置

```js
// nuxt.config.js

// Plugins to run before rendering page (https://go.nuxtjs.dev/config-plugins)
  plugins: [
    {
      src: '~/plugins/axios',
      ssr: true   // 默认为true，即开启服务端渲染，若是不支持服务端渲染的插件则在此处写false
    }
  ],
```

在`axios.js`文件中

```js
// plugins/axios.js

export default function({$axios, redirect, route, store}) {
  // 超时设定
  $axios.defaults.timeout = 1000

  // 请求拦截
  $axios.onRequest(config => {
    config.headers.token = 'xxx'  // 添加token
    return config
  })
  
  // 响应拦截
  $axios.onResponse(res => {
    if (res.data.err === 2 && route.fullPath !== '/login') {
      redirect('xxx')
    }
  })
}
```



拦截器起到的作用

- 请求拦截：每次在发送请求时，携带token。
- 响应拦截：如果token失效了，拿不到数据时，会返回一些错误信息，可以根据错误信息做页面的跳转



## 4. Loading页面配置与定制

在`nuxt.config.js`中配置

```js
// nuxt.config.js

// 定义系统默认的loading效果，或指定loading组件
  // loading: {
  //   color: 'red',
  //   height: '3px'
  // },
  loading: '~/components/loading.vue',  // 直接指定loading组件
```



## 5. Vuex的使用

Nuxt.js内部集成了Vuex，有两种使用方式

- **模块方式：** `store`目录下的每个`.js`文件都会被转换成状态树[指定命名的子模块]
- **Classic方式：**`store/index.js`返回创建`Vuex.store`实例的方法（不建议使用）

在这两种方式中，`state`的值都始终是一个函数，里面的各类值都要对外暴露

这里主要是模块方式

```js
// store/index.js

// 这里是主模块

// state（函数）
export const state = () => ({
  token: '',
  userInfo: ''
})

// mutations（对象）
export const mutations = {
  setToken (state, payload) {
    state.token = payload
  },
  setUserInfo (state, payload) {
    state.userInfo = payload
  }
}

// actions（对象）
export const actions = {
  nuxtServerInit(store, context) {  // 可以拿到store的实例，context上下文
    // 在这个钩子里，初始化东西到store中
    console.log('nuxtServerinit')
  },
}

// getters
export const getters = {
  getToken(state) {
    // return state.token ? '有' : '无'   // 在getter中也可以进行计算属性的处理
    return state.token
  }
}
```

可以在`store`目录下创建其他`.js`的文件，如`home.js`

```js
// store/home.js

// home模块

export const state = () => ({
  err: 1,
  data: {}
})

export const mutations = {
  // 这里的state指的是当前home模块的state，state也可以写成当前模块的名字home
  setHome(state, payload) {
    state.err = payload.err
    state.data = payload.data
  }
}

export const actions = {
  updateHome({commit, state, dispatch}, payload) {
    commit('setHome', {err: 0, data: {title: "home"}})  // 异步commit提交到mutations中
  }
}
// 然后在pages页面中可以通过dispatch发出各种actions请求
// 给不同模块发出请求时，前面要带模块的前缀，如home/updateHome
```

`asyncData`和`fetch`虽然不能获取this实例，但context上下文信息中可直接获取`store`状态树来操作



## 6. 状态持久化

token信息必须要由客户端携带，可以用cookie或者localStorage和sessionStorage实现

> #### cookie实现

需要安装`cookie-universal-nuxt`来做cookie相关的状态持久化

```
npm i cookie-universal-nuxt --save
```

然后在`nuxt.config.js`中引入

```js
// nuxt.config.js

// Modules (https://go.nuxtjs.dev/config-modules)
  modules: [
    'cookie-universal-nuxt'
  ],
```

此时的登录逻辑

- 安装完`cookie-universal-nuxt`后，上下文context中会多出一个`$cookies`

- 登录时，同步`Vuex`和`cookie`。若是强制刷新，则从`nuxtServerInit`钩子中取出`cookie`，然后同步到`Vuex`。

- 登录之后，axios拦截器每次都会读取`Vuex`来取值，取token

> #### localStorage/sessionStorage实现

收到服务端返回的token后，将其存储在localStorage/sessionStorage中即可，其余步骤类似



## 7. UI组件库的使用

- 在`plugins`文件夹下创建相关的`.js`文件
- 在`nuxt.config.js`的css中引入相关样式，plugins中引入`.js`文件
- 在`nuxt.config.js`的`build`中

```js
// 此处为打包时的处理，如果不希望在打包时对ui库进行打包，则在此设置transpile属性，将其在打包的时候挑出来

// Nuxt.js 允许你在自动生成的 vendor.bundle.js 文件中添加一些模块，以减少应用 bundle 的体积。

build: {
    /*
    ** You can extend webpack config here
    */
    vendor: ['antd-ui', 'axios'],
    transpile: [/ant-design-vue/]
}
```



## 8. meta信息注入

全局的meta信息在`nuxt.config.js`中配置

为了避免子组件中的 meta 标签不能正确覆盖父组件中相同的标签而产生重复的现象，建议利用 `hid` 键为 meta 标签配一个唯一的标识编号。

```js
// process.env.npm_package_description可以拿到package.json中的description

head: {
    titleTemplate: 'xxx',
    meta: [
      { charset: 'utf-8' },
      { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      // hid在每个配置中都要保持一致，来使得所有页面都使用同一个hid，这样子组件中的meta标签不会覆盖父组件中的meta标签。所以需要保持hid一致，页面中就不会有重复的内容出现
      { hid: 'description', name: 'description', content: process.env.npm_package_description || '' }
    ],
    // link表示可以插入进来的内容，比如icon
    link: [
      { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }
    ],
    script: [
      { src: '外部资源' }
    ]
  },
```

各页面中的meta在页面中单独配置

```js
// 在export default中

head () {
    return {
      title: '登录',
      meta: [
        { hid: 'signin', name: 'signin', content: 'xxx' }
      ]
    }
  }
```

可以在最外层创建一个`app.html`文件，来使自己自定义的内容与页面html的已有内容结合

```html
<! DOCTYPE html>
<html {{HTML_ATTRS]}>
    <head {{HEAD_ATTRS)1
        {{HEAD}}
        <!--加入个性的内容-->
    </head>
	<body {{BODY_ATTRS}}>
        {{APP}}
	</body>
</html>
```



## 9. sass及less引入

安装`@nuxtjs/style-resources`

```
npm i @nuxtjs/style-resources --save
```

这样就可以直接全局使用less中的变量，不用每个文件都去引入了

接下来在`nuxt.config.js`中配置

```js
// nuxt.config.js

/*
** Nuxt.js modules
*/
modules: [
  '@nuxtjs/style-resources'
],
styleResources: {
  less: './assets/style/variable.less'	// 指定路径
}
```



## 10. 资源引入

- 资源一般放在`asset`s或`static`下面。放在`static`下一般是非优化的资源，会被直接搬迁到打完包后的生产环境下，`assets`下的资源可以被`webpack`做一些打包优化的操作。

- 引用`assets`下的资源时一般用相对路径，引用`static`下的资源时一般用绝对路径

- `~`代表从根目录下找，也是一种相对路径引用的方法

- 引入外部资源时，可以在app.html中添加，也可以在nuxt.config.js中添加

```html
// app.html

<script src="外部资源"></script>
```

```js
// nuxt.config.js

/*
** Headers of the page
*/
head: {
  link: [
      { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }
    	{ rel: 'stylesheet', href: 'http://xxxx' }
  ],
  script: [
    { src: '外部资源' }
  ]
},
```

- 也可以在每个页面中单独添加head引入



## 11. Nuxt中使用Koa

在Nuxt.js中使用Koa可以做跨域请求的代理，会比Nuxt.js自身的代理配置更为灵活

首先在根目录下创建`server.js`文件

```js
// server.js
// 每次运行都是从server.js启动

const fs = require('fs')
const Koa = require('koa')
const consola = require('consola')
const { Nuxt, Builder } = require('nuxt')
const bodyparser = require('koa-bodyparser')
const dotenv = require('dotenv')
const config = require('./nuxt.config.js')
const proxy = require('./middleware/proxy')	// 引入proxy.js，用作代理转发

const app = new Koa()

const isLocal = app.env === 'local'

function getEnvVariables () {
  try {
    let envSourcePath = '.env'
    const localEnvSourcePath = '.env.local'
    const localCurrentEnvSourcePath = `.env.${process.env.NODE_ENV}.local`
    const currentEnvSourcePath = `.env.${process.env.NODE_ENV}`

    if (fs.existsSync(localCurrentEnvSourcePath)) {
      envSourcePath = localCurrentEnvSourcePath
    } else if (fs.existsSync(currentEnvSourcePath)) {
      envSourcePath = currentEnvSourcePath
    } else if (fs.existsSync(localEnvSourcePath)) {
      envSourcePath = localEnvSourcePath
    }

    console.info(`Environment ${envSourcePath} file loaded.`)
    dotenv.config({ path: envSourcePath })
  } catch (err) {
    console.error('ERROR:', err)
  }
}

async function start () {
  getEnvVariables()
  // Instantiate nuxt.js
  const nuxt = new Nuxt({	// 实例化nuxt.js
    dev: isLocal,
    ...config
  })

  const {
    host = process.env.HOST || '127.0.0.1',
    port = process.env.PORT || 3000
  } = nuxt.options.server

  // Build in development
  if (isLocal) {
    const builder = new Builder(nuxt)
    await builder.build()
  } else {
    await nuxt.ready()
  }

  app.use(bodyparser())

  proxy(app)	// 这里做代理

  app.use(async (ctx, next) => {
    await next()
    ctx.status = 200
    ctx.respond = false // Bypass Koa's built-in response handling
    ctx.req.ctx = ctx // This might be useful later on, e.g. in nuxtServerInit or with nuxt-stash
    // nuxt.render(ctx.req, ctx.res)
    return new Promise((resolve, reject) => {
      ctx.res.on('close', resolve)
      ctx.res.on('finish', resolve)
      nuxt.render(ctx.req, ctx.res, promise => {
        // nuxt.render passes a rejected promise into callback on error.
        promise.then(resolve).catch(reject)
      })
    })
  })

  app.listen(port, host)
  consola.ready({		// 打印日志
    message: `Server listening on http://${host}:${port}`,
    badge: true
  })
}

start()	// 运行上述start函数

```

中间件中的proxy.js函数

```js
// middleware/proxy.js
// 实际用作转发的函数

const axios = require('axios')
const config = require('../nuxt.config.js')
const appConfig = require('./serverConfig').appConfig

const getHttpConfig = function (host) {
  const _config = appConfig(host)		// 得到appConfig中的config数据

  const apiUrl = _config.apiBaseAddress		// 这里获取到实际要发送的apiUrl
  const appKey = _config.appKey
  return {
    apiUrl,
    appKey
  }
}

// 做代理转发的函数
module.exports = (app) => {
  const prefix = '/virt-ai-api'
  config.prefix = prefix
  app.use(async (ctx, next) => {
    const { host, path, method, headers } = ctx
    const { authorization } = headers
    if (path.startsWith(prefix)) {
      ctx.status = 200
      ctx.body = {
        success: true
      }
      ctx.set('Content-Type', 'application/json')
      const { apiUrl, appKey: AppKey } = getHttpConfig(host)
      const headers = {
        AppKey,
        'Content-Type': 'application/json;charset=UTF-8'
      }
      if (authorization) {
        headers.authorization = authorization
      }
      try {
        process.env.NODE_TLS_REJECT_UNAUTHORIZED = '0'
        const res = await axios({
          method,
          url: apiUrl + path.replace(prefix, ''),		// 实际发送的请求
          headers,
          params: {
            ...ctx.request.query
          },
          data: {
            ...ctx.request.body
          }
        })
        if (res.status === 200) {
          // 返回数据
          ctx.body = res.data
          // 设置响应头
          ctx.set('Content-Type', 'application/json')
        } else {
          ctx.status = res.status
          ctx.body = {
            success: res
          }
          ctx.set('Content-Type', 'application/json')
        }
      } catch (e) {
        // console.log(e)
        ctx.body = {
          success: false
        }
        ctx.set('Content-Type', 'application/json')
      }
    } else {
      await next()
    }
  })
}

```

中间件中的serverConfig.js

```js
// middleware/serverConfig.js

const AppEnv = require('./serverAppEnv')

module.exports.appConfig = (host) => {
  const config = {
    mobileDomain: process.env.VUE_APP_MOBILE_DOMAIN,
    apiBaseAddress: process.env.VUE_APP_API_BASE_ADDRESS,
    appKey: process.env.VUE_APP_APP_KEY
  }

  const slotConfig = {
    mobileDomain: process.env.VUE_APP_MOBILE_DOMAIN_SLOT,
    apiBaseAddress: process.env.VUE_APP_API_BASE_ADDRESS_SLOT
  }

  return new AppEnv(config, slotConfig, host).config	// 返回上述config
}

```

中间件中的serverAppEnv.js

```js
// middleware/serverAppEnv.js

class AppEnv {
  constructor (config, slotConfig, hostname) {
    this.config = config		// 闭包中的config
    this.hostname = hostname	// 闭包中的hostname
    if (this.isSlotEnv()) {
      console.log('slot environment')
      this.setSlotEnvConfig(slotConfig)
    }
  }

  isSlotEnv () {
    const subDomain = this.getSubDomain()
    const slotRegex = /-slot\d+$/
    return slotRegex.test(subDomain)
  }

  getSubDomain () {
    return this.hostname.split('.').slice(0, -2).join('.')
  }

  setSlotEnvConfig (slotConfig) {
    for (const property in slotConfig) {
      if (slotConfig[property]) {
        this.config[property] = slotConfig[property]
      }
    }
  }
}

module.exports = AppEnv

```



## 12. pages中的属性

> #### layout

`layouts` *根*目录下的所有文件都属于个性化布局文件，可以在页面组件中利用 `layout` 属性来引用。

- **类型：** `String` 或 `Function` (默认值： `'default'`)

使用 `layout` 属性来为页面指定使用哪一个布局文件：

```js
export default {
  layout: 'blog',
  // 或
  layout(context) {
    return 'blog'
  }
}
```

在上面的例子中， Nuxt.js 会使用 `layouts/blog.vue` 作为当前页面组件的布局文件。



> #### loading

- 在页面切换的时候，Nuxt.js 使用内置的加载组件显示加载进度条。你可以定制它的样式，禁用或者创建自己的加载组件。

在你的组件中你可以使用`this.$nuxt.$loading.start()`来启动加载条。使用`this.$nuxt.$loading.finish()`来使加载条结束。

```javascript
export default {
  mounted() {
    this.$nextTick(() => {
      this.$nuxt.$loading.start()

      setTimeout(() => this.$nuxt.$loading.finish(), 500)
    })
  }
}
```

如果要在`mounted`方法中启动它，请确保使用`this.$nextTick`来调用它，因为`$loading`可能无法立即使用。

- 禁用加载进度条

页面切换的时候如果不想显示加载进度条，可以在 `nuxt.config.js` 里面增加 `loading: false` 的配置：

```js
module.exports = {
  loading: false
}
```



> scrollToTop

scrollToTop 属性用于控制页面渲染前是否滚动至页面顶部。

默认情况下，从当前页面切换至目标页面时，Nuxt.js 会让目标页面滚动至顶部。但是在嵌套子路由的场景下，Nuxt.js 会保持当前页面的滚动位置，除非在子路由的页面组件中将 `scrollToTop` 设置为 `true`。

```html
<template>
  <h1>子页面组件</h1>
</template>

<script>
  export default {
    scrollToTop: true
  }
</script>
```



## 13. 补充

在线上环境可通过 `this.__Nuxt__` 来查看当前实例中的各种值

