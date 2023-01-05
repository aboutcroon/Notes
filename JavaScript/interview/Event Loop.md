## 异步

提起异步，相信每个人都知道，异步背后的“靠山”就是event loops。这里的异步准确的说应该叫浏览器的event loops或者说是javaScript运行环境的event loops，因为[ECMAScript](https://tc39.github.io/ecma262/)中没有event loops，event loops是在[HTML Standard](https://html.spec.whatwg.org/#event-loops)定义的。event loops规范中定义了浏览器何时进行渲染更新，了解它有助于性能优化。

首先看一个例子，思考例子中代码的执行顺序：

```js
console.log('start')

setTimeout( function () {
  console.log('setTimeout')
}, 0 )

Promise.resolve().then(function() {
  console.log('promise1')
}).then(function() {
  console.log('promise2')
})

console.log('end')

// start
// end
// promise1
// promise2
// setTimeout
```

上面的顺序是在chrome运行得出的，而在safari 9.1.2中测试时，promise1 promise2会在setTimeout的后面执行，但是在safari 10.0.1中得到了和chrome一样的结果。为何浏览器有不同的表现，了解tasks, microtasks队列就可以解答这个问题。



## event loop 的定义

event loop翻译出来就是**事件循环**，可以理解为实现异步的一种方式。我们看看[event loop](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop)在**HTML Standard**中的定义章节：

> 为了协调事件，用户交互，脚本，渲染，网络等，用户代理必须使用本节所述的`event loop`。

**事件，用户交互，脚本，渲染，网络**这些都是我们所熟悉的东西，他们都是由event loop协调的。触发一个`click`事件，进行一次`ajax`请求，背后都有`event loop`在运作。



知道了`event loops`的大致定义，我们再深入了解下`event loops`。

> 有两种event loops，一种在[浏览器上下文](https://html.spec.whatwg.org/multipage/browsers.html#browsing-context)，一种在[Web Workers](https://html.spec.whatwg.org/multipage/workers.html#workers)中。

> 每一个用户代理必须至少有一个浏览器上下文event loop，但是每个[单元的相似源浏览器上下文](https://html.spec.whatwg.org/multipage/browsers.html#unit-of-related-similar-origin-browsing-contexts)至多有一个event loop。

> event loop 总是具有至少一个浏览器上下文，当一个event loop的浏览器上下文全都销毁的时候，event loop也会销毁。一个浏览器上下文总有一个event loop去协调它的活动。

> Web Workers的event loop相对简单一些，一个worker对应一个event loop，[worker进程模型](https://html.spec.whatwg.org/multipage/workers.html#run-a-worker)管理event loop的生命周期。

这里反复提到的一个词是[browsing contexts](https://html.spec.whatwg.org/multipage/browsers.html#browsing-context)（浏览器上下文）。

> **浏览器上下文**是一个将 Document 对象呈现给用户的环境。在一个 Web 浏览器内，一个标签页或窗口常包含一个浏览上下文，如一个 [iframe](https://html.spec.whatwg.org/multipage/embedded-content.html#the-iframe-element) 或一个 [frameset](https://html.spec.whatwg.org/multipage/obsolete.html#frameset) 内的若干 frame。



对于这些资料阐述的event loop做个总结：

- 每个线程都有自己的`event loop`。
- 浏览器可以有多个`event loop`，`browsing contexts`和`web workers`就是相互独立的。
- 所有同源的`browsing contexts`可以共用`event loop`，这样它们之间就可以相互通信。



## task

**介绍：**

> 一个event loop有一个或者多个task队列。

> 当用户代理安排一个任务，必须将该任务增加到相应的event loop的一个tsak队列中。

> 每一个task都来源于指定的任务源，比如可以为鼠标、键盘事件提供一个task队列，其他事件又是一个单独的队列。可以为鼠标、键盘事件分配更多的时间，保证交互的流畅。

task队列就是一个先进先出的队列，由指定的任务源去提供任务。

**哪些是task任务源呢？**

[Generic task sources](https://html.spec.whatwg.org/multipage/webappapis.html#generic-task-sources)中有提及：

> **DOM操作任务源：**
> 此任务源被用来响应dom操作，例如一个元素以非阻塞的方式[插入文档](https://html.spec.whatwg.org/multipage/infrastructure.html#insert-an-element-into-a-document)。

> **用户交互任务源：**
> 此任务源用于对用户交互作出反应，例如键盘或鼠标输入。响应用户操作的事件（例如[click](https://w3c.github.io/uievents/#event-type-click)）必须使用task队列。

> **网络任务源：**
> 网络任务源被用来响应网络活动。

> **history traversal任务源：**
> 当调用history.back()等类似的api时，将任务插进task队列。

task任务源非常宽泛，比如`ajax`的`onload`，`click`事件，基本上我们经常绑定的各种事件都是task任务源，还有数据库操作（IndexedDB ），需要注意的是`setTimeout`、`setInterval`、`setImmediate`也是task任务源。总结来说task任务源：

- setTimeout
- setInterval
- setImmediate
- I/O
- UI rendering



## microtask

**介绍：**

> 每一个event loop都有一个microtask队列，一个microtask会被排进microtask队列而不是task队列。

> 有两种microtasks：分别是solitary callback microtasks和compound microtasks。规范值只覆盖solitary callback microtasks。

> 如果在初期执行时，[spin the event loop](https://html.spec.whatwg.org/multipage/webappapis.html#spin-the-event-loop)，microtasks有可能被移动到常规的task队列，在这种情况下，microtasks任务源会被task任务源所用。通常情况，task任务源和microtasks是不相关的。

microtask 队列和task 队列有些相似，都是先进先出的队列，由指定的任务源去提供任务。

不同的是：`一个event loop里只有一个microtask 队列。`

**HTML Standard**没有具体指明哪些是microtask任务源，通常认为是microtask任务源有：

- process.nextTick
- promises
- Object.observe
- MutationObserver



**NOTES:**

1. Promise的定义在 ECMAScript规范而不是在HTML规范中，但是ECMAScript规范中有一个[jobs](http://ecma-international.org/ecma-262/6.0/index.html#sec-jobs-and-job-queues)的概念和microtasks很相似。**[在Promises/A+规范的Notes 3.1](https://promisesaplus.com/#notes)中提及了promise的then方法可以采用“宏任务（macro-task）”机制或者“微任务（micro-task）”机制来实现**。所以开头提及的promise在不同浏览器的差异正源于此。

2. 有的浏览器将`then`放入了macro-task队列，有的放入了micro-task 队列。一个普遍的共识是promises属于microtasks队列。比如一篇博文[Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)中提及了一个讨论[vague mailing list discussions](https://esdiscuss.org/topic/the-initialization-steps-for-web-browsers#content-16)。



## event loop的处理过程（Processing model）

在**HTML Standard**的[Processing model](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model)中定义了`event loop`的循环过程：

> 一个event loop只要存在，就会不断执行下边的步骤：
> 1.在tasks队列中选择最老的一个task,用户代理可以选择任何task队列，如果没有可选的任务，则跳到下边的microtasks步骤。
> 2.将上边选择的task设置为[正在运行的task](https://html.spec.whatwg.org/multipage/webappapis.html#currently-running-task)。
> 3.Run: 运行被选择的task。
> 4.将event loop的[currently running task](https://html.spec.whatwg.org/multipage/webappapis.html#currently-running-task)变为null。
> 5.从task队列里移除前边运行的task。
> 6.Microtasks: 执行[microtasks任务检查点](https://html.spec.whatwg.org/multipage/webappapis.html#perform-a-microtask-checkpoint)。（也就是执行microtasks队列里的任务）
> 7.更新渲染（Update the rendering）...
> 8.如果这是一个worker event loop，但是没有任务在task队列中，并且[WorkerGlobalScope](https://html.spec.whatwg.org/multipage/workers.html#workerglobalscope)对象的closing标识为true，则销毁event loop，中止这些步骤，然后进行定义在[Web workers](https://html.spec.whatwg.org/multipage/workers.html#workers)章节的[run a worker](https://html.spec.whatwg.org/multipage/workers.html#run-a-worker)。
> 9.返回到第一步。

event loop会不断循环上面的步骤，概括说来：

- `event loop`会不断循环的去取`tasks`队列的中最老的一个任务推入栈中执行，并在当次循环里依次执行并清空`microtask`队列里的任务。
- 执行完`microtask`队列里的任务，有**可能**会渲染更新。（浏览器很聪明，在一帧以内的多次dom变动浏览器不会立即响应，而是会积攒变动以最高60HZ的频率更新视图）



在当次循环里依次执行并清空`microtask`队列里的任务，这句话所带来的现象就是：先会执行最外层task（script），然后将script里面的任务都推入栈，这些任务里，Microtasks 会先执行，task 后执行。



## microtask的检查点（microtask checkpoint）

`event loop`运行的第6步，执行了一个`microtask checkpoint`，下面是**HTML Standard**描述的`microtask checkpoint`：

> 当用户代理去执行一个microtask checkpoint，如果microtask checkpoint的flag（标识）为false，用户代理必须运行下面的步骤：
> 1.将microtask checkpoint的flag设为true。
> 2.Microtask queue handling: 如果event loop的microtask队列为空，直接跳到第八步（Done）。
> 3.在microtask队列中选择最老的一个任务。
> 4.将上一步选择的任务设为event loop的[currently running task](https://html.spec.whatwg.org/multipage/webappapis.html#currently-running-task)。
> 5.运行选择的任务。
> 6.将event loop的[currently running task](https://html.spec.whatwg.org/multipage/webappapis.html#currently-running-task)变为null。
> 7.将前面运行的microtask从microtask队列中删除，然后返回到第二步（Microtask queue handling）。
> 8.Done: 每一个[environment settings object](https://html.spec.whatwg.org/multipage/webappapis.html#environment-settings-object)它们的 [responsible event loop](https://html.spec.whatwg.org/multipage/webappapis.html#responsible-event-loop)就是当前的event loop，会给environment settings object发一个[ rejected promises ](https://html.spec.whatwg.org/multipage/webappapis.html#notify-about-rejected-promises)的通知。
> 9.[清理IndexedDB的事务](https://w3c.github.io/IndexedDB/#cleanup-indexed-database-transactions)。
> 10.将microtask checkpoint的flag设为flase。



由此可得，`microtask checkpoint`所做的就是执行microtask队列里的任务。那什么时候会调用`microtask checkpoint`呢?

- [当上下文执行栈为空时，执行一个microtask checkpoint。](https://html.spec.whatwg.org/multipage/webappapis.html#clean-up-after-running-a-callback)
- 在event loop的第六步（Microtasks: Perform a microtask checkpoint）执行checkpoint，也就是在运行task之后，更新渲染之前。



## 执行栈（JavaScript execution context stack）

task和microtask都是推入栈中执行的。

javaScript是单线程，也就是说只有一个主线程，主线程有一个栈，每一个函数执行的时候，都会生成新的`execution context（执行上下文）`，执行上下文会包含一些当前函数的参数、局部变量之类的信息，它会被推入栈中，[ running execution context（正在执行的上下文）](https://tc39.github.io/ecma262/#running-execution-context)始终处于栈的顶部。当函数执行完后，它的执行上下文会从栈弹出。

![execution context stack](/Users/liuchang/Documents/markdown/面试题/JS/assets/execution context stack.png)



## 完整异步过程

规范晦涩难懂，做一个形象的比喻：
主线程类似一个加工厂，它只有一条流水线，待执行的任务就是流水线上的原料，只有前一个加工完，后一个才能进行。event loops就是把原料放上流水线的工人。只要已经放在流水线上的，它们会被依次处理，称为**同步任务**。一些待处理的原料，工人会按照它们的种类排序，在适当的时机放上流水线，这些称为**异步任务**。

过程图：

![event loop](/Users/liuchang/Documents/markdown/面试题/JS/assets/event loop.png)

举个简单的例子，假设一个script标签的代码如下：

```js
Promise.resolve().then(function promise1 () {
       console.log('promise1');
    })
setTimeout(function setTimeout1 (){
    console.log('setTimeout1')
    Promise.resolve().then(function  promise2 () {
       console.log('promise2');
    })
}, 0)

setTimeout(function setTimeout2 (){
   console.log('setTimeout2')
}, 0)
```

`运行过程`：

首先script里的代码被列为一个task，放入task队列。

**循环1：**

- 【task队列：script ；microtask队列：】

1. 从task队列中取出script任务，推入栈中执行。
2. promise1列为microtask，setTimeout1列为task，setTimeout2列为task。

- 【task队列：setTimeout1 setTimeout2；microtask队列：promise1】

1. script任务执行完毕，执行microtask checkpoint，取出microtask队列的promise1执行。

**循环2：**

- 【task队列：setTimeout1 setTimeout2；microtask队列：】

1. 从task队列中取出setTimeout1，推入栈中执行，将promise2列为microtask。

- 【task队列：setTimeout2；microtask队列：promise2】

1. 执行microtask checkpoint，取出microtask队列的promise2执行。

**循环3：**

- 【task队列：setTimeout2；microtask队列：】

1. 从task队列中取出setTimeout2，推入栈中执行。
2. setTimeout2任务执行完毕，执行microtask checkpoint。

- 【task队列：；microtask队列：】



## event loop中的Update the rendering（更新渲染）

渲染的基本流程：

1. 处理 HTML 标记并构建 DOM 树。
2. 处理 CSS 标记并构建 CSSOM 树， 将 DOM 与 CSSOM 合并成一个渲染树。
3. 根据渲染树来布局，以计算每个节点的几何信息。
4. 将各个节点绘制到屏幕上

**Note: 可以看到渲染树的一个重要组成部分是CSSOM树，绘制会等待css样式全部加载完成才进行，所以css样式加载的快慢是首屏呈现快慢的关键点。**



更新渲染（Update the rendering）的时机：

- 在一轮event loop中多次修改同一dom，只有最后一次会进行绘制。
- 渲染更新（Update the rendering）会在event loop中的tasks和microtasks完成后进行，但并不是每轮event loop都会更新渲染，这取决于是否修改了dom和浏览器觉得是否有必要在此时立即将新状态呈现给用户。如果在一帧的时间内（时间并不确定，因为浏览器每秒的帧数总在波动，16.7ms只是估算并不准确）修改了多处dom，浏览器可能将变动积攒起来，只进行一次绘制，这是合理的。
- 如果希望在每轮event loop都即时呈现变动，可以使用requestAnimationFrame。



## 应用

event loop的大致循环过程，可以用下边的图表示：

![loop1](/Users/liuchang/Documents/markdown/面试题/JS/assets/loop1.png)

假设现在执行到currently running task，我们对批量的dom进行异步修改，我们将此任务插进task：

![loop2](/Users/liuchang/Documents/markdown/面试题/JS/assets/loop2.png)

此任务插进microtasks：

![loop3](/Users/liuchang/Documents/markdown/面试题/JS/assets/loop3.png)

可以看到如果task队列如果有大量的任务等待执行时，将dom的变动作为microtasks而不是task能更快的将变化呈现给用户。



**同步简简单单就可以完成了，为啥要异步去做这些事？**

对于一些简单的场景，同步完全可以胜任，如果得对dom反复修改或者进行大量计算时，使用异步可以作为缓冲，优化性能。

例子：

```html
<div id='result'>this is result</div>
```

有一个计算平方的函数，并且会将结果响应到对应的元素

```js
function bar (num, id) {
  const  product  = num  * num
  const resultEle = document.getElementById( id )
  resultEle.textContent = product
}
```

现在我们制造些问题，假设现在很多同步函数引用了bar，在一轮event loop里，可能bar会被调用多次，并且其中有几个是对id='result'的元素进行操作。就像下边一样：

```js
...
bar( 2, 'result' )
...
bar( 4, 'result' )
...
bar( 5, 'result' )
...
```

似乎这样的问题也不大，但是当计算变得复杂，操作很多dom的时候，这个问题就不容忽视了。

用我们上边讲的event loop知识，修改一下bar：

```js
const store = {}
let flag = false
function bar (num, id) {
  store[id] = num
  if(!flag){
    Promise.resolve().then(function () {
       for(let k in store){
         const num = store[k]
         const product  = num * num
         const resultEle = document.getElementById(k)
         resultEle.textContent = product
       }
    })
    flag = true
  }
}
```

现在我们用一个store去存储参数，统一在microtasks阶段执行，过滤了多余的计算，即使同步过程中多次对一个元素修改，也只会响应最后一次。



## 参考文章

https://github.com/aooy/blog/issues/5

https://html.spec.whatwg.org/multipage/webappapis.html#event-loop



