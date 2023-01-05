# 学习

## 参数

### 参数获取

参数获取也就是如何在服务器中拿到发送过来的参数。根据传参方式的不同有不同的获取方式。



传参方式与获取方式：

1. 放 header 中

通过 `const headers = ctx.request.header` 来获取

例如 `token` 的传递



2. 放 body 中（post请求）

不能直接获取，要通过 `koa-bodyparser` 来获取

先引入 koa-bodyparser，再注册，因为它不是一个中间件，所以 app.use 的时候，在 parser 后面加个括号

```js
const Koa = require('koa')
const parser = require('koa-bodyparser')

const app = new Koa()

app.use(parser())

app.listen(3000)
```

然后就可以 `const body = ctx.request.body`



3. 放 `?` 后面（查询参数）

通过 `const query = ctx.request.query` 获取



4. 路径中携带参数，放在版本号后面（params）

通过 `const path = ctx.params` 获取



### 参数校验

有些参数传的不对，有的参数不能缺少必须要传。

但每个参数一一校验会很繁琐。所以我们要依赖第三方的校验库。



## 处理异常

### 异步异常处理

执行函数时，有下列几种情况：

1. 没有发生异常，正确返回结果
2. 没有发生异常，不需要返回结果
3. 发生异常



于是，在函数处理上，可以采取以下两种方式：

1. 判断出了异常，`return false / null`。（不推荐，因为这样会丢失异常的信息）
2. 判断出了异常，抛出异常 `throw new Error()`。（推荐，符合编程规范，可以让开发者知道什么地方发生了什么错误）

```js
function func() {
  try {
    1 / a	// a未定义，所以是错的
  } catch (error) {
    throw error	// 捕捉到错误之后，将其抛出
  }
  return 'success'
}
```

函数 1 调用函数 2，函数 2 调用函数 3，在每个函数中都去捕获异常的话会很繁琐，所以需要一个在最顶部的函数，来监听任何的异常。



但是 `try catch` 对异步的异常很难捕获，所以我们需要弄清楚如何处理异步的异常。

异步的异常处理，要通过 `Promise async await` 一起。在异步过程中 `reject` 的值是异常，上一层的函数通过 `await` 则可以捕获成功：

```js
async function func2() {
  try {
    await func3()	// await对一个表达式求值
  } catch (error) {	// 此处可捕获成功
    console.log(error)
  }
}

function func3() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const r = Math.random()
      if (r < 0.5) {
        reject('error async')
      }
    },1000)
  })
}
```

在后续，Koa 中的各种库和包都已经封装好了返回给我们的 Promise，所以我们就不用再像上述一样自己封装 Promise了，然后我们可以直接加上 `async await`，然后 `try catch` 就可以捕获到异步的异常了。



## 全局异常处理中间件

使用 koa 中间件来实现全局的异常处理

1. 首先在全局监听到这个错误
2. 然后输出一段 `有意义` 的提示信息（不是系统的那种让人看不懂的提示信息）

如下：

```js
const catchError = async (ctx, next) => {
  try {
    await next()
  } catch (error) {
    ctx.body = '服务器出了问题'
  }
}

module.exports = catchError
```

使用了 AOP（面向切面编程）思想。



### 已知错误与未知错误

在程序中捕捉到的 Error 不应该直接返回到客户端去，因为 Error 的信息量是非常大的，除了包含常见的错误名称和错误原因，还有堆栈调用等信息，这些复杂的信息不应该返回到客户端去，这些信息应该进行简化，然后返回一些清晰明了的信息给前端。

我们需要了解 HTTP 的各种状态码，然后自己定义具体的 error_code 来告知详细的错误。



错误类型如下

已知型错误：我们可以预料到的错误。例如校验某个字段时，输入的字段类型不对。

未知型错误：程序的潜在错误，是无意识下产生的错误，预料不到。例如连接数据库时，账号密码出错了。



### 定义异常返回格式

如果我们在 catch 到的 Error 中可以取到我们自己定义的错误信息及错误码，就会很方便。那么我们需要在 Error 上定义自己的错误信息与错误码。

那么，如何定义呢，如下：

```js
router.post('url', (ctx, next) => {
  const query = ctx.request.query
  const request_method = ctx.method		// HTTP 动词
  const request_url = ctx.path		// HTTP 的 url
  if (!query) {	// if (某种类型的错误)
    const error = new Error('错误信息')	// 错误的message是在这里传递的，因为Error自身已经有message这个属性了，然后则可以通过error.message来取到，或者也可以通过error.message = '错误信息'这样子来写
    error.error_code = 10001	// 定义 error_code 的属性
    error.status = 400
    throw error		// 将其抛出
  }
})
```

上述代码中，根据相应的已知错误，来自定义 `error_code`，然后将其抛出，最后我们在全局异常捕获中就能 catch 到 Error，那么这个 Error 中就会携带 `error_code`（HTTP 动词可以通过 `ctx.method` 来拿到，url 可以通过 `ctx.path` 来拿到）。

那么，全局异常捕获器可以通过是否有 `error_code` 来判断是否是 `已知错误`：

```js
const catchError = async (ctx, next) => {
  try {
    await next()
  } catch (error) {
    if (error.error_code) {		// 判断是否是已知错误
      ctx.body = {
        msg: error.message,
        error_code: error.error_code
      }
      ctx.status = error.status	// 状态码
    }
  }
}

module.exports = catchError
```



### 定义异常基类

一个一个的这么写异常，会很繁琐，所以我们可以用面向对象的方式来写，可以新建一个 `.js` 文件，在文件里封装一个类：

```js
class HttpException extends Error {		// 定义HttpException类，并继承内置的Error，因为要将HttpException抛出去，所以其必须是一个Error类型
  constructor(error_code = 10000, code = 400, msg = '服务器异常') {	// 传入参数，定义默认值
    super()
    this.error_code = error_code
    this.code = code
    this.msg = msg
  }
}

module.exports = HttpException
```

改写后，在业务代码中引入该 `HttpException` 类：

```js
const HttpException = require('../../core/http-exception')

router.post('url', (ctx, next) => {
  const query = ctx.request.query
  const request_method = ctx.method
  const request_url = ctx.path
  if (!query) {
    const error = new HttpException(10000, 400, '错误信息')
    throw error
  }
})
```

```js
const HttpException = require('../../core/http-exception')

const catchError = async (ctx, next) => {
  try {
    await next()
  } catch (error) {
    if (error instanceof HttpException) {		// 根据是否是HttpException的类来判断是否是已知错误
      ctx.body = {
        msg: error.message,
        error_code: error.error_code
        request_url: ctx.method + ctx.path
      }
      ctx.status = error.code
    } else {
      // 处理未知错误
      ctx.body = {
        ...
      }
      ctx.status = 500	// 未知错误的状态码一般是500
    }
  }
}

module.exports = catchError
```



### 定义特定异常类

可以定义特定的异常类，来继承基类，那么遇到相应的错误就不用进行 if 的判断了，直接使用特定的异常类。

```js
class HttpException extends Error {
  constructor(error_code = 10000, code = 400, msg = '服务器异常') {
    super()
    this.error_code = error_code
    this.code = code
    this.msg = msg
  }
}

class ParameterException extends HttpException {
  constructor(error_code = 10000, msg = '服务器异常') {
    this.code = 400
    this.error_code = error_code
    this.msg = msg
  }
}

module.exports = {
  HttpException,
  ParameterException
}
```

可以将这些所有的异常类，在全局初始化时，都放在一个 `global` 对象下面，这样就不用在每个业务代码里面都要引入相应的异常类了。



## 配置文件与环境

抛出异常时，我们要区分是 `开发环境` 还是 `生产环境`，开发环境则需要抛出异常到终端。

可以新建一个 `config.js` 文件来配置当前环境是 dev 还是 prod，然后在全局初始化时引入它进行环境的判断：

```js
module.exports = {
  environment: 'dev'
}
```



## 数据库设计

通过 ORM 方式来操作数据库



### Sequelize

使用 `Sequelize` 可以将代码中的 model 自动对应生成数据库中的表

新建 `db.js` 文件来配置 Sequelize

Sequelize 接收四个参数：数据库名称，username，password，一个 js 对象（用来传递更多参数）

像数据库名称，用户名等不应该直接写进去，应该写到配置文件 `config.js` 中，然后再导入进去：

```js
module.exports = {
  environment: 'dev',
  database: {
    dbName: 'xx',
    host: 'localhost',
    port: 3306,
    user: 'root',
    password: '123456',
  }
}
```

然后，在 `db.js` 中：

```js
const Sequelize = require('sequelize')

const {
    dbName,
    host,
    port,
    user,
    password
} = require('../config/config').database

const sequelize = new Sequelize(dbName, user, password, {
    dialect: 'mysql',		// 指定数据库的类型是mysql，这样就可以连接mysql，需要先用npm安装mysql的驱动
    host,
    port,
    logging: true,	// 显示sql操作
    timezone: '+08:00',		// 不然会相差8个小时
    define: {	// 配置个性化参数
        // create_time && update_time
        timestamps: true,
        // delete_time
        paranoid: true,
        createdAt: 'created_at',
        updatedAt: 'updated_at',
        deletedAt: 'deleted_at',
        // 把驼峰命名转换为下划线
        underscored: true,
        freezeTableName: true,
        scopes: {
            bh: {
                attributes: {
                    exclude: ['updated_at', 'deleted_at', 'created_at']
                }
            }
        }
    }
})

module.exports = {
    sequelize
}
```

接着在 `models` 文件夹下创建相应的模型，例如 `user.js`。



# 业务

## koa 的 default status 为 404

问题：router.use(ctx => {

 ctx.status = 200

})



## [Koa-body](https://github.com/koajs/koa-body)

### 上传文件

若要上传文件，首先要将 `multipart` 置为 true

```js
const body = require('koa-body')
app.use(body({
  multipart: true // 允许上传多个文件
}))
```

使用 `koa-body` 无法向 `multer` 直接通过 `diskStorage` 进行存储路径和文件名的配置，需要借助 `onFileBegin:(name,file)=>{}`进行。

具体，参考文章：http://www.ptbird.cn/koa-body-diy-upload-dir-and-filename.html



若要通过请求发送二进制文件，那么client端需要通过 formData 才能发送文件，且 `Content-Type: multipart/form-data`

```js
const formData = new FormData()
formData.append('file', blob, fileName) // blob即我们生成的二进制文件
```

然后将这个 formData 放到 POST 请求的 body 中即可



> koa 若要接收该二进制文件，不能通过 ctx.request.body 直接取，需要这样 `const { file } = ctx.request.files`
>
> 不过现在有更好的解决方法，直接更改 koa-body 将上传的文件存放到指定地址，然后再通过路径去拿就可以了，因为如果不指定地址，koa-body 上传的文件也会默认有个暂存的地址的。如果像上述这样拿到文件后还需要将文件再写入新的存放地址

