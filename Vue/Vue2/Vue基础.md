# 1. `computed` 与 `mounted` 哪个先执行

`computed` 执行的时机是内部所依赖的数据对象改变时。初识时，是实例中的 data 创建了就开始执行了



# 2. Vue 项目中的 `assets`

assets/style 下面放置 css 文件，分别为 `common.less/reset.less` 和 `variable.less`

common.less/reset.less 用来重置样式，或提供全局的公共样式（也可以在其中书写 ui 插件的公共样式），例如：

```less
@import '~ant-design-vue/dist/antd.less';
/* http://meyerweb.com/eric/tools/css/reset/
   v2.0-modified | 20110126
   License: none (public domain)
*/

html, body, div, span, applet, object, iframe,
h1, h2, h3, h4, h5, h6, p, blockquote, pre,
a, abbr, acronym, address, big, cite, code,
del, dfn, em, img, ins, kbd, q, s, samp,
small, strike, strong, sub, sup, tt, var,
b, u, i, center,
dl, dt, dd, ol, ul, li,
fieldset, form, label, legend,
table, caption, tbody, tfoot, thead, tr, th, td,
article, aside, canvas, details, embed,
figure, figcaption, footer, header, hgroup,
menu, nav, output, ruby, section, summary,
time, mark, audio, video {
  margin: 0;
  padding: 0;
  border: 0;
  vertical-align: baseline;
}

/* make sure to set some focus styles for accessibility */
:focus {
  outline: 0;
}

/* HTML5 display-role reset for older browsers */
article, aside, details, figcaption, figure,
footer, header, hgroup, menu, nav, section {
  display: block;
}

body {
  line-height: 1;
}

ol, ul {
  list-style: none;
}
```

```less
// 全局公共样式
html, body {
  min-height: 100% !important;
}
ul,li {
  list-style: none;
  line-height: 1;
}
#app {
  height: 100%;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
```

或者在 theme 文件夹下单独定义一个 ui 插件的公共样式：

```less
@import 'view-design/src/styles/index.less';

@primary-color          : #3160A8;

@border-radius-base     : 0px;
@border-radius-small    : 0px;
@btn-border-radius      : 0px;
@btn-border-radius-small: 0px;

/****************reset********************/
@table-border-color     : rgba(49,96,168,0.2);
.ivu-table {
  .ivu-table-header {
    position: relative;
    border-top: 1px solid @table-border-color;
  }  
  .ivu-table-fixed-header {
    border-top: 1px solid transparent;
  }
  .ivu-table-fixed::before, .ivu-table-fixed-right::before, .ivu-table-fixed-left::before {
    display: none;
  }
  &:before {
    background: @table-border-color;
  }
  th {
    background: #f1f5f9;
  }
  th, td {
    border-bottom: none;
  }
  tr:nth-child(even) td {
    background: #f8fafd;
  }
  tr.ivu-table-row-hover td {
    background: #f8f8f8;
  }
  .ivu-table-cell {
    padding-left: 12px;
    padding-right: 12px;
  }
  td.ivu-table-expanded-cell {
    padding: 20px;
  }
}
```



variable.less 中定义 less 的变量，可在每一个单文件的 style 内被引用，例如：

```less
@content-width: 1200px;
@header-height: 80px;
@grey: #7A8599;
@black: #1A2640;
@blue: #0073E6;
@text-grey: #aaa;
@red: #F2303D;
@grey1: #333;
@grey2: #666;
@grey3: #999;
@border: rgba(218,230,247,1);

@keyframes moveing {
  from {
    width:0;
    opacity:.7;
  }
  to {
    width:100%;
    opacity:0;
  }
}
.flex (@justify: center, @align: center, @direction: row, @wrap: nowrap) {
  display: flex;
  flex-direction: @direction;
  flex-wrap: @wrap;
  justify-content: @justify;
  align-items: @align;
}
.logo (@url: "~@/assets/images/logo.png", @width: 184px, @height: 32px){
  width: @width;
  height: @height;
  text-indent: -9999px;
  background-image: url(@url);
  background-size: contain;
}
```

#### variable.less 配置全局变量的方法：

使用 vue add 来安装 style-resources-loader，并在配置中选择 less 语言：

```undefined
vue add style-resources-loader
```

在webpack中配置：

```js
pluginOptions: {
  'style-resources-loader': {
    preProcessor: 'less',
    patterns: [
      path.resolve(__dirname, './src/assets/style/variable.less')
    ]
  }
},
```



# 3. 动态 class 绑定

动态 class 还可以绑定一个 `computed` 值：

```html
<div v-bind:class="classObject"></div>
```

```js
data: {
  isActive: true,
  error: null
},
computed: {
  classObject: function () {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}
```

参考网址：https://cn.vuejs.org/v2/guide/class-and-style.html



# 4. 命名路由

使用命名路由进行匹配时，若匹配到的路由有子路由，如下：

```js
{
    path: 'activate',
    name: 'Activate',
    meta: {
      title: '激活',
      icon: 'activate'
    },
    isMenu: true,
    component: Skeleton,
    children: [
      {
        path: '',
        name: 'LicenseList',
        meta: {
          title: 'license列表',
          menuName: 'Activate'
        },
        component: LicenseList
      },
      {
        path: ':id',
        name: 'LicenseDetail',
        meta: {
          title: 'license详情',
          menuName: 'Activate'
        },
        component: LicenseDetail
      }
    ]
  }
```

此时若 push({ name: Activate})，那么会匹配到 activate 路径的路由，但我们原意是要在进入 activate 路由时默认进入第一个子路由。当我们是 push('/activate') 时可以实现，但 push 命名路由时不能实现，所以此时我们要在路由上加 redirect:

```js
{
    path: 'activate',
    name: 'Activate',
    meta: {
      title: '激活',
      icon: 'activate'
    },
    isMenu: true,
    component: Skeleton,
    redirect: {
      name: 'LicenseList'
    },
    children: [
      {
        path: '',
        name: 'LicenseList',
        meta: {
          title: 'license列表',
          menuName: 'Activate'
        },
        component: LicenseList
      },
      {
        path: ':id',
        name: 'LicenseDetail',
        meta: {
          title: 'license详情',
          menuName: 'Activate'
        },
        component: LicenseDetail
      }
    ]
  }
```



# 5. methods

## 5.1 函数参数的问题

如果是某个 ui 库（例如ant-design）上的事件，直接绑定在模版中，不传参数，那么 methods 中写的函数也会自动有对应的参数产生。

但如果是 methods 函数调用 methods 中的函数，那么就是普通函数，这个时候如果没传参数，则会出错。这时默认的第一个参数是 event。



这时`更好的解决方法`是：给函数参数加一个默认值就好了，这样不穿参数就直接使用默认值



# 6. nextTick



# 7. computed

## 7.1 使用场景

computed 设计初衷是因为不在模版中放入太多的逻辑：

```html
<div id="example">
  {{ message.split('').reverse().join('') }}
</div>
```

所以，对于任何复杂逻辑，都应当使用**计算属性**。



例如：

```html
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
```

```js
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
```

```
结果：

Original message: "Hello"

Computed reversed message: "olleH"
```



## 7.2 不能带参数

computed 中的函数不能带参数，例如：

```html
<router-link :to="toPath(item)"></routerlink>
```

```js
computed: {
  toPath(item) {
    return item.name
  }
}
```

这样写是错误的



## 7.3 顺序

在 computed 中执行 `console.log()`，例如：

```js
computed: {
    ...mapState(['userInfo', 'server']),
    breadcrumbRoutes () {
      const match = this.$route.matched.filter(route => route.meta.title && !route.meta.blank)
      console.log(match) // 这里输出的 match 不一定是上面这行代码得到的 match，可能是执行完下面 if 语句后才得到的 match
      if (match[0].name === 'Server') {
        while (match.length) {
          match.pop()
        }
        const snode = { name: 'ServerNode', meta: { title: '服务节点' } }
        const currentServer = { name: 'Server', meta: { title: `${this.server.ip}` } }
        match.push(snode)
        match.push(currentServer)
      }
      const list = [{ name: 'Dashboard', meta: { title: '首页' } }, { name: 'Dashboard', meta: { title: `${this.controller.ip}` } }]
      const res = list.concat(match)
      return res
    }
  },
```

注意代码中，`console.log` 输出的 match 不一定是上面这行代码得到的 match，可能是执行完下面 if 语句后才得到的 match



## 7.4 异步数据

computed 中所依赖的数据从 vuex 中取，但是 computed 执行时 vuex 还未执行怎么办？

回答：vuex中的数据是通过某个请求获取到的，而这个请求肯定是异步的请求，computed 展示数据时，可能这个数据还未获取到，但等这个异步请求完成后，就会获取到并展示出来了。此时如果在 computed 中写了 this.server.ip，那么在 this.server 还未获取到时便会报错，虽然请求完成后便不会报错了，但是在请求完成前这行代码会报错，这时候要写成 `this.server?.ip`





# 8. v-if

v-if 中可以使用 `===` 和 `&&` 等符号



## 8.1 异步问题

例如：

这里的逻辑是要当其为不运行时才显示，而因为该数据是异步获取的，那么在模版渲染完成后数据还未获取到，所以一开始 taskInfo.running 为 undefined，取反之后为 true，所以不管运不运行，一开始会先显示 true，然后等数据获取到后才会 显示 false，会造成闪一下的状况。

```html
<i class="iconfont icon-icon_Timestamp icon" v-if="!taskInfo.running"></i>
```

将其加上一句判定后，仍然不行，data 中的 taskInfo 初始时为 {}，但 {} 仍然会为 true，因为空对象是引用类型，所以空对象和空数组都是 true

```html
<i class="iconfont icon-icon_Timestamp icon" v-if="taskInfo && !taskInfo.running"></i>
```

解决方案是这样子，再加上一个异步的数据，使其同时进行判定

```html
<i class="iconfont icon-icon_Timestamp icon" v-if="taskInfo.endTime && !taskInfo.running"></i>
```

第二个解决方案是使用一个 computed 值计算之后再放到 v-if 中



# 9. scoped

\>>> 和 /deep/ 深度选择器（应该尽量少的使用 scoped，这样会使打包面积增大）

在 vue 中才有 scoped

在有less-loader或 css-loader 时使用 /deep/

在没有 loader 时可使用 >>>，比如写 css 可以使用 >>>，写 less 时尽量使用 /deep/

在原生的 css 里没有深度选择器



一般也不建议使用深度选择器，有兼容性问题，推荐的做法是写两个style，一个有 scoped，一个没有 scoped



# 10. 父子组件间传值

父组件给子组件传值时，不能传递 null，这时会报警告

此时要给其一个空值 [] 或 {}

例如：

```html
<div class="content mt20">
      <page-table
        :data="dataList"
        :columns="columns"
        :pagination="{
          total: this.total,
          current: this.pageIndex,
          pageSize: this.pageSize,
          onChange: this.onPageChange,
          onPageSizeChange: this.onPageSizeChange
        }"
        @on-sort-change="onTableSort"
      ></page-table>
</div>

<!-- 比如这里传递给子组件的 dataList 不能为 null 值，所以要加上 this.dataList = res.data || [] -->
```



# 11. template

template 中内嵌 template 时，template 不会渲染成元素，用 div 的话会被渲染成元素

参考网址：https://segmentfault.com/q/1010000010727886



# 12. created 和 mounted 中哪里放异步请求数据

主要区别是是否能获取 dom，如果要在页面创建时获取并改变 dom，就用 mounted，例如网页翻译。

初始化 data 的话随便放哪儿都一样，按思维方式一般都是渲染时就先去获取数据也就是放 created 里面 



# 13. mutations 与 actions

问题：在vuex中更新state时，为什么将异步方法写在actions中，而不是mutations，这是为什么？



vuex中更新state的方法：

首页，在 vuex 中只有 mutations 可以更新state

- commit 一个 mutation，mutation 负责更改 state
- dispatch 一个 action，在 action 中 commit 一个 mutation

所以按照上述使用方法，我们在使用时，如果不涉及异步操作，可以直接 commit 一个 mutation 去更改 state，如果有异步就需要将异步方法写在 dispatch 中，然后在 dispatch 中commit mutation 去更改 state



现在来回答开始提到的问题：

我们从源码角度来回答这个问题，以下代码来自 vuex 源码，做了部分提示警告逻辑的删减，不影响整体逻辑

commit

```js
// 这个就是 Store 类的 commit 方法
function commit (_type, _payload, _options) {
    // check object-style commit
    const {type, payload, options} = unifyObjectStyle(_type, _payload, _options)
    // 定义mutation对象，type 其实就是我们要操作的 mutation 的方法名， payload 是参数（载荷）
    const mutation = { type, payload }
    // entry 就是要被执行的 mutation 方法
    const entry = this._mutations[type]
    
    // 省略了一些中间不影响逻辑的代码
    
    // 注册一些回调函数，可以看到 mutation 方法（entry）最终是在这个回调函数中执行，直接就执行结束，没有任何的 return 以及 异步处理，这样也就是说在 commit 中不可以写异步逻辑
    this._withCommit(() => {
        entry.forEach(function commitIterator (handler) {
            handler(payload)
        })
    })
    this._subscribers
    .slice() // shallow copy to prevent iterator invalidation if subscriber synchronously calls unsubscribe
    .forEach(sub => sub(mutation, this.state))
}
```

dispatch

```js
function dispatch (_type, _payload) {
    // check object-style dispatch
    const {type,payload} = unifyObjectStyle(\type, _payload)
    // 定义 action 对象，type 是 actions 对象中的一个属性，payload 是载荷
    const action = { type, payload }
    // 待执行的 action 方法
    const entry = this._actions[type]
    try {
        this._actionSubscribers
        .slice() // shallow copy to prevent iterator invalidation if subscriber synchronously calls unsubscribe
        .filter(sub => sub.before)
        .forEach(sub => sub.before(action, this.state))
    } catch (e) {}
    
    // 在这里执行了 action 方法，并将结果用 Promise.all 方法处理
    const result = entry.length > 1
    ? Promise.all(entry.map(handler => handler(payload)))
    : entry[0](payload)
    
    // 这里的 result.then 执行完以后会 return 一个结果出去（其实就是我们自己调用的异步逻辑的结果）
    return result.then(res => {
        try {
            this._actionSubscribers
            .filter(sub => sub.after)
            .forEach(sub => sub.after(action, this.state))
        } catch (e) {}
        return res
    })
}
```



# 14. Vue.set

向响应式对象上添加新 property，并确保这个新 property 同样是响应式的，且触发视图更新。

因为 Vue 无法探测普通的新增 property 

https://cn.vuejs.org/v2/api/#Vue-set



# 15. vm.$watch

immediate 参数用法

https://cn.vuejs.org/v2/api/#vm-watch

作用：通常 watch 是在该值变化时才执行回调，指定 immediate 为 true，可使其在开始侦听时就调用一次回调函数，而不用等到其第一次发生变化时才调用。

使用场景：子组件通过 props 接收父组件传递过来的值时，使用 watch 侦听该值，为了使父组件一传递过来就执行回调，这时候需要指定 immediate 为 true



watch 不能用来监听 dom 元素高度变化

只能 ResizeObserver 和 window.onresize



# 16. Vuex

vuex 分模块后，如何使用 mapState

https://segmentfault.com/q/1010000019232996



vuex 的 getter 是相当于 state 的 computed 属性



vuex 分模块之后如何使用 mapState

https://segmentfault.com/q/1010000019232996

扩展：vuex 如何分模块



# 17. Vue Router

this.$route.query，读取时，数字会变成字符串格式



# 18. v-model

v-model.number 可以将input双向绑定的字符串值自动转化成number类型
参考https://www.sunzhongwei.com/vuejs-v-model-automatically-converts-string-to-number



# 19. 插槽

具名插槽的v-slot:title 一定要用在 template 标签中，不能用在 div 标签上

template 标签上不能加 class 等，要加在 template 里面的 div 里



# 20. 业务

```js
@click=“handle()”

handle(record) {

}
```

当上述是@click=“handle”时，传入的第一个参数会变成 event

当上述是@click=“handle()”时，传入的第一个参数才会变成 undefined

