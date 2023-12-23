## 1.CSS基础

### CSS选择器及其优先级

| **选择器**     | **格式**      | **优先级权重** |
| -------------- | ------------- | -------------- |
| 内联样式       | style         | 1000           |
| id选择器       | #id           | 100            |
| 类选择器       | #classname    | 10             |
| 属性选择器     | a[ref=“eee”]  | 10             |
| 伪类选择器     | li:last-child | 10             |
| 标签选择器     | div           | 1              |
| 伪元素选择器   | li::after     | 1              |
| 相邻兄弟选择器 | h1+p          | 0              |
| 子选择器       | ul>li         | 0              |
| 后代选择器     | li a          | 0              |
| 通配符选择器   | *             | 0              |

> 有几个注意事项
>
> - !important 声明的样式优先级最高
> - 如果优先级相同，则最后出现的样式生效
> - 继承得到的样式的优先级最低
> - 通用选择器（*）、子选择器（>）和相邻同胞选择器（+）并不在这四个等级中，所以它们的权值都为 0 
> - **样式表的来源**不同时，优先级顺序为：内联样式 > 内部样式 > 外部样式 > 浏览器用户自定义样式 > 浏览器默认样式。

### CSS中可继承与不可继承属性有哪些

**无继承性的属性**

1. **display**：规定元素应该生成的框的类型
2. **文本属性**：
   - vertical-align：垂直文本对齐
   - text-decoration：规定添加到文本的装饰
   - text-shadow：文本阴影效果
   - white-space：空白符的处理
   - unicode-bidi：设置文本的方向
3. **盒子模型的属性**：width、height、margin、border、padding、
4. **背景属性**：background、background-color、background-image、background-repeat、background-position、background-attachment
5. **定位属性**：float、clear、position、top、right、bottom、left、min-width、min-height、max-width、max-height、overflow、clip、z-index
6. **生成内容属性**：content、counter-reset、counter-increment
7. **轮廓样式属性**：outline-style、outline-width、outline-color、outline
8. **页面样式属性**：size、page-break-before、page-break-after
9. **声音样式属性**：pause-before、pause-after、pause、cue-before、cue-after、cue、play-during



**有继承性的属性**

1. **字体系列属性**

   - font-family：字体系列
   - font-weight：字体的粗细
   - font-size：字体的大小
   - font-style：字体的风格

2. **文本系列属性**

   - text-indent：文本缩进
   - text-align：文本水平对齐
   - line-height：行高
   - word-spacing：单词之间的间距
   - letter-spacing：中文或者字母之间的间距
   - text-transform：控制文本大小写（就是uppercase、lowercase、capitalize这三个）
   - color：文本颜色

3. **元素可见性**

   visibility：控制元素显示隐藏

4. **列表布局属性**

   list-style：列表风格，包括 list-style-type、list-style-image 等

5. **光标属性**

   cursor：光标显示为何种形态



### display 的属性值及其作用

| **属性值**   | **作用**                                                   |
| ------------ | ---------------------------------------------------------- |
| none         | 元素不显示，并且会从文档流中移除。                         |
| block        | 块类型。默认宽度为父元素宽度，可设置宽高，换行显示。       |
| inline       | 行内元素类型。默认宽度为内容宽度，不可设置宽高，同行显示。 |
| inline-block | 默认宽度为内容宽度，可以设置宽高，同行显示。               |
| list-item    | 像块类型元素一样显示，并添加样式列表标记。                 |
| table        | 此元素会作为块级表格来显示。                               |
| inherit      | 规定应该从父元素继承display属性的值。                      |



### 隐藏元素的方法有哪些

**display: none**：渲染树不会包含该渲染对象，因此该元素不会在页面中占据位置，也不会响应绑定的监听事件。

**visibility: hidden**：元素在页面中仍占据空间，但是不会响应绑定的监听事件。

**opacity: 0**：将元素的透明度设置为 0，以此来实现元素的隐藏。元素在页面中仍然占据空间，并且能够响应元素绑定的监听事件。

**position: absolute**：通过使用绝对定位将元素移除可视区域内，以此来实现元素的隐藏。

**z-index: 负值**：来使其他元素遮盖住该元素，以此来实现隐藏。

**clip/clip-path** ：使用元素裁剪的方法来实现元素的隐藏，这种方法下，元素仍在页面中占据位置，但是不会响应绑定的监听事件。

**transform: scale(0,0)**：将元素缩放为 0，来实现元素的隐藏。这种方法下，元素仍在页面中占据位置，但是不会响应绑定的监听事件。



### link 和 @import 的区别

两者都是外部引用 CSS 的方式，它们的区别如下：

- link 是 XHTML 标签，除了加载 CSS 外，还可以定义 RSS 等其他事务；@import 属于 CSS范畴，**只能加载 CSS**。
- link 引用 CSS 时，在**页面载入时同时加载**；@import 需要页面**网页完全载入以后加载**。
- link 是 XHTML 标签，**无兼容问题**；@import 是在 CSS2.1 提出的，低版本的浏览器不支持。
- link 支持使用 Javascript 控制 DOM 去改变样式；而 @import 不支持。



### 🔴 transition 和 animation 的区别

**transition是过渡属性**，强调过渡，它的实现需要触发一个事件（比如鼠标移动上去，焦点，点击等）才执行动画。它类似于 flash 的补间动画，设置一个开始关键帧，一个结束关键帧。

**animation是动画属性**，它的实现不需要触发事件，设定好时间之后可以自己执行，且可以循环一个动画。它也类似于 flash 的补间动画，但是它可以设置多个关键帧（用@keyframe定义）完成动画。



### 🔴 display:none 与 visibility:hidden 的作用及区别

这两个属性都是让元素隐藏，不可见。**两者区别如下：**

**在 Render tree 中**

- `display: none`会让元素完全从 Render tree 中消失，渲染时不会占据任何空间
- `visibility: hidden`不会让元素从 Render tree 中消失，渲染的元素还会占据相应的空间，只是内容不可见

**是否是继承属性**

- `display: none`是非继承属性，子孙节点会随着父节点从 Render tree 消失，通过修改子孙节点的属性也无法显示
- `visibility: hidden`是继承属性，子孙节点消失是由于继承了`hidden`，通过设置`visibility: visible`可以让子孙节点显示

**回流与重绘**

修改常规文档流中元素的 `display` 通常会造成文档的重排但是修改`visibility`属性只会造成本元素的重绘



### 🔴 伪元素和伪类的区别和作用

**伪类**

将特殊的效果添加到特定选择器上。它是已有元素上添加类别的，不会产生新的元素。例如：

```css
a:hover {
  color: #FF00FF
}
p:first-child {
  color: red
}
p:visited {
  color: red
}
p:focus {
  color: red
}
```

**伪元素**

在内容元素的前后插入额外的元素或样式，但是这些元素实际上并不在文档中生成。它们只在外部显示可见，但不会在文档的源代码中找到它们，因此，称为“伪”元素。例如：

```css
p::before {
  content:"第一章";
}
p::after {
  content:"Hot!";
}
p::first-line {
  background:red;
}
div::first-letter {
  color: red;
  font-size:30px;
}
```



**具体案例**

下面的伪类语法表达的意思是：**文档里的那些已经被用户访问过的 a 标签，当鼠标悬浮在它上面的时候，颜色为红色。**

```css
a:visited:hover {
  color: red;
}
```

![伪类](https://raw.githubusercontent.com/edwineo/Notes/main/CSS/interview/assets/%E4%BC%AA%E7%B1%BB.png)

可以看到，不管是 visited 还是 hover，都表示了 a 标签的**某种状态**，选择器始终选择的是伪类前面的元素，只不过当出现伪类描述的状态时，样式才生效



我们再来看一下伪元素的案例，下面的语法表达的意思是： **div 标签里面首字的颜色为红色。**

```css
div::first-letter {
  color: red;
}
```

![伪元素](https://raw.githubusercontent.com/edwineo/Notes/main/CSS/interview/assets/%E4%BC%AA%E5%85%83%E7%B4%A0.png)

与伪元素不同的是，“首字“并不是 div 元素的某种状态，而是**浏览器创建出来的一个虚拟的元素**，我们可以在上图右下角看到 `Pseudo ::first-letter element` 提示，相当于浏览器选中的第一个“彻”字

> 注意，::first-letter 伪元素只作用于块状元素上面，如果把 div 换成 span，就无法选中首字了，除非手动设置 span 的 display 属性为 block 或 inline-block 等块状值。



从上面的案例当中，我们可以得出两个结论：

- 伪类用于**向某些已经存在的选择器添加特殊效果（当状态改变时）**
- 伪元素用于**将特殊效果添加到不存在的虚拟元素中（浏览器自动创建）**



**总结**

伪类的本质是类（class），它是通过在元素选择器上加⼊伪类改变元素状态，作用于添加了伪类的元素选择器本身，只不过限定了触发的状态条件。

伪元素的本质是元素（element），作用于该虚拟元素的内容本身。这些元素只对外部显示可见，而不存在于文档流中，因此称为“伪”元素。

> 冒号(`:`)用于`CSS3`伪类，双冒号(`::`)用于`CSS3`伪元素。`::before`就是以一个子元素的存在，定义在元素主体内容之前的一个伪元素。并不存在于`dom`之中，只存在于页面之中。
>
> **注意：** `:before `和 `:after` 这两个伪元素，是在`CSS2.1`里新出现的。起初，伪元素的前缀使用的是单冒号语法，但随着`Web`的进化，在`CSS3`的规范里，伪元素的语法被修改成使用双冒号，成为`::before`、`::after`。



### 对 requestAnimationframe 的理解

实现动画效果的方法比较多，Javascript 中可以通过定时器 setTimeout 来实现，CSS3 中可以使用 transition 和 animation 来实现，HTML5 中的 canvas 也可以实现。除此之外，HTML5 提供一个专门用于请求动画的API，那就是 requestAnimationFrame，顾名思义就是**请求动画帧**。

MDN对该方法的描述：

> window.requestAnimationFrame() 告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画。该方法需要传入一个回调函数作为参数，该回调函数会在浏览器下一次重绘之前执行。



### 🔴 对盒模型的理解

CSS3中的盒模型有以下两种：**标准盒子模型**、**IE盒子模型**

![border-box](https://raw.githubusercontent.com/aboutcroon/Notes/main/CSS/interview/assets/content-box.png)

![border-box](https://raw.githubusercontent.com/aboutcroon/Notes/main/CSS/interview/assets/border-box.png)

盒模型都是由四个部分组成的，分别是 margin、border、padding 和 content。

标准盒模型和 IE 盒模型的区别在于设置 width 和 height 时，所对应的范围不同：

- 标准盒模型的 width 和 height 属性的范围只包含了 content，
- IE 盒模型的 width 和 height 属性的范围包含了 border、padding 和 content。

可以通过修改元素的 box-sizing 属性来改变元素的盒模型：

- `box-sizing: content-box`表示标准盒模型（默认值）
- `box-sizing: border-box`表示IE盒模型（怪异盒模型）

> 若在页面最顶部声明了 DOCTYPE 类型，所有的浏览器都会把盒模型默认解释为标准盒模型：`<!DOCTYPE html>`。不然的话，IE 浏览器会将盒子模型解释为 IE 盒模型，FireFox 等会将其解释为标准盒模型
>
> 依然可以使用 css3 的 box-sizing 属性进行切换



### 为什么有时候通过 **translate** 来改变位置⽽不是通过定位

|           | 介绍                                  | 好处                                                         | 坏处                                                         |
| --------- | ------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| translate | translate 是 transform 属性的⼀个值。 | 1. 改变 transform 或 opacity 不会触发浏览器回流（reflow）或重绘（repaint），只会触发复合（compositions）。⽽改变绝对定位会触发回流，进⽽触发重绘和复合。 | translate 改变位置时，元素依然会占据其原始空间，绝对定位就不会发⽣这种情况。 |
|           |                                       | 2. transform 使浏览器为元素创建⼀个 GPU 图层，但改变绝对定位会使⽤到 CPU。 因此 translate 更⾼效，可以缩短平滑动画的绘制时间。 |                                                              |



### li 与 li 之间有看不见的空白间隔是什么原因引起的？如何解决？

浏览器会把 inline 内联元素间的空白字符（空格、换行、Tab等）渲染成一个空格。为了美观，通常是一个`<li>`放在一行，这导致`<li>`换行后产生换行字符，它变成一个空格，占用了一个字符的宽度。

**解决办法：**

- 将所有`<li>`写在同一行。不足：代码不美观。
- 消除`<ul>`的字符间隔 letter-spacing: -8px，但是这也设置了`<li>`内的字符间隔，因此需要将`<li>`内的字符间隔设为默认 letter-spacing: normal。



###  CSS3中有哪些新特性

- 新增各种CSS选择器 （:not(.input) 所有 class 不是“input”的节点）
- 圆角 （border-radius: 8px）
- 多列布局 （multi-column layout）
- 阴影和反射 （Shadoweflect）
- 文字特效 （text-shadow）
- 文字渲染 （Text-decoration）
- 线性渐变 （gradient）
- 旋转 （transform）
- 增加了旋转,缩放,定位,倾斜,动画,多背景



### 常见的图片格式及使用场景

**JPEG/JPG**

优点：

- 有损压缩
- 体积小、加载快

缺点：

- 处理矢量图形和 Logo 等线条感较强、颜色对比强烈的图像时，人为压缩导致的图片模糊会相当明显；
- 不支持透明度处理，透明图片需要 PNG 来呈现。



**PNG**

优点：

- 无损压缩
- 质量高，对线条的处理更加细腻
- 支持透明

缺点：

- 体积太大

应用场景：

- 小的 Logo
- 颜色简单且对比强烈的图片或背景等



**SVG**

优点：

- 文件体积更小
- 可压缩性更强
- 图片可无限放大而不失真❗️❗️
- 灵活性高，是文本文件，可以像写代码一样定义它

缺点：

- 渲染成本较高，对性能不好
- 学习成本较高，因为它是可编程的

应用场景：

- 用于适配 n 种分辨率，而不失真的场景



**Base64**

概念：

Base64 是一种用于传输 8Bit 字节码的编码方式，通过对图片进行 Base64 编码，我们可以直接将编码结果写入 HTML 或者写入 CSS，从而减少 HTTP 请求的次数。

Base64 是作为雪碧图的补充而存在的。

优点：

- 直接将编码结果写入 HTML 或者写入 CSS，从而减少 HTTP 请求的次数

缺点：

- 不适合传输大体积的图片，因为 Base64 编码后，图片大小会膨胀为原文件的 4/3
- 需要进行编码，是可编程的

应用场景：

- 传输非常小的图片时
- 图片无法以雪碧图的形式与其它小图结合时
- 图片的更新频率非常低（不需我们重复编码和修改文件内容，维护成本较低）

编码工具：webpack 的 [url-loader](https://github.com/webpack-contrib/url-loader)



**WebP**

概念：

于2010年被提出，是 Google 专为 Web 开发的一种旨在加快图片加载速度的图片格式，它支持有损压缩和无损压缩。

优点：

- 像 JPEG 一样对细节丰富的图片信手拈来
- 像 PNG 一样支持透明
- 像 GIF 一样可以显示动态图片

缺点：

- 兼容性太差
- 会增加服务器的负担



###  对 CSSSprites 的理解

CSSSprites（雪碧图），将一个页面涉及到的所有图片都包含到一张大图中去，然后利用CSS的 **background-image，background-repeat，background-position** 属性的组合进行背景定位。

**优点：**

- ❗️利用`CSS Sprites`能很好地减少网页的http请求，从而大大提高了页面的性能，这是`CSS Sprites`**最大的优点**；
- `CSS Sprites`能减少图片的字节，把3张图片合并成1张图片的字节总是小于这3张图片的字节总和。

**缺点：**

- 在图片合并时，要把多张图片有序的、合理的合并成一张图片，还要留好足够的空间，防止板块内出现不必要的背景。在宽屏及高分辨率下的自适应页面，如果背景不够宽，很容易出现背景断裂；
- `CSSSprites`在开发的时候相对来说有点麻烦，需要借助`photoshop`或其他工具来对每个背景单元测量其准确的位置。
- `CSS Sprites`在维护的时候比较麻烦，页面背景有少许改动时，就要改这张合并的图片，无需改的地方尽量不要动，这样避免改动更多的`CSS`，如果在原来的地方放不下，又只能（最好）往下加图片，这样图片的字节就增加了，还要改动`CSS`。



### 对 line-height 的理解及其赋值方式

**line-height的概念**

- line-height 指一行文本的高度，包含了字间距，实际上是**下一行基线到上一行基线距离**；
- 如果一个标签没有定义 height 属性，那么其最终表现的高度由 line-height 决定；
- 一个容器没有设置高度，那么撑开容器高度的是 line-height，而不是容器内的文本内容；
- 把 line-height 值设置为 height 一样大小的值可以**实现单行文字的垂直居中**；
- line-height 和 height 都能撑开一个高度；

**line-height 的赋值方式**

- 带单位：px 是固定值，而 em 会参考父元素 font-size 值计算自身的行高
- 纯数字：会把比例传递给后代。例如，父级行高为 1.5，子元素字体为 18px，则子元素行高为 1.5 * 18 = 27px
- 百分比：将计算后的值传递给后代



### 🔴 CSS 优化和提高性能的方法有哪些

**加载性能：**

1. **css 压缩**，将写好的 css 进行打包压缩，可以减小文件体积。
2. **css 单一样式**，当需要下边距和左边距的时候，很多时候会选择使用 margin:top 0 bottom 0；但 margin-bottom: bottom; margin-left: left; 执行效率会更高。
3. **减少使用 @import，建议使用 link**，因为后者在页面加载时一起加载，前者是等待页面加载完成之后再进行加载。

**选择器性能：**

> CSS 选择器是从右到左匹配的，选择器的最后面的部分为关键选择器（key selector）（即用来匹配目标元素的部分）。当使用后代选择器的时候，浏览器会遍历所有子元素来确定是否是指定的元素

1. **id 和 class 选择器不应该和多余的标签选择器写在一起**，过滤掉无关的规则（这样样式系统就不会浪费时间去匹配它们了）
2. **避免使用通配符**，如*{}计算次数惊人，我们应该只对需要用到的元素进行选择
3. **尽量少的去对标签进行选择，而是用 class 和 id**
4. **尽量少的去使用后代选择器**，后代选择器的开销是最高的，我们应尽量将选择器的深度降到最低（最高不要超过三层），更多的使用类来关联每一个标签元素
5. **可以通过继承而来的属性，则不重复指定规则**

**🔴 渲染性能（重点关注）（优化 CSSOM Tree 的构建）：**

1. 删除不必要的样式，对 CSS 进行最小化，压缩和缓存

   > - 去除空规则 {}
   > - 属性值为 0 时，不加单位
   > - 属性值为浮动小数 0.xx，可以省略小数点之前的 0

2. 减少渲染的阻塞

   > 浏览器会阻塞渲染，直到它解析完全部的样式，但不会阻塞渲染它认为不会使用的样式，例如打印样式表。
   >
   > 所以我们可以基于媒体查询将 CSS 分成多个文件，可以防止在下载未使用的 CSS 期间阻止渲染

3. 启用 CSS3 硬件加速，可以将被动画化的节点从主线程移到 GPU 上

   参考：https://developer.mozilla.org/zh-CN/docs/Learn/Performance/CSS

   > 以下是能将渲染移动到 composite 时期的一些属性和元素
   >
   > 属性：
   >
   > - transform
   > - opacity
   > - position: fixed
   > - will-change
   > - filter
   >
   > 元素：
   >
   > - `<video>`
   > - `<canvas>`
   > - `<iframe>`

4. `will-change` 属性

   > CSS [`will-change`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/will-change) 属性可以告诉浏览器元素的哪些属性需要修改，使浏览器能够在元素实际更改之前设置优化，通过在实际更改前执行耗时的工作以提升性能。
   >
   > ```css
   > will-change: opacity, transform;
   > ```

5. `font-display` 属性

   > 根据 [@font-face](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@font-face) 规则，[font-display](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@font-face/font-display) 属性定义了浏览器如何加载和显示字体文件，允许文本在字体加载或加载失败时显示回退字体。
   >
   > ```css
   > @font-face {
   >   font-family: someFont;
   >   src: url(/path/to/fonts/someFont.woff) format('woff');
   >   font-weight: 400;
   >   font-style: normal;
   >   font-display: fallback;
   > }
   > ```

6. `contain` 属性

   > CSS 的 `contain`属性允许我们指出哪些元素及其内容会作为其余部分独立于文档树的。这样浏览器就会针对 DOM 的有限区域而不是整个页面重新计算布局，样式，绘画，大小或它们的任意组合。

7. 慎重使用高性能属性：浮动、定位。会创建新的浮动层和定位层

8. 减少页面回流与重绘

**可维护性、健壮性：**

1. 规定 CSS 属性的书写顺序，格式（使用 stylelint 工具）
2. 使用推荐的 CSS 命名方式，合理组织 CSS 文件
4. 将具有相同属性的样式抽离出来，整合并通过 class 在页面中进行使用，提高 css 的可维护性。
5. 样式与内容分离：将 css 代码定义到外部 css 中。



### display: inline-block 什么时候会显示间隙？

- 有空格时会有间隙，可以删除空格解决；
- `margin`正值时，可以让`margin`使用负值解决；
- 使用`font-size`时，可通过设置`font-size:0`、`letter-spacing`、`word-spacing`解决；



### 🔴 单行、多行文本溢出省略号

单行

```css
{
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

多行

```css
{
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;  /* 作为弹性伸缩盒子模型显示 */
  -webkit-box-orient: vertical; /* 设置伸缩盒子的子元素排列方式：从上到下垂直排列 */	
  -webkit-line-clamp: 2; /* 显示的行数 */
}
```

注意：由于上面的三个属性都是 CSS3 的属性，所以要在前面加一个`-webkit-`来兼容一部分浏览器。



###  Sass、Less 是什么？为什么要使用他们？

他们都是 CSS 预处理器，是 CSS 上的一种抽象层。他们是一种特殊的语法/语言编译成的 CSS。 

**例如 Less 是一种动态样式语言，将 CSS 赋予了动态语言的特性**，如变量，继承，运算， 函数，Less 既可以在客户端上运行 (支持 IE 6+, Webkit, Firefox)，也可以在服务端运行 (借助 Node.js)。



**为什么要使用它们？**

- 结构清晰，便于扩展。 可以方便地屏蔽浏览器私有语法差异。封装对浏览器语法差异的重复处理， 减少无意义的机械劳动。
- 可以轻松实现多重继承。 完全兼容 CSS 代码，可以方便地应用到老项目中。Less 只是在 CSS 语法上做了扩展，所以老的 CSS 代码也可以与 LESS 代码一同编译。



### 对媒体查询的理解

媒体查询的组成

- ⼀个可选的媒体类型
- 零个或多个使⽤媒体功能限制了样式表范围的表达式，例如宽度、⾼度和颜⾊。

媒体查询，是在 CSS3 中的新特性，允许内容的呈现针对⼀个特定范围的输出设备⽽进⾏裁剪，⽽不必改变内容本身，适合 web ⽹⻚应对不同型号的设备⽽做出对应的响应适配。

媒体查询包含⼀个可选的媒体类型，和满⾜ CSS3 规范的条件下的零个或多个表达式，这些表达式描述了媒体特征，最终会被解析为 true 或 false。如果媒体查询中指定的媒体类型匹配展示⽂档所使⽤的设备类型，并且所有的表达式的值都是 true，那么该媒体查询的结果为true。那么媒体查询内的样式将会⽣效。

```javascript
<!-- link 元素中的 CSS 媒体查询 --> 
<link rel="stylesheet" media="(max-width: 800px)" href="example.css" />

<!-- 样式表中的 CSS 媒体查询 --> 
<style> 
@media (max-width: 600px) { 
  .facet_sidebar { 
    display: none; 
  } 
}
</style>
```

简单来说，使用 @media 查询，可以针对不同的媒体类型定义不同的样式。@media 可以针对不同的屏幕尺寸设置不同的样式，特别是**需要设置设计响应式的页面**，@media 是非常有用的。当重置浏览器大小的过程中，页面也会根据浏览器的宽度和高度重新渲染页面。



### 对 CSS 工程化的理解

CSS 工程化是为了解决以下问题：

1. **宏观设计**：CSS 代码如何组织、如何拆分、模块结构怎样设计？
2. **编码优化**：怎样写出更好的 CSS？
3. **构建**：如何处理我的 CSS，才能让它的打包结果最优？
4. **可维护性**：代码写完了，如何最小化它后续的变更成本？如何确保任何一个同事都能轻松接手？

以下三个方向都是时下比较流行的、普适性非常好的 CSS 工程化实践：

- 预处理器：Less、 Sass 等；
- 重要的工程化插件： Postcss；
- Webpack loader 等



### 如何判断元素是否到达可视区域

以元素为例：

el.scrollTop + el.clientHeight > el.scrollHeight - 1

以图片显示为例：

- `window.innerHeight` 是浏览器可视区的高度；
- `document.body.scrollTop || document.documentElement.scrollTop` 是浏览器滚动的过的距离；
- `imgs.offsetTop` 是元素顶部距离文档顶部的高度（包括滚动条的距离）；
- 内容达到显示区域的：`img.offsetTop < window.innerHeight + document.body.scrollTop;`

![scroll photo](https://raw.githubusercontent.com/aboutcroon/Notes/main/CSS/interview/assets/scroll%20photo.png)



### z-index属性在什么情况下会失效

通常 z-index 的使用是在有两个重叠的标签，在一定的情况下控制其中一个在另一个的上方或者下方出现。

z-index 值越大就越是在上层。z-index 只能在 position 属性值为 relative，absolute 或是fixed 的元素上有效

> position 的属性：默认为static、fixed、relative、absolute、sticky

z-index 属性在下列情况下会失效：

- 元素没有设置 position 属性为非 static 属性。

  解决：设置该元素的 position 属性为relative，absolute 或是 fixed 中的一种；

- 父元素 position 为 relative 时，子元素的 z-index 失效。

  解决：父元素 position 改为absolute 或 static；

- 元素在设置 z-index 的同时还设置了 float 浮动。

  解决：float 去除，改为 display：inline-block



### 🔴 CSS 样式隔离的手段

**概念**

CSS 没有本地作用域，一旦生效就会应用于全局，所以很容易出现冲突。

`SPA应用`流行了之后这个问题变得更加突出了，因为对于 SPA 应用来说所有页面的样式代码都会加载到同一个环境中，样式冲突的概率会大大加大。

带来以下弊端：

- 样式冲突

- 很难为选择器起名字

- 团队多人协作困难

- 无用的 CSS 样式堆积

  > 我们很难辨认出项目中哪些CSS样式代码是有用的哪些是无用的，这就导致了我们不敢轻易删除代码中可能是无用的样式。这样随着时间的推移，项目中无用的 CSS 样式就会堆积，**项目变得越来越重量级**，**开发成本越来越高**

- 不利于基于状态的样式定义

  > 对于 SPA 应用来说，特别是一些交互复杂的页面，页面的样式通常要根据组件的状态变化而发生变化，通常是通过不同的状态定义不同的`className名`，没有样式隔离的话类名的定义就很容易冲突



CSS 样式隔离就是为了解决这些问题。

抛开微前端的概念不谈，就算当前流行的前端框架也在解决 CSS 隔离的路上做出了相应的动作，其中就有 Vue，虽然 React 并没有对 CSS 隔离做处理，但是 React 关于这方面的插件也不少，开源的解决方案也特别多。

CSS 隔离是将 CSS 样式通过特殊方法安置在独立环境中，暂时避免和其他 CSS 污染。



**方案**

1. BEM（Block Element Modifier）

   一种命名方法论，例如以 `.block__element--modifier`或者说`block-name__element-name--modifier-name`形式命名，命名有含义，也就是`模块名 + 元素名 + 修饰器名`

2. CSS Modules

   顾名思义，CSS Modules 就是将 CSS 代码模块化，可以避免本模块样式被污染，并且可以很方便的复用 CSS 代码

   根据`CSS Modules`在Gihub上的[项目](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fcss-modules%2Fcss-modules)，它被解释为：所有的类名和动画名称默认都有各自的作用域的 CSS 文件

   所以`CSS Modules`既不是官方标准，也不是浏览器的特性，而是**在构建步骤（例如使用 Webpack，记住 css-loader）中对 CSS 类名和选择器`限定作用域`的一种方式**（类似于命名空间）

   依赖`webpack css-loader`，配置如下，现在 Webpack 已经默认开启 CSS Modules 功能了

   ```js
   {
     test: /.css$/,
     loader: "style-loader!css-loader?modules"
   }
   ```

   我们先看一个示例：

   将`CSS`文件`style.css`引入为`style`对象后，通过`style.title`的方式使用`title class`：

   ```jsx
   import style from './style.css';
   
   export default () => {
     return (
       <p className={style.title}>
         I am KaSong.
       </p>
     );
   };
   ```

   对应`style.css`：

   ```css
   .title {
     color: red;
   }
   ```

   打包工具会将`style.title`编译为`带哈希的字符串`

   ```jsx
   <h1 class="_3zyde4l1yATCOkgn-DBWEL">
     Hello World
   </h1>
   ```

   同时`style.css`也会编译：

   ```css
   ._3zyde4l1yATCOkgn-DBWEL {
     color: red;
   }
   ```

   这样，就产生了独一无二的`class`，解决了`CSS`模块化的问题

   使用了 CSS Modules 后，就相当于给每个 class 名外加加了一个 `:local`，以此来实现样式的局部化，如果你想切换到全局模式，使用对应的 `:global`。

   `:local` 与 `:global` 的区别是 CSS Modules 只会对 `:local` 块的 class 样式做 `localIdentName` 规则处理，`:global` 的样式编译后不变

   ```css
   .title {
     color: red;
   }
   
   :global(.title) {
     color: green;
   }
   ```

   可以看到，依旧使用CSS，但使用JS来管理样式依赖， 最大化地结合现有 CSS 生态和 JS 模块化能力，发布时依旧编译出单独的 JS 和 CSS

3. CSS in JS

   `CSS in JS`是2014年推出的一种**设计模式**，它的核心思想是`把CSS直接写到各自组件中`，也就是说`用JS去写CSS`，而不是单独的样式文件里

   > CSS-in-JS在`React社区`的热度是最高的，这是因为 React 本身不会管用户怎么去为组件定义样式的问题，而 Vue 和 Angular 都有属于框架自己的一套定义样式的方案

   上面的例子使用 React 改写如下

   ```js
   const style = {
     color: 'red',
     fontSize: '46px'
   };
   
   const clickHandler = () => alert('hi'); 
   
   ReactDOM.render(
     <h1 style={style} onclick={clickHandler}>
        Hello, world!
     </h1>,
     document.getElementById('example')
   );
   ```

   上面代码在一个文件里面，封装了**结构、样式和逻辑**，完全违背了 `关注点分离` 的原则

   但是，这有利于 `组件的隔离`。每个组件包含了所有需要用到的代码，不依赖外部，组件之间没有耦合，很方便复用。所以，随着 React 的走红和组件模式深入人心，这种 `关注点混合` 的新写法逐渐成为主流。

4. CSS 预处理器

   利用预处理器的嵌套语法，人为严格遵守嵌套首类名不一致，可以解决无作用域样式污染问题，需要借助编译工具的处理。

   我们常见的预处理器（[PostCSS](https://link.juejin.cn/?target=http%3A%2F%2Fpostcss.org%2F) 是后处理器）：

   - [Sass](https://link.juejin.cn/?target=https%3A%2F%2Fsass-lang.com%2F)
   - [LESS](https://link.juejin.cn/?target=https%3A%2F%2Flesscss.org%2F)
   - [Stylus](https://link.juejin.cn/?target=http%3A%2F%2Fstylus-lang.com%2F)

5. Shadow DOM

   Web components 的一个重要属性是封装——可以将标记结构、样式和行为隐藏起来，并与页面上的其他代码相隔离，保证不同的部分不会混在一起，可使代码更加干净、整洁。其中，Shadow DOM 接口是关键所在，它可以将一个隐藏的、独立的 DOM 附加到一个元素上。

   Shadow DOM 不是一个新事物，在过去的很长一段时间里，浏览器用它来封装一些元素的内部结构。以一个有着默认播放控制按钮的 [`<video>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/video) 元素为例。你所能看到的只是一个 `<video>` 标签，实际上，在它的 Shadow DOM 中，包含了一系列的按钮和其他控制器。Shadow DOM 标准允许你为你自己的元素（custom element）维护一组 Shadow DOM。

   也就是说这样样式可以只对一定范围内的 DOM 结构起作用，也就起到了样式隔离的效果。

6. Vue scoped

   当 `<style>` 标签有 `scoped` 属性时，它的 `CSS` 只作用于当前组件中的元素

   通过使用 `PostCSS` 来实现以下代码的转换：

   ```html
   <style scoped>
   .example {
     color: red;
   }
   </style>
   
   <template>
     <div class="example">hi</div>
   </template>
   ```

   转换结果：

   ```html
   <style>
   .example[data-v-f3f3eg9] {
     color: red;
   }
   </style>
   
   <template>
     <div class="example" data-v-f3f3eg9>hi</div>
   </template>
   ```

   使用 `scoped` 后，**父组件的样式将不会渗透到子组件中**

   不过一个子组件的根节点会同时受其父组件的 scoped CSS 和子组件的 scoped CSS 的影响。这样设计是为了让父组件可以从布局的角度出发，调整其子组件根元素的样式，父组件可以利用`深度作用选择器`来影响所有子组件样式

   可以使用 `>>>` 操作符：

   ```html
   <style scoped>
   .a >>> .b { /* ... */ }
   </style>
   ```

   上述代码将会编译成：

   ```css
   .a[data-v-f3f3eg9] .b { /* ... */ }
   ```

   有些像 `Sass` 之类的预处理器无法正确解析 `>>>`。这种情况下你可以使用 `/deep/` 或 `::v-deep` 操作符取而代之——两者都是 `>>>` 的别名，同样可以正常工作。



**总结**

|             | 概念                                             | 优点                                     | 缺点                                                         |
| ----------- | ------------------------------------------------ | ---------------------------------------- | ------------------------------------------------------------ |
| BEM         | 不同项目用不同的前缀，人为遵守命名规则来避免冲突 | 简单                                     | 需要人为约束，且有时命名太长                                 |
| CSS Modules | 通过编译生成不冲突的选择器类名                   | 可靠，易用                               | 需要频繁的写 style.xxx，只能在构建期使用，依赖打包工具如 css-loader |
| CSS in JS   | CSS 和 JS 编码在一起最终生成不冲突的选择器       | 彻底避免冲突，且和 JS 结合，写起来较灵活 | 是在在客户端动态生成 CSS 的，所以运行时开销较大，可读性较差，不能结合预处理器使用 |
| 预处理器    | 同 BEM，利用嵌套实现                             | 简单，提高开发效率                       | 需要借助相关的编译工具                                       |
| Shadow DOM  | 浏览器原生 CSS sandbox 支持的一种封装隔离        | 原生支持                                 | 只适用于特定场景，且有浏览器兼容问题                         |
| Vue scoped  | Vue 内置的样式隔离方案                           | 简单好用                                 | 只适用于 Vue 框架                                            |



### 🔴 CSS3 如何开启硬件加速（GPU 加速）

CSS3 硬件加速又叫做 GPU 加速，是利用 GPU 进行渲染，减少 CPU 操作的一种优化方案。

在 GPU 渲染的过程中，一些元素会因为符合了某些规则，而被提升为**独立的层**，一旦独立出来，就不会影响其它 DOM 的布局，所以我们可以利用这些规则，将经常变换的 DOM 主动提升到独立的层，那么在浏览器的一帧运行中，就可以减少 Layout 和 Paint 的时间了。



**创建独立图层**

哪些规则能让浏览器主动帮我们创建独立的层呢？

1. 3D 或者透视变换（perspective，transform） 的 CSS 属性。
2. 使用加速视频解码的 video 元素。
3. 拥有 3D（WebGL） 上下文或者加速 2D 上下文的 canvas 元素。
4. 混合插件（Flash)。
5. 对自己的 opacity 做 CSS 动画或使用一个动画 webkit 变换的元素。
6. 拥有加速 CSS 过滤器的元素。
7. 元素有一个包含复合层的后代节点（换句话说，就是一个元素拥有一个子元素，该子元素在自己的层里）。
8. 元素有一个兄弟元素在复合图层渲染，并且该兄弟元素的 z-index 较小，那这个元素也会被应用到复合图层。



**开启 GPU 加速**

CSS 中的以下几个属性能触发硬件加速：

1. transform（常用）
2. opacity
3. position: fixed
4. will-change
5. filter

如果有一些元素不需要用到上述属性，但是需要触发硬件加速效果，可以使用一些小技巧来诱导浏览器开启硬件加速。

```css
.element {
  -webkit-transform: translateZ(0);
  -moz-transform: translateZ(0);
  -ms-transform: translateZ(0);
  -o-transform: translateZ(0);
  transform: translateZ(0); 
  /**或者**/
  transform: rotateZ(360deg);
  transform: translate3d(0, 0, 0);
}
```



**要注意的问题**

- 过多地开启硬件加速可能会耗费较多的内存，因此什么时候开启硬件加速，给多少元素开启硬件加速，需要用测试结果说话。
- GPU 渲染会影响字体的抗锯齿效果。这是因为 GPU 和 CPU 具有不同的渲染机制，即使最终硬件加速停止了，文本还是会在动画期间显示得很模糊。



### 🔴 如何做 CSS 动画

**创建动画序列，需要使用 [`animation`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/animation) 属性或其子属性**，该属性允许配置动画时间、时长以及其他动画细节，但该属性不能配置动画的实际表现。

**动画的实际表现是由 [`@keyframes`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@keyframes) 规则实现**，具体情况参见[使用 keyframes 定义动画序列](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Animations/Using_CSS_animations#使用_keyframes_定义动画序列)小节部分。



## 2.页面布局

### 🔴 常见的 CSS 布局单位

常用的布局单位包括像素（`px`），百分比（`%`），`em`，`rem`，`vw/vh`。

**（1）像素**（`px`）

是页面布局的基础，一个像素表示终端（电脑、手机、平板等）屏幕所能显示的最小的区域，像素分为两种类型：CSS像素和物理像素

- **CSS像素**：为web开发者提供，在CSS中使用的一个抽象单位；
- **物理像素**：只与设备的硬件密度有关，任何设备的物理像素都是固定的。

**（2）百分比**（`%`）

当浏览器的宽度或者高度发生变化时，通过百分比单位可以使得浏览器中的组件的宽和高随着浏览器的变化而变化，从而实现响应式的效果。一般认为子元素的百分比相对于直接父元素。

**🔴（3）em和rem**

相对于 px 更具灵活性，它们都是相对长度单位，它们之间的区别：**em相对于父元素，rem相对于根元素。**

- **em：** 文本**相对长度单位**。其值并不固定，一般会继承父级元素的字体大小。任意浏览器的默认字体尺寸都是 16px，所有未经调整的浏览器都符合： 1em=16px。但如果父级设置了其他字体高度，那此时的 1em 就等于父级设置好的字体高度。
- **rem：** rem 是 CSS3 新增的一个单位，同样，它也是一个**相对长度单位**，不过相对的是 HTML 根元素。通过它既可以做到只修改根元素就成比例地调整所有字体大小，实现当屏幕分辨率变化时让元素也随之变化；又可以**避免字体大小逐层复合的连锁反应**。

**（4）vw/vh**

与视图窗口有关的单位

- vw：相对于视窗的宽度，视窗宽度是 100vw；
- vh：相对于视窗的高度，视窗高度是 100vh；
- vmin：vw 和 vh 中的较小值；
- vmax：vw 和 vh 中的较大值；

**vw/vh** 和百分比很类似，两者的区别：

- 百分比（`%`）：大部分相对于祖先元素，也有相对于自身的情况比如（border-radius、translate等)
- vw/vm：相对于视窗的尺寸



### 🔴 px、em、rem 的使用场景

- 对于只需要适配少部分移动设备，且分辨率对页面影响不大的，使用 px 即可 。
- 对于需要适配各种移动设备，使用 rem，例如需要适配 iPhone 和 iPad 等分辨率差别比较挺大的设备。



### 两栏布局

一般两栏布局指的是**左边一栏宽度固定，右边一栏宽度自适应**

两栏布局的具体实现：

- 利用flex布局，将左边元素设置为固定宽度 200px，将右边的元素设置为 flex:1。

```css
.outer {
  display: flex;
  height: 100px;
}
.left {
  width: 200px;
  background: tomato;
}
.right {
  flex: 1;
  background: gold;
}
```



### 三栏布局

三栏布局一般指的是页面中一共有三栏，**左右两栏宽度固定，中间自适应的布局**

三栏布局的具体实现：

> 参考：https://zhuanlan.zhihu.com/p/58355168

- 圣杯布局

  利用浮动和负边距来实现。**基本布局之后使用向左浮动，center 栏留出两边位置，然后使用相对定位将左右两栏通过`margin-left`定位到相应位置。**

  center 放在最前面是为了让中间一栏最先加载、渲染出来

  ```html
  <div class="outer">
    <div class="center">中间</div>
    <div class="left">左</div>
    <div class="right">右</div>
  </div>
  ```

  ```css
  /* 基本样式 */
  .outer {
    height: 100px;
  }
  
  .center, .left, .right {
    float: left;
    height: 100px;
  }
  
  .left {
    width: 100px;
    height: 100px;
    background: tomato;
    margin-left: -100%; /* 让 left 与 center 在同一行显示 */
  }
  
  .right {
    width: 200px;
    height: 100px;
    background: gold;
    margin-left: -200px; /* 让 right 与 center 在同一行显示 */
  }
  
  .center {
    width: 100%;
    height: 100px;
    background: lightgreen;
  }
  
  /* 圣杯样式 */
  .outer {
    padding-left: 100px;
    padding-right: 200px;
  }
  
  .left {
      position: relative; /* 一定要加上 relative */
      left: -100px;
  }
  .right {
      position: relative; /* 一定要加上 relative */
      right: -200px;
  }
  ```

- 双飞翼布局

  通过浮动和负边距来实现。双飞翼布局相对于圣杯布局来说，左右位置的保留是通过中间列的 margin 值来实现的，而不是通过父元素的 padding 来实现的。

  ```html
  <div class="outer">
    <div class="center">
      <div class="wrapper">中间</div>
    </div>
    <div class="left">左</div>
    <div class="right">右</div>
  </div>
  ```

  ```css
  /* 基本样式 */
  .outer {
    height: 100px;
  }
  
  .center, .left, .right {
    float: left;
    height: 100px;
  }
  
  .left {
    width: 100px;
    height: 100px;
    background: tomato;
    margin-left: -100%; /* 让 left 与 center 在同一行显示 */
  }
  
  .right {
    width: 200px;
    height: 100px;
    background: gold;
    margin-left: -200px; /* 让 right 与 center 在同一行显示 */
  }
  
  .center {
    width: 100%;
    height: 100px;
    background: lightgreen;
  }
  
  /* 双飞翼样式 */
  .wrapper {
    height: 100px;
    margin-left: 100px;
    margin-right: 200px;
  }
  ```

- flex 布局

  左右两栏设置固定大小，中间一栏设置为 flex:1
  
  ```css
  .outer {
    display: flex;
    height: 100px;
  }
  
  .left {
    width: 100px;
    background: tomato;
  }
  
  .right {
    width: 100px;
    background: gold;
  }
  
  .center {
    flex: 1;
    background: lightgreen;
  }
  ```



### 🔴 水平居中

inline 元素

```css
text-align: center;
```

block 元素

```css
margin: 0 auto;
```

absolute 元素

```css
left: 50%;
margin-left: -100px; /* 元素宽度的一半 */
```



### 🔴 垂直居中

inline 元素

```css
line-height: 21px; /* line-height 的值等于 height 的值 */
height: 21px;
```

absolute 元素

```css
top: 50%;
margin-top: -100px; /* 元素高度的一半 */
```

```css
transform: (-50%, -50%);
```

```css
top: 0;
left: 0;
right: 0;
bottom: 0;
margin: auto;
```



### 🔴 水平垂直居中

- 利用绝对定位，先将元素的左上角通过 top:50% 和 left:50% 定位到页面的中心，然后再通过 translate 来将元素的中心点调整到页面的中心点。

  > 该方法需要**考虑浏览器兼容问题。**

  ```css
  .parent {
    position: relative;
  }
  .child {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }
  ```

- 利用绝对定位，设置四个方向的值都为 0，并将 margin 设置为 auto，由于宽高固定，因此对应方向实现平分，可以实现水平和垂直方向上的居中。

  > 该方法适用于**盒子有宽高**的情况

  ```css
  .parent {
    position: relative;
  }
  .child {
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    margin: auto;
  }
  ```

- 使用 flex 布局，通过 align-items: center 和 justify-content: center 设置容器的垂直和水平方向上为居中对齐，然后它的子元素也可以实现垂直和水平的居中。

  > 该方法要**考虑兼容的问题**，该方法在移动端用的较多

  ```css
  .parent {
    display: flex;
    justify-content:center;
    align-items:center;
  }
  ```



### 🔴 回流（重排）与重绘

浏览器使用流式布局模型 (Flow Based Layout)。页面渲染过程：

- 解析 HTML 构建 `DOM Tree`

- 解析 CSS 构建 `CSSOM Tree`

- 浏览器将上面两者结合，构建渲染树（Render Tree），渲染树只包含渲染网页所需的节点。

- **（Layout）**根据 `Render Tree`，进行回流，计算出节点在页面上的大小和位置，得到节点的几何位置

- **（Paint）**根据 `Render Tree` 和回流得到的几何信息，结合所有节点的样式，重绘得到节点的绝对像素

- **（Composite）**最后将像素发给 GPU，GPU 会将多个图层合并为一个图层，将节点展示到页面上

  > 由于浏览器使用流式布局，对`Render Tree`的计算通常只需要遍历一次就可以完成。
  >
  > 但`table`及其内部元素除外，他们可能需要**多次计算**，通常要花 3 倍于同等元素的时间，这也是为什么要避免使用`table`布局的原因之一。



**流程图**

![render png](https://raw.githubusercontent.com/aboutcroon/Notes/main/CSS/interview/assets/render.png)



**概念**

当`Render Tree`中部分或全部元素的尺寸、结构、或某些属性发生改变时，浏览器会重新计算元素在页面上的大小和位置，并进行部分或全部文档的渲染，这个过程称为回流。

> 每个页面至少需要一次回流，就是在页面第一次加载的时候，这时候是一定会发生回流的，因为要构建`render tree`。

当页面中元素样式的改变并不影响它在文档流中的位置时（例如：`color`、`background-color`、`visibility`等），浏览器会将新样式赋予给元素并重新绘制它，这个过程称为重绘。



会导致回流的操作：

- 页面首次渲染

- 浏览器窗口大小发生变化

- 元素尺寸、位置发生变化

- 元素内容变化（文字数量、图片大小等）

- 元素字体大小变化

- 添加或删除可见的 DOM 元素

- 激活 CSS 伪类（例如：hover）

- 🔴 查询某些属性或调用某些方法

  属性：

  `clientWidth`、`clientHeight`、`clientTop`、`clientLeft`
  `offsetWidth`、`offsetHeight`、`offsetTop`、`offsetLeft`
  `scrollWidth`、`scrollHeight`、`scrollTop`、`scrollLeft`

  方法：

  `window.resize()`

  `scrollTo()`

  `scrollIntoView()`

  `scrollIntoViewIfNeeded()`

  `getComputedStyle()`

  `getBoundingClientRect()`



**性能影响**

**回流比重绘的代价要更高。**

有时即使仅仅回流一个单一的元素，它的父元素以及任何跟随它的元素也会产生回流。



现代浏览器会对频繁的回流或重绘操作进行优化：

浏览器会维护一个队列，把所有引起回流和重绘的操作放入队列中，如果队列中的任务数量或者时间间隔达到一个阈值时，浏览器就会将队列清空，进行一次批处理，这样可以把多次回流和重绘变成一次。

🔴 但当你访问以下属性或方法时，浏览器会立刻清空队列：

`clientWidth`、`clientHeight`、`clientTop`、`clientLeft`
`offsetWidth`、`offsetHeight`、`offsetTop`、`offsetLeft`
`scrollWidth`、`scrollHeight`、`scrollTop`、`scrollLeft`
`width`、`height`
`getComputedStyle()`
`getBoundingClientRect()`

以上都是一些**计算信息**，因为队列中可能会有影响到这些属性或方法返回值的操作，即使你希望获取的信息与队列中操作引发的改变无关，浏览器也会强行清空队列，**确保你拿到的值是最精确的**。



**如何避免**

CSS：

- 避免使用 table 布局
- 尽可能在 DOM 树的最末端改变 class
- 避免设置多层内联样式
- 对具有复杂动画的元素使用绝对定位，使它脱离文档流（position 为 absolute 或 fixed），否则会引起父元素及后续其他元素的频繁回流
- 非必要不使用 CSS 表达式（例如 calc()）

JS：

- 避免频繁操作样式，最好合并成一次操作
- 避免频繁操作 DOM，可以创建一个 documentFragment 对象去操作 DOM，最后再把它添加到文档中
- 先将元素设置为 display: none，然后对其进行多次 DOM 操作之后，再放回页面上
- 避免频繁读取会引发回流与重绘的属性，如果需要多次使用，就用一个变量缓存起来



### 🔴 对 BFC 的理解

**概念**

Box：CSS 布局的对象和基本单位，一个页面是由很多个 Box 组成的，这个 Box 就是我们所说的盒模型

Formatting context：块级上下文格式化，它是页面中的一块渲染区域，并且有一套渲染规则，它决定了其子元素将如何定位，以及和其他元素之间的关系和相互作用

Block Formatting Context（块级格式化上下文），是 Web 页面的可视化 CSS 渲染的一部分，是布局过程中生成块级盒子的区域，也是浮动元素与其他元素的交互限定区域



通俗来讲，BFC是一个独立的布局环境，可以理解为一个容器，在这个容器中按照一定规则进行物品摆放，并且不会影响其他环境中的物品。如果一个元素符合触发 BFC 的条件，则 BFC 中的元素布局将不受外界影响。



**创建 BFC 的条件**

- 根元素：body
- 元素设置浮动，float 除 none 以外的值
- 元素设置绝对定位，position 为 absolute 或 fixed
- display 设置为 inline-block, flex, table-cell, table-caption 等
- overflow 设置为 hidden, auto, scroll



**BFC 的特点**

- 垂直方向上，自上而下排列，和文档流的排列方式一致
- 在 BFC 中上下相邻的两个容器的 margin 会重叠
- 计算 BFC 的高度时，需要计算浮动元素的高度
- BFC 区域不会与浮动的容器发生重叠
- BFC 是独立的容器，容器内部元素不会影响外部元素
- 每个元素的左 margin 值和容器的左 border 相接触



**BFC 的作用**

- **解决 margin 重叠问题**：由于 BFC 是一个独立的区域，内部的元素和外部的元素互不影响，将两个元素变为两个 BFC，就解决了 margin 重叠的问题。

- **解决高度塌陷的问题**：在对子元素设置浮动后，父元素会发生高度塌陷，也就是父元素的高度变为0。解决这个问题，只需要把父元素变成一个 BFC。常用的办法是给父元素设置`overflow:hidden`。

- **创建自适应两栏布局**：可以用来创建自适应两栏布局：左边的宽度固定，右边的宽度自适应。

  ```css
  .left{
       width: 100px;
       height: 200px;
       background: red;
       float: left;
   }
   .right{
       height: 300px;
       background: blue;
       overflow: hidden;
   }
  
  <div class="left"></div>
  <div class="right"></div>
  ```



### 🔴 [flex: 1](https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex)

flex: 1 是一个 CSS 简写属性，是以下属性的简写：

- [`flex-grow`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex-grow)：增长系数，flex 容器中分配剩余空间的相对比例
- [`flex-shrink`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex-shrink)：收缩系数，当 flex 容器宽度不够时进行收缩的大小
- [`flex-basis`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex-basis)：**元素在主轴方向上的初始大小**



可以使用 1 个，2 个或 3 个值来指定 flex 属性

**单值语法**: 值必须为以下其中之一

- 🔴 **一个无单位数 ，它会被当作 flex-grow 的值，然后 flex-shrink 的值被假定为 1，flex-basis 的值被假定为 0**
- 一个有效的宽度值，它会被当作 flex-basis 的值
- 关键字 none，auto 或 initial

**双值语法**: 第一个值必须为一个无单位数，并且它会被当作 flex-grow 的值。第二个值必须为以下之一：

- 一个无单位数：它会被当作 flex-shrink 的值
- 一个有效的宽度值：它会被当作 flex-basis 的值

**三值语法：**

- 第一个值必须为一个无单位数，并且它会被当作 flex-grow 的值。
- 第二个值必须为一个无单位数，并且它会被当作 flex-shrink 的值。
- 第三个值必须为一个有效的宽度值，并且它会被当作 flex-basis 的值。



**关键字**

initial（默认值）：元素会根据自身宽高设置尺寸。它会缩短自身以适应 flex 容器，但不会伸长并吸收 flex 容器中的额外自由空间来适应 flex 容器。相当于将属性设置为`flex: 0 1 auto`。

auto：元素会根据自身的宽度与高度来确定尺寸，但是会伸长并吸收 flex 容器中额外的自由空间，也会缩短自身来适应 flex 容器。这相当于将属性设置为 "`flex: 1 1 auto`".

none：元素会根据自身宽高来设置尺寸。它是完全非弹性的：既不会缩短，也不会伸长来适应 flex 容器。相当于将属性设置为"`flex: 0 0 auto`"。



## 3. 定位与浮动

### 🔴 Position 的属性有哪些，作用是什么

| 属性值   | 概述                                                         |
| -------- | ------------------------------------------------------------ |
| static   | 默认值，没有定位，元素出现在正常的文档流中，会忽略 top, bottom, left, right 或者 z-index 声明，块级元素从上往下纵向排布，⾏级元素从左往右横向排列。 |
| absolute | 生成绝对定位的元素，相对于 static 定位之外（absolute/relative/fixed）的一个父元素进行定位（如果没找到就以浏览器边界定位）。元素的位置通过 left、top、right、bottom 属性进行规定。 |
| relative | 生成相对定位的元素，相对于其原来的位置进行定位。元素的位置通过 left、top、right、bottom 属性进行规定。 |
| fixed    | 生成绝对定位的元素，相对于屏幕视⼝（viewport）的位置来进行定位，元素的位置在屏幕滚动时不会改变，⽐如回到顶部的按钮⼀般都是⽤此定位⽅式。通过 left、top、right、bottom 属性指定元素位置。 |
| inherit  | 规定从父元素继承 position 属性的值                           |



## 4. 场景应用

### 🔴 画三角形

CSS绘制三角形主要用到的是 border 属性

平时在给盒子设置边框时，往往都设置很窄，就可能误以为边框是由矩形组成的。**实际上，border 属性是由三角形组成的**，下面看一个例子：

```css
div {
  width: 0;
  height: 0;
  border: 100px solid;
  border-color: orange blue red green;
}
```

将元素的长宽都设为 0，显示出来的效果是这样的：

![triangle](https://raw.githubusercontent.com/edwineo/Notes/main/CSS/interview/assets/triangle.png)

所以我们可以根据 border 这个特性来绘制三角形

```css
div {
  width: 0;
  height: 0;
  border-top: 50px solid red;
  border-right: 50px solid transparent;
  border-left: 50px solid transparent;
}
```

![triangle-bottom](https://raw.githubusercontent.com/edwineo/Notes/main/CSS/interview/assets/triangle-bottom.png)

```css
div {
  width: 0;
  height: 0;
  border-bottom: 50px solid red;
  border-right: 50px solid transparent;
  border-left: 50px solid transparent;
}
```

![triangle-top](https://raw.githubusercontent.com/edwineo/Notes/main/CSS/interview/assets/triangle-top.png)

```css
div {
  width: 0;
  height: 0;
  border-right: 50px solid red;
  border-top: 50px solid transparent;
  border-bottom: 50px solid transparent;
}
```

![triangle-left](https://raw.githubusercontent.com/edwineo/Notes/main/CSS/interview/assets/triangle-left.png)



```css
div {
  width: 0;
  height: 0;
  border-left: 50px solid red;
  border-top: 50px solid transparent;
  border-bottom: 50px solid transparent;
}
```

![triangle-right](https://raw.githubusercontent.com/edwineo/Notes/main/CSS/interview/assets/triangle-right.png)



总体的原则就是通过上下左右的 border 来控制三角形的方向，**用 border 的宽度比来控制三角形的角度**。



### 🔴 画 0.5px 的线

> 参考：https://juejin.cn/post/6844903582370643975

**采用 transform: scale() 的方式**，该方法用来定义元素的 2D 缩放转换

```css
.before {
  width: 10px;
  height: 1px;
  background: black;
}

.after {
  width: 10px;
  height: 1px;
  background: black;
  transform: scaleY(0.5);
  transform-origin: 50% 100%; /* 距离x轴左侧偏移50%，距离y轴顶部偏移100% */
}
```

通过 `transform: scale` 会导致 Chrome 上的线条变虚，而粗细几乎没有变化。我们可以指定变换的原点，加上这个 [transform-origin](https://developer.mozilla.org/zh-CN/docs/Web/CSS/transform-origin) 就不会有虚化



**采用 box-shadow 的方式**

```css
.before {
  width: 10px;
  height: 1px;
  background: black;
}

.after {
  width: 10px;
  height: 1px;
  background: none;
  box-shadow: 0 0.5px 0 #000; /* x偏移量0，y偏移量0.5px，阴影模糊半径0，阴影颜色#000 */
}
```

设置 [box-shadow](https://developer.mozilla.org/zh-CN/docs/Web/CSS/box-shadow) 的第二个参数为 0.5px，表示阴影垂直方向的偏移为 0.5px



**采用 meta viewport 的方式（移动端）**

```css
<meta name="viewport" content="width=device-width, initial-scale=0.5, minimum-scale=0.5, maximum-scale=0.5"/>
```

这样就能缩放到原来的 0.5 倍，如果是 1px 那么就会变成 0.5 px。viewport 只针对于移动端，只在移动端上才能看到效果

