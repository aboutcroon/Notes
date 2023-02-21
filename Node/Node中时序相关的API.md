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

一个 `setTimeout` 可以是一个 Timer 事件，一个文件读写是一个系统的 I/O 事件。对于浏览器来说，通常是没有后者的。而处理系统 I/O 事件，各系统平台早在多年前就有了成熟的方案了。如 Linux 的 epoll、macOS 的 kqueue、Windows 的 IOCP 等，这些统称为 I/O 多路复用。



Node.js 的事件循环基石就基于此。它**基于 Ryan Dahl 自己开发的** **libuv** **，完成了自己的事件循环与异步** **I/O**。



libuv 是一个聚焦异步 I/O 的跨平台库。它就是为 Node.js 而生的，后续才衍生出对其他项目的支持，如 [Luvit](https://link.juejin.cn/?target=https%3A%2F%2Fluvit.io%2F)、[Julia](https://link.juejin.cn/?target=https%3A%2F%2Fjulialang.org%2F)、[uvloop](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FMagicStack%2Fuvloop) 等（[更多](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Flibuv%2Flibuv%2Fblob%2Fv1.x%2FLINKS.md)）。

libuv 官网文档上“设计概览”页中的流程图。

![event I/O](https://raw.githubusercontent.com/edwineo/Notes/main/Node/assets/event%20I%3AO.png)