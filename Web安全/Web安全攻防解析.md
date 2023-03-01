以下是常见的 Web 攻击类型以及防御的方法

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

![xss_1](https://raw.githubusercontent.com/edwineo/Notes/main/Web%E5%AE%89%E5%85%A8/assets/websesure_xss1.png)

攻击步骤为：

1. 攻击者构造出特殊的 URL，其中包含恶意代码。
2. 用户打开带有恶意代码的 URL 时，网站服务端将恶意代码从 URL 中取出，拼接在 HTML 中返回给浏览器。
3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。



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

![example1](https://raw.githubusercontent.com/edwineo/Notes/main/Web%E5%AE%89%E5%85%A8/assets/example1.gif)

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

![xss_2](https://raw.githubusercontent.com/edwineo/Notes/main/Web%E5%AE%89%E5%85%A8/assets/websesure_xss2.png)

![xss_4](https://raw.githubusercontent.com/edwineo/Notes/main/Web%E5%AE%89%E5%85%A8/assets/websecure_xss4.png)

举个例子，对于评论功能来说，就得防范持久型 XSS 攻击，因为我可以在评论中输入一些恶意内容：

![xss_3](https://raw.githubusercontent.com/edwineo/Notes/main/Web%E5%AE%89%E5%85%A8/assets/websesure_xss3.png)

持久型 XSS 漏洞的主要注入页面方式和非持久型 XSS 漏洞类似，只不过持久型的恶意攻击内容不是来源于 URL，referer，forms 等，而是来源于**后端从数据库中读出来的数据** 。持久型 XSS 攻击不需要诱骗点击，黑客只需要在提交表单的地方完成注入即可。

但是这种 XSS 攻击的成本相对还是很高的，攻击成功需要同时满足以下几个条件：

- POST 请求提交表单后端没做转义直接入库。
- 后端从数据库中取出数据没做转义直接输出给前端。
- 前端拿到后端数据没做转义直接渲染成 DOM。

持久型 XSS 有以下几个特点：

- 持久性，植入在数据库中
- 盗取用户敏感私密信息
- 危害面广



### 3. DOM 型 XSS

通过恶意脚本修改页面的 DOM 节点，是发生在前端的攻击

基于 DOM 攻击大致需要经历以下几个步骤

1. 攻击者构造出特殊的 URL，其中包含恶意代码
2. 用户打开带有恶意代码的 URL
3. 用户浏览器接受到响应后执行解析，前端 JavaScript 取出 URL 中的恶意代码并执行
4. 恶意代码窃取用户数据并发送到攻击者的网站，冒充用户行为，调用目标网站接口执行攻击者指定的操作。

DOM 型 XSS 跟前两种 XSS 的区别：DOM 型 XSS 攻击中，取出和执行恶意代码由浏览器端完成，是属于前端 JavaScript 自身的安全漏洞，而其他两种 XSS 都属于服务端的安全漏洞。



### 4. 如何防御 XSS

对于 XSS 攻击来说，通常有三种方式可以用来防御。

#### CSP

CSP 本质上就是建立白名单，开发者明确告诉浏览器哪些外部资源可以加载和执行。我们只需要配置规则，如何拦截是由浏览器自己实现的。我们可以通过这种方式来尽量减少 XSS 攻击。

通常可以通过两种方式来开启 CSP：

- 设置 meta 标签的方式

  ```html
  <meta http-equiv="Content-Security-Policy" content="default-src 'self'; img-src https://*; child-src 'none';">
  ```

- 设置 HTTP Header 中的 `Content-Security-Policy`

  只允许加载本站资源:`Content-Security-Policy: default-src 'self'`

  只允许加载 HTTPS 协议图片:`Content-Security-Policy: img-src https://*`

  允许加载任何来源框架:`Content-Security-Policy: child-src 'none'`

  如需了解更多属性，请查看 [Content-Security-Policy文档](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FHTTP%2FHeaders%2FContent-Security-Policy)

对于这种方式来说，只要开发者配置了正确的规则，那么即使网站存在漏洞，攻击者也不能执行它的攻击代码，并且 CSP 的兼容性也不错。

#### 转义字符

用户的输入永远不可信任的，最普遍的做法就是转义输入输出的内容，对于引号、尖括号、斜杠进行转义

```javascript
function escape(str) {
  str = str.replace(/&/g, '&amp;')
  str = str.replace(/</g, '&lt;')
  str = str.replace(/>/g, '&gt;')
  str = str.replace(/"/g, '&quto;')
  str = str.replace(/'/g, '&#39;')
  str = str.replace(/`/g, '&#96;')
  str = str.replace(/\//g, '&#x2F;')
  return str
}
```

#### 白名单

但是对于显示富文本来说，显然不能通过上面的办法来转义所有字符，因为这样会把需要的格式也过滤掉。

对于这种情况，通常采用白名单过滤的办法，当然也可以通过黑名单过滤，但是考虑到需要过滤的标签和标签属性实在太多，更加推荐使用白名单的方式：

```js
// 只允许a标签，该标签只允许href, title, target这三个属性
var options = {
  whiteList: {
    a: ["href", "title", "target"],
  },
};
// 使用以上配置后，下面的HTML
// <a href="#" onclick="hello()"><i>大家好</i></a>
// 将被过滤为
// <a href="#">大家好</a>
```

以上示例使用了 [js-xss](https://github.com/leizongmin/js-xss) 来实现。

#### HttpOnly Cookie

这是预防 XSS 攻击窃取用户 cookie 最有效的防御手段。

Web应用程序在设置 cookie 时，将其属性设为 HttpOnly，则能禁止通过 document.cookie 的方式获取 cookie，这样就可以避免该网页的 cookie 被客户端恶意 JavaScript 窃取，保护用户 cookie 信息。



## CSRF

CSRF(Cross Site Request Forgery)，即跨站请求伪造，是一种常见的 Web 攻击，它利用用户已登录的身份，在用户毫不知情的情况下，以用户的名义完成非法操作。

### 1. 原理

下面是 CSRF 攻击的原理：

![csrf](https://raw.githubusercontent.com/edwineo/Notes/main/Web%E5%AE%89%E5%85%A8/assets/csrf.png)

完成 CSRF 攻击必须要有三个条件：

- 用户已经登录了站点 A，并在本地记录了 cookie
- 在用户没有登出站点 A 的情况下（也就是 cookie 生效的情况下），访问了恶意攻击者提供的引诱危险站点 B (B 站点要求访问站点A)。
- 站点 A 没有做任何 CSRF 防御



我们来看一个例子：

当我们登入转账页面后，突然眼前一亮**惊现"XXX隐私照片，不看后悔一辈子"的链接**，耐不住内心躁动，立马点击了该危险的网站，但当这页面一加载，便会执行`submitForm`这个方法来提交转账请求，从而将 10 块钱转给黑客。



### 2. 如何防御

防范 CSRF 攻击可以遵循以下几种规则：

- Get 请求不对数据进行修改
- 不让第三方网站访问到用户 Cookie
- 阻止第三方网站请求接口
- 请求时附带验证信息，比如验证码或者 Token

#### SameSite

可以对 Cookie 设置 SameSite 属性。该属性表示 Cookie 不随着跨域请求发送，可以很大程度减少 CSRF 的攻击，但是该属性目前并不是所有浏览器都兼容。

#### Referer Check

HTTP Referer 是 header 的一部分，当浏览器向 web 服务器发送请求时，一般会带上 referer信息告诉服务器是从哪个页面链接过来的，服务器籍此可以获得一些信息用于处理。可以通过检查请求的来源来防御 CSRF 攻击。正常请求的 referer 具有一定规律，如在提交表单的 referer 必定是在该页面发起的请求。所以可以**通过检查 http 包 head 中的 referer 的值是不是这个页面，来判断是不是CSRF攻击**。

但在某些情况下如从 https 跳转到 http，浏览器处于安全考虑，不会发送 referer，服务器就无法进行 check 了。若与该网站同域的其他网站有 XSS 漏洞，那么攻击者可以在其他网站注入恶意脚本，受害者进入了此类同域的网址，也会遭受攻击。出于以上原因，无法完全依赖Referer Check 作为防御 CSRF 的主要手段。但是可以通过 Referer Check 来监控 CSRF 攻击的发生。

#### Anti CSRF Token

目前比较完善的解决方案是加入 Anti-CSRF-Token。即发送请求时在 HTTP 请求中以参数的形式加入一个随机产生的 token，并在服务器建立一个拦截器来验证这个 token。服务器读取浏览器当前域 cookie 中这个 token 值，会进行校验该请求当中的 token 和 cookie 当中的 token 值是否都存在且相等，才认为这是合法的请求。否则认为这次请求是违法的，拒绝该次服务。

**这种方法相比 Referer Check 要安全很多**，token 可以在用户登陆后产生并放于 session 或 cookie 中，然后在每次请求时服务器把 token 从 session 或 cookie 中拿出，与本次请求中的token 进行比对。由于 token 的存在，攻击者无法再构造出一个完整的 URL 实施 CSRF 攻击。

但在多个页面共存时，当某个页面消耗掉 token 后，其他页面的表单保存的还是被消耗掉的那个 token，其他页面的表单提交时会出现token错误。

#### 验证码

应用程序和用户进行交互过程中，特别是账户交易这种核心步骤，强制用户输入验证码，才能完成最终请求。在通常情况下，验证码够很好地遏制 CSRF 攻击。**但增加验证码降低了用户的体验，网站不能给所有的操作都加上验证码**。所以只能将验证码作为一种辅助手段，在关键业务点设置验证码。



## 点击劫持

点击劫持是一种视觉欺骗的攻击手段。攻击者将需要攻击的网站通过 iframe 嵌套的方式嵌入自己的网页中，并将 iframe 设置为透明，在页面中透出一个按钮诱导用户点击。

具有以下特点：

- 隐蔽性较高，骗取用户操作
- "UI-覆盖攻击"
- 利用 iframe 或者其它标签的属性

### 1. 原理

用户在登陆 A 网站的系统后，被攻击者诱惑打开第三方网站，而第三方网站通过 iframe 引入了 A 网站的页面内容，用户在第三方网站中点击某个按钮（被装饰的按钮），实际上是点击了 A 网站的按钮。 

接下来我们举个例子：我在优酷发布了很多视频，想让更多的人关注它，就可以通过点击劫持来实现

![click1](https://raw.githubusercontent.com/edwineo/Notes/main/Web%E5%AE%89%E5%85%A8/assets/click1.png)

从上图可知，攻击者通过图片作为页面背景，隐藏了用户操作的真实界面，当你按耐不住好奇点击按钮以后，真正的点击的其实是隐藏的那个页面的订阅按钮，然后就会在你不知情的情况下订阅了。

![click2](https://raw.githubusercontent.com/edwineo/Notes/main/Web%E5%AE%89%E5%85%A8/assets/click2.png)

### 2. 如何防御

#### X-FRAME-OPTIONS

`X-FRAME-OPTIONS` 是一个 HTTP 响应头，在现代浏览器中有很好的支持。这个 HTTP 响应头就是为了防御用 iframe 嵌套的点击劫持攻击。

该响应头有三个值可选，分别是

- DENY，表示页面不允许通过 iframe 的方式展示
- SAMEORIGIN，表示页面可以在相同域名下通过 iframe 的方式展示
- ALLOW-FROM，表示页面可以在指定来源的 iframe 中展示



## URL 跳转漏洞

借助未验证的 URL 跳转，将应用程序引导到不安全的第三方区域，从而导致的安全问题。

### 1. 原理

黑客利用 URL 跳转漏洞来诱导安全意识低的用户点击，导致用户信息泄露或者资金的流失。其原理是黑客构建恶意链接（链接需要进行伪装，尽可能迷惑），发在 QQ 群或者是浏览量多的贴吧/论坛中。 安全意识低的用户点击后，经过服务器或者浏览器解析后，跳到恶意的网站中。

![url](https://raw.githubusercontent.com/edwineo/Notes/main/Web%E5%AE%89%E5%85%A8/assets/url.png)

恶意链接需要进行伪装，经常的做法是熟悉的链接后面加上一个恶意的网址，这样才能迷惑用户。

### 2. 实现方式

有以下三种实现方式

- Header 头跳转
- Javascript 跳转
- META 标签跳转

### 3. 如何防御

#### referer 的限制

如果确定了传递 URL 参数进入的来源，我们可以通过该方式实现安全限制，保证该 URL 的有效性，避免恶意用户自己生成跳转链接

#### 使用相对路径

在链接中使用相对路径而不是绝对路径可以防止攻击者通过构造包含恶意 URL 的参数来重定向用户。相对路径只能在站点内部导航，因此可以减少安全风险。



## SQL 注入

SQL注入是一种常见的Web安全漏洞，攻击者利用这个漏洞，可以访问或修改数据，或者利用潜在的数据库漏洞进行攻击。

### 1. 原理

举个例子，我们在登录的时候，后端的 SQL 语句可能是如下这样的：

```ini
let querySQL = `
    SELECT *
    FROM user
    WHERE username='${username}'
    AND psw='${password}'
`;
```

这是我们经常见到的登录页面，但如果有一个恶意攻击者输入的用户名是 `admin' --`，密码随意输入，就可以直接登入系统了。因为我们之前预想的 SQL 语句是：

```sql
SELECT * FROM user WHERE username='admin' AND psw='password'
```

但是恶意攻击者用奇怪用户名将你的 SQL 语句变成了如下形式：

```sql
SELECT * FROM user WHERE username='admin' --' AND psw='xxxx'
```

在 SQL 中,`' --`是闭合和注释的意思，-- 是注释后面的内容的意思，所以查询语句就变成了：

```sql
SELECT * FROM user WHERE username='admin'
```

所谓的万能密码，本质上就是 SQL 注入的一种利用方式。

![sql](https://raw.githubusercontent.com/edwineo/Notes/main/Web%E5%AE%89%E5%85%A8/assets/sql.png)

一次 SQL 注入包括以下几个过程：

- 获取用户请求参数
- 拼接到代码当中
- SQL 语句按照我们构造参数的语义执行成功

SQL 注入的必备条件： 

- 可以控制输入的数据
- 服务器要执行的代码拼接了控制的数据

SQL注入的本质：数据和代码未分离，即数据当做了代码来执行。



### 2. 危害

- 获取数据库信息
  - 管理员后台用户名和密码
  - 获取其他数据库敏感信息：用户名、密码、手机号码、身份证、银行卡信息……
- 获取服务器权限
- 植入 Webshell，获取服务器后门
- 读取服务器敏感文件



### 3. 如何防御

- **严格限制 Web 应用的数据库的操作权限**，给此用户提供仅仅能够满足其工作的最低权限，从而最大限度的减少注入攻击对数据库的危害
- **后端代码检查输入的数据是否符合预期**，严格限制变量的类型，例如使用正则表达式进行一些匹配处理。
- **对进入数据库的特殊字符（'，"，\，<，>，&，\*，; 等）进行转义处理，或编码转换**。基本上所有的后端语言都有对字符串进行转义处理的方法，比如 lodash 的 lodash._escapehtmlchar 库。
- **所有的查询语句建议使用数据库提供的参数化查询接口**，参数化的语句使用参数而不是将用户输入变量嵌入到 SQL 语句中，即不要直接拼接 SQL 语句。例如 Node.js 中的 mysqljs 库的 query 方法中的 ? 占位参数。



## OS 命令注入攻击

OS 命令注入和 SQL 注入差不多，只不过 SQL 注入是针对数据库的，而 OS 命令注入是针对操作系统的。

### 1. 原理

OS 命令注入攻击，是指通过 Web 应用，执行非法的操作系统命令达到攻击的目的。只要在能调用 Shell 函数的地方就有存在被攻击的风险。倘若调用 Shell 时存在疏漏，就可以执行插入的非法命令。

命令注入攻击可以向 Shell 发送命令，让 Windows 或 Linux 操作系统的命令行启动程序。也就是说，通过命令注入攻击可执行操作系统上安装着的各种程序。

![os](https://raw.githubusercontent.com/edwineo/Notes/main/Web%E5%AE%89%E5%85%A8/assets/os.png)

黑客构造命令提交给 web 应用程序，web 应用程序提取黑客构造的命令，拼接到被执行的命令中，因黑客注入的命令打破了原有命令结构，导致 web 应用执行了额外的命令，最后 web 应用程序将执行的结果输出到响应页面中。



我们通过一个例子来说明其原理，假如需要实现一个需求：用户提交一些内容到服务器，然后在服务器执行一些系统命令去返回一个结果给用户

```js
// 以 Node.js 为例，假如在接口中需要从 github 下载用户指定的 repo
const exec = require('mz/child_process').exec;
let params = {/* 用户输入的参数 */};
exec(`git clone ${params.repo} /some/path`);
```

如果 `params.repo` 传入的是 `https://github.com/admin/admin.github.io.git` 确实能从指定的 git repo 上下载到想要的代码。 但是如果 `params.repo` 传入的是 `https://github.com/xx/xx.git && rm -rf /* &&` 恰好你的服务是用 root 权限起的就糟糕了。



### 2. 如何防御

- 后端对前端提交内容进行规则限制（比如正则表达式）。
- 在调用系统命令前对所有传入参数进行命令行参数转义过滤。
- 不要直接拼接命令语句，借助一些工具做拼接、转义预处理，例如 Node.js 的 `shell-escape npm`包

