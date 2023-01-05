# 安装

```
// stylelint-order 属性顺序校验
npm install -D \
    stylelint \
    stylelint-config-standard \
    stylelint-webpack-plugin \
    stylelint-config-idiomatic-order \
    stylelint-order
```



# 添加stylelint配置文件

```
安装
// stylelint-order 属性顺序校验
npm install -D \
    stylelint \
    stylelint-config-standard \
    stylelint-webpack-plugin \
    stylelint-config-idiomatic-order \
    stylelint-order
添加stylelint配置文件
/**
 * 配置方式一：stylelint-config-idiomatic-order，无需指定plugins与额外的rules
 * 配置方式二：设置plugins，stylelint-order，并手动指定order/properties-order
 */
module.exports = {
  extends: ['stylelint-config-standard', 'stylelint-config-idiomatic-order'],
  // plugins: ['stylelint-order']
  rules: {
    'font-family-no-missing-generic-family-keyword': [
      true,
      {
        ignoreFontFamilies: ['iconfont', /PingFang/]
      }
    ],
    'at-rule-no-unknown': [
      true,
      {
        ignoreAtRules: [/border/, 'primary-color']
      }
    ]
  //   'order/order': [
  //     'at-rules',
  //     'declarations',
  //     'custom-properties',
  //     'rules',
  //     'dollar-variables'
  //   ],
  //   'order/properties-order': [
  //     'position',
  //     'z-index',
  //     'top',
  //     'bottom',
  //     'left',
  //     'right',
  //     'float',
  //     'clear',
  //     'columns',
  //     'columns-width',
  //     'columns-count',
  //     'column-rule',
  //     'column-rule-width',
  //     'column-rule-style',
  //     'column-rule-color',
  //     'column-fill',
  //     'column-span',
  //     'column-gap',
  //     'display',
  //     'grid',
  //     'grid-template-rows',
  //     'grid-template-columns',
  //     'grid-template-areas',
  //     'grid-auto-rows',
  //     'grid-auto-columns',
  //     'grid-auto-flow',
  //     'grid-column-gap',
  //     'grid-row-gap',
  //     'grid-template',
  //     'grid-template-rows',
  //     'grid-template-columns',
  //     'grid-template-areas',
  //     'grid-gap',
  //     'grid-row-gap',
  //     'grid-column-gap',
  //     'grid-area',
  //     'grid-row-start',
  //     'grid-row-end',
  //     'grid-column-start',
  //     'grid-column-end',
  //     'grid-column',
  //     'grid-column-start',
  //     'grid-column-end',
  //     'grid-row',
  //     'grid-row-start',
  //     'grid-row-end',
  //     'flex',
  //     'flex-grow',
  //     'flex-shrink',
  //     'flex-basis',
  //     'flex-flow',
  //     'flex-direction',
  //     'flex-wrap',
  //     'justify-content',
  //     'align-content',
  //     'align-items',
  //     'align-self',
  //     'order',
  //     'table-layout',
  //     'empty-cells',
  //     'caption-side',
  //     'border-collapse',
  //     'border-spacing',
  //     'list-style',
  //     'list-style-type',
  //     'list-style-position',
  //     'list-style-image',
  //     'ruby-align',
  //     'ruby-merge',
  //     'ruby-position',
  //     'box-sizing',
  //     'width',
  //     'min-width',
  //     'max-width',
  //     'height',
  //     'min-height',
  //     'max-height',
  //     'padding',
  //     'padding-top',
  //     'padding-right',
  //     'padding-bottom',
  //     'padding-left',
  //     'margin',
  //     'margin-top',
  //     'margin-right',
  //     'margin-bottom',
  //     'margin-left',
  //     'border',
  //     'border-width',
  //     'border-top-width',
  //     'border-right-width',
  //     'border-bottom-width',
  //     'border-left-width',
  //     'border-style',
  //     'border-top-style',
  //     'border-right-style',
  //     'border-bottom-style',
  //     'border-left-style',
  //     'border-color',
  //     'border-top-color',
  //     'border-right-color',
  //     'border-bottom-color',
  //     'border-left-color',
  //     'border-image',
  //     'border-image-source',
  //     'border-image-slice',
  //     'border-image-width',
  //     'border-image-outset',
  //     'border-image-repeat',
  //     'border-top',
  //     'border-top-width',
  //     'border-top-style',
  //     'border-top-color',
  //     'border-top',
  //     'border-right-width',
  //     'border-right-style',
  //     'border-right-color',
  //     'border-bottom',
  //     'border-bottom-width',
  //     'border-bottom-style',
  //     'border-bottom-color',
  //     'border-left',
  //     'border-left-width',
  //     'border-left-style',
  //     'border-left-color',
  //     'border-radius',
  //     'border-top-right-radius',
  //     'border-bottom-right-radius',
  //     'border-bottom-left-radius',
  //     'border-top-left-radius',
  //     'outline',
  //     'outline-width',
  //     'outline-color',
  //     'outline-style',
  //     'outline-offset',
  //     'overflow',
  //     'overflow-x',
  //     'overflow-y',
  //     'resize',
  //     'visibility',
  //     'font',
  //     'font-style',
  //     'font-variant',
  //     'font-weight',
  //     'font-stretch',
  //     'font-size',
  //     'font-family',
  //     'font-synthesis',
  //     'font-size-adjust',
  //     'font-kerning',
  //     'line-height',
  //     'text-align',
  //     'text-align-last',
  //     'vertical-align',
  //     'text-overflow',
  //     'text-justify',
  //     'text-transform',
  //     'text-indent',
  //     'text-emphasis',
  //     'text-emphasis-style',
  //     'text-emphasis-color',
  //     'text-emphasis-position',
  //     'text-decoration',
  //     'text-decoration-color',
  //     'text-decoration-style',
  //     'text-decoration-line',
  //     'text-underline-position',
  //     'text-shadow',
  //     'white-space',
  //     'overflow-wrap',
  //     'word-wrap',
  //     'word-break',
  //     'line-break',
  //     'hyphens',
  //     'letter-spacing',
  //     'word-spacing',
  //     'quotes',
  //     'tab-size',
  //     'orphans',
  //     'writing-mode',
  //     'text-combine-upright',
  //     'unicode-bidi',
  //     'text-orientation',
  //     'direction',
  //     'text-rendering',
  //     'font-feature-settings',
  //     'font-language-override',
  //     'image-rendering',
  //     'image-orientation',
  //     'image-resolution',
  //     'shape-image-threshold',
  //     'shape-outside',
  //     'shape-margin',
  //     'color',
  //     'background',
  //     'background-image',
  //     'background-position',
  //     'background-size',
  //     'background-repeat',
  //     'background-origin',
  //     'background-clip',
  //     'background-attachment',
  //     'background-color',
  //     'background-blend-mode',
  //     'isolation',
  //     'clip-path',
  //     'mask',
  //     'mask-image',
  //     'mask-mode',
  //     'mask-position',
  //     'mask-size',
  //     'mask-repeat',
  //     'mask-origin',
  //     'mask-clip',
  //     'mask-composite',
  //     'mask-type',
  //     'filter',
  //     'box-shadow',
  //     'opacity',
  //     'transform-style',
  //     'transform',
  //     'transform-box',
  //     'transform-origin',
  //     'perspective',
  //     'perspective-origin',
  //     'backface-visibility',
  //     'transition',
  //     'transition-property',
  //     'transition-duration',
  //     'transition-timing-function',
  //     'transition-delay',
  //     'animation',
  //     'animation-name',
  //     'animation-duration',
  //     'animation-timing-function',
  //     'animation-delay',
  //     'animation-iteration-count',
  //     'animation-direction',
  //     'animation-fill-mode',
  //     'animation-play-state',
  //     'scroll-behavior',
  //     'scroll-snap-type',
  //     'scroll-snap-destination',
  //     'scroll-snap-coordinate',
  //     'cursor',
  //     'touch-action',
  //     'caret-color',
  //     'ime-mode',
  //     'object-fit',
  //     'object-position',
  //     'content',
  //     'counter-reset',
  //     'counter-increment',
  //     'will-change',
  //     'pointer-events',
  //     'all',
  //     'page-break-before',
  //     'page-break-after',
  //     'page-break-inside',
  //     'widows'
  //   ]
  }
}

配置vue.config.js
// 添加StyleLintPlugin插件
const StyleLintPlugin = require('stylelint-webpack-plugin')
module.exports = {
  // ...省略配置
  configureWebpack: {
    plugins: [
      // ...其他插件
      new StyleLintPlugin({
        files: ['**/*.{vue,htm,html,css,sss,less,scss,sass}'],
        fix: false, // 是否自动修复
        cache: true, // 是否缓存
        emitErrors: true,
        failOnError: false
      })
    ]
 }
 // ...省略配置
}
配置package.json
配置scripts
// scripts 添加以下命令
"lint:style": "stylelint **/*.{vue,css,less} --custom-syntax postcss-html --fix"


配置git hook
// package.json
{
    // ...
    "lint-staged": {
        "*.{vue,js}": [
            "eslint --fix",
            "git add"
        ],
        "*.{html,vue,css,sass,scss}": [
            "stylelint --fix",
            "git add",
        ]
    },
    "husky": {
        "hooks": {
            "pre-commit": "lint-staged",
        }
    }
}
配置vscode
安装插件stylelint


插件设置修改Custom Syntax


配置自动修复
// setting.json
"editor.formatOnSave": false,
"editor.codeActionsOnSave": {
        "source.fixAll.eslint": true,
        "source.fixAll.stylelint": true 
}
注意
setting.json文件中，不要设置config.stylelint，否则不会读取这里的配置文件
配置文件中添加以下代码

// setting.json
"css.validate": false,
"less.validate": false,
"scss.validate": false

```



# 配置vue.config.js

```
// 添加StyleLintPlugin插件
const StyleLintPlugin = require('stylelint-webpack-plugin')
module.exports = {
  // ...省略配置
  configureWebpack: {
    plugins: [
      // ...其他插件
      new StyleLintPlugin({
        files: ['**/*.{vue,htm,html,css,sss,less,scss,sass}'],
        fix: false, // 是否自动修复
        cache: true, // 是否缓存
        emitErrors: true,
        failOnError: false
      })
    ]
 }
 // ...省略配置
}
```



# 配置package.json

### 配置scripts

```
// scripts 添加以下命令
"lint:style": "stylelint **/*.{vue,css,less} --custom-syntax postcss-html --fix"
```

### 配置git hook

```
// package.json
{
    // ...
    "lint-staged": {
        "*.{vue,js}": [
            "eslint --fix",
            "git add"
        ],
        "*.{html,vue,css,sass,scss}": [
            "stylelint --fix",
            "git add",
        ]
    },
    "husky": {
        "hooks": {
            "pre-commit": "lint-staged",
        }
    }
}
```



# 配置vscode

### 安装插件stylelint

<img src="/Users/croon/Library/Application Support/typora-user-images/截屏2021-03-11 下午9.01.13.png" alt="截屏2021-03-11 下午9.01.13" style="zoom:50%;" />

### 插件设置修改Custom Syntax

<img src="/Users/croon/Library/Application Support/typora-user-images/截屏2021-03-11 下午9.01.27.png" alt="截屏2021-03-11 下午9.01.27" style="zoom:50%;" />

### 配置自动修复

```
// setting.json
"editor.formatOnSave": false,
"editor.codeActionsOnSave": {
        "source.fixAll.eslint": true,
        "source.fixAll.stylelint": true 
}
```

### 注意

setting.json文件中，不要设置config.stylelint，否则不会读取这里的配置文件
配置文件中添加以下代码

```
// setting.json
"css.validate": false,
"less.validate": false,
"scss.validate": false
```





![截屏2021-03-11 下午9.02.19](/Users/croon/Library/Application Support/typora-user-images/截屏2021-03-11 下午9.02.19.png)

