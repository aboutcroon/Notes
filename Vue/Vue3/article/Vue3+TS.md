defineComponent 是可以接收四种不同的参数类型的

Vue3当中函数组件就是一个function，不像vue2中要通过一个functional来定义



对于vue3，props的使用仍然是需要声明的

```tsx
props: {
  age: {
    type: Number as PropType<number>
  }
}
```

写一个统一的以复用

```js
const PropsType = {
  msg: String,
  age: {
    type: Number,
    required: true
  }
} as const
```

> props如果直接写在defineComponent里面，则会默认它是readOnly类型，但是如果写在外面就没有上下文来使其是readOnly类型，这时我们就需要加上 `as const`。
>
> 源码里的注释:
>
> *// the Readonly constraint allows TS to treat the type of { required: true }*
>
> *// as constant instead of boolean.*



vue模版最终会转换成h函数，h函数中最终调用的是creatVnode函数，也就相当于我们直接用createVNode函数来写



使用this.xxx时，其实是调用了this.proxy，当vue发现xxx是一个ref时，会自动将其解除ref然后返回它的value



h函数可以放在 `render()` 中，也可以放在 `setup()的return中`，reactive和ref变量改变会引起setup的return函数的重新执行，但setup函数中的内容只执行一次，所以对变量的某些引用应该放在return函数中



如果子组件有一个props是required，但是父组件不传值给他时vscode也不会报错，因为vue文件export出来的是一个通用的类型，并不会说给每个子组件专门有一个类型。解决方案就是，将vue文件改成jsx文件



