# 一、ES2015

## 1.1 let 与 const

let 声明的变量只在 let 命令所在的代码块内有效。

```js
{  let a = 0;
   var b = 1; 
} 

a  // ReferenceError: a is not defined
b  // 1
```

const 声明一个只读的常量，一旦声明，常量的值就不能改变。这也意味着，一旦声明必须初始化，否则会报错。

```js
const PI = "3.1415926";
PI  // 3.1415926

const MY_AGE;  // SyntaxError: Missing initializer in const declaration 
```

暂时性死区：

```js
var PI = "a";
if (true) { 
  console.log(PI);  // ReferenceError: PI is not defined 
  const PI = "3.1415926"; 
}
```

ES6 明确规定，代码块内如果存在 let 或者 const，代码块会对这些命令声明的变量从块的开始就形成一个封闭作用域。代码块内，在声明变量 PI 之前使用它会报错。

## 1.2 解构赋值

解构赋值是对赋值运算符的扩展。

他是一种针对数组或者对象进行模式匹配，然后对其中的变量进行赋值。

在解构中，有下面两部分参与：

1. 解构的源：解构赋值表达式的右边部分
2. 解构的目标：解构赋值表达式的左边部分

### 1.2.1 数组的解构

基本

```js
let [a, b, c] = [1, 2, 3]; 
// a = 1 
// b = 2 
// c = 3
```

可嵌套

```js
let [a, [[b], c]] = [1, [[2], 3]]; 
// a = 1 
// b = 2 
// c = 3
```

可忽略

```js
let [a, , b] = [1, 2, 3]; 
// a = 1 
// b = 3
```

不完全解构

```js
let [a = 1, b] = []; 
// a = 1, b = undefined
```

rest 运算符

```js
let [a, ...rest] = [1, 2, 3]; 
//a = 1
//rest = [2, 3]
```

字符串

```js
// 在数组的解构中，解构的目标若为可遍历对象，皆可进行解构赋值。可遍历对象即实现 Iterator 接口的数据。
let [a, b, c, d, e] = 'hello'; 
// a = 'h' 
// b = 'e' 
// c = 'l' 
// d = 'l' 
// e = 'o'
```

### 1.2.2 对象的解构

基本

```js
let { foo, bar } = { foo: 'aaa', bar: 'bbb' };
// foo = 'aaa' 
// bar = 'bbb'  

let { baz : foo } = { baz : 'ddd' }; 
// foo = 'ddd'
```

可嵌套可忽略

```js
let obj = {p: ['hello', {y: 'world'}] }; 
let {p: [x, { y }] } = obj; 
// x = 'hello' 
// y = 'world' 

let obj = {p: ['hello', {y: 'world'}] }; let {p: [x, {  }] } = obj; 
// x = 'hello'
```

不完全解构

```js
let obj = {p: [{y: 'world'}] }; let {p: [{ y }, x ] } = obj; 
// x = undefined 
// y = 'world'
```

rest 运算符

```js
let {a, b, ...rest} = {a: 10, b: 20, c: 30, d: 40}; 
// a = 10 
// b = 20 
// rest = {c: 30, d: 40}
```

解构默认值

```js
let {a = 10, b = 5} = {a: 3}; 
// a = 3; b = 5; 

let {a: aa = 10, b: bb = 5} = {a: 3}; 
// aa = 3; bb = 5;
```



## 1.3 Symbol

一种新的原始数据类型 Symbol ，表示独一无二的值，最大的用法是用来定义对象的唯一属性名。

Symbol 函数栈不能用 new 命令，因为 Symbol 是原始数据类型，不是对象。可以接受一个字符串作为参数，为新创建的 Symbol 提供描述，用来显示在控制台或者作为字符串的时候使用，便于区分。

```js
let sy = Symbol("KK"); 
console.log(sy);   // Symbol(KK) 
typeof(sy);        // "symbol"  

// 相同参数 Symbol() 返回的值不相等 
let sy1 = Symbol("kk");  
sy === sy1;       // false
```

由于每一个 Symbol 的值都是不相等的，所以 Symbol 可以作为对象的属性名，可以保证属性不重名。

```js
let sy = Symbol("key1");  
// 写法1 
let syObject = {}; 
syObject[sy] = "kk"; 
console.log(syObject);    // {Symbol(key1): "kk"}  

// 写法2 
let syObject = {  [sy]: "kk" }; 
console.log(syObject);    // {Symbol(key1): "kk"}  

// 写法3 
let syObject = {}; 
Object.defineProperty(syObject, sy, {value: "kk"}); 
console.log(syObject);   // {Symbol(key1): "kk"}
```

Symbol 作为对象属性名时不能用.运算符，要用方括号。因为.运算符后面是字符串，所以取到的是字符串 sy 属性，而不是 Symbol 值 sy 属性。

```js
let syObject = {}; syObject[sy] = "kk";  syObject[sy];  // "kk" syObject.sy;   // undefined
```



## 1.4 Map

Map 对象保存键值对。任何值(对象或者原始值) 都可以作为一个键或一个值。

Map 与 Object 的区别：

- 一个 Object 的键只能是字符串或者 Symbols，但一个 Map 的键可以是任意值。
- Map 中的键值是有序的（FIFO 原则），而添加到对象中的键则不是。
- Map 的键值对个数可以从 size 属性获取，而 Object 的键值对个数只能手动计算。
- Object 都有自己的原型，原型链上的键名有可能和你自己在对象上的设置的键名产生冲突。

### Map 的迭代

for...of

```js
var myMap = new Map();
myMap.set(0, "zero");
myMap.set(1, "one");
  
// 将会显示两个 log。 一个是 "0 = zero" 另一个是 "1 = one"
for (var [key, value] of myMap) {
  console.log(key + " = " + value);
}
// 等价于
for (var [key, value] of myMap.entries()) {
  console.log(key + " = " + value);
}
/* 这个 entries 方法返回一个新的 Iterator 对象，它按插入顺序包含了 Map 对象中每个元素的 [key, value] 数组。 */
  
 
// 将会显示两个log。 一个是 "0" 另一个是 "1"
for (var key of myMap.keys()) {
  console.log(key);
}
/* 这个 keys 方法返回一个新的 Iterator 对象， 它按插入顺序包含了 Map 对象中每个元素的键。 */
  
 
// 将会显示两个log。 一个是 "zero" 另一个是 "one"
for (var value of myMap.values()) {
  console.log(value);
}
/* 这个 values 方法返回一个新的 Iterator 对象，它按插入顺序包含了 Map 对象中每个元素的值。 */
```

forEach()

```js
let myMap = new Map();
myMap.set(0, "zero");
myMap.set(1, "one");
  
// 将会显示两个 logs。 一个是 "0 = zero" 另一个是 "1 = one"
myMap.forEach(function(value, key) {
  console.log(key + " = " + value);
})
```



## 1.5 Set

Set 对象允许存储任何类型的唯一值，无论是原始值或者是对象引用。

Set 对象存储的值总是唯一的，所以需要判断两个值是否恒等。有几个特殊值需要特殊对待：

- +0 与 -0 在存储判断唯一性的时候是恒等的，所以不重复；
- undefined 与 undefined 是恒等的，所以不重复；
- NaN 与 NaN 是不恒等的，但是在 Set 中只能存一个，不重复。

基本示例：

```js
let mySet = new Set();
  
mySet.add(1); // Set(1) {1}
mySet.add(5); // Set(2) {1, 5}
mySet.add(5); // Set(2) {1, 5} 这里体现了值的唯一性
mySet.add("some text");
// Set(3) {1, 5, "some text"} 这里体现了类型的多样性
 
var o = {a: 1, b: 2};
mySet.add(o);
mySet.add({a: 1, b: 2});
// Set(5) {1, 5, "some text", {…}, {…}}
// 这里体现了对象之间引用不同不恒等，即使值相同，Set 也能存储
```



## 1.6 函数

### 1.6.1 默认参数

```js
function fn(name,age=17){
 console.log(name+","+age);
}
fn("Amy",18);  // Amy,18
fn("Amy","");  // Amy,
fn("Amy");     // Amy,17
```

注意点：使用函数默认参数时，不允许有同名的参数。

### 1.6.2 rest不定参数

不定参数用来表示不确定参数个数，形如，...变量名，由...加上一个具名参数标识符组成。具名参数只能放在参数组的最后，并且有且只有一个不定参数。

```js
function f(...values){
    console.log(values.length);
}
f(1,2);      //2
f(1,2,3,4);  //4
```

### 1.6.3 箭头函数

```js
var a = v => console.log(v)
// 等同于
var b = function(v){
  console.log(v)
}
 
var f = () => 5
// 等同于
var f = function () { return 5 }
```

如果箭头函数的代码块部分多于一条语句，就需要使用大括号将他们括起来。并且使用return返回：

```js
var sum = (num1, num2) => { return num1 + num2; }
```

由于大括号被解释为代码块，所以如果箭头函数直接返回一个对象，必须在对象外面加上括号，否则会报错：

```js
// 报错
let getTempItem = id => { id: id, name: "Temp" };
 
// 不报错
let getTempItem = id => ({ id: id, name: "Temp" });
```

箭头函数可以与变量解构结合使用：

```js
const full = ({ first, last }) => first + ' ' + last;
 
// 等同于
function full(person) {
  return person.first + ' ' + person.last;
}
```

箭头函数有几个使用注意点：

（1）函数体内的 `this` 对象，就是定义时所在的对象，而不是使用时所在的对象。

（2）不可以当作构造函数，也就是说，不可以使用 `new` 命令，否则会抛出一个错误。

（3）不可以使用 `arguments` 对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。

（4）不可以使用 `yield` 命令，因此箭头函数不能用作 Generator 函数



## 1.7 Class

ES2015 中的类（class）是在基于原型的面向对象模式上简单包装的语法糖。拥有一个 单一且方便的声明形式将更易于使用，并且 鼓励混合使用。



类（class）支持基于原型的`继承`、`super 调用`、 `实例`和`静态方法`以及`构造函数`。

```js
// ES5写法
function Point(x, y) {
  this.x = x;
  this.y = y;
}
 
Point.prototype.toString = function () {
  return '(' + this.x + ', ' + this.y + ')';
};
 
var p = new Point(1, 2);
```

```js
// 用class改写
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
 
  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
```

> 注意，定义`toString()`方法的时候，前面不需要加上`function`这个关键字，直接把函数定义放进去了就可以了。另外，方法与方法之间不需要逗号分隔，加了会报错。



类的数据类型就是函数，类本身就指向构造函数：

```js
class Point {
  // ...
}
 
typeof Point // "function"
Point === Point.prototype.constructor // true
```

构造函数的 `prototype `属性，在 ES6 的“类”上面继续存在。事实上，类的所有方法都定义在类的 `prototype `属性上面。因此，在类的实例上面调用方法，其实就是调用原型上的方法。



另外，类的内部所有定义的方法，都是`不可枚举`的（non-enumerable）：

```js
class Point {
  constructor(x, y) {
    // ...
  }
 
  toString() {
    // ...
  }
}
 
Object.keys(Point.prototype)
// []
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]
```

上面代码中，`toString()`方法是`Point`类内部定义的方法，它是不可枚举的。这一点与 ES5 的行为不一致。



## 1.8 字符串

### 1.8.1 模版字符串

模板字符串为构造字符串提供了语法糖。

模板字符串是增强版的字符串，用反引号（`）标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量。

```js
// 普通字符串
`In JavaScript '\n' is a line-feed.`
 
// 多行字符串
`In JavaScript this is
 not legal.`
 
console.log(`string text line 1
string text line 2`);
 
// 字符串中嵌入变量
let name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`
```

上面代码中的模板字符串，都是用反引号表示。如果在模板字符串中需要使用反引号，则前面要用反斜杠转义。

```js
let greeting = `\`Yo\` World!`;
```

模板字符串中嵌入变量，需要将变量名写在`${}`之中。这个大括号内部可以放入任意的 JavaScript 表达式，可以进行运算，以及引用对象属性。

```js
let x = 1;
let y = 2;
 
`${x} + ${y} = ${x + y}`
// "1 + 2 = 3"
 
`${x} + ${y * 2} = ${x + y * 2}`
// "1 + 4 = 5"
 
let obj = {x: 1, y: 2};
`${obj.x + obj.y}`
// "3"
```

模板字符串之中还能调用函数。

```js
function fn() {
  return "Hello World";
}
 
`foo ${fn()} bar`
// foo Hello World bar
```

### 1.8.2 子串的识别

ES6 之前判断字符串是否包含子串，用 indexOf 方法，ES6 新增了子串的识别方法，如下：

- **includes()**：返回布尔值，判断是否找到参数字符串。
- **startsWith()**：返回布尔值，判断参数字符串是否在原字符串的头部。
- **endsWith()**：返回布尔值，判断参数字符串是否在原字符串的尾部。

以上三个方法都可以接受两个参数，需要搜索的字符串，和可选的搜索起始位置索引。

例如：

```js
let string = "apple,banana,orange";
string.includes("banana");     // true
string.startsWith("apple");    // true
string.endsWith("apple");      // false
string.startsWith("banana",6)  // true
```

注意点：

- 这三个方法只返回布尔值，如果需要知道子串的位置，还是得用 indexOf 和 lastIndexOf 。
- 这三个方法如果传入了正则表达式而不是字符串，会抛出错误。而 indexOf 和 lastIndexOf 这两个方法，它们会将正则表达式转换为字符串并搜索它。

### 1.8.3 repeat()

repeat() 返回新的字符串，表示将字符串重复指定次数返回。

```js
console.log("Hello,".repeat(2));  // "Hello,Hello,"
 
// 参数如果是小数，则向下取整
console.log("Hello,".repeat(3.2));  // "Hello,Hello,Hello,"
 
// 如果参数是 0 至 -1 之间的小数，会进行取整运算，0 至 -1 之间的小数取整得到 -0 ，等同于 repeat 零次
console.log("Hello,".repeat(-0.5));  // ""
 
// 如果参数是 NaN，则 repeat 零次
```



## 1.9 数值

### 1.9.1 Number.isFinite()

该方法用于检查一个数值是否为有限的（ finite ），即不是 Infinity

```js
console.log( Number.isFinite(1));   // true
console.log( Number.isFinite(0.1)); // true
  
// NaN 不是有限的
console.log( Number.isFinite(NaN)); // false
  
console.log( Number.isFinite(Infinity));  // false
console.log( Number.isFinite(-Infinity)); // false
```

### 1.9.2 Number.parseInt()

逐步减少全局方法，用于全局变量的模块化。

该方法的行为没有发生改变。

### 1.9.3 Math 对象的扩展

Math 对象上新增了 17 个数学相关的静态方法，这些方法只能在 Math 对象上调用。

详情见：[Math 对象的扩展](https://es6.ruanyifeng.com/#docs/number#Math-对象的扩展)



## 1.10 对象

### 1.10.1 属性的简洁表示法

ES6允许对象的属性直接写变量，这时候属性名是变量名，属性值是变量值：

```js
const age = 12;
const name = "Amy";
const person = {age, name};
person   //{age: 12, name: "Amy"}

//等同于
const person = {age: age, name: name}
```

方法名也可以简写：

```js
const person = {
  sayHi(){
    console.log("Hi");
  }
}
person.sayHi();  //"Hi"

//等同于
const person = {
  sayHi:function(){
    console.log("Hi");
  }
}
person.sayHi();//"Hi"
```

### 1.10.2 属性名表达式

ES6允许用表达式作为属性名，但是一定要将表达式放在方括号内：

```js
const obj = {
 ["he"+"llo"](){
   return "Hi";
  }
}
obj.hello();  //"Hi"
```

注意点：属性的简洁表示法和属性名表达式不能同时使用，否则会报错。

### 1.10.3 Object.assign(target, source_1, source_2, ...)

该方法用于将源对象的所有可枚举属性复制到目标对象中。

基本用法：

```js
let target = {a: 1};
let object2 = {b: 2};
let object3 = {c: 3};
Object.assign(target,object2,object3); 
 
// 第一个参数是目标对象，后面的参数是源对象
target;  // { a: 1, b: 2, c: 3 }
```

- 如果目标对象和源对象有同名属性，或者多个源对象有同名属性，则后面的属性会覆盖前面的属性。
- 如果该函数只有一个参数，当参数为对象时，直接返回该对象；当参数不是对象时，会先将参数转为对象然后返回。

```js
Object.assign(3);         // Number {3}
typeof Object.assign(3);  // "object"
```

但是，因为 null 和 undefined 不能转化为对象，所以会报错。

还需要注意，Object.assign() 的属性拷贝是`浅拷贝`。

### 1.10.4 Object.is(value1, value2)

用来比较两个值是否严格相等，与（===）基本类似。

基本用法：

```js
Object.is("q","q");      // true
Object.is(1,1);          // true
Object.is([1],[1]);      // false
Object.is({q:1},{q:1});  // false
```

与（===）的区别（`弥补 === 的不足点`）：

```js
//一是+0不等于-0
Object.is(+0,-0);  //false
+0 === -0  //true
 
//二是NaN等于本身
Object.is(NaN,NaN); //true
NaN === NaN  //false
```



## 1.11 数组

### 1.11.1 Array.of()

该方法将参数中所有值作为元素形成数组。

```js
console.log(Array.of(1, 2, 3, 4)); // [1, 2, 3, 4]
  
// 参数值可为不同类型
console.log(Array.of(1, '2', true)); // [1, '2', true]
  
// 参数为空时返回空数组
console.log(Array.of()); // []
```

这个方法的主要目的，是`弥补数组构造函数 Array() 的不足`。因为参数个数的不同，会导致 Array() 的行为有差异：

```js
Array() // []
Array(3) // [, , ,]
Array(3, 11, 8) // [3, 11, 8]
```

上面代码中，Array 方法没有参数、一个参数、三个参数时，返回结果都不一样。只有当参数个数不少于 2 个时，Array() 才会返回由参数组成的新数组。参数个数只有一个时，实际上是指定数组的长度。

### 1.11.2 Array.from(arrayLike, mapFn, this)

将类数组对象或可迭代对象转化成数组。

```js
// 参数为数组,返回与原数组一样的数组
console.log(Array.from([1, 2])); // [1, 2]
  
// 参数含空位
console.log(Array.from([1, , 3])); // [1, undefined, 3]
 
// 参数为类数组
Array.from(arrayLike[, mapFn[, thisArg]]) // 返回值为转换后的数组
```

第二个参数是可选的，为一个 map 函数，用于对每个元素进行处理，放入数组的是处理后的元素：

```js
console.log(Array.from([1, 2, 3], (n) => n * 2)); // [2, 4, 6]
```

第三个参数是可选的，是一个 this，可指定 map 函数执行时的 this 对象

#### 类数组对象

一个类数组对象必须含有 length 属性，且元素属性名必须是数值或者可转换为数值的字符：

```js
let arr = Array.from({
  0: '1',
  1: '2',
  2: 3,
  length: 3
});
console.log(); // ['1', '2', 3]
  
// 没有 length 属性,则返回空数组
let array = Array.from({
  0: '1',
  1: '2',
  2: 3,
});
console.log(array); // []
  
// 元素属性名不为数值且无法转换为数值，返回长度为 length 元素值为 undefined 的数组 
let array1 = Array.from({
  a: 1,
  b: 2,
  length: 2
});
console.log(array1); // [undefined, undefined]
```

#### 可迭代对象

实现了 Iterator 接口，例如 Map, Set, String：

```js
// 转换 Map
let map = new Map();
map.set('key0', 'value0');
map.set('key1', 'value1');
console.log(Array.from(map)); // [['key0', 'value0'],['key1',
// 'value1']]
 
// 转换 Set
let arr = [1, 2, 3];
let set = new Set(arr);
console.log(Array.from(set)); // [1, 2, 3]
 
// 转换字符串
let str = 'abc';
console.log(Array.from(str)); // ["a", "b", "c"]
```

### 1.11.3 Array.prototype.find()

查找数组中符合条件的元素，若有多个符合条件的元素，则返回第一个元素。

```js
let arr = Array.of(1, 2, 3, 4);
console.log(arr.find(item => item > 2)); // 3
  
// 数组空位处理为 undefined
console.log([, 1].find(n => true)); // undefined
```

find 和 filter 都是不改变原数组的方法，但是 find 只查出第一个符合条件的结果就直接返回，而 filter 则返回全部的结果，且返回结果仍然为数组。

### 1.11.4 Array.prototype.findIndex()

查找数组中符合条件的元素索引，若有多个符合条件的元素，则返回第一个元素索引。

```js
let arr = Array.of(1, 2, 1, 3);
// 参数1：回调函数
// 参数2(可选)：指定回调函数中的 this 值
console.log(arr.findIndex(item => item = 1)); // 0
  
// 数组空位处理为 undefined
console.log([, 1].findIndex(n => true)); //0
```

### 1.11.5 Array.prototype.fill()

将一定范围索引的数组元素内容填充为单个指定的值。

```js
let arr = Array.of(1, 2, 3, 4);
 
// 参数1：用来填充的值
// 参数2：被填充的起始索引
// 参数3(可选)：被填充的结束索引，默认为数组末尾
console.log(arr.fill(0,1,2)); // [1, 0, 3, 4]
```

### 1.11.6 Array.prototype.copyWithin()

将一定范围索引的数组元素修改为此数组另一指定范围索引的元素。

```js
// 参数1：被修改的起始索引
// 参数2：被用来覆盖的数据的起始索引
// 参数3(可选)：被用来覆盖的数据的结束索引，默认为数组末尾
console.log([1, 2, 3, 4].copyWithin(0,2,4)); // [3, 4, 3, 4]
  
// 参数1为负数表示倒数
console.log([1, 2, 3, 4].copyWithin(-2, 0)); // [1, 2, 1, 2]
  
console.log([1, 2, ,4].copyWithin(0, 2, 4)); // [, 4, , 4]
```

### 1.11.7 实例方法 entries(), keys() 和 values()

ES6 提供三个新的方法—— entries()，keys() 和 values() ——用于遍历数组。它们都返回一个遍历器对象，可以用 for...of 循环进行遍历，唯一的区别是 keys() 是对键名的遍历、values() 是对键值的遍历，entries() 是对键值对的遍历。

```js
for (let index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0
// 1
 
for (let elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'
 
for (let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"
```



## 1.12 迭代器

Iterator 是 ES6 引入的一种新的遍历机制，迭代器有两个核心概念：

- 迭代器是一个统一的接口，它的作用是使各种数据结构可被便捷的访问，它是通过一个键为 Symbol.iterator 的方法来实现
- 迭代器是用于遍历数据结构元素的指针（如数据库中的游标）

迭代的过程如下：

- 通过 Symbol.iterator 创建一个迭代器，指向当前数据结构的起始位置
- 随后通过 next 方法进行向下迭代指向下一个位置， next 方法会返回当前位置的对象，对象包含了 value 和 done 两个属性， value 是当前属性的值， done 用于判断是否遍历结束
- 当 done 为 true 时则遍历结束

例如：

```js
const items = ["zero", "one", "two"];
const it = items[Symbol.iterator]();
  
it.next();
>{value: "zero", done: false}
it.next();
>{value: "one", done: false}
it.next();
>{value: "two", done: false}
it.next();
>{value: undefined, done: true}
```

可迭代的数据结构有：Array（arguments 也可遍历）, String, Map, Set，我们可以使用 for...of 循环来对这些数据结构进行迭代。

## 1.13 模块

ES6 的模块自动开启严格模式，不管你有没有在模块头部加上 use strict。

模块中可以导入和导出各种类型的变量，如函数，对象，字符串，数字，布尔值，类等。

每个模块都有自己的上下文，每一个模块内声明的变量都是局部变量，不会污染全局作用域。

每一个模块只加载一次（是单例的）， 若再去加载同目录下同文件，直接从内存中读取。

### 1.13.1 export 与 import

模块导入导出各种类型的变量，如字符串，数值，函数，类。

- 导出的函数声明与类声明必须要有名称（export default 命令另外考虑）。 
- 不仅能导出声明还能导出引用（例如函数）。
- export 命令可以出现在模块的任何位置，但必需处于模块顶层。
- import 命令会提升到整个模块的头部，首先执行。

```js
/*-----export [test.js]-----*/
let myName = "Tom";
let myAge = 20;
let myfn = function(){
    return "My name is" + myName + "! I'm '" + myAge + "years old."
}
let myClass =  class myClass {
    static a = "yeah!";
}
export { myName, myAge, myfn, myClass }
  
/*-----import [xxx.js]-----*/
import { myName, myAge, myfn, myClass } from "./test.js";
console.log(myfn());// My name is Tom! I'm 20 years old.
console.log(myAge);// 20
console.log(myName);// Tom
console.log(myClass.a );// yeah!
```

建议使用大括号指定所要输出的一组变量写在文档尾部，明确导出的接口。

函数与类都需要有对应的名称。

### 1.13.2 as 的用法

export 命令导出的接口名称，须和模块内部的变量有一一对应关系。导入的变量名，须和导出的接口名称相同，即顺序可以不一致。

不同模块导出接口名称命名重复时，可使用 as 重新定义变量名。

```js
/*-----export [test.js]-----*/
let myName = "Tom";
export { myName as exportName }
  
/*-----import [xxx.js]-----*/
import { exportName } from "./test.js";
console.log(exportName);// Tom
使用 as 重新定义导出的接口名称，隐藏模块内部的变量
/*-----export [test1.js]-----*/
let myName = "Tom";
export { myName }
/*-----export [test2.js]-----*/
let myName = "Jerry";
export { myName }
/*-----import [xxx.js]-----*/
import { myName as name1 } from "./test1.js";
import { myName as name2 } from "./test2.js";
console.log(name1);// Tom
console.log(name2);// Jerry
```

### 1.13.3 import 命令的特点

只读属性：不允许在加载模块的脚本里面，改写接口的引用指向，即可以改写 import 变量类型为对象的属性值，不能改写 import 变量类型为基本类型的值。

```js
import {a} from "./xxx.js"
a = {}; // error
  
import {a} from "./xxx.js"
a.foo = "hello"; // a = { foo : 'hello' }
```

单例模式：多次重复执行同一句 import 语句，那么只会执行一次，而不会执行多次。import 同一模块，声明不同接口引用，会声明对应变量，但只执行一次 import 。

```js
import { a } "./xxx.js";
import { a } "./xxx.js";
// 相当于 import { a } "./xxx.js";
  
import { a } from "./xxx.js";
import { b } from "./xxx.js";
// 相当于 import { a, b } from "./xxx.js";
```

静态执行特性：import 是静态执行，所以不能使用表达式和变量。

```js
import { "f" + "oo" } from "methods";
// error
let module = "methods";
import { foo } from module;
// error
if (true) {
  import { foo } from "method1";
} else {
  import { foo } from "method2";
}
// error
```

### 1.13.4 export default 命令

- 在一个文件或模块中，export、import 可以有多个，export default 仅有一个。
- export default 中的 default 是对应的导出接口变量。
- 通过 export 方式导出，在导入时要加{ }，export default 则不需要。
- export default 向外暴露的成员，可以使用任意变量来接收。

```js
var a = "My name is Tom!";
export default a; // 仅有一个
export default var c = "error";
// error，default 已经是对应的导出变量，不能跟着变量声明语句
  
import b from "./xxx.js"; // 不需要加{}， 使用任意变量接收
```

## 1.14 Proxy

Proxy 可以对目标对象的读取、函数调用等操作进行拦截，然后进行操作处理。它不直接操作对象，而是像代理模式，通过对象的代理对象进行操作，在进行这些操作时，可以添加一些需要的额外操作。



基本用法：

一个 Proxy 对象由两个部分组成： target 、 handler 。

在通过 Proxy 构造函数生成实例对象时，需要提供这两个参数。 target 即目标对象， handler 是一个对象，声明了代理 target 的指定行为。

```js
let target = {
    name: 'Tom',
    age: 24
}
let handler = {
    get: function(target, key) {
        console.log('getting '+key);
        return target[key]; // 不是target.key
    },
    set: function(target, key, value) {
        console.log('setting '+key);
        target[key] = value;
    }
}
let proxy = new Proxy(target, handler)
proxy.name     // 实际执行 handler.get
proxy.age = 25 // 实际执行 handler.set
// getting name
// setting age
// 25
```



```js
// target 可以为空对象
let targetEpt = {}
let proxyEpt = new Proxy(targetEpt, handler)
// 调用 get 方法，此时目标对象为空，没有 name 属性
proxyEpt.name // getting name
// 调用 set 方法，向目标对象中添加了 name 属性
proxyEpt.name = 'Tom'
// setting name
// "Tom"
// 再次调用 get ，此时已经存在 name 属性
proxyEpt.name
// getting name
// "Tom"
  
// 通过构造函数新建实例时其实是对目标对象进行了浅拷贝，因此目标对象与代理对象会互相
// 影响
console.log(targetEpt)
// {name: "Tom"}
  
// handler 对象也可以为空，相当于不设置拦截操作，直接访问目标对象
let targetEmpty = {}
let proxyEmpty = new Proxy(targetEmpty,{})
proxyEmpty.name = "Tom"
console.log(targetEmpty) // {name: "Tom"}
```

### 1.14.1 get(target, propKey, receiver)

用于 target 对象上 propKey 的读取操作。

```js
let exam ={
    name: "Tom",
    age: 24
}
let proxy = new Proxy(exam, {
  get(target, propKey, receiver) {
    console.log('Getting ' + propKey);
    return target[propKey];
  }
})
proxy.name
// Getting name
// "Tom"
```

get() 方法可以继承：

```js
let proxy = new Proxy({}, {
  get(target, propKey, receiver) {
      // 实现私有属性读取保护
      if(propKey[0] === '_'){
          throw new Erro(`Invalid attempt to get private     "${propKey}"`);
      }
      console.log('Getting ' + propKey);
      return target[propKey];
  }
});
  
let obj = Object.create(proxy);
obj.name
// Getting name
```

### 1.14.2 set(target, propKey, value, receiver)

用于拦截 target 对象上的 propKey 的赋值操作。如果目标对象自身的某个属性，不可写且不可配置，那么set方法将不起作用。

```js
let validator = {
    set: function(obj, prop, value) {
        if (prop === 'age') {
            if (!Number.isInteger(value)) {
                throw new TypeError('The age is not an integer');
            }
            if (value > 200) {
                throw new RangeError('The age seems invalid');
            }
        }
        // 对于满足条件的 age 属性以及其他属性，直接保存
        obj[prop] = value;
    }
};
let proxy= new Proxy({}, validator)
proxy.age = 100;
proxy.age           // 100
proxy.age = 'oppps' // 报错
proxy.age = 300     // 报错
```

第四个参数 receiver 表示原始操作行为所在对象，一般是 Proxy 实例本身。

```js
const handler = {
    set: function(obj, prop, value, receiver) {
        obj[prop] = receiver;
    }
};
const proxy = new Proxy({}, handler);
proxy.name= 'Tom';
proxy.name=== proxy // true
  
const exam = {}
Object.setPrototypeOf(exam, proxy)
exam.name = "Tom"
exam.name === exam // true
```

注意，严格模式下，set代理如果没有返回true，就会报错。

### 1.14.3 apply(target, ctx, args)

用于拦截函数的调用、call 和 reply 操作。target 表示目标对象，ctx 表示目标对象上下文，args 表示目标对象的参数数组。

```js
function sub(a, b){
    return a - b;
}
let handler = {
    apply: function(target, ctx, args){
        console.log('handle apply');
        return Reflect.apply(...arguments);
    }
}
let proxy = new Proxy(sub, handler)
proxy(2, 1)
// handle apply
// 1
```



还有更多的操作方法，详情见：[Proxy 实例方法](https://es6.ruanyifeng.com/#docs/proxy#Proxy-实例的方法)



## 1.15 Reflect

ES6 中将 Object 的一些明显属于语言内部的方法移植到了 Reflect 对象上（当前某些方法会同时存在于 Object 和 Reflect 对象上），未来的新方法会只部署在 Reflect 对象上。

Reflect 对象对某些方法的返回结果进行了修改，使其更合理。

Reflect 对象使用函数的方式实现了 Object 的命令式操作。

### 1.15.1 Reflect.get(target, name, receiver)

查找并返回 target 对象的 name 属性。

```js
let exam = {
    name: "Tom",
    age: 24,
    get info(){
        return this.name + this.age;
    }
}
Reflect.get(exam, 'name'); // "Tom"
  
// 当 target 对象中存在 name 属性的 getter 方法， getter 方法的 this 会绑定 // receiver
let receiver = {
    name: "Jerry",
    age: 20
}
Reflect.get(exam, 'info', receiver); // Jerry20
  
// 当 name 为不存在于 target 对象的属性时，返回 undefined
Reflect.get(exam, 'birth'); // undefined
  
// 当 target 不是对象时，会报错
Reflect.get(1, 'name'); // TypeError
```

### 1.15.2 Reflect.set(target, name, value, receiver)

将 target 的 name 属性设置为 value。返回值为 boolean ，true 表示修改成功，false 表示失败。当 target 为不存在的对象时，会报错。

```js
let exam = {
    name: "Tom",
    age: 24,
    set info(value){
        return this.age = value;
    }
}
exam.age; // 24
Reflect.set(exam, 'age', 25); // true
exam.age; // 25
  
// value 为空时会将 name 属性清除
Reflect.set(exam, 'age', ); // true
exam.age; // undefined
  
// 当 target 对象中存在 name 属性 setter 方法时，setter 方法中的 this 会绑定 // receiver , 所以修改的实际上是 receiver 的属性,
let receiver = {
    age: 18
}
Reflect.set(exam, 'info', 1, receiver); // true
receiver.age; // 1
  
let receiver1 = {
    name: 'oppps'
}
Reflect.set(exam, 'info', 1, receiver1);
receiver1.age; // 1
```

### 1.15.3 Reflect.has(obj, name)

是 name in obj 指令的函数化，用于查找 name 属性在 obj 对象中是否存在。返回值为 boolean。如果 obj 不是对象则会报错 TypeError。

```js
let exam = {
    name: "Tom",
    age: 24
}
Reflect.has(exam, 'name'); // true
```



还有更多的操作方法，详情见：[Reflect 静态方法](https://es6.ruanyifeng.com/#docs/reflect#静态方法)



## 1.16 Promise 对象

Promise 是异步编程的一种解决方案。

从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。

### 1.16.1 Promise 状态

Promise 异步操作有三种状态：pending（进行中）、fulfilled（已成功）和 rejected（已失败）。除了异步操作的结果，任何其他操作都无法改变这个状态。

Promise 对象只有：从 pending 变为 fulfilled 和从 pending 变为 rejected 的状态改变。只要处于 fulfilled 和 rejected ，状态就不会再变了即 resolved（已定型）。

```js
const p1 = new Promise(function(resolve,reject){
    resolve('success1');
    resolve('success2');
});
const p2 = new Promise(function(resolve,reject){ 
    resolve('success3');
    reject('reject');
});
p1.then(function(value){ 
    console.log(value); // success1
});
p2.then(function(value){
    console.log(value); // success3
});
```

状态的缺点有：

- 无法取消 Promise ，一旦新建它就会立即执行，无法中途取消。
- 如果不设置回调函数，Promise 内部抛出的错误，不会反应到外部。
- 当处于 pending 状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。

### 1.16.2 then 方法

then 方法接收两个函数作为参数，第一个参数是 Promise 执行成功时的回调，第二个参数是 Promise 执行失败时的回调，两个函数只会有一个被调用。

在 JavaScript 事件队列的当前运行完成之前，回调函数永远不会被调用。

```js
const p = new Promise(function(resolve,reject){
  resolve('success');
});
  
p.then(function(value){
  console.log(value);
});
  
console.log('first');
// first
// success
```

then 方法将返回一个 resolved 或 rejected 状态的 Promise 对象用于链式调用，且 Promise 对象的值就是这个返回值。

```js
const p = new Promise(function(resolve,reject){
  resolve(1);
}).then(function(value){ // 第一个then // 1
  console.log(value);
  return value * 2;
}).then(function(value){ // 第二个then // 2
  console.log(value);
}).then(function(value){ // 第三个then // undefined
  console.log(value);
  return Promise.resolve('resolve');
}).then(function(value){ // 第四个then // resolve
  console.log(value);
  return Promise.reject('reject');
}).then(function(value){ // 第五个then //reject:reject
  console.log('resolve:' + value);
}, function(err) {
  console.log('reject:' + err);
});
```



## 1.17 Generator 函数

ES6 新引入了 Generator 函数，可以通过 yield 关键字，把函数的执行流挂起，为改变执行流程提供了可能，从而为异步编程提供解决方案。 

详情见：[Generator 函数的语法](https://es6.ruanyifeng.com/#docs/generator)



# 二、ES2016

## 2.1 Array.prototype.includes()

includes() 方法用来判断一个数组是否包含一个特定的值，如果包含则返回 true，否则返回 false

```js
const array1 = [1, 2, 3];
 
console.log(array1.includes(2));
// true
 
const pets = ['cat', 'dog', 'bat'];
 
console.log(pets.includes('cat'));
// true
 
console.log(pets.includes('at'));
// false
```

## 2.2 求幂运算符

新增了一个求幂运算符 **，等价于 Math.pow()

```js
1 ** 2; // 1
 
// 右结合，从右至左计算
2 ** 2 ** 3; // 256
 
let a = 2;
a **= 2;
// 等价于 a = a * a;
 
let b = 3;
b **= 3;
// 等价于 b = b * b * b;
```



# 三、ES2017

## 3.1 字符串补全

- **padStart**：返回新的字符串，表示用参数字符串从头部（左侧）补全原字符串。
- **padEnd**：返回新的字符串，表示用参数字符串从尾部（右侧）补全原字符串。

以上两个方法接受两个参数，第一个参数是指定生成的字符串的最小长度，第二个参数是用来补全的字符串。如果没有指定第二个参数，默认用空格填充。

```js
console.log("h".padStart(5,"o"));  // "ooooh"
console.log("h".padEnd(5,"o"));    // "hoooo"
console.log("h".padStart(5));      // "    h"
```

## 3.2 Object.values()

该方法返回一个包含所有对象自身属性值的数组

```js
const person = { name: 'Fred', age: 87 }
Object.values(person) // ['Fred', 87]
```

同样的，也适用于数组

```js
const people = ['Fred', 'Tony'] 
Object.values(people) // ['Fred', 'Tony']
```

## 3.3 Object.entries()

该方法返回一个包含所有对象自身属性的数组，作为[key, value] 对

```js
const person = { name: 'Fred', age: 87 } 
Object.entries(person) // [['name', 'Fred'], ['age', 87]]
```

同样的，也适用于数组

```js
const people = ['Fred', 'Tony'] 
Object.entries(people) // [['0', 'Fred'], ['1', 'Tony']]
```

## 3.4 Object.getOwnPropertyDescriptors()

此方法返回对象的所有自有（非继承的）属性描述符。

描述符是属性（property）的一组特性（attributes），它由以下的子集组成：

- **value**：属性的值
- **writable**：`true` 表示改属性可以被修改
- **get**：属性的 getter 函数，在读取属性时调用
- **set**：属性的 setter 函数，在属性设置值时调用
- **configurable**：如果为 `false` ，则不能删除属性，也不能更改任何属性，但值除外
- **enumerable**：如果属性是可枚举的，则为 `true`

Object.getOwnPropertyDescriptors() 接收一个对象，并返回一个带有描述符集的对象

## 3.5 函数参数列表中和函数调用中的尾随逗号

此功能允许在函数声明和函数调用中使用尾随逗号：

```js
const doSomething = (var1, var2,) => {
  //...
}
doSomething('test2', 'test2',)
```

## 3.6 Async 函数

引入了 Async Functions (异步函数) 的概念，这是 ECMAScript 版本中引入的最重要的变化。

## 3.7 共享内存（SharedArrayBuffer）和 Atomics

WebWorkers 用于在浏览器中创建多线程程序。

从ES2017开始，你可以使用 SharedArrayBuffer 在 Web worker 及其创建者之间创建共享内存数组。

详情见：[共享内存和 Atomics](https://www.html.cn/archives/7724)



# 四、ES2018

## 4.1 展开运算符（...）

展开语法, 可以在函数调用/数组构造时, 将数组表达式或者字符串在语法层面展开；还可以在构造字面量对象时, 将对象表达式按 key-value 的方式展开：

```js
function sum(x, y, z) {
  return x + y + z;
}
 
const numbers = [100, 200, 400];
 
console.log(sum(...numbers)); // expected output: 700
console.log({...numbers});  // expected ouput: Object { 0: 100, 1: 200, 2: 400 }
// expected output: 6
```



## 4.2 Asynchronous iteration（异步迭代）

for await…of 语句会在异步或者同步可迭代对象上创建一个迭代循环，包括 String，Array，Array-like 对象（比如arguments 或者NodeList)，TypedArray，Map， Set和自定义的异步或者同步可迭代对象。

其会调用自定义迭代钩子，并为每个不同属性的值执行语句。下面是迭代异步生成器的例子：

```js
async function* asyncGenerator() {
  var i = 0;
  while (i < 3) {
    yield i++;
  }
}
 
(async function() {
  for await (num of asyncGenerator()) {
    console.log(num);
  }
})();
// 0
// 1
// 2
```



## 4.3 Promise.prototype.finally

当一个 promise 得到满足（fulfilled）时，它会一个接一个的调用 then() 方法；如果在此期间发生错误，则跳过 then() 方法执行 catch() 方法

finally() 则允许你运行一些代码，不论 promise 执行成功还是失败

```js
fetch('file.json')
  .then(data => data.json())
  .catch(error => console.error(error))
  .finally(() => console.log('finished'))
```



## 4.4 正则表达式改进

### 4.4.1 lookahead（先行断言）和 lookbehind（后行断言）

先行断言(lookahead)使用 ?= 符号，是已有的功能。

下面是一个先行断言(lookahead)：你可以使用 ?= 匹配一个字符串，该字符串后面跟着一个特定的子字符串：

```js
/Roger(?=Waters)/
  
/Roger(?= Waters)/.test('Roger is my dog') //false
/Roger(?= Waters)/.test('Roger is my dog and Roger Waters is a famous musician') //true
```

?! 执行逆操作，匹配一个字符串，该字符串后面没有一个特定的子字符串：

```js
/Roger(?!Waters)/
  
/Roger(?! Waters)/.test('Roger is my dog') //true
/Roger(?! Waters)/.test('Roger Waters is a famous musician') //false
```

后行断言(lookbehind) 逆操作，是一个新功能，使用 ?< !

```js
/(?<!Roger) Waters/
  
/(?<!Roger) Waters/.test('Pink Waters is my dog') //true
/(?<!Roger) Waters/.test('Roger is my dog and Roger Waters is a famous musician') //false
```

### 4.4.2 Unicode 属性转义 \p{...} 和 \P{...}

在正则表达式模式中，您可以使用 \d 匹配任何数字，\s 匹配任何不为空格的字符，\w 匹配任何字母数字字符，依此类推。

这个新功能将扩展此概念到引入 \p{} 匹配所有 Unicode 字符，否定为 \P{} 。

### 4.4.3 命名捕获组

在 ES2018 中，可以为捕获组分配一个名称，而不是仅在结果数组中分配一个 slot（插槽）：

```js
const re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/
const result = re.exec('2015-01-02')
  
// result.groups.year === '2015';
// result.groups.month === '01';
// result.groups.day === '02';
```

### 4.4.4 正则表达式的 s 标志

s 标志是 ‘single line'(单行)的缩写，它使(.)匹配新的行字符。如果没有它，(.)将匹配普通字符，而不是新行：

```js
/hi.welcome/.test('hi\nwelcome') // false
/hi.welcome/s.test('hi\nwelcome') // true
```



# 五、ES2019

## 5.1 String.prototype.trimStart() 和 String.prototype.trimEnd()

在接收用户输入的文本时，我们经常使用 String.prototype.trim() 来去除首尾的空格、换行符等。

现在可以通过 trimStart() 和 trimEnd() 来进行首尾的单独去除。trimLeft() 和 trimRight() 是他们的别名。

```js
const string = ' Hello ES2021! ';

string.trimStart();
// 'Hello ES2021! '
string.trimEnd();
// ' Hello ES2021!'
```

## 5.2 Object.fromEntries()

ES2017 为我们引入了 Object.entries，可以把一个对象转为 [key, value] 键值对的形式，可以运用于像 Map 这种结构中。

ES2019 的 Object.fromEntries() 则用于把键值对还原成对象结构。

```js
const entries = [ ['foo', 'bar'] ];
const object = Object.fromEntries(entries);
// { foo: 'bar' }
```

## 5.3 Array.prototype.flat() 和 Array.prototype.flatMap()

这两个方法用于将数组展平，通过传入层级深度作为参数（默认为1），来为下层的数组提升层级：

```js
[1, 2, [3, 4]].flat();
// [ 1, 2, 3, 4 ]
[1, 2, [3, 4, [5, 6]]].flat(2);
// [ 1, 2, 3, 4, 5, 6 ]
```

Array.prototype.flatMap() 是 Array.prototype.map() 和 Array.prototype.flat() 的组合，通过对 map 调整后的数据尝试展平操作：

```js
[1, 2, [3, 4]].flatMap(v => {
  if (typeof v === 'number') {
    return v * 2
  } else {
    return v.map(v => v * 2)
  }
})
// [2, 4, 6, 8]
```

## 5.4 catch 的参数改为可选

之前，在进行 try...catch 错误处理过程中，如果没有给 catch 传参数的话，代码就会报错。

而有时候我们并不关心错误情况，如：

```js
const isValidJSON = json => {
  try {
    JSON.parse(json);
    return true;
  } catch (unusedError) {
    // Unused error parameter
    return false;
  }
};
```

在 ES2019 后，我们则可以省略 catch 绑定的参数和括号：

```js
const isValidJSON = json => {
  try {
    JSON.parse(json);
    return true;
  } catch {
    return false;
  }
};
```

## 5.5 Symbol.prototype.description

Symbol 是 ES6 中引入的基本数据类型，可以用作对象属性的标识符。

描述属性是只读的，可用于获取符号对象的描述，更好了解它的作用。例如：

```js
const symbol = Symbol('This is a Symbol');
symbol;
// Symbol(This is a Symbol)
Symbol.description;
// 'This is a Symbol'
```

## 5.6 JSON superset（超集）

在之前，如果JSON字符串中包含有行分隔符(\u2028) 和段落分隔符(\u2029)，那么在解析过程中会报错：

```js
JSON.parse('"\u2028"'); 
// SyntaxError
```

现在ES2019对它们提供了支持：

```js
JSON.parse('"\u2028"'); 
// ''
```

## 5.7 JSON.stringify

一些表情的字符，例如'\uD83D'，会被认为这是一个无效的文本字符串。在早期版本中，这些字符将替换为特殊字符：

```js
JSON.stringify('\uD83D'); 
// '"�"'
```

现在在字符代码之前插入转义字符，结果仍是可读且有效的UTF-8/UTF-16代码：

```js
JSON.stringify('\uD83D'); 
// '"\\ud83d"'
```

## 5.8 Array.prototype.sort() 变得更加稳定

在之前，规范中不允许不稳定的排序算法，例如快速排序：

```js
let array = [
  {a: 1, b: 2},
  {a: 2, b: 2},
  {a: 1, b: 3},
  {a: 2, b: 4},
  {a: 5, b: 3}
];
array.sort((a, b) => a.a - b.a);
// [{a: 1, b: 2}, {a: 1, b: 3}...] / [{a: 1, b: 3}, {a: 1, b: 2}...]
```

现在所有主流浏览器都使用稳定的排序算法。实际上，这意味着如果我们有一个对象数组，并在给定的键上对它们进行排序，那么列表中的元素将保持相对于具有相同键的其他对象的位置。

## 5.9 Function.prototype.toString() 的重新修订

在之前，Function.prototype.toString()只会返回函数的主体，但没有注释和空格。

从 ES2019 开始，Function.prototype.toString() 将从头到尾返回源代码中的实际文本片段。这意味着还将返回注释、空格和语法详细信息。

```js
foo.toString();
// 'function /* a comment  */ foo () {}'
```



# 六、ES2020

## 6.1 class 的私有变量

通过 # 可以给 class 添加私有变量：

```js
class Counter {
  #number = 10
  increment() {
    this.#number++
  }
  getNum() {
    return this.#number
  }
}
const counter = new Counter()
counter.increment();
 
console.log(counter.getNum())    //11
console.log(counter.#number)    //调用class的私有变量会报错：SyntaxError
```



## 6.2 Promise.allSettled

当我们处理多个 Promise 时，特别是当它们相互依赖时，记录每个 Promise 所发生的事情来调试错误是很有必要的。

通过 Promise.allSettled，我们可以创建一个新的 Promise，它只在所有传递给它的 Promise 都完成时返回一个数组，其中包含每个 Promise 的数据。

```js
const p1 = new Promise((res, rej) => setTimeout(res, 1000));
 
const p2 = new Promise((res, rej) => setTimeout(rej, 1000));
 
Promise.allSettled([p1, p2]).then(data => console.log(data));
 
// [
//   Object { status: "fulfilled", value: undefined},
//   Object { status: "rejected", reason: undefined}
// ]
```



## 6.3 空位合并运算符（.?）

由于 JavaScript 是动态类型的，因此在分配变量时，需要牢记 JavaScript 对真实/错误值的处理。 如果我们有一个带有某些值的对象，有时我们希望允许使用从技术上讲是虚假的值，例如空字符串或数字 0。设置默认值不是一个非常合理的方法，因为默认值会覆盖应为有效值的值。

通常情况下，我们使用 ||

```js
let person = {
  profile: {
    name: "",
    age: 0
  }
};
 
console.log(person.profile.name || "Anonymous"); // Anonymous
console.log(person.profile.age || 18); // 18
```

现在我们可以使用 ?? 操作符，使其类型更严格一些，这只允许在值为 null 或 undefined 时使用默认值。

```js
let person = {
  profile: {
    name: "",
    age: 0
  }
};
 
console.log(person.profile.name ?? "Anonymous"); // ""
console.log(person.profile.age ?? 18); // 0
```



## 6.4 可选链（.?）

与合并空操作符类似，JavaScript 在处理错误值时可能无法按我们希望的方式工作。如果我们想要的是未定义的，我们可以返回一个值，但是如果到它的路径是未定义的呢？通过在.之前添加一个问号，我们可以使值的路径的任何部分成为可选的，这样我们仍然可以与它交互。

```js
let person = {};
 
console.log(person.profile.name ?? "Anonymous"); // person.profile is undefined
console.log(person?.profile?.name ?? "Anonymous");
console.log(person?.profile?.age ?? 18);
```



## 6.5 BigInt

JavaScript 可以处理的最大安全整数是(2^53 - 1)，我们可以在 MAX_SAFE_INTEGER 中看到。

但是，最大的整数有时候可能会不够用。因此 ES2020 引入了 BigInt 这个标准内置对象，可以表示任意大的整数。

```js
const theBiggestInt = 9007199254740991n;
const alsoHuge = BigInt(9007199254740991); // 9007199254740991n
const hugeString = BigInt("9007199254740991"); // 9007199254740991n
const hugeHex = BigInt("0x1fffffffffffff"); // 9007199254740991n
const hugeBin = BigInt("0b11111111111111111111111111111111111111111111111111111"); // 9007199254740991n
```

BigInt 在某些方面类似于 Number ，但是也有几个关键的不同点：

- 不能用于 Math 对象中的方法；
- 不能和任何 Number 实例混合运算，两者必须转换成同一种类型。在两种类型来回转换时要小心，因为 BigInt 变量在转换成 Number 变量时可能会丢失精度。



## 6.6 动态 import

关键字 import 可以像调用函数一样来动态的导入模块。以这种方式调用，将返回一个 promise。

```js
import('/modules/my-module.js')
  .then((module) => {
    // Do something with the module.
  });
```

这种使用方式也支持 await 关键字。

```js
let module = await import('/modules/my-module.js');
```



## 6.7 globalThis

JavaScript 语言存在一个顶层对象，它提供全局环境（即全局作用域），所有代码都是在这个环境中运行。但是，顶层对象在各种实现里面是不统一的：

- 浏览器里面，顶层对象是 window，但 Node 和 Web Worker 没有window。
- 浏览器和 Web Worker 里面，self 也指向顶层对象，但是 Node 没有 self。
- Node 里面，顶层对象是 global，但其他环境都不支持。

ES2020 的 globalThis 对象则始终会引用着全局对象，而不用关心代码在哪运行。



## 6.8 String.prototype.matchAll()

matchAll() 方法返回一个包含所有匹配正则表达式的结果及分组捕获组的迭代器。可以使用一个扩展运算符（...），把迭代器转换为数组。

```js
let regexp = /t(e)(st(\d?))/g;
let str = 'test1test2';
let array = [...str.matchAll(regexp)];
console.log(array[0]);
console.log(array[1]);
```



## 6.9 for-in mechanics

以前，不同引擎下 for-in 循环出来的内容顺序可能是不一样的。

ES2020 后，顺序进行了标准化



## 6.10 import.meta

开发者使用一个模块时，有时需要知道模板本身的一些信息（比如模块的路径）。

ES2020 为 import 命令添加了一个元属性 import.meta，返回当前模块的元信息。

import.meta 只能在模块内部使用，如果在模块外部使用会报错。这个属性返回一个对象，该对象的各种属性就是当前运行的脚本的元信息。具体包含哪些属性，标准没有规定，由各个运行环境自行决定。

一般来说，import.meta 至少会有下面两个属性：

- import.meta.url

import.meta.url 返回当前模块的 URL 路径。举例来说，如果当前模块主文件的路径是 https://foo.com/main.js，import.meta.url 就返回这个路径。如果模块里面还有一个数据文件 data.txt，那么就可以用下面的代码，获取这个数据文件的路径：

```js
new URL('data.txt', import.meta.url)
```

- import.meta.scriptElement

import.meta.scriptElement 是浏览器特有的元属性，返回加载模块的那个 <script> 元素，相当于 document.currentScript 属性。

```js
// HTML 代码为
// <script type="module" src="my-module.js" data-foo="abc"></script>
 
// my-module.js 内部执行下面的代码
import.meta.scriptElement.dataset.foo
// "abc"
```



# 七、ES2021

## 7.1 String.prototype.replaceAll()

字符串的实例方法 replace() 只能替换第一个匹配，如果要替换所有的匹配，则可以使用 ES2021 的 replaceAll() 方法，可以一次性替换所有匹配：

```js
'aabbcc'.replaceAll('b', '_') 
// 'aa__cc'
```

它的用法与 replace() 相同，返回一个新字符串，不会改变原字符串。



## 7.2 Promise.any()

Promise.any() 方法，该方法接收一组 Promise 实例作为参数，包装成一个新的 Promise 实例返回。只要参数实例有一个变成 fulfilled 状态，包装实例就会变成 fulfilled 状态；如果所有参数实例都变成 rejected 状态，包装实例就会变成 rejected 状态。

Promise.any() 跟 Promise.race() 方法很像，只有一点不同，就是不会因为某个 Promise 变成 rejected 状态而结束。

```js
const promises = [
  fetch('/endpoint-a').then(() => 'a'),
  fetch('/endpoint-b').then(() => 'b'),
  fetch('/endpoint-c').then(() => 'c'),
];
try {
  const first = await Promise.any(promises);
  console.log(first);
} catch (error) {
  console.log(error);
}
 
// 上面代码中，Promise.any()方法的参数数组包含三个 Promise 操作。
// 其中只要有一个变成fulfilled，Promise.any()返回的 Promise 对象就变成fulfilled。
// 如果所有三个操作都变成rejected，那么await命令就会抛出错误。
```

Promise.any() 抛出的错误，不是一个一般的错误，而是一个 AggregateError 实例。它相当于一个数组，每个成员对应一个被 rejected 的操作所抛出的错误。下面是 AggregateError 的实现示例：

```js
new AggregateError() extends Array -> AggregateError
 
const err = new AggregateError();
err.push(new Error("first error"));
err.push(new Error("second error"));
throw err;
```



## 7.3 逻辑赋值操作符（Logical Assignment Operators）

在 JavaScript 中，有许多赋值表达式和逻辑操作符，在 ES2021 新提案里，我们将有能力结合逻辑操作符和赋值表达式。下面是一些 &&、 || 和 ?? 的例子。

### 7.3.1 && 逻辑赋值操作符

该操作符用来在仅当左侧变量为真值（truthy）时，才将右侧变量赋值给左侧变量。

```js
// `&&` 逻辑赋值操作符
let num1 = 5
let num2 = 10
  
num1 &&= num2
  
console.log(num1) // 10
  
// 行 5 也可写作以下方式
// 1. num1 && (num1 = num2)
// 2. if (num1) num1 = num2
```

### 7.3.2 || 逻辑赋值操作符

该操作符用来在仅当左侧变量为虚值（falsy）时，才将右侧变量赋值给左侧变量。

```js
// `||` 逻辑赋值操作符
let num1
let num2 = 10
  
num1 ||= num2
  
console.log(num1) // 10
  
// 行 5 也可写作以下方式
// 1. num1 || (num1 = num2)
// 2. if (!num1) num1 = num2
```

### 7.3.3 ?? 逻辑赋值操作符

ES2020 已经引入了空值合并操作符（Nullish Coalescing operator，即 ??），该操作符亦可与赋值表达式结合。在仅当左侧变量为 undefined 或 null 时，该操作符才将右侧变量赋值给左侧变量。

```js
// `??` 逻辑赋值操作符
let num1
let num2 = 10
  
num1 ??= num2
console.log(num1) // 10
  
num1 = false
num1 ??= num2
console.log(num1) // false
  
// 行 5 也可写作以下方式
// num1 ?? (num1 = num2)
```



## 7.4 数值分隔符（Numeric separators）

数值分隔符的引入将使我们可以通过使用 _（下划线）符号在数字分组间提供一个隔离以便于阅读数值（而不改变实际数值）。例如：

```js
let number = 100_000
  
console.log(number)
  
/****  输出  ****/
// 100000
```



## 7.5 WeakRef

WeakRef 对象允许保留对另一个对象的弱引用，而不会阻止被弱引用对象被GC回收



# 八、参考网站

[ES6 入门教程](https://es6.ruanyifeng.com/)

[tc39 proposals](https://github.com/tc39/proposals/blob/master/finished-proposals.md)

[菜鸟教程 ES6](https://www.runoob.com/w3cnote/es6-tutorial.html)

[ES2020-ES2015](https://blog.csdn.net/qq_40511157/article/details/105596686)

