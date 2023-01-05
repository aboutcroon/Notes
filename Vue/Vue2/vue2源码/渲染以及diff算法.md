## 浏览器的渲染

当浏览器接收到一个`Html`文件时，`JS`引擎和浏览器的渲染引擎便开始工作了。从渲染引擎的角度，它首先会将`html`文件解析成一个`DOM`树，与此同时，浏览器将识别并加载`CSS`样式，并和`DOM`树一起合并为一个渲染树。有了渲染树后，渲染引擎将计算所有元素的位置信息，最后通过绘制，在屏幕上打印最终的内容。`JS`引擎和渲染引擎虽然是两个独立的线程，但是JS引擎却可以触发渲染引擎工作，当我们通过脚本去修改元素位置或外观时，`JS`引擎会利用`DOM`相关的`API`方法去操作`DOM`对象,此时渲染引擎变开始工作，渲染引擎会触发回流或者重绘。下面是回流重绘的两个概念：

- 回流： 当我们对`DOM`的修改引发了元素尺寸的变化时，浏览器需要重新计算元素的大小和位置，最后将重新计算的结果绘制出来，这个过程称为回流。
- 重绘： 当我们对`DOM`的修改只单纯改变元素的颜色时，浏览器此时并不需要重新计算元素的大小和位置，而只要重新绘制新样式。这个过程称为重绘。

**很显然回流比重绘更加耗费性能**。

通过了解浏览器基本的渲染机制，我们很容易联想到当不断的通过`JS`修改`DOM`时，不经意间会触发到渲染引擎的回流或者重绘，这个性能开销是非常巨大的。因此为了降低开销，我们需要做的是尽可能减少`DOM`操作。



## 虚拟 DOM

虚拟`DOM`就是为了解决频繁操作`DOM`引发性能问题的产物。`Virtual DOM` 是将页面的状态抽象为`JS`对象的形式，本质上是`JS`和真实`DOM`的中间层，当我们想用`JS`脚本大批量进行`DOM`操作时，会优先作用于`Virtual DOM`这个`JS`对象，最后通过对比将要改动的部分通知并更新到真实的`DOM`。尽管最终还是操作真实的`DOM`，但`Virtual DOM`可以将多个改动合并成一个批量的操作，从而减少 `DOM` 重排的次数，进而缩短了生成渲染树和绘制所花的时间。

例如：

```html
// 真实DOM
<div id="real">
  <span>dom</span>
</div>
```

```js
// 真实DOM对应的JS对象
{
    tag: 'div',
    data: {
        id: 'real'
    },
    children: [{
        tag: 'span',
        children: 'dom'
    }]
}
```



## VNode 构造函数

在 `vdom` 文件夹下有定义 VNode 构造函数的文件

```js
// src/core/vdom/vnode.js
const VNode = function VNode (tag,data,children,text,elm,context,componentOptions,asyncFactory) {
  this.tag = tag; // 标签
  this.data = dat  // 数据
  this.children = children // 子节点
  this.text = text
  ···
  ···
}
```

`Vnode`定义的属性差不多有20几个，显然用`Vnode`对象要比真实`DOM`对象描述的内容要简单得多，它只用来单纯描述节点的关键属性，例如标签名，数据，子节点等。并没有保留跟浏览器相关的`DOM`方法。除此之外，`Vnode`也会有其他的属性用来扩展`Vue`的灵活性。

`vnode.js` 中也定义了创建`Vnode`的相关方法。

```js
// src/core/vdom/vnode.js
// 创建注释vnode节点
var createEmptyVNode = function (text: string = '') {
  var node = new VNode()
  node.text = text // 具体的注释信息
  node.isComment = true // 是否是注释节点
  return node
}
```

```js
// src/core/vdom/vnode.js
// 创建文本vnode节点
function createTextVNode (val: string | number) {
    return new VNode(undefined, undefined, undefined, String(val))
}
```

```js
// src/core/vdom/vnode.js
// 克隆节点就是把一个已经存在的节点复制一份出来，它主要是为了做模板编译优化时使用
function cloneVNode (vnode: VNode): VNode {
  const cloned = new VNode(
    vnode.tag,
    vnode.data,
    vnode.children && vnode.children.slice(),
    vnode.text,
    vnode.elm,
    vnode.context,
    vnode.componentOptions,
    vnode.asyncFactory
  )
  cloned.ns = vnode.ns
  cloned.isStatic = vnode.isStatic
  cloned.key = vnode.key
  cloned.isComment = vnode.isComment
  cloned.fnContext = vnode.fnContext
  cloned.fnOptions = vnode.fnOptions
  cloned.fnScopeId = vnode.fnScopeId
  cloned.asyncMeta = vnode.asyncMeta
  cloned.isCloned = true
  return cloned
}
```

**注意：`cloneVnode`对`Vnode`的克隆只是一层浅拷贝，它不会对子节点进行深度克隆。**



## Virtual DOM 的创建

回顾一下挂载的流程，挂载的过程是调用`Vue`实例上`$mount`方法，而`$mount`的核心是`mountComponent`函数。如果我们传递的是`template`模板，模板会先经过编译器的解析，并最终根据不同平台生成对应代码，此时对应的就是将`with`语句封装好的`render`函数；如果传递的是`render`函数，则跳过模板编译过程，直接进入下一个阶段。

下一阶段是拿到`render`函数，调用`vm._render()`方法将`render`函数转化为`Virtual DOM`，并最终通过`vm._update()`方法将`Virtual DOM`渲染为真实的`DOM`节点。

```js
Vue.prototype.$mount = function(el, hydrating) {
  ···
  return mountComponent(this, el)
}

function mountComponent() {
  ···
  updateComponent = function () {
    vm._update(vm._render(), hydrating)
  }
}
```



## diff 算法

以这样一个列表为例：

```html
<ul>
  <li>1</li>
  <li>2</li>
</ul>
```

那么它的 `vnode` 也就是虚拟 dom 节点大概是这样的。

```js
{
  tag: 'ul',
  children: [
    { tag: 'li', children: [ { vnode: { text: '1' }}]  },
    { tag: 'li', children: [ { vnode: { text: '2' }}]  },
  ]
}
```

假设更新以后，我们把子节点的顺序调换了一下：

```js
{
  tag: 'ul',
  children: [
+   { tag: 'li', children: [ { vnode: { text: '2' }}]  },
+   { tag: 'li', children: [ { vnode: { text: '1' }}]  },
  ]
}
```

首先，响应式数据更新后，触发了 `渲染 Watcher` 的回调函数 `vm._update(vm._render())`去驱动视图更新，

`vm._render()` 其实生成的就是 `vnode`，而 `vm._update` 就会带着新的 `vnode` 去走触发 `__patch__` 过程。

直接看看 `ul` 这个 `vnode` 的 `patch` 过程的判断逻辑。

```js
// src/core/vdom/patch.js
return function patch (oldVnode, vnode, hydrating, removeOnly) {
  if (isUndef(vnode)) {
    if (isDef(oldVnode)) invokeDestroyHook(oldVnode)
    return
  }

  let isInitialPatch = false
  const insertedVnodeQueue = []

  if (isUndef(oldVnode)) {
    // empty mount (likely as component), create new root element
    isInitialPatch = true
    createElm(vnode, insertedVnodeQueue)
  } else {
    const isRealElement = isDef(oldVnode.nodeType)
    if (!isRealElement && sameVnode(oldVnode, vnode)) {
      // patch existing root node
      patchVnode(oldVnode, vnode, insertedVnodeQueue, null, null, removeOnly)
    } else {
      if (isRealElement) {
        // mounting to a real element
        // check if this is server-rendered content and if we can perform
        // a successful hydration.
        if (oldVnode.nodeType === 1 && oldVnode.hasAttribute(SSR_ATTR)) {
          oldVnode.removeAttribute(SSR_ATTR)
          hydrating = true
        }
        if (isTrue(hydrating)) {
          if (hydrate(oldVnode, vnode, insertedVnodeQueue)) {
            invokeInsertHook(vnode, insertedVnodeQueue, true)
            return oldVnode
          } else if (process.env.NODE_ENV !== 'production') {
            warn(
              'The client-side rendered virtual DOM tree is not matching ' +
              'server-rendered content. This is likely caused by incorrect ' +
              'HTML markup, for example nesting block-level elements inside ' +
              '<p>, or missing <tbody>. Bailing hydration and performing ' +
              'full client-side render.'
            )
          }
        }
        // either not server-rendered, or hydration failed.
        // create an empty node and replace it
        oldVnode = emptyNodeAt(oldVnode)
      }

      // replacing existing element
      const oldElm = oldVnode.elm
      const parentElm = nodeOps.parentNode(oldElm)

      // create new node
      createElm(
        vnode,
        insertedVnodeQueue,
        // extremely rare edge case: do not insert if old element is in a
        // leaving transition. Only happens when combining transition +
        // keep-alive + HOCs. (#4590)
        oldElm._leaveCb ? null : parentElm,
        nodeOps.nextSibling(oldElm)
      )

      // update parent placeholder node element, recursively
      if (isDef(vnode.parent)) {
        let ancestor = vnode.parent
        const patchable = isPatchable(vnode)
        while (ancestor) {
          for (let i = 0; i < cbs.destroy.length; ++i) {
            cbs.destroy[i](ancestor)
          }
          ancestor.elm = vnode.elm
          if (patchable) {
            for (let i = 0; i < cbs.create.length; ++i) {
              cbs.create[i](emptyNode, ancestor)
            }
            // #6513
            // invoke insert hooks that may have been merged by create hooks.
            // e.g. for directives that uses the "inserted" hook.
            const insert = ancestor.data.hook.insert
            if (insert.merged) {
              // start at index 1 to avoid re-invoking component mounted hook
              for (let i = 1; i < insert.fns.length; i++) {
                insert.fns[i]()
              }
            }
          } else {
            registerRef(ancestor)
          }
          ancestor = ancestor.parent
        }
      }

      // destroy old node
      if (isDef(parentElm)) {
        removeVnodes([oldVnode], 0, 0)
      } else if (isDef(oldVnode.tag)) {
        invokeDestroyHook(oldVnode)
      }
    }
  }

  invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)
  return vnode.elm
}
```



### 不是相同节点

`sameNode`为 false 的话，直接销毁旧的 `vnode`，渲染新的 `vnode`。这也解释了为什么 `diff` 是同层对比。

```JS
// src/core/vdom/patch.js
function sameVnode (a, b) {
  return (
    a.key === b.key && (
      (
        a.tag === b.tag &&
        a.isComment === b.isComment &&
        isDef(a.data) === isDef(b.data) &&
        sameInputType(a, b)
      )
    )
  )
}
```

它是用来判断节点是否可用的关键函数，可以看到，判断是否是 `sameVnode`，传递给节点的 `key` 是关键。

### 是相同节点，要尽可能的做节点的复用

是相同的节点，则会调用 `patchVnode` 方法

```js
// src/core/vdom/patch.js
function patchVnode (
  oldVnode,
  vnode,
  insertedVnodeQueue,
  ownerArray,
  index,
  emoveOnly
) {
  if (oldVnode === vnode) {
    return
  }

  if (isDef(vnode.elm) && isDef(ownerArray)) {
    // clone reused vnode
    vnode = ownerArray[index] = cloneVNode(vnode)
  }

  const elm = vnode.elm = oldVnode.elm

  if (isTrue(oldVnode.isAsyncPlaceholder)) {
    if (isDef(vnode.asyncFactory.resolved)) {
      hydrate(oldVnode.elm, vnode, insertedVnodeQueue)
    } else {
      vnode.isAsyncPlaceholder = true
    }
    return
  }

  // reuse element for static trees.
  // note we only do this if the vnode is cloned -
  // if the new node is not cloned it means the render functions have been
  // reset by the hot-reload-api and we need to do a proper re-render.
  if (isTrue(vnode.isStatic) &&
      isTrue(oldVnode.isStatic) &&
      vnode.key === oldVnode.key &&
      (isTrue(vnode.isCloned) || isTrue(vnode.isOnce))
     ) {
    vnode.componentInstance = oldVnode.componentInstance
    return
  }

  let i
  const data = vnode.data
  if (isDef(data) && isDef(i = data.hook) && isDef(i = i.prepatch)) {
    i(oldVnode, vnode)
  }

  const oldCh = oldVnode.children
  const ch = vnode.children
  if (isDef(data) && isPatchable(vnode)) {
    for (i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode)
    if (isDef(i = data.hook) && isDef(i = i.update)) i(oldVnode, vnode)
  }
  if (isUndef(vnode.text)) {
    if (isDef(oldCh) && isDef(ch)) {
      if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
    } else if (isDef(ch)) {
      if (process.env.NODE_ENV !== 'production') {
        checkDuplicateKeys(ch)
      }
      if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
      addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
    } else if (isDef(oldCh)) {
      removeVnodes(oldCh, 0, oldCh.length - 1)
    } else if (isDef(oldVnode.text)) {
      nodeOps.setTextContent(elm, '')
    }
  } else if (oldVnode.text !== vnode.text) {
    nodeOps.setTextContent(elm, vnode.text)
  }
  if (isDef(data)) {
    if (isDef(i = data.hook) && isDef(i = i.postpatch)) i(oldVnode, vnode)
  }
}
```

#### 如果新 vnode 是文字 vnode

就直接调用浏览器的 `dom api` 把节点的文字内容直接替换掉就好。

#### 如果新 vnode 不是文字 vnode

##### 如果有新 children 而没有旧 children

说明是新增 children，直接 `addVnodes` 添加新子节点。

##### 如果有旧 children 而没有新 children

说明是删除 children，直接 `removeVnodes` 删除旧子节点

##### 如果新旧 children 都存在

那么就是 `diff算法` 想要考察的最核心的点了，也就是新旧节点的 `diff` 过程。

```js
// 旧首节点
let oldStartIdx = 0
// 新首节点
let newStartIdx = 0
// 旧尾节点
let oldEndIdx = oldCh.length - 1
// 新尾节点
let newEndIdx = newCh.length - 1
```

这些变量分别指向`旧节点的首尾`、`新节点的首尾`。

根据这些指针，在一个 `while` 循环中不停的对新旧节点的两端的进行对比，直到没有节点可以对比。



然后我们接着进入 `diff` 过程，每一轮都是同样的对比，其中某一项命中了，就递归的进入 `patchVnode` 针对单个 `vnode` 进行的过程（如果这个 `vnode` 又有 `children`，那么还会来到这个 `diff children` 的过程 ）：

1. 旧首节点和新首节点用 `sameNode` 对比
2. 旧尾节点和新首节点用 `sameNode` 对比
3. 旧首节点和新尾节点用 `sameNode` 对比
4. 旧尾节点和新尾节点用 `sameNode` 对比
5. 如果以上逻辑都匹配不到，再把所有旧子节点的 `key` 做一个映射表，然后用新 `vnode` 的 `key` 去找出在旧节点中可以复用的位置

然后不停的把匹配到的指针向内部收缩，直到新旧节点有一端的指针相遇（说明这个端的节点都被patch过了）。

在指针相遇以后，还有两种情况：

1. 有新节点需要加入。
   如果更新完以后，`oldStartIdx > oldEndIdx`，说明旧节点都被 `patch` 完了，但是有可能还有新的节点没有被处理到。那么就会去新增子节点。
2. 有旧节点需要删除。
   如果新节点先 patch 完了，那么此时会走 `newStartIdx > newEndIdx` 的逻辑，那么就会去删除多余的旧子节点。



整个流程如图：

![patch流程](https://raw.githubusercontent.com/aboutcroon/Notes/main/Vue/Vue2/vue2%E6%BA%90%E7%A0%81/assets/patch%20process.png)



第一步：

![diff1](https://raw.githubusercontent.com/aboutcroon/Notes/main/Vue/Vue2/vue2%E6%BA%90%E7%A0%81/assets/diff1.png)



第二步：

![diff2](https://raw.githubusercontent.com/aboutcroon/Notes/main/Vue/Vue2/vue2%E6%BA%90%E7%A0%81/assets/diff2.png)



第三步：

![diff3](https://raw.githubusercontent.com/aboutcroon/Notes/main/Vue/Vue2/vue2%E6%BA%90%E7%A0%81/assets/diff3.png)



第四步：

![diff4](https://raw.githubusercontent.com/aboutcroon/Notes/main/Vue/Vue2/vue2%E6%BA%90%E7%A0%81/assets/diff4.png)



第五步：

![diff5](https://raw.githubusercontent.com/aboutcroon/Notes/main/Vue/Vue2/vue2%E6%BA%90%E7%A0%81/assets/diff5.png)



第六步：

![diff6](https://raw.githubusercontent.com/aboutcroon/Notes/main/Vue/Vue2/vue2%E6%BA%90%E7%A0%81/assets/diff6.png)



## 为什么不要以 index 作为 key？

### 影响节点复用，造成性能的消耗

来看这个场景：

```vue
<div id="app">
  <ul>
    <item
      :key="index"
      v-for="(num, index) in nums"
      :num="num"
      :class="`item${num}`"
    />
  </ul>
  <button @click="change">改变</button>
</div>
<script>
  const vm = new Vue({
    name: "parent",
    el: "#app",
    data: {
      nums: [1, 2, 3]
    },
    methods: {
      change () {
        this.nums.reverse()
      }
    },
    components: {
      item: {
        props: ["num"],
        template: `<div>{{ num }}</div>`,
        name: "child"
      }
    }
  })
</script>
```

初始时：

```js
[
  {
    tag: "item",
    key: 0,
    props: {
      num: 1
    }
  },
  {
    tag: "item",
    key: 1,
    props: {
      num: 2
    }
  },
  {
    tag: "item",
    key: 2,
    props: {
      num: 3
    }
  }
]
```

reverse 之后

```js
[
  {
    tag: "item",
    key: 0,
    props: {
+     num: 3
    }
  },
  {
    tag: "item",
    key: 1,
    props: {
+     num: 2
    }
  },
  {
    tag: "item",
    key: 2,
    props: {
+     num: 1
    }
  }
]
```

key的顺序没变，传入的值完全变了。这会导致一个什么问题？

本来按照最合理的逻辑来说，`旧的第一个vnode` 是应该直接完全复用 `新的第三个vnode`的，因为它们本来就应该是同一个vnode，自然所有的属性都是相同的。

但是在进行子节点的 `diff` 过程中，会在 `旧首节点和新首节点用 `sameNode` 对比。` 这一步命中逻辑，因为现在`新旧两次首部节点` 的 `key` 都是 `0`了，然后把旧的节点中的第一个 `vnode` 和 新的节点中的第一个 `vnode` 进行 `patchVnode` 操作。在进行 `patchVnode` 的时候，会去检查 `props` 有没有变更，如果有的话，会通过 `_props.num = 3` 这样的逻辑去更新这个响应式的值，触发 `dep.notify`，触发子组件视图的重新渲染等一套很重的逻辑（props 的更新触发重新渲染）。

而这些重新渲染的操作，都可以通过直接复用 `第三个vnode` 来避免，因为我们使用 `index` 作为 `key`，而导致所有的优化失效了。



### 节点删除问题

另外，除了会导致性能损耗以外，在`删除子节点`的场景下还会造成更严重的错误，

有下列场景：

```vue
<body>
  <div id="app">
    <ul>
      <li v-for="(value, index) in arr" :key="index">
        <test />
      </li>
    </ul>
    <button @click="handleDelete">delete</button>
  </div>
  </div>
</body>
<script>
  new Vue({
    name: "App",
    el: '#app',
    data() {
      return {
        arr: [1, 2, 3]
      };
    },
    methods: {
      handleDelete() {
        this.arr.splice(0, 1);
      }
    },
    components: {
      test: {
        template: "<li>{{ Math.random() }}</li>"
      }
    }
  })
</script>
```

初始时的 vnode 列表：

```js
[
  {
    tag: "li",
    key: 0,
    // 这里子组件对应的是第一个 假设子组件的text是1
  },
  {
    tag: "li",
    key: 1,
    // 这里子组件对应的是第二个 假设子组件的text是2
  },
  {
    tag: "li",
    key: 2,
    // 这里子组件对应的是第三个 假设子组件的text是3
  }
]
```

有一个细节需要注意，Vue 对于组件的 `diff` 是不关心子组件内部实现的，它只会看你在模板上声明的传递给子组件的一些属性是否有更新（比如声明的 `props`、`listeners`等属性）。

也就是和v-for平级的那部分（回顾一下判断 `sameNode` 的时候，只会判断`key`、 `tag`、`是否有data的存在（不关心内部具体的值）`、`是否是注释节点`、`是否是相同的input type`）来判断是否可以复用这个节点。

```html
<li v-for="(value, index) in arr" :key="index"> // 这里声明的属性
  <test />
</li>
```

那么，点击删除子元素后，`vnode 列表` 变成：

```js
[
  // 第一个被删了
  {
    tag: "li",
    key: 0,
    // 这里其实上一轮子组件对应的第二个 子组件的text是2
  },
  {
    tag: "li",
    key: 1,
    // 这里其实子组件对应的第三个 假设子组件的text是3
  }
]
```

虽然在注释里我们自己清楚的知道，第一个 `vnode` 被删除了，但是对于 Vue 来说，它是感知不到子组件里面到底是什么样的实现（它不会深入子组件去对比文本内容），那么这时候 Vue 会怎么 `patch` 呢？

由于对应的 `key`使用了 `index`导致的错乱，它会把

1. `原来的第一个节点text: 1`直接复用。
2. `原来的第二个节点text: 2`直接复用。
3. 然后发现新节点里少了一个，直接把多出来的第三个节点`text: 3` 丢掉。

所以，我们本应该把 `text: 1`节点删掉，然后`text: 2`、`text: 3` 节点复用，就变成了错误的把 `text: 3` 节点给删掉了。



## 为什么不要用随机数作为 key？

同理，由上述内容可以得到，如果是随机数作为 key，那么 v-for 的数组一变化，每次的 key 都会变成全新的随机数。

上面说到，`diff` 子节点的首尾对比如果都没有命中，就会进入 `key` 的详细对比过程，简单来说，就是利用旧节点的 `key -> index` 的关系建立一个 `map` 映射表，然后用新节点的 `key` 去匹配，如果没找到的话，就会调用 `createElm` 方法 **重新建立** 一个新节点。

这个更新过程也就是：`123` -> 前面重新创建三个子组件 -> `321123` -> 删除、销毁后面三个子组件 -> `321`。

也就是每次都会创建新的组件和销毁旧组件，成本颇大，本来仅仅是对组件移动位置就可以完成更新了。

