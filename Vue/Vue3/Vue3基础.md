## 一些 issue

https://github.com/vuejs/vue-next/issues/1167



## TS 相关

### JSON.parse

```ts
const tokenInfo = JSON.parse(localStorage.getItem('tokenInfo') || '{}')
```

这里，如果我们不加个后面的兼容内容，那么就会报错 `Argument of type 'string | null' is not assignable to parameter of type 'string'.
  Type 'null' is not assignable to type 'string'.`

因为 `JSON.parse` 中的内容永远是要一个 string，如果为 null 的话，我们就要兼容一下，并且我们不能写成`''` 空字符串，必须要写成 `'{}'`，不然还会报 `Unexpected end of JSON input` 的错误



## TSX相关

引入组件时，组件的文件命名一定要是大驼峰，不能是小驼峰，否则会出现 `The requested module './xxx.js' does not provide an export named 'xxx'` 的报错



有关 TSX 使用 vue 中的一些指令的参考（例如v-show， v-model）

https://github.com/cangshudada/vite-vue3-tsx



Vue3 中可以使用 TSX 语法，是因为有 `@vue/babel-plugin-jsx` 编译，而如果使用 vite，则是 `@vitejs/plugin-vue-jsx`，使用规则他们两个都一样



## VueRouter

```ts
import { useRouter, RouteLocationOptions } from 'vue-router'
const router = useRouter()

router.push({
  name: 'detail',
  params: {
    text: item.text,
    done: item.done
  }
} as RouteLocationOptions)
```

一定要用 RouteLocationOptions 类型

参考文章：https://www.chuckliu.site/2021/03/09/vue3-ts-%E5%88%9D%E4%BD%93%E9%AA%8C/



## globalProperties

```ts
// main.ts
/** 将全局静态配置注入到应用中  */
app.config.globalProperties.$filter = {
  foo () {
    return 123
  },
  bar () {
    console.log('我是bar')
    return '我是bar'
  }
}
```

在 template 中使用

```ts
{{ $filter.foo() }}
```

在setup中使用

```ts
import { defineComponent, getCurrentInstance } from 'vue'
export default defineComponent({
  setup () {
    const current = getCurrentInstance()
    console.log(current.appContext.config.globalProperties.$filter.bar())
  }
})
```

参考文章：https://blog.csdn.net/qq_45223390/article/details/120825646?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-0.no_search_link&spm=1001.2101.3001.4242.1



## CSS

可以使用 CSS module

[vue3+ts jsx写法css module处理方案](https://www.jianshu.com/p/be1778a76763)

但是推荐 tailwind 来实现 Atomic/Utility-First CSS 解决方案（原子化CSS）



## 状态管理

参考文章：

https://juejin.cn/post/6955387358728421383#comment

https://zhuanlan.zhihu.com/p/351519484

https://juejin.cn/post/6844904131610542087



## 日常使用

### watch

watch reactive 变量时：

```ts
watch(() => selectedKeys.roleKeys, (roleKeys) => {
  userListFilter.value = roleKeys
})
```



watch ref 变量时：

```ts
watch(userList, (value) => {
  userListFilter.value = value
})
```



### 响应式

```ts
const userList = ref([])
const userListFilter = ref(userList.value)
```

当 userList 有值时，template 中使用到 userList 的会重新渲染，但是 userListFilter 并不会获得新的值，需要像下面这样才可以：

```ts
const userList = ref([])
const userListFilter = ref(userList.value)
watch(userList, (value) => {
  userListFilter.value = value
})
```



### template ref

```html
<div ref="test">
```

```js
setup () {
  const test = ref(null)
  onMounted(() => {
    console.log(test)
  })
  return {
    test
  }
}
```

使用 template ref 时，要记得将其 return 出去，然后在 mounted 钩子中才能取到值。

