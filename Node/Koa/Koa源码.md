## 原理

Koa.js 是基于中间件模式的 HTTP 服务框架，底层原理是离不开 Node.js 的 `http` 原生模块的



示例：

```js
const http = require('http')
const PORT = 3001

// 控制器
const controller = {
  index(req, res) {
    res.end('This is index page')
  },
  home(req, res) {
    res.end('This is home page')
  },
  _404(req, res) {
    res.end('404 Not Found')
  }
}

// 路由器
const router = (req, res) => {
  if( req.url === '/' ) {
    controller.index(req, res)
  } else if( req.url.startsWith('/home') ) {
    controller.home(req, res)
  } else {
    controller._404(req, res)
  }
}

// 服务
const server = http.createServer(router)
server.listen(PORT, function() {
  console.log(`the server is started at port ${PORT}`)
})
```



## Koa 文件目录

koa2 目录结构只有四个文件，搭起了整个 server 的框架。

koa 只是负责**开头（接受请求）**和**结尾（响应请求）**，对请求的处理都是由中间件来实现。

```
├── lib
│   ├── application.js // 负责串联起 context request response 和中间件
│   ├── context.js // 一次请求的上下文
│   ├── request.js // koa 中的请求
│   └── response.js  // koa 中的响应
└── package.json
```

application.js：负责串联起 context request response 和中间件

context.js：一次请求的上下文

request.js：koa 中的请求

response.js：koa 中的响应



## new Koa()

Koa 实际上是一个 class，继承自 node 的 [events 事件触发器](http://nodejs.cn/api/events.html#events_events)。

new Koa() 时，大致实际执行了下列代码（简化版）：

```js
// lib/application.js
class Application extends Emitter {
  constructor (options) {
    super()
    options = options || {}
    this.proxy = options.proxy || false
    this.subdomainOffset = options.subdomainOffset || 2
    this.proxyIpHeader = options.proxyIpHeader || 'X-Forwarded-For'
    this.maxIpsCount = options.maxIpsCount || 0
    this.env = options.env || process.env.NODE_ENV || 'development'
    if (options.keys) this.keys = options.keys
    this.middleware = []
    this.context = Object.create(context)
    this.request = Object.create(request)
    this.response = Object.create(response)
    if (util.inspect.custom) {
      this[util.inspect.custom] = this.inspect
    }
  }
  
  listen () {
    const fnMiddleware = compose(this.middleware)
    const ctx = this.context
    const handleResponse = () => respond(ctx)
    const onerror = function() {
      console.log('onerror')
    }
    fnMiddleware(ctx).then(handleResponse).catch(onerror)
  }

  use (fn) {
    ...
    this.middleware.push(fn)
    return this
  }
}
```

this.request 和 this.response 就是继承自文件 `lib/request.js` 和 `lib/response.js` 中的 request 和 response。 request 和 response 做的事情并不复杂，就是将 Node 原生的 http 的 req 和 res 再次做了封装，便于读取和设置，每个属性和方法的封装都不复杂，具体的属性和方法参考 koa 的官方文档。

至于 context，context 是一个 **传递的纽带**，每个中间件传递的就是 context，它承载了这次访问的所有内容，直到其被最终 response 掉。this.context 继承自文件 `lib/context.js`，就是一个普通的对象。context 还额外提供了 error 和 cookie 的处理，因为 app 是继承自 Emitter 只有它能订阅 `onerror` 事件，所以传递的 context 要包裹一个 app 来在中间件中传递（在下面的 createContext 函数中会出现）。



除了 respond 函数，我们可以看到，重点是在 listen 函数中的 `compose 函数`，详细看一下这个函数。



## koa-compose

通过`app.use()` 添加了若干函数后，接下来要把它们串起来执行。这时就使用到了 `compose 函数`。

以下是 compose 函数的源码：

```js
function compose (middleware) {
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  /**
   * @param {Object} context
   * @return {Promise}
   * @api public
   */

  return function (context, next) {
    // last called middleware #
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```

传入一个数组，返回一个函数。然后发现 compose 其实就是类似这样的结构：

```js
async function fn1 (ctx, next) {
  console.log('action 001')
  ctx.data.push(1)
  await next()
  console.log('action 006')
  ctx.data.push(6)
}

async function fn2 (ctx, next) {
  console.log('action 002')
  ctx.data.push(2)
  await next()
  console.log('action 005')
  ctx.data.push(5)
}

async function fn3 (ctx, next) {
  console.log('action 003')
  ctx.data.push(3)
  await next()
  console.log('action 004')
  ctx.data.push(4)
}

const fnMiddleware = function (context) {
    return Promise.resolve(
      fn1 (context, function next (){
        return Promise.resolve(
          fn2 (context, function next (){
              return Promise.resolve(
                  fn3 (context, function next (){
                    return Promise.resolve()
                  })
              )
          })
        )
    })
  )
}

fnMiddleware(ctx).then(handleResponse).catch(onerror)

```

在 fn1 中执行 `await next()` 时，即是执行 `return Promise.resolve(fn2(context, function next(...)))`，然后再到 fn2, fn3 ...

最后一个中间件中有调用`next`函数，则返回`Promise.resolve`。如果没有，则不执行`next`函数。 这样就把所有中间件串联起来了。这也就是我们常说的洋葱模型。

> 这种把函数存储下来的方式，在很多源码中都有看到。比如`lodash`源码的惰性求值，`vuex`也是把`action`等函数存储下，最后才去调用。



## 错误处理

koa 中文文档中的 [错误处理](https://github.com/demopark/koa-docs-Zh-CN/blob/master/error-handling.md) 描述如下

> 错误事件侦听器可以用 `app.on('error')` 指定。如果未指定错误侦听器，则使用默认错误侦听器。错误侦听器接收所有中间件链返回的错误，如果一个错误被捕获并且不再抛出，它将不会被传递给错误侦听器。如果没有指定错误事件侦听器，那么将使用 `app.onerror`，除非 `error.expose` 为 true 或 `app.silent` 为 true 或 `error.status` 为 404，否则只简单记录错误。

我们可以自己在中间件中 try-catch 错误，若不 try-catch，错误则会分发到错误事件侦听器，而这个错误事件侦听器，我们可以自己用 `app.on('error')` 来定义，若未自己定义，则使用系统默认的错误事件侦听器，也就是下面讲到的 `app.onerror()` 函数。



### 错误事件侦听器的定义

执行 `app.listen()` 的时候：

```js
// lib/application.js
listen (...args) {
  debug('listen')
  const server = http.createServer(this.callback())
  return server.listen(...args)
}
```

callback 函数如下所示：

```js
// lib/application.js
callback () {
  const fn = compose(this.middleware)
  // listenerCount 返回正在监听的名为 error 的事件的监听器的数量，属于 node 自带的
  if (!this.listenerCount('error')) this.on('error', this.onerror)

  const handleRequest = (req, res) => {
    const ctx = this.createContext(req, res)
    return this.handleRequest(ctx, fn)
  }

  return handleRequest
}
```

`if (!this.listenerCount('error')) this.on('error', this.onerror)` 这句表明，如果没有自定义的错误事件侦听器，则使用 onerror 函数来侦听错误。

onerror 函数如下所示：

```js
// lib/application.js
onerror (err) {
  if (!(err instanceof Error)) throw new TypeError(util.format('non-error thrown: %j', err))

  if (404 == err.status || err.expose) return
  if (this.silent) return

  const msg = err.stack || err.toString()
  console.error()
  console.error(msg.replace(/^/gm, '  '))
  console.error()
}
```

onerror 函数是默认的错误事件侦听器（最外层的错误事件侦听器，全局错误事件侦听器），用来输出错误日志。官方文档中写道：

> 默认情况下，将错误输出到 stderr，除非 `app.silent` 为 `true`。 当 `err.status` 是 `404` 或 `err.expose` 是 `true` 时默认错误处理程序也不会输出错误。 要执行自定义错误处理逻辑，如集中式日志记录，您可以添加一个 “error” 事件侦听器。



### 每次请求时是如何捕获错误的

在每次请求时，会执行 `this.createContext(req, res)` 函数来初始化新的 ctx

```js
// lib/application.js
createContext (req, res) {
  const context = Object.create(this.context)
  const request = context.request = Object.create(this.request)
  const response = context.response = Object.create(this.response)
  context.app = request.app = response.app = this
  context.req = request.req = response.req = req
  context.res = request.res = response.res = res
  request.ctx = response.ctx = context
  request.response = response
  response.request = request
  context.originalUrl = request.originalUrl = req.url
  context.state = {}
  return context
}
```

为什么在构造函数中，this.context 是继承自 `context.js` 文件，这里在每次请求时又继承自 `this.context` 呢？
这样的好处是你可以为你的 app 设定一个类似模板的 context，这样一来每个请求的 context 在继承时就会有一些预设的方法或属性，this.request 和 this.response 也同理。



紧接着，会执行 `this.handleRequest()` 函数，传入 ctx 和中间件函数 fnMiddleware。

```js
// lib/application.js
handleRequest(ctx, fnMiddleware) {
  const res = ctx.res
  // 默认状态码设置为 404
  res.statusCode = 404
  const onerror = err => ctx.onerror(err)
  const handleResponse = () => respond(ctx)
  onFinished(res, onerror)
  return fnMiddleware(ctx).then(handleResponse).catch(onerror)
}
```

可以看到，中间件中抛出的错误会 catch 到 `ctx.onerror()` 函数中，然后 `ctx.onerror()` 函数中则会通过 `this.app.emit('error', err, this)` 来分发 error 事件，最后在最外层的 `app.on('error', app.onerror)` 被捕获。



## 整个流程

koa 中一次 http 请求的流程示意图如下：

![koa流程图](/Users/croon/Library/Application Support/typora-user-images/koa流程图.png)

运行中有一个很重要和基本的概念就是：**HTTP 请求是幂等的。**

一次和多次请求某一个资源应该具有同样的副作用，也就是说每个 HTTP 请求都会有一套全新的 context，request，response。



可以看到，若未抛出错误，则最终进入到 respond 函数中：

```js
// lib/application.js
function respond (ctx) {
  // allow bypassing koa
  if (false === ctx.respond) return

  if (!ctx.writable) return

  const res = ctx.res
  let body = ctx.body
  const code = ctx.status

  // ignore body
  if (statuses.empty[code]) {
    // strip headers
    ctx.body = null
    return res.end()
  }

  if ('HEAD' === ctx.method) {
    if (!res.headersSent && !ctx.response.has('Content-Length')) {
      const { length } = ctx.response
      if (Number.isInteger(length)) ctx.length = length
    }
    return res.end()
  }

  // status body
  if (null == body) {
    if (ctx.req.httpVersionMajor >= 2) {
      body = String(code)
    } else {
      body = ctx.message || String(code)
    }
    if (!res.headersSent) {
      ctx.type = 'text'
      ctx.length = Buffer.byteLength(body)
    }
    return res.end(body)
  }

  // responses
  if (Buffer.isBuffer(body)) return res.end(body)
  if ('string' == typeof body) return res.end(body)
  if (body instanceof Stream) return body.pipe(res)

  // body: json
  body = JSON.stringify(body)
  if (!res.headersSent) {
    ctx.length = Buffer.byteLength(body)
  }
  res.end(body)
}
```



## Koa2 与 Koa1 的对比

Koa 中文文档中的 [从 Koa v1.x 迁移到 v2.x](https://github.com/demopark/koa-docs-Zh-CN/blob/master/migration.md) 描述了 Koa2 与 Koa1 的区别。

在 Koa1 中，使用 yield next 进入下一个中间件；在 koa2 中，中间件使用 async 函数，直接使用 await next() 进入下一个中间件。

在 Koa1 中，主要是使用 generator 函数实现中间件，然后在 callback() 函数中用 `co.wrap` 来将其转换。

Koa2 中，则是使用 `koa-compose` 将中间件使用`Promise`串联起来。Koa2 中也兼容 generator 函数实现中间件的方式。从源码中可以看到 `app.use`时有一层判断，判断是否是`generator`函数，如果是则用`koa-convert`暴露的方法`convert`来转换重新赋值，再存入`middleware`，后续再使用。

```js
// lib/applicant.js
class Koa extends Emitter {
  use (fn) {
    if (typeof fn !== 'function') throw new TypeError('middleware must be a function!')
    if (isGeneratorFunction(fn)) {
      deprecate('Support for generators will be removed in v3. ' +
                'See the documentation for examples of how to convert old middleware ' +
                'https://github.com/koajs/koa/blob/master/docs/migration.md')
      fn = convert(fn)
    }
    debug('use %s', fn._name || fn.name || '-')
    this.middleware.push(fn)
    return this
  }
}
```

`koa-convert` 源码挺多，核心代码其实是这样：

```js
function convert(){
 return function (ctx, next) {
    return co.call(ctx, mw.call(ctx, createGenerator(next)))
  }
  function * createGenerator (next) {
    return yield next()
  }
}
```

最后还是通过`co`来转换的。



## Koa 与 Express 的对比

功能实用上具体参考 [Koa与Express](https://github.com/demopark/koa-docs-Zh-CN/blob/master/koa-vs-express.md)



## 参考文章

[Koa2 中文文档](https://github.com/demopark/koa-docs-Zh-CN)

[Koa2 源码及流程分析](https://github.com/fi3ework/blog/issues/40)

[学习 koa 源码的整体架构，浅析koa洋葱模型原理和co原理](https://github.com/lxchuan12/koa-analysis)

[Koa2进阶学习笔记](https://github.com/chenshenhai/koa2-note)

