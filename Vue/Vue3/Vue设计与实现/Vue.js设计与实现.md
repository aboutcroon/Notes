# 框架设计

## 命令式与声明式

从范式上来看，视图层框架通常分为**命令式**和**声明式**。



命令式：关注过程，自然语言描述能够与代码产生一一对应的关系，代码本身描述的是“做事的过程”。

```js
const div = document.querySelector('#app') // 获取div
div.innerText = 'hello world' // 设置文本内容
div.addEventListener('click', () => { alert('ok') }) // 绑定点击事件
```



声明式：更加关注结果，我们提供的是一个“结果”，而怎样去实现这个“结果”，我们并不关心。

```html
<div @click="() => alert('ok')">hello world</div>
```



因此，我们知道，Vue.js的内部实现一定是命令式的，而暴露给用户的却更加声明式。



### 性能与可维护性

**声明式代码的性能不优于命令式的代码**



假设现在我们要将div标签的文本修改为hello vue3

用命令式代码可以直接实现

```js
div.textContent = 'hello vue3'
```

现在思考一下，还有没有其他办法比上面这句代码的性能更好？答案是“没有”

理论上，命令式代码可以做到极致的性能优化，因为我们明确的知道哪些发生了变更，只做必要的修改就行了。

但是，声明式代码不一定能做到这一点，因为它描述的是结果：

```html
<!-- 之前：-->
<div>hello world</div>
<!-- 之后：-->
<div>hello vue3</div>
```

对于框架来说，为了实现最优的更新性能，它需要找到前后的差异并只更新变化的地方，但是最终完成这次更新的代码仍然是：

```js
div.textContent = 'hello vue3'
```



如果我们把直接修改的性能消耗定义为A，把找出差异的性能消耗定义为B，那么有：

命令式代码的更新性能消耗 = A

声明式代码的更新性能消耗 = B + A

因此，最理想的情况是，当找出差异的性能消耗为0时，声明式代码与命令式代码的性能相同，但是无法做到超越。



既然性能方面命令式代码是更好的选择，那么为什么vue.js要选择声明式的设计方案呢？

原因就在于声明式代码的可维护性更强。

在采用命令式代码开发的时候，我们需要维护实现目标的整个过程，包括要手动完成DOM元素的创建、更新、删除等工作。而声明式代码展示的就是我们要的**结果**，看上去更加直观，至于实现的过程，并不需要我们关心，vue.js都帮我们封装好了。



这体现了在框架设计上要做出的关于可维护性与性能之间的权衡。在采用声明式提升可维护性的同时，性能就会有一定的损失，框架设计者要做的就是：**在保持可维护性的同时，让性能损失最小化。**



### 虚拟DOM的性能

前面讲到，如果我们能够最小化**找出差异的性能消耗**，就可以让声明式代码的性能无限接近命令式代码的性能。

**而所谓的虚拟DOM，就是为了最小化找出差异的性能消耗而出现的。**



我们知道，采用虚拟DOM的更新技术的性能理论上不可能比原生JS操作DOM更高。这里所说的原生JS实际上指的是类似document.createElement之类的DOM操作方法，并不包括 `innerHTML`，因为它比较特殊，需要单独讨论。



接下来我们来比较一下，**使用innerHTML来操作页面** 和 **虚拟DOM**，这两个性能如何。



#### 创建页面

对于innerHTML来说，为了创建页面，我们需要构造一段HTML字符串：

```js
const html = `
	<div><span>...</span></div>
`
```

接着将该字符串赋值给DOM元素的innerHTML属性：

```js
div.innerHTML = html
```

然而这句话远没有看上去那么简单，为了渲染出页面，我们需要**把字符串解析成DOM树**，这是一个DOM层面的计算，涉及DOM的运算远比JS层面的计算性能差。

所以，性能公式为：`innerHTML创建页面的性能 = HTML字符串拼接的计算量 + innerHTML的DOM计算量`



对于虚拟DOM，创建页面的过程分为两步：

1. 创建JS对象，这个对象可以理解为真实DOM的描述
2. 递归的遍历虚拟DOM树并创建真实DOM

所以，性能公式为：`虚拟DOM创建页面的性能 = 创建JS对象的计算量 + 创建真实DOM的计算量`



可见，在创建页面时，无论是纯JS层面的计算，还是DOM层面的计算，其实两者差距不大



#### 更新页面

我们通过下面表格来对比：

|          | 虚拟DOM               | innerHTML                     |
| -------- | --------------------- | ----------------------------- |
| 纯JS运算 | 创建新的JS对象 + diff | 渲染HTML字符串                |
| DOM运算  | 必要的DOM更新         | 销毁所有旧DOM + 新建所有新DOM |

可以发现，在更新页面时，虚拟DOM在JS层面的运算要多出一个diff的性能消耗，然而它毕竟也是JS层面的运算，所以不会产生数量级的差异。但innerHTML在DOM层面的运算需要全量的更新，销毁再重建，这时虚拟DOM的优势就体现出来了。

另外，当更新页面时，无论页面多大，都只会更新变化的内容，而对于innerHTML来说，页面越大，就意味着更新时的性能消耗越大。

|          | 虚拟DOM          | innerHTML      |
| -------- | ---------------- | -------------- |
| 性能因素 | 与数据变化量相关 | 与模版大小相关 |



### 性能比较

粗略的比较一下innerHTML、虚拟DOM、原生JS（createElement等方法）在更新页面时的性能：

性能：innerHTML < 虚拟DOM < 原生JS

| innerHTML    | 虚拟DOM    | 原生JS     |
| ------------ | ---------- | ---------- |
| 心智负担中等 | 心智负担小 | 心智负担大 |
| 性能差       | 性能不错   | 性能高     |
|              | 可维护性强 | 可维护性差 |

可以看到，因为虚拟DOM是声明式的，因此心智负担小，可维护性强，性能虽然比不上极致优化的原生JS，但是在保证心智负担和可维护性的前提下相当不错。



至此，我们有必要思考一下：有没有办法做到，既声明式的描述UI，又具备原生JS的性能呢？我们在之后的章节继续讨论。



## 运行时和编译时

当设计一个框架的时候，我们有三种选择：

1. 纯运行时的
2. 运行时 + 编译时的
3. 纯编译时的

至于选择哪种，这需要我们根据目标框架的特征，以及对框架的期望，做出合适的决策。



### 纯运行时

假设我们设计了一个框架，它提供一个Render函数，用户可以为该函数提供一个树型结构的数据对象，然后Render函数会根据该对象递归的将数据渲染成DOM元素，我们规定树型结构的数据对象如下：

```js
const obj = {
  tag: 'div',
  children: [
    { tag: 'span', children: 'hello world' }
  ]
}
```

Render函数的实现：

```js
function Render (obj, root) {
  const el = document.createElement(obj, tag)
  if (typeof obj.children === 'string') {
    const text = document.createTextNode(obj.children)
    el.appendChild(text)
  } else if (obj.children) {
    // 数组，递归调用Render，使用el作为root参数
    obj.children.forEach(child => Render(child, el))
  }
  
  // 将元素添加到root
  root.appendChild(el)
}
```

有了这个函数，用户就可以这样使用它：

```js
const obj = {
  tag: 'div',
  children: [
    { tag: 'span', children: 'hello world' }
  ]
}
// 渲染到body下
Render(obj, document.body)
```

在浏览器中运行上述代码，就可以看到我们预期的内容。



这就是纯运行时的框架，可以发现，用户在使用它渲染内容时，直接为Render函数提供了一个树型结构的数据对象。**这里面不涉及任何额外的步骤，用户也不需要学习额外的知识。**



### 运行时 + 编译时

如果有一天，你的用户抱怨说：“手写树型结构的数据对象太麻烦了，而且不直观，能不能支持用类似于HTML标签的方式描述树型结构的数据对象呢？”

为了满足用户的需求，我们可以引入**编译**的手段，把HTML标签编译成树型结构的数据对象，这样不就可以继续使用Render函数了吗？

例如

```html
<div>
  <span>hello world</span>
</div>
```

编译成

```js
const obj = {
  tag: 'div',
  children: [
    { tag: 'span', children: 'hello world' }
  ]
}
```

为此，我们编写了一个叫做compiler的程序，它的作用就是把HTML字符串编译成树型结构的数据对象，于是交付给用户去用了，用户可以这样使用：

```js
const html = `
	<div><span>hello world</span></div>
`
// 编译得到HTML树型结构的数据对象
const obj = compiler(html)
// 进行渲染
Render(obj, document.body)
```

这时我们的框架就变成了一个**运行时 + 编译时**的框架。



### 纯编译时

运行时编译的框架，是代码运行的时候才开始编译，而这会产生一定的性能开销。因此我们也可以在构建的时候就执行compiler程序将用户提供的内容编译好，等到运行时就无需编译了，这对性能是非常友好的。

同时，既然编译器可以把HTML字符串编译成数据对象，那么肯定也能直接编译成命令式的代码

例如

```html
<div>
  <span>hello world</span>
</div>
```

编译成

```js
const div = document.createElement('div')
const span = document.createElement('span')
span.innerText = 'hello world'
div.appendChild(span)
document.body.appendChild(div)
```

这样我们只需要一个compiler函数就可以了，连Render都不需要了。其实这就变成了一个**纯编译时**的框架，因为我们不支持任何运行时的内容。用户的代码通过编译器编译后才能运行。



### 优势比较

这几个方案各有利弊。

首先是纯运行时的框架，由于它没有编译过程，因此我们没办法分析用户提供的内容。

但如果加入编译步骤，那就大不一样了，我们可以分析用户提供的内容，看看哪些内容未来可能会改变，哪些内容永远不会改变，这样我们就可以在编译的时候提取这些信息，然后将其传递给Render函数，Render函数得到这些信息之后就可以做进一步的优化了。

假如我们的框架是纯编译时的，那么它也可以分析用户提供的内容。由于不需要任何运行时，而是直接编译成可执行的JS代码，因此性能可能会更好，但是这种做法有损灵活性，即用户提供的内容必须编译后才能用。



Vue.js 3 仍然保持了运行时 + 编译时的架构，在保持灵活性的基础上能够尽可能的去优化。

等后面讲解编译优化相关的内容时，你会看到Vue.js 3在保留运行时的情况下，其性能甚至不输纯编译时的框架。



# Vue3的设计思路

# 响应式系统



# 响应式系统的作用与实现

## 响应式数据与副作用函数

副作用函数也就是会产生副作用的函数，例如：

```js
function effect () {
  document.body.innerText = 'hello world'
}
```



假设在一个副作用函数中读取了某个对象的属性：

```js
const obj = { text: 'hello world' }
function effect () {
  document.body.innerText = obj.text
}
```

当`obj.text`的值发生变化时，我们希望副作用函数`effect`会重新执行，如果能实现这个目标，那么对象obj就是响应式数据。



## 响应式数据的基本实现

如何才能让obj变成响应式数据呢？通过观察我们发现两点线索：

1. 当副作用函数effect执行时，会触发字段obj.text的`读取`操作
2. 当修改obj.text的值时，会触发字段obj.text的`设置`操作

如果我们能拦截一个对象的读取和设置操作，事情就变得简单了。



当读取字段obj.text时，我们可以把副作用函数effect存储到一个“桶”里面

![4-1](https://raw.githubusercontent.com/aboutcroon/Notes/main/Vue/Vue3/Vue%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0/assets/4-1.png)

接着，当设置obj.text时，再把副作用函数从“桶”里取出并执行即可

![4-2](https://raw.githubusercontent.com/aboutcroon/Notes/main/Vue/Vue3/Vue%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0/assets/4-2.png)



那么我们怎么才能拦截一个对象属性的读取和设置操作呢？在ES2015之前我们采用Object.defineProperty函数来实现，在ES2015之后，我们可以使用代理对象Proxy来实现：

```js
// 存储对象的桶
const bucket = new Set()

// 原始数据
const data = { text: 'hello world' }
// 对原始数据的代理
const obj = new Proxy(data, {
  // 拦截读取操作
  get (target, key) {
    // 将副作用函数effect添加到存储副作用函数的桶中
    bucket.add(effect)
    return target[key]
  },
  // 拦截设置操作
  set (target, key, newVal) {
    // 设置属性值
    target[key] = newVal
    // 将副作用函数从桶里取出并执行
    bucket.forEach(fn => fn())
    // 返回true代表设置操作成功
    return true
  }
})
```

这时，我们更改data.text的值后，即可实现effect函数的调用

但是目前的实现存在很多缺陷，例如我们直接通过名字（effect）来获取副作用函数，这种硬编码方式很不灵活。副作用函数的名字应该可以任意的取，我们完全可以把副作用函数命名为myEffect，甚至是一个匿名函数，因此我们要想办法去掉这种硬编码的机制。



## 设计一个完整的响应式系统

首先我们需要提供一个用来注册副作用函数的机制：

```js
// 用一个全局变量存储被注册的副作用函数
let activeEffect
const fn = () => {
  document.body.innerText = obj.text
}
// effect函数用于注册副作用函数
function effect (fn) {
  acticeEffect = fn
  fn()
}
```

所以当effect函数执行时，我们可以把匿名的副作用函数fn赋值给全局变量 activeEffect，接着执行fn，然后就触发了响应式数据obj.text的读取操作，进而触发代理对象Proxy的get拦截函数。



下一个问题是，当我们在obj上设置一个不存在的属性时：

```js
obj.notExist = 'hello vue3'
```

这个操作是在副作用函数fn之外进行的，但这时也会触发obj的设置操作，从而触发副作用函数的执行。导致该问题的根本原因是：**没有在副作用函数与被操作的目标字段之间建立明确的联系**。我们需要重新设计“桶”的数据结构。



如果用target来表示被代理的obj，用key来表示被操作的属性，用effectFn来表示被注册的副作用函数，那么会有以下关系：

```
target
   ｜—— key1
        ｜——effectFn1
   ｜—— key2
        ｜——effectFn2
```



我们可以用WeakMap代替Set作为“桶”的数据结构：

- WeakMap由target --> Map构成
- Map由 key --> Set构成

关系如图所示：

<img src="/Users/liuchang/Documents/markdown/学习/Vue3/Vue.js设计与实现.assets/4-3.png" alt="4-3" style="zoom:50%;" />



> ❓**为何要使用WeakMap？**
>
> 这涉及到垃圾回收机制，WeakMap的key是弱引用，它不影响垃圾回收器的工作，所以一旦表达式执行完毕，垃圾回收器就会将key从内存中移除。
>
> 所以WeakMap经常用于存储那些只有当key所引用的对象存在时（没有被回收）才有价值的信息，例如上面的场景中，如果target对象没有任何引用了，说明用户侧不再需要它了，这时垃圾回收器会完成回收任务。但如果使用Map，那么这个target将不会被回收，最终可能导致内存溢出。



初步实现：

```js
const obj = new Proxy (data, {
  // 拦截读取操作
  get(target, key) {
    // 将副作用函数activeEffect 添加到存储副作用函数的桶中
    track(target, key)
    return target[key]
	},
  // 拦截设置操作
  set(target, key, newVal) {
    //设置属性值
    target[key] = newVal
    // 把副作用函数从桶里取出并执行
    trigger (target, key)
	}
})

// 在get 拦截函数内调用 track 函数追踪变化
function track(target, key) {
	// 没有activeEffect，直接 return
	if (!activeEffect) return
	let depsMap = bucket.get(target)
	if (!depsMap) {
  	bucket.set(target, (depsMap = new Map()))
  }
	let deps = depsMap.get(key)
	if (!deps) {
		depsMap.set(key, (deps = new Set()))
  }
	deps.add(activeEffect)
}

// 在set 拦截函数内调用 trigger 函数触发变化
function trigger (target, key) {
	const depsMap = bucket.get(target)
	if (!depsMap) return
	const effects = depsMap.get(key)
	effects && effects.forEach(fn => fn())
}
```

当读取属性值时，我们直接在get拦截函数里编写把副作用函数收集到 “桶”里的这部分逻辑单独封装到一个 track 函数中，函数的名字叫 track 是为了表达追踪的含义。同样，我们也可以把触发副作用函数重新执行的逻辑封装到trigger函数中。



## 分支切换与cleanup

切换分支的定义：

```js
const data = { ok: true, text: 'hello' }
const obj = new Proxy(data, { /* ... */ })

effect(function effectFn() {
  document.body.innerText = obj.ok ? obj.text : 'not'
})
```

当字段obj.ok的值发生变化时，代码执行的分支会跟着变化，这就是所谓的分支切换。



这时，副作用函数与响应式数据之间建立的联系如下：

```
data
 ｜—— ok
      ｜——effectFn
 ｜—— text
      ｜——effectFn
```

副作用函数 effectFn 分别被字段 data.ok 和字段 data.text 所对应的依赖集合收集

也就是说，当我们切换分支时，`obj.ok = false`，此时不会再读取 obj.text 的值，那么 obj.text 的值此时再变化后就不应该再继续执行 effectFn，我们需要及时清理遗留的副作用函数，以达到这个效果。



**解决这个问题的思路是**：每次副作用函数执行时，我们需要先把它从所有与之关联的依赖集合中删除，当副作用函数执行完毕后，会重新建立联系，但在新的联系中不会包含遗留的副作用函数。

要将一个副作用函数从所有关联的依赖集合中移除，就需要明确知道哪些依赖集合中包含它，因此我们需要重新设计副作用函数：

```js
// 用一个全局变量存储被注册的副作用函数
let activeEffect
// effect函数用于注册副作用函数
function effect (fn) {
  const effectFn = () => {
    // 当 effectFn 执行时，将其设置为当前激活的副作用函数
    activeEffect = effectFn
    fn()
  }
  // activeEffect.deps 用来存储所有与该副作用函数相关联的依赖集合
  effectFn.deps = []
  effectFn()
}
```

在 effect 内部我们定义了新的 effectFn 函数，并为其添加了 effectFn.deps 属性，该属性是一个数组，用来存储所有包含当前副作用函数的依赖集合。



那么 effectFn.deps 数组中的依赖集合是如何收集的呢？其实是在 track 函数中：

```js
// 在get 拦截函数内调用 track 函数追踪变化
function track(target, key) {
	// 没有activeEffect，直接 return
	if (!activeEffect) return
	let depsMap = bucket.get(target)
	if (!depsMap) {
  	bucket.set(target, (depsMap = new Map()))
  }
	let deps = depsMap.get(key)
	if (!deps) {
		depsMap.set(key, (deps = new Set()))
  }
  // 把当前激活的副作用函数添加到依赖集合 deps 中
	deps.add(activeEffect)

  // deps 就是一个与当前副作用函数存在联系的依赖集合
  // 将其添加到 activeEffect.deps 数组中
  activeEffect.deps.push(deps) // 新增
}
```

关系如图所示：

![4.7](https://raw.githubusercontent.com/aboutcroon/Notes/main/Vue/Vue3/Vue%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0/assets/4.7.png)



有了这个联系后，我们就可以在每次副作用函数执行时，根据 effectFn.deps 获取所有相关联的依赖集合，进而将副作用函数从依赖集合中移除：

```js
// 用一个全局变量存储被注册的副作用函数
let activeEffect
// effect函数用于注册副作用函数
function effect (fn) {
  const effectFn = () => {
    cleanup(effectFn) // 新增
    activeEffect = effectFn
    fn()
  }
  // activeEffect.deps 用来存储所有与该副作用函数相关联的依赖集合
  effectFn.deps = []
  effectFn()
}
```



## 嵌套的 effect 与 effect 栈

effect 是可以发生嵌套的，例如：

```js
effect(function fn1 () {
  effect(function fn2 () {
    /*...*/
  })
})
```



什么情况下会出现嵌套的 effect 呢？实际上 Vue.js 的渲染函数就是在一个 effect 中执行的：

```js
// Foo 组件
const Foo = {
  render () {
    return /*...*/
  }
}
```

在一个 effect 中执行 Foo 组件的渲染函数：

```js
effect(() => {
  Foo.render()
})
```

当组件发生嵌套时，例如 Foo 组件渲染了 Bar 组件：

```js
const Bar = {
  render () {
    return /*...*/
  }
}

const Foo = {
  render () {
    return <Bar />
  }
}
```

此时就发生了 effect 嵌套，它相当于：

```js
effect(() => {
  Foo.render()
  // 嵌套
  effect(() => {
    Bar.render()
  })
})
```



但观察现在的代码，我们用全局变量 activeEffect 来存储通过 effect 函数注册的副作用函数，这意味着同一时刻 activeEffect 所存储的副作用函数只能有一个。当副作用函数发生嵌套时，内层副作用函数的执行会覆盖 activeEffect 的值，并且永远不会恢复到原来的值。这时如果再有响应式数据进行依赖收集，即使这个响应式数据是在外层副作用函数中读取的，它们收集到的副作用函数也都会是内层副作用函数，这就是问题所在。



为了解决这个问题，我们需要一个副作用函数栈 effectStack，在副作用函数执行时，将当前副作用函数压入栈中，待副作用函数执行完毕后，将其从栈中弹出，并始终让 activeEffect 指向栈顶的副作用函数。这样就能做到一个响应式数据只会收集直接读取其值的副作用函数，而不会出现相互影响的情况。

如以下代码所示：

```js
// 用一个全局变量存储当前激活的 effect 函数
let activeEffect
// effect 栈
const effectStack = [] // 新增

function effect (fn) {
  const effectFn = () => {
    cleanup(effectFn)
    // 当调用 effect 注册副作用函数时，将副作用函数复制给 activeEffect
    activeEffect = effectFn
    // 在调用副作用函数之前将当前副作用函数压入栈中
    effectStack.push(effectFn) // 新增
    fn()
    // 在当前副作用函数执行完毕后，将当前副作用函数弹出栈，并把 activeEffect 还原为之前的值
    effectStack.pop() // 新增
    activeEffect = effectStack[effectStack.length - 1] // 新增
  }
  // activeEffect.deps 用来存储所有与该副作用函数相关联的依赖集合
  effectFn.deps = []
  // 执行副作用函数
  effectFn()
}
```

这样，将内层副作用函数 effectFn2 执行完毕后，他会被弹出栈，并将副作用函数 effectFn1 设置为 activeEffect。这样一来，响应式数据就只会收集直接读取其值的副作用函数作为依赖，从而避免发生错乱。



## 避免无限递归循环

实现一个完善的响应式系统要考虑诸多细节，下面要介绍的无限递归循环就是其中之一。

举个例子：

```js
const data = { foo: 1 }
const obj = new Proxy(data, { /*...*/ })

effect(() => obj.foo++)
```

可以看到，在 effect 注册的副作用函数内有一个自增操作 obj.foo++，该操作会引起栈溢出：

```
Uncaught RangeError: Maximum call stack size exceeded
```

为什么会这样呢？

实际上，我们可以把 `obj.foo++` 这个自增操作分开来看，它相当于 `obj.foo = obj.foo + 1`。在这个语句中，既会读取 obj.foo 的值，又会设置 obj.foo 的值，而这就是导致问题的根本原因。

首先读取 obj.foo 的值，这会触发 track 操作，将当前副作用函数收集到 “桶” 中，接着将其加 1 后再赋值给 obj.foo，此时又会同时触发 trigger 操作，即把 “桶” 中的副作用函数取出并执行。此时又会触发相同的操作，会无限递归的调用自己，于是就产生了栈溢出。



通过分析，我们发现读取和设置操作是在同一个副作用函数内进行的，此时无论是 track 时收集的副作用函数，还是 trigger 时要触发执行的副作用函数，都是 activeEffect。基于此，我们可以在 trigger 动作发生时增加守卫条件：如果 trigger 触发执行的副作用函数与当前正在执行的副作用函数相同，则不触发执行，如下代码所示：

```js
function trigger (target, key) {
	const depsMap = bucket.get(target)
	if (!depsMap) return
	const effects = depsMap.get(key)
  
  const effectsToRun = new Set()
  
	effects && effects.forEach(effectFn => {
    // 如果 trigger 触发执行的副作用函数与当前正在执行的副作用函数相同，则不触发执行
    if (effectFn !== activeEffect) { // 新增
      effectsToRun.add(effectFn)
    }
  })
  effectsToRun.forEach(effectFn => effectFn())
}
```

这样我们就能避免无限递归调用，从而避免栈溢出。



## 调度执行

可调度性：当 trigger 动作触发副作用函数重新执行时，有能力决定副作用函数执行的**时机**、**次数**以及**方式**

可调度性是响应系统非常重要的特性





