## 背景

### 模块化的演进

随着互联网的深入发展，前端行业模块化的复杂度越来越高，导致我们在实现前端模块化的道路上不断受阻，但随着时间的推进，前端也出现了很多标准和工具来解决这些问题。以下是前端模块化的几个发展阶段：

1. 文件划分方式
2. 命名空间方式
3. IIFE
4. IIFE 依赖参数
5. 前端模块化规范的出现



随着 JavaScript 的标准逐渐走向完善，现在我们前端模块化已经发展到非常成熟的地步了，而且对前端模块化规范的最佳实践方式也基本实现了统一。

- 在 Node.js 环境中，我们遵循 CommonJS 规范
- 在浏览器环境中，我们遵循 ES Modules 规范

CommonJS 是属于内置模块系统，我们在 Node.js 环境中使用的时候，就不会有环境支持问题，只需要按照标准使用 require 和 module 即可。但是对于 ES Modules 规范来说，兼容的情况会相对差一些。因为 ES Modules 是 ECMAScript 2015（ES6）中才定义的模块系统，是近几年才制定的标准，所以肯定会存在环境兼容的问题。在这个标准刚推出的时候，几乎所有主流的浏览器都不支持。但是随着 Webpack 等一系列打包工具的流行，这一规范才开始逐渐被普及。

经过发展，ES Modules 现在已经成为最主流的前端模块化标准。相比于 AMD 这种社区提出的开发规范，ES Modules 是在语言层面实现的模块化，因此它的标准更为完善也更为合理。现在，绝大多数浏览器都已经开始原生支持 ES Modules 特性了。所以如今，我们需要重点学习的是如何在不同的环境中去更好的使用 ES Modules。



### 模块打包工具的出现

随着模块化思想的引入，前端的应用又会产生一些新的问题：

1.  ES Modules 模块系统的环境兼容问题。尽管现在主流浏览器的最新版本都支持这一特性，但是目前还无法保证用户的浏览器使用情况。所以我们还是需要解决兼容问题。
2. 过多且零散的模块文件导致浏览器的频繁发送网络请求。模块化的方式划分出来的模块文件过多，前端应用运行在浏览器中，每一个文件都需要单独从服务器请求回来，影响前端应用的工作效率。
3. 前端应用日益复杂，在前端应用开发过程中不仅仅只有 JavaScript 代码需要模块化，HTML 和 CSS 这些资源文件也会面临需要被模块化的问题。这些文件也都应该看作前端应用中的一个模块，只不过这些模块的种类和用途跟 JavaScript 不同。



所以我们需要模块化打包工具来帮我们解决上述的一些问题，一个好的模块打包工具需要具备以下基本功能：

1. 需要具备编译代码的能力，将开发阶段编写的那些包含新特性的代码转换为能够兼容大多数环境的代码，解决环境兼容问题。

2. 具备将散落的模块打包到一起的能力，解决浏览器频繁请求模块文件的问题。

   > 我们只在开发阶段才需要模块化的文件划分，因为它能够帮我们更好地组织代码，到了实际运行阶段，这种划分就没有必要了。

3. 具备支持不同种类的前端模块类型，可以将开发过程中涉及的样式、图片、字体等所有资源文件都作为模块使用，这样就能拥有一个统一的模块化方案。



所以如今，Webpack 可以看作是现代化前端应用的模块化管理工具，Webpack 以模块化思想为核心，帮助我们前端开发者更好的管理整个前端工程。



## 核心机制

Webpack 本质上是一个模块化打包工具，它通过“万物皆模块”这种设计思想（从官方文档的图可以看出），巧妙地实现了整个前端项目的模块化。

Webpack 的核心机制就两个：loader和plugin。loader 负责完成项目中各种各样资源模块的加载，从而实现整体项目的模块化，作用范围在模块的加载环节。plugin 是用来解决项目中除了资源模块打包以外的其他自动化工作。plugin 的能力范围更广，用途也更多。借助插件，我们就可以轻松实现前端工程化中绝大多数经常用到的功能。plugin的作用范围在每一个环节，我们通过往不同钩子环节上挂载不同的任务，就可以扩展 Webpack 的能力。



## 核心工作流程

### Webpack CLI 启动打包流程

源码在 [webpack-cli](https://github.com/webpack/webpack-cli) 模块中。为了增强 Webpack 本身的灵活性，所以 CLI 部分的代码从 Webpack 4 开始被单独抽出

webpack cli 的作用是将 CLI 参数和 Webpack 配置文件中的配置整合，得到一个完整的配置对象。

那么它是如何将配置整合的呢？

webpack cli 会通过 yargs 模块解析 CLI 参数，所谓 CLI 参数指的就是我们在运行 webpack 命令时通过命令行传入的参数，例如 --mode=production（如果出现重复的情况，会优先使用 CLI 参数）。

### 载入 Webpack 核心模块，创建 Compiler 对象

在 webpack cli 将配置项整合之后，我们会得到一个 options 配置项。

紧接着，我们会运行到 webpack 的核心模块，源码直接在 webpack 模块中。入口文件是 [lib/webpack.js](https://github.com/webpack/webpack/blob/v4.43.0/lib/webpack.js)。

这里我们首先会校验 webpack cli 传递过来的 options 参数是否符合要求，紧接着判断 options 的类型。这里传入的 options 不仅仅可以是一个对象，还可以是一个数组。



接下来我们通过传递的 options 配置项，来创建一个 `Compiler 对象`。

> 这个 Compiler 对象是整个 Webpack 工作过程中最核心的对象，负责完成整个项目的构建工作。



如果我们传入的 options 是一个数组，那么 Webpack 内部创建的就是一个 `MultiCompiler`，也就是说 Webpack 这时会支持同时开启**多路打包**，配置数组中的每一个成员就是一个独立的 webpack 配置选项：

```js
compiler = new MultiCompiler(
  Array.from(options).map(option => webpack(option))
)
```

如果我们传入的是一个普通的对象，就会按照我们最熟悉的方式创建一个 Compiler 对象，进行单线打包：

```js
compiler = webpack(options)
```



创建了 Compiler 对象过后，Webpack 开始注册我们 options 配置中的每一个插件。接下来 Webpack 工作过程的生命周期就要开始了，所以必须在这里将插件先注册，这样才能确保插件中的每一个钩子都能被命中。

Webpack 注册插件的方式如下：

```js
// 注册已配置的插件
if (options.plugins && Array.isArray(options.plugins)) {
  for (const plugin of options.plugins) {
    if (typeof plugin === 'function') {
      plugin.call(compiler, compiler)
    } else {
      plugin.apply(compiler)
    }
  }
}
```

注册完插件后，会触发两个钩子：environment, afterEnvironment。

然后创建内置的插件（后面 make 阶段会用到）：

```js
compiler.options = new WebpackOptionsApply().process(options, compiler)
```



### 使用 Compiler 对象开始编译整个项目

接下来 Webpack 会使用 Compiler 对象开始编译整个项目。



首先，会判断配置选项中是否启用了 watch 监视模式。如果是监视模式就调用 Compiler 对象的 watch 方法，以监视模式启动构建（这不属于我们关心的主线）。

如果不是监视模式，就调用 Compiler 对象的 run 方法，开始构建整个应用。

```js
if (firstOptions.watch || options.watch) {
  ...
} else {
  compiler.run((err, stats) => {
    ...
  })
}
```



这里是使用 compiler.run 方法进行构建，run 方法的具体文件在 webpack 模块下的 [lib/Compiler.js](https://github.com/webpack/webpack/blob/v4.43.0/lib/Compiler.js) 中。

run 方法中，会先触发 beforeRun 和 run 两个钩子，然后调用了当前对象的 this.compile 方法，真正开始编译整个项目（最关键的部分）。

this.compiler 方法内部主要创建了一个 Compilation 对象，也就是一次构建过程中的**上下文对象**，里面包含了这次构建中全部的资源和信息。创建完 Compilation 对象过后，会触发 make 钩子，进入整个构建过程最核心的 make 阶段。



### 从入口文件开始，解析模块依赖，形成依赖关系树

make 阶段会从入口文件开始，解析模块依赖，形成依赖关系树。然后将递归到的每个模块交给不同的 Loader 处理：

```js
this.hooks.make.callAsync(compilation, err => {
  if (err) return callback(err)

  compilation.finish(err => { ... })
})
```

make 阶段的调用过程比较特别。这个阶段并不会直接调用某个对象的某个方法，而是采用事件触发机制来触发执行的插件，所有之前监听了这个 make 事件的插件会在这时候开始执行。

Webpack 中有6个内置插件都监听了 make 事件。这些插件是前面创建 Compiler 对象的时候创建的。



我们默认使用的是单一入口打包的方式，所以这里最终会执行其中的 SingleEntryPlugin 插件。SingleEntryPlugin 插件中调用了 Compilation 对象的 addEntry 方法，**开始解析我们源代码中的入口文件**。addEntry 方法中又会调用 _addModuleChain 方法，将入口模块添加到模块依赖列表中，一层层的递进。



### 递归依赖树，将每个模块交给对应的 Loader 处理

上个步骤中，我们形成了依赖关系树，接下来 Webpack 会通过 Compilation 对象的 buildModule 方法进行模块构建，buildModule 方法中会执行**具体的 Loader**，处理特殊资源的加载。

build 完成过后，Webpack 通过 acorn 库生成模块代码的 AST 语法树。根据语法树分析这个模块是否还有依赖的模块，如果有则继续递归的循环 build 每一个依赖。



### 合并 Loader 处理完的结果，将打包结果输出到 dist 目录

到这里核心的流程基本已完成，所有的依赖解析完成，build 阶段结束。

最后 Webpack 会合并生成需要输出的 bundle.js 文件，并写入 dist 目录。



## 总结

以上就是 Webpack 运行的核心流程。Webpack 的源码比较复杂且繁多，如果直接阅读的话很难看懂，我们可以先理解核心流程，这些流程涉及到的源码文件都分散在 Webpack 的各个模块中，由此我们可以对单独的一个流程通过查阅对应源码的方式来深入理解 Webpack 的工作原理。

了解了核心流程的你，想必接下来会更有兴趣的去探讨其中的实现细节吧！

