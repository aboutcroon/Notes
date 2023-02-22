## Node.js 介绍

Node.js 的官网介绍

```
Node.js 是一个开源的、跨平台的 JavaScript 运行时环境。
```

所以，Node.js 并不是一门语言，而是一个 **JavaScript 运行时环境**。实际上它的语言是 JavaScript

> 这就跟 PHP、Python、Ruby 这类不一样，它们既代表语言，也可代表执行它们的运行时环境（或解释器）。



Node.js 与 JavaScript 真要按分层说，大概是这么个意思，如下图：

![node-hierarchy](https://raw.githubusercontent.com/edwineo/Notes/main/Node/assets/node-hierarchy.png)

这里我们从下往上梳理：

- 最下面一层是**脚本语言规范（Spec）** ，由于我们讲的是 Node.js，所以这里最下层只写 ECMAScript。
- 再往上一层就是**对于该规范的实现**了，如 JavaScript、JScript 以及 ActionScript 等都属于对 ECMAScript 的实现。
- 然后就是**执行引擎**，JavaScript 常见的引擎有 V8、SpiderMonkey、QuickJS 等。
- 最上面就是**运行时环境**了，比如基于 V8 封装的运行时环境有 Chromium、Node.js、Deno、CloudFlare Workers 等等。而我们所说的 Node.js 就是在运行时环境这一层。



## Node.js 的事件循环与 I/O

在 MDN 里面，有这么一段话：

> JavaScript 有一个基于事件循环的并发模型，事件循环负责执行代码、收集和处理事件以及执行队列中的子任务。这个模型与其它语言中的模型截然不同，比如 C 和 Java。

但是，上面那段话讲的是 JavaScript，不是 Node.js。

JavaScript 的确是有一个基于事件循环的并发模型。但这以前是针对前端在 DOM 中进行操作的。V8 自己有实现一套事件循环，但 **Node.js 中的事件循环则是自己实现的**。

与浏览器不同，作为能跑后端服务的 JavaScript 运行时，必须要有处理系统事件的能力。比如去处理各种文件描述符对应的读写事件。一个最简单的事件循环的伪代码可如下：

```js
while (还有事件在监听) {
  const events = 从监听中获取所有事件信息;
  for (const event of events) {
    处理(event);
  }
}
```

一个 `setTimeout` 是一个 Timer 事件，一个文件读写是一个系统的 I/O 事件。对于浏览器来说，通常是没有后者的。而处理系统 I/O 事件，各系统平台早在多年前就有了成熟的方案了。如 Linux 的 epoll、macOS 的 kqueue、Windows 的 IOCP 等，这些统称为 I/O 多路复用。



Node.js 的事件循环基石就基于此。它**基于 Ryan Dahl 自己开发的** **libuv** **，完成了自己的事件循环与异步** **I/O**。



libuv 是一个聚焦异步 I/O 的跨平台库。它就是为 Node.js 而生的，后续才衍生出对其他项目的支持，如 [Luvit](https://link.juejin.cn/?target=https%3A%2F%2Fluvit.io%2F)、[Julia](https://link.juejin.cn/?target=https%3A%2F%2Fjulialang.org%2F)、[uvloop](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FMagicStack%2Fuvloop) 等（[更多](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Flibuv%2Flibuv%2Fblob%2Fv1.x%2FLINKS.md)）。

以下是 libuv 官网文档上“设计概览”页中的流程图。

![loop_iteration](https://raw.githubusercontent.com/edwineo/Notes/main/Node/assets/loop_iteration.png)

在 libuv 中，一次小循环的执行顺序分别是：

1. 定时器；
2. Pending 态的 I/O 事件；
3. 空转事件；
4. 准备事件；
5. Poll I/O 事件；
6. 复查事件；
7. 扫尾。

定时器和 I/O 事件大家比较耳熟能详，如 `setTimeout()` 等就是依靠定时器，网络请求等就是依靠 I/O 事件。

所以，在 Node.js 中，异步体现在方方面面。不一定所有异步都与 I/O 有关。`Timer` 相关的 API 就是与 I/O 无关的异步 API。

一个事件循环除了 `Poll for I/O` 之外，前前后后还包了好几层。所谓的 `Timer` 就在 `Run due timers` 这层。而 `setImmediate()`、`process.nextTick()` 这些则分别在其他几层。



## Node.js 中的 setTimeout

### setTimeout 到底是哪的函数？怎么实现的？

很多人可能都有一种误解，一个 JavaScript 运行时天生就应该有 `setTimeout` 这些函数。因为它实在是太常用了，在 Web 应用中真的是无处不在，在各种跨端场景中，也是最基础的 API 之一。Node.js、Deno 这些运行时都有。**它就是引擎中提供的 API！**

**但你要这么想就错了。** `setTimeout` 并非 ECMAScript 规范，而是 [Web API](https://link.juejin.cn/?target=https%3A%2F%2Fhtml.spec.whatwg.org%2Fmultipage%2Ftimers-and-user-prompts.html%23dom-settimeout-dev)。之所以 Node.js、Deno 这些都有，是因为这个 API 太深入人心了，不实现不舒服。我们可以粗暴地理解为，ECMAScript 不定义穿插于事件循环不同 Tick 间的 API。



所以，setTimeout 到底是怎么实现的？其实，setTimeout 在不同运行时的实现机制是不一样的。

接下来我们介绍 Node.js 中的 setTimeout

Node.js 的 setTimeout 并不完全按规范来实现。比如 setTimeout 的返回值 `id` 在规范中，是一个整数（`integer`），而 Node.js 实际上返回的是一个对象。还有一个就是网上常有的八股问题，如果嵌套层级大于 5，超时时间小于 4，则定义超时时间最小为 4——这个八股问题对 Node.js 同样不生效。

在 Node.js 中，setTimeout 实际上是生成一个 Timeout 类的实例，在其内部控制定时器并触发回调，并且这个函数返回的也是该实例，而并不是整数。

```js
function setTimeout(callback, after, arg1, arg2, arg3) {
  // ...
  const timeout = new Timeout(callback, after, args, false, true);
  insert(timeout, timeout._idleTimeout);
  return timeout;
}
```

这里有几个点好提。第一个是 `args` 的生成。我们看到 Node.js 声明 `setTimeout` 的时候，其参数除了前两个外，后面还额外附加了 `arg1`、`arg2` 和 `arg3`。这个是根据经验定的 3 个调用参数，基本上的情况下，调用 `setTimeout()` 回调函数的参数不会超过 3 个，于是这里显示声明 3 个，为生成 `args` 用。

```js
let i, args;
switch (arguments.length) {
  // fast cases
  case 1:
  case 2:
    break;
  case 3:
    args = [arg1];
    break;
  case 4:
    args = [arg1, arg2];
    break;
  default:
    args = [arg1, arg2, arg3];
    for (i = 5; i < arguments.length; i++) {
      // Extend array dynamically, makes .apply run much faster in v6.0.0
      args[i - 2] = arguments[i];
    }
    break;
}
```

如果刚好是 3 个参数以内的，在生成 `args` 的时候会直接声明一个定长数组，这比后续不断去扩展数组来得快。如果再超长了，那再去动态扩展 `args` 数组长度。



### 两个关键对象：Map 与优先队列

1. 一个 `Map`，其键名为 `timeout` 的值，键值为一条存储同 `timeout` 值的所有 `Timeout` 实例的链表，链表中额外存了一个最近的最终超时时间，用于做一些判断；
2. 一个[优先队列](https://link.juejin.cn/?target=https%3A%2F%2Fzh.wikipedia.org%2Fzh-hans%2F%E5%84%AA%E5%85%88%E4%BD%87%E5%88%97)，以链表的最终超时时间点为优先队列的权重。



这两个对象看起来像这样

![map queue](https://raw.githubusercontent.com/edwineo/Notes/main/Node/assets/map_queue.png)

从图中看，这是一个拥有 10 个 `Timeout` 的场景的两个对象表示图。10ms 定时器的最近过期时间为 1676275438020，64ms 的为 1676275438027，而 100ms 的则为 1676275438011。这个时候，优先队列中，以最近超时为权重，自然就是 100ms 的链表、10ms 链表、64ms 链表这个顺序了。



