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
| 伪元素选择器   | li:after      | 1              |
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

   list-style：列表风格，包括list-style-type、list-style-image等

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



### transition 和 animation 的区别

**transition是过度属性**，强调过度，它的实现需要触发一个事件（比如鼠标移动上去，焦点，点击等）才执行动画。它类似于 flash 的补间动画，设置一个开始关键帧，一个结束关键帧。

**animation是动画属性**，它的实现不需要触发事件，设定好时间之后可以自己执行，且可以循环一个动画。它也类似于 flash 的补间动画，但是它可以设置多个关键帧（用@keyframe定义）完成动画。



### display:none 与 visibility:hidden 的区别

这两个属性都是让元素隐藏，不可见。**两者区别如下：**

**在渲染树中**

- `display:none`会让元素完全从渲染树中消失，渲染时不会占据任何空间；
- `visibility:hidden`不会让元素从渲染树中消失，渲染的元素还会占据相应的空间，只是内容不可见。

**是否是继承属性**

- `display:none`是非继承属性，子孙节点会随着父节点从渲染树消失，通过修改子孙节点的属性也无法显示；
- `visibility:hidden`是继承属性，子孙节点消失是由于继承了`hidden`，通过设置`visibility:visible`可以让子孙节点显示；

**回流与重绘**

修改常规文档流中元素的 `display` 通常会造成文档的重排但是修改`visibility`属性只会造成本元素的重绘



### 伪元素和伪类的区别和作用

**伪元素**

在内容元素的前后插入额外的元素或样式，但是这些元素实际上并不在文档中生成。它们只在外部显示可见，但不会在文档的源代码中找到它们，因此，称为“伪”元素。例如：

```css
p::before {content:"第一章：";}
p::after {content:"Hot!";}
p::first-line {background:red;}
p::first-letter {font-size:30px;}
```

**伪类**

将特殊的效果添加到特定选择器上。它是已有元素上添加类别的，不会产生新的元素。例如：

```css
a:hover {color: #FF00FF}
p:first-child {color: red}
```

**总结：** 伪类是通过在元素选择器上加⼊伪类改变元素状态，⽽伪元素通过对元素的操作进⾏对元素的改变。



### 对 requestAnimationframe 的理解

实现动画效果的方法比较多，Javascript 中可以通过定时器 setTimeout 来实现，CSS3 中可以使用 transition 和 animation 来实现，HTML5 中的 canvas 也可以实现。除此之外，HTML5 提供一个专门用于请求动画的API，那就是 requestAnimationFrame，顾名思义就是**请求动画帧**。

MDN对该方法的描述：

> window.requestAnimationFrame() 告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画。该方法需要传入一个回调函数作为参数，该回调函数会在浏览器下一次重绘之前执行。



### 对盒模型的理解

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



### CSS 优化和提高性能的方法有哪些

**加载性能：**

1. css压缩：将写好的css进行打包压缩，可以减小文件体积。
2. css单一样式：当需要下边距和左边距的时候，很多时候会选择使用 margin:top 0 bottom 0；但 margin-bottom: bottom; margin-left: left; 执行效率会更高。
3. 减少使用 @import，建议使用 link，因为**后者在页面加载时一起加载，前者是等待页面加载完成之后再进行加载**。

**选择器性能：**

1. 关键选择器（key selector）。选择器的最后面的部分为关键选择器（即用来匹配目标元素的部分）。CSS选择符是从右到左进行匹配的。当使用后代选择器的时候，浏览器会遍历所有子元素来确定是否是指定的元素等等；
2. 如果规则拥有 ID 选择器作为其关键选择器，则不要为规则增加标签。过滤掉无关的规则（这样样式系统就不会浪费时间去匹配它们了）。
3. 避免使用通配规则，如*{}计算次数惊人，只对需要用到的元素进行选择。
4. 尽量少的去对标签进行选择，而是用 class 和 id。
5. 尽量少的去使用后代选择器，降低选择器的权重值。后代选择器的开销是最高的，尽量将选择器的深度降到最低，最高不要超过三层，更多的使用类来关联每一个标签元素。
6. 了解哪些属性是可以通过继承而来的，然后避免对这些属性重复指定规则。

**渲染性能：**

1. 慎重使用高性能属性：浮动、定位。
2. 尽量减少页面重排、重绘。
3. 去除空规则: {}。空规则的产生原因一般来说是为了预留样式。去除这些空规则无疑能减少 css 文档体积。
4. 属性值为 0 时，不加单位。
5. 属性值为浮动小数 0.xx，可以省略小数点之前的 0。
6. 标准化各种浏览器前缀：带浏览器前缀的在前。标准属性在后。
7. 不使用 @import 前缀，它会影响 css 的加载速度。
8. 选择器优化嵌套，尽量避免层级过深。
9. css 雪碧图，同一页面相近部分的小图标，方便使用，减少页面的请求次数，但是同时图片本身会变大，使用时，优劣考虑清楚，再使用。
10. 正确使用 display 的属性，**由于 display 的作用，某些样式组合会无效**，徒增样式体积的同时也影响解析性能。
11. 不滥用web字体。对于中文网站来说WebFonts可能很陌生，国外却很流行。web fonts通常体积庞大，而且一些浏览器在下载web fonts时会阻塞页面渲染损伤性能。

**可维护性、健壮性：**

1. 将具有相同属性的样式抽离出来，整合并通过 class 在页面中进行使用，提高 css 的可维护性。
2. 样式与内容分离：将 css 代码定义到外部 css 中。



### display: inline-block 什么时候会显示间隙？

- 有空格时会有间隙，可以删除空格解决；
- `margin`正值时，可以让`margin`使用负值解决；
- 使用`font-size`时，可通过设置`font-size:0`、`letter-spacing`、`word-spacing`解决；



### 单行、多行文本溢出隐藏

- 单行文本溢出

```css
overflow: hidden;            // 溢出隐藏
text-overflow: ellipsis;      // 溢出用省略号显示
white-space: nowrap;         // 规定段落中的文本不进行换行
```

- 多行文本溢出

```css
overflow: hidden;            // 溢出隐藏
text-overflow: ellipsis;     // 溢出用省略号显示
display: -webkit-box;         // 作为弹性伸缩盒子模型显示。
-webkit-box-orient: vertical; // 设置伸缩盒子的子元素排列方式：从上到下垂直排列
-webkit-line-clamp: 3;        // 显示的行数
```

注意：由于上面的三个属性都是 CSS3 的属性，没有浏览器可以兼容，所以要在前面加一个`-webkit-`来兼容一部分浏览器。



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





## 1. 页面布局

### 三栏布局

题目：假设高度已知，请写出三栏布局，其中左栏、右栏宽度各为 300px，中间自适应。



absolute 布局方式：

```html
<body>
  <section class="absolute">
    <div class="wrapper">
      <div class="left">left</div>
      <div class="center">center</div>
      <div class="right">right</div>
    </div>
  
    <style type="text/css">
    .absolute .wrapper{
      width: 100%;
      height: 100px;
      position: relative;
    }
    .absolute .wrapper div {
      height: 100px;
    }
    .absolute .left{
      position: absolute;
      left: 0;
      width: 300px;
      background: red;
    }
    .absolute .center{
      position: absolute;
      left: 300px;
      right: 300px;
      background: yellow;
    }
    .absolute .right{
      position: absolute;
      right: 0;
      width: 300px;
      background: blue;
    }
    </style>
  </section>
</body>
```



flex 布局方式：

```html
<body>
  <section class="flex">
    <div class="wrapper">
      <div class="left">left</div>
      <div class="center">center</div>
      <div class="right">right</div>
    </div>

   <style type="text/css">
    .flex {
      margin-top: 100px;
    }
    .flex .wrapper{
      width: 100%;
      height: 100px;
      display: flex;
    }
    .flex .left{
      width: 300px;
      background: red;
    }
    .flex .center{
      flex: 1;
      background: yellow;
    }
    .flex .right{
      width: 300px;
      background: blue;
    }
    </style>
  </section>
</body>
```



### 水平垂直居中

absolute + transform

> 兼容性依赖 translate，但不需要知道子元素宽高

```html
<style type="text/css">
  .out{
    position: relative;
    width: 300px;
    height: 300px;
    background: red;
  }

  .inner{
    position: absolute;
    background: yellow;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }
</style>
```



absolute + margin: auto

> 这种方法兼容性很好，缺点是需要知道子元素的宽高

```html
<style type="text/css">
  .out {
    position: relative;
    width: 300px;
    height: 300px;
    background: red;
  }

  .inner {
    position: absolute;
    width: 100px;
    height: 100px;
    background: yellow;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    margin: auto;
  }
</style>
```



absolute + calc（absolute + 负 margin）

> 这种方法的兼容性依赖于 calc，且也需要知道宽高。与 absolute + 负 margin 的方案类似

```html
<style type="text/css">
  .out{
    position: relative;
    width: 300px;
    height: 300px;
    background: red;
  }

  .inner{
    position: absolute;
    width: 100px;
    height: 100px;
    background: yellow;
    left: calc(50% - 50px);
    top: calc(50% - 50px);
  }
</style>
```





flex

> flex 实现起来比较简单，三行代码即可搞定，可通过父元素指定子元素的对齐方式。

```html
<style type="text/css">
  .out{
    display: flex;
    justify-content: center;
    align-items: center;
    width: 300px;
    height: 300px;
    background: red;
  }

  .inner{
    background: yellow;
    width: 100px;
    height: 100px;
  }
</style>
```



## 2. CSS盒模型

面试问题：

- 基本概念：标准模型 + IE模型
- 标准模型 和 IE模型的区别
- CSS如何设置这两种模型
- JS如何设置和获取盒模型对应的宽和高
- 实例题（根据盒模型解释边距重叠）
- BFC（边距重叠解决方案）



### 1. 标准模型 与 IE模型的区别

标准模型与 IE 模型的区别在于宽高的计算方式不同。

标准模型：content 的宽高

IE模型：content + padding + border 的宽高



### 2. 如何设置这两种模型

```css
//设置标准模型
box-sizing: content-box;

//设置IE模型
box-sizing: border-box;
```

> box-sizing 的默认值是 content-box，即默认标准模型。

### 3. JS 如何设置盒模型的宽高

假设已经获得的节点为 `dom`

```js
//只能获取内联样式设置的宽高
dom.style.width/height

//获取渲染后即时运行的宽高，值是准确的。但只支持 IE
dom.currentStyle.width/height

//获取渲染后即时运行的宽高，值是准确的。兼容性更好
window.getComputedStyle(dom).width/height;

//获取渲染后即时运行的宽高，值是准确的。兼容性也很好，一般用来获取元素的绝对位置，getBoundingClientRect()会得到4个值：left, top, width, height
dom.getBoundingClientRect().width/height;
```



## 3. BFC

Block Formatting Context（块级格式化上下文）



## 4. flex

### flex:1 是由哪些属性组成，分别代表什么意思？

flex-grow: 增长系数，flex 容器中分配剩余空间的相对比例

flex-shrink: 收缩系数，当 flex 容器宽度不够时进行收缩的大小

flex-basis: 元素在主轴方向上的初始大小



初始: 0 1 auto

flex-1: 1 1 0

flex-auto: 1 1 auto



参考：https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex

