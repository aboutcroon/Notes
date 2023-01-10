![overview](https://raw.githubusercontent.com/aboutcroon/Notes/main/JavaScript/interview/assets/js%20overview.png)

## 7. 异步编程

### 异步编程的实现方式

JavaScript中的异步机制可以分为以下几种：

1. 回调函数，多个回调函数嵌套的时候会造成回调函数地狱，上下两层的回调函数间的代码耦合度太高，不利于代码的可维护。
2. Promise，可将嵌套的回调函数作为链式调用。但是有时会造成多个 then 的链式调用，可能会造成代码的语义不够明确。
3. generator
4. async 函数，可以将异步逻辑，转化为同步的顺序来书写，并且这个函数可以自动执行。

### setTimeout、Promise、Async/Await 的区别

`setTimeout` 是**宏任务**，Promise 里面的 then 方法是**微任务**，async/await 中 await 语法后面紧跟的表达式是同步的，但 await 下面的代码是异步的，属于**微任务**。



setTimeout 是异步执行函数，当 js 主线程执行到此函数时，不会等待 setTimeout 中的回调函数，会直接进行 setTimeout 下面的语句的执行，当执行完当前事件循环的时候，setTimeout 中的回调会在下一次宏任务的事件循环中被执行。

示例：

```js
console.log('setTimeout start');
setTimeout(function(){
    console.log('setTimeout execute');
})
console.log('setTimeout end');

//输出顺序
1. setTimeout start 
2. setTimeout end 
3. setTimeout execute
```



Promise 本身是同步的立即执行函数，但在执行体中执行 resolve 或者 reject 的时候是异步的，即 **then方法是异步的**。

等同步代码完成后，才会去执行 resolve / reject 中的方法。then() 里面的回调就是一个 task，如果是 resolve 或者 reject 出一个结果了，那么就会把这个 task 放入微任务队列中，等待执行；如果一直是 pending，这个 task 就会放入事件循环的未来的某个(可能下一个)回合的微任务队列中。

setTimeout 的回调也是个 task ，它会被放入宏任务队列，即使是 0ms 的情况。

示例：

```js
console.log('script start')
let promise1 = new Promise(function (resolve) {
    console.log('promise1')
    resolve()
    console.log('promise1 end')
}).then(function () {
    console.log('promise2')
})
setTimeout(function(){
    console.log('settimeout')
})
console.log('script end')

// 输出顺序
1.script start
2.promise1
3.promise1 end
4.script end
5.promise2
6.setimeout
```



async 函数返回的是一个 Promise 对象，在遇到 await 之前是同步执行的，在遇到 await 的时候，会让出主线程，阻塞后面代码的执行。

> 可以理解为 await 之前的代码相当于 Promise 中 resolve 之前的同步代码， await 之后的代码相当于 Promise.then() 里面的回调

async 函数需要等待 await 后的函数执行完成并且有了返回结果（Promise 对象）之后，才能继续执行下面的代码。await 通过返回一个 Promise 对象来实现同步的效果。

来看 async/await，promise，setTimeout 组合起来的执行顺序：

```js
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
}
async function async2() {
    console.log('async2');
}

console.log('start');
setTimeout(() => {
    console.log('setTimeout');
}, 0);
async1();
new Promise(resolve => {
    console.log('promise1');
    resolve();
}).then(() => {
    console.log('promise2');
})
console.log('end');

// 输出顺序
1. start
2. async1 start
3. async2
4. promise1
5. end
6. async1 end
7. promise2
8. setTimeout
```

### 对 Promise 的理解

**Promise 是异步编程的一种解决方案**，最早由社区提出。Promise 是一个构造函数，接收一个回调函数作为参数，返回一个 Promise 实例。

一个 Promise 实例有三种状态

- Pending（进行中）
- Resolved（已完成）
- Rejected（已拒绝）

当把一件事情交给 Promise 时，它的状态就是 Pending，任务完成了状态就变成了 Resolved，没有完成失败了就变成了 Rejected。

Promise 实例有两个过程

- pending -> fulfilled : Resolved（已完成）
- pending -> rejected：Rejected（已拒绝）

实例的状态只能由 pending 转变 resolved 或者 rejected 状态，并且状态一经改变，就凝固了，无法再被改变了。

状态的改变是通过 resolve() 和 reject() 函数来实现的，可以在异步操作结束后调用这两个函数改变 Promise 实例的状态，它的原型上定义了一个 then 方法，使用这个 then 方法可以为两个状态的改变注册回调函数。这个回调函数属于微任务，会在本轮事件循环的末尾执行。

> **注意：** 在构造 `Promise` 的时候，构造函数内部的代码是立即执行的

### Promise 的基本用法

Promise 对象可以使用 `new Promise` 的形式创建，也可以使用 `Promise.resolve(value)` 或 `Promise.reject(value)` 的形式创建。

> `Promise.resolve(x)` 可以看作是 `new Promise(resolve => resolve(x))` 的简写，可以用于快速封装字面量对象或其他对象，将其封装成 Promise 实例。

Promise 有五个常用的方法：then()、catch()、finally、Promise.all()、Promise.race()。

catch 方法相当于 `then` 方法的第二个参数，指向 `reject` 的回调函数。不过`catch`方法还有一个作用，就是在执行`resolve`回调函数时，如果出现错误，抛出异常，不会停止运行，而是进入`catch`方法中。

Promise.all 可以完成并行任务， 它接收一个数组，数组的每一项都是一个 `promise` 对象。当数组中所有的 `promise` 状态都达到 `resolved` 的时候，Promise.all 方法的状态就会变成 `resolved`，如果有一个状态变成了 `rejected`，那么 Promise.all 方法的状态就会变成 `rejected`。

Promise.race 与 all 方法一样，只不过是当最先执行完的事件执行完之后，就直接返回该 `promise` 对象的值。如果第一个 Promise 对象状态变成 `resolved`，那自身的状态变成了 `resolved`；反之第一个 Promise 变成 `rejected`，那自身状态就会变成 `rejected`。

### Promise解决了什么问题

如果后一个请求需要依赖于前一个请求成功后，将数据往下传递，那么会导致多个 ajax 请求嵌套的情况，代码不够直观。

Promise 的链式调用，解决了回调地狱的问题。

### 对 async/await 的理解

async/await 其实是 `Generator` 的语法糖，它能实现的效果都能用 then 的链式调用来实现，它是为优化 then 的链式调用而开发出来的。从字面上来看，async 是“异步”的简写，await 则为等待，所以很好理解 async 用于申明一个 function 是异步的，而 await 用于等待一个异步方法执行完成。当然语法上强制规定 await 只能出现在 asnyc 函数中。

### await 到底在等待什么

await 表达式的运算结果取决于它等待的是什么。

- 如果它等待的不是一个 Promise 对象而是一个任意表达式，那 await 表达式的运算结果就是它等待的这个表达式的返回结果。
- 如果它等待的是一个 Promise 对象，那 await 就忙起来了，它会阻塞后面的代码，等着 Promise 对象 resolve，然后得到 resolve 的值，作为 await 表达式的运算结果。

看一个示例：

```javascript
function testAsy(x){
   return new Promise(resolve => { setTimeout(() => {
         resolve(x);
       }, 3000)
     }
   )
}
async function testAwt(){    
  let result = await testAsy('hello world');
  console.log(result);
  console.log('cuger')
}
testAwt();
console.log('cug')

// 立即输出 cug
// 3秒钟之后出现 hello world
// 3秒钟之后出现 cuger
```

await 暂停当前 async 的执行，所以 cug 最先输出，hello world 和 cuger 是3秒钟后同时出现的。

这就是 await 必须用在 async 函数中的原因。**因为 async 是异步函数，它的调用不会造成主线程同步代码的阻塞**，它内部所有的阻塞都被封装在一个 Promise 对象中异步执行。

## 8. 面向对象

### 对象创建的方式有哪些

1. 工厂模式
2. 构造函数模式
3. 原型模式
4. 组合使用构造函数和原型模式
5. 动态原型模式
6. 寄生构造函数模式

### 对象继承的方式有哪些

1. 原型链的方式
2. 借用构造函数的方式
3. 组合继承，将原型链与构造函数组合起来
4. 原型式继承
5. 寄生式继承
6. 寄生式组合继承

## 9. 垃圾回收与内存泄漏

### 垃圾回收的概念

#### 垃圾回收

JavaScript 代码运行时，需要分配内存空间来储存变量和值。当变量不再参与运行时，就需要系统收回被占用的内存空间，这就是垃圾回收。

#### 回收机制

JS 引擎中有一个后台进程称为垃圾回收器，它监视所有对象，观察对象是否可被访问，然后按照固定的时间间隔周期性的删除掉那些不可访问的对象即可。



### 垃圾回收的方式

现在各大浏览器通常用采用的垃圾回收方式有两种：

#### 引用计数

最早最简单的垃圾回收机制，就是给一个占用物理空间的对象附加一个引用计数器，当有其它对象引用这个对象时，这个对象的引用计数加 1，反之解除时就减 1，当该对象引用计数为 0 时就会被回收。

该方式很简单，但会引起内存泄漏：

```js
// 循环引用的问题
function temp () {
    const obj1 = {}
    const obj2 = {}
    obj1.a = obj2 // obj1 引用 obj2
    obj2.a = obj1 // obj2 引用 obj1
}
```

这种情况下每次调用 `temp` 函数，`obj1` 和 `obj2` 的引用计数都是 `2` ，会使这部分内存永远不会被释放，即内存泄漏。现在已经很少使用了，只有低版本的 IE 使用这种方式。

> 只能手动释放变量占用的内存：
>
> ```js
> obj1.a = null
> obj2.a = null
> ```

#### 标记清除

V8 引擎中主垃圾回收器就采用标记清除法进行垃圾回收。主要流程如下：

1. 在变量进入执行上下文时打上“进入”标记
2. 同时在变量离开执行上下文时也打上“离开”标记
3. 标记为“离开”的变量，从此以后是无法访问这个变量的，在下一次垃圾回收时该变量占用的内存将会被释放

> 所以我们需要尽可能的使用 const 和 let，因为 const 和 let 使 JS 有了块级作用域，当块级作用域比函数作用域更早结束时，垃圾回收器可以更早介入。

![mark and sweep](https://raw.githubusercontent.com/aboutcroon/Notes/main/JavaScript/interview/assets/mark%20and%20sweep.webp)

> 拓展
>
> 如果一个对象被多次引用时，例如作为另一对象的键、值或子元素时，将该对象引用设置为 `null` 时，该对象是不会被回收的，依然存在。例如：
>
> ```javascript
> let a = {}
> let arr = [a]
> 
> a = null;
> console.log(arr)
> // [{}]
> ```
>
> 如果作为 `Map` 的键呢？则也是如此，ES6 考虑到了这一点，推出了： `WeakMap` 。它对于值的引用都是不计入垃圾回收机制的，所以名字里面才会有一个"Weak"，表示这是弱引用（对对象的弱引用是指当该对象应该被GC回收时不会阻止GC的回收行为）。
>
> `Map` 相对于 `WeakMap` ：
>
> - `Map` 的键可以是任意类型，`WeakMap` 只接受对象作为键（null除外），不接受其他类型的值作为键。
> - `Map` 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键； `WeakMap` 的键是弱引用，键所指向的对象可以被垃圾回收，**此时键是无效的**。
> - `Map` 可以被遍历， `WeakMap` 不能被遍历。

### 如何减少垃圾回收

当代码比较复杂时，浏览器进行垃圾回收的代价会变得比较大，所以应该尽量减少垃圾回收。

- 数组优化：在清空一个数组时，我们可以尽量将数组长度设为 0，`arr.length = 0`，而不是 `arr = []`，这样会创建一个新的空对象，对垃圾回收不友好。
- 对象优化：对象尽量复用，对于不再使用的对象，将其设置为 null 使之尽快被回收。
- 函数优化：在循环中的函数表达式，如果有可以复用的部分，尽量放在函数的外面。

### 哪些情况会导致内存泄漏

1. **意外的全局变量**
2. **被遗忘的计时器或回调函数**：设置了 setInterval 定时器，而忘记取消它，如果定时器的回调函数有对外部变量的引用的话，那么这个变量会被一直留在内存中，而无法被回收。
3. **脱离 DOM 的引用**：获取一个 DOM 元素的引用，而后这个 DOM 元素被删除，那么由于一直保留了对这个元素的引用，那么它也将无法被回收。
4. **不合理的闭包**：使用闭包的方式不合理时，将导致某些变量一直被引用从而一直无法被回收。



