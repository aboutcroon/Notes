## 优先级

被谁调用，this就指向谁。

七步口诀：

1. 箭头函数
2. new
3. bind
4. apply 和 call
5. obj.
6. 直接调用
7. 不在函数里

## 箭头函数

箭头函数排在第一个是因为它的 this 不会被改变，所以只要当前函数是箭头函数，那么就不用再看其他规则了。

箭头函数的 this 指向是在创建它时外层 this 的指向。这里的重点有两个：

1. **创建箭头函数时**，就已经确定了它的 this 指向。
2. 箭头函数内的 this 指向**外层的 this**。

所以要知道箭头函数的 this 就得先知道外层 this 的指向，需要继续在外层应用七步口诀。

## new

**当使用 new 关键字调用函数时，函数中的 this 一定是 JS 创建的新对象。**

读者可能会有疑问，“如果使用 new 关键调用箭头函数，是不是箭头函数的 this 就会被修改呢？”。

答案是：箭头函数不能当做构造函数，所以不能与 new 一起执行。

## bind

bind 是指 [Function.prototype.bind()](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FFunction%2Fbind)。

### 多次 bind 时只认第一次 bind 的值

```js
function func() {
  console.log(this)
}

func.bind(1).bind(2)() // 1
```

### 箭头函数中 this 不会被 bind 修改

```js
func = () => {
  // 这里 this 指向取决于外层 this，参考口诀 7 「不在函数里」
  console.log(this)
}

func.bind(1)() // 指向Window，因为口诀 1 优先
```

### bind 与 new

```js
function func() {
  console.log(this, this.__proto__ === func.prototype)
}

boundFunc = func.bind(1)
new boundFunc() // Object true，口诀 2 优先
```

## apply 和 call

`apply()` 和 `call()` 的第一个参数都是 this，区别在于通过 apply 调用时实参是放到数组中的，而通过 call 调用时实参是逗号分隔的。

### bind 函数中 this 不会被修改

```js
function func() {
  console.log(this)
}

boundFunc = func.bind(1)
boundFunc.apply(2) // 1，口诀 3 优先
```

## obj.

```js
function func() {
  console.log(this.x)
}

obj = { x: 1 }
obj.func = func
obj.func() // 1
```

同理，箭头函数和 bind 函数的优先级更高。

## 直接调用

在函数不满足前面的场景，被直接调用时，this 将指向全局对象。在浏览器环境中全局对象是 Window，在 Node.js 环境中是 Global。

```js
function func() {
  console.log(this)
}

func() // Window
```

来一个复杂的例子，外层的 outerFunc 就起个迷惑目的。

```js
function outerFunc() {
  console.log(this) // { x: 1 }

  function func() {
    console.log(this) // Window
  }

  func()
}

outerFunc.bind({ x: 1 })()
```

> 可以看成 function f1 = outerFunc.bind({ x: 1 }) 然后再执行 f1()，所以最后也相当于上面的例子1一样，是由 window 调用的

## 不在函数里

不在函数中的场景，可分为浏览器的 `<script />` 标签里，或 Node.js 的模块文件里。

1. 在 `<script />` 标签里，this 指向 Window。
2. 在 Node.js 的模块文件里，this 指向 Module 的默认导出对象，也就是 module.exports。



## 非严格模式

严格模式是在 ES5 提出的。在 ES5 规范之前，也就是非严格模式下，this 不能是 undefined 或 null。所以**在非严格模式下，通过上面七步口诀，如果得出 this 指向是 undefined 或 null，那么 this 会指向全局对象。**在浏览器环境中全局对象是 `Window`，在 Node.js 环境中是 `Global`。严格模式下则会为 undefined 或 null。

例如下面的代码，在非严格模式下，this 都指向全局对象。

```js
function a() {
  console.log("function a:", this);
  (() => {
    console.log("arrow function: ", this)
  })()
}

a()

a.bind(null)()

a.bind(undefined)()

a.bind().bind(2)()

a.apply()
```



## 参考文章

https://juejin.cn/post/6946021671656488991

