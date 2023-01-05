## tailwindcss.config.js

这个配置文件，theme 里面扩展的样式必须写到 extends 里面，不然的话算是重置该样式



更改 html 的 font-size，就会改变 tailwindcss 默认乘以 4 的规则，所以我们应该更改 body 的



main.js 中如果在引入 tailwind 之后再引入 iconfont ，那么我们在如下 html 上时

```html
 <i class="iconfont icon-icon_enlarge_cancel mr-0.5 text-xl" />
```

这样 iconfont 的样式就会覆盖 tailwind 的样式

所以我们需要在引入 tailwind 之前引入 iconfont



## preflight

tailwindcss 会有一些固定样式，如果不需要它们，可以将 preflight 变为 false

```js
corePlugins: {
  container: false,
  preflight: false
}
```

