## 从文件系统导入多个模块

Vite 支持使用特殊的 `import.meta.glob` 函数从文件系统导入多个模块：

```js
const modules = import.meta.glob('./dir/*.js')
```

将webpack语法写成vite语法

```js
export async function loadAllPlugins (app: ReturnType<typeof createApp>) {
  // const files = require.context('.', true, /\.ts$/)
  // files.keys().forEach(key => {
  //   if (typeof files(key).default === 'function') {
  //     if (key !== './index.ts') files(key).default(app)
  //   }
  // })
  // 替换成
  const modules = import.meta.glob('./**.ts')
  for (const path in modules) {
    const mod = await modules[path]()
    if (typeof mod.default === 'function') {
      if (path !== './index.ts') mod.default(app)
    }
  }
}
```

文档地址：[Glob导入](https://cn.vitejs.dev/guide/features.html#glob-import)



**NOTE**：注意，这里是有一个异步导入的，所以我们在 main.ts 中需要 await 之后，再 mount，不然会出现警告 `[Vue warn]: Failed to resolve component`

```js
/** 加载所有 Plugins */
await loadAllPlugins(app)

app.mount('#app')
```



## @import '~ant-design'

当使用 ant-design-vue 等 ui 组件时，若需要定制主题，则需要引入样式入口文件

```less
@import '～ant-design-vue/dist/antd.less'; // 引入官方提供的 less 样式入口文件
```

但是项目会报错：`cant find xxx`

这是因为 ~ 这种路径写法是 vue-cli 中的约定，它使我们可以引用 node_modules 包内的资源，详见文档： [URL转换规则](https://link.zhihu.com/?target=https%3A//cli.vuejs.org/zh/guide/html-and-static-assets.html%23url-%E8%BD%AC%E6%8D%A2%E8%A7%84%E5%88%99)

所以在 vite 中，直接

```less
@import 'node_modules/ant-design-vue/dist/antd.less'; // 引入官方提供的 less 样式入口文件
```



## require

### 模块引入

**webpack**

```js
const modulesFiles = require.context('./modules', true, /.js$/)
```

**vite**

```js
const modulesFiles = import.meta.globEager("./modules/*.js")
```



### 图片引入

webpack通过 require 引入的图片，不加require，图片会引入不成功

**webpack**

```js
const img = require(‘@/images/log.png’)
```

**vite**

在vite中不需要添加require，可以直接通过路径引入

```js
const img = ‘/images/log.png’
```



> 参考文章：https://juejin.cn/post/7054562620132720648
