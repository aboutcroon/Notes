## 1.CSS基础

### CSS选择器及其优先级

| **选择器**     | **格式**      | **优先级权重** |
| -------------- | ------------- | -------------- |
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

