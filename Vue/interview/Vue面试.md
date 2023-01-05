## v-model

> 参考官方文档：[Vue2的自定义组件v-model](https://cn.vuejs.org/v2/guide/components-custom-events.html#%E8%87%AA%E5%AE%9A%E4%B9%89%E7%BB%84%E4%BB%B6%E7%9A%84-v-model)，[Vue3的v-model](https://v3.cn.vuejs.org/guide/migration/v-model.html#%E6%A6%82%E8%A7%88)



### Vue2 写法

#### 基本元素上的使用

```html
<!-- v-model -->
<input type="text" v-model="value">

<!-- v-bind 和 v-on -->
<input type="text" :value="value" @input="value = $event.target.value">
```



#### 组件上的使用

```html
<!-- ParentComponent.vue -->

<ChildComponent v-model="pageTitle" />
```

```js
// ChildComponent.vue

export default {
  // 如果想要更改 prop 或事件名称，则需要在 ChildComponent 组件中添加 model 选项，如果不需要更改 prop 或事件名称，则不需要这个 model，默认直接对应于 props 里的 value
  model: {
    prop: 'title', // 这里的 title 对应下面 props 里面的 title，如果这里叫其他值则名字对应为其他值
    event: 'change'
  },
  props: {
    // 这将允许 `value` 属性用于其他用途
    value: String,
    // 使用 `title` 代替 `value` 作为 model 的 prop
    title: {
      type: String,
      default: 'Default title'
    }
  }
}
```

所以，在这个例子中 `v-model` 是以下的简写：

```html
<ChildComponent :title="pageTitle" @change="pageTitle = $event" />
```



实际中的例子：

```html
<!-- Parent.vue -->
<template>
  <div class="parent">
    <!-- type='text' 通过 v-bind="$attrs" 传给了input -->
    <Child type='text' v-model="value"></Child>
    <!-- 相当于 -->
    <Child type='text' :value="value" @input="value = $event"></Child>
  </div>
</template>

<script>
import Child from './child.vue'
export default {
  components: {
    Child
  },
  data() {
    return {
      value: 'false'
    }
  },
}
</script>

<!-- Child.vue -->
<template>
  <div class="child">
  <!-- 这里的v-bind="$attrs"是继承了其他各位的attribute(属性) 这里: type='text' -->
    <input v-bind="$attrs" :value="value" @input="update">
  </div>
</template>

<script>
export default {
  // 这里不需要更改 prop 或事件名称，则不需要使用 model，默认直接对应于 props 里的 value
  props: {
    value: {
      type: String
    }
  },
  methods: {
    update (e) {
      this.$emit('input', e.target.value) // 这个事件直接传递到了 Parent.vue
    }
  },
}
</script>
```

但是，我们改为 `type="checkbox"` 时就失效了，此时就需要用到 `model`



我们可以看官方文档中的 [表单输入绑定](https://cn.vuejs.org/v2/guide/forms.html#%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95) 的介绍

> `v-model` 在内部为不同的输入元素使用不同的 property 并抛出不同的事件：
>
> - text 和 textarea 元素使用 `value` property 和 `input` 事件；
> - checkbox 和 radio 使用 `checked` property 和 `change` 事件；
> - select 字段将 `value` 作为 prop 并将 `change` 作为事件。

因为 v-model 是 v-bind 和 v-on 的结合，默认 v-bind 绑定的是 **value**，v-on 绑定的是 **input 事件**，所以当 input 类型为 text 的时候 value 和 input 正好对应上去了，但是在 type 为 checkbox 时，v-bind 绑定的是 **checked**，v-on 绑定的是 **change 事件**，因此无效，所以我们需要使用 `model` 重新命名它：

```html
<!-- Parent.vue -->
<template>
  <div class="parent">
    <Child type='checkbox' v-model="value"></Child>
  </div>
</template>

<script>
import Child from './child.vue'
export default {
  components: {
    Child
  },
  data() {
    return {
      value: false
    }
  },
}
</script>

<!-- Child.vue -->
<template>
  <div class="child">
    <input v-bind="$attrs" :checked="value" @change="update">
  </div>
</template>

<script>
export default {
  model: {
    prop: 'value',
    event: 'change'
  },
  props: {
    value: {
      type: String
    }
  },
  methods: {
    update (e) {
      this.$emit('change', e.target.value)
    }
  },
}
</script>
```



#### 使用 v-bind.sync

> 2.3.0+ 新增

示例：

```html
<ChildComponent :title.sync="pageTitle" />
```

相当于：

```html
<!-- ParentComponent.vue -->

<ChildComponent :title="pageTitle" @update:title="pageTitle = $event" />
```

```js
// ChildComponent.vue
this.$emit('update:title', newValue)
```



因为在某些情况下，我们可能需要对 `某一个 prop` 进行“双向绑定”(除了前面用 `v-model` 绑定 prop 的情况)。不幸的是，真正的双向绑定会带来维护上的问题，因为子组件可以变更父组件，且在父组件和子组件两侧都没有明显的变更来源。

为此，我们建议使用 `update:myPropName` 抛出事件。

所以，这里的应用场景是在父子组件传值，且子组件要修改这个数据的时候使用。它的原理是利用 EventBus，子组件触发事件，父组件响应事件并实现数据的更新，避免由子组件直接修改父组件传过来的内容。（如果子组件直接操作，vue会有警告提示）



### Vue3 写法

在 3.x 中，自定义组件上的 `v-model` 相当于传递了 `modelValue` prop 并接收抛出的 `update:modelValue` 事件：

```html
<ChildComponent v-model="pageTitle" />

<!-- 是以下的简写: -->

<ChildComponent :modelValue="pageTitle" @update:modelValue="pageTitle = $event" />
```



#### v-model 参数

若需要更改 `model` 的名称，现在我们可以为 `v-model` 传递一个*参数*，以作为组件内 `model` 选项的替代：

```html
<ChildComponent v-model:title="pageTitle" />

<!-- 是以下的简写: -->

<ChildComponent :title="pageTitle" @update:title="pageTitle = $event" />
```

这也可以作为 `.sync` 修饰符的替代，而且允许我们在自定义组件上使用多个 `v-model`。

```html
<ChildComponent v-model:title="pageTitle" v-model:content="pageContent" />

<!-- 是以下的简写： -->

<ChildComponent
  :title="pageTitle"
  @update:title="pageTitle = $event"
  :content="pageContent"
  @update:content="pageContent = $event"
/>
```



#### v-model 修饰符

除了像 `.trim` 这样的 2.x 硬编码的 `v-model` 修饰符外，现在 3.x 还支持自定义修饰符：

```html
<ChildComponent v-model.capitalize="pageTitle" />
```

我们可以在 [Custom Events](https://v3.cn.vuejs.org/guide/component-custom-events.html#处理-v-model-修饰符) 部分中了解有关自定义 `v-model` 修饰符的更多信息。



### 迁移策略

参考官方文档：[迁移策略](https://v3.cn.vuejs.org/guide/migration/v-model.html#%E8%BF%81%E7%A7%BB%E7%AD%96%E7%95%A5)



## 为什么Data是一个函数

组件中的 data 写成一个函数，可以让数据函数返回值形式定义，这样每复用一次组件，就会返回一份新的 data，类似于给每个组件实例创建一个私有的数据空间，`让各个组件实例维护各自的数据`。

而单纯的写成对象形式，就使得所有组件实例共用了一份 data，就会造成一个变了全都会变的结果



## Vue 组件通讯有哪几种方式

1. props 和$emit。父组件向子组件传递数据是通过 prop 传递的，子组件传递数据给父组件是通过$emit 触发事件来做到的

2. $parent, $children, $root。获取当前组件的父组件和当前组件的子组件，获取根组件

3. $attrs 和$listeners A->B->C。Vue 2.4 开始提供了$attrs 和$listeners 来解决这个问题（不常用）

4. 父组件中通过 provide 来提供变量，然后在子组件中通过 inject 来注入变量。

   > 官方不推荐在实际业务中使用，但是写组件库时很常用。
   >
   > 设计项目，通常追求有清晰的数据流向和合理的组件层级关系便于调试和维护，然而 provide 和 inject 支持任意层级都能访问，导致数据追踪比较困难。不知道哪一层级声明了 provide 又或是哪些层级使用了 inject。造成比较大的维护成本。因此，除组件库或高阶插件外，Vue 建议用 Vuex 解决或其他办法处理。

5. $refs 获取组件实例

6. eventBus 事件总线的方式，兄弟组件数据传递的情况下适合使用

   > https://segmentfault.com/a/1190000038203554
   >
   > ```ts
   > import Vue from 'vue'
   > //因为是全局的一个'仓库'，所以初始化要在全局初始化
   > const EventBus = new Vue()
   > 
   > // 发送事件
   > EventBus.$emit("aMsg", this.MsgA)
   > // 接受事件
   > EventBus.$on("aMsg", (data) => {
   >   //将A组件传递过来的参数data赋值给msgB
   >   this.msgB = data
   > })
   > ```
   >
   > 

7. vuex 状态管理



## Vue 的生命周期方法有哪些 一般在哪一步发请求

`beforeCreated`

在实例初始化之后，数据观测(data observer) 和 event/watcher 事件配置之前被调用。在当前阶段 data、methods、computed 以及 watch 上的数据和方法都不能被访问

`created`

实例已经创建完成之后被调用。在这一步，实例已完成以下的配置：数据观测(data observer)，属性和方法的运算， watch/event 事件回调。

**这里没有$el**，如果非要想与 Dom 进行交互，可以通过 vm.$nextTick 来访问 Dom

`beforeMount`

在挂载开始之前被调用：相关的 render 函数首次被调用。

`mounted`

在挂载完成后发生，在当前阶段，真实的 Dom 挂载完毕，**数据完成双向绑定，可以访问到 Dom 节点**

`beforeUpdate`

数据更新时调用，发生在虚拟 DOM 重新渲染和打补丁（patch）之前。可以在这个钩子中进一步地更改状态，这不会触发附加的重渲染过程

`updated`

发生在更新完成之后，当前阶段组件 Dom 已完成更新。要注意的是避免在此期间更改数据，因为这可能会导致无限循环的更新，该钩子在服务器端渲染期间不被调用。

`beforeDestroy`

实例销毁之前调用。在这一步，实例仍然完全可用。我们可以在这时进行善后收尾工作，比如清除计时器。

`destroyed`

Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。 该钩子在服务器端渲染期间不被调用。

`activated`

Keep-alive专属，组件被激活时调用

`deactivated`

Keep-alive专属，组件被销毁时调用



**异步请求在哪一步发起？**

可以在钩子函数 created、beforeMount、mounted 中进行异步请求，因为在这三个钩子函数中，data 已经创建，可以将服务端端返回的数据进行赋值。

如果异步请求不需要依赖 Dom 推荐在 created 钩子函数中调用异步请求，因为在 created 钩子函数中调用异步请求有以下优点：

- 能更快获取到服务端数据，减少页面  loading 时间；
- ssr  不支持 beforeMount 、mounted 钩子函数，所以放在 created 中有助于一致性；



## v-if 和 v-show

v-if 在编译过程中会被转化成三元表达式，条件不满足时不渲染此节点。

v-show 会被编译成指令，条件不满足时控制样式将对应节点隐藏 （display:none）



使用场景：v-if 适用于在运行时很少改变条件，不需要频繁切换条件的场景；v-show 适用于需要非常频繁切换条件的场景。



> 拓展：display: none、visibility: hidden 和 opacity: 0 之间的区别？
>
> |                    | 是否占据空间 | 子元素是否继承                                          | 事件绑定 | 过渡动画transition |
> | :----------------- | :----------- | :------------------------------------------------------ | -------- | ------------------ |
> | display: none      | 是           | 否，父元素都不存在了，子元素也不会显示出                | 无法触发 | 无效               |
> | visibility: hidden | 否           | 是，可通过设置子元素的 visibility: visible 来显示子元素 | 无法触发 | 无效               |
> | opacity: 0         | 否           | 是，但不能通过设置子元素opacity: 1来重新显示            | 可触发   | 生效               |



## Vue内置指令

| 指令                      | 定义                                                         |
| ------------------------- | ------------------------------------------------------------ |
| v-bind                    | 绑定属性，动态更新HTML元素上的属性，比如 v-bind:class        |
| v-on                      | 用于监听DOM事件，例如 v-on:click, v-on:keyup                 |
| v-model                   | value 和 input 的语法糖                                      |
| v-if / v-else-if / v-else | 在 render 函数里会被转化成三元表达式                         |
| v-show                    | 通过display来决定显示或隐藏                                  |
| v-for                     | 循环指令；优先级比v-if高，会先解析v-for再解析v-if，所以最好不要一起使用，尽量使用计算属性去解决；注意增加唯一key值 |
| v-once                    | 定义它的元素或组件只渲染一次，包括元素或组件的所有子节点，首次渲染后，不再随数据的变化重新渲染，将被视为静态内容 |
| v-html                    | 赋值就是变量innerHTML，注意防治xss攻击                       |
| v-text                    | 更新元素的textContent                                        |
| v-pre                     | 跳过这个元素以及子元素的编译过程，以此来加快整个项目的编译速度 |



## v-if 与 v-for 不要共同使用

原因：

1. v-for的优先级比v-if的优先级高

   > 源码中会使用if, else if 来判断指令，而 v-for 在 v-if 的前面，所以优先级要高

2. 哪怕只渲染出一小部分用户的元素，也得在每次重渲染的时候遍历整个列表，不论活跃用户是否发生了变化



所以，如果把 `v-if` 和 `v-for` 同时用在同一个元素上，会带来性能方面的浪费（每次渲染都会先循环再进行条件判断）



## 怎样理解vue的单向数据流

数据总是从父组件传到子组件，子组件没有权利修改父组件传过来的数据，只能通过触发事件来请求父组件对原始数据进行修改。这样会防止从子组件意外改变父级组件的状态，从而导致你的应用的数据流向难以理解。

> 注意：在子组件直接用 v-model 绑定父组件传过来的 prop 这样是不规范的写法，在开发环境会报警告。如果实在要改变父组件的 prop 值 可以在 data 里面定义一个变量 并用 prop 的值初始化它 之后用 $emit 通知父组件去修改



## computed 和 watch 的区别及运用场景

**区别：**

computed 是计算属性，依赖其他属性计算值，并且 computed 的值有缓存，只有当计算值变化才会返回内容，它可以设置 getter 和 setter。

watch 监听到值的变化就会执行回调，在回调中可以进行一些逻辑操作。



**运用场景：**

计算属性一般用在模板渲染中，某个值是依赖了其它的响应式对象甚至是计算属性计算而来；

而侦听属性适用于观测某个值的变化去完成一段复杂的业务逻辑



## vue2响应式数据原理

整体思路：`数据劫持`+`观察者模式`

对象内部通过 defineReactive 方法，使用 Object.defineProperty 将属性进行劫持（只会劫持已经存在的属性），数组则是通过重写数组方法来实现。当页面使用对应属性时，每个属性都拥有自己的 dep 属性，存放他所依赖的 watcher（依赖收集），当属性变化后会通知自己对应的 watcher 去更新(派发更新)。

相关代码如下：

```js
class Observer {
  // 观测值
  constructor(value) {
    this.walk(value);
  }

  walk(data) {
    // 对象上的所有属性依次进行观测
    let keys = Object.keys(data);
    for (let i = 0; i < keys.length; i++) {
      let key = keys[i];
      let value = data[key];
      defineReactive(data, key, value);
    }
  }
}

// Object.defineProperty数据劫持核心 兼容性在ie9以及以上
function defineReactive(data, key, value) {
  observe(value); // 递归关键
  // --如果value还是一个对象会继续走一遍odefineReactive 层层遍历一直到value不是对象才停止
  //   思考？如果Vue数据嵌套层级过深 >>性能会受影响
  Object.defineProperty(data, key, {
    get() {
      console.log("获取值");

      //需要做依赖收集过程 这里代码没写出来
      return value;
    },
    set(newValue) {
      if (newValue === value) return;
      console.log("设置值");
      //需要做派发更新过程 这里代码没写出来
      value = newValue;
    },
  });
}
export function observe(value) {
  // 如果传过来的是对象或者数组 进行属性劫持
  if (
    Object.prototype.toString.call(value) === "[object Object]" ||
    Array.isArray(value)
  ) {
    return new Observer(value);
  }
}
```

