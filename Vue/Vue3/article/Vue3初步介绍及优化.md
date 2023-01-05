## Vue's core engine

### Reactivity Module（响应式模块）

响应式模块允许我们创建 Javascript 响应对象，并可以观察其变化

### Compiler Module（编译器模块）

获取 HTML 模版，并将他们编译成渲染函数，这会在运行时的浏览器中发生 ，浏览器只接收这些渲染函数来渲染页面

### Renderer Module（渲染模块）

该模块包括三个阶段：

#### 渲染阶段(render)

该阶段将调用 render 函数，它返回一个虚拟 DOM 节点

#### 挂载阶段(mount)

使用虚拟 DOM 节点，并调用 DOM API 来创建网页

#### 补丁阶段(patch)

渲染器将旧的虚拟节点和新的虚拟节点进行比较，并只更新网页变化的的部分



## Vue3相较于Vue2的优化

### 源码优化

#### 更好的代码管理方式：monorepo

Vue.js 2.x 的源码托管在 src 目录，然后依据功能拆分出了 compiler（模板编译的相关代码）、core（与平台无关的通用运行时代码）、platforms（平台专有代码）、server（服务端渲染的相关代码）、sfc（.vue 单文件解析相关代码）、shared（共享工具代码） 等目录。

![vue2结构](/Users/liuchang/Documents/markdown/学习/Vue3/assets/vue2结构.png)

而到了 Vue.js 3.0 ，整个源码是通过 monorepo 的方式维护的，根据功能将不同的模块拆分到 packages 目录下面不同的子目录中。

![monorepo](/Users/liuchang/Documents/markdown/学习/Vue3/assets/monorepo.png)



可以看出，相对于 Vue.js 2.x 的源码组织方式，monorepo 把这些模块拆分到不同的 package 中，每个 package 有各自的 API、类型定义和测试。这样使得`模块拆分更细化`，`职责划分更明确`，模块之间的依赖关系也更加明确，开发人员也更容易阅读、理解和更改所有模块源码，提高代码的可维护性。

另外，一些 package（比如 reactivity 响应式库）是可以独立于 Vue.js 使用的，这样用户如果只想使用 Vue.js 3.0 的响应式能力，可以单独依赖这个响应式库而不用去依赖整个 Vue.js，减小了引用包的体积大小，而 Vue.js 2 .x 是做不到这一点的。



#### 使用Typescript

使用 TypeScript 重构了整个项目，提供了更好的类型检查，能支持复杂的类型推导。



### 性能优化

#### 源码体积优化

- 首先，移除一些冷门的 feature（比如 filter、inline-template 等）
- 其次，引入 tree-shaking 的技术，减少打包体积。

#### 数据劫持优化

我们都知道，Vue.js 2.x 内部都是通过 Object.defineProperty 这个 API 去劫持数据的 getter 和 setter：

```js
Object.defineProperty(data, 'a',{
  get(){
    // track
  },
  set(){
    // trigger
  }
})
```



这会带来一些问题：

- 它必须预先知道要拦截的 key 是什么，所以它并不能检测对象属性的添加和删除
- 对于一个嵌套层级较深的对象，如果要劫持它内部深层次的对象变化，就需要递归遍历这个对象，执行 Object.defineProperty 把每一层对象数据都变成响应式的。毫无疑问，如果我们定义的响应式数据过于复杂，这就会有相当大的性能负担。



为了解决上述 2 个问题，Vue.js 3.0 使用了 Proxy API 做数据劫持，它的内部是这样的：

```ts
observed = new Proxy(data, {
  get() {
    // track
  },
  set() {
    // trigger
  }
})
```

由于它劫持的是`整个对象`，那么自然对于对象的属性的增加和删除都能检测到。

但要注意的是，Proxy API 并不能监听到内部深层次的对象变化，因此 Vue.js 3.0 的处理方式是在 getter 中去递归响应式，这样的好处是真正访问到的内部对象才会变成响应式，而不是无脑递归，这样无疑也在很大程度上提升了性能。



### 编译优化

以下是 Vue.js 2.x 从 new Vue 开始渲染成 DOM 的流程



![vue2编译](/Users/liuchang/Documents/markdown/学习/Vue3/assets/vue2编译.png)

上面说过的响应式过程就发生在图中的 init 阶段，另外 template compile to render function 的流程是可以借助 vue-loader 在 webpack 编译阶段离线完成，并非一定要在运行时完成。

所以，在优化整个 Vue.js 的运行时，除了数据劫持部分的优化，Vue3 也在耗时相对较多的 patch 阶段想办法，并且它通过在编译阶段优化编译的结果，来实现运行时 patch 过程的优化。

例如我们要更新下面这个组件，我们就得先 diff div, 然后 diff props of div, diff children of div, diff p, diff props of p, diff children of p...

```HTML
<template>
  <div id="content">
    <p class="text">static text</p>
    <p class="text">static text</p>
    <p class="text">{{message}}</p>
    <p class="text">static text</p>
    <p class="text">static text</p>
  </div>
</template>
```

可以看到，因为这段代码中只有一个动态节点，所以这里有很多 diff 和遍历其实都是不需要的，这就会导致 vnode 的性能跟模版大小正相关，跟动态节点的数量无关，当一些组件的整个模版内只有少量动态节点时，这些遍历都是性能的浪费。

整个 diff 过程如图所示：

![vue2diff](/Users/liuchang/Documents/markdown/学习/Vue3/assets/vue2diff.png)

在 Vue3 中，它通过编译阶段对静态模板的分析，编译生成了 Block tree。Block tree 是一个将模版基于动态节点指令切割的嵌套区块，每个区块内部的节点结构是固定的，而且每个区块只需要以一个 Array 来追踪自身包含的动态节点。借助 Block tree，Vue3 将 vnode **更新性能**由与模版整体大小相关提升为**与动态内容的数量相关**，这是一个非常大的性能突破。



### 语法 API 优化：Composition API

Composition API 可以优化我们写代码的逻辑组织。Vue2中是按照 methods、computed、data、props 这些不同的选项分类，在大型组件中，这些逻辑关注点是非常分散的。而在Vue3中，将某个逻辑关注点相关的代码全都放在一个函数里了，这样当需要修改一个功能时，就不再需要在文件中跳来跳去。



Composition API 还可以优化我们代码的逻辑复用。

在 Vue2 中，我们通常会用 mixins 去复用逻辑，当我们一个组件混入大量不同的 mixins 的时候，会存在两个非常明显的问题：命名冲突和数据来源不清晰。

Vue3 设计的 Composition API，就很好地解决了 mixins 的这两个问题。

一个典型的例子：

```ts
// mouse.ts

import { ref, onMounted, onUnmounted } from 'vue'
export default function useMousePosition() {
  const x = ref(0)
  const y = ref(0)
  const update = e => {
    x.value = e.pageX
    y.value = e.pageY
  }
  onMounted(() => {
    window.addEventListener('mousemove', update)
  })
  onUnmounted(() => {
    window.removeEventListener('mousemove', update)
  })
  return { x, y }
}
```

上面我们约定了 useMousePosition 这个函数为 hook 函数，然后在组件中使用：

```html
<template>
  <div>
    Mouse position: x {{ x }} / y {{ y }}
  </div>
</template>
<script>
  import useMousePosition from './mouse'
  export default {
    setup() {
      const { x, y } = useMousePosition()
      return { x, y }
    }
  }
</script>
```

可以看到，整个数据来源清晰了，即使去编写更多的 hook 函数，也不会出现命名冲突的问题。

Composition API 除了在逻辑复用方面有优势，也会有更好的类型支持，因为它们都是一些函数，在调用函数时，自然所有的类型就被推导出来了，不像 Options API 所有的东西使用 this。另外，Composition API 对 tree-shaking 友好，代码也更容易压缩。

但是，Composition API 属于 API 的增强，它并不是 Vue3 组件开发的唯一方式，如果你的组件足够简单，你还是可以使用 Vue2 的 Options API

