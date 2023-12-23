组件是 Vue 中一个非常重要的概念，因为整个 Vue 应用的页面都是通过组件渲染来实现的。那么 Vue 内部是如何将一个组件渲染成真实 DOM 的呢？这就是本文会深入分析的地方。

## 组件的介绍

组件是一个抽象的概念，它是对一棵 DOM 树的抽象。

一个组件可以通过“模板加对象描述”的方式创建。

```html
<template>
  <div>
    <p>Hello World</p>
  </div>
</template>
<script>
  // 对象描述
</script>
```

## 初始化应用

首先，整个组件树是由根组件开始渲染的，所以我们需要从根组件开始分析

看一下 Vue2 与 Vue3 分别是如何从根组件开始初始化整个应用的

```js
// 在 Vue.js 2.x 中，初始化一个应用的方式如下
import Vue from 'vue'
import App from './App'
const app = new Vue({
  render: h => h(App)
})
app.$mount('#app')
```

```js
// 在 Vue.js 3.0 中，初始化一个应用的方式如下
import { createApp } from 'vue'
import App from './app'
const app = createApp(App)
app.mount('#app')
```

可以看到，它们语法上有一定差别，但是本质上都是把 App 组件挂载到 id 为 app 的 DOM 节点上。接下来我们就来着重分析一下 Vue3 是如何初始化应用的。

在 Vue3 中，初始化应用是从 createApp 开始，顾名思义这是一个入口函数，它是 Vue3 对外暴露的一个函数，我们来看一下它的内部实现：

```js
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

整体分为两个部分：

1. 创建 app 对象
2. 重写 app.mount 方法

### 创建 app 对象

可以看到，app 对象是通过 `const app = ensureRenderer().createApp(...args)` 创建的，ensureRenderer 函数是用来创建一个渲染器对象的，它的内部代码如下：

```js
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
function createRenderer(options) {
  return baseCreateRenderer(options)
}
function baseCreateRenderer(options) {
  function render(vnode, container) {
    // 组件渲染的核心逻辑
  }

  return {
    render,
    createApp: createAppAPI(render)
  }
}
function createAppAPI(render) {
  // createApp createApp 方法接受的两个参数：根组件的对象和 prop
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

代码中使用了 ensureRenderer() 来延时创建渲染器。这样做的好处是当用户只依赖响应式包的时候，就不会创建渲染器，因此可以通过 tree-shaking 的方式移除核心渲染逻辑相关的代码。

我们知道，Vue3 中的渲染器是支持跨平台的，所以不同平台可以自定义专属的渲染器，上述代码中的 rendererOptions 即为我们可以传入的自定义渲染器的参数。

渲染器其实就是 `包含平台渲染核心逻辑的 JS 对象`，这个 JS 对象就是：

```js
{
  render,
  createApp: createAppAPI(render)
}
```

里面包括初始化应用的 createApp 方法，包括了 render 函数（实际用来渲染的方法）。

渲染器内部的 createApp 方法接受 rootComponent 和 rootProps 两个参数，我们通常就会将 `App.vue` 作为 rootComponent 参数传入。

createApp 内部创建了一个 app 对象，它会提供 mount 方法，这个方法就是用来挂载组件的（里面也包括了渲染的方法 render，所以会先进行渲染再挂载）。

在整个 app 对象创建过程中，Vue3 利用了闭包和函数柯里化的技巧，很好地实现了参数保留。比如，在执行 app.mount  的时候，并不需要传入渲染器 render，这是因为在执行 createAppAPI 的时候渲染器 render 参数已经被保留下来了。

### 重写 app.mount 方法

为什么要重写 app.mount 方法呢？为什么不把相关逻辑直接放在 app 对象的 mount 方法内部来实现呢？

这是因为 Vue 不仅仅是为 Web 平台服务，它的目标是支持跨平台渲染，而 createApp 函数内部的 app.mount 方法是一个标准的可跨平台的组件渲染流程，这里面的代码不应该包含任何特定平台相关的逻辑。我们需要根据不同的平台在外面去重写这个 mount 方法。

标准的可跨平台的组件渲染流程：

```js
mount(rootContainer) {
  // 创建根组件的 vnode
  const vnode = createVNode(rootComponent, rootProps)
  // 利用渲染器渲染 vnode
  render(vnode, rootContainer)
  app._container = rootContainer
  return vnode.component.proxy
}
```

web 平台下对 mount 方法的重写：

```js
app.mount = (containerOrSelector) => {
  // 标准化容器
  const container = normalizeContainer(containerOrSelector)
  if (!container)
    return
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

web 平台中，首先会通过 normalizeContainer 标准化容器。normalizeContainer 内部主要是为了兼容 Vue2 的写法，所以这一步可以传字符串选择器或者 DOM 对象，但如果是字符串选择器，就需要把它转成 DOM 对象，作为最终挂载的容器。

```js
function normalizeContainer(
  container: Element | ShadowRoot | string
): Element | null {
  if (isString(container)) {
    const res = document.querySelector(container)
    if (__DEV__ && !res) {
      warn(
        `Failed to mount app: mount target selector "${container}" returned null.`
      )
    }
    return res
  }
  if (
    __DEV__ &&
    window.ShadowRoot &&
    container instanceof window.ShadowRoot &&
    container.mode === 'closed'
  ) {
    warn(
      `mounting on a ShadowRoot with \`{mode: "closed"}\` may lead to unpredictable bugs`
    )
  }
  return container as any
}
```

然后做一个 if 判断，如果组件对象没有定义 render 函数和 template 模板，则取容器的 innerHTML 作为组件模板内容；接着在挂载前清空容器内容，最终再调用实际的 app.mount 的方法走标准的组件渲染流程。

从 app.mount 开始，才算真正进入组件渲染流程。

## 核心渲染流程

核心的组件渲染流程分为以下两个步骤：

1. 创建 vnode
2. 渲染 vnode

### 创建 vnode

首先，会创建 vnode。

一个普通 button 按钮的 vnode 如下：

```js
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

一个组件节点的 vnode 如下：

```js
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

可以看到，组件节点的 type 是一个对象，而 `这个对象其实就是模版编译阶段生成的每个组件的 render 函数执行之后生成的对象`。

所以，当我们引入 `App.vue` 的时候，就相当于引入了该组件对象，然后组件对象中又会引入子组件的对象，一层套一层。

rootComponent 参数就是这么一个对象，我们会将这个对象传入 createVNode，创建一个最终的 vnode 对象去进行渲染

```js
const vnode = createVNode(rootComponent, rootProps)
```

```js
function createVNode(type, props = null
,children = null) {
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

createVNode 内部会处理组件对象的 props，然后添加 shapeFlag 类型，最后标准化子节点。

### 渲染 vnode

创建好 App.vue 组件的 vnode 之后，mount 方法中会通过 render 渲染函数来渲染 vnode，并挂载到 rootContainer 上

```js
render(vnode, rootContainer)
```

render 函数中即为主体的渲染流程，我们看下它的具体实现

```js

const render = (vnode, container) => {
  if (vnode == null) {
    // 销毁组件
    if (container._vnode) {
      unmount(container._vnode, null, null, true)
    }
  } else {
    // 创建或者更新组件
    patch(container._vnode || null, vnode, container)
  }
  // 缓存 vnode 节点，表示已经渲染
  container._vnode = vnode
}
```

可以看到，当 vnode 不存在时，则执行组件销毁；当 vnode 存在时，则执行创建或者更新组件的逻辑 patch。

patch 函数就是整个组件渲染的入口分发函数：

```js
const patch = (n1, n2, container, anchor = null, parentComponent = null, parentSuspense = null, isSVG = false, optimized = false) => {
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
      break
    case Comment:
      // 处理注释节点
      break
    case Static:
      // 处理静态节点
      break
    case Fragment:
      // 处理 Fragment 元素
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

比较重要的是前三个参数：

1. 第一个参数 n1 表示旧的 vnode，当 n1 为 null 的时候，表示是一次挂载的过程；n1 不为 null 时，则是一次更新的过程
2. 第二个参数 n2 表示新的 vnode 节点，后续会根据这个 vnode 类型执行不同的处理逻辑
3. 第三个参数 container 表示 DOM 容器，也就是 vnode 渲染生成 DOM 后，会挂载到 container 下面

接下来进入主体流程，这里主要分析对组件的处理与对普通 DOM 元素的处理

#### 对组件的处理

组件的处理入口是 processComponent 函数

```js
const processComponent = (n1, n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized) => {
  if (n1 == null) {
   // 挂载组件
   mountComponent(n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized)
  }
  else {
    // 更新组件
    updateComponent(n1, n2, parentComponent, optimized)
  }
}
```

如果 n1 为 null，则执行首次渲染，否则执行更新组件的流程。

本文主要分析初次渲染的过程，我们看一下 mountComponent 函数

```js
const mountComponent = (initialVNode, container, anchor, parentComponent, parentSuspense, isSVG, optimized) => {
  // 创建组件实例
  const instance = (initialVNode.component = createComponentInstance(initialVNode, parentComponent, parentSuspense))
  // 设置组件实例
  setupComponent(instance)
  // 设置并运行带副作用的渲染函数
  setupRenderEffect(instance, initialVNode, container, anchor, parentSuspense, isSVG, optimized)
}
```

函数主要分为三个部分：

1. 创建组件实例 instance 对象
2. 设置组件实例
3. 设置并运行带副作用的渲染函数 setupRenderEffect

创建组件实例很简单，就是创建一个我们日常见到的 instance 对象。这里不展开讲。

接下来是设置组件实例，这一步主要是维护组件的上下文，包括对 props、插槽，以及其他实例的属性的初始化处理。注意，setup 函数就是在这一步当中被执行的。

上面两步对组件的创建与处理，会在下一篇文章中详细介绍。

组件实例与组件内数据处理完毕后，然后是第三步，这一步则是接下来渲染流程的主体。

设置并运行带副作用的渲染函数 setupRenderEffect

```js
const setupRenderEffect = (instance, initialVNode, container, anchor, parentSuspense, isSVG, optimized) => {
  // 创建响应式的副作用渲染函数
  instance.update = effect(function componentEffect() {
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
  }, prodEffectOptions)
}
```

这里会利用响应式库的 effect 函数创建一个副作用渲染函数 componentEffect，这个函数就是我们组件数据变化之后重新执行的副作用渲染函数。

> 在 componentEffect 副作用渲染函数内部也会判断这是一次初始渲染还是组件更新，所以不用担心之后数据变化时重新渲染逻辑的执行

componentEffect 函数内部，会判断是否是 isMounted，否则会执行更新逻辑。

如果尚未 Mounted，则是初次渲染，首先创建 subTree，subTree 就是组件 template 下包含的整个树的 vnode，也就是子树 vnode。

注意，在这一步，renderComponentRoot 函数内部会执行 render 函数创建整个组件树内部的 vnode，把这个 vnode 再经过内部一层标准化，就得到了该函数的返回结果：子树 vnode。

> ❗️
>
> 设置组件实例 setupComponent(instance) 这一步中，将 setup 中的数据与模版已经建立起了联系，并且完成了响应式的监听。所以组件的 render 函数执行时，相当于初次读取 setup 中的变量值，所以会触发 get 收集依赖。
>
> 之后当我们组件数据变化后，就会触发 set 通知到所有的副作用函数的重新执行，然后再执行副作用渲染函数 componentEffect，函数内部再重新执行组件的 render 函数，此时模版读到的值就已经变成新的值了。

我们来看一下 renderComponentRoot 函数的内部：

```js
// 如果是有状态的组件
if (vnode.shapeFlag & ShapeFlags.STATEFUL_COMPONENT) {
  // withProxy is a proxy with a different `has` trap only for
  // runtime-compiled render functions using `with` block.
  const proxyToUse = withProxy || proxy
  result = normalizeVNode(
    render!.call(
      proxyToUse,
      proxyToUse!,
      renderCache,
      props,
      setupState,
      data,
      ctx
    )
  )
  fallthroughAttrs = attrs
}
```

执行组件的 render 函数，并且将其使用传入 normalizeVNode：

```js
export function normalizeVNode(child: VNodeChild): VNode {
  if (child == null || typeof child === 'boolean') {
    // empty placeholder
    return createVNode(Comment)
  } else if (isArray(child)) {
    // fragment
    return createVNode(
      Fragment, // 最终都是从这个 createVNode 开始，将每个 vnode 都作为 Fragment 对象来进行操作，操作完之后再 insert 到真实 DOM 中
      null,
      // #3666, avoid reference pollution when reusing vnode
      child.slice()
    )
  } else if (typeof child === 'object') {
    // already vnode, this should be the most common since compiled templates
    // always produce all-vnode children arrays
    return cloneIfMounted(child)
  } else {
    // strings and numbers
    return createVNode(Text, null, String(child))
  }
}
```

当传入的是组件 vnode 时，会直接通过 createVNode 创建 Fragment 类型的 vnode，这样后面就可以将每个 vnode 都作为 Fragment 对象来进行操作，操作完之后再 insert 到真实 DOM 中，提高性能。

这里的 createVNode 就对应着渲染 vnode 之前的创建 vnode 步骤一样，只是之前是根组件的 createVNode，这里是根组件下面的子组件的 createVNode。

至此 subTree 的生成介绍完毕。

接下来，生成 subTree 之后，会继续调用 patch 函数，把 subTree 挂载到 container 中。所以，我们又回到了 patch 函数（所以它是整个渲染做分发的入口）。

#### 对普通 DOM 元素的处理

然后就又是区分对组件的处理与对普通 DOM 元素的处理了。

接下来 patch 到子节点，是普通元素时，就走 processElement 的逻辑

```js
const processElement = (n1, n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized) => {
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
```

然后进入 mountElement 渲染元素节点

```js
const mountElement = (vnode, container, anchor, parentComponent, parentSuspense, isSVG, optimized) => {
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

这里主要分为 4 个步骤：

1. 创建 DOM 元素节点

   首先是执行 hostCreateElement 函数，这个函数是一个通用的创建逻辑，不同平台的实现方式不同。在 web 平台它的作用就变成了 createElement 函数。

   > 如果是其他平台比如 Weex，hostCreateElement 方法就不再是操作 DOM，而是平台相关的 API 了，这些平台相关的方法是在创建渲染器阶段作为参数传入的

   ```js
   function createElement(tag, isSVG, is) {
     isSVG ? document.createElementNS(svgNS, tag)
       : document.createElement(tag, is ? { is } : undefined)
   }
   ```

   可以看到，这个方法内部就是调用了底层的 DOM API document.createElement 来创建元素，所以在 Vue 的底层终究是要操作 DOM 的。操作 DOM 才是实际的渲染页面。

2. 处理 props

   如果有 props 的话，给这个 DOM 节点添加相关的 class、style、event 等属性，并做相关的处理。

3. 处理 children

   如果子节点是文本，则直接执行 hostSetElementText。同理，在 web 平台实际上执行了 setElementText，将文本设置为节点内容

   ```js
   function setElementText(el, text) {
     el.textContent = text
   }
   ```

   如果子节点是数组，那么执行 mountChildren 方法

   ```js
   const mountChildren = (children, container, anchor, parentComponent, parentSuspense, isSVG, optimized, start = 0) => {
     for (let i = start; i < children.length; i++) {
       // 预处理 child
       const child = (children[i] = optimized
         ? cloneIfMounted(children[i])
         : normalizeVNode(children[i]))
       // 递归 patch 挂载 child
       patch(null, child, container, anchor, parentComponent, parentSuspense, isSVG, optimized)
     }
   }
   ```

   这里的预处理 child 主要是做编译优化。

   接着会遍历 children 获取到每一个 child，然后递归执行 patch 方法将每一个 child 挂载到父节点上（此时传入的参数 container 是父节点），直到递归的终点，也就是又执行到文本节点 setElementText 为止。

   > 这里执行的是 patch 而不是 mountElement，因为普通元素里也会嵌套组件元素，所以每次都需要重回 patch 逻辑慢慢往下走。

4. 挂载 DOM 元素到 container 上

   处理完所有子节点后，最后通过 hostInsert 方法把创建的 DOM 元素节点挂载到 container 上，在 Web 环境下是 insert 方法

   ```js
   function insert(child, parent, anchor) {
     if (anchor) {
       parent.insertBefore(child, anchor)
     }
     else {
       parent.appendChild(child)
     }
   }
   ```

   这里会做一个 if 判断，如果有参考元素 anchor，就执行 parent.insertBefore ，否则执行 parent.appendChild 来把 child 添加到 parent 下，完成节点的挂载。

   因为 insert 的执行是在处理子节点后，所以挂载的顺序是先子节点，后父节点，最终挂载到最外层的容器上。

   所以，渲染是节点树的先序遍历，挂载是节点树的后序遍历。

## 总结

整个流程梳理下来可以通过一张图总结

![组件渲染流程](https://github.com/edwineo/Notes/blob/main/Vue/Vue3/article/assets/%E7%BB%84%E4%BB%B6%E6%B8%B2%E6%9F%93%E6%B5%81%E7%A8%8B.png?raw=true)

 这里挂载 vnode 的函数就是 patch，每次都是经过 patch 的分发之后，再根据节点的类型执行不同类型的节点处理函数。往复循环递归每个组件，最终就完成了整个页面的渲染。

