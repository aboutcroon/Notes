## creatApp

文件入口

```ts
import { creatApp } from Vue
import App from './App.ts'

creatApp(App).mount('#app')
```



我们需要了解 createApp 里面都做了些什么

createApp 其实是个入口函数，它是 Vue.js 对外暴露的一个函数，我们来看一下它的内部实现：

```ts
// packages/runtime-dom/src/index.ts

const createApp = ((...args) => {
  // 创建 app 对象
  const app = ensureRenderer().createApp(...args)
  const { mount } = app
  // 重写 mount 方法
  app.mount = (containerOrSelector) => {
    // ...
  }
  return app
})
```

可以看到，createApp 主要做了两件事情：创建 app 对象和重写 app.mount 方法



## 创建 app 对象

首先，使用了 ensureRenderer().createApp() 来创建 app 对象 ：

```ts
const app = ensureRenderer().createApp(...args)
```

其中 ensureRenderer() 用来创建一个渲染器对象，它的内部代码是这样的：

```ts
// packages/runtime-dom/src/index.ts

// 渲染相关的一些配置，比如更新属性的方法，操作 DOM 的方法
const rendererOptions = {
  patchProp,
  ...nodeOps
}
let renderer
// 延时创建渲染器，当用户只依赖响应式包的时候，可以通过 tree-shaking 移除核心渲染逻辑相关的代码
function ensureRenderer() {
  return renderer || (renderer = createRenderer(rendererOptions))
}
```

> 这里先用 ensureRenderer() 来延时创建渲染器，这样做的好处是当用户只依赖响应式包的时候，就不会创建渲染器，因此可以通过 tree-shaking 的方式移除核心渲染逻辑相关的代码。

这个渲染器（可以简单地把渲染器理解为包含平台渲染核心逻辑的 JavaScript 对象），是为跨平台渲染做准备的



结合上面的代码继续深入，在 Vue.js 3.0 内部通过 createRenderer 创建一个渲染器，这个渲染器内部会有一个 createApp 方法

```ts
// packages/runtime-core/src/renderer.ts

function createRenderer(options) {
  return baseCreateRenderer(options)
}

function baseCreateRenderer(options) {
  function render(vnode, container) {
    // 组件渲染的核心逻辑
  }

	...

  return {
    render,
    createApp: createAppAPI(render)
  }
}
```

createApp 方法是执行 createAppAPI 方法返回的函数，接受了 rootComponent 和 rootProps 两个参数，我们在应用层面执行 createApp(App) 方法时，会把 App 组件对象作为根组件传递给 rootComponent。这样，createApp 内部就创建了一个 app 对象，它会提供 mount 方法，这个方法是用来挂载组件的。

```ts
// packages/runtime-core/src/apiCreateApp.ts

function createAppAPI(render) {
  // createApp 方法接受的两个参数：根组件的对象和 prop
  return function createApp(rootComponent, rootProps = null) {
    const app = {
      _component: rootComponent,
      _props: rootProps,
      mount(rootContainer) {
        // 创建根组件的 vnode
        const vnode = createVNode(rootComponent, rootProps)
        // 利用渲染器渲染 vnode
        render(vnode, rootContainer)
        app._container = rootContainer
        return vnode.component.proxy
      }
    }
    return app
  }
}
```

> 在整个 app 对象创建过程中，Vue.js 利用闭包和函数柯里化的技巧，很好地实现了参数保留。比如，在执行 app.mount 的时候，并不需要传入渲染器 render，这是因为在执行 createAppAPI 的时候渲染器 render 参数已经被保留下来了。

可以看到，`createApp` 是将若干属性和方法挂载在 `app` 这个变量中，最后返回 `app`。



## 重写 app.mount 方法

根据前面的分析，我们知道 createApp 返回的 app 对象已经拥有了 mount 方法了，但在入口函数中，接下来的逻辑却是对 app.mount 方法的重写。为什么要重写这个方法，而不把相关逻辑放在 app 对象的 mount 方法内部来实现呢？

因为 Vue.js 不仅仅是为 Web 平台服务，它的目标是支持跨平台渲染，而 createApp 函数内部的 app.mount 方法是一个标准的可跨平台的组件渲染流程：先创建vnode，再渲染vnode

```ts
mount(rootContainer) {
  // 创建根组件的 vnode
  const vnode = createVNode(rootComponent, rootProps)
  // 利用渲染器渲染 vnode
  render(vnode, rootContainer)
  app._container = rootContainer
  return vnode.component.proxy
}
```

此外，参数 rootContainer 也可以是不同类型的值，比如，在 Web 平台它是一个 DOM 对象，而在其他平台（比如 Weex 和小程序）中可以是其他类型的值。所以这里面的代码不应该包含任何特定平台相关的逻辑，也就是说这些代码的执行逻辑都是与平台无关的。因此我们需要在外部重写这个方法，来`完善 Web 平台下的渲染逻辑`。



看看 app.mount 重写都做了哪些事情：

```ts
// packages/runtime-dom/src/index.ts

app.mount = (containerOrSelector) => {
  // 标准化容器
  const container = normalizeContainer(containerOrSelector)
  if (!container) return

  const component = app._component
   // 如组件对象没有定义 render 函数和 template 模板，则取容器的 innerHTML 作为组件模板内容
  if (!isFunction(component) && !component.render && !component.template) {
    component.template = container.innerHTML
  }
  // 挂载前清空容器内容
  container.innerHTML = ''
  // 真正的挂载
  return mount(container)
}
```

首先是通过 normalizeContainer 标准化容器（这里可以传字符串选择器或者 DOM 对象，但如果是字符串选择器，就需要把它转成 DOM 对象，作为最终挂载的容器），然后做一个 if 判断，如果组件对象没有定义 render 函数和 template 模板，则取容器的 innerHTML 作为组件模板内容；接着在挂载前清空容器内容，最终再调用 app.mount 的方法走标准的组件渲染流程。



从 app.mount 开始，才算真正进入组件渲染流程，接下来，重点看一下核心渲染流程做的两件事情：创建 vnode 和渲染 vnode。



### 创建 vnode

vnode 本质上是用来描述 DOM 的 JavaScript 对象，它在 Vue.js 中可以描述不同类型的节点，比如**普通元素节点**、**组件节点**等。



**普通元素节点**，例如：

```html
<button class="btn" style="width:100px;height:50px">click me</button>
```

我们可以用 vnode 这样表示`<button>`标签：

```ts
const vnode = {
  type: 'button',
  props: { 
    'class': 'btn',
    style: {
      width: '100px',
      height: '50px'
    }
  },
  children: 'click me'
}
```

其中，type 属性表示 DOM 的标签类型；props 属性表示 DOM 的一些附加信息，比如 style 、class 等；children 属性表示 DOM 的子节点，它也可以是一个 vnode 数组，只不过 vnode 可以用字符串表示简单的文本 。



然后，vnode 除了可以像上面那样用于描述一个真实的 DOM，也可以用来描述组件。**组件节点** 示例：

```html
<custom-component msg="test"></custom-component>
```

我们可以用 vnode 这样表示 `<custom-component>` 组件标签：

```ts
const CustomComponent = {
  // 在这里定义组件对象
}
const vnode = {
  type: CustomComponent,
  props: { 
    msg: 'test'
  }
}
```

组件 vnode 其实是**对抽象事物的描述**，这是因为我们并不会在页面上真正渲染一个 `<custom-component>` 标签，而是渲染组件内部定义的 HTML 标签。

除了上两种 vnode 类型外，还有纯文本 vnode、注释 vnode 等等。



另外，Vue3 内部还针对 vnode 的 type，做了更详尽的分类，包括 Suspense、Teleport 等，且把 vnode 的类型信息做了编码，以便在后面的 patch 阶段，可以根据不同的类型执行相应的处理逻辑：

```ts
const shapeFlag = isString(type)
  ? 1 /* ELEMENT */
  : isSuspense(type)
    ? 128 /* SUSPENSE */
    : isTeleport(type)
      ? 64 /* TELEPORT */
      : isObject(type)
        ? 4 /* STATEFUL_COMPONENT */
        : isFunction(type)
          ? 2 /* FUNCTIONAL_COMPONENT */
          : 0
        ......
```

> vnode的优点：
>
> 1. 首先是**抽象**，引入 vnode，可以把渲染过程抽象化，从而使得组件的抽象能力也得到提升。
> 2. 其次是**跨平台**，因为 patch vnode 的过程不同平台可以有自己的实现，基于 vnode 再做服务端渲染、Weex 平台、小程序平台的渲染都变得容易了很多。
>
> 
>
> 不过这里要特别注意，使用 vnode 并不意味着不用操作 DOM 了，很多人会误以为 vnode 的性能一定比手动操作原生 DOM 好，这个其实是不一定的。
>
> 因为，首先这种基于 vnode 实现的 MVVM 框架，在每次 render to vnode 的过程中，渲染组件会有一定的 JavaScript 耗时，特别是大组件，比如一个 1000 * 10 的 Table 组件，render to vnode 的过程会遍历 1000 * 10 次去创建内部 cell vnode，整个耗时就会变得比较长，加上 patch vnode 的过程也会有一定的耗时，当我们去更新组件的时候，用户会感觉到明显的卡顿。虽然 diff 算法在减少 DOM 操作方面足够优秀，但最终还是免不了操作 DOM，所以说性能并不是 vnode 的优势。



**那么，Vue.js 内部是如何创建这些 vnode 的呢？**

在 app.mount 函数内部是通过 createVNode 函数创建了根组件的 vnode：

```ts
const vnode = createVNode(rootComponent, rootProps)
```

我们来看一下 createVNode 函数的大致实现：

```ts
// packages/runtime-core/src/vnode.ts

export const createVNode = (
  __DEV__ ? createVNodeWithArgsTransform : _createVNode
) as typeof _createVNode

function _createVNode(type, props = null, children = null) {
  if (props) {
    // 处理 props 相关逻辑，标准化 class 和 style
  }
  // 对 vnode 类型信息编码
  const shapeFlag = isString(type)
    ? 1 /* ELEMENT */
    : isSuspense(type)
      ? 128 /* SUSPENSE */
      : isTeleport(type)
        ? 64 /* TELEPORT */
        : isObject(type)
          ? 4 /* STATEFUL_COMPONENT */
          : isFunction(type)
            ? 2 /* FUNCTIONAL_COMPONENT */
            : 0
  const vnode = {
    type,
    props,
    shapeFlag,
    // 一些其他属性
  }
  // 标准化子节点，把不同数据类型的 children 转成数组或者文本类型
  normalizeChildren(vnode, children)
  return vnode
}
```

通过上述代码可以看到，其实 createVNode 做的事情很简单，就是：

1. 对 props 做标准化处理
2. 对 vnode 的类型信息编码
3. 创建 vnode 对象
4. 标准化子节点 children 。



我们现在拥有了这个 vnode 对象，接下来要做的事情就是把它渲染到页面中去。



### 渲染 vnode

```ts
// packages/runtime-core/src/render.ts

const render = (vnode, container, isSVG) => {
  if (vnode == null) {
    // 销毁组件
    if (container._vnode) {
      unmount(container._vnode, null, null, true)
    }
  } else {
    // 创建或者更新组件
    patch(container._vnode || null, vnode, container, null, null, null, isSVG)
  }
  // 缓存 vnode 节点，表示已经渲染
  container._vnode = vnode
}
```

如果它的第一个参数 vnode 为空，则执行销毁组件的逻辑，否则执行创建或者更新组件的逻辑。

接着看上面渲染 vnode 的代码中涉及的 patch 函数的实现：

```ts
// packages/runtime-core/src/render.ts

const patch = (
  n1, // 上次渲染过的vnode（旧节点）
  n2, // 新传进来的vnode（新节点）
  container,
  anchor = null,
  parentComponent = null,
  parentSuspense = null,
  isSVG = false,
  optimized = false
) => {
  if (n1 === n2) {
    return
  }

  // 如果存在新旧节点, 且新旧节点类型不同，则销毁旧节点
  if (n1 && !isSameVNodeType(n1, n2)) {
    anchor = getNextHostNode(n1)
    unmount(n1, parentComponent, parentSuspense, true)
    n1 = null
  }

  const { type, shapeFlag } = n2
  switch (type) {
    case Text:
      // 处理文本节点
      processText(n1, n2, container, anchor)
      break
    case Comment:
      // 处理注释节点
      processCommentNode(n1, n2, container, anchor)
      break
    case Static:
      // 处理静态节点
      ...
      break
    case Fragment:
      // 处理 Fragment 元素
      ...
      break
    default:
      if (shapeFlag & 1 /* ELEMENT */) {
        // 处理普通 DOM 元素
        processElement(n1, n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized)
      }
      else if (shapeFlag & 6 /* COMPONENT */) {
        // 处理组件
        processComponent(n1, n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized)
      }
      else if (shapeFlag & 64 /* TELEPORT */) {
        // 处理 TELEPORT
      }
      else if (shapeFlag & 128 /* SUSPENSE */) {
        // 处理 SUSPENSE
      }
  }
}
```

patch 这个函数有两个功能：

1. 根据 vnode 挂载 DOM（创建）
2. 根据新旧 vnode 更新 DOM（更新）



我们分析一下创建的过程。在创建的过程中，patch 函数接受多个参数，这里我们目前只重点关注前三个：

1. 第一个参数 **n1 表示旧的 vnode**，当 n1 为 null 的时候，表示是一次挂载的过程；
2. 第二个参数 **n2 表示新的 vnode 节点**，后续会根据这个 vnode 类型执行不同的处理逻辑；
3. 第三个参数 **container 表示 DOM 容器**，也就是 vnode 渲染生成 DOM 后，会挂载到 container 下面。



对于渲染的节点，我们这里重点关注**两种类型**节点的渲染逻辑：

1. 对`组件`的处理
2. 对`普通 DOM 元素`的处理



#### 对组件的处理

由于初始化渲染的是 App 组件，它是一个组件 vnode，所以我们来看一下组件的处理逻辑是怎样的。首先是用来处理组件的 processComponent 函数的实现，然后是挂载组件的 mountComponent 函数的实现

```ts
// packages/runtime-core/src/render.ts

const processComponent = (
  n1,
  n2,
  container,
  anchor,
  parentComponent,
  parentSuspense,
  isSVG,
  optimized
) => {
  if (n1 == null) {
   // 挂载组件
   mountComponent(n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized)
  }
  else {
    // 更新组件
    updateComponent(n1, n2, parentComponent, optimized)
  }
}

const mountComponent = (
  initialVNode,
  container,
  anchor,
  parentComponent,
  parentSuspense,
  isSVG,
  optimized
) => {
  // 创建组件实例
  const instance = initialVNode.component || (initialVNode.component = createComponentInstance(initialVNode, parentComponent, parentSuspense))
  // 设置组件实例
  setupComponent(instance)
  // 设置并运行带副作用的渲染函数
  setupRenderEffect(instance, initialVNode, container, anchor, parentSuspense, isSVG, optimized)
}
```

可以看到，挂载组件函数 mountComponent 主要做三件事情：

1. 创建组件实例
2. 设置组件实例
3. 设置并运行带副作用的渲染函数。



首先是**创建组件实例**，Vue.js 3.0 虽然不像 Vue.js 2.x 那样通过类的方式去实例化组件，但内部也通过对象的方式去创建了当前渲染的组件实例（createComponentInstance函数）。

其次**设置组件实例**，instance 保留了很多组件相关的数据，维护了组件的上下文，包括对 props、插槽，以及其他实例的属性的初始化处理。

最后是**运行带副作用的渲染函数** setupRenderEffect，我们重点来看一下这个函数的实现：

```ts
// packages/runtime-core/src/render.ts

const setupRenderEffect = (
  instance,
  initialVNode,
  container,
  anchor,
  parentSuspense,
  isSVG,
  optimized
) => {
  const componentUpdateFn = () => {
    if (!instance.isMounted) {
      // 渲染组件生成子树 vnode
      const subTree = (instance.subTree = renderComponentRoot(instance))
      // 把子树 vnode 挂载到 container 中
      patch(null, subTree, container, anchor, instance, parentSuspense, isSVG)
      // 保留渲染生成的子树根 DOM 节点
      initialVNode.el = subTree.el
      instance.isMounted = true
    }
    else {
      // 更新组件
    }
  }

  // 创建响应式的副作用渲染函数
  const effect = new ReactiveEffect(
    componentUpdateFn,
    () => queueJob(instance.update),
    instance.scope // track it in component's effect scope
  )

  const update = (instance.update = effect.run.bind(effect) as SchedulerJob)
}
```

该函数利用响应式库的 ReactiveEffect 函数创建了一个副作用渲染函数 componentUpdateFn

当组件的数据发生变化时，effect 函数包裹的内部渲染函数 componentUpdateFn 会重新执行一遍，从而达到重新渲染组件的目的。

渲染函数内部也会判断这是一次初始渲染还是组件更新。这里我们只分析初始渲染流程。



**初始渲染主要做两件事情：渲染组件生成 subTree、把 subTree 挂载到 container 中。**

首先，是渲染组件生成 subTree，它也是一个 vnode 对象。这里要注意区别 subTree 和 initialVNode（在 Vue.js 3.0 中，根据命名我们已经能很好地区分它们了，而在 Vue.js 2.x 中它们分别命名为 _vnode 和 $vnode）。

举个例子，在父组件 App 中里引入了 Hello 组件：

```html
<template>
  <div class="app">
    <p>This is an app.</p>
    <hello></hello>
  </div>
</template>
```

在 Hello 组件中是 `<div>` 标签包裹着一个 `<p>` 标签：

```html
<template>
  <div class="hello">
    <p>Hello, Vue 3.0!</p>
  </div>
</template>
```

在 App 组件中， `<hello>` 节点渲染生成的 vnode ，对应的就是 Hello 组件的 initialVNode，我们可以把它称作“组件 vnode”。而 Hello 组件内部整个 DOM 节点对应的 vnode 就是执行 renderComponentRoot 渲染生成对应的 subTree，我们可以把它称作“子树 vnode”。

我们知道每个组件都会有对应的 render 函数，即使你写 template，也会编译成 render 函数，而 renderComponentRoot 函数就是去执行 render 函数创建整个组件树内部的 vnode，把这个 vnode 再经过内部一层标准化，就得到了该函数的返回结果：子树 vnode。

渲染生成子树 vnode 后，接下来就是继续调用 patch 函数把子树 vnode 挂载到 container 中了。

那么我们又再次回到了 patch 函数，会继续对这个子树 vnode 类型进行判断，对于上述例子，App 组件的根节点是 `<div>` 标签，那么对应的子树 vnode 也是一个普通元素 vnode，那么我们接下来看 **对普通 DOM 元素的处理流程。**



#### 对普通DOM元素的处理

处理普通 DOM元素的 processElement 函数的实现：

```ts
// packages/runtime-core/src/render.ts

const processElement = (
  n1,
  n2,
  container,
  anchor,
  parentComponent,
  parentSuspense,
  isSVG,
  optimized
) => {
  isSVG = isSVG || n2.type === 'svg'
  if (n1 == null) {
    //挂载元素节点
    mountElement(n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized)
  }
  else {
    //更新元素节点
    patchElement(n1, n2, parentComponent, parentSuspense, isSVG, optimized)
  }
}

const mountElement = (
  vnode,
  container,
  anchor,
  parentComponent,
  parentSuspense,
  isSVG,
  optimized
) => {
  let el
  const { type, props, shapeFlag } = vnode
  // 创建 DOM 元素节点
  el = vnode.el = hostCreateElement(vnode.type, isSVG, props && props.is)
  if (props) {
    // 处理 props，比如 class、style、event 等属性
    for (const key in props) {
      if (!isReservedProp(key)) {
        hostPatchProp(el, key, null, props[key], isSVG)
      }
    }
  }
  if (shapeFlag & 8 /* TEXT_CHILDREN */) {
    // 处理子节点是纯文本的情况
    hostSetElementText(el, vnode.children)
  }
  else if (shapeFlag & 16 /* ARRAY_CHILDREN */) {
    // 处理子节点是数组的情况
    mountChildren(vnode.children, el, null, parentComponent, parentSuspense, isSVG && type !== 'foreignObject', optimized || !!vnode.dynamicChildren)
  }
  // 把创建的 DOM 元素节点挂载到 container 上
  hostInsert(el, container, anchor)
}
```

主要看挂载元素的 mountElement 函数，可以看到，挂载元素函数主要做四件事：

1. 创建 DOM 元素节点
2. 处理 props
3. 处理 children
4. 挂载 DOM 元素到 container 上。



首先是创建 DOM 元素节点，通过 hostCreateElement 方法创建，这是一个平台相关的方法，我们来看一下它在 Web 环境下的定义：

```ts
// packages/runtime-dom/src/nodeOps.ts

function createElement (tag, isSVG, is, props) => {
  const el = isSVG
  ? doc.createElementNS(svgNS, tag)
  : doc.createElement(tag, is ? { is } : undefined)

  if (tag === 'select' && props && props.multiple != null) {
    ;(el as HTMLSelectElement).setAttribute('multiple', props.multiple)
  }

  return el
}
```

它调用了底层的 DOM API document.createElement 创建元素，所以本质上 Vue.js 强调不去操作 DOM ，只是希望用户不直接碰触 DOM，它并没有什么神奇的魔法，底层还是会操作 DOM。

另外，如果是其他平台比如 Weex，hostCreateElement 方法就不再是操作 DOM ，而是平台相关的 API 了，这些平台相关的方法是在 `创建渲染器阶段作为参数传入的`。



创建完 DOM 节点后，接下来要做的是判断如果有 props 的话，给这个 DOM 节点添加相关的 class、style、event 等属性，并做相关的处理，这些逻辑都是在 hostPatchProp 函数内部做的。



接下来是对子节点的处理，我们知道 DOM 是一棵树，vnode 同样也是一棵树，并且它和 DOM 结构是一一映射的。

如果子节点是纯文本，则执行 hostSetElementText 方法，它在 Web 环境下通过设置 DOM 元素的 textContent 属性设置文本：

```ts
// packages/runtime-dom/src/nodeOps.ts

function setElementText (el, text) => {
  el.textContent = text
}
```

如果子节点是数组，则执行 mountChildren 方法：

```ts
// packages/runtime-core/src/renderer.ts

const mountChildren = (
  children,
  container,
  anchor,
  parentComponent,
  parentSuspense,
  isSVG,
  optimized,
  start = 0
) => {
  for (let i = start; i < children.length; i++) {
    // 预处理 child
    const child = (children[i] = optimized
      ? cloneIfMounted(children[i])
      : normalizeVNode(children[i]))
    // 递归 patch 挂载 child
    patch(
      null,
      child,
      container,
      anchor,
      parentComponent,
      parentSuspense,
      isSVG,
      optimized
    )
  }
}
```

子节点的挂载逻辑同样很简单，遍历 children 获取到每一个 child，然后递归执行 patch 方法挂载每一个 child 。注意，这里有对 child 做预处理的情况（属于编译优化的内容）。

可以看到，mountChildren 函数的第二个参数是 container，而我们调用 mountChildren 方法传入的第二个参数是在 mountElement 时创建的 DOM 节点，这就很好地建立了父子关系。

通过 patch，我们就可以构造完整的 DOM 树，完成组件的渲染。



处理完所有子节点后，最后通过 hostInsert 方法把创建的 DOM 元素节点挂载到 container 上，它在 Web 环境下这样定义：

```ts
// packages/runtime-dom/src/nodeOps.ts

function insert(child, parent, anchor) {
  if (anchor) {
    parent.insertBefore(child, anchor)
  }
  else {
    parent.appendChild(child)
  }
}
```

> 在 mountChildren 的时候递归执行的是 patch 函数，而不是 mountElement 函数，这是因为子节点可能有其他类型的 vnode，比如组件 vnode。
>
> 在真实开发场景中，嵌套组件的场景很多，前面我们举的 App 和 Hello 组件的例子就是嵌套组件的场景。组件 vnode 主要维护着组件的定义对象，组件上的各种 props，而组件本身是一个抽象节点，它自身的渲染其实是通过执行组件定义的 render 函数渲染生成的子树 vnode 来完成，然后再 patch 。通过这种递归的方式，无论组件的嵌套层级多深，都可以完成整个组件树的渲染。



## 流程图

![组件渲染流程](/Users/liuchang/Documents/markdown/学习/Vue3/assets/组件渲染流程.png)

