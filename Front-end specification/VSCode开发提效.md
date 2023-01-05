## 1.1 [Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur)

Vetur 是 vscode 中必备的 Vue 开发工具，它所支持的功能如下所示。

### 1.1.1 语法高亮

Vetur 支持 .vue 文件的语法高亮显示，而且除了支持 template 模板以外，还支持大多数主流的前端开发脚本和插件，比如Sass和TypeScript，完整的高亮语法如下所示：

| syntax                   | lang    | required extension                                           |
| ------------------------ | ------- | ------------------------------------------------------------ |
| \<template\>             | html    |                                                              |
| \<template lang="pug"\>  | pug     |                                                              |
| \<template lang="jade"\> | pug     |                                                              |
| \<template lang="haml"\> | haml    | [Better Haml](https://marketplace.visualstudio.com/items?itemName=karunamurti.haml) or [Ruby Haml](https://marketplace.visualstudio.com/items?itemName=vayan.haml) |
| \<template lang="slm"\>  | slm     | [Slm Syntax](https://marketplace.visualstudio.com/items?itemName=mrmlnc.vscode-slm) |
| \<template lang="slim"\> | slim    | [Slim](https://marketplace.visualstudio.com/items?itemName=sianglim.slim) |
| \<style\>                | css     |                                                              |
| \<style lang="postcss"\> | postcss |                                                              |
| \<style lang="scss"\>    | scss    |                                                              |
| \<style lang="sass"\>    | sass    | [Sass](https://marketplace.visualstudio.com/items?itemName=Syler.sass-indented) |
| \<style lang="less"\>    | less    |                                                              |
| \<style lang="stylus"\>  | stylus  | [language-stylus](https://marketplace.visualstudio.com/items?itemName=sysoev.language-stylus) |
| \<script\>               | js      |                                                              |
| \<script lang="ts"\>     | ts      |                                                              |
| \<script lang="coffee"\> | coffee  |                                                              |

Vetur 也支持 vue 指令（例如：v-if 或 :attribute=）和 vue 插值的语法高亮显示（例如：{{ variable }} ）。

### 1.1.2 错误检查

Vetur对以下语言进行错误检查：

- \<template\>： html
- \<style\>：css，scss，less
- \<script\>：js，ts

也可以在 settings.json 文件中关闭错误检查：

```json
"vetur.validation.[template/style/script]" = false
```

通过配置 "vetur.experimental.templateInterpolationService": true 可以让我们在模版中也能检测 ts 语法。



如果想要使用 ESLint 规则来做错误检查，则需要以下配置。

- 在 settings.json 中使用以下命令关闭 Vetu r的模板验证：

```json
"vetur.validation.template": false
```

- 确保具有 [ESLint插件](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)，这样错误检查将会来自 ESLint 插件，而不是 Vetur。
- 在工作区根目录中安装 ESLint：

```nginx
yarn add -D eslint eslint-plugin-vue
```

- 在 .eslintrc 文件中配置 ESLint 规则。例如：

```json
{
  "extends": [
    "eslint:recommended",
    "plugin:vue/recommended"
  ],
  "rules": {
    "vue/html-self-closing": "off"
  }
}
```

### 1.1.3 格式化

Vetur 支持格式 `html/pug/css/scss/less/postcss/stylus/js/ts`。

#### Formatter

Vetur 可以配合下列 Formatter 使用：

- prettier
- perttier-eslint
- stylus-supremacy
- vscode-typescript
- sass-formatter

当 Vetur 观察到本地安装的 Formatter 时，它将更优先使用本地的 Formatter。

你可以在 vscode 配置中通过设置 “vetur.format.defaultFormatter” 来选择每种语言的默认格式（将一种语言的 Formatter 设置为 none 则表示禁用该语言的 Formatter），以下为默认值：

```json
{
  "vetur.format.defaultFormatter.html": "prettier",
  "vetur.format.defaultFormatter.pug": "prettier",
  "vetur.format.defaultFormatter.css": "prettier",
  "vetur.format.defaultFormatter.postcss": "prettier",
  "vetur.format.defaultFormatter.scss": "prettier",
  "vetur.format.defaultFormatter.less": "prettier",
  "vetur.format.defaultFormatter.stylus": "stylus-supremacy",
  "vetur.format.defaultFormatter.js": "prettier",
  "vetur.format.defaultFormatter.ts": "prettier",
  "vetur.format.defaultFormatter.sass": "sass-formatter"
}
```

还可以通过 “vetur.format.defaultFormatterOptions” 来设置风格工具（Vetur 推荐使用2个空格来规定 editor.tabSize 和 editor.insertSpace，对于 html和css/scss/less 格式则使用 js-beautify 来规范）：

```json
{
  "vetur.format.defaultFormatterOptions": {
    "js-beautify-html": {  // 备用的 html formatter
      "wrap_attributes": "force-expand-multiline"
    },
    "prettyhtml": {
      "printWidth": 100,
      "singleQuote": false,
      "wrapAttributes": false,
      "sortAttributes": false
    },
  }
}
```

#### Vetur Formatter config

可以使用全局开关 “vetur.format.enable” 来打开和关闭 Vetur Formatter。如果要让 Prettier 来完全处理文件格式，这将很有用。

也可以配置全局的 Formatter 选项，比如以下这两个设置将被所有 Formatter 继承：

```json
{
  "vetur.format.options.tabSize": 2,
  "vetur.format.options.useTabs": false
}
```

但当其找到本地配置（如 .prettierrc 文件）时，Vetur 会更优先的使用本地配置，也就是本地配置中的 Formatter 选项会覆盖 Vetur Formatter 的全局配置。

### 1.1.4 智能感知

Vetur 为 html 标签和属性提供了智能感知。同样也适用于 Vue 特定属性，比如 v-if 和 @click。

### 1.1.5 代码片段

Vetur 允许为每种嵌入式语言使用代码片段

例如，为TypeScript定义的代码段将在TypeScript区域中提供：

```vue
<script lang="ts">
  // Use TS snippets here
</script>
```

两个例外：

- 使用 vue-html 代码片段时，需要在 <template></template> 内部
- vue 可以在所有地方之外使用代码片段

```vue
<template>
  <!-- Use `vue-html` snippets here -->
</template>
<!-- Use `vue` snippets here -->
<style>
</style>
```

## 1.2 [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)

### 1.2.1 基本使用

- 在工作区根目录中安装 ESLint：

```nginx
npm install eslint -D
```

- 在vscode中安装 ESLint 插件。
- 在 .eslintrc 或 .eslintrc.js 文件中配置 ESLint 规则，在 .eslintignore 文件中配置要忽略的内容
- .eslintrc 和.eslintrc.js 写法的主要区别是：.eslintrc 中写一个 json，.eslintrc.js 中需要写 module.exports = { }

### 1.2.2 配置说明

.eslintrc 文件中的主要配置如下：

```json
{  
  "root": true,
  "env": {
    ...
  },
  "globals": {
    ...
  },
  "extends": [
   ...
  ],
  "parserOptions": {
    ...
  },
  // 在 rules 中添加自定义的规则
  "rules": {
    'space-before-function-paren': 'off',
    'no-console': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'warn' : 'off'
    ...
  }
}
```

env 选项决定你的脚本将要运行在什么环境,常见的有 browser、node 等

globals 是额外的全局变量

rules 是配置规则

- 配置错误规则的三种等级：0 或 off 表示关闭规则；1 或 warn 表示打开规则，并且作为一个警告，不会导致检查不通过；2 或 error 表示打开规则，并且作为一个错误，会导致检查代码不通过。
- 配置形式：

```json
// 第一个参数是错误等级,第二个参数是处理方式
"array-bracket-spacing": [2, "never"]
```

### 1.2.3 自动修复

可以直接在 package.json 中配置：

```json
"scripts": {
  "eslint": "eslint . --fix"
},
```

也可以在 vscode 的 settings.json 文件中配置：

```json
"editor.codeActionsOnSave": {
  "source.fixAll": true
}
```

### 1.2.4 常用配置

rules 中的常用配置项：

```json
"no-alert": 0,//禁止使用alert confirm prompt
"no-array-constructor": 2,//禁止使用数组构造器
"no-bitwise": 0,//禁止使用按位运算符
"no-caller": 1,//禁止使用arguments.caller或arguments.callee
"no-catch-shadow": 2,//禁止catch子句参数与外部作用域变量同名
"no-class-assign": 2,//禁止给类赋值
"no-cond-assign": 2,//禁止在条件表达式中使用赋值语句
"no-console": 2,//禁止使用console
"no-const-assign": 2,//禁止修改const声明的变量
"no-constant-condition": 2,//禁止在条件中使用常量表达式 if(true) if(1)
"no-continue": 0,//禁止使用continue
"no-control-regex": 2,//禁止在正则表达式中使用控制字符
"no-debugger": 2,//禁止使用debugger
"no-delete-var": 2,//不能对var声明的变量使用delete操作符
"no-div-regex": 1,//不能使用看起来像除法的正则表达式/=foo/
"no-dupe-keys": 2,//在创建对象字面量时不允许键重复 {a:1,a:1}
"no-dupe-args": 2,//函数参数不能重复
"no-duplicate-case": 2,//switch中的case标签不能重复
"no-else-return": 2,//如果if语句里面有return,后面不能跟else语句
"no-empty": 2,//块语句中的内容不能为空
"no-empty-character-class": 2,//正则表达式中的[]内容不能为空
"no-empty-label": 2,//禁止使用空label
"no-eq-null": 2,//禁止对null使用==或!=运算符
"no-eval": 1,//禁止使用eval
"no-ex-assign": 2,//禁止给catch语句中的异常参数赋值
"no-extend-native": 2,//禁止扩展native对象
"no-extra-bind": 2,//禁止不必要的函数绑定
"no-extra-boolean-cast": 2,//禁止不必要的bool转换
"no-extra-parens": 2,//禁止非必要的括号
"no-extra-semi": 2,//禁止多余的冒号
"no-fallthrough": 1,//禁止switch穿透
"no-floating-decimal": 2,//禁止省略浮点数中的0 .5 3.
"no-func-assign": 2,//禁止重复的函数声明
"no-implicit-coercion": 1,//禁止隐式转换
"no-implied-eval": 2,//禁止使用隐式eval
"no-inline-comments": 0,//禁止行内备注
"no-inner-declarations": [2, "functions"],//禁止在块语句中使用声明（变量或函数）
"no-invalid-regexp": 2,//禁止无效的正则表达式
"no-invalid-this": 2,//禁止无效的this，只能用在构造器，类，对象字面量
"no-irregular-whitespace": 2,//不能有不规则的空格
"no-iterator": 2,//禁止使用__iterator__ 属性
"no-label-var": 2,//label名不能与var声明的变量名相同
"no-labels": 2,//禁止标签声明
"no-lone-blocks": 2,//禁止不必要的嵌套块
"no-lonely-if": 2,//禁止else语句内只有if语句
"no-loop-func": 1,//禁止在循环中使用函数（如果没有引用外部变量不形成闭包就可以）
"no-mixed-requires": [0, false],//声明时不能混用声明类型
"no-mixed-spaces-and-tabs": [2, false],//禁止混用tab和空格
"linebreak-style": [0, "windows"],//换行风格
"no-multi-spaces": 1,//不能用多余的空格
"no-multi-str": 2,//字符串不能用\换行
"no-multiple-empty-lines": [1, {"max": 2}],//空行最多不能超过2行
"no-native-reassign": 2,//不能重写native对象
"no-negated-in-lhs": 2,//in 操作符的左边不能有!
"no-nested-ternary": 0,//禁止使用嵌套的三目运算
"no-new": 1,//禁止在使用new构造一个实例后不赋值
"no-new-func": 1,//禁止使用new Function
"no-new-object": 2,//禁止使用new Object()
"no-new-require": 2,//禁止使用new require
"no-new-wrappers": 2,//禁止使用new创建包装实例，new String new Boolean new Number
"no-obj-calls": 2,//不能调用内置的全局对象，比如Math() JSON()
"no-octal": 2,//禁止使用八进制数字
"no-octal-escape": 2,//禁止使用八进制转义序列
"no-param-reassign": 2,//禁止给参数重新赋值
"no-path-concat": 0,//node中不能使用__dirname或__filename做路径拼接
"no-plusplus": 0,//禁止使用++，--
"no-process-env": 0,//禁止使用process.env
"no-process-exit": 0,//禁止使用process.exit()
"no-proto": 2,//禁止使用__proto__属性
"no-redeclare": 2,//禁止重复声明变量
"no-regex-spaces": 2,//禁止在正则表达式字面量中使用多个空格 /foo bar/
"no-restricted-modules": 0,//如果禁用了指定模块，使用就会报错
"no-return-assign": 1,//return 语句中不能有赋值表达式
"no-script-url": 0,//禁止使用javascript:void(0)
"no-self-compare": 2,//不能比较自身
"no-sequences": 0,//禁止使用逗号运算符
"no-shadow": 2,//外部作用域中的变量不能与它所包含的作用域中的变量或参数同名
"no-shadow-restricted-names": 2,//严格模式中规定的限制标识符不能作为声明时的变量名使用
"no-spaced-func": 2,//函数调用时 函数名与()之间不能有空格
"no-sparse-arrays": 2,//禁止稀疏数组， [1,,2]
"no-sync": 0,//nodejs 禁止同步方法
"no-ternary": 0,//禁止使用三目运算符
"no-trailing-spaces": 1,//一行结束后面不要有空格
"no-this-before-super": 0,//在调用super()之前不能使用this或super
"no-throw-literal": 2,//禁止抛出字面量错误 throw "error";
"no-undef": 1,//不能有未定义的变量
"no-undef-init": 2,//变量初始化时不能直接给它赋值为undefined
"no-undefined": 2,//不能使用undefined
"no-unexpected-multiline": 2,//避免多行表达式
"no-underscore-dangle": 1,//标识符不能以_开头或结尾
"no-unneeded-ternary": 2,//禁止不必要的嵌套 var isYes = answer === 1 ? true : false;
"no-unreachable": 2,//不能有无法执行的代码
"no-unused-expressions": 2,//禁止无用的表达式
"no-unused-vars": [2, {"vars": "all", "args": "after-used"}],//不能有声明后未被使用的变量或参数
"no-use-before-define": 2,//未定义前不能使用
"no-useless-call": 2,//禁止不必要的call和apply
"no-void": 2,//禁用void操作符
"no-var": 0,//禁用var，用let和const代替
"no-warning-comments": [1, { "terms": ["todo", "fixme", "xxx"], "location": "start" }],//不能有警告备注
"no-with": 2,//禁用with
 
"array-bracket-spacing": [2, "never"],//是否允许非空数组里面有多余的空格
"arrow-parens": 0,//箭头函数用小括号括起来
"arrow-spacing": 0,//=>的前/后括号
"accessor-pairs": 0,//在对象中使用getter/setter
"block-scoped-var": 0,//块语句中使用var
"brace-style": [1, "1tbs"],//大括号风格
"callback-return": 1,//避免多次调用回调什么的
"camelcase": 2,//强制驼峰法命名
"comma-dangle": [2, "never"],//对象字面量项尾不能有逗号
"comma-spacing": 0,//逗号前后的空格
"comma-style": [2, "last"],//逗号风格，换行时在行首还是行尾
"complexity": [0, 11],//循环复杂度
"computed-property-spacing": [0, "never"],//是否允许计算后的键名什么的
"consistent-return": 0,//return 后面是否允许省略
"consistent-this": [2, "that"],//this别名
"constructor-super": 0,//非派生类不能调用super，派生类必须调用super
"curly": [2, "all"],//必须使用 if(){} 中的{}
"default-case": 2,//switch语句最后必须有default
"dot-location": 0,//对象访问符的位置，换行的时候在行首还是行尾
"dot-notation": [0, { "allowKeywords": true }],//避免不必要的方括号
"eol-last": 0,//文件以单一的换行符结束
"eqeqeq": 2,//必须使用全等
"func-names": 0,//函数表达式必须有名字
"func-style": [0, "declaration"],//函数风格，规定只能使用函数声明/函数表达式
"generator-star-spacing": 0,//生成器函数*的前后空格
"guard-for-in": 0,//for in循环要用if语句过滤
"handle-callback-err": 0,//nodejs 处理错误
"id-length": 0,//变量名长度
"indent": [2, 4],//缩进风格
"init-declarations": 0,//声明时必须赋初值
"key-spacing": [0, { "beforeColon": false, "afterColon": true }],//对象字面量中冒号的前后空格
"lines-around-comment": 0,//行前/行后备注
"max-depth": [0, 4],//嵌套块深度
"max-len": [0, 80, 4],//字符串最大长度
"max-nested-callbacks": [0, 2],//回调嵌套深度
"max-params": [0, 3],//函数最多只能有3个参数
"max-statements": [0, 10],//函数内最多有几个声明
"new-cap": 2,//函数名首行大写必须使用new方式调用，首行小写必须用不带new方式调用
"new-parens": 2,//new时必须加小括号
"newline-after-var": 2,//变量声明后是否需要空一行
"object-curly-spacing": [0, "never"],//大括号内是否允许不必要的空格
"object-shorthand": 0,//强制对象字面量缩写语法
"one-var": 1,//连续声明
"operator-assignment": [0, "always"],//赋值运算符 += -=什么的
"operator-linebreak": [2, "after"],//换行时运算符在行尾还是行首
"padded-blocks": 0,//块语句内行首行尾是否要空行
"prefer-const": 0,//首选const
"prefer-spread": 0,//首选展开运算
"prefer-reflect": 0,//首选Reflect的方法
"quotes": [1, "single"],//引号类型 `` "" ''
"quote-props":[2, "always"],//对象字面量中的属性名是否强制双引号
"radix": 2,//parseInt必须指定第二个参数
"id-match": 0,//命名检测
"require-yield": 0,//生成器函数必须有yield
"semi": [2, "always"],//语句强制分号结尾
"semi-spacing": [0, {"before": false, "after": true}],//分号前后空格
"sort-vars": 0,//变量声明时排序
"space-after-keywords": [0, "always"],//关键字后面是否要空一格
"space-before-blocks": [0, "always"],//不以新行开始的块{前面要不要有空格
"space-before-function-paren": [0, "always"],//函数定义时括号前面要不要有空格
"space-in-parens": [0, "never"],//小括号里面要不要有空格
"space-infix-ops": 0,//中缀操作符周围要不要有空格
"space-return-throw-case": 2,//return throw case后面要不要加空格
"space-unary-ops": [0, { "words": true, "nonwords": false }],//一元运算符的前/后要不要加空格
"spaced-comment": 0,//注释风格要不要有空格什么的
"strict": 2,//使用严格模式
"use-isnan": 2,//禁止比较时使用NaN，只能用isNaN()
"valid-jsdoc": 0,//jsdoc规则
"valid-typeof": 2,//必须使用合法的typeof的值
"vars-on-top": 2,//var必须放在作用域顶部
"wrap-iife": [2, "inside"],//立即执行函数表达式的小括号风格
"wrap-regex": 0,//正则表达式字面量用小括号包起来
"yoda": [2, "never"]//禁止尤达条件
```



## 1.3 [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)

安装 prettier 插件后，将 prettier 选为默认格式化代码的方式，在 vscode 中使用 shift+alt+f 快捷键即可格式化代码。

若想在保存时自动格式化代码，则在 vscode 的 settings.json 文件中配置 "editor.formatOnSave": true 即可。

### 1.3.1 配置方式

- 通过 vscode 的 settings.json 文件配置：

```json
{
  "prettier.singleQuote": true,
  "prettier.semi": false,
  "prettier.printWidth": 400,
  "prettier.trailingComma": "none"
}
```

- 通过添加自定义配置文件 .prettierrc 来配置：

```json
{
  "singleQuote": true,
  "semi": false,
  "printWidth": 400,
  "trailingComma": "es5"
}
```

### 1.3.2 常用配置

常用配置项：

```json
"prettier.useEditorConfig": false, // 使prettier忽略.editorconfig中的配置规则
"prettier.printWidth": 400, // 超过最大值换行    
"prettier.tabWidth": 4, // 缩进字节数   
"prettier.useTabs": false, // 缩进不使用tab，使用空格    
"prettier.semi": true, // 句尾添加分号    
"prettier.singleQuote": true, // 使用单引号代替双引号    
"prettier.proseWrap": "preserve", // 默认值。因为使用了一些折行敏感型的渲染器（如GitHub comment）而按照markdown文本样式进行折行    
"prettier.arrowParens": "avoid", //  (x) => {} 箭头函数参数只有一个时是否要有小括号。avoid：省略括号    
"prettier.bracketSpacing": true, // 在对象，数组括号与文字之间加空格 "{ foo: bar }"    
"prettier.disableLanguages": ["vue"], // 不格式化vue文件，vue文件的格式化单独设置    
"prettier.endOfLine": "auto", // 结尾是 \n \r \n\r auto    
"prettier.eslintIntegration": false, //不让prettier使用eslint的代码格式进行校验    
"prettier.htmlWhitespaceSensitivity": "ignore",
"prettier.ignorePath": ".prettierignore", // 不使用prettier格式化的文件填写在项目的.prettierignore文件中   
"prettier.jsxBracketSameLine": false, // 在jsx中把'>' 是否单独放一行    
"prettier.jsxSingleQuote": false, // 在jsx中使用单引号代替双引号    
"prettier.parser": "babylon", // 格式化的解析器，默认是babylon    
"prettier.requireConfig": false, // Require a 'prettierconfig' to format prettier    
"prettier.stylelintIntegration": false, //不让prettier使用stylelint的代码格式进行校验    
"prettier.trailingComma": "es5", // 在对象或数组最后一个元素后面是否加逗号（在ES5中加尾逗号）（设置为"none"则不添加尾逗号）    
"prettier.tslintIntegration": false // 不让prettier使用tslint的代码格式进行校验}
"prettier.vueIndentScriptAndStyle": false, // 不让vue文件在<script></script>之间的代码使用vue单独的缩进方式
```

> Settings should be read from：
>
> 1. Prettier configuration file (`.prettierrc`)
> 2. `.editorconfig`
> 3. Visual Studio Code Settings (`settings.json`)
>
> But as long as there's an `.editorconfig` in the project without `.prettierrc`, Settings which have been written in the `VSCode's settings.json` are no longer valid, even if they are not mentioned in the `.editorconfig`.

## 1.4 [Markdown All in One](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one)

可以在 vscode 中快速的编辑和预览 markdown 文档，具有键盘快捷键、目录、自动预览等功能。

## 1.5 [Auto Import](https://marketplace.visualstudio.com/items?itemName=steoates.autoimport)

在 import 时，自动去查找、分析、然后提供代码补全。对于 TypeScript 和 TSX，可以适用。

## 1.6 [Auto Close Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-close-tag)

自动关闭标签，在开始标记的结束括号中键入后，将自动插入结束标记。

## 1.7 [Auto Rename Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-rename-tag)

自动重命名配对的 HTML / XML 标记。

## 1.8 [IntelliSense for CSS class names in HTML](https://marketplace.visualstudio.com/items?itemName=Zignd.html-css-class-completion)

基于你的项目以及通过 link 标签引用的外部文件，该智能插件提供 HTML 中 CSS class 名字的补全。

## 1.9 [CSS Peek](https://marketplace.visualstudio.com/items?itemName=pranaygp.vscode-css-peek)

可以在 HTML 中通过 CSS id 或则 class 来定位到其定义。

## 1.10 [npm](https://marketplace.visualstudio.com/items?itemName=eg2.vscode-npm-script)

这个扩展支持运行在包中定义的 npm 脚本，并根据 package.json 中定义的依赖关系验证已安装模块。

## 1.11 [npm Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.npm-intellisense)

可以在导入 npm 模块时，自动补全模块名称。

## 1.12 [SVG Viewer](https://marketplace.visualstudio.com/items?itemName=cssho.vscode-svgviewer)

一个用来预览SVG的插件。

## 1.13 [Bracket Pair Colorizer](https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer)

可以使匹配的括号进行高亮显示，以区分不同的代码块。

## 1.14 [Debugger for Chrome](https://marketplace.visualstudio.com/items?itemName=msjsdiag.debugger-for-chrome)

用 Chrome 来 Debug 你的 JavaScript 代码。

## 1.15 [Code Runner](https://marketplace.visualstudio.com/items?itemName=formulahendry.code-runner)

支持多种语言的代码的立即执行。

支持的语言有：C, C++, Java, JavaScript, PHP, Python, Perl, Perl 6, Ruby, Go, Lua, Groovy, PowerShell, BAT/CMD, BASH/SH, F# Script, F# (.NET Core), C# Script, C# (.NET Core), VBScript, TypeScript, CoffeeScript, Scala, Swift, Julia, Crystal, OCaml Script, R, AppleScript, Elixir, Visual Basic .NET, Clojure, Haxe, Objective-C, Rust, Racket, AutoHotkey, AutoIt, Kotlin, Dart, Free Pascal, Haskell, Nim, D。

## 1.16 [Path Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense)

智能路径提示，可以在你输入文件路径时智能提示接下来的路径。

## 1.17 [Codelf](https://marketplace.visualstudio.com/items?itemName=unbug.codelf)

Codelf 是一个用来给变量命名的网站，你只要输入你想起的中文名，它就会给你提供很多建议的命名。

可在 vscode 中安装 Codelf 插件，选中字段点击右键即可弹出codelf选项，可快速搜索该词语对应的相应命名。

## 1.18 [Local History](https://marketplace.visualstudio.com/items?itemName=xyz.local-history)

当仅凭 Ctrl+z 撤销操作无法找回刚才删除的代码时，则可使用 Local history 来找回本地删除的代码。

安装这款插件之后在侧边栏会出现 LOCAL HISTORY 的字样，每当我们保存更改时，它都会备份一份历史文件，当我们需要恢复之前版本时，只需要点击一下对应的文件即可。此外，它还会在编辑框显示对比详情，能够让你对修改位置一目了然。

## 1.19 [file-size](https://marketplace.visualstudio.com/items?itemName=zh9528.file-size)

实时查看当前文件的大小。

## 1.20 [Image preview](https://marketplace.visualstudio.com/items?itemName=kisstkondoros.vscode-gutter-preview)

在左侧可以预览图片，鼠标悬停在图片的路径上时可以显示图片。

## 1.21 [Import Cost](https://marketplace.visualstudio.com/items?itemName=wix.vscode-import-cost)

可以方便的查看所导入的程序包的大小。

# 2. 用户代码片段

## 2.1 用户代码片段

在 vscode 中点击“首选项” → “用户片段”，然后选择 html.json 或其他现有代码片段即可创建自定义的用户代码片段。

如下述 Example 所示：

```json
{
	// Example:
	"vue3 snippets": {
	  "prefix": "vue3",
	  "body": [
		"<template>",
  		"  <div class=\"\"></div>",
		"</template>",
		"",
		"<script lang=\"ts\">",
		"import { defineComponent} from 'vue'",
		"export default defineComponent({",
		"  setup() {",
		"",
		"  }",
		"})",
		"</script>",
		"",
		"<style>",
		"",
		"</style>"
	  ]
	}
}
```

此时，在全局中，输入 vue3 并按下 tab 键即可直接生成上述“body”中内容。

# 3. 常用快捷键

## 3.1 常用快捷键

### 3.1.1 通用

| Press                | Function        |
| -------------------- | --------------- |
| Ctrl + Shift + P，F1 | 显示命令面板    |
| Ctrl + P             | 转到文件        |
| Ctrl + Shift + M     | 显示问题面板    |
| Ctrl + Shift + N     | 新窗口 / 实例   |
| Ctrl + Shift + W     | 关闭窗口 / 实例 |

### 3.1.2 基础编辑

| Press                  | Function                        |
| ---------------------- | ------------------------------- |
| Ctrl + X               | 剪切当前行（空选定）            |
| Ctrl + C               | 复制当前行（空选定）            |
| Ctrl + Shift + K       | 删除当前行                      |
| Ctrl + Enter           | 在下面插入行                    |
| Alt + ↑ / ↓            | 向上 / 向下移动当前行           |
| Shift + Alt + ↓ / ↑    | 向上 / 向下复制当前行           |
| Ctrl + Shift + ↓ / ↑   | 选中当前光标以下 / 以上所有内容 |
| Ctrl + Alt + ↓ / ↑     | 多行编辑                        |
| Shift + Alt + 鼠标左键 | 多列编辑                        |

### 3.1.3 搜索和替换

| Press    | Function                       |
| -------- | ------------------------------ |
| Ctrl + F | 查找                           |
| Ctrl + D | 将选择内容添加到下一个查找匹配 |

### 3.1.4 编辑器管理

| Press      | Function       |
| ---------- | -------------- |
| Ctrl + W   | 关闭当前编辑器 |
| Ctrl + K F | 关闭当前文件夹 |
| Ctrl + \   | 拆分编辑器     |

### 3.1.5 文件管理

| Press    | Function   |
| -------- | ---------- |
| Ctrl + N | 创建新文件 |
| Ctrl + O | 打开文件   |
| Ctrl + S | 保存文件   |

### 3.1.6 集成终端

| Press            | Function     |
| ---------------- | ------------ |
| Control + `      | 显示集成终端 |
| Ctrl + Shift + C | 打开外部终端 |

