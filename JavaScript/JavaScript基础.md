# String

## 转换成 String

### 两种转换方法

有两种方式把一个值转换为字符串。

#### 1. toString()

首先是使用几乎所有值都有的 `toString() 方法`。这个方法唯一的用途就是返回当前值的字符串等价物。比如：

```js
let age = 11; 
let ageAsString = age.toString(); // 字符串"11" 

let found = true; 
let foundAsString = found.toString(); // 字符串"true"
```

toString()方法可见于数值、布尔值、对象和字符串值。（没错，字符串值也有 toString()方法，该方法只是简单地返回自身的一个副本。）`null 和 undefined 值没有 toString()方法。`

#### 2. String()

如果你不确定一个值是不是 null 或 undefined，可以使用 `String() 转型函数`，它始终会返回表示相应类型值的字符串。String()函数遵循如下规则:

```
如果值有 toString()方法，则调用该方法（不传参数）并返回结果。
如果值是 null，返回"null"。
如果值是 undefined，返回"undefined"。
```

下面看几个例子：

```js
let value1 = 10;
let value2 = true;
let value3 = null;
let value4;

console.log(String(value1)); // "10" 
console.log(String(value2)); // "true" 
console.log(String(value3)); // "null" 
console.log(String(value4)); // "undefined"
```



### 数字转换成字符串

#### 使用 `+ ''`

性能之间的对比为 + '' > String() > toFixed()

#### 使用 `toString()`

整数和小数都能通过这个方法完整的转换成字符串

#### 要保留小数点后几位时，使用 `toFixed()`

可以保留小数位数，但会四舍五入，若要不四舍五入，可以使用 `Math.floor()` 先乘以 10 的倍数向下取整，然后再除以相同的 10 的倍数：

```js
// 四舍五入
var num =2.446242342;
num = num.toFixed(2);  // 输出结果为 2.45

// 不四舍五入
Math.floor(15.7784514000 * 100) / 100   // 输出结果为 15.77
```



## 字符串的 `replace` 和 `replaceAll`

历史上，字符串的实例方法`replace()`只能替换第一个匹配：

```js
'aabbcc'.replace('b', '_')
// 'aa_bcc'
```



ES2021 引入了 `replaceAll()` 方法，可以一次性替换所有匹配。它的用法与`replace()`相同，`返回一个新字符串，不会改变原字符串`：

```js
'aabbcc'.replaceAll('b', '_')
// 'aa__cc'
```



参考 `ECMAScript6 阮一峰 5.10`

https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/replace



## 模版字符串

模版字符串中可以进行计算：

```js
const height = 234
this.$refs.GPUFill.style.height = `${height * GRate}px`
```



img 标签的 src 中可以使用：

```html
<img :src="require(`@/assets/images/service/live_${serverNode.live_server}.png`)" alt="健康">
```



可以使用 && 和 ||：

```html
<div class="value">{{summary.server && summary.server.total_server}}</div>

<div class="value">{{summary.license && summary.license.physical_device_count || 0}}</div>

<div class="num">{{ summary.license && summary.license.activation_methods | activation_methods }}</div>
```



## 插值表达式

插值表达式中不能使用 `?.` 可选链

不能使用三元表达式，可以使用 && 和 ||



## 翻转字符串

```js
const str = 'abcde'
str.split('').reverse().join('')
```



## substr,slice,substring 的用法与区别

这三个方法都是截取字符串

先看方法的签名

```js
slice(start, end); // 参数可为负数。第二个参数是指定结束位置。
substring(start, end); // 参数为负数被替换成0。交换参数位置，小的在前。第二个参数是指定结束位置。
substr(start, length); // 参数可为负数。第二个参数是指定截取长度。
```

从签名可以看出 `substr` 和其他两个的差别，substr 第二个参数指定截取的长度，slice 和 substirng 第二个参数指定截取的结束位置

`slice` 和 `substring` 的差别在于slice的参数可以是负数，而substring不行。

slice 中的 start 如果为负数，会从尾部算起，-1表示倒数第一个，-2表示倒数第2个，end 同理，但是换算后 end 的位置一定要在 start 之后，否则返回空字符串。

substring 会取 start 和 end 中较小的值为 start,二者相等返回空字符串，任何一个参数为负数被替换为 0 (即该值会成为 start 参数)。



# Number

## String 转换成 Number

```js
// 报错，currentId 要求是 Number，而 route.params.id 是字符串
const currentId = route.params.id

// 添加一个 + 号即可
const currentId = +route.params.id
```



有 3 个函数可以将非数值转换为数值：`Number()`、`parseInt()` 和 `parseFloat()`。Number()是转型函数，可用于任何数据类型。后两个函数主要用于将字符串转换为数值。

Number()函数基于如下规则执行转换。

- 布尔值，true 转换为 1，false 转换为 0。 

- 数值，直接返回。

- null，返回 0。 

- undefined，返回 NaN。 

- 字符串，应用以下规则。
  - 如果字符串包含数值字符，包括数值字符前面带加、减号的情况，则转换为一个十进制数值。因此，Number("1")返回 1，Number("123")返回 123，Number("011")返回 11（忽略前面的零）。
  - 如果字符串包含有效的浮点值格式如"1.1"，则会转换为相应的浮点值（同样，忽略前面的零）。
  - 如果字符串包含有效的十六进制格式如"0xf"，则会转换为与该十六进制值对应的十进制整数值。
  - 如果是空字符串（不包含字符），则返回 0。 
  - 如果字符串包含除上述情况之外的其他字符，则返回 NaN。 

- 对象，调用 valueOf()方法，并按照上述规则转换返回的值。如果转换结果是 NaN，则调用toString()方法，再按照转换字符串的规则转换。

下面是几个具体的例子：

```js
let num1 = Number("Hello world!"); // NaN 
let num2 = Number(""); // 0 
let num3 = Number("000011"); // 11 
let num4 = Number(true); // 1
```



考虑到用 Number()函数转换字符串时相对复杂且有点反常规，通常在需要得到整数时可以优先使用 parseInt()函数。

parseInt()函数更专注于字符串是否包含数值模式。字符串最前面的空格会被忽略，从第一个非空格字符开始转换。如果第一个字符不是数值字符、加号或减号，parseInt()立即返回 NaN。这意味着空字符串也会返回 NaN（这一点跟 Number()不一样，它返回 0）。如果第一个字符是数值字符、加号或减号，则继续依次检测每个字符，直到字符串末尾，或碰到非数值字符。比如，"1234blue"会被转换为 1234，因为"blue"会被完全忽略。类似地，"22.5"会被转换为 22，因为小数点不是有效的整数字符。

假设字符串中的第一个字符是数值字符，parseInt()函数也能识别不同的整数格式（十进制、八进制、十六进制）。换句话说，如果字符串以"0x"开头，就会被解释为十六进制整数。如果字符串以"0"开头，且紧跟着数值字符，在非严格模式下会被某些实现解释为八进制整数。

下面几个转换示例有助于理解上述规则：

```js
let num1 = parseInt("1234blue"); // 1234 
let num2 = parseInt(""); // NaN 
let num3 = parseInt("0xA"); // 10，解释为十六进制整数
let num4 = parseInt(22.5); // 22 
let num5 = parseInt("70"); // 70，解释为十进制值
let num6 = parseInt("0xf"); // 15，解释为十六进制整数
```



不同的数值格式很容易混淆，因此 parseInt()也接收第二个参数，用于指定底数（进制数）。如果知道要解析的值是十六进制，那么可以传入 16 作为第二个参数，以便正确解析：

```js
let num = parseInt("0xAF", 16); // 175
```



## if 判断中的 number

```js
const num = -1
if (num) {
  console.log('yes') // 可以输出
}

const num = 0
if (num) {
  console.log('yes') // 不输出
}
```

`除 0 和 NaN 之外的任何数字都将转换为 true ！！！`

所以平常判断时要加上：

```js
const num = -1
if (num > 0) {
  console.log('yes') // 不输出
}
```





```js
-1 == true;        //false
-1 == false        //false
-1 ? true : false; //true
```

在前两种情况下，布尔值被强制转换为数字，-1 表示 true，0 表示 false。

在最后一种情况下，它是一个转换为布尔值的数字，除 0 和 NaN  之外的任何数字都将转换为true。这个测试用例实际上更像这样：

```js
-1 == 1; // false
-1 == 0; // false
true ? true : false; // true
```



## 判断是否为整数

Number.isInteger()

https://blog.csdn.net/Vasilis_1/article/details/78613086



# Object

## `Object.is() !!!!`

参考 `红宝书第 8 章` 的 `8.1.5`

用于解决 === 操作符也无能为力的特殊情况（-0, +0, NaN 的相等判定），为改善这类情况，ECMAScript 6 规范新增了 `Object.is()`。



## 对象的属性是变量（对象里面的 key 设置为变量）

对象的属性是变量时，要使用 `[]` 形式，若是字符串则可以使用 `.` 形式

```js
for (const item in server) {
  // bad
  server.item
  
  // good
  server[item]
}
```



## 对象解构

注意，当 Function(page)，而 page 中包括 current 和 size 时才可以解构：Function({ current, size })

而当 Function(current, size) 时，不能解构成 Function({ current, size })。属于`低级错误`



若是 Function(pagination, filters, sorter, { currentDataSource })

则我们结构时 Function({ current }, f, sorter)，这里解构出来的 current 是第一个参数 pagination.current





# Array

## ES6 的 `find` 和 `filter` 区别

数组的 `find` 方法参考 `红宝书第 6 章` 的 `6.2.12 断言函数`

数组的 `filter` 方法参考 `红宝书第 6 章` 的 `6.2.13`

https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/filter



区别：

find 和 filter 都是不改变原数组的方法，但是 find 只查出第一个符合条件的结果就直接返回，而 filter 则返回全部的结果，且返回结果仍然为数组。

> Vue中，filter 可以直接在 v-if 里使用



## Array.prototype.`indexOf()`

参考网址：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/indexOf



## Array.prototype.map()

Array.prototype.map() 中的回调函数必须得写一个，不能为空，会报错

例如：

```js
const arr = [1, 2, 3]

const newArr = arr.map() // 报错
const newArr = arr.map(x => x) //正确
```



## Array.prototype.reduce()

使用数组的 reduce 方法时，当数组长度为1，一定要指定一个初始值，不然一开始会将 arr[0] 当成初始值，然后 arr[1] 没有，就会报错。

有初始值时，从 arr[0] 开始加。

无初始值时，arr[0] 作为初始值，从 arr[1] 开始加。



## Array.prototype.concat()

concat方法不改变原数组！！！必须要用返回值

```js
const arr3 = arr1.cancat(arr2)
```





# Function

## ResizeObserver 方法

window.onsize() 可以侦听整个窗口的 size 变化。而如果要侦听某个 Dom 元素的 size 变化的话，就要使用 `ResizeObserver` 方法



使用场景：

在项目中 引入了 Echarts 进行绘图，在将菜单隐藏或打开的时候，不能触发 window.resize() 方法，导致绘制的图形发生了偏移（也就是此时 Echarts 不会触发 resize() 方法，只有当 window.resize() 变化时才会自动触发），目前需要的步骤就是在让菜单打开或者隐藏的时候，手动触发一次 resize 方法，让 Echarts 图形自适应。



解决方案：

1. 监听 Echarts 外层的 Dom 元素的 size 变化，变化时触发回调函数，在回调函数中执行 Echarts 的 resize() 方法。

> 不过这样子会报一个警告，因为当父组件引用子组件时，父组件的 mounted() 在先，子组件的 mounted() 在后，也就是说如果在 A 组件中引入 Echarts 作为子组件，然后在 A 组件的 mounted() 钩子函数中执行 this.$refs.chart.resize()，这时候子组件 Echarts 还未进行渲染，而通常 initChart() 这个初始化 Echarts 的方法会写在 Echarts 组件中的 mounted() 钩子函数中时，那么就会报这个警告。
>
> 因为这时候还未 initChart() 形成 Echarts 的实例，就去执行 Echarts 的 resize() 方法，所以就会报错：`resize() is not a function`

> 所以我们应该直接监听 Echarts 的 canvas 挂载的那个 DOM 上

2. 在将菜单隐藏或打开的时候执行 resize() 方法

> 这个时候可以封装一个通用的 Echarts 的 resize() 方法，将其绑定在 window.onresize 上，然后当菜单隐藏或打开时，也执行 window.onresize，那么这个时候就会自动执行这个方法



参考网址：

https://juejin.cn/post/6844903749253595143#heading-8

https://developer.mozilla.org/zh-CN/docs/Web/API/ResizeObserver



## switch

switch 语句不支持传入的检测值为 null 和 undefined 的情况



# Promise

异常捕获

```js
// 这个 action 是一个 promise，当这个 action 抛出一个错误时，可以使用 Promise.catch() 捕获
// 因为 promise 内部无论是 reject 或者 throw new Error，都可以通过 catch 回调捕获
await store.dispatch('user/getInfo').catch()


// 这两种情况都无法捕获
// 因为 promise 内部的错误不会冒泡出来（不论是 reject 还是 throw new Error），而是被 promise 吃掉了，只有通过 promise.catch 才可以捕获，所以用 Promise 一定要写 catch 啊
try {
  Promise.reject('err')
} catch (err) {
  console.log(err)
}

try {
  new Promise(() => {
    throw new Error('err')
  })
} catch (err) {
  console.log(err)
}


// 这两种情况可以捕获，因为 promise 的错误被 await 求值出来了
try {
  await Promise.reject('err')
} catch (err) {
  console.log(err)
}

try {
  await new Promise(() => {
    throw new Error('err')
  })
} catch (err) {
  console.log(err)
}

// 但这些情况都可以通过 Promise.catch() 来捕获
```





# 其他

## `console.log`

更改 `console.log` 中字体的颜色：

```js
// 添加 %c
console.log('%c 当前环境：%s', 'color: red', process.env.NODE_ENV)
console.log('%c 当前版本：%s', 'color: orange', process.env.VUE_APP_BUILD_VERSION)
```



`console.dir` 可以查看错误的详细信息



console 详解：

https://segmentfault.com/a/1190000004528137



## 可选链 `?.`

`?.` 不能用在 `vue 的模版(即不能用在 HTML 中)`中的插值表达式和模版字符串里，但可以用在 js 函数里的插值表达式和模版字符串里



可选链的使用场景：

- 在 computed 中，若直接使用，或者在模版字符串中使用到了某个 data 中的变量，这个变量是多层级的，而此时这个变量需要异步获取到

  - 例如：

  - ```js
    breadcrumbRoutes () {
      const match = this.$route.matched.filter(route => route.meta.title && !route.meta.blank)
      const snode = { name: 'ServerNode', meta: { title: '服务节点' } }
      const currentServer = { name: 'ServerTask', meta: { title: `${this.server?.ip}` } }
    }
    ```



## 时间转换

```js
 //格林尼治2019-03-19T16:00:00.000Z  ==>>  2019-03-20 00:00:00   与北京时间8小时时差
            //格林尼治2019-03-19T16:00:00.000Z  ==>>  2019-03-20 00:00:00 
	       // //时间戳1553547600000  转 //  2019-03-25T21:00:00.000Z
            function formDate(dateForm) {
                if (dateForm === "") {  //解决deteForm为空传1970-01-01 00:00:00
                    return "";
                }else{
                    var dateee = new Date(dateForm).toJSON();
                    var date = new Date(+new Date(dateee)+ 8 * 3600 * 1000).toISOString().replace(/T/g,' ').replace(/\.[\d]{3}Z/,'');
                    return date;
                }
            }
            console.log(formDate("2019-03-19T16:00:00.000Z"))//2019-03-20 00:00:00
            console.log(formDate(1553547600000))             //2019-03-26 05:00:00
            console.log(formatDateT(1553547600000))          //2019-03-25T21:00:00.000Z
 
             //时间戳1553547600000  转 //  2019-03-25T21:00:00.000Z
            function formatDateT(dataTime){
                // var timestamp3 = 1551686227000; 
                var timestamp = dataTime;
                var newDate = new Date(dataTime ); 
                // var newDate = new Date(dataTime + 8 * 3600 * 1000 );               
 
                newDate.getTime(timestamp * 1000);                
                // console.log(newDate.toDateString());//Mon Mar 11 2019                
                // console.log(newDate.toGMTString()); //Mon, 11 Mar 2019 06:55:07 GMT                
                // console.log(newDate.toISOString()); //2019-03-11T06:55:07.622Z
                return newDate.toISOString()
            }
```

参考网址：https://blog.csdn.net/weixin_42481234/article/details/88566334



```js
new Date('') // Invalid Date
new Date(null) // Thu Jan 01 1970 08:00:00 GMT+0800 (中国标准时间)
new Date('').toJSON() // null
new Date(+new Date(null)+ 8 * 3600 * 1000).toISOString().replace(/T/g,' ').replace(/\.[\d]{3}Z/,'') // 1970-01-01 00:00:00
```



## window.onresize

window.onresize 也需要解绑：window.onresize = ‘ ’



## 检测滚动条

检测一个 scroll 的 div 是否有滚动条，有滚动条时给右侧留个宽度

```js
// 如果超过高度需要滚动时，则右边留出滚动条的宽度
const scrollWidth = ref(14);
watch(
  selectedController,
  (val) => {
    if (val && selectedServerList?.value?.length > 6) {
      // 一定要在 nextTick 中，否则页面内容还未渲染
      nextTick(() => {
        const element: HTMLElement | null =
          document.querySelector(".scroll-box");
        if (element && element.offsetWidth !== element.clientWidth) {
          // 计算滚动条的宽度
          scrollWidth.value = element.offsetWidth - element.clientWidth + 14;
        }
      });
    } else {
      scrollWidth.value = 14;
    }
  },
  { immediate: true }
);
```

