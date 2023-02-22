常见的攻击的类型以及防御的方法。



## XSS

XSS (Cross-Site Scripting)，跨站脚本攻击，因为缩写和 CSS 重叠，所以只能叫 XSS。跨站脚本攻击是指通过存在安全漏洞的 Web 网站注册用户的浏览器内运行非法的 HTML 标签或 JavaScript 进行的一种攻击。

跨站脚本攻击有可能造成以下影响：

- 利用虚假输入表单骗取用户个人信息。
- 利用脚本窃取用户的 Cookie 值，被害者在不知情的情况下，帮助攻击者发送恶意请求。
- 显示伪造的文章或图片。

**XSS 的原理是恶意攻击者往 Web 页面里插入恶意可执行网页脚本代码，当用户浏览该页面时，嵌入其中 Web 里面的脚本代码会被执行，从而可以达到攻击者盗取用户信息或其他侵犯用户安全隐私的目的**。

XSS 的攻击方式千变万化，但还是可以大致细分为几种类型。

### 1. 非持久型 XSS（反射型 XSS）

非持久型 XSS 漏洞，攻击相对于受害者而言是一次性的。

一般是通过给别人发送**带有恶意脚本代码参数的 URL**。当用户点击一个恶意链接，或者提交一个表单，或者进入一个恶意网站时，注入脚本进入被攻击者的网站。Web 服务器将注入的脚本，比如一个错误信息，搜索结果等返回到用户的浏览器上。由于浏览器认为这个响应来自"可信任"的服务器，所以会执行这段脚本。

例如下面的一段代码，我们通过页面 url 的一个参数来控制页面的展示内容：

```js
const Koa = require("koa");
const app = new Koa();

app.use(async ctx => {
  // ctx.body 即服务端响应的数据
  ctx.body = ctx.query.userName;
})

app.listen(3000, () => {
  console.log('启动成功');
});
```

此时访问 `http://127.0.0.1:3000?userName=xiaoming` 可以看到页面上展示了`xiaoming`，但此时我们访问 `http://127.0.0.1:3000?userName=<script>alert("反射型 XSS 攻击")</script>`, 可以看到页面弹出 alert。

![example1]()

通过这个操作，我们会发现用户将一段含有恶意代码的请求提交给服务器，服务器在接收到请求时，又将恶意代码反射给浏览器端，顾名思义，这就是反射型 XSS 攻击。

> 此时，Web 服务器不会存储反射型 XSS 攻击的恶意脚本，这是和存储型 XSS 攻击不同的地方。

在实际的开发过程中，我们会碰到这样的场景，在页面 A 中点击某个操作，这个按钮操作是需要登录权限的，所以需要跳转到登录页面，登录完成之后再跳转回 A 页面，我们是这么处理的，跳转登录页面的时候，会加一个参数 returnUrl，表示登录完成之后需要跳转到哪个页面，即这个地址是这样的 `http://xxx.com/login?returnUrl=http://xxx.com/A`，

假如这个时候把 returnUrl 改成一个 script 脚本，而你在登录完成之后，如果没有对 returnUrl进行合法性判断，而直接通过 `window.location.href=returnUrl`，那么这个时候这个恶意脚本就会执行。



非持久型 XSS 漏洞攻击有以下几点特征：

- 即时性，不经过服务器存储，直接通过 HTTP 的 GET 和 POST 请求就能完成一次攻击，通过恶意脚本拿到用户隐私数据。

  > POST 请求需要构造表单提交页面，并引导用户点击，触发条件比较苛刻，所以比较少见

- 攻击者需要诱骗点击，必须要通过用户点击链接才能发起

- 反馈率低，所以较难发现和响应修复

- 盗取用户敏感保密信息



为了防止出现非持久型 XSS 漏洞，需要确保这么几件事情：

- Web 页面渲染的所有内容或者渲染的数据**都必须来自于服务端**。
- 尽量不要从 `URL`，`document.referrer`，`document.forms` 等这种 DOM API 中获取数据直接渲染。
- 尽量不要使用 `eval`, `new Function()`，`document.write()`，`document.writeln()`，`window.setInterval()`，`window.setTimeout()`，`innerHTML`，`document.createElement()` 等可执行字符串的方法。
- 如果做不到以上几点，也必须对涉及 DOM 渲染的方法传入的字符串参数做 [`encodeURI`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/encodeURI) 或 [`encodeURIComponent`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent) 转义（或者对任何的字段都需要做转义编码）。



### 2. 持久型 XSS（存储型 XSS）

持久型 XSS 漏洞，一般存在于 Form 表单提交等交互功能，如文章留言，提交文本信息等，黑客利用 XSS 漏洞，将恶意内容经正常功能提交进入数据库持久保存，当前端页面获得后端从数据库中读出的注入代码时，恰好将其渲染执行。

主要注入页面方式和非持久型 XSS 漏洞类似，只不过持久型的不是来源于 URL，referer，forms 等，而是来源于**后端从数据库中读出来的数据** 。持久型 XSS 攻击不需要诱骗点击，黑客只需要在提交表单的地方完成注入即可，但是这种 XSS 攻击的成本相对还是很高。





