![截屏2021-02-21 下午8.24.39](/Users/croon/Library/Application Support/typora-user-images/截屏2021-02-21 下午8.24.39.png)



```js
const Koa = require('koa')
const app = new Koa

app.use(async ctx => {
  ctx.body = 'hello world'
})

app.listen(3000)
```



## koa-router

`npm install koa-router`



路由加前缀

```js
router.prefix('/api')
```



## koa 路由进阶配置

- 路由按照功能模块区分

- 路由压缩：koa-comboine-routers，只使用一次 app.use() 就把路由全部给引入进来
- 静态资源：koa-static，指定一个静态资源的目录，就可以对其进行访问了



## 中间件插件

koa 开发 Restful 接口

koa-body, @koa/cors, koa-json

```js
const koaBody = require('koa-body')
const cors = require('@koa/cors')

// 处理 request 请求里过来的数据
app.use(koaBody())
// 处理跨域的请求
app.use(cors())
```



koa-helmet

使我们的请求能够获得一个安全的头部

```js
const helmet = require('koa-helmet')

app.use(helmet())
```



koa-static

```js
const static = require('koa-static')
const path = require('path')

app.use(static(path.join(__dirname, '../public'))) // 即可直接通过资源文件名加载 public 下面的静态资源了
```



## koa 配置开发热加载 & ES6 语法支持

搭建一个开发和生产中用到的 node 开发热加载及 es6 语法支持



nodemon

js 发生变化的时候可以重启服务

`npm install nodemon -D`

在 `package.json` 中，运行 `npm run serve` 即可

```js
"scripts": {
  "serve": "nodemon src/app.js"
}
```

与 webpack 的 watch 参数很类似



`npm install webpack webpack-cli -D`

`npm install clean-webpack-plugin webpack-node-externals @babel/core @babel/node @babel/preset-env babel-loader cross-env -D`

webpack-node-externals 可以排除不会使用到的 node 模块，即对 node_modules 下面的文件做排除处理

@babel/preset-env 可以对新特性进行支持

babel-loader 是在 webpack 中使用到的 loader

cross-env 设置环境变量



使用了 babel 之后，在文件里写了 es6 的语法之后，就不要用 node app.js 来启动了，要用 babel-node app.js

nodemon 则要使用 nodemon --exec babel-node app.js 来执行



## webpack 调试

终端输入 `npx node --inspect-brk ./node_modules/.bin/webpack --inline --progress` 命令来进行调试

接下来进入相应的网站查看chrome://inspect/#devices

可以 console.log(webpackConfig) 将配置打印出来看有没有错

然后在其代码上部加上 debugger

