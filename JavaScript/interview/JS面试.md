# JS基础

## 1.数据类型

JS 数据类型分为两大类，九个数据类型：

1. 原始类型
2. 对象类型



其中原始类型又分为七种类型，分别为：

- `boolean`
- `number`
- `string`
- `undefined`
- `null`
- `symbol`
- **`bigint`**

对象类型分为两种，分别为：

- `Object`
- `Function`

其中 `Object` 中又包含了很多子类型，比如 `Array`、`RegExp`、`Math`、`Map`、`Set` 等等，也就不一一列出了。

原始类型存储在栈上，对象类型存储在堆上，但是它的引用地址还是存在栈上。



### 常见考点

1. JS 类型有哪些？

2. 对象的修改，比如说往函数里传一个对象进去，函数内部修改参数。

   ```js
   function test(person) {
     person.age = 26
     person = {
       name: 'yyy',
       age: 30
     }
   
     return person
   }
   const p1 = {
     name: 'lc',
     age: 25
   }
   const p2 = test(p1)
   console.log(p1) // { name: 'lc', age: 26 }
   console.log(p2) // { name: 'yyy', age: 30 }
   ```

   - 首先，函数传参是传递对象指针的副本
   - 到函数内部修改参数的属性这步，我相信大家都知道，当前 `p1` 的值也被修改了
   - 但是当我们重新为 `person` 分配了一个对象时就出现了分歧，最后的 `person` 拥有了一个新的地址（指针），也就和 `p1` 没有任何关系了，导致了最终两个变量的值是不相同的。

   

   这类题目我们只需要牢记以下几点：

   - 对象存储的是引用地址，传来传去、赋值给别人那都是在传递值（存在栈上的那个内容），别人一旦修改对象里的属性，大家都被修改了。

   - 但是一旦对象被重新赋值了，只要不是原对象被重新赋值，那么就永远不会修改原对象。

   

3. 大数相加、相乘算法题，可以直接使用 `bigint`。

   [BigInt MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/BigInt)

   https://blog.csdn.net/m0_46515581/article/details/108180950

   ```js
   // 绝对值大于或等于 2^53 的数值文本过大，无法用整数准确表示。ts(80008)
   const a = 9007199254740992n
   const b = 9007199254740992n
   
   const c = a + b
   
   console.log(c)
   ```

   

4. `NaN` 如何判断？

   通过 `isNaN()` 和 `Number.isNaN()`

   参考：https://segmentfault.com/a/1190000011800831

   

   **为什么NaN!==NaN?**

   NaN只是Number上的一个静态属性。

   Number('echo')     //NaN

   比如Number('echo')会得到NaN，它只是为了告诉你这个值不是一个数字，一种表示方法，而非一个精准有效的值，因此NaN不能参与计算，也无法与自身比较。

   

   > Object.is() 可以解决`+0`等于`-0`，`NaN`不等于自身的问题。
   >
   > 参考：[ES6阮一峰](https://es6.ruanyifeng.com/#docs/object-methods#Object-is)



## 2.类型判断

### typeof

原始类型中除了 `null`，其它类型都可以通过 `typeof` 来判断。

`typeof null` 的值为 `object`，如果想具体判断 `null` 类型的话直接 `xxx === null` 即可。

对于对象类型来说，`typeof` 只能具体判断函数的类型为 `function`，其它均为 `object`。

### instanceof

`instanceof` 内部通过`原型链`的方式来判断是否为构建函数的实例，常用于判断具体的对象类型。

> 注意 `[] instanceof Object` 和 `[] instanceof Array` 的结果都为 `true`，因为 Array 也是 Object 的实例，所以判断 Array 可以使用 `Array.isArray([])` 来判断

### 构造函数

其实我们还可以直接通过构建函数来判断类型：

```js
[].constructor === Array // true
```

### Object.prototype.toString

前几种方式或多或少都存在一些缺陷，`Object.prototype.toString` 综合来看是最佳选择，能判断的类型最完整。

```js
Object.prototype.toString.call(null) // [Object null]
Object.prototype.toString.call(1) // [Object Number]
Object.prototype.toString.call('') // [Object String]
Object.prototype.toString.call(1n) // [Object BigInt]
Object.prototype.toString.call([]) // [Object Array]
Object.prototype.toString.call({}) // [Object Object]
Object.prototype.toString.call(function () {}) // [Object Function]
```



### 常见考点

1. JS 类型如何判断，有哪几种方式可用
2. `instanceof` 原理
3. 手写 `instanceof`



## 3.垃圾回收机制

JS 中的垃圾数据都是由垃圾回收（Garbage Collection，缩写为 GC）器自动回收的，不需要手动释放，它是如何做到的呢？

很简单，JS 引擎中有一个后台进程称为垃圾回收器，它监视所有对象，观察对象是否可被访问，然后按照固定的时间间隔周期性的删除掉那些不可访问的对象即可。

现在各大浏览器通常用采用的垃圾回收有两种方法：

- 引用计数
- 标记清除

### 引用计数

最早最简单的垃圾回收机制，就是给一个占用物理空间的对象附加一个引用计数器，当有其它对象引用这个对象时，这个对象的引用计数加一，反之解除时就减一，当该对象引用计数为 0 时就会被回收。

该方式很简单，但会引起内存泄漏：

```js
// 循环引用的问题
function temp(){
    var a={};
    var b={};
    a.o = b;
    b.o = a;
}
```

这种情况下每次调用 `temp` 函数，`a` 和 `b` 的引用计数都是 `2` ，会使这部分内存永远不会被释放，即内存泄漏。现在已经很少使用了，只有低版本的 IE 使用这种方式。



### 标记清除

V8 中主垃圾回收器(Garbage Collection)就采用标记清除法进行垃圾回收。主要流程如下：

1. 在变量进入执行上下文时打上“进入”标记
2. 同时在变量离开执行上下文时也打上“离开”标记
3. 从此以后，无法访问这个变量，在下一次垃圾回收时进行内存的释放

> 所以我们需要尽可能的使用const和let，因为const和let使JS有了块级作用域，当块级作用域比函数作用域更早结束时，垃圾回收程序更早介入

![mark and sweep](https://raw.githubusercontent.com/aboutcroon/Notes/main/JavaScript/interview/assets/mark%20and%20sweep.webp)

#### WeakMap 与 Map

但如果一个对象被多次引用时，例如作为另一对象的键、值或子元素时，将该对象引用设置为 `null` 时，该对象是不会被回收的，依然存在。

```javascript
var a = {}
var arr = [a]

a = null;
console.log(arr)
// [{}]
```



如果作为 `Map` 的键呢？

```js
let a = {}; 
const map = new Map()
map.set(a, '1')

a = null

console.log(map.keys()) // MapIterator {{}}
console.log(map.values()) // MapIterator {"1"}
```

可以看到，即使将 a 对象设置成 `null`，该对象仍不会被回收，依然存在，如果想让 a 置为 `null` 时，该对象被回收，该怎么做呢？



ES6 考虑到了这一点，推出了： `WeakMap` 。它对于值的引用都是不计入垃圾回收机制的，所以名字里面才会有一个"Weak"，表示这是弱引用（对对象的弱引用是指当该对象应该被GC回收时不会阻止GC的回收行为）。

`Map` 相对于 `WeakMap` ：

- `Map` 的键可以是任意类型，`WeakMap` 只接受对象作为键（null除外），不接受其他类型的值作为键。
- `Map` 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键； `WeakMap` 的键是弱引用，键所指向的对象可以被垃圾回收，**此时键是无效的**。
- `Map` 可以被遍历， `WeakMap` 不能被遍历。



下面再使用 `WeakMap`，重复上述代码：

```js
let a = {}; 
const map = new WeakMap()
map.set(a, '1')

a = null

console.log(map.get(a)) // undefined
console.log(map.has(a)) // false
```

这时 a 对象就被回收了



## 4. HTTP 状态码

常见状态码

| code | message                | 说明                     | 使用场景                                                     |
| :--- | :--------------------- | :----------------------- | :----------------------------------------------------------- |
| 200  | 请求成功               | Ok                       | 所有成功的请求中。                                           |
| 201  | 创建xx成功             | Created                  | 服务器创建数据成功时。                                       |
| 202  | 已接收请求，处理中     | Accept                   | 短信发送、邮件通知、模板消息推送等耗时较长需要队列支持的场景。 |
| 204  | 删除成功 / 更新成功    | No Content               | 使用 `Delete` 删除资源成功时，或使用 `Put` 更新资源成功时。  |
| 301  | 请求的资源已被永久迁移 | Moved Permanently        | 请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替。 |
| 302  | 请求的资源被临时迁移   | Found                    | 临时移动。与301类似，但资源只是临时被移动，客户端应继续使用原有URI。 |
| 304  | 所请求的资源未修改     | Not Modified             | 所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源。 |
| 400  | 请求语法错误           | Bad Request              | 客户端请求的语法错误，服务器无法理解。                       |
| 401  | 身份未认证             | Unauthorized             | 未进行身份认证的用户访问需要认证的API时，或`token`无效 / 过期时。 |
| 403  | 权限不足               | Forbidden                | 该状态码可以理解为没有权限访问该请求，服务器收到请求但拒绝提供服务。例如当普通用户请求操作管理员用户时。 |
| 404  | 资源未找到             | Not Found                | 当用户请求的资源不存在时，都必须返回该状态码。若该资源已永久不存在，则应该返回 `410` 响应。 |
| 405  | HTTP 请求方法不被允许  | Method Not Allowed       | 客户端请求中的方法被禁止，当客户端使用的 HTTP 请求方法不被服务器允许时，必须返回该状态码。该响应必须返回一个带有 `Allow` 的响应头信息用以表示出当前资源能够接受的请求方法的列表。 |
| 406  | 不支持的数据格式       | Not Acceptable           | 服务器无法根据客户端请求的内容特性完成请求。当 API 在不支持客户端指定的数据格式时，应该返回此状态码。例如支持 `JSON` 和 `XML` 输出的 `API` 被指定返回 `YAML` 格式的数据时。(Http 协议一般通过请求首部的 `Accept` 来指定数据格式) |
| 408  | 客户端请求超时         | Request Time-out         | 服务器等待客户端发送的请求时间过长，超时。当客户端请求超时的时候必须返回该状态码，需要注意的是，该状态码表示 **客户端请求超时**，在涉及第三方 `API` 调用超时的时候，不可返回该状态码。 |
| 409  | 请求存在冲突           | Conflict                 | 服务器完成客户端的 `PUT` 请求时可能返回此代码，服务器处理请求时发生了冲突。该状态码表示因为请求存在冲突无法处理。例如通过手机号码提供注册功能的 API ，当用户提交的手机号已存在时，必须返回此状态码。 |
| 410  | 该资源永久被删除       | Gone                     | 客户端请求的资源已经不存在。 `410` 不同于 `404` ，如果资源以前有现在被永久删除了可使用 `410` 代码，可通过 `301` 代码指定资源的新位置。 |
| 413  | 请求数据过大           | Request Entity Too Large | 由于请求的实体数据过大，服务器无法处理，因此拒绝请求。为防止客户端的连续请求，服务器可能会关闭连接。如果只是服务器暂时无法处理，则会包含一个 `Retry-After` 的响应头，告知客户端可以在多少时间后重新尝试访问。 |
| 414  | 请求的url过长          | Request-URI Too Large    | 请求的URI过长，服务器无法处理。                              |
| 415  | 不允许上传的图片格式   | Unsupported Media Type   | 服务器无法处理请求附带的媒体格式。例如只允许上传图片格式的文件，但是客户端提交媒体文件非法或不是图片类型，这时应该返回该状态码。 |
| 429  | 请求次数过多           | Too Many Request         | 该状态码表示用户请求次数超过允许范围。例如 API 设定为 `60次/分钟`，当用户在一分钟内请求次数超过 60 次后，都应该返回该状态码。并且在响应头中加入相应内容。 |
| 500  | 服务器错误             | Internal Server Error    | 服务器内部错误，无法完成请求。该状态码必须在服务器出错时抛出。 |
| 503  | 服务器维护中           | Service Unavailable      | 由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的 `Retry-After` 响应头信息中。 |
