# 1. 命名风格

## 1.1 命名前缀

使用带前缀的命名空间，防止命名冲突。

```css
.adw-help {} /* AdWords */
#maia-note {} /* Maia */
```

## 1.2 id 和 class 命名分隔符

选择器中使用连字符，以提高可读性。

```css
/* bad  “demo” 和 “image” 之间没有分隔符 */
.demoimage {}
 
/* bad 使用下划线 */
.error_status {}
 
/* good */
#video-id {}
.ads-sample {}
```



# 2. 值与单位

## 2.1 单位

值为 0 时不用添加单位。

```css
margin: 0;
padding: 0;
```


值在 -1 和 1 之间时，不需要加 0。

```css
font-size: .8em;
```

## 2.2 颜色值

颜色统一用 16 进制表示，重复的字母省略。除了是带有 `alpha` 的颜色信息可以使用 `rgba()`，通常情况下都不允许使用 `rgba()`。使用 `rgba()` 时每个逗号后必须保留一个空格。

```css
/* bad */
color: #eebbcc;
 
/* good */
color: #ebc;
 
/* good */
box-shadow: 0 0 2px rgba(0, 128, 0, .3);
```



颜色值不能使用命名颜色，颜色值中的英文字符应统一采用小写。

```css
/* bad */
.success {
  color: lightgreen;
}
 
/* good */
.success {
  color: #90ee90;
}
 
/* bad */
.success {
  color: #90ee90;
  background-color: #ACA
}
 
/* good */
.success {
  color: #90ee90;
  background-color: #aca
}
```



# 3. 选择器及属性

## 3.1 选择器

除非需要，否则不要在 id 或 class 前加元素名进行限定。

```css
/* bad */
ul#example {}
div.error {}

/* good */
#example {}
.error {}
```

## 3.2 属性简写

尽量使用 CSS 中可以简写的属性 (如 font)，可以提高编码效率以及代码可读性。

```css
/* bad */
border-top-style: none;
font-family: palatino, georgia, serif;
font-size: 100%;
line-height: 1.6;
padding-bottom: 2em;
padding-left: 1em;
padding-right: 1em;
padding-top: 0;

/* good */
border-top: 0;
font: 100%/1.6 palatino, georgia, serif;
padding: 0 1em 2em;
```



# 4. 书写顺序

## 4.1 书写顺序

按照以下顺序书写属性（带有浏览器前缀的属性与其相应的属性写一起），以减少不必要的页面重绘。

```css
div {
  /* 定位属性 */
  display;
  visibility;
  float;
  clear;
  
  position;
  top;
  right;
  botttom;
  left;
  z-index;
  overflow;

  /* 自身属性 */
  width;
  min-width;
  max-width;
  height;
  min-height;
  max-height;

  margin;
  margin-top;
  margin-right;
  margin-bottom;
  margin-left;

  padding;
  padding-top;
  padding-right;
  padding-bottom;
  padding-left;
  
  border-width;
  border-top-width;
  border-right-width;
  border-bottom-width;
  border-left-width;

  border-style;
  border-top-style;
  border-right-style;
  border-bottom-style;
  border-left-style;

  border-color;
  border-top-color;
  border-right-color;
  border-bottom-color;
  border-left-color;

  outline;

  list-style;

  table-layout;
  caption-side;
  border-collapse;
  border-spacing;
  empty-cells;
  
  /* 文字样式, 文本属性 */
  font;
  font-family;
  font-size;
  line-height;
  font-weight;
  text-align;
  text-indent;
  text-transform;
  text-decoration;
  letter-spacing;
  word-spacing;
  white-space;
  vertical-align;
  color;

  background;
  background-color;
  background-image;
  background-repeat;
  background-position;

  /* 其他属性 */
  opacity;

  cursor;

  content;
  quotes;
}
```



# 5. 通用

## 5.1 选择器的嵌套层级

选择器的嵌套层级应不大于 3 级，位置靠后的限定条件应尽可能精确。

```css
/* bad */
.page .header .login #username input {}
.comment div * {}

/* good */
#username input {}
.comment .avatar {}
```

## 5.2 引号

属性选择器和属性值用单引号，URI 的值不需要引号。

```css
/* bad */
@import url("//www.google.com/css/maia.css");

html {
  font-family: "open sans", arial, sans-serif;
}

/* good */
@import url(//www.google.com/css/maia.css);

html {
  font-family: 'open sans', arial, sans-serif;
}
```

## 5.3 样式块风格

在选择器和 {} 之间用空格隔开。

```css
/* bad 没有空格 */
#video{
  margin-top: 1em;
}

/* bad 没有必要的换行 */
#video
{
  margin-top: 1em;
}

/* good */
#video {
  margin-top: 1em;
}
```

## 5.4 属性名书写风格

属性名和值之间都应有一个空格。

```css
/* bad */
h3 {
  font-weight:bold;
}

/* good */
h3 {
  font-weight: bold;
}
```

## 5.5 选择器分隔

每个选择器都另起一行。

```css
/* bad */
a:focus, a:active {
  position: relative; top: 1px;
}

/* good */
h1,
h2,
h3 {
  font-weight: normal;
  line-height: 1.2;
}
```



选择器之间都用空行隔开。

```css
html {
  background: #fff;
}

body {
  margin: auto;
  width: 50%;
}
```

## 5.6 分段注释

用注释把 CSS 分成各个部分。

```css
/* Header */

#adw-header {}

/* Footer */

#adw-footer {}

/* Gallery */

.adw-gallery {}
```

## 5.7 2D 位置

2D 位置初始值为 `0% 0%`，但在只有一个方向的值时，另一个方向的值会被解析为 center。为避免理解上的困扰，应同时给出两个方向的值。

```css
/* bad */
body {
  background-position: top; /* 水平50% 垂直0% */
}

/* good */
body {
  background-position: center top; /* 水平50% 垂直0% */
}
```

##  5.8 !important

尽量不使用 `!important` 声明。当需要强制指定样式且不允许任何场景覆盖时，才可通过标签内联和 `!important` 定义样式。

必须注意的是，仅在设计上 `确实不允许任何其它场景覆盖样式` 时，才使用内联的 `!important` 样式。

## 5.9 z-index

将 `z-index` 进行分层，对文档流外绝对定位元素的视觉层级关系进行管理。

同层的多个元素，如多个由用户输入触发的 Dialog，在该层级内使用相同的 `z-index` 或递增 `z-index`。

每层使用100个 `z-index` 来容纳足够的元素，如果每层元素较多，可以调整这个数值。

在可控环境下，期望显示在最上层的元素，`z-index` 指定为 `999999`。



# 6. 字体

## 6.1 字体顺序

font-family 按「西文字体在前、中文字体在后」、「效果佳 (质量高/更能满足需求) 的字体在前、效果一般的字体在后」的顺序编写，最后必须指定一个通用字体族( `serif` / `sans-serif` )。

```css
h1 {
  font-family: "Helvetica Neue", Arial, "Hiragino Sans GB", "WenQuanYi Micro Hei", "Microsoft YaHei", sans-serif;
}
```

## 6.2 字体大小写

在同一个项目中的 font-family，应都采用大写。

```css
/* bad */
body {
  font-family: arial, sans-serif;
}

h1 {
  font-family: Arial, "Microsoft YaHei", sans-serif;
}

/* good */
body {
  font-family: Arial, sans-serif;
}

h1 {
  font-family: Arial, "Microsoft YaHei", sans-serif;
}
```

## 6.3 font-weight

font-weight 属性必须使用数值方式描述。400 等同于 normal，700 等同于 bold。

```css
/* bad */
h1 {
  font-weight: bold;
}

/* good */
h1 {
  font-weight: 700;
}
```



# 7. 变换与动画

## 7.1 指定 transition-property

使用 transition 时应指定 transition-property。

```css
/* bad */
.box {
  transition: all 1s;
}

/* good */
.box {
  transition: color 1s, border-color 1s;
}
```

## 7.2 尽可能在浏览器能高效实现的属性上添加过渡和动画

在可能的情况下应选择这样四种变换：

- transform: translate(npx, npx);
- transform: scale(n);
- transform: rotate(ndeg);
- opacity: 0.1;

典型的，可以使用 translate 来代替 left 作为动画属性。

```css
/* bad */
.box {
  left: 0;
  transition: left 1s;
}
.box:hover {
  left: 20px; /* move right for 20px */
}

/* good */
.box {
  transition: transform 1s;
}
.box:hover {
  transform: translate(20px); /* move right for 20px */
}
```



# 8. 响应式

## 8.1 效果更佳的样式

尽可能给出在高分辨率设备 (Retina) 下效果更佳的样式。

## 8.2 Media Query

Media Query 不得单独编排，必须与相关的规则一起定义。

```css
/* bad */
/* header styles */
/* main styles */
/* footer styles */

@media (...) {
    /* header styles */
    /* main styles */
    /* footer styles */
}

/* good */
/* header styles */
@media (...) {
    /* header styles */
}

/* main styles */
@media (...) {
    /* main styles */
}

/* footer styles */
@media (...) {
    /* footer styles */
}
```



# 9. 兼容性

## 9.1 属性前缀

带私有前缀的属性由长到短排列，按冒号位置对齐，标准属性放在最后。

```css
.box {
  -webkit-box-sizing: border-box;
     -moz-box-sizing: border-box;
          box-sizing: border-box;
}
```



# 10. 文件命名

## 10.1 文件命名

- 全局样式表：global.css

- 重置默认样式：reset.css

- 布局样式：layout.css

- 文字样式：font.css



# 11. id 及 class 命名

## 11.1 id 及 class 命名

统一采用 `kebab-case` 命名。

- 无嵌套结构时

第一个字符表示一个块，第二个字符表示块级下面的元素。

示例：

```html
<div class="article">
  <button class="article-btn"></button>
</div>
 
<div class="head">
  <h1 class="head-title">title</h1>
</div>
```

- 有嵌套结构时

直接根据层级关系命名，无需用 - 连接。

示例：

```html
<div class="article">
  <button class="btn"></button>
</div>
 
<div class="head">
  <h1 class="title">title</h1>
</div>
```



# 12. 常用命名

使用有含义的 id 和 class 名称。id 和 class 应该尽量简短，但同时要容易理解。

## 12.1 页面结构

| 功能     | id / class      |
| :------- | :-------------- |
| 包裹容器 | container / box |
| 页头     | header          |
| 页面主体 | main            |
| 页尾     | footer          |
| 导航     | nav             |
| 侧边栏   | sidebar         |

## 12.2 状态

| 功能                     | id / class |
| :----------------------- | :--------- |
| 成功                     | success    |
| 危险                     | danger     |
| 警告                     | warning    |
| 错误                     | error      |
| 亮色                     | light      |
| 暗色                     | dark       |
| 禁用                     | disabled   |
| 激活                     | active     |
| 选中（单选框 / 复选框）  | checked    |
| 默认选中项（下拉选择框） | selected   |
| 加载                     | loading    |

## 12.3 尺寸

| 功能 | id / class |
| :--- | :--------- |
| 大   | big        |
| 中   | middle     |
| 小   | small      |
| 更大 | bigger     |
| 更小 | smaller    |

## 12.4 位置

| 功能     | id / class |
| :------- | :--------- |
| 第一个   | first      |
| 第二个   | second     |
| 第三个   | third      |
| 最后一个 | last       |
| 上一个   | prev       |
| 下一个   | next       |
| 左       | left       |
| 右       | right      |
| 中       | center     |

## 12.5 组件

| 功能            | id / class |
| :-------------- | :--------- |
| 卡片            | card       |
| 列表            | list       |
| 图片            | img        |
| 轮播图          | banner     |
| 轮滑            | slide      |
| 使用swiper时    | swiper     |
| 菜单栏          | menu       |
| 分页            | pagination |
| 标题            | title      |
| 内容            | content    |
| 日期            | date       |
| 时间            | time       |
| 头像            | avatar     |
| 标志            | logo       |
| 下划线 / 修饰线 | line       |
| 登录条          | login-bar  |
| 搜索框          | search     |
| 功能区          | box        |
| 表单            | form       |
| 表格            | table      |
| 弹窗            | dialog     |
| 抽屉式弹窗      | drawer     |


