## 红宝书定义

例子：

```js
let x = 3
setTimeout(() => x = x + 4, 1000)
```

为了让后续代码能够使用 x，`异步执行的函数需要在更新 x 的值以后通知其他代码`。如果程序不需要这个值，那么就只管继续执行，不必等待这个结果了。

设计一个能够知道 x 什么时候可以读取的系统是非常难的。JavaScript 在实现这样一个系统的过程中也经历了几次迭代。



### 以往的异步编程模式

通常需要深度嵌套的回调函数（俗称“回调地狱”）来解决。



### Promise基础

Promise 可以通过 new 操作符来实例化。创建新 Promise 时需要传入执行器（executor）函数作为参数。

Promise 是一个有状态的对象，可能处于如下3种状态之一：

- pending
- fulfilled
- rejected

pending 是初始状态，可以从这个初始状态转变为 fulfilled 或 rejected 状态，并且是不可逆的，转变状态后则不再改变。但是，也不能保证 Promise 必然会脱离 pending 状态。因此，组织合理的代码无论 Promise 解决（resolve）还是拒绝（reject），甚至永远处于待定（pending）状态，都应该具有恰当的行为。



期约的状态是私有的，不能直接通过JavaScript检测到。这主要是为了避免根据读取到的期约状态，以同步方式处理期约对象。另外，期约的状态也不能被外部JavaScript代码修改。这与不能读取该状态的原因是一样的：期约故意将异步行为封装起来，从而隔离外部的同步代码。



#### 通过执行函数控制期约状态

由于期约的状态是私有的，所以只能在内部进行操作。内部操作在期约的执行器函数中完成。执行器函数主要有两项职责：初始化期约的异步行为和控制状态的最终转换。其中，控制期约状态的转换是通过调用它的两个函数参数实现的。这两个函数参数通常都命名为resolve()和reject()。调用resolve()会把状态切换为兑现，调用reject()会把状态切换为拒绝。`另外，调用reject()也会抛出错误`（后面会讨论这个错误）。

#### Promise.solve

期约并非一开始就必须处于待定状态，然后通过执行器函数才能转换为落定状态。通过调用Promise.resolve()静态方法，可以实例化一个解决的期约。下面两个期约实例实际上是一样的：

```js
let p1 = new Promise((resolve, reject) => resolve())
let p2 = Promise.resolve()
```

这个解决的期约的值对应着传给Promise.resolve()的 `第一个参数`。使用这个静态方法，实际上可以把任何值都转换为一个期约

对这个静态方法而言，如果传入的参数本身是一个期约，那它的行为就类似于一个空包装。因此，Promise.resolve()可以说是一个幂等方法，如下所示：

```js
let p = Promise.resolve(7)
setTimeout(console.log, 0, p === Promise.resolve(p)) // true
```



注意，这个静态方法能够包装任何非期约值，包括错误对象，并将其转换为解决的期约。

#### Promise.reject

与Promise.resolve()类似，Promise.reject()会实例化一个拒绝的期约并抛出一个异步错误（这个错误不能通过try/catch捕获，而只能通过拒绝处理程序捕获）。

Promise.reject()并没有照搬Promise.resolve()的幂等逻辑。如果给它传一个期约对象，则这个期约会成为它返回的拒绝期约的理由：

```js
setTimeout(console.log, 0, Promise.reject(Promise.solve()))
// Promise<rejected>: Promise<resolved>
```



注意⚠️

拒绝期约的错误并不会抛到执行同步代码的线程里，而是通过浏览器异步消息队列来处理的。因此，try/catch块并不能捕获reject抛出的错误。代码一旦开始以异步模式执行，则唯一与之交互的方式就是使用异步结构——更具体地说，就是期约的方法（then， catch）。



#### Promise.prototype.then()

Promise.prototype.then()是为期约实例添加处理程序的主要方法。这个then()方法接收最多两个参数：onResolved处理程序和onRejected处理程序。这两个参数都是可选的，如果提供的话，则会在期约分别进入“兑现”和“拒绝”状态时执行。

Promise.prototype.then()方法也会返回一个新的期约实例。这个新期约实例基于onResovled处理程序的返回值构建。

onRejected处理程序也与之类似：onRejected处理程序返回的值也会被Promise.resolve()包装。乍一看这可能有点违反直觉，但是想一想，onRejected处理程序的任务不就是捕获异步错误吗？因此，拒绝处理程序在捕获错误后不抛出异常是符合期约的行为，应该返回一个解决期约。

#### Promise.prototype.catch()

Promise.prototype.catch()方法用于给期约添加拒绝处理程序。这个方法只接收一个参数：onRejected处理程序。事实上，这个方法就是一个 `语法糖`，调用它就相当于调用Promise.prototype.then(null, onRejected)。

#### Promise.prototype.finally()

Promise.prototype.finally()方法用于给期约添加onFinally处理程序，这个处理程序在期约转换为解决或拒绝状态时都会执行。这个方法可以避免onResolved和onRejected处理程序中出现冗余代码。但onFinally处理程序没有办法知道期约的状态是解决还是拒绝，所以这个方法主要用于添加清理代码。

Promise.prototype.finally()方法返回一个新的期约实例，这个新期约实例不同于then()或catch()方式返回的实例。因为onFinally被设计为一个状态无关的方法，所以在大多数情况下它将表现为父期约的传递。对于已解决状态和被拒绝状态都是如此。



> ❗️new Promise 里的内容会立即执行，但 then 里面的内容会进入队列进行排队

#### 期约连锁与期约合成

每个期约实例的方法（then()、catch()和finally()）都会返回一个新的期约对象



### 异步函数 async/await

async关键字用于声明异步函数。这个关键字可以用在函数声明、函数表达式、箭头函数和方法上。

`使用async关键字可以让函数具有异步特征`，但总体上其代码仍然是同步求值的。而在参数或闭包方面，异步函数仍然具有普通JavaScript函数的正常行为。

不过，异步函数如果使用return关键字返回了值（如果没有return则会返回undefined），这个值会被Promise.resolve()包装成一个期约对象。异步函数始终返回期约对象。

与在期约处理程序中一样，在异步函数中抛出错误会返回拒绝的期约。不过，拒绝期约的错误不会被异步函数捕获。



await关键字会暂停执行异步函数后面的代码，让出JavaScript运行时的执行线程。这个行为与生成器函数中的yield关键字是一样的。await关键字同样是尝试“解包”对象的值，然后将这个值传给表达式，再异步恢复异步函数(async函数)的执行。

要完全理解await关键字，必须知道它并非只是等待一个值可用那么简单。JavaScript运行时在碰到await关键字时，会记录在哪里暂停执行。等到await右边的值可用了，JavaScript运行时会向消息队列中推送一个任务，这个任务会恢复异步函数的执行。

`异步函数可以暂停执行，而不阻塞主线程`。无论是编写基于期约的代码，还是组织串行或平行执行的异步代码，使用异步函数都非常得心应手。异步函数可以说是现代JavaScript工具箱中最重要的工具之一。

