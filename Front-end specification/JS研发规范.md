# 1. 变量类型及声明

## 1.1 变量类型

#### 1.1.1 基本类型

- `string`
- `number`
- `boolean`
- `null`
- `undefined`
- `symbol`
- `bigint`



`Symbol`和`BigInt`不能被正确的polyfill。 所以在不能原生支持上述两种类型的环境[浏览器]中，不应该使用。

#### 1.1.2 复杂类型

- `object`
- `array`
- `function`

## 1.2 变量声明

#### 1.2.1 使用`const`赋值，避免使用`var`

> 这样可以确保你不会改变你的初始值，因为重复引用会导致许多bug和代码难以理解的问题。

```javascript
// bad
var a = 1
var b = 2

// good
const a = 1
const b = 2
```

#### 1.2.2 如果需要对参数重新赋值，使用`let`，不使用`var`

```javascript
// bad
var count = 1
if (true) {
  count += 1
}

// good
let count = 1
if (true) {
  count += 1
}
```

#### 1.2.3 每个变量都用一个 `const` 或 `let `

```javascript
// bad
const items = getItems(),
    goSportsTeam = true,
    dragonball = 'z'

// good
const items = getItems()
const goSportsTeam = true
const dragonball = 'z'
```

#### 1.2.4 `const`放一起，`let`放一起

```javascript
// bad
let i
const items = getItems()
let dragonball
const goSportsTeam = true
let len

// good
const goSportsTeam = true
const items = getItems()
let dragonball
let i
let length
```

#### 1.2.5 在需要的地方再声明变量，但是要放在合理的位置

```javascript
// bad
function checkName(hasName) {
  const name = getName()

  if (hasName === 'test') {
    return false
  }

  if (name === 'test') {
    return false
  }

  return name
}

// good
function checkName(hasName) {
  if (hasName === 'test') {
    return false
  }

  // 在需要的时候分配
  const name = getName()

  if (name === 'test') {
    return false
  }

  return name
}
```

#### 1.2.6 在赋值的时候避免在 `=` 前/后换行， 如果你的赋值语句超出最大长度， 那就用小括号把这个值包起来再换行

```javascript
// bad
const foo =
  superLongLongLongLongLongLongLongLongFunctionName()

// bad
const foo
  = 'superLongLongLongLongLongLongLongLongString'

// good
const foo = (
  superLongLongLongLongLongLongLongLongFunctionName()
)

// good
const foo = 'superLongLongLongLongLongLongLongLongString'
```

#### 1.2.7 不允许有定义后未使用的变量，或定义后未使用的参数



# 2. 类型基本规范

## 2.1 Object

#### 2.1.1 创建对象时，使用字面量创建方法

```javascript
// bad
const item = new Object()

// good
const item = {}
```

#### 2.1.2 单行定义的对象，最后一个成员不以逗号结尾；多行定义的对象，最后一个成员以逗号结尾

```javascript
// bad
const a = { k1: v1, k2: v2, }
const b = {
  k1: v1,
  k2: v2
}

// good
const a = { k1: v1, k2: v2 }
const b = {
  k1: v1,
  k2: v2,
}
```

#### 2.1.3 创建一个带有动态属性名的对象时，使用属性表达式定义

> 对象`obj`的最后一个属性名，需要通过计算得到。这时最好采用属性表达式，在新建`obj`的时候，将该属性与其他属性定义在一起。这样一来，所有属性就都在一个地方定义了。

```javascript
function getKey(k) {
  return `a key named ${k}`
}

// bad
const obj = {
  id: 5,
  name: 'San Francisco',
}
obj[getKey('attName')] = true

// good
const obj = {
  id: 5,
  name: 'San Francisco',
  [getKey('attName')]: true,
}
```

#### 2.1.4 创建对象方法时，使用简写

```javascript
// bad
const atom = {
  value: 1,

  addValue: function (value) {
    return atom.value + value;
  }
}

// good
const atom = {
  value: 1,

  // 对象的方法
  addValue(value) {
    return atom.value + value;
  }
}
```

#### 2.1.5 写对象属性值时，使用缩写

```javascript
const lukeSkywalker = 'Luke Skywalker';

// bad
const obj = {
  lukeSkywalker: lukeSkywalker
}

// good
const obj = {
  lukeSkywalker
}
```

#### 2.1.6 将所有的属性值缩写放在对象声明的开始

```javascript
const anakinSkywalker = 'Anakin Skywalker'
const lukeSkywalker = 'Luke Skywalker'

// bad
const obj = {
  episodeOne: 1,
  twoJediWalkIntoACantina: 2,
  lukeSkywalker,
  episodeThree: 3,
  mayTheFourth: 4,
  anakinSkywalker
}

// good
const obj = {
  lukeSkywalker,
  anakinSkywalker,
  episodeOne: 1,
  twoJediWalkIntoACantina: 2,
  episodeThree: 3,
  mayTheFourth: 4
}
```

#### 2.1.7 key值中，只对那些无效的标示使用引号 `''`

> 这种方式主观上易读，并且优化了代码高亮。

```javascript
// bad
const bad = {
  'foo': 3,
  'bar': 4,
  'data-blah': 5
}

// good
const good = {
  foo: 3,
  bar: 4,
  'data-blah': 5
}
```

#### 2.1.8 对象尽量静态化，一旦定义，就不得随意添加新的属性。如果添加属性不可避免，则要使用`Object.assign`方法

```javascript
// bad
const a = { x: 3 }
a.y = 4

// good
const a = { x: 3 }
Object.assign(a, { y: 4 })
```

#### 2.1.9 合并多个对象或对象浅拷贝时，若不返回新对象，使用`Object.assign()`；若要返回新对象，使用扩展运算符（`...`）

```javascript
// 返回初始对象
const original = { a: 1, b: 2 }
Object.assign(original, { c: 3 })	// original { a: 1, b: 2, c: 3 }

// 返回新对象
const original = { a: 1, b: 2 }
const newOriginal = { ...original, ...{ c: 3 } } // newOriginal { a: 1, b: 2, c: 3 }
```

#### 2.1.10 获取对象指定的某几个属性时，使用对象的rest解构运算符（`...`）

```javascript
const original = { a: 1, b: 2, c: 3 }

// rest赋值运算符
const { a, ...rest } = original // rest => { b: 2, c: 3 }
```

#### 2.1.11 所有配置项都应该集中在一个对象，放在最后一个参数，布尔值不可以直接作为参数

```javascript
// bad
function divide(a, b, option = false ) {
}

// good
function divide(a, b, { option = false } = {}) {
}
```



## 2.2 Array

#### 2.2.1 创建数组时，使用字面量创建方法

```javascript
// bad
const items = new Array()

// good
const items = []
```

#### 2.2.2 向数组中添加值时，使用`Array.push()`

```javascript
const someStack = []

// bad
someStack[someStack.length] = 'abc'

// good
someStack.push('abc')
```

#### 2.2.3 拷贝数组时，使用扩展运算符（`...`）

```javascript
// bad
const len = items.length
const itemsCopy = []
for (let i = 0; i < len; i++) {
  itemsCopy[i] = items[i]
}

// good
const itemsCopy = [...items]
```

#### 2.2.4 将可迭代的对象（实现了Iterator接口）转为数组时，使用扩展运算符（`...`）

```javascript
// arguments对象
function foo() {
  const args = [...arguments]
}
```

#### 2.2.5 将类数组对象转为数组时，使用`Array.from`方法

```javascript
const foo = document.querySelectorAll('.foo')
const arrLike = { 0: 'foo', 1: 'bar', 2: 'baz', length: 3 }

const nodes = Array.from(foo)
const arr = Array.from(arrLike)
```

#### 2.2.6 需要转换成数组去做map遍历时， 使用 `Array.from`而不是 `...` 运算符

> 因为这样可以避免创建一个临时数组。

```javascript
// bad
const baz = [...foo].map(bar)

// good
const baz = Array.from(foo, bar)
```

#### 2.2.7 在数组方法的回调函数中使用return语句

> 如果函数体由一条返回一个表达式的语句组成， 并且这个表达式没有副作用， 这个时候可以忽略return。

```javascript
// good 函数有多条语句
[1, 2, 3].map((x) => {
  const y = x + 1
  return x * y
});


// good 函数只有一个语句
[1, 2, 3].map(x => x + 1);


// bad 没有返回值，在第一次迭代后acc就变成undefined了
[[0, 1], [2, 3], [4, 5]].reduce((acc, item, index) => {
  const flatten = acc.concat(item)
  acc[index] = flatten
})

// good
[[0, 1], [2, 3], [4, 5]].reduce((acc, item, index) => {
  const flatten = acc.concat(item)
  acc[index] = flatten
  return flatten
})


// bad
inbox.filter((msg) => {
  const { subject, author } = msg
  if (subject === 'Mockingbird') {
    return author === 'Harper Lee'
  } else {
    return false
  }
})

// good 这样写简便
inbox.filter((msg) => {
  const { subject, author } = msg;
  if (subject === 'Mockingbird') {
    return author === 'Harper Lee'
  }

  return false
})
```

#### 2.2.8 如果一个数组有很多行，在数组的 `[` 后和 `]` 前断行

```javascript
// bad
const arr = [
  [0, 1], [2, 3], [4, 5],
]

const objectInArray = [{
  id: 1,
}, {
  id: 2,
}]

const numberInArray = [
  1, 2,
]

// good
const arr = [[0, 1], [2, 3], [4, 5]]

const objectInArray = [
  {
    id: 1,
  },
  {
    id: 2,
  },
]

const numberInArray = [
  1,
  2,
]
```



## 2.3 String

#### 2.3.1 静态字符串一律使用单引号，不使用双引号，动态字符串使用反引号

```javascript
// bad
const a = "foobar"
const b = 'foo' + a + 'bar'

// good
const a = 'foobar'
const b = `foo${a}bar`
```

#### 2.3.2 超过100个字符的字符串不应该用string串联成多行

> 被折断的字符串看起来是糟糕的，而且使得代码更不易被搜索。

```javascript
// bad
const errorMessage = 'This is a super long error that was thrown because \
of Batman. When you stop to think about how Batman had anything to do \
with this, you would get nowhere \
fast.'

// bad
const errorMessage = 'This is a super long error that was thrown because ' +
  'of Batman. When you stop to think about how Batman had anything to do ' +
  'with this, you would get nowhere fast.'

// good
const errorMessage = 'This is a super long error that was thrown because of Batman. When you stop to think about how Batman had anything to do with this, you would get nowhere fast.'
```

#### 2.3.3 不要在字符串中使用`eval()`

#### 2.3.4 不要使用不必要的转义字符

> 反斜线的可读性差，所以只在必须使用时才使用

```javascript
// bad
const foo = '\'this\' \i\s \"quoted\"'

// good
const foo = '\'this\' is "quoted"'

//best
const foo = `my name is '${name}'`
```



## 2.4 Function

#### 2.4.1 使用默认值语法设置函数参数的默认值，并将默认参数赋值放在最后

```javascript
// bad
function handleThings(opts) {
  opts = opts || {}
}

// good
function handleThings(opts = {}) {
  // ...
}
```

#### 2.4.2 不使用`arguments`变量，使用rest运算符（`...`）代替

> 因为rest运算符显式表明你想要获取参数，而且`arguments`是一个类似数组的对象，而rest运算符可以提供一个真正的数组。

```javascript
// bad
function concatenateAll() {
  const args = Array.prototype.slice.call(arguments);
  return args.join('')
}

// good
function concatenateAll(...args) {
  return args.join('')
}
```

#### 2.4.3 在`if`或`for`等块语句中，使用函数表达式，而不是函数声明

> 在ECMA-262中 [块 `block`] 的定义是： 一系列的语句。但是函数声明不是一个语句，函数表达式才是一个语句。

```javascript
// bad
if (currentUser) {
  function test() {
    console.log('Nope.')
  }
}

// good
let test
if (currentUser) {
  test = () => {
    console.log('Yup.')
  }
}
```

#### 2.4.4 不要用`arguments`命名参数，他的优先级高于每个函数作用域自带的 `arguments` 对象， 这会导致函数自带的 `arguments` 值被覆盖

```javascript
// bad
function foo(name, options, arguments) {
  // ...
}

// good
function foo(name, options, args) {
  // ...
}
```

#### 2.4.5 函数签名部分要有空格

```javascript
// bad
const f = function(){}
const g = function (){}
const h = function() {}

// good
const x = function () {}
const y = function a() {}
```

#### 2.4.6 不要修改参数

>  不要修改参数对象的数据结构，保留参数原始值和数据结构。

```javascript
// bad
function f1(obj) {
  obj.key = 1
}

// good
function f2(obj) {
  const key = obj.hasOwnProperty('key') ? obj.key : 1
}
```

#### 2.4.7 不要对参数重新赋值

```javascript
// bad
function f1(a) {
  a = 1
  // ...
}

function f2(a) {
  if (!a) { a = 1; }
  // ...
}

// good
function f3(a) {
  const b = a || 1
  // ...
}

function f4(a = 1) {
  // ...
}
```



## 2.5 箭头函数

#### 2.5.1 使用匿名函数当作参数的场合，尽量使用箭头函数代替

> 这样更简洁，而且绑定了外部作用域的 this。

```javascript
// bad
[1, 2, 3].map(function (x) {
  return x * x
})

// good
[1, 2, 3].map((x) => {
  return x * x
})

// best
[1, 2, 3].map(x => x * x)
```

#### 2.5.2 当表达式涉及多行时，将其包裹在圆括号中

> 可读性更强。

```javascript
// bad
['get', 'post', 'put'].map(httpMethod => Object.prototype.hasOwnProperty.call(
    httpMagicObjectWithAVeryLongName,
    httpMethod
  )
)

// good
['get', 'post', 'put'].map(httpMethod => (
  Object.prototype.hasOwnProperty.call(
    httpMagicObjectWithAVeryLongName,
    httpMethod
  )
))
```

#### 2.5.3 立即执行函数写成箭头函数的形式，并将其整个包裹在圆括号里

> 简单的、单行的、不会复用的函数，建议采用箭头函数。如果函数体较为复杂，行数较多，还是应该采用传统的函数写法。

```javascript
(() => {
  console.log('Welcome')
}())
```

#### 2.5.4 使用箭头函数取代`Function.prototype.bind`，不再使用 self/_this/that 来绑定this

```javascript
// bad
const self = this
const boundMethod = function(...params) {
  return method.apply(self, params)
}

// good
const boundMethod = (...params) => method.apply(this, params)
```

#### 2.5.5 当你的函数只有一个参数并且函数体没有大括号时，就删除圆括号，否则参数总是放在圆括号里

```javascript
// bad
[1, 2, 3].map((x) => x * x)

// good
[1, 2, 3].map(x => x * x)

// good
[1, 2, 3].map(number => (
  `A long string with the ${number}. It’s so long that we don’t want it to take up space on the .map line!`
))

// bad
[1, 2, 3].map(x => {
  const y = x + 1;
  return x * y;
})

// good
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
})
```

#### 2.5.6 避免箭头函数(`=>`)和比较操作符（`<=, >=`）混淆

```javascript
// bad
const itemHeight = (item) => item.height <= 256 ? item.largeSize : item.smallSize

// bad
const itemHeight = (item) => item.height >= 256 ? item.largeSize : item.smallSize

// good
const itemHeight = (item) => (item.height <= 256 ? item.largeSize : item.smallSize)

// good
const itemHeight = (item) => {
  const { height, largeSize, smallSize } = item
  return height <= 256 ? largeSize : smallSize
}
```

#### 2.5.7 在隐式return中强制约束函数体的位置，就写在箭头后面

```javascript
// bad
(foo) =>
  bar;

(foo) =>
  (bar);

// good
(foo) => bar;
(foo) => (bar);
(foo) => (
   bar
)
```



## 2.6 Module

#### 2.6.1 使用`import`取代`require`

> Module 语法是 JavaScript 模块的标准写法。

```javascript
// bad
const moduleA = require('moduleA')
const func1 = moduleA.func1
const func2 = moduleA.func2

// good
import { func1, func2 } from 'moduleA'
```

#### 2.6.2 一个路径只`import`一次

```javascript
// bad
import foo from 'foo'
// … some other imports … //
import { named1, named2 } from 'foo'

// good
import foo, { named1, named2 } from 'foo'

// good
import foo, {
  named1,
  named2,
} from 'foo'
```

#### 2.6.3 使用`export`取代`module.exports`

```javascript
// bad
module.exports = Breadcrumbs

// good
export default Breadcrumbs
```

#### 2.6.4 不要在模块输入中使用通配符`*`

> 因为这样可以确保你的模块之中，有一个默认输出（export default）。

```javascript
// bad
import * as myObject from './importModule'

// good
import myObject from './importModule'
```

#### 2.6.5 如果模块默认输出一个函数，函数名的首字母应该小写

```javascript
function makeStyleGuide() {
}

export default makeStyleGuide
```

#### 2.6.6 如果模块默认输出一个对象，对象名的首字母应该大写

```javascript
const StyleGuide = {
  es6: {
  }
}

export default StyleGuide
```

#### 2.6.7 不要导出可变的量

> 变化通常都是需要避免的，特别是当你要输出可变的绑定。总的来说，我们应该导出常量。

```javascript
// bad
let foo = 3
export { foo }

// good
const foo = 3
export { foo }
```

#### 2.6.8 在一个单一导出模块里，用 `export default` 更好

> 鼓励使用更多文件，每个文件只做一件事情并导出，这样可读性和可维护性更好。

```javascript
// bad
export function foo() {}

// good
export default function foo() {}
```

#### 2.6.9 `import` 放在其他所有语句之前

```javascript
// bad
import foo from 'foo'
foo.init()

import bar from 'bar'

// good
import foo from 'foo'
import bar from 'bar'

foo.init()
```



## 2.7 解构

#### 2.7.1 函数的参数如果是对象的成员，优先使用解构赋值

```javascript
// bad
function getFullName(obj) {
  const firstName = obj.firstName;
  const lastName = obj.lastName;
}

// good
function getFullName(obj) {
  const { firstName, lastName } = obj;
}

// best
function getFullName({ firstName, lastName }) {
}
```

#### 2.7.2 使用数组成员对变量赋值时，优先使用解构赋值（数组长度不大时）

```javascript
const arr = [1, 2, 3, 4]

// bad
const first = arr[0]
const second = arr[1]

// good
const [first, second] = arr
```

#### 2.7.3 如果函数返回多个值，优先使用对象的解构赋值，而不是数组的解构赋值

> 这样便于以后添加返回值，以及更改返回值的顺序。

```javascript
// bad
function processInput(input) {
  return [left, right, top, bottom]
}

// good
function processInput(input) {
  return { left, right, top, bottom }
}

const { left, right } = processInput(input)
```

#### 2.7.4 两数交换值时，使用解构

```javascript
let a1 = 1
let a2 = 2;

[a1, a2] = [a2, a1]
```



## 2.8 Class

#### 2.8.1 总是用`Class`，取代需要`prototype`的操作。并且使用`extends`实现继承

> 因为 Class 的写法更简洁，更易于理解。

```javascript
// bad
function Queue(contents = []) {
  this._queue = [...contents]
}
Queue.prototype.pop = function() {
  const value = this._queue[0];
  this._queue.splice(0, 1)
  return value
}

// good
class Queue {
  constructor(contents = []) {
    this._queue = [...contents]
  }
  pop() {
    const value = this._queue[0]
    this._queue.splice(0, 1)
    return value
  }
}
```

#### 2.8.2 除非外部库或框架需要使用特定的非静态方法，否则类方法应该使用`this`或被做成静态方法， 作为一个实例方法应该表明它根据接收者的属性有不同的行为

```javascript
// bad
class Foo {
  bar() {
    console.log('bar');
  }
}

// good - this 被使用了
class Foo {
  bar() {
    console.log(this.bar);
  }
}

// good - constructor 不一定要使用this
class Foo {
  constructor() {
    // ...
  }
}

// good - 静态方法不需要使用 this
class Foo {
  static bar() {
    console.log('bar');
  }
}
```



## 2.9 Iterators & Generators

#### 2.9.1 不要用遍历器，尽量用JavaScript高级函数代替`for-in`、 `for-of`

>  用数组的这些迭代方法： `map()` / `every()` / `filter()` / `find()` / `findIndex()` / `reduce()` / `some()` / ... , 用对象的这些方法 `Object.keys()` / `Object.values()` / `Object.entries()` 去产生一个数组， 这样就能去遍历对象了。

```javascript
const numbers = [1, 2, 3, 4, 5]

// bad
let sum = 0
for (let num of numbers) {
  sum += num
}
sum === 15

// good
let sum = 0
numbers.forEach(num => sum += num)
sum === 15

// best (use the functional force)
const sum = numbers.reduce((total, num) => total + num, 0);
sum === 15

// bad
const increasedByOne = []
for (let i = 0; i < numbers.length; i++) {
  increasedByOne.push(numbers[i] + 1)
}

// good
const increasedByOne = []
numbers.forEach(num => increasedByOne.push(num + 1))

// best (keeping it functional)
const increasedByOne = numbers.map(num => num + 1)
```

#### 2.9.2 不要使用generator



## 2.10 属性

#### 2.10.1 访问属性时使用点符号`.`

```javascript
const luke = {
  jedi: true,
  age: 28,
}

// bad
const isJedi = luke['jedi']

// good
const isJedi = luke.jedi
```

#### 2.10.2 当获取的属性是变量时用方括号`[]`取

```javascript
const luke = {
  jedi: true,
  age: 28,
}

function getProp(prop) {
  return luke[prop]
}

const isJedi = getProp('jedi')
```

#### 2.10.3 做幂运算时用幂操作符 `**` 

```javascript
// bad
const binary = Math.pow(2, 10)

// good
const binary = 2 ** 10
```



## 2.11 操作符

#### 2.11.1 除以下情况外，句尾不加分号

- 当一行代码是以` ( `开头的时候，则在前面补上一个分号用以避免一些语法解析错误。

```javascript
function say() {
  console.log('hello world')
}

;say()

;(function () {
  console.log('hello')
})()
```

- 当一行代码是以` [ `开头的时候，则在前面补上一个分号用以避免一些语法解析错误。

```javascript
;['apple', 'banana'].forEach(function (item) {
	console.log(item)
 })
```

- 当一行代码是以`开头的时候，则在前面补上一个分号用以避免一些语法解析错误。

```javascript
;`hello`.toString()
```

#### 2.11.2 在 case 和 default 分句里用大括号创建一块包含语法声明的区域

```javascript
// bad
switch (foo) {
  case 1:
    let x = 1
    break
  case 2:
    const y = 2
    break
  case 3:
    function f() {
      // ...
    }
    break
  default:
    class C {}
}
 
// good
switch (foo) {
  case 1: {
    let x = 1
    break
  }
  case 2: {
    const y = 2
    break
  }
  case 3: {
    function f() {
      // ...
    }
    break
  }
  case 4:
    bar()
    break
  default: {
    class C {}
  }
}
```

#### 2.11.3  三元表达式不应该嵌套，通常是单行表达式

```javascript
// bad
const foo = maybe1 > maybe2
  ? "bar"
  : value1 > value2 ? "baz" : null
 
// better
const maybeNull = value1 > value2 ? 'baz' : null
 
const foo = maybe1 > maybe2
  ? 'bar'
  : maybeNull
 
// best
const maybeNull = value1 > value2 ? 'baz' : null
 
const foo = maybe1 > maybe2 ? 'bar' : maybeNull
```

#### 2.11.4 避免不需要的三元表达式

```javascript
// bad
const foo = a ? a : b
const bar = c ? true : false
const baz = c ? false : true
 
// good
const foo = a || b
const bar = !!c
const baz = !c
```

#### 2.11.5 用圆括号来混合这些操作符。 只有当标准的算术运算符(`+`, `-`, `*`, & `/`)， 并且它们的优先级显而易见时，可以不用圆括号括起来

```javascript
// bad
const foo = a && b < 0 || c > 0 || d + 1 === 0
 
// bad
const bar = a ** b - 5 % d
 
// bad
// 别人会陷入(a || b) && c 的迷惑中
if (a || b && c) {
  return d
}
 
// good
const foo = (a && b < 0) || c > 0 || (d + 1 === 0)
 
// good
const bar = (a ** b) - (5 % d)
 
// good
if (a || (b && c)) {
  return d
}
 
// good
const bar = a + b / c * d
```



## 2.12 代码块

#### 2.12.1 用大括号包裹多行代码块

```javascript
// bad
if (test)
  return false
 
// good
if (test) return false
 
// good
if (test) {
  return false
}
 
// bad
function foo() { return false }
 
// good
function bar() {
  return false
}
```

#### 2.12.2 if 表达式的 else 和 if 的关闭大括号要在同一行

```javascript
// bad
if (test) {
  thing1()
  thing2()
}
else {
  thing3()
}
 
// good
if (test) {
  thing1()
  thing2()
} else {
  thing3()
}
```

#### 2.12.3 if 语句中的return

如果 `if` 语句中的 return 总是需要用 return 返回，那后续的 `else` 就不需要写了。`if` 块中包含 `return`，它后面的 `else if` 块中也包含了 `return`，这个时候就可以把 `return` 分到多个 `if` 语句块中。

```javascript
// bad
function foo() {
  if (x) {
    return x
  } else {
    return y
  }
}

// bad
function cats() {
  if (x) {
    return x
  } else if (y) {
    return y
  }
}

// bad
function dogs() {
  if (x) {
    return x
  } else {
    if (y) {
      return y
    }
  }
}

// good
function foo() {
  if (x) {
    return x
  }

  return y
}

// good
function cats() {
  if (x) {
    return x
  }

  if (y) {
    return y
  }
}

// good
function dogs(x) {
  if (x) {
    if (z) {
      return y
    }
  } else {
    return z
  }
}
```



## 2.13 注释

#### 2.13.1 多行注释用 /** ... */

```javascript
// bad
// make() returns a new element
// based on the passed in tag name
//
// @param {String} tag
// @return {Element} element
function make(tag) {

  // ...

  return element
}

// good
/**
 * make() returns a new element
 * based on the passed-in tag name
 */
function make(tag) {

  // ...

  return element
}
```

#### 2.13.2 单行注释用 `//`

将单行注释放在被注释区域上面。如果注释不是在第一行，那么注释前面就空一行。

```javascript
// bad
const active = true  // is current tab

// good
// is current tab
const active = true

// bad
function getType() {
  console.log('fetching type...');
  // set the default type to 'no type'
  const type = this._type || 'no type'

  return type
}

// good
function getType() {
  console.log('fetching type...')

  // set the default type to 'no type'
  const type = this._type || 'no type'

  return type
}

// also good
function getType() {
  // set the default type to 'no type'
  const type = this._type || 'no type'

  return type
}
```

#### 2.13.3 注释开头的空格

所有注释开头空一个空格，方便阅读。

```javascript
// bad
//is current tab
const active = true

// good
// is current tab
const active = true

// bad
/**
 *make() returns a new element
 *based on the passed-in tag name
 */
function make(tag) {

  // ...

  return element
}

// good
/**
 * make() returns a new element
 * based on the passed-in tag name
 */
function make(tag) {

  // ...

  return element
}
```



## 2.14 空格

#### 2.14.1 tab 用两个空格

```javascript
// bad
function foo() {
∙∙∙∙const name
}

// bad
function bar() {
∙const name
}

// good
function baz() {
∙∙const name
}
```

#### 2.14.2 在大括号前空一格

```javascript
// bad
function test(){
  console.log('test')
}

// good
function test() {
  console.log('test')
}

// bad
dog.set('attr',{
  age: '1 year',
  breed: 'Bernese Mountain Dog',
})

// good
dog.set('attr', {
  age: '1 year',
  breed: 'Bernese Mountain Dog',
})
```

#### 2.14.3 在控制语句(`if`, `while` 等)的圆括号前空一格

在函数调用和定义时，参数列表和函数名之间不空格。

```javascript
// bad
if(isJedi) {
  fight ()
}

// good
if (isJedi) {
  fight()
}

// bad
function fight () {
  console.log ('Swooosh!')
}

// good
function fight() {
  console.log('Swooosh!')
}
```

#### 2.14.4 用空格来隔开运算符

```javascript
// bad
const x=y+5

// good
const x = y + 5
```

#### 2.14.5 在一个代码块后和下一条语句前空一行

```javascript
// bad
if (foo) {
  return bar
}
return baz

// good
if (foo) {
  return bar
}

return baz

// bad
const obj = {
  foo() {
  },
  bar() {
  },
}
return obj

// good
const obj = {
  foo() {
  },

  bar() {
  },
}

return obj

// bad
const arr = [
  function foo() {
  },
  function bar() {
  },
]
return arr

// good
const arr = [
  function foo() {
  },

  function bar() {
  },
]

return arr
```

#### 2.14.6 不要用空白行填充块

```javascript
// bad
function bar() {

  console.log(foo)

}

// also bad
if (baz) {

  console.log(qux)
} else {
  console.log(foo)

}

// good
function bar() {
  console.log(foo)
}

// good
if (baz) {
  console.log(qux)
} else {
  console.log(foo)
}
```

#### 2.14.7 花括号中需要加空格

```javascript
// bad
const foo = {clark: 'kent'}

// good
const foo = { clark: 'kent' }
```

#### 2.14.8 在对象的字面量属性中，key value 之间要有空格

```javascript
// bad
var obj = { "foo" : 42 }
var obj2 = { "foo":42 }

// good
var obj = { "foo": 42 }
```



## 2.15 类型转换

#### 2.15.1 转换成布尔值

```javascript
const age = 0

// bad
const hasAge = new Boolean(age)

// good
const hasAge = Boolean(age)

// best
const hasAge = !!age
```

#### 2.15.2 转换成Number

优先用 `Number` 做类型转换，因为`parseInt`转换string常需要带上基数。

```javascript
const inputValue = '4'

// bad
const val = new Number(inputValue)

// bad
const val = +inputValue

// bad
const val = inputValue >> 0

// bad
const val = parseInt(inputValue)

// good
const val = Number(inputValue)

// good
const val = parseInt(inputValue, 10)
```



# 3. 命名基本规范

## 3.1 JavaScript 常规命名

#### 3.1.1 避免用一个字母命名，要让你的命名可描述

```javascript
// bad
function q() {
  // ...
}

// good
function query() {
  // ...
}
```

#### 3.1.2 用小驼峰式命名对象、函数、实例

```javascript
// bad
const OBJEcttsssss = {}
const this_is_my_object = {}
function c() {}

// good
const thisIsMyObject = {}
function thisIsMyFunction() {}
```

#### 3.1.3 用大驼峰式命名类

```javascript
// bad
function user(options) {
  this.name = options.name
}

const bad = new user({
  name: 'nope',
})

// good
class User {
  constructor(options) {
    this.name = options.name
  }
}

const good = new User({
  name: 'yup',
})
```

#### 3.1.4 不要用前置或后置下划线

前置下划线通常在概念上意味着“private”

```javascript
// bad
this.__firstName__ = 'Panda'
this.firstName_ = 'Panda'
this._firstName = 'Panda'

// good
this.firstName = 'Panda'
```

#### 3.1.5 系统常量命名全部大写，单词用_下划线连接

要确保是 const 定义的， 保证不能被改变。

```javascript
// bad
const auditStatus = 'failed'

// good
const AUDIT_STATUS = 'failed'
```



## 3.2 Vue 常规命名

#### 3.2.1 组件名为多个单词

组件名应该始终是多个单词的，并始终需要命名（根组件 `App` 以及 `<transition>`、`<component>` 之类的 Vue 内置组件除外），采用PascalCase命名方式。

组件文件名与组件名保持一致。

```javascript
// bad
Vue.component('Detail', {
  // ...
})

export default {
  name: 'Detail',
  // ...
}


// good
Vue.component('NewsDetail', {
  // ...
})

export default {
  name: 'NewsDetail',
  // ...
}
```

#### 3.2.2 基础组件名

应用特定样式和约定的基础组件 (也就是展示类的、无逻辑的或无状态的组件) 应该全部以一个特定的前缀开头，比如 `Base`。

```javascript
// bad
components/
|- MyButton.vue
|- VueTable.vue
|- Icon.vue

// good
components/
|- BaseButton.vue
|- BaseTable.vue
|- BaseIcon.vue
```

#### 3.2.3 单例组件名

只应该拥有单个活跃实例的组件应该以 `The` 前缀命名，以示其唯一性。这不意味着组件只可用于一个单页面，而是每个页面只使用一次。这些组件永远不接受任何 prop，因为它们是为你的应用定制的，而不是它们在你的应用中的上下文。如果你发现有必要添加 prop，那就表明这实际上是一个可复用的组件，只是目前在每个页面里只使用一次。

```javascript
// bad
components/
|- Heading.vue
|- MySidebar.vue

// good
components/
|- TheHeading.vue
|- TheSidebar.vue
```

#### 3.2.4 紧密耦合的组件名

和父组件紧密耦合的子组件应该以父组件名作为前缀命名。如果一个组件只在某个父组件的场景下有意义，这层关系应该体现在其名字上。因为编辑器通常会按字母顺序组织文件，所以这样做可以把相关联的文件排在一起。

```javascript
// bad
components/
|- TodoList.vue
|- TodoItem.vue
|- TodoButton.vue
|- NavigationForSearchSidebar.vue

// good
components/
|- TodoList.vue
|- TodoListItem.vue
|- TodoListItemButton.vue
|- SearchSidebar.vue
|- SearchSidebarNavigation.vue
```

#### 3.2.5 组件名中的单词顺序

组件名应该以高级别的 (通常是一般化描述的) 单词开头，以描述性的修饰词结尾。

```javascript
// bad
components/
|- ClearSearchButton.vue
|- ExcludeFromSearchInput.vue
|- LaunchOnStartupCheckbox.vue
|- RunSearchButton.vue

// good
components/
|- SearchButtonClear.vue
|- SearchButtonRun.vue
|- SearchInputQuery.vue
|- SearchInputExcludeGlob.vue
```

#### 3.2.6 模版中使用组件

在模版中使用组件时统一采用kebab-case命名。

```vue
<template>
	<div>
    <base-component></base-component>
  </div>
<template>
import BaseComponent from '...'
<script>
  components: {
    BaseComponent
  }
</script>
```

#### 3.2.7 完整单词的组件名

组件名应该倾向于完整单词而不是缩写，这样可以带来非常好的明确性。

```javascript
// bad
components/
|- SdSettings.vue
|- UProfOpts.vue

// good
components/
|- StudentDashboardSettings.vue
|- UserProfileOptions.vue
```

#### 3.2.8 Prop

在声明 prop 的时候，其命名应该始终使用 `camelCase`，而在模板和 JSX 中应该始终使用 `kebab-case`。

因为在 JavaScript 中更自然的是 `camelCase`，而在 HTML 中则是 `kebab-case`。

```vue
// bad
props: {
  'greeting-text': String
}
<WelcomeMessage greetingText="hi"/>

// good
props: {
  greetingText: String
}
<WelcomeMessage greeting-text="hi"/>
```

