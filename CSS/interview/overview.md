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

