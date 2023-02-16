# 1. white-space: nowrap

规定段落中的文本不进行换行，其可能的值为：

| 值       | 描述                                                         |
| :------- | :----------------------------------------------------------- |
| normal   | 默认。空白会被浏览器忽略。                                   |
| pre      | 空白会被浏览器保留。其行为方式类似 HTML 中的 \<pre\> 标签。  |
| nowrap   | 文本不会换行，文本会在在同一行上继续，直到遇到 \<br\> 标签为止。 |
| pre-wrap | 保留空白符序列，但是正常地进行换行。                         |
| pre-line | 合并空白符序列，但是保留换行符。                             |
| inherit  | 规定应该从父元素继承 white-space 属性的值。                  |



# 2. 渐变色

CSS `linear-gradient()` 函数用于创建一个表示两种或多种颜色线性渐变的图片。其结果属于 \<gradient\> 数据类型，是一种特别的 \<image\> 数据类型。

```css
/* 渐变轴为45度，从蓝色渐变到红色 */
/* 若不规定方向，则方向默认为从上到下 */
linear-gradient(45deg, blue, red);

/* 从右下到左上、从蓝色渐变到红色 */
linear-gradient(to left top, blue, red);

/* 从下到上，从蓝色开始渐变、到高度40%位置是绿色渐变开始、最后以红色结束 */
linear-gradient(0deg, blue, green 40%, red);
```

参考网址：https://developer.mozilla.org/zh-CN/docs/Web/CSS/linear-gradient()

https://www.runoob.com/css3/css3-gradients.html



# 3. opacity遮挡字体

若给 div 标签设置 opacity 属性，则在 div 中书写的文字也会自动继承 opacity 属性，则文字也会变成半透明从而显示不出颜色。

解决方案：

1. 用 css3 的 rgba 颜色 rgba(0,0,0,0.5)，最后那个0.5是透明度。缺点是老版本ie不支持rgba。
2. 里面的 div1、2 移出来不要和最外面 div 成为父子关系(就不会继承)，然后大 div 依然为opacity: 0.85，再用定位之类的办法把 div1、2 移动到大 div 区域上面去。

`一般使用第二种，使其不继承。`

`opacity` 参考网址：https://developer.mozilla.org/zh-CN/docs/Web/CSS/opacity



# 4. cursor: pointer

使其放上去有手指显示

与 hover 伪类一样

参考网址：https://developer.mozilla.org/zh-CN/docs/Web/CSS/cursor



# 5. css样式中变灰的部分是继承而来的

在开发者模式中，有的 css 样式被线划掉了，是被覆盖掉了；有的样式变灰了，说明是继承而来的属性，是有生效的。

参考：https://zhidao.baidu.com/question/1452490958990633260.html



# 6. flex 布局

flex-around 周边会留有间距，flex-between 周边不会留有间距。



# 7. min-width min-height

1. 通常在界面缩太小后，使得 div 中的元素被挤压，这时给 div 设置 min-width

2. ❗️在不想让图片随着界面的缩小而缩小而影响图片质量时，要设置 `max-width = 100%` 和 `max-height = 100%`

3. 页面 ajax 数据未返回时，放置数据的地方为空，则没有宽度会变扁，然后数据返回来之后又会撑起来，这时就会抖动。防止抖动可以给其设置 min-width 和 min-height



# 8. iconfont 和 svg 的冲突

要在页面上更方便的批量使用 svg，则需要使用 svg-sprite-loader 插件。

使用该插件时，我们自定义配置文件，会习惯性的将以 icon 开头的文件夹下的所有 svg 格式文件给引入，如果你放置 iconfont 的文件夹名称也是 icon 开头，则 iconfont.svg 也会被错误的引入进去，此时需要更换文件夹名，例如改成 font。



# 9. css 选择器

## 9.1 `>` 的作用

CSS中的>是代表选择器的层级关系为直接子级。如下示例代码：

```html
<body>
    <div class="wrap"></div>
    <div class="content">
        <div class="wrap"></div>
    </div>
</body>
```

body .wrap{}  这样写控制的就是 body 里边的所有 class 为 wrap 的元素，包括所有子孙级。

body > .wrap{} 这样写就只会控制直接子级，content 里边的 wrap 就不会受控制。 

## 9.2 `::before`

统一模版：

```less
span {
  color: @blue;
  &::before {
    content: '';
    position: absolute;
    left: 0;
    top: 50%;
    transform: translateY(-50%);
    width: 4px;
    height: 35%;
    background: blue;
    border-top-right-radius: 5px;
    border-bottom-right-radius: 5px;
    transition: transform 0.15s cubic-bezier(0.215, 0.61, 0.355, 1), opacity 0.15s cubic-bezier(0.215, 0.61, 0.355, 1);
  }
}
```



## 9.3 `::after` 伪元素的继承



## 9.4 `:not()`

```css
.fancy {
  text-shadow: 2px 2px 3px gold;
}

/* 类名不是 `.fancy` 的 <p> 元素 */
p:not(.fancy) {
  color: green;
}

/* 非 <p> 元素 */
body :not(p) {
  text-decoration: underline;
}
```

参考网址：https://developer.mozilla.org/zh-CN/docs/Web/CSS/:not



# 10. border-radius

border-radius 也是一个简写属性

四个属性分别为：`border-top-left-radius` `border-top-right-radius` `border-bottom-right-radius` `border-bottom-left-radius`

参考网址：https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-radius



# 11. 动画中的 ease seae-in ease-in-out ease-out

| 值                    | 描述                                                         |
| :-------------------- | :----------------------------------------------------------- |
| linear                | 规定`以相同速度开始至结束`的过渡效果（等于 cubic-bezier(0,0,1,1)）。(匀速) |
| ease                  | 规定`慢速开始，然后变快，然后慢速结束`的过渡效果（cubic-bezier(0.25,0.1,0.25,1)）（相对于匀速，中间快，两头慢）。 |
| ease-in               | 规定以`慢速开始`的过渡效果（等于 cubic-bezier(0.42,0,1,1)）（相对于匀速，开始的时候慢，之后快）。 |
| ease-out              | 规定以`慢速结束`的过渡效果（等于 cubic-bezier(0,0,0.58,1)）（相对于匀速，开始时快，结束时候间慢，）。 |
| ease-in-out           | 规定以`慢速开始和结束`的过渡效果（等于 cubic-bezier(0.42,0,0.58,1)）（相对于匀速，（开始和结束都慢）两头慢）。 |
| cubic-bezier(n,n,n,n) | 在 cubic-bezier 函数中定义自己的值。可能的值是 0 至 1 之间的数值。 |

参考网址：https://blog.csdn.net/Candy_10181/article/details/80611009



# 12. 字体的 font-size 和 height

`字体的 height 是有作用效果的 ！！`

`height` 和 `line-height` 相同则会上下居中

`font-size` 决定字体大小

`width` 则通常不添加



# 13. CSS 引入顺序

CSS 文件的引入顺序也决定了哪个属性覆盖哪个属性

例如：

```js
import '@/common/plugins/antd.js'
import '@/assets/style/common.less'
import '@/assets/theme/antd.less'

// 这里全局样式的 body 会被引进来的 antd 的原有样式覆盖
// 但自定义的 antd.less 中写的 body 属性 又会覆盖掉 antd 的原有样式
```



# 14. flex

## 14.1 flex 中的 span

```html
<div class="flex">
    <span class="title">GPU</span>
    <span>已使用/总数：</span>
    <span>显存：</span>
</div>
```

当外层 div 开启 flex 定位之后，里面的 span 元素也会变成块级，随之 span 上的 margin-bottom 也会生效



## 14.2 两侧固定宽度，中间自适应

flex 实现左右两侧宽度固定，中间自适应

这时当外层 box 宽度增大时，中间元素会自动增大，但当 box 宽度变小时，中间元素宽度不会变，从而会挤压两侧的宽度

解决方案：使中间中间元素变成 BFC，overflow: hidden，这是不影响其他因素的原理



## 14.3 flex布局当overflow时无法滑动到顶部的问题

加一个margin: auto

原理是：当子元素超过了父元素的高度时，如果我们针对子元素设置了`margin:auto;`, 那么flex布局会采用我们的*margin*属性来布局, 顶部就可以滑动到了。

[https://love.fishman.wang/2016/05/21/flex%E5%B8%83%E5%B1%80%E5%BD%93overflow%E6%97%B6%E6%97%A0%E6%B3%95%E6%BB%91%E5%8A%A8%E5%88%B0%E9%A1%B6%E9%83%A8/](https://love.fishman.wang/2016/05/21/flex布局当overflow时无法滑动到顶部/)



## 14.4 flex 布局一行放不下时换行

flex 布局一行放不下时换行，使用 flex-wrap



## flex 子元素 height: 100%

父元素 flex，子元素  height: 100%，当子元素超出父元素时会自动 shrink，但子元素设置 min-height: 100% 时，超出部分不会自动 shrink

当父元素flex, 子元素 min-height: 100% 时，孙元素的高度无法撑开子元素。将父元素的flex去掉即可



# 15. height: 100% 问题

给div 加滚动条，高度为百分比时不生效，此时要在所有父级元素加上 height: 100%

参考网址：https://www.365jz.com/article/24849



当是如下结构时：

```html
<div class="box">
  <div class="a"> </div>
  <div class="b"> </div>
</div>
```

如果 box 有一个高度，然后 a 有一个高度，b 的高度设置为 100%，那么当 box 中的内容滑动时，因为 b 的高度会超出去，所以会出现没办法滑到最底部的问题，所以此时 b 的高度要设置为 calc(100% - a)

但如果 box 是 flex 布局的话，就不会有这个问题，因为这样 flex 子元素的 `flex-shrink: 1`，会自动压缩，如果把 `flex-shrink: 0` 则不会自动压缩



最外层的包裹元素 height: 100% 最好配合着 min-height 一起使用，这样子页面在向上收缩时，能有个最小高度，所以就不会使得最外层包裹元素高度太小，而内容的高度都超出了包裹元素的高度



# 16. div

div 块级元素，width 自动为 100% 占满整行，但高度不会



# 17. css 实现进度条

```html
<div class="line">
  <div class="pass" :style="{ width: percent }" />
</div>

<div class="today">
  <span v-show="percent !== '100%'" :style="{ left: percent }">
     {{ percent === '100%' ? '已失效' : '今天' }}
  </span>
</div>
```

`:style="{ width: percent }"` 和 `:style="{ left: percent }"` 分别实现进度还有进度上的文字



# 18. 业务

父级元素可以使用 flex 定位，使内部绝对定位的子元素居中

transform: translate 只能用于 block 元素

`cursor: not-allowed` 不能与 `pointer-events:none` 同时使用



# 19. inherit

## line-height

line-height 可以直接继承父选择器的，但 height 不行，所以一般有 `height: 100%`，但没有 line-height: 100%

