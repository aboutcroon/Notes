## package.json文件修改

升级版本的依赖包，以及需要新添加的依赖包

```json
"dependencies": {
  "ant-design-vue": "^3.2.6", // "ant-design-vue": "^1.7.2"
  "core-js": "^3.8.3", // "core-js": "^3.6.4",
  "vue": "^3.2.13", // "vue": "^2.6.11",
  "vue-i18n": "^9.1.10", // "vue-i18n": "^8.26.8",
  "vue-router": "^4.0.12", // "vue-router": "^3.1.6",
  "pinia": "^2.0.14", // "vuex": "^3.1.3",
},
"devDependencies": {
  "@vue/cli-plugin-babel": "~5.0.0", // "@vue/cli-plugin-babel": "~4.3.0",
  "@vue/cli-plugin-eslint": "~5.0.0", // "@vue/cli-plugin-eslint": "~4.3.0",
  // "@vue/cli-plugin-router": "~4.3.0", 已删除
  // "@vue/cli-plugin-vuex": "~4.3.0", 已删除
  "@vue/cli-service": "~5.0.0", // "@vue/cli-service": "~4.3.0",
  // "@vue/eslint-config-standard": "^5.1.2", 已删除
  "eslint": "^7.32.0", // "eslint": "^7.18.0",
  "eslint-plugin-vue": "^8.0.3", // "eslint-plugin-vue": "^6.2.2",
  "less": "^4.0.0", // "less": "^3.11.1",
  "less-loader": "^10.0.0", // "less-loader": "^6.1.0",
  "postcss": "^8.4.14", // "postcss": "^7.0.39",
  // "style-resources-loader": "^1.3.2", 已删除
  "tailwindcss": "^3.1.3", // "tailwindcss": "npm:@tailwindcss/postcss7-compat@^2.2.17",
}
```



## Webpack 5迁移

`vue.config.js` 文件可以先空着，将 vue3 语法迁移完之后再迁移 webpack5 语法，这个很快

参考文档：

https://webpack.js.org/migrate/

https://github.com/webpack/webpack-dev-server/blob/master/migration-v4.md (webpack-dev-server迁移到v4)



## vue-router 4迁移

2.x

```js
// router/index.js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes: [
    {
      path: '',
      redirect: {
        name: 'Summaries'
      },
      children: []
    }
  ]
})

export default router
```

```js
// main.js
import Vue from 'vue'
import App from './App.vue'
import router from './router'

const vue = new Vue({
  router,
  render: h => h(App)
}).$mount('#app')
```



3.x

```js
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  // base 配置被作为 createWebHistory (其他 history 也一样)的第一个参数传递
  history: createWebHistory(process.env.BASE_URL),
  routes: [
    {
      path: '',
      redirect: {
        name: 'Summaries'
      },
      children: []
    }
  ]
})

export default router
```

```js
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

const app = createApp(App)
app.mount('#app')
```



参考文档：https://router.vuejs.org/zh/guide/migration/index.html



## vuex 迁移至 pinia

vuex

```js
// store/index.js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    token: null,
    userInfo: null,
    config: {},
    controllerList: [],
    controller: null,
    serverList: [],
    server: null,
    fullScreen: false,
    summary: {},
    summaryList: [],
    size: 'middle',
    windowWidth: 0,
    features: {},
    hideBread: false
  },
  getters: {},
  mutations: {},
  actions: {}
})

export default store
```

```js
// main.js
import Vue from 'vue'
import App from './App.vue'
import store from './store'

const vue = new Vue({
  store,
  render: h => h(App)
}).$mount('#app')
```



pinia

options api 写法

```js
// store/index.js
import { defineStore } from 'pinia'

const useStore = defineStore('main', {
  state: () => ({
    count: 0
  }),
  actions: {
    increment () {
      this.count++
    }
  },
  getters: {
    double: state => state.count * 2
  }
})

const useCounterStore = defineStore('counter', {
  ...
})

export {
	useStore,
  useCounterStore
}
```

composition api 写法

```js
// store/index.js
import { ref, computed } from 'vue'
import { defineStore } from 'pinia'

const useStore = defineStore('main', () => {
  const count = ref(0)
  function increment() {
    count.value++
  }
  const double = computed(() => {
    return count * 2
  })
  
  return {
    count,
    increment,
    double
  }
})

const useCounterStore = defineStore('counter', () => {
  ...
})

export {
	useStore,
  useCounterStore
}
```

main.js 中

```js
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import { createPinia } from 'pinia'

const app = createApp(App)
const pinia = createPinia()
app.use(pinia)
app.mount('#app')
```

使用到的文件中：

```html
<!-- options api -->
<template>
  <div>{{ store.count }}</div>
</template>
<script>
import { useStore } from '@/store'
import { defineComponent, ref } from 'vue'
export default defineComponent({
  data () {
    const store = useStore()
    return {
      store
    }
  },
  created () {
    const store = useStore()
    console.log(store.count)
  },
  methods: {
    handleCount () {
      const store = useStore()
    	store.$patch({ count: 1 }) // 相当于 mutation
      console.log(store.count) // 1
    }
  }
})
</script>
```

> 不要用 options api 的方式来使用 pinia（例如 mapState, mapActions），因为 views 里有的页面会比 main.js 先执行，这时 computed 里面的 mapState 就会报错，因为 main.js 里 pinia 的 defineStore 还未初始化就使用了，然后报错。因此页面中的 pinia 我们需要全部用函数声明的方式来使用，在函数内使用即可。



参考文档：https://pinia.vuejs.org/cookbook/migration-vuex.html



## ant-design-vue迁移

参考文档：

https://www.antdv.com/docs/vue/migration-v2-cn

https://www.antdv.com/docs/vue/migration-v3-cn

### 如何引入

2.x

```js
// antd.js
import Vue from 'vue'
import { Button } from 'ant-design-vue'
import 'ant-design-vue/dist/antd.css'

const antObj = { Button }

Vue.prototype.$message = message
Vue.prototype.$modal = Modal
Object.values(antObj).forEach(item => {
  Vue.use(item)
})
```

```js
// main.js
import moment from 'moment'
import 'moment/locale/zh-cn'

moment.locale('zh-cn')
```



3.x

```js
// antd.js
import { Button } from 'ant-design-vue'
import 'ant-design-vue/dist/antd.css'

const antObj = { Button }

export const setupAntd = (app) => {
  app.config.globalProperties.$message = message
  app.config.globalProperties.$modal = Modal
  Object.values(antObj).forEach(item => {
    app.use(item)
  })
}
```

```js
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import dayjs from 'dayjs'
import 'dayjs/locale/zh-cn'
import { setupAntd } from './antd.js'

dayjs.locale('zh-cn')
const app = createApp(App)
setupAntd(app)
```



### table

#### customRender 入参更改

```js
// before
Function(text, record, index) {}

// now
Function({text, record, index, column}) {}
```



#### slot，slot-scope 去除

所有 slot，slot-scope 改成具名插槽形式

```html
<!-- before -->
<a-table>
  <div slot="status" slot-scope="record">
    {{ record.InUsed ? 'a' : 'b' }}
  </div>
  <div slot="expandedRowRender" slot-scope="record">
    <ExpandRow :row="record.VGPUs" />
  </div>
</a-table>

<!-- now -->
<a-table>
  <template #headerCell="{ column }">
    <template v-if="column.key === 'ratio'">
      <span style="margin-right: 3px;">{{ usedCalc }}</span>
    </template>
  </template>
  <template #bodyCell="{ column, record }">
    <template v-if="column.key === 'groupId'">
      <div>{{ record.groupID ? record.groupID : '-' }}</div>
    </template>
  </template>
  <template #expandedRowRender="{ record }">
    <ExpandRow :row="record" @openTendency="openTendency" />
  </template>
</a-table>
```



#### 控制expandRow的显隐

当部分record有expandRow而部分record没有expandRow时

```html
<!-- before -->
<div v-if="!record.NativeDevice" slot="expandedRowRender" slot-scope="record">
  <ExpandRow :row="record.VGPUs" />
</div>

<!-- now -->
<template #expandedRowRender="{ record }">
  <div v-if="!record.NativeDevice">
    <ExpandRow :row="record.VGPUs" />
  </div>
</template>
```



#### 自定义expandIcon

```js
// before
expandIcon (props) {
  if (props.record?.NativeDevice) {
    return
  }
  if (props.expanded) {
    return <i onClick={ () => { props.onExpand() } } />
	}
	return <i onClick={ () => { props.onExpand() } }></i>
}

// now
expandIcon (props) {
  if (props.record?.NativeDevice) {
    return
  }
  // onExpand函数需要传入record和事件e
  if (props.expanded) {
    return <i class="arrow_click" onClick={ e => { props.onExpand(props.record, e) } } />
	}
	return <i class="arrow_normal" onClick={ e => { props.onExpand(props.record, e) } }></i>
}
```



#### 样式变化

1. 有sorter的列hover上去时自带一个提示的tooltip
2. 每一列之间自带一个 divider 间隔 icon



#### sorter

antd2 中，若设为 `sorter: (a, b) => a - b`，则在 `@change` 回调函数中的 sorter 参数不会有值，我们代码中当其有值时才进行服务端的排序。

但在 antd3 中，不管是设为 `sorter: (a, b) => a - b` 还是设为 `sorter: true`，在 `@change` 回调函数中的 sorter 参数都会有值，所以这时我们在不排序的时候不应该设置 `@change` 回调函数。



### form

#### formModel 与 form 合并

formModel已去除，现在全部使用form

prop 更改为 name

```html
<!-- bofore -->
<a-form-model>
  <a-form-model-item prop="client_id"></a-form-model-item>
</a-form-model>

<!-- now -->
<a-form>
  <a-form-item name="client_id"></a-form-item>
</a-form>
```



#### 自定义校验规则

表单自定义校验规则更改为返回 Promise 方式，这个全局都要改

```js
// before
const validateOld = (rule, value, callback) => {
  if (value === '') {
    callback(new Error('xxx'))
  } else {
    callback()
  }
}

// now
const validateOld = async (rule, value) => {
  if (value === '') {
    return Promise.reject('xxx')
  } else {
    return Promise.resolve()
  }
}
```



### tooltip

#### 自定义指令

自定义指令需要写在原生DOM上

```html
<!-- before -->
<a-tooltip v-show="isShow">
  <i class="iconfont icon-ico_help" />
</a-tooltip>

<!-- now -->
<a-tooltip>
  <i v-show="isShow" class="iconfont icon-ico_help" />
</a-tooltip>
```



### input

#### 样式变化

1. [a-input-search](https://www.antdv.com/components/input-cn#components-input-demo-search-input)的样式发生了变化，由于结构变化宽度也会发生一定的变化

2. input 框如果被包裹在 div 下面，并且单独放一行或者跟按钮这种放在同一行，那默认高度会变成28px而不是32px，迁移时可以将 div 换成 template，或者添加 `allow-clear` 属性

   > 因为现在input会被包裹在一个span标签下，所以不会高度不会撑开

   ```html
   <!-- before -->
   <section class="flex">
     <div v-if="curTab === 'custom'">
       <a-input v-model="searchModel.allocationId" />
       <a-input v-model="searchModel.groupId" />
       <a-input v-model="searchModel.clientId" />
     </div>
     <div class="ml-4">
       <a-button @click="handleReset">
         {{ $t('monitor.record.reset') }}
       </a-button>
     </div>
   </section>
   
   <!-- now -->
   <section class="flex">
     <template v-if="curTab === 'custom'">
       <a-input v-model:value="searchModel.allocationId" />
       <a-input v-model:value="searchModel.groupId" />
       <a-input v-model:value="searchModel.clientId" />
     </template>
     <div class="ml-4">
       <a-button @click="handleReset">
         {{ $t('monitor.record.reset') }}
       </a-button>
     </div>
   </section>
   ```

3. 不是必要的话，建议不要在组件 dom 上加内联样式，可以在其内部或外部新写一个 template 或 div 再加样式。

   之前直接在 input dom 上添加的内联样式可能会发生变化，例如宽度，行高等（以前直接是input标签，现在是在input标签外套了一层span）

   ```html
   <a-input style="width: 330px; margin-right: 8px;" />
   ```

   需要重新检查下样式，因为dom结构变化了，所以添加上去的位置变化了



### drawer

#### 样式变化

close 关闭按钮从右边换到了左边



#### close 事件

下面的 drawer 声明了事件 `@close="handleClose"`

```html
<a-drawer
    placement="top"
    :visible="visible"
    :height="310"
    :mask="false"
    @close="handleClose"
    :bodyStyle="{
      paddingTop: '0'
    }"
  >
</a-drawer>
```

handleClose 如下

```js
handleClose () {
  this.$emit('close')
}
```

这时会报警告 `Invalid prop: type check failed for prop "onClose". Expected Function, got Array`，因为 drawer 在我们点击关闭时会自动帮我们触发这个 close 事件，所以我们不用重复声明，直接将 close 事件去掉就行。

但是此时点击蒙层是无法关闭抽屉的，因为它是 false，所以我们最好改成下面这样达到相同的效果，所以最终结果为下：

```html
<a-drawer
    placement="top"
    :visible="visible"
    :height="310"
    :maskStyle="{
      background: 'transparent'
    }"
    :bodyStyle="{
      paddingTop: '0'
    }"
  >
</a-drawer>
```





### date-picker, range-picker

#### 时间库更改

以前使用 moment 库，现在改为使用 dayjs 库

以前用到的 moment 的地方全都替换成 dayjs



#### 去除自定义样式

a-range-picker 以前可以自定义样式，现在只能是自带的样式

```html
<!-- before -->
<a-range-picker
  v-model="searchModel.sinceTime"
  format="YYYY-MM-DD HH:mm:ss"
  :placeholder="['', '']"
  :disabled-date="disabledDate"
  :show-time="{
    defaultValue: [moment('00:00:00', 'HH:mm:ss'), moment('23:59:59', 'HH:mm:ss')]
  }"
>
  <div>
    <span v-if="!searchModel.sinceTime">
      <a-icon type="calendar" />
      <div>xxx</div>
    </span>
  </div>
</a-range-picker>

<!-- now -->
<a-range-picker
  v-model:value="searchModel.sinceTime"
  format="YYYY-MM-DD HH:mm:ss"
  :placeholder="[$t('common.startTime'), $t('common.endTime')]"
  :disabled-date="disabledDate"
  :show-time="{
    defaultValue: [dayjs('00:00:00', 'HH:mm:ss'), dayjs('23:59:59', 'HH:mm:ss')]
  }"
>
  <template #suffixIcon>
    <calendar-outlined />
  </template>
</a-range-picker>
```



#### 日期选择器中文化

```js
// before
import moment from 'moment'
import 'moment/locale/zh-cn'
moment.locale('zh-cn')

// now
import dayjs from 'dayjs'
import 'dayjs/locale/zh-cn'
dayjs.locale('zh-cn')
```



#### 选择框内值清空的问题

要手动将值赋为null才能清空时间选择框展示的时间



#### 毫秒问题

时间选择器format到秒的时候，change回调中的时间值会随便给一个毫秒值，导致两个相同的时间不一样，所以需要手动指定精确到秒

```js
let [start, end] = values
start = start.endOf('s')
end = end.endOf('s')
```





#### ts中使用dayjs的问题

参考：https://dayjs.gitee.io/docs/zh-CN/installation/typescript



### select

#### 带搜索功能的select框迁移

参考官方文档用法

```html
<!-- before -->
<a-select option-filter-prop="children" :filter-option="filterOption"></a-select>

<!-- now -->
<a-select :filter-option="filterOption"></a-select>
```

```js
// before
methods: {
  filterOption (input, option) {
    return option.componentOptions.children[0].text.toLowerCase().indexOf(input.toLowerCase()) >= 0
  }
}

// now
methods: {
  filterOption (input, option) {
    return option.value.toLowerCase().indexOf(input.toLowerCase()) >= 0
  }
}
```

####  dropdownRender 语法更改

使用 `dropdownRender` 对下拉菜单进行自由扩展时，语法需要修改

```html
<!-- before -->
<template>
  <a-select
    v-model="value"
    style="width: 120px"
    :options="items.map(item => ({ value: item }))"
  >
    <template slot="dropdownRender" slot-scope="menu">
      <v-nodes :vnodes="menu" />
      <a-divider style="margin: 4px 0" />
      <div
        style="padding: 4px 8px; cursor: pointer"
        @mousedown="e => e.preventDefault()"
        @click="addItem"
      >
        <plus-outlined />
        Add item
      </div>
    </template>
  </a-select>
</template>
<script>
export default {
  components: {
    VNodes: {
      functional: true,
      render: (h, ctx) => ctx.props.vnodes
    }
  }
}
</script>

<!-- now -->
<template>
  <a-select
    v-model:value="value"
    style="width: 120px"
    :options="items.map(item => ({ value: item }))"
  >
    <template #dropdownRender="{ menuNode: menu }">
      <v-nodes :vnodes="menu" />
      <a-divider style="margin: 4px 0" />
      <div
        style="padding: 4px 8px; cursor: pointer"
        @mousedown="e => e.preventDefault()"
        @click="addItem"
      >
        <plus-outlined />
        Add item
      </div>
    </template>
  </a-select>
</template>
<script lang="ts">
export default defineComponent({
  components: {
    VNodes: (_, { attrs }) => {
      return attrs.vnodes
    }
  }
})
</script>
```



### icon

a-icon组件已去除，现在每一个icon都单独作为一个组件引入进来



### empty

暂无数据的背景色会变化



### 通用

1. v-model需要绑定具名事件，例如：`v-model:value`, `v-model:visible`
2. dom元素上的slot属性定义方式更换：`slot="title" slot-scope="text, record"` 换成 `#title="{ text, record }"`
3. 样式问题无法避免，antd3的class结构有很多更改，影响到很多之前自定义样式的class选择器，需要找到每个组件的自定义样式重新修改class



## 迁移至ts

### 配置

package.json

```json
"devDependencies": {
  "@typescript-eslint/eslint-plugin": "^5.4.0",
  "@typescript-eslint/parser": "^5.4.0",
  "@vue/cli-plugin-babel": "~5.0.0",
  "@vue/cli-plugin-eslint": "~5.0.0",
  "@vue/cli-plugin-typescript": "~5.0.0",
  "@vue/cli-service": "~5.0.0",
  "@vue/eslint-config-standard": "^6.1.0",
  "@vue/eslint-config-typescript": "^9.1.0",
  "eslint": "^7.32.0",
  "eslint-plugin-import": "^2.25.3",
  "eslint-plugin-node": "^11.1.0",
  "eslint-plugin-promise": "^5.1.0",
  "eslint-plugin-vue": "^8.0.3",
  "typescript": "~4.5.5"
}
```



tsconfig.json（具体每个配置项的作用可以查看ts官方配置文档）

```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    "strict": true,
    "jsx": "preserve",
    "moduleResolution": "node",
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "useDefineForClassFields": true,
    "sourceMap": true,
    "allowJs": true, // ❗️添加allowJs才允许引入js文件，不然会报错：Could not find declaration file for module 'X' Error
    "baseUrl": ".",
    "types": [
      "webpack-env"
    ],
    "paths": {
      "@/*": [
        "src/*"
      ]
    },
    "lib": [
      "esnext",
      "dom",
      "dom.iterable",
      "scripthost"
    ]
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.js",
    "src/**/*.tsx",
    "src/**/*.vue",
    "tests/**/*.ts",
    "tests/**/*.tsx"
  ],
  "exclude": [
    "node_modules"
  ]
}
```



让ts识别.vue文件，在src目录下创建 `shims.vue.d.ts`

```ts
/* eslint-disable */
declare module '*.vue' {
  import type { DefineComponent } from 'vue'
  const component: DefineComponent<{}, {}, any>
  export default component
}
```



.eslintrc.js

```js
extends: [
  'plugin:vue/vue3-essential',
  'eslint:recommended',
  '@vue/typescript/recommended'
],
parserOptions: {
  ecmaVersion: 2020
}
```



### 语法迁移

#### dom 类型校验

TypeScript does not have the `style` property on `Element`.

所以Element变量不能对style做修改

```js
// before
const buildInfo = document.createElement('div')
buildInfo.style = 'position: fixed; z-index: 99999; bottom: 0; right: 0; padding: 5px 15px; background: green; color: white;'

// now
const buildInfo = document.createElement('div')
buildInfo.setAttribute('style', 'position: fixed; z-index: 99999; bottom: 0; right: 0; padding: 5px 15px; background: green; color: white;')
```



#### 语法过渡

js向ts过渡时，暂时将所有ts文件中的类型用any写



## eslint

`.eslintrc.js`

```js
module.exports = {
  root: true,
  env: {
    node: true
  },
  extends: [
    'plugin:vue/vue3-essential',
    'eslint:recommended',
    '@vue/typescript/recommended'
  ],
  parserOptions: {
    ecmaVersion: 2020
  },
  rules: {
    'vue/require-default-prop': 'off',
    'vue/multi-word-component-names': 'off',
    'no-unused-vars': 'off',
    '@typescript-eslint/no-unused-vars': 'off',
    "@typescript-eslint/no-empty-function": 'off',
    "@typescript-eslint/no-explicit-any": 'off'
  }
}
```



## Vue3的新特性

Vue3英文文档：https://vuejs.org/

Vue3中文文档：https://staging-cn.vuejs.org/

迁移参考文档：

https://v3-migration.vuejs.org/breaking-changes/

https://v3.cn.vuejs.org/guide/migration/introduction.html#%E6%A6%82%E8%A7%88



### [Composition API](https://vuejs.org/guide/extras/composition-api-faq.html)

### [SFC Composition API Syntax Sugar (`<script setup>`)](https://vuejs.org/api/sfc-script-setup.html)

### [Teleport](https://vuejs.org/guide/built-ins/teleport.html)

### [Fragments](https://v3-migration.vuejs.org/new/fragments.html)

在Vue3中，组件现在正式支持多根节点组件，也就是Fragments

```html
<!-- vue2.x Layout.vue -->
<template>
  <div>
    <header>...</header>
    <main>...</main>
    <footer>...</footer>
  </div>
</template>

<!-- vue3.x Layout.vue -->
<template>
  <header>...</header>
  <main v-bind="$attrs">...</main>
  <footer>...</footer>
</template>
```

> 在3.x中，我们可以有多个根结点，但注意，我们需要明确的声明哪一个节点来继承父节点上的 attributes，例如上面的`<main v-bind="$attrs">...</main>` ，即是将父节点上的 attributes 放到 main 节点上
>
> 关于 attributes 的继承规则，参考 https://vuejs.org/guide/components/attrs.html#fallthrough-attributes



### [Emits Component Option](https://vuejs.org/api/options-state.html#emits)

emits 声明由组件触发的自定义事件

在3.x中，我们可以以两种形式声明触发的事件：

- 使用字符串数组的简易形式。
- 使用对象的完整形式。该对象的每个 property 键是事件的名称，值是 `null` 或一个验证函数。



数组语法：

```js
export default {
  emits: ['check'],
  created() {
    this.$emit('check')
  }
}
```

对象语法：

```js
export default {
  emits: {
    // 没有验证函数
    click: null,

    // 具有验证函数
    submit: (payload) => {
      if (payload.email && payload.password) {
        return true
      } else {
        console.warn(`Invalid submit event payload!`)
        return false
      }
    }
  }
}
```

> 官方文档强烈建议使用 `emits` 记录每个组件所触发的所有事件。这尤为重要，因为在3.x中[移除了 `.native` 修饰符](https://v3.cn.vuejs.org/guide/migration/v-on-native-modifier-removed.html)。任何未在 `emits` 中声明的事件监听器都会被算入组件的 `$attrs`，并将默认绑定到组件的根节点上。
>
> 详情参考：https://v3.cn.vuejs.org/guide/migration/emits-option.html



### [`createRenderer` API from `@vue/runtime-core`](https://vuejs.org/api/custom-renderer.html) to create custom renderers

在3.x中，我们可以创建一个自定义渲染器。通过提供平台特定的节点创建以及更改 API，可以在非 DOM 环境中也享受到 Vue 核心运行时的特性。

### [SFC State-driven CSS Variables (`v-bind` in `style`)](https://vuejs.org/api/sfc-css-features.html#v-bind-in-css)

单文件组件的 `<style>` 标签支持使用 `v-bind` CSS 函数将 CSS 的值链接到动态的组件状态：

```html
<template>
  <div class="text">hello</div>
</template>

<script>
export default {
  data() {
    return {
      color: 'red'
    }
  }
}
</script>

<style>
.text {
  color: v-bind(color);
}
</style>
```



### [SFC `<style scoped>` can now include global rules or rules that target only slotted content](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0023-scoped-styles-changes.md)

带 `scoped` 的 style 现在可以包括全局规则或只针对槽内容的规则：

```html
<style scoped>
/* deep selectors */
::v-deep(.foo) {}
/* shorthand */
:deep(.foo) {}

/* targeting slot content */
::v-slotted(.foo) {}
/* shorthand */
:slotted(.foo) {}

/* one-off global rule */
::v-global(.foo) {}
/* shorthand */
:global(.foo) {}
</style>
```

所以，之前`/deep/`的写法需要全部换成`:deep()`

### [Suspense](https://vuejs.org/guide/built-ins/suspense.html)

`<Suspense>` 是一项实验性功能。不能保证它会成为稳定版，而且在那之前，相关 API 也可能会发生变化。



## Vue3语法迁移与变化

### 全局API

Vue 2.x 有许多全局 API 和配置，它们可以全局改变 Vue 的行为，例如：Vue.component, Vue.directive 等。

Vue3.x 中我们调用 `createApp` 返回一个应用实例 app

```js
import { createApp } from 'vue'

const app = createApp({})
```

任何全局改变 Vue 行为的 API 现在都会移动到应用实例上：

| 2.x 全局 API               | 3.x 实例 API (`app`)                       |
| -------------------------- | ------------------------------------------ |
| Vue.config                 | app.config                                 |
| Vue.config.productionTip   | 移除                                       |
| Vue.config.ignoredElements | app.config.compilerOptions.isCustomElement |
| Vue.component              | app.component                              |
| Vue.directive              | app.directive                              |
| Vue.mixin                  | app.mixin                                  |
| Vue.use                    | app.use                                    |
| Vue.prototype              | app.config.globalProperties                |
| Vue.extend                 | 移除                                       |

所有其他不全局改变行为的全局 API 现在都是具名导出：

- Vue.nextTick
- Vue.observable (用 Vue.reactive 替换)
- Vue.version
- Vue.compile
- Vue.set
- Vue.delete

示例：

```js
import { nextTick } from 'vue'
```

> 因为上述例如 `Vue.nextTick()` 这样的全局 API 是不支持 tree-shake 的，不管它们实际上是否被使用了，都会被包含在最终的打包产物中。在 Vue 3 中，全局和内部 API 都经过了重构，并考虑到了 tree-shaking 的支持。因此，对于 ES 模块构建版本来说，全局 API 现在通过具名导出进行访问。



除了公共 API，许多`内部组件/帮助器`现在也以具名的方式导出。这允许编译器只在代码被使用到时才引入并输出它。例如以下模板：

```html
<transition>
  <div v-show="ok">hello</div>
</transition>
```

将编译为类似于以下的内容：

```js
import { h, Transition, withDirectives, vShow } from 'vue'

export function render() {
  return h(Transition, [withDirectives(h('div', 'hello'), [[vShow, this.ok]])])
}
```

这实际上意味着只有在应用实际使用了 `Transition` 组件它才会被导入。换句话说，如果应用没有使用任何 `Transition` 组件，那么用于支持此功能的代码将不会出现在最终的打包产物中。



### v-model

#### 用于自定义组件时，v-model prop 和 event 默认名称已更改

- prop：`value` -> `modelValue`
- 事件：`input` -> `update:modelValue`

> 所以之前如果在父组件 dom 上写了v-model，并且子组件里emit了input事件的，现在需要手动监听这个input事件。



#### `v-bind` 的 `.sync` 修饰符和组件的 `model` 选项已移除，可在 `v-model` 上加一个参数代替

在 Vue 2.0 中，开发者使用 `v-model` 指令时必须使用名为 `value` 的 prop。如果开发者出于不同的目的需要使用其他的 prop，他们就不得不使用 `v-bind.sync` 或者 `model` 选项。

但在 3.x 中，我们可以直接写成 `v-model:xxx` 后面这个 xxx 参数则是我们自定义的 prop 的名称，所以我们也就不再需要 .sync 和 model



检查代码中所有使用 `.sync` 的部分并将其替换为 `v-model`：

```html
<ChildComponent :title.sync="pageTitle" />

<!-- 替换为 -->

<ChildComponent v-model:title="pageTitle" />
```

对于所有不带参数的 `v-model`，确保分别将 prop 和 event 命名更改为 `modelValue` 和 `update:modelValue`

```html
<ChildComponent v-model="pageTitle" />
```

```js
// ChildComponent.vue

export default {
  props: {
    modelValue: String // 以前是`value：String`
  },
  emits: ['update:modelValue'],
  methods: {
    changePageTitle(title) {
      this.$emit('update:modelValue', title) // 以前是 `this.$emit('input', title)`
    }
  }
}
```



注意使用修饰符时的顺序

```html
<!-- before -->
<a-input v-model.number="formModel.search"></a-input>

<!-- now -->
<a-input v-model:value.number="formModel.search"></a-input>

<!-- bad -->
<a-input v-model.number:value="formModel.search"></a-input>
```



#### 新增功能

- 现在可以在同一个组件上使用多个 `v-model` 绑定
- 现在可以自定义 `v-model` 修饰符



### key attribute

- 对于 `v-if`/`v-else`/`v-else-if` 的各分支项 `key` 将不再是必须的，因为现在 Vue 会自动生成唯一的 `key`。（如果手动提供 `key`，那么每个分支必须使用唯一的 `key`。你将不再能通过故意使用相同的 `key` 来强制重用分支。）
- `<template v-for>` 的 `key` 应该设置在 `<template>` 标签上 (而不是设置在它的子节点上)。



### v-if 和 v-for 的优先级

2.x 版本中在一个元素上同时使用 `v-if` 和 `v-for` 时，`v-for` 会优先作用。

3.x 版本中则相反 `v-if` 总是优先于 `v-for` 生效。



但我们还是需要避免在同一元素上同时使用两者



### `v-on.native`修饰符移除

`v-on` 的 `.native` 修饰符已被移除。

在3.x中新增的[`emits` 选项](https://v3.cn.vuejs.org/guide/migration/emits-option.html)允许子组件定义真正会被触发的事件。因此，对于子组件中未被定义为组件触发的所有事件监听器，Vue 现在会直接将它们作为原生事件监听器添加到子组件的根元素中 (除非在子组件的选项中设置了 `inheritAttrs: false`)。所以我们也不再需要 .native 修饰。



### render 函数的 API 变化

`h` 现在是全局导入，而不是作为参数传递给渲染函数

```js
// before
export default {
  render (h) {
    return h('div')
  }
}

// now
import { h } from 'vue'
export default {
  render () {
    return h('div')
  }
}
```

否则可能会遇到 `h is not defined` 的错误



### slots 用法统一

- 将所有 `this.$scopedSlots` 替换为 `this.$slots`。
- 将所有 `this.$slots.mySlot` 替换为 `this.$slots.mySlot()`。



### `$attrs`

在2.x中，`class` 和 `style` 与其他的 attribute 不一样，他们没有被包含在 `$attrs` 中。

在3.x中， `$attrs` 包含了**所有的** attribute。



### 被移除的一些 API

#### 数字按键修饰符

不再支持使用数字 (即键码) 作为 `v-on` 修饰符。

现在建议对任何要用作修饰符的键使用 kebab-cased (短横线) 名称。

```html
<!-- before -->
<input v-on:keyup.13="submit" />

<!-- now -->
<input v-on:keyup.page-down="nextPage">

<!-- 同时匹配 q 和 Q -->
<input v-on:keypress.q="quit">
```



#### 事件 API

`$on`，`$off` 和 `$once` 实例方法已被移除，组件实例不再实现事件触发接口。

所以之前使用 eventBus 事件总线的代码需要替换掉

```js
const eventBus = new Vue()
eventBus.$on('custom-event', () => {
  console.log('Custom event triggered!')
})
eventBus.$emit('custom-event')
```

现在我们的事件总线模式可以替换为使用外部的、实现了事件触发器接口的库，例如官方推荐的库 [mitt](https://github.com/developit/mitt)

```js
import mitt from 'mitt'
const eventBus = mitt()
eventBus.$on('custom-event', () => {
  console.log('Custom event triggered!')
})
eventBus.$emit('custom-event')
```



#### filters

过滤器已移除，且不再支持



#### $children

在 2.x 中，开发者可以使用 `this.$children` 访问当前实例的直接子组件，在 Vue 3.x 中将 `$children` property 移除且不再支持。



#### Vue.$set被移除，不再需要

参考：https://v3.cn.vuejs.org/guide/migration/migration-build.html#%E5%AE%8C%E5%85%A8%E5%85%BC%E5%AE%B9



### 自定义指令 directives

自定义指令的钩子函数已经被重命名，以更好地与组件的生命周期保持一致。

- **created** - 新增，在元素的 attribute 或事件监听器被应用之前调用。
- bind → **beforeMount**
- inserted → **mounted**
- **beforeUpdate**：新增，在元素本身被更新之前调用，与组件的生命周期钩子十分相似。
- update → 移除，该钩子与 `updated` 有太多相似之处，因此它是多余的。请改用 `updated`。
- componentUpdated → **updated**
- **beforeUnmount**：新增，与组件的生命周期钩子类似，它将在元素被卸载之前调用。
- unbind -> **unmounted**



### data

#### 组件选项 `data` 的声明不再接收纯 JavaScript `object`，而是只接收一个 `function`

#### mixin合并行为变更

当来自组件的 `data()` 及其 mixin 被合并时，合并操作现在将被**浅层次**地执行：

```js
const Mixin = {
  data() {
    return {
      user: {
        name: 'Jack',
        id: 1
      }
    }
  }
}

const CompA = {
  mixins: [Mixin],
  data() {
    return {
      user: {
        id: 2
      }
    }
  }
}
```

在 Vue 2.x 中，生成的 `$data` 是：

```js
{
  "user": {
    "id": 2,
    "name": "Jack"
  }
}
```

在 3.0 中，其结果是：

```js
{
  "user": {
    "id": 2
  }
}
```



### 在 prop 的默认函数中访问`this`

生成 prop 默认值的工厂函数不再能访问 `this`。组件接收到的原始 prop 将作为参数传递给默认函数，也可以使用 `inject` 来访问注入的 property。

```js
import { inject } from 'vue'

export default {
  props: {
    theme: {
      default (props) {
        // `props` 是传递给组件的，在任何类型/默认强制转换之前的原始值
        // 也可以使用 `inject` 来访问注入的 property
        return inject('theme', 'default-theme')
      }
    }
  }
}
```



### 过渡动画的class类名更改

过渡类名 `v-enter` 修改为 `v-enter-from`

过渡类名 `v-leave` 修改为 `v-leave-from`



## 使用 `@vue/compat` 进行 vue3 的迁移

参考：

https://v3-migration.vuejs.org/migration-build.html

https://zhuanlan.zhihu.com/p/400438477





- vue3插件引入方式
- 是否只是弹层类的组件不能写v-if指令

