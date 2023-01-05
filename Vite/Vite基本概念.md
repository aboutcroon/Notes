# Vite 介绍

## 组成

引用 vite 官方文档中介绍

> vite 是一种新型前端构建工具，能够显著提升前端开发体验。它主要由两部分组成：
>
> - 一个开发服务器，它基于 [原生 ES 模块](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) 提供了 [丰富的内建功能](https://cn.vitejs.dev/guide/features.html)，如速度快到惊人的 [模块热更新（HMR）](https://cn.vitejs.dev/guide/features.html#hot-module-replacement)。
> - 一套构建指令，它使用 [Rollup](https://rollupjs.org/) 打包你的代码，并且它是预配置的，可输出用于生产环境的高度优化过的静态资源。



可见，vite 主要是用于开发环境，提高我们在开发环境构建项目的速度，vite 在开发环境下使用 [esbuild](https://esbuild.github.io/)，这个库在开发环境时进行文件编译，这个工具的性能比传统的 webpack，rollup 性能要高很多。

而在生产环境下，vite 使用 rollup 进行打包，所以其在生产环境下就相当于是 rollup。



## 旨在解决的现实问题

引用 vite 官方文档中介绍

> 在浏览器支持 ES 模块之前，JavaScript 并没有提供的原生机制让开发者以模块化的方式进行开发。这也正是我们对 “打包” 这个概念熟悉的原因：使用工具抓取、处理并将我们的源码模块串联成可以在浏览器中运行的文件。
>
> 时过境迁，我们见证了诸如 [webpack](https://webpack.js.org/)、[Rollup](https://rollupjs.org/) 和 [Parcel](https://parceljs.org/) 等工具的变迁，它们极大地改善了前端开发者的开发体验。
>
> 然而，当我们开始构建越来越大型的应用时，需要处理的 JavaScript 代码量也呈指数级增长。包含数千个模块的大型项目相当普遍。我们开始遇到性能瓶颈 —— 使用 JavaScript 开发的工具通常需要很长时间（甚至是几分钟！）才能启动开发服务器，即使使用 HMR，文件修改后的效果也需要几秒钟才能在浏览器中反映出来。如此循环往复，迟钝的反馈会极大地影响开发者的开发效率和幸福感。
>
> Vite 旨在利用生态系统中的新进展解决上述问题：浏览器开始原生支持 ES 模块，且越来越多 JavaScript 工具使用编译型语言编写。



vite 旨在解决项目在开发环境下构建速度慢的问题，并且使用 ESM 模块方式来将其改进

可见，vite 非常的快，是得益于它整体的架构和 ES6 的速度



## 比较

参考官方文档：[与其他非打包解决方案的比较](https://cn.vitejs.dev/guide/comparisons.html#comparisons-with-other-no-bundler-solutions)

### Snowpack

开发时的构建和 vite 非常像，但 vite 生产时的构建是直接用 rollup 进行打包的（与 Vue cli 进行打包的结果差不多）；但 snowpack 生产时的打包与其开发时是一样的，这会导致浏览器兼容性不好，因为开发时使用兼容性略低的方案是没有关系的，但生产环境要在不同的浏览器上，要较大的考虑到兼容性。

### rollup

rollup是一个更加专一的工具

rollup 诞生的目的就是为了 build ESM modules，它是只专注于 build Javascript，而不关注平台的这么一个工具

对于 webpack 来说，它编译出来的代码会有很多 webpack 的工具函数，来帮助我们加载模块。但 rollup 不会有其专有的函数在里面，它只是遵循 ESM 规范来 build



## 与传统构建工具的区别（webpack，rollup）

webpack，rollup 的概念是一种工具，而 vite 更上一层，它是用于帮助我们更好更快的开发一个项目。

webpack 全面，rollup 专一，vite 好用易上手。

vite 是为项目而生的，而不是为构建而生的，vite 是一个更加集成化的前端构建工具（前端构建平台）

### High level api

### 不包含编译能力

vite 本身不包含编译能力，它的编译能力是源自于 esbuild 和 rollup，它只是集成了 rollup 的功能，然后启动了 dev-server，在中间进行串联和管理

### 完全基于ESM加载方式来开发的

### 相比较之下减少的工作量

- 集成了 dev-server（主要功能）
- 各类 loader（在 webpack 中我们要配置 css loader, babel loader, ts loader 等），让我们可以直接 import 一个 css 就可以用了
- 内置了 build 命令，让我们可以去 build 一个类库或者一个项目，直接通过 `vite build` 去 build，webpack 则需要自己去写这么一个命令，开发和生产的配置也会有所不一样。（vite build时，使用的就是 rollup，所以和 webpack build时差不多）



## vite 的优势

vite 更应该是与 vue cli，create-react-app 这样的集成了 webpack 的工具去做对比

优点：

1. vite 没有那么多复杂晦涩的配置，上手简单，开发效率极高，社区成本低（兼容 rollup 插件，vite 天生就支持所有的 rollup 插件，采用 rollup 插件的使用格式，对理解插件，建立一个插件更简单）
2. vite 有自身的插件系统
3. 依赖预构建，将AMD，UMD，IIFE，commonjs 等转成 es module 形式，对依赖进行强缓存，并缓存在node_module/.vite下，加快服务构建。
4. 提供基于 ESM 的 HMR API，比传统的 HMR 过程更简单，反映更迅速。
5. 使用 esbuild 转译 ts，tsx，jsx，转译迅速，约是 tsc 速度的 20~30 倍。
6. 内部构建了 css 预处理器，tsx，jsx，postCss，css Module 的支持，不用用户配置，开箱即用。
7. 不同于传统打包方式，传统打包方式启动必须优先抓取并构建你的整个应用，然后才能提供服务。vite 则直接请求源码，返回过程中解析转化源码，而模块划分工作由浏览器提供。



## 构建方式

引用官方文档的话

> Vite 通过在一开始将应用中的模块区分为 **依赖** 和 **源码** 两类，改进了开发服务器启动时间。
>
> - **依赖** 大多为在开发时不会变动的纯 JavaScript。一些较大的依赖（例如有上百个模块的组件库）处理的代价也很高。依赖也通常会存在多种模块化格式（例如 ESM 或者 CommonJS）。
>
>   Vite 将会使用 [esbuild](https://esbuild.github.io/) [预构建依赖](https://cn.vitejs.dev/guide/dep-pre-bundling.html)。Esbuild 使用 Go 编写，并且比以 JavaScript 编写的打包器预构建依赖快 10-100 倍。
>
> - **源码** 通常包含一些并非直接是 JavaScript 的文件，需要转换（例如 JSX，CSS 或者 Vue/Svelte 组件），时常会被编辑。同时，并不是所有的源码都需要同时被加载（例如基于路由拆分的代码模块）。
>
>   Vite 以 [原生 ESM](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) 方式提供源码。这实际上是让浏览器接管了打包程序的部分工作：Vite 只需要在浏览器请求源码时进行转换并按需提供源码。根据情景动态导入代码，即只在当前屏幕上实际使用时才会被处理。



依赖就是我们项目中使用到的第三方库，源码则是我们项目中自己写的文件代码



传统的构建工具如下图

![bundle based](https://raw.githubusercontent.com/aboutcroon/Notes/main/Vite/assets/bundle%20based.png)

从入口开始，将文件编译，最终打包成一个 bundle，每增加多一个文件，速度就会慢一点。



对于 vite，如下图

![native ESM based](https://raw.githubusercontent.com/aboutcroon/Notes/main/Vite/assets/native%20ESM%20based.png)

vite 速度快的原因有两大点：

原因1：它启动的时候就没有去做编译，它只是做了一些预编译（一些需要去提前编译的文件才做了一些编译），初次访问时只会编译首页出现的内容，直到我们进入下一个页面时，才会要要用到的模块继续编译，所以是一个实时编译的过程

原因2：vite 使用到了 [esbuild](https://esbuild.github.io/) 工具，这个工具在开发环境进行文件编译，这个工具的性能比传统的 webpack rollup 性能要高很多

# Vite 基础应用

## vite 创建 vue3 项目

vite 1.0 版本是以 vue3 为主要的构建对象的

半年之后，vite 更新到 2.0 版本，是一个跨框架的版本，内部不包括任何框架相关的内容，它通过 `插件` 的方式来提供不同的前端框架的开发编译功能

### 创建项目

```
npm init vite@latest
```

创建项目后，可以看到它只给了我们一个目录结构，并没有 `npm install`

public 目录下存放一些静态文件，图片等不需要编译的内容，vite 提供了路径映射，可以让我们更方便的去 import

![vite vue3](https://raw.githubusercontent.com/aboutcroon/Notes/main/Vite/assets/vite%20vue3.png)

webpack 和 rollup 的编译入口是一个 js 文件，vite 的编译入口是一个 html 文件，因为 vite 一开始不编译 main.js 那些文件，它是让浏览器去加载 index.html 文件，[index.html 文件位于最外层](https://cn.vitejs.dev/guide/#index-html-and-project-root)，通过 script 标签，就会去加载 src 下面的 main.js 文件，然后 vite 才会对 main.js 去进行编译。这就是 vite 以一个 html 文件作为入口的最大因素所在



## vite 创建 vue2 项目

通过使用插件的方式来支持 vue2 项目，详情可参考 [awesome-vite](https://github.com/vitejs/awesome-vite#plugins) 中的 vue2 模版 **[ vite-vue2-windicss-starter](https://github.com/lstoeferle/vite-vue2-windicss-starter)**，其中是使用到了 [vite-plugin-vue2](https://github.com/underfin/vite-plugin-vue2) 这个插件



## vite 中使用 Typescript

vite 天生就已经支持 ts 了，在开发环境中 vite 使用的是 ES6，ES6 本身就是支持 ts语法的，但是 vite 只会将 ts 编译成 js 让我们在浏览器环境去运行它，但不会去做校验。

所以我们需要自己用 tsc 进行校验，下载 Typescript，添加 `tsconfig.json` 文件。

如果我们要校验 `.vue` 文件里的 ts，那么就要使用 vue-tsc for SFC 的依赖



package.json 文件

```json
"build": "vue-tsc --noEmit && tsc --noEmit && vite build"
```



## tsconfig.json

里面的配置项

```json
{
  "compilerOptions": {
  	"target": "esnext",
    "module": "esnext", // 使用ES Module 这是必须的
    "jsx": "preserve", // 这个 jsx 如果进行检验的话，都是遵循 react 规范的而不遵循其他插件规则的，所以这里用 preserve 让它不进行检验，而是留给 vite 插件 @vitejs/plugin-vue-jsx 来校验
    "sourceMap": true, // 开启 sourceMap，可以让我们直接在浏览器中看到 ts 的代码，调试起来更方便
    "esModuleInterop": true // 方便我们去做一些 import，如果没开启的话，就要 import * as react from 'react'，如果开启了之后，就可以 import react from 'react'
    "lib": ["esnext", "dom"], // 有了 dom，我们就能识别当前浏览器既定的 dom 类，作为 global
		"isolatedModules": true,
  	"types": ["vite/client"]
	},
	"include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx"] // 让 ts 知道要编译那哪些文件夹下的哪些文件来作为 ts 的目标源
}
```

### [types: ["vite/client"]](https://cn.vitejs.dev/guide/features.html#client-types)

告诉我们 vite 内部支持的一些内部变量的类型

- Asset imports

  告诉我们 import 一个静态文件的返回类型，比如 import 一个 png，返回类型是 string

- env

  环境变量

- HMR api

  hot相关的 api

### [IsolatedModules](https://cn.vitejs.dev/guide/features.html#typescript-compiler-options)

在 vite 中开启这个项可以帮助我们在开发过程中就能识别的三个 vite 在运行后会产生的问题，并且在运行 tsc 校验的时候也能校验得到

#### Exports of Non-Value Identifiers

```ts
type A = {
  name: string
}
```

```ts
import { A } from './types'

export { A }
```

当我们在文件中引入另一个文件中的 A 类型，然后再 export 出去时，会帮助我们提示错误。因为 vite 将 ts 编译成 js 后，我们引入的 A 类型就不存在了，因为 js 中不存在类型，所以他并不是一个实体值。

#### Non-Module Files

我们的文件必须要有 module 这个概念，每个文件都是一个 module。

也就是每个 ts 文件中都需要有 import 或者 export

#### References to const enum members

```ts
declare const enum Num {
  First = 0,
  Second = 1
}
const a = {
  age: Num.first // 这里会报错
}
```

我们不能直接使用 Num.First，因为编译后的 js 中，declare 那一行会直接去掉，Num.First 这个 enum 会被直接替换成一个常量，所以作用域中是没有 Num 的



## [静态资源处理](https://cn.vitejs.dev/guide/features.html#static-assets)

```ts
import test from './test?url' // 路径
import test from './test?raw' // 将内容全都 stringfy
import Worker from './worker?worker' // 生成一个 webWoker 对象
import pkg from './package.json'

const worker = new Worker()
```

正常的话，需要将一个 webworker 单独当成一个 js 文件去引入，再去 new Worker

参考 vite 中的 [Web Worker](https://cn.vitejs.dev/guide/features.html#web-workers)



## [环境变量和模式](https://cn.vitejs.dev/guide/env-and-mode.html#env-variables-and-modes)

通过 `import.meta.env` 来取得

包含以下内容

- MODE（通过 MODE 可以改变环境变量的读取，比如`"dev": "vite --mode test"`，这样当前就是 test 环境了）
- BASE_URL
- PROD
- DEV
- SSR

可以在根目录下建 `.env` 文件来声明其他环境变量（`.env.development` 声明的是开发环境下的环境变量, `.env.production` 声明的是生产环境下的环境变量），参考 [.env 文件](https://cn.vitejs.dev/guide/env-and-mode.html#env-files)



自行添加的环境变量如下：

```
// .env

VITE_TITLE=test // 前面必须加上 VITE 前缀
```



如果新的环境变量需要拥有类型，且有 ts 智能提示，那么需要在 env.d.ts 中添加

```ts
/// <reference types="vite/client" />

interface ImportMetaEnv { // 这个类型会添加到 vite 给我们提供的 ImportMetaEnv 这个已有的类型上去
	VITE_TITLE: string
}
```



# Vite 高级应用

## [热更新](https://cn.vitejs.dev/guide/features.html#hot-module-replacement)

每个框架的热更新都不一样

vite 有一套自己形成体系的 HMR API，并不是使用 rollup 的

热更新的功能是在 `plugins: [vue(), vueJsx()]` 中被引入的，插件中自动集成了的



初步使用（[hot.accept(cb)](https://cn.vitejs.dev/guide/api-hmr.html#hot-acceptcb)）：

```ts
if (import.meta.hot) { // vite build 的代码中就没有 hot
  import.meta.hot.accept(newModule => { // accept 表示用新的module替换老的module
    newModule.render() // 热更新的模块进行重新渲染一次，如果只使用 render() 那么就是老的 module 的 render
  })
}
```

热更新相关的代码都必须写在 `import.meta.hot` 的 if 语句下面，这样在生产环境 rollup 就会将其 Treeshaking



通过 websocket 来实现热更新

![HMR ws](https://raw.githubusercontent.com/aboutcroon/Notes/main/Vite/assets/HMR%20ws.png)

acceptedPath 的文件路径就是我们实际修改的文件对应的路径



再切换到 js 请求，可见，更新之后会有新的对这个文件的一次请求，这时新的 js 文件就会替换老的 js 文件，这样就形成了热更新的过程：

1. 在 server 端发现了文件更新
2. 推送一个事件到前端浏览器中
3. 浏览器再对这个文件进行重新请求，并替换在浏览器中替换老的模块

如果是没有经过 `import.meta.hot.accept` 的文件，每次更改后就会重新渲染一遍页面（也就是页面刷新）



## [Glob 导入](https://cn.vitejs.dev/guide/features.html#glob-import)

可以通过正则表达式来 import 一组文件（类似于 webpack 的 require.context）

这是 vite 单独提供给我们的功能（来源于第三方的库 [fast-glob](https://github.com/mrmlnc/fast-glob)），只能在 vite 中使用，并不是一个规范，我们可以看到打包后的文件

直接被编译成了 `"文件名: import(..)"` 的形式

![glob import](https://raw.githubusercontent.com/aboutcroon/Notes/main/Vite/assets/glob%20import.png)

使用 globEager 则可以不用异步引入，可直接引入所有模块



import.meta.glob 编译后:

```ts
const modules = {
  './dir/foo.js': () => import('./dir/foo.js'),
  './dir/bar.js': () => import('./dir/bar.js')
}
```



import.meta.globEager 编译后:

```ts
import * as __glob__0_0 from './dir/foo.js'
import * as __glob__0_1 from './dir/bar.js'
const modules = {
  './dir/foo.js': __glob__0_0,
  './dir/bar.js': __glob__0_1
}
```



## [依赖预构建（预编译）](https://cn.vitejs.dev/guide/dep-pre-bundling.html#dependency-pre-bundling)

预构建是让 vite 速度这么快的原因之一（预构建通过 esbuild 执行，所以它通常非常快。）

对于 node_modules 中安装的第三方库，vite 在第一次启动之前，会对这些依赖包进行一个预编译，然后放到一个 cache 里面，之后我们用到的包就直接去这个 cache 里面取（文件夹位于 node_modules/.vite）



[预编译的目的及意义](https://cn.vitejs.dev/guide/dep-pre-bundling.html#the-why)：

1. **CommonJS 和 UMD 兼容性:** 将 common.js 或 UMD 等其他形式转换成 ESM （因为我们开发时，vite 全部是依赖于浏览器原生的 ESM 加载方式去运行的）
2. **性能:** 将零散的文件打包到一起
3. 缓存



将零散的文件打包到一起，比如 lodash 有上百个函数分别放在不同的文件中，我们通过 ESM 的方式 import lodash，就会一下子引入上百个文件，这个功能是针对一个库里面有非常多个文件的情况，如果不打包到一起，那么浏览器会产生很多 js 请求



node_modules 中有个 [.vite](https://cn.vitejs.dev/config/#cachedir) 的文件夹，这里就是生成缓存的地方，我们在里面可以看到很多依赖文件

![vite cache](https://raw.githubusercontent.com/aboutcroon/Notes/main/Vite/assets/vite%20cache.png)

之后可以直接读这里的缓存文件，可以不再走任何跟编译有关的东西



另外，预编译过程有一个很重要的步骤，就是需要把 common js 的部分转换成 ESM

因为我们开发时，vite 全部是依赖于浏览器原生的 ESM 加载方式去运行的



如果我们将 .vite 文件夹去掉，项目是跑不起来的

同时，我们可以[自定义行为](https://cn.vitejs.dev/guide/dep-pre-bundling.html#customizing-the-behavior)，在 vite.config.ts 中添加 [optimizeDeps.include](https://cn.vitejs.dev/config/#optimizedeps-include) 选项，这个表示 vite 中哪些文件是需要预编译的，相对应的，exclude 就是哪些文件不需要预编译，平时如果有不是 ESM 的模块被加载进 node_modules 中了，那么我们就需要手动将其添加到预编译文件中



另外，我们项目中的 js 文件，vite 在请求时都会加上 `Cache-Control = no-cache` 来避免浏览器缓存，以防我们更新了文件但页面不刷新，但第三方的库一般都会设置缓存，此时 vite 会设置 max-age, immutable 来进行强缓存，那么我们加载第三方库的时候不用重新发送请求，可以直接使用缓存，这样提高了加载速度，参考官方文档[浏览器缓存](https://cn.vitejs.dev/guide/dep-pre-bundling.html#browser-cache)。



# [Vite 插件系统](https://cn.vitejs.dev/guide/api-plugin.html)

## 介绍

vite 插件其实是一个受限制的 rollup 插件，rollup 的 hook 会实现 rollup 插件在不同阶段做的不同的事情，而 vite 支持的 rollup 插件只是支持其中的一部分功能，它不是所有的 hook 都会去使用，所以如果你使用的 rollup 插件都在 vite 支持的那几个 hook 里面，那么它就可以在 vite 中去使用



[vite 插件命名规范](https://cn.vitejs.dev/guide/api-plugin.html#conventions)：

1. rollup-plugin-xxx （在 rollup 和 vite 中都能进行使用的插件）
2. vite-plugin-xxx（仅支持 vite，vite 也有自己的钩子，在 rollup 中肯定是不能用的）



[vite 与 rollup 通用的钩子](https://cn.vitejs.dev/guide/api-plugin.html#universal-hooks)：

1. 服务启动时（npm run dev）：options, buildStart
2. 对于每个模块，会兼容 resolveId, load, transform 这几个钩子，resolveId 就是去找到对应的文件，load 就是去加载对应的源码，transform 就是将源码转变成目标代码
3. 服务器关闭时，会去调用 buildEnd 和 closeBundle 钩子



```js
export default defineConfig({
  plugins: [], // 这里的 plugin 只会遵守 vite 的钩子执行
  build: {
    rollupOptions: {
      plugins: [] // 这里的插件可以完全符合 rollup 的执行方式
    }
  }
})
```



[vite 独有的钩子](https://cn.vitejs.dev/guide/api-plugin.html#vite-specific-hooks)：

1. config：用于在插件中根据其他的配置去更新配置
2. configResolved：所有的插件对应的 config 都执行后
3. configureServer：进行一些 vite 的 devServer 的中间件的操作
4. transformIndexHtml：可以对入口 html 文件进行操作和转换
5. handleHotUpdate：处理热更新



## 插件执行时机

vite 中插件执行是有顺序概念，vite 的顺序比 rollup 的要丰富一点

我们可以通过一个变量来控制 vite 插件的执行时机



在 vite 中有[三个执行时机](https://cn.vitejs.dev/guide/using-plugins.html#enforcing-plugin-ordering)：

1. pre
2. normal (在 vite 核心插件执行后，已经 build 插件执行之前被执行，此时 vite 的代码编译还没有开始)
3. post（在 vite build 之后，执行代码构建部分的工作，比如代码的 minimize）



一个插件示例：

```ts
export default (enforce?: 'pre' | 'post') => {
  return {
    name: 'test',
    enforce,
    buildStart () {
      console.log('buildStart', enforce)
		},
    resolveId () {
      console.log('resolveid', enforce)
    }
  }
}
```



## [插件 API](https://cn.vitejs.dev/guide/api-plugin.html#plugin-api)

vite 独有的插件，使用方法参考 [vite 独有钩子](https://cn.vitejs.dev/guide/api-plugin.html#vite-specific-hooks)

通用的插件，使用方法参考 [rollup 插件文档](https://rollupjs.org/guide/en/#plugin-development)



## [HMR API](https://cn.vitejs.dev/guide/api-hmr.html#hmr-api)

ImportMeta 的类型：

```ts
interface ImportMeta {
  readonly hot?: {
    readonly data: any

    accept(): void
    accept(cb: (mod: any) => void): void
    accept(dep: string, cb: (mod: any) => void): void
    accept(deps: string[], cb: (mods: any[]) => void): void

    dispose(cb: (data: any) => void): void
    decline(): void
    invalidate(): void

    on(event: string, cb: (...args: any[]) => void): void
  }
}
```



vite 热更新与 webpack 热更新的区别：

webpack 有一整套模块管理方式，设计了一整套模块代理的功能，也就是说，比如我们在 main.js 中 import  renderA.js，然后就会拿到一个 `_webpack_module_renderA_` 的对象，但这个对象并不指向 renderA 中的内容，而是一个类似 new proxy()，我们执行 `.render()` 时，实际上是 proxy 中执行了一个函数去读取对应的目标模块上面的对象，而这个目标对象更新了之后，`.render()` 返回的也就是这个新的函数（相当于一个引用，这个引用总是不会变的）

而 vite 是基于 ESM 的加载方式，所以不会再给自己去实现一套模块管理的功能，所以 vite 里面不适合去使用模块代理的方式实现 HMR API。vite 的热更新方式则是：某个模块若更新了代码，这个模块就要去执行这个更新代码的逻辑，accept 中的回调就可以接收已更新的模块。

所以 vite 新模块执行后，旧模块还是依然存在的，一些遗留的代码可能还会继续执行（例如setInterval），可能会导致内存溢出，这样我们就需要其他的 api 去解决这个问题，这些都是一些副作用。例如，main.js 中引入了 renderA.js 中的代码，那么 main.js 热更新之后，renderA.js 中的 setInterval 就会重新执行，这种重新执行是不可控的。解决方案是使用 [hot.dispose](https://cn.vitejs.dev/guide/api-hmr.html#hot-dispose-cb)：

```ts
let index = 0
const timer = setInterval(() => {
  console.log(index++)
}, 1000)

if (import.meta.hot) {
  import.meta.hot.dispose(() => { // 在新模块执行，而旧模块即将卸载之前
    if (timer) {
      clearInterval(timer)
    }
  })
}
```



HMR 的 API 我们可能在项目的某些特殊需求中会单独用到，而像我们用到的大多数插件，比如 vue 插件，热更新已经是被集成进去了，所以在日常开发中是开箱即用的。

官方文档中也有说

> 手动 HMR API 主要用于框架和工具作者。
>
> 作为最终用户，HMR 可能已经在特定于框架的启动器模板中为你处理过了。



# [rollup](https://rollupjs.org/guide/en/) 的简单介绍

## 介绍

rollup 更注重于去打包现代的项目，去打包成一个独立的类库，并没有很多优化的配置项，是开源类库的优先选择

rollup 是以 ESM 标准为目标的构建工具

rollup 有 Tree shaking 功能



将 rollup 全局安装到我们的电脑上

```
npm i -g rollup
```



rollup 有那么几种打包模式（format）（如果不指定的话，就直接为 es 模式）：umd(后面三种类型都兼容的文件类型)，amd，cjs，es，iife(立即执行文件的形式)



## 命令行的使用

```
rollup -i index.js --file dist.js --format cjs
```

-i 是输入文件，--file 是输出文件，--format 是我们需要输出的文件类型



多个输入文件时，输出为一个文件夹

```
rollup -i index.js -i a.js --dir dist
```



## 使用 rollup.config.js 文件

该文件默认使用 es 语法，若要使用 cjs 语法，则该将文件重命名为 rollup.config.cjs

```js
export default {
  input: "index.js",
  output: {
    file: "dist.js",
    format: "umd",
    name: "Index"
  }
}
```

然后执行

```
rollup --config rollup.config.js
```



## plugin

rollup 的插件都是函数形式

插件地址：https://github.com/rollup/awesome

rollup 会在不同的阶段设置不同的 hook，然后插件里面通过 hook 设置不同阶段的功能



# [esbuild](https://esbuild.github.io/) 简单介绍

## 使用

esbuild 是用 go 语言来写的，所以并不是原生支持 js 文件的，它只是做语法解析的，并不能运行 js 文件，esbuild 中没有配置文件，所以 esbuild 全是用命令行去使用的，或者是我们在 js 中自己去调用命令行（vite 就是这么做的）



esbuild 出来的都是 es6 语法，不能生成 es5 语法，所以 vite 在开发环境使用它的话是没有问题的，生产环境才需要考虑 es5 的问题。



详细参考官方文档

