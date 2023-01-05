## 写TS前的基础

### 构造函数

当通过 new 函数() 时，此时这个函数就是构造函数（日后会演变成 TS 类的构造器）

### 原型

很好的使用原型，可以避免内存空间的浪费，比如一些公共的数组和方法，使用原型则可以一直用同一个，而使用构造函数则会为每一个实例的公共方法和数组都在堆中开辟单独的空间



**面试题**

增加或修改原型对象的属性或方法后，所有的实例对象可以立即访问得到。但创建实例后，再覆盖原型，实例对象无法访问到，因为原型在堆中的地址已经改变了。



### TS 类

```ts
// 类的本质就是一个函数
class Person {
  // 实例属性/对象属性/实例的变量/对象的变量/类的(非静态)的属性/属性
  public name: string = 'noname'
  public age: number = 0

  constructor (_name: string, _age: number) {
    this.name = _name
    this.age = _age
  }
  
  public doEat (who: string, address: string): void {
    console.log(`${this.name}在${address}和${who}吃饭`)
  }
}

// 通过构造函数（构造器）来赋值
let person = new Person('Kathy', 23)
person.doEat('Mike', 'SOHO')
```

创建对象一共做了三件事：

1. 在堆中为类的某个对象（实例）分配一个空间
2. 调用对应的构造函数（构造器）（new Person() 自动匹配无参数的构造器）
3. 把对象赋值给对象变量



### 函数重载

函数签名 = 函数名称 + 函数参数 + 函数参数类型 + 返回值类型，在 TS 函数重载中，包含了`实现签名（最靠近函数体的那一个签名）`和`重载签名（可以有多个）`，实现签名是一种函数签名，重载签名也是一种函数签名



函数重载定义：包含了以下规则的一组函数就是 TS 函数重载

**规则1：**由一个实现签名 + 一个或多个重载签名合成

**规则2：**外部调用函数重载定义的函数时，**只能**调用重载签名，**不能**调用实现签名，这看似矛盾的规则，其实是 TS 的规定：实现签名下的函数体是给重载签名编写的，实现签名只是在定义时起到了统领所有重载签名的作用，在执行调用时就看不到实现签名了

**规则3：**调用重载函数时，会根据传递的参数来判断你调用的是哪一个函数

**规则4：**只有一个函数体，只有实现签名配备了函数体，所有的重载签名都只有签名，没有配备函数体

**规则5：关于参数类型规则完整总结**

无论重载签名有几个参数，参数类型是何种类型，实现签名都可以是一个无参函数签名；实现签名参数个数可以少于重载签名的参数个数，但实现签名如果准备包含重载签名的某个位置的参数，那实现签名就必须兼容所有重载签名该位置的参数类型【联合类型或 any 或 unknow 类型的一种】

例子：

```ts
function getMessage (value: number): Message
function getMessage (value: MessageType, readCount: number): Message[]
function getMessage (value: number | MessageType, readCount: number = 1): Message | Message[] {
  ...
}
// 实现签名的 readCount 参数一定要给一个默认值，因为第一个重载签名没有 readCount 这个参数，当函数体中要用到这个参数而重载签名中又未传入时，就会去找实现签名中的这个参数的默认值
```

**规则6：关于重载签名和实现签名的返回值类型规则完整总结**

必须给重载签名提供返回值类型，TS 无法默认推导

提供给重载签名的返回值类型不一定为其执行时的真实返回值类型，可以为重载签名提供真实返回值类型，也可以提供 void 或 unknow 或 any 类型，如果重载签名的返回值类型是 void 或 unknow 或 any 类型，那么将由实现签名来决定重载签名执行时的真实返回值类型。当然为了调用时能有自动提示 + 可读性更好 + 避免可能出现了类型强制转换，强烈建议为重载函数提供真实返回值类型。

不管重载签名返回值类型是何种类型【包括后面讲的泛型类型】，实现签名都可以返回 any 类型或 unknow 类型，当然一般我们两者都不选择，让 TS 默认为实现签名自动推导返回值类型。



### 方法重载

方法就是特定场景下的函数，一般来说，由对象变量【实例变量】直接调用的函数都是方法

比如：

1. 在函数内部，用 `this.xxx = () => {}` 这个格式加上去的就是方法
2. TS 类中定义的函数是方法【TS 类中定义的方法就是编译后 JS 底层 prototype 的一个函数】
3. 接口内部定义的函数是方法【注意：不是接口函数】
4. type 内部定义的函数是方法【注意：不是 type 函数】



### 继承

```ts
// 父类
class Vechile {
  public brand: string // 品牌
  public vechileNo: string // 车牌号
  constructor (brand_: string, vechileNo_: string) {
    this.brand = brand_
    this.vechileNo = vechileNo_
  }
  
  public calculate () {
    
  }
}

// 子类
class Car extends Vechile {
  public type: string
  constructor (brand_: string, vechileNo_: string, type_: string) {
    super(brand_, vichileNo_) // 相当于 Vechile.call(this, brand_, vechileNo_)
    this.type = type_
  }

  public calculate () {
    console.log(`${this.brand} is ${this.type}`)
  }
}

const car = new Car('普拉多', '京A10291', '汽车')
car.calculate()
```





## 原始数据类型

### void

在 TypeScript 中，可以用 `void` 表示没有任何返回值的函数：

```ts
function alertName(): void {
  alert('My name is Tom')
}
```



关于以下这段代码会报错的

```ts
let num: number = undefined
let unusable: void = null
```

此时配置文件 tsconfig.json 的 strict 是 false;(默认是true)

```json
"strict": true, /* Enable all strict type-checking options. */
```

默认是严格校验类型的。
如果改成false，就不会报错。
但是默认strict都是true，还是要按照严格来；

### Unknow 类型

https://blog.csdn.net/z591102/article/details/107151164/?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-0.no_search_link&spm=1001.2101.3001.4242.1

any 和 unknown 在 TypeScript 中是所谓的“顶部类型”。any 和 unknown 是包含所有值的集合。顺便说一句，TypeScript 还有 *bottom type* never，它是空集。

### 数组类型

三种表示方法：

```ts
let fibonacci: number[] = [1, 1, 2, 3, 5];
```

```ts
let fibonacci: Array<number> = [1, 1, 2, 3, 5];
```

```ts
interface NumberArray {
  [index: number]: number;
}
let fibonacci: NumberArray = [1, 1, 2, 3, 5];
```



用 `any` 表示数组中允许出现任意类型：

```ts
let list: any[] = ['xcatliu', 25, { website: 'http://xcatliu.com' }];
```



### 联合类型

如果 `let str: string | number` ，那么 `str.` 后面提示的方法是 string 和 number 这两种类型的交集方法，并不是并集



### 父类与子类

```ts
let x: number = 3
let y: any = x // 等号右边的类型可以看成等号左边的类型的子类

let z: any = 3
let k: number = z // any 既可以充当所有类型的父类也可以充当所有类型的子类

let z: unkown = 3
let k: number = z // unkown 只能充当所有类型的父类，不能充当子类
```



## 类型断言

定义：把两种能有`重叠关系`的数据类型进行相互转换的一种 TS 语法，把其中的一种数据类型转换成另外一种数据类型。类型断言和类型转换产生的效果一样，但语法格式不同。



什么是重叠关系：

1. 如果A，B都是类并且有继承关系，父类可以被断言为子类，子类也可以被断言成父类。但一般都是父类断言成子类







- 联合类型可以被断言为其中一个类型
- 父类可以被断言为子类，子类也可以被断言成父类
- 任何类型都可以被断言为 any
- any 可以被断言为任何类型
- 要使得 `A` 能够被断言为 `B`，只需要 `A` 兼容 `B` 或 `B` 兼容 `A` 即可

> 前四种情况都是最后一个的特例



TypeScript 是结构类型系统，类型之间的对比只会比较它们最终的结构，而会忽略它们定义时的关系。

例如：

```typescript
interface Animal {
  name: string
}
interface Cat {
  name: string
  run(): void
}
```

这里，TypeScript 并不关心 `Cat` 和 `Animal` 之间定义时是什么关系，而只会看它们最终的结构有什么关系——所以它与 `Cat extends Animal` 是等价的：

```typescript
interface Animal {
    name: string
}
interface Cat extends Animal {
    run(): void
}
```

所以，`Cat` 类型的 `tom` 可以赋值给 `Animal` 类型的 `animal` 了（就像面向对象编程中我们可以将子类的实例赋值给类型为父类的变量）：

```typescript
interface Animal {
    name: string
}

interface Cat {
    name: string
    run(): void
}

let tom: Cat = {
    name: 'Tom',
    run: () => { console.log('run') }
}

let animal: Animal = tom // Cat 类型的 tom 可以赋值给 Animal 类型的 animal
```

我们把它换成 TypeScript 中更专业的说法，即：`Animal` 兼容 `Cat`。

当 `Animal` 兼容 `Cat` 时，它们就可以互相进行类型断言了：

```ts
interface Animal {
    name: string
}
interface Cat {
    name: string
    run(): void
}

function testAnimal(animal: Animal) {
    return (animal as Cat)
}
function testCat(cat: Cat) {
    return (cat as Animal)
}
```

这样设计的原因是：

- 允许 `animal as Cat` 是因为「父类可以被断言为子类」，这个前面已经学习过了
- 允许 `cat as Animal` 是因为既然子类拥有父类的属性和方法，那么被断言为父类，获取父类的属性、调用父类的方法，就不会有任何问题，故「子类可以被断言为父类」



## 声明文件

当我们需要依赖一个全局变量的声明文件时，由于全局变量不支持通过 `import` 导入，当然也就必须使用三斜线指令来引入了（在全局变量的声明文件中，是不允许出现 `import`, `export` 关键字的。一旦出现了，那么他就会被视为一个 npm 包或 UMD 库，就不再是全局变量的声明文件了。故当我们在书写一个全局变量的声明文件时，如果需要引用另一个库的类型，那么就必须用三斜线指令了）

```ts
// types/node-plugin/index.d.ts

/// <reference types="node" />

export function foo(p: NodeJS.Process): string
```

```ts
// src/index.ts

import { foo } from 'node-plugin'

foo(global.process)
```

在上面的例子中，我们通过三斜线指引入了 `node` 的类型，然后在声明文件中使用了 `NodeJS.Process` 这个类型。最后在使用到 `foo` 的时候，传入了 `node` 中的全局变量 `process`。





