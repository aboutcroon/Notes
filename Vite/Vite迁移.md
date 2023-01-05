从 Webpack 5 迁移至 Vite 3，整体步骤如下：

1. 查看 vite 官网，通过 `npm create vite@latest` 创建一个新的 Vite 项目，进行比较
2. 比较 package.json 中 Vite 必需的依赖，加进来



## package.json

以下为 vite 必需的依赖和设置（如果需要 Typescript），添加进来：

```json
	"type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc && vite build",
    "preview": "vite preview"
  },	
	"dependencies": {
    "vue": "^3.2.41"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^3.2.0",
    "typescript": "^4.6.4",
    "vite": "^3.2.3",
    "vue-tsc": "^1.0.9"
  }
```



## vite.config.ts

基本的 `vite.config.ts` 文件

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()]
})
```

报错 "@" 的路径问题，我们需要在 `vite.config.ts` 中加上 alias。

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { createHtmlPlugin } from 'vite-plugin-html'
import dyText from "./public/dynamic/dyText"
import path from 'path'

export default defineConfig({
  plugins: [
    vue(),
    createHtmlPlugin({
      /**
       * 需要注入 index.html ejs 模版的数据
       */
       inject: {
        data: {
          title: dyText.title || ''
        }
       }
    })
  ],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src')
    }
  },
})
```

支持 jsx，安装 `@vitejs/plugin-vue-jsx` 插件：

```sh
npm i @vitejs/plugin-vue-jsx -D
```



### server 配置

参考官方文档：https://cn.vitejs.dev/config/server-options.html

主要是配置 server.proxy，使用 vite 的代理来解决跨域



## index.html

将 `index.html` 文件从 `public` 目录移到根目录下，并做部分修改：

```html
<!-- before -->
<body>
  <div id="app"></div>
</body>

<!-- now -->
<body>
  <div id="app"></div>
  <script type="module" src="/src/main.ts"></script>
</body>
```

`npm run dev` 运行之后，此时 html 文件中的 ejs 语法会报错，我们需要 vite 的插件来支持我们在 html 中写 ejs 语法：

```sh
npm i vite-plugin-html -D
```

> 插件地址：https://github.com/vbenjs/vite-plugin-html

详细使用参考插件的 github 文档



## router

渲染函数的使用发生变化：

```js
// before
const Skeleton = {
  render: h => <router-view></router-view>
}

// now
import { h } from 'vue'
const Skeleton = h('router-view')
```

> vue3 以前，h 是作为 render 函数的入参来使用，不用单独引入。
>
> vue3 中，h 是直接从 vue 中引入进来的。



## postcss.config.js

postcss.config.js 中的内容：

```js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  }
}
```

在 vite3 中，所有的代码都必须使用 ESM 写法，CommonJS 写法的文件需要命名为 `.cjs` 。所以我们将 `postcss.config.js` 改成 `postcss.config.cjs` 就可以。

> 参考：https://github.com/BuilderIO/qwik/issues/836#issuecomment-1193114640



## tsconfig.json

为了支持 vite 中的类型，需要添加：

```json
"types": ["vite/client"]
```



## 页面代码

### process.env 替换成 import.meta.env

`process.env` 替换成 `import.meta.env`，注意只能在沿着 `main.ts` 往下找的 module 文件中使用，其他文件依然无法使用 `import.meta`



### 去除 require

之前的动态导入部分：

```js
if (xx) {
  require('././xxx')
}
```

现在需要把 require 改成 import

参考动态导入 https://zh.javascript.info/modules-dynamic-imports

```js
if (xx) {
  import('././xxx')
}
```

> import xx from 'xxx' 是静态的导入，会提到文件最上方执行





### 报错

### [vite] Internal server error: Cannot read properties of undefined (reading 'length')

属于 tailwind.config.js 中出的问题，将文件名改为 `tailwind.config.cjs`



### Failed to parse source for import analysis because the content contains invalid JS syntax. If you are using JSX, make sure to name the file with the .jsx or .tsx extension.

原因是在 js 文件中写了 jsx 的语法，类似 `() => <div></div>`

解决方案是：

1. 把 `.js` 文件改为 `.jsx` 文件
2. 使用 loader: { '.js': 'jsx' } 来配置

第二种解法，参考 esbuild 的配置 https://esbuild.github.io/content-types/#ts-vs-tsx，https://github.com/vitejs/vite/issues/769

但是在 vite 中找不到哪里可以传递配置到 esbuild，只能先使用第一种方法了。

