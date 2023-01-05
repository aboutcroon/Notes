# 同步与异步请求

前端请求数据时，发送 ajax 请求是异步的

而渲染整个 html 页面时是同步的请求

在 axios 的拦截器中将请求的 headers 中添加一项：`headers['X-Requested-With'] = 'XMLHttpRequest'`，来表明这个请求是异步的请求，即所有通过 axios 发送出去的请求都是异步的，而那些没加上这个字段的页面的渲染请求则是同步的

那么服务端可以判断：

```js
if (status === 401 && ctx.get('X-Requested-With') !== 'XMLHttpRequest') {
  ctx.redirect('/')
}
```

这样子，当登录过期时，则直接`不渲染页面`，直接跳到首页，首页再继续跳转到登录页重新登录



# status 与 code

`status` 是 http 的状态码

`code` 是我们自己在请求的返回信息中添加的字段



# Axios

post 和 put 方法假如要传入 query 的话，要这样传（名字为params），data是body的参数，params 才是 query：

axios.put(‘https://example.com', data, { params: form.getHeaders() })

参考：https://github.com/axios/axios#form-data





token 为什么安全

为什么 axios 的 header 中没有加 token

```js
if (status === 401 && ctx.get('X-Requested-With') !== 'XMLHttpRequest') {
  ctx.redirect('/')
} else {
  ctx.status = status
  ctx.body = msg
}
```

这一句是做什么



set-cookie

https://segmentfault.com/a/1190000012371083

