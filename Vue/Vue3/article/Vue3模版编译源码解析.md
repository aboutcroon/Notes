我们知道，当以无构建步骤方式使用 Vue 时，组件模板要么是写在页面的 HTML 中，要么是内联的 JavaScript  字符串。在这些场景中，为了执行动态模板编译，Vue  需要将模板编译器运行在浏览器中。

但当我们日常使用 Vue 的时候，模版编译部分的内容基本上都没接触到过。因为这时模版编译的步骤是在开发环境下完成的。如果我们使用了构建步骤，由于提前编译了模板，那么就无须再在浏览器中运行了。这时模版编译是在我们项目打包时由 vite 或 webpack 的 vue-loader 完成的。

虽然模版编译已不必存在于项目的运行过程中，但是理解模版编译的原理，对我们日常的 vue 开发是有很大帮助的。在我们写模版语法时，就能同时知道哪些地方是可以做优化的。

接下来，我们就来逐步分析一下 Vue3 中的模版编译过程。

## 模版编译的入口

在 Vue3 中，编译场景分为 `服务端 SSR 编译` 和 `web 编译`，本文我们着重分析的是 web 端的编译。

首先，从编译入口 compile 开始

```js
// web 模版编译的入口
export function compile(
  template: string,
  options: CompilerOptions = {}
): CodegenResult {
  return baseCompile(
    template,
    extend({}, parserOptions, options, {
      nodeTransforms: [
        // ignore <script> and <tag>
        // this is not put inside DOMNodeTransforms because that list is used
        // by compiler-ssr to generate vnode fallback branches
        ignoreSideEffectTags,
        ...DOMNodeTransforms,
        ...(options.nodeTransforms || [])
      ],
      directiveTransforms: extend(
        {},
        DOMDirectiveTransforms,
        options.directiveTransforms || {}
      ),
      transformHoist: __BROWSER__ ? null : stringifyStatic
    })
  )
}
```

主要返回了一个 baseCompile 函数，主要的编译流程都在 baseCompile 函数中

```js
// 真正的编译过程都在 baseCompile 函数里执行
export function baseCompile(
  template: string | RootNode,
  options: CompilerOptions = {}
): CodegenResult {
  const onError = options.onError || defaultOnError
  const isModuleMode = options.mode === 'module'
  /* istanbul ignore if */
  if (__BROWSER__) {
    if (options.prefixIdentifiers === true) {
      onError(createCompilerError(ErrorCodes.X_PREFIX_ID_NOT_SUPPORTED))
    } else if (isModuleMode) {
      onError(createCompilerError(ErrorCodes.X_MODULE_MODE_NOT_SUPPORTED))
    }
  }

  const prefixIdentifiers =
    !__BROWSER__ && (options.prefixIdentifiers === true || isModuleMode)
  if (!prefixIdentifiers && options.cacheHandlers) {
    onError(createCompilerError(ErrorCodes.X_CACHE_HANDLER_NOT_SUPPORTED))
  }
  if (options.scopeId && !isModuleMode) {
    onError(createCompilerError(ErrorCodes.X_SCOPE_ID_NOT_SUPPORTED))
  }

  // 1. 解析 template 生成 AST
  const ast = isString(template) ? baseParse(template, options) : template
  // 2. AST 转换（为了添加编译优化的相关属性）
  const [nodeTransforms, directiveTransforms] =
    getBaseTransformPreset(prefixIdentifiers)
  transform( 
    ast,
    extend({}, options, {
      prefixIdentifiers,
      nodeTransforms: [
        ...nodeTransforms,
        ...(options.nodeTransforms || []) // user transforms 用户自定义的 transform
      ],
      directiveTransforms: extend(
        {},
        directiveTransforms,
        options.directiveTransforms || {} // user transforms 用户自定义的 transform
      )
    })
  )
  
  // 3. 生成代码
  return generate(
    ast,
    extend({}, options, {
      prefixIdentifiers
    })
  )
}
```

源码很长，我们可以总结一下，整个编译流程主要分为三大块：

1. 解析 template 生成 AST 树
2. 对 AST 树做转换
3. 生成代码

我们接下来就从这三块内容深入分析。

## 1. 解析 template 生成 AST 树

解析 template 生成 AST 树，又可以称为 `parse 阶段`。该过程是`词法分析过程，为了构造基础的 AST 节点对象`。

该阶段会将 template 进行 parse，parse 之后生成 AST 语法树。在 JS 中，AST 语法树可以用对象来表示，但 AST 语法树不是 JS 中的数据结构，而是一个各种语言中都有的模型，像我们使用 babel 编译新语法时，也会先转换成 AST。AST 语法树的应用场景十分广泛。

### AST 对象

我们先看一段示例，比如我们有如下的 template

```html
<div class="app"> 
  <!-- 这是一段注释 --> 
  <hello> 
    <p>{{ msg }}</p> 
  </hello> 
  <p>This is an app</p> 
</div>
```

它经过 parse 阶段后，会生成如下 AST 对象

```json
{ 
  "type": 0, 
  "children": [ 
    { 
      "type": 1, 
      "ns": 0, 
      "tag": "div", 
      "tagType": 0, 
      "props": [ 
        { 
          "type": 6, 
          "name": "class", 
          "value": { 
            "type": 2, 
            "content": "app", 
            "loc": { 
              "start": { 
                "column": 12, 
                "line": 1, 
                "offset": 11 
              }, 
              "end": { 
                "column": 17, 
                "line": 1, 
                "offset": 16 
              }, 
              "source": "\"app\"" 
            } 
          }, 
          "loc": { 
            "start": { 
              "column": 6, 
              "line": 1, 
              "offset": 5 
            }, 
            "end": { 
              "column": 17, 
              "line": 1, 
              "offset": 16 
            }, 
            "source": "class=\"app\"" 
          } 
        } 
      ], 
      "isSelfClosing": false, 
      "children": [ 
        { 
          "type": 3, 
          "content": " 这是一段注释 ", 
          "loc": { 
            "start": { 
              "column": 3, 
              "line": 2, 
              "offset": 20 
            }, 
            "end": { 
              "column": 18, 
              "line": 2, 
              "offset": 35 
            }, 
            "source": "<!-- 这是一段注释 -->" 
          } 
        }, 
        { 
          "type": 1, 
          "ns": 0, 
          "tag": "hello", 
          "tagType": 1, 
          "props": [], 
          "isSelfClosing": false, 
          "children": [ 
            { 
              "type": 1, 
              "ns": 0, 
              "tag": "p", 
              "tagType": 0, 
              "props": [], 
              "isSelfClosing": false, 
              "children": [ 
                { 
                  "type": 5, 
                  "content": { 
                    "type": 4, 
                    "isStatic": false, 
                    "isConstant": false, 
                    "content": "msg", 
                    "loc": { 
                      "start": { 
                        "column": 11, 
                        "line": 4, 
                        "offset": 56 
                      }, 
                      "end": { 
                        "column": 14, 
                        "line": 4, 
                        "offset": 59 
                      }, 
                      "source": "msg" 
                    } 
                  }, 
                  "loc": { 
                    "start": { 
                      "column": 8, 
                      "line": 4, 
                      "offset": 53 
                    }, 
                    "end": { 
                      "column": 17, 
                      "line": 4, 
                      "offset": 62 
                    }, 
                    "source": "{{ msg }}" 
                  } 
                } 
              ], 
              "loc": { 
                "start": { 
                  "column": 5, 
                  "line": 4, 
                  "offset": 50 
                }, 
                "end": { 
                  "column": 21, 
                  "line": 4, 
                  "offset": 66 
                }, 
                "source": "<p>{{ msg }}</p>" 
              } 
            } 
          ], 
          "loc": { 
            "start": { 
              "column": 3, 
              "line": 3, 
              "offset": 38 
            }, 
            "end": { 
              "column": 11, 
              "line": 5, 
              "offset": 77 
            }, 
            "source": "<hello>\n    <p>{{ msg }}</p>\n  </hello>" 
          } 
        }, 
        { 
          "type": 1, 
          "ns": 0, 
          "tag": "p", 
          "tagType": 0, 
          "props": [], 
          "isSelfClosing": false, 
          "children": [ 
            { 
              "type": 2, 
              "content": "This is an app", 
              "loc": { 
                "start": { 
                  "column": 6, 
                  "line": 6, 
                  "offset": 83 
                }, 
                "end": { 
                  "column": 20, 
                  "line": 6, 
                  "offset": 97 
                }, 
                "source": "This is an app" 
              } 
            } 
          ], 
          "loc": { 
            "start": { 
              "column": 3, 
              "line": 6, 
              "offset": 80 
            }, 
            "end": { 
              "column": 24, 
              "line": 6, 
              "offset": 101 
            }, 
            "source": "<p>This is an app</p>" 
          } 
        } 
      ], 
      "loc": { 
        "start": { 
          "column": 1, 
          "line": 1, 
          "offset": 0 
        }, 
        "end": { 
          "column": 7, 
          "line": 7, 
          "offset": 108 
        }, 
        "source": "<div class=\"app\">\n  <!-- 这是一段注释 -->\n  <hello>\n    <p>{{ msg }}</p>\n  </hello>\n  <p>This is an app</p>\n</div>" 
      } 
    } 
  ], 
  "helpers": [], 
  "components": [], 
  "directives": [], 
  "hoists": [], 
  "imports": [], 
  "cached": 0, 
  "temps": 0, 
  "loc": { 
    "start": { 
      "column": 1, 
      "line": 1, 
      "offset": 0 
    }, 
    "end": { 
      "column": 7, 
      "line": 7, 
      "offset": 108 
    }, 
    "source": "<div class=\"app\">\n  <!-- 这是一段注释 -->\n  <hello>\n    <p>{{ msg }}</p>\n  </hello>\n  <p>This is an app</p>\n</div>" 
  } 
}
```

这个 AST 对象可以完整地描述它在模板中映射的节点信息。

以下是 AST 对象中的常见字段：

- type 字段是节点的类型
- tag 字段是节点的标签
- props 字段是节点的属性
- loc 字段是节点对应代码的相关信息
- children 字段指向它的子节点的对象数组
- ...

这里要注意，**AST 对象根节点其实是一个虚拟节点**，**它并不会映射到一个具体节点**。设计这么一个虚拟节点的目的是什么呢？

因为 Vue3 和 Vue2 有一个很大的不同——Vue3 支持了 Fragment 的语法，即组件可以有多个根节点。但是对于一棵树而言，必须有一个根节点，所以虚拟节点在这种场景下就非常有用了，它可以作为 AST 的根节点，然后其 children 则包含了真实的多个根节点。

### 主要解析流程

大致了解了 AST 之后，接下来我们看一下如何根据模板字符串 template 来构建这个 AST 对象。

parse 阶段的入口是从 baseParse 函数开始

```js
export function baseParse(
  content: string,
  options: ParserOptions = {}
): RootNode {
  // 创建解析上下文
  const context = createParserContext(content, options)
  const start = getCursor(context)
  // 解析子节点，并创建 AST 根节点
  return createRoot(
    parseChildren(context, TextModes.DATA, []),
    getSelection(context, start)
  )
}
```

这个函数主要做了三件事情：

1. 创建解析上下文 context
2. 解析子节点
3. 创建 AST 根节点

#### 创建解析上下文 context

通过 createParserContext 函数

```js
// 默认解析配置 
const defaultParserOptions = { 
  delimiters: [`{{`, `}}`], 
  getNamespace: () => 0 /* HTML */, 
  getTextMode: () => 0 /* DATA */, 
  isVoidTag: NO, 
  isPreTag: NO, 
  isCustomElement: NO, 
  decodeEntities: (rawText) => rawText.replace(decodeRE, (_, p1) => decodeMap[p1]), 
  onError: defaultOnError 
} 
function createParserContext(content, options) { 
  return { 
    options: extend({}, defaultParserOptions, options), 
    column: 1, 
    line: 1, 
    offset: 0, 
    originalSource: content, 
    source: content, 
    inPre: false, 
    inVPre: false 
  } 
}
```

context 就是返回的这个 JS 对象，它维护着解析过程中的上下文。

- options：解析的相关配置
- column：当前代码的列号
- line：当前代码的行号
- originalSource：最初的原始代码
- source：当前代码
- offset：当前代码相对于原始代码的偏移量
- inPre：当前代码是否在 pre 标签内
- inVpre：当前代码是否在 v-pre 指令的环境下

后续解析的过程中，会始终维护和更新这个 context，它能够表示当前解析的状态。

#### 解析子节点

先整体看一下 parseChildren 函数的结构

```js
function parseChildren(context, mode, ancestors) { 
  const parent = last(ancestors) 
  const ns = parent ? parent.ns : 0 /* HTML */ 
  const nodes = [] 

  // 自顶向下分析代码，生成 nodes 

  let removedWhitespace = false 
  // 空白字符管理 

  return removedWhitespace ? nodes.filter(Boolean) : nodes 
} 
```

这是一个核心的过程，整个过程的目的是为了解析并创建 AST 节点数组 nodes。

首先，自顶向下分析代码，在解析的过程中，根据不同的情况生成不同类型的节点 node，最后放入 AST 节点数组 nodes。

下面是生成 AST 节点数组的流程

```js
function parseChildren(context, mode, ancestors) { 
  // 父节点 
  const parent = last(ancestors) 
  const ns = parent ? parent.ns : 0 /* HTML */ 
  const nodes = [] 
  // 判断是否遍历结束 
  while (!isEnd(context, mode, ancestors)) { 
    const s = context.source 
    let node = undefined 
    if (mode === 0 /* DATA */ || mode === 1 /* RCDATA */) { 
      if (!context.inVPre && startsWith(s, context.options.delimiters[0])) { 
        // 处理 {{ 插值代码 
        node = parseInterpolation(context, mode) 
      } 
      else if (mode === 0 /* DATA */ && s[0] === '<') { 
        // 处理 < 开头的代码 
        if (s.length === 1) { 
          // s 长度为 1，说明代码结尾是 <，报错 
          emitError(context, 5 /* EOF_BEFORE_TAG_NAME */, 1) 
        } 
        else if (s[1] === '!') { 
          // 处理 <! 开头的代码 
          if (startsWith(s, '<!--')) { 
            // 处理注释节点 
            node = parseComment(context) 
          } 
          else if (startsWith(s, '<!DOCTYPE')) { 
            // 处理 <!DOCTYPE 节点 
            node = parseBogusComment(context) 
          } 
          else if (startsWith(s, '<![CDATA[')) { 
            // 处理 <![CDATA[ 节点 
            if (ns !== 0 /* HTML */) { 
              node = parseCDATA(context, ancestors) 
            } 
            else { 
              emitError(context, 1 /* CDATA_IN_HTML_CONTENT */) 
              node = parseBogusComment(context) 
            } 
          } 
          else { 
            emitError(context, 11 /* INCORRECTLY_OPENED_COMMENT */) 
            node = parseBogusComment(context) 
          } 
        } 
        else if (s[1] === '/') { 
          // 处理 </ 结束标签 
          if (s.length === 2) { 
            // s 长度为 2，说明代码结尾是 </，报错 
            emitError(context, 5 /* EOF_BEFORE_TAG_NAME */, 2) 
          } 
          else if (s[2] === '>') { 
            // </> 缺少结束标签，报错 
            emitError(context, 14 /* MISSING_END_TAG_NAME */, 2) 
            advanceBy(context, 3) 
            continue 
          } 
          else if (/[a-z]/i.test(s[2])) { 
            // 多余的结束标签 
            emitError(context, 23 /* X_INVALID_END_TAG */) 
            parseTag(context, 1 /* End */, parent) 
            continue 
          } 
          else { 
            emitError(context, 12 /* INVALID_FIRST_CHARACTER_OF_TAG_NAME */, 2) 
            node = parseBogusComment(context) 
          } 
        } 
        else if (/[a-z]/i.test(s[1])) { 
          // 解析标签元素节点 
          node = parseElement(context, ancestors) 
        } 
        else if (s[1] === '?') { 
          emitError(context, 21 /* UNEXPECTED_QUESTION_MARK_INSTEAD_OF_TAG_NAME */, 1) 
          node = parseBogusComment(context) 
        } 
        else { 
          emitError(context, 12 /* INVALID_FIRST_CHARACTER_OF_TAG_NAME */, 1) 
        } 
      } 
    } 
    if (!node) { 
      // 解析普通文本节点 
      node = parseText(context, mode) 
    } 
    if (isArray(node)) { 
      // 如果 node 是数组，则遍历添加 
      for (let i = 0; i < node.length; i++) { 
        pushNode(nodes, node[i]) 
      } 
    } 
    else { 
      // 添加单个 node 
      pushNode(nodes, node) 
    } 
  } 
}
```

解析上下文 context 的状态会在解析过程中不断的发生变化，我们可以通过 context.source 拿到当前解析剩余的代码 s，然后根据 s 进行正则匹配，匹配开始的标签，然后根据不同的情况走不同的分支处理逻辑。

在解析的过程中，可能会遇到各种错误，都会通过 emitError 方法报错。

##### 注释节点的解析

我们先看一下其中注释节点的解析过程

```js
function parseComment(context) { 
  const start = getCursor(context) 
  let content 
  // 常规注释的结束符 
  const match = /--(\!)?>/.exec(context.source) 
  if (!match) { 
    // 没有匹配的注释结束符 
    content = context.source.slice(4) 
    advanceBy(context, context.source.length) 
    emitError(context, 7 /* EOF_IN_COMMENT */) 
  } 
  else { 
    if (match.index <= 3) { 
      // 非法的注释符号 
      emitError(context, 0 /* ABRUPT_CLOSING_OF_EMPTY_COMMENT */) 
    } 
    if (match[1]) { 
      // 注释结束符不正确 
      emitError(context, 10 /* INCORRECTLY_CLOSED_COMMENT */) 
    } 
    // 获取注释的内容 
    content = context.source.slice(4, match.index) 
    // 截取到注释结尾之间的代码，用于后续判断嵌套注释 
    const s = context.source.slice(0, match.index) 
    let prevIndex = 1, nestedIndex = 0 
    // 判断嵌套注释符的情况，存在即报错 
    while ((nestedIndex = s.indexOf('<!--', prevIndex)) !== -1) { 
      advanceBy(context, nestedIndex - prevIndex + 1) 
      if (nestedIndex + 4 < s.length) { 
        emitError(context, 16 /* NESTED_COMMENT */) 
      } 
      prevIndex = nestedIndex + 1 
    } 
    // 前进代码到注释结束符后 
    advanceBy(context, match.index + match[0].length - prevIndex + 1) 
  } 
  return { 
    type: 3 /* COMMENT */, 
    content, 
    loc: getSelection(context, start) 
  } 
}
```

首先它会利用注释结束符的正则表达式去匹配代码（进入 parseComment 前是用正则匹配注释开始符），紧接着获取开始与结束符中间的内容，最后再通过调用 advanceBy 前进代码到注释结束符后。

最终返回的就是一个描述注释节点的对象。type 表示它是一个注释节点，content 表示注释的内容，loc 表示注释的代码开头和结束的位置信息。

##### 插值的解析

接下来，我们来看插值的解析过程（如果标签上指定 v-pre，则会跳过插值的解析，因为这样会标明它是静态的不变的标签）

```js
function parseInterpolation(context, mode) { 
  // 从配置中获取插值开始和结束分隔符，默认是 {{ 和 }} 
  const [open, close] = context.options.delimiters 
  const closeIndex = context.source.indexOf(close, open.length) 
  if (closeIndex === -1) { 
    emitError(context, 25 /* X_MISSING_INTERPOLATION_END */) 
    return undefined 
  } 
  const start = getCursor(context) 
  // 代码前进到插值开始分隔符后 
  advanceBy(context, open.length) 
  // 内部插值开始位置 
  const innerStart = getCursor(context) 
  // 内部插值结束位置 
  const innerEnd = getCursor(context) 
  // 插值原始内容的长度 
  const rawContentLength = closeIndex - open.length 
  // 插值原始内容 
  const rawContent = context.source.slice(0, rawContentLength) 
  // 获取插值的内容，并前进代码到插值的内容后 
  const preTrimContent = parseTextData(context, rawContentLength, mode) 
  const content = preTrimContent.trim() 
  // 内容相对于插值开始分隔符的头偏移 
  const startOffset = preTrimContent.indexOf(content) 
  if (startOffset > 0) { 
    // 更新内部插值开始位置 
    advancePositionWithMutation(innerStart, rawContent, startOffset) 
  } 
  // 内容相对于插值结束分隔符的尾偏移 
  const endOffset = rawContentLength - (preTrimContent.length - content.length - startOffset) 
  // 更新内部插值结束位置 
  advancePositionWithMutation(innerEnd, rawContent, endOffset); 
  // 前进代码到插值结束分隔符后 
  advanceBy(context, close.length) 
  return { 
    type: 5 /* INTERPOLATION */, 
    content: { 
      type: 4 /* SIMPLE_EXPRESSION */, 
      isStatic: false, 
      isConstant: false, 
      content, 
      loc: getSelection(context, innerStart, innerEnd) 
    }, 
    loc: getSelection(context, start) 
  } 
}
```

parseInterpolation 的实现也很简单，首先它会尝试找插值的结束分隔符，如果找不到则报错。

如果找到，则前进代码到插值开始分隔符后，然后通过 parseTextData 获取插值中间的内容并前进代码到插值内容后。整个过程比较简单。

##### 普通文本的解析

普通静态文本的解析也是通过执行 parseTextData 获取文本的内容。与插值大同小异，且更简单，这里不做过多分析。

##### 元素节点的解析

元素节点的解析就复杂一点，因为其下面会包含子节点，需要不断递归解析。

```js
function parseElement(context, ancestors) {
  // 是否在 pre 标签内
  const wasInPre = context.inPre
  // 是否在 v-pre 指令内
  const wasInVPre = context.inVPre
  // 获取当前元素的父标签节点
  const parent = last(ancestors)
  // 解析开始标签，生成一个标签节点，并前进代码到开始标签后
  const element = parseTag(context, 0 /* Start */, parent)
  // 是否在 pre 标签的边界
  const isPreBoundary = context.inPre && !wasInPre
  // 是否在 v-pre 指令的边界
  const isVPreBoundary = context.inVPre && !wasInVPre
  if (element.isSelfClosing || context.options.isVoidTag(element.tag)) {
    // 如果是自闭和标签，直接返回标签节点
    return element
  }
  // 下面是处理子节点的逻辑
  // 先把标签节点添加到 ancestors，入栈
  ancestors.push(element)
  const mode = context.options.getTextMode(element, parent)
  // 递归解析子节点，传入 ancestors
  const children = parseChildren(context, mode, ancestors)
  // ancestors 出栈
  ancestors.pop()
  // 添加到 children 属性中
  element.children = children
  // 结束标签
  if (startsWithEndTagOpen(context.source, element.tag)) {
    // 解析结束标签，并前进代码到结束标签后
    parseTag(context, 1 /* End */, parent)
  }
  else {
    emitError(context, 24 /* X_MISSING_END_TAG */, 0, element.loc.start);
    if (context.source.length === 0 && element.tag.toLowerCase() === 'script') {
      const first = children[0];
      if (first && startsWith(first.loc.source, '<!--')) {
        emitError(context, 8 /* EOF_IN_SCRIPT_HTML_COMMENT_LIKE_TEXT */)
      }
    }
  }
  // 更新标签节点的代码位置，结束位置到结束标签后
  element.loc = getSelection(context, element.loc.start)
  if (isPreBoundary) {
    context.inPre = false
  }
  if (isVPreBoundary) {
    context.inVPre = false
  }
  return element
}
```

主要分为三个部分：

1. 解析开始标签
2. 递归执行 parseElement 解析子节点
3. 解析闭合标签

执行 parseElement 时，会利用栈结构 ancestors，每个子节点解析完毕后，我们就把当前节点出栈。

通过不断地递归解析，我们就可以完整地解析整个模板，并且标签类型的 AST 节点会保持对子节点数组的引用。
这样就构成了一个树形的数据结构，所以整个解析过程构造出的 AST 节点数组就能很好地映射整个模板的 DOM 结构。

##### 空白字符管理

当我们解析完子节点，会进行空白字符管理，用于提高编译的效率。其主要内容就是删除掉一些空白字符。比如 div 标签到下一行会有一个换行符，并且每个开始标签前面也有空白字符。

#### 创建 AST 根节点

解析子节点的步骤过后，进入到创建 AST 根节点的流程，这个过程比较简单

```js
function createRoot(children, loc = locStub) {
  return {
    type: 0 /* ROOT */,
    children,
    helpers: [],
    components: [],
    directives: [],
    hoists: [],
    imports: [],
    cached: 0,
    temps: 0,
    codegenNode: undefined,
    loc
  }
}
```

会创建一个 type 为 0 的根节点，这个根节点其实是一个虚拟节点。因为 Vue3 支持了 Fragment 的写法，允许在 template 中有多个根节点。但是实际上一个树必须有一个根节点，所以这里就创建了一个虚拟的根节点，用于更好的进行模版编译。

## 2. AST 转换

template 的解析过后，最终拿到了一个 AST 节点对象。这个对象是对模板的完整描述，但是它还不能直接拿来生成代码，因为它的语义化还不够，没有包含和编译优化的相关属性，所以还需要进一步转换。

AST 转换是 transform 阶段，是语法分析阶段，把 AST 节点做一层转换，构造出语义化更强，信息更加丰富的 codegenCode，它在后续的代码生成阶段起着非常重要的作用。所以，AST 转换主要是为了添加 `编译优化` 相关的属性。

这个过程比较复杂，分支较多。

我们首先会通过 getBaseTransformPreset 方法获取节点和指令转换的方法，然后调用 transform 方法做 AST 转换，并且把这些节点和指令的转换方法作为配置的属性参数传入。

```js
// 获取节点和指令转换的方法
const [nodeTransforms, directiveTransforms] = getBaseTransformPreset()
// AST 转换
transform(ast, extend({}, options, {
  prefixIdentifiers,
  nodeTransforms: [
    ...nodeTransforms,
    ...(options.nodeTransforms || []) // 用户自定义  transforms
  ],
  directiveTransforms: extend({}, directiveTransforms, options.directiveTransforms || {} // 用户自定义 transforms
  )
}))
```

我们不需要进一步去看每个转换函数的实现，只要大致了解有哪些转换函数即可，这些转换函数会在后续执行 transform 的时候调用。

我们主要来看 transform 函数的实现：

```js
function transform(root, options) {
  const context = createTransformContext(root, options)
  traverseNode(root, context)
  if (options.hoistStatic) {
    hoistStatic(root, context)
  }
  if (!options.ssr) {
    createRootCodegen(root, context)
  }
  root.helpers = [...context.helpers]
  root.components = [...context.components]
  root.directives = [...context.directives]
  root.imports = [...context.imports]
  root.hoists = context.hoists
  root.temps = context.temps
  root.cached = context.cached
}
```

transform 的核心流程主要有四步：

1. 创建 transform 上下文
2. 遍历 AST 节点
3. 静态提升
4. 创建根代码生成节点

接下来，我们就好好分析一下每一步主要做了什么。

### 创建 transform 上下文

和 parse 过程一样，在 transform 阶段也会创建一个上下文对象，它的实现过程是这样的：

```js
function createTransformContext(root, { prefixIdentifiers = false, hoistStatic = false, cacheHandlers = false, nodeTransforms = [], directiveTransforms = {}, transformHoist = null, isBuiltInComponent = NOOP, expressionPlugins = [], scopeId = null, ssr = false, onError = defaultOnError }) {
  const context = {
    // 配置
    prefixIdentifiers,
    hoistStatic,
    cacheHandlers,
    nodeTransforms,
    directiveTransforms,
    transformHoist,
    isBuiltInComponent,
    expressionPlugins,
    scopeId,
    ssr,
    onError,
    // 状态数据
    root,
    helpers: new Set(),
    components: new Set(),
    directives: new Set(),
    hoists: [],
    imports: new Set(),
    temps: 0,
    cached: 0,
    identifiers: {},
    scopes: {
      vFor: 0,
      vSlot: 0,
      vPre: 0,
      vOnce: 0
    },
    parent: null,
    currentNode: root,
    childIndex: 0,
    // methods
    helper(name) {
      context.helpers.add(name)
      return name
    },
    helperString(name) {
      return `_${helperNameMap[context.helper(name)]}`
    },
    replaceNode(node) {
      context.parent.children[context.childIndex] = context.currentNode = node
    },
    removeNode(node) {
      const list = context.parent.children
      const removalIndex = node
        ? list.indexOf(node)
        : context.currentNode
          ? context.childIndex
          : -1
      if (!node || node === context.currentNode) {
        // 移除当前节点
        context.currentNode = null
        context.onNodeRemoved()
      }
      else {
        // 移除兄弟节点
        if (context.childIndex > removalIndex) {
          context.childIndex--
          context.onNodeRemoved()
        }
      }
      // 移除节点
      context.parent.children.splice(removalIndex, 1)
    },
    onNodeRemoved: () => { },
    addIdentifiers(exp) {
    },
    removeIdentifiers(exp) {
    },
    hoist(exp) {
      context.hoists.push(exp)
      const identifier = createSimpleExpression(`_hoisted_${context.hoists.length}`, false, exp.loc, true)
      identifier.hoisted = exp
      return identifier
    },
    cache(exp, isVNode = false) {
      return createCacheExpression(++context.cached, exp, isVNode)
    }
  }
  return context
}
```

这个上下文对象 context 维护了 transform 过程的一些配置，比如前面的节点和指令的转换函数等，还维护了 transform 过程的一些状态数据，比如当前处理的 AST 节点，当前 AST 节点在子节点中的索引，以及当前 AST 节点的父节点等

### 遍历 AST 节点

遍历 AST 节点的过程很关键，因为核心的转换过程就是在遍历中实现的。

```js
function traverseNode(node, context) {
  context.currentNode = node
  // 节点转换函数
  const { nodeTransforms } = context
  const exitFns = []
  for (let i = 0; i < nodeTransforms.length; i++) {
    // 有些转换函数会设计一个退出函数，在处理完子节点后执行
    const onExit = nodeTransforms[i](node, context)
    if (onExit) {
      if (isArray(onExit)) {
        exitFns.push(...onExit)
      }
      else {
        exitFns.push(onExit)
      }
    }
    if (!context.currentNode) {
      // 节点被移除
      return
    }
    else {
      // 因为在转换的过程中节点可能被替换，恢复到之前的节点
      node = context.currentNode
    }
  }
  switch (node.type) {
    case 3 /* COMMENT */:
      if (!context.ssr) {
        // 需要导入 createComment 辅助函数
        context.helper(CREATE_COMMENT)
      }
      break
    case 5 /* INTERPOLATION */:
      // 需要导入 toString 辅助函数
      if (!context.ssr) {
        context.helper(TO_DISPLAY_STRING)
      }
      break
    case 9 /* IF */:
      // 递归遍历每个分支节点
      for (let i = 0; i < node.branches.length; i++) {
        traverseNode(node.branches[i], context)
      }
      break
    case 10 /* IF_BRANCH */:
    case 11 /* FOR */:
    case 1 /* ELEMENT */:
    case 0 /* ROOT */:
      // 遍历子节点
      traverseChildren(node, context)
      break
  }
  // 执行转换函数返回的退出函数
  let i = exitFns.length
  while (i--) {
    exitFns[i]()
  }
}
```

递归遍历 AST 节点，会针对每个不同类型的节点执行不同类型的转换函数，有些转换函数还会设计一个退出函数。

当你执行转换函数后，它会返回一个新函数，然后在你处理完子节点后再执行这些退出函数，这是因为有些逻辑的处理需要依赖子节点的处理结果才能继续执行。

主要看三种类型的转换函数。

#### Element 节点转换函数

```js
const transformElement = (node, context) => {
  if (!(node.type === 1 /* ELEMENT */ &&
    (node.tagType === 0 /* ELEMENT */ ||
      node.tagType === 1 /* COMPONENT */))) {
    return
  }
  // 返回退出函数，在所有子表达式处理并合并后执行
  return function postTransformElement() {
    // 转换的目标是创建一个实现 VNodeCall 接口的代码生成节点
    const { tag, props } = node
    const isComponent = node.tagType === 1 /* COMPONENT */
    const vnodeTag = isComponent
      ? resolveComponentType(node, context)
      : `"${tag}"`
    const isDynamicComponent = isObject(vnodeTag) && vnodeTag.callee === RESOLVE_DYNAMIC_COMPONENT
    // 属性
    let vnodeProps
    // 子节点
    let vnodeChildren
    // 标记更新的类型标识，用于运行时优化
    let vnodePatchFlag
    let patchFlag = 0
    // 动态绑定的属性
    let vnodeDynamicProps
    let dynamicPropNames
    let vnodeDirectives
    // 动态组件、svg、foreignObject 标签以及动态绑定 key prop 的节点都被视作一个 Block
    let shouldUseBlock =
      isDynamicComponent ||
      (!isComponent &&
        (tag === 'svg' ||
          tag === 'foreignObject' ||
          findProp(node, 'key', true)))
    // 处理 props
    if (props.length > 0) {
      const propsBuildResult = buildProps(node, context)
      vnodeProps = propsBuildResult.props
      patchFlag = propsBuildResult.patchFlag
      dynamicPropNames = propsBuildResult.dynamicPropNames
      const directives = propsBuildResult.directives
      vnodeDirectives =
        directives && directives.length
          ? createArrayExpression(directives.map(dir => buildDirectiveArgs(dir, context)))
          : undefined
    }
    // 处理 children
    if (node.children.length > 0) {
      if (vnodeTag === KEEP_ALIVE) {
        // 把 KeepAlive 看做是一个 Block，这样可以避免它的子节点的动态节点被父 Block 收集
        shouldUseBlock = true
        // 2. 确保它始终更新
        patchFlag |= 1024 /* DYNAMIC_SLOTS */
        if ((process.env.NODE_ENV !== 'production') && node.children.length > 1) {
          context.onError(createCompilerError(42 /* X_KEEP_ALIVE_INVALID_CHILDREN */, {
            start: node.children[0].loc.start,
            end: node.children[node.children.length - 1].loc.end,
            source: ''
          }))
        }
      }
      const shouldBuildAsSlots = isComponent &&
        // Teleport不是一个真正的组件，它有专门的运行时处理
        vnodeTag !== TELEPORT &&
        vnodeTag !== KEEP_ALIVE
      if (shouldBuildAsSlots) {
        // 组件有 children，则处理插槽
        const { slots, hasDynamicSlots } = buildSlots(node, context)
        vnodeChildren = slots
        if (hasDynamicSlots) {
          patchFlag |= 1024 /* DYNAMIC_SLOTS */
        }
      }
      else if (node.children.length === 1 && vnodeTag !== TELEPORT) {
        const child = node.children[0]
        const type = child.type
        const hasDynamicTextChild = type === 5 /* INTERPOLATION */ ||
          type === 8 /* COMPOUND_EXPRESSION */
        if (hasDynamicTextChild && !getStaticType(child)) {
          patchFlag |= 1 /* TEXT */
        }
        // 如果只是一个普通文本节点、插值或者表达式，直接把节点赋值给 vnodeChildren
        if (hasDynamicTextChild || type === 2 /* TEXT */) {
          vnodeChildren = child
        }
        else {
          vnodeChildren = node.children
        }
      }
      else {
        vnodeChildren = node.children
      }
    }
    // 处理 patchFlag 和 dynamicPropNames
    if (patchFlag !== 0) {
      if ((process.env.NODE_ENV !== 'production')) {
        if (patchFlag < 0) {
          vnodePatchFlag = patchFlag + ` /* ${PatchFlagNames[patchFlag]} */`
        }
        else {
          // 获取 flag 对应的名字，生成注释，方便理解生成代码对应节点的 pathFlag
          const flagNames = Object.keys(PatchFlagNames)
            .map(Number)
            .filter(n => n > 0 && patchFlag & n)
            .map(n => PatchFlagNames[n])
            .join(`, `)
          vnodePatchFlag = patchFlag + ` /* ${flagNames} */`
        }
      }
      else {
        vnodePatchFlag = String(patchFlag)
      }
      if (dynamicPropNames && dynamicPropNames.length) {
        vnodeDynamicProps = stringifyDynamicPropNames(dynamicPropNames)
      }
    }
    node.codegenNode = createVNodeCall(context, vnodeTag, vnodeProps, vnodeChildren, vnodePatchFlag, vnodeDynamicProps, vnodeDirectives, !!shouldUseBlock, false /* disableTracking */, node.loc)
  }
}
```

只有当 AST 节点是组件或者普通元素节点时，才会返回 postTransformElement 这个退出函数，而且它会在该节点的子节点逻辑处理完毕后执行。

这是为什么呢？

我们看一下主体流程。

1. 首先会判断这个节点是不是一个 Block 节点，这是为了运行时的更新优化，Vue.js 3.0 设计了一个 Block tree 的概念。

   Block tree 是一个将模版基于动态节点指令切割的嵌套区块，每个区块只需要以一个 Array 来追踪自身包含的动态节点。

   借助 Block tree，Vue.js 将 vnode 更新性能由与模版整体大小相关提升为与动态内容的数量相关，极大优化了 diff 的效率，模板的动静比越大，这个优化就会越明显。

   因此在编译阶段，我们需要找出哪些节点可以构成一个 Block，其中动态组件（模版根节点）、svg、foreignObject 标签以及动态绑定的 prop 的节点都被视作一个 Block。

2. 处理节点的 props

   从 AST 节点的 props 对象中进一步解析出指令 vnodeDirectives、动态属性 dynamicPropNames，以及更新标识 patchFlag。

   `patchFlag` 主要用于标识节点更新的类型，在组件更新的优化中会用到。

3. 处理节点的 children

   对于一个组件节点而言，如果它有子节点，则说明是组件的插槽，另外还会有对一些内置组件比如 KeepAlive、Teleport 的处理逻辑。

   对于一个普通元素节点，我们通常直接拿节点的 children 属性给 vnodeChildren 即可。

4. 对前面解析 props 求得的 patchFlag 和 dynamicPropNames 做进一步处理

   根据 patchFlag 的值从 PatchFlagNames 中获取 flag 对应的名字，从而生成注释。

   然后把数组 dynamicPropNames 转化生成 vnodeDynamicProps 字符串，便于后续对节点生成代码逻辑的处理。

5. 通过 createVNodeCall 创建实现 VNodeCall 接口的代码生成节点

   ```js
   function createVNodeCall(context, tag, props, children, patchFlag, dynamicProps, directives, isBlock = false, disableTracking = false, loc = locStub) {
     if (context) {
       if (isBlock) {
         context.helper(OPEN_BLOCK)
         context.helper(CREATE_BLOCK)
       }
       else {
         context.helper(CREATE_VNODE)
       }
       if (directives) {
         context.helper(WITH_DIRECTIVES) 
       }
     }
     return {
       type: 13 /* VNODE_CALL */,
       tag,
       props,
       children,
       patchFlag,
       dynamicProps,
       directives,
       isBlock,
       disableTracking,
       loc
     }
   }
   ```

   返回了一个代码生成节点对象，包含了传入的参数数据（context.helpers 数组，目的是为了后续代码生成阶段，生成一些辅助代码）

   它转换后生成的 node.codegenNode 示例：

   ```json
   {
     "children": [
       // 子节点
     ],
     "directives": undefined,
     "dynamicProps": undefined,
     "isBlock": false,
     "isForBlock": false,
     "patchFlag": undefined,
     "props": {
       // 属性相关
     },
     "tag": "div",
     "type": 13
   }
   ```

   这个 codegenNode 相比之前的 AST 节点对象，多了很多和编译优化相关的属性，它们会在代码生成阶段会起到非常重要的作用！！！

#### 表达式节点转换函数

Element 节点有子节点，会继续递归，递归的过程就会经过表达式节点。

由于表达式本身不会再有子节点，所以它也不需要退出函数，直接在进入函数时做转换处理即可。

> 注意：只有在 Node.js 环境下的编译或者是 Web 端的非生产环境下才会执行 transformExpression

```js
const transformExpression = (node, context) => {
  if (node.type === 5 /* INTERPOLATION */) {
    // 处理插值中的动态表达式
    node.content = processExpression(node.content, context)
  }
  else if (node.type === 1 /* ELEMENT */) {
    // 处理元素指令中的动态表达式
    for (let i = 0; i < node.props.length; i++) {
      const dir = node.props[i]
      // v-on 和 v-for 不处理，因为它们都有各自的处理逻辑
      if (dir.type === 7 /* DIRECTIVE */ && dir.name !== 'for') {
        const exp = dir.exp
        const arg = dir.arg
        if (exp &&
          exp.type === 4 /* SIMPLE_EXPRESSION */ &&
          !(dir.name === 'on' && arg)) {
          dir.exp = processExpression(exp, context, dir.name === 'slot')
        }
        if (arg && arg.type === 4 /* SIMPLE_EXPRESSION */ && !arg.isStatic) {
          dir.arg = processExpression(arg, context)
        }
      }
    }
  }
}
```

它首先会通过 processExpression 函数，转换插值和元素指令中的动态表达式，把简单的表达式对象转换成复合表达式对象。

例如：{{ msg + test }}

执行 parse 后生成的表达式节点 node.content 值为一个简单的表达式对象

```js
{
  "type": 4,
  "isStatic": false,
  "isConstant": false,
  "content": "msg + test"
}
```

然后再经过这一步的 processExpression 处理后，node.content 的值变成了一个复合表达式对象

```js
{
  "type": 8,
  "children": [
    {
      "type": 4,
      "isConstant": false,
      "content": "_ctx.msg",
      "isStatic": false
    },
    " + ",
    {
      "type": 4,
      "isConstant": false,
      "content": "_ctx.test",
      "isStatic": false
    }
  ],
  "identifiers": []
}
```

注意，这里很重要。

children 属性中，把表达式 msg + test 拆成了三部分，其中变量 msg 和 test 对应都加上了前缀 _ctx。这样，就将插值表达式中的变量与 _ctx 中（也就是 setup 中）的变量建立了联系，当我们初次 render 时，就会触发响应式数据的 get，就会收集依赖。

> 为什么 Vue.js 2.x 编译的结果没有 _ctx 前缀？
>
> 这是因为 Vue.js 2.x 的编译结果使用了 with，比如上述模板，在 Vue.js 2.x 最终编译的结果：with(this){return _s(msg + test)}。
> 它利用 with 的特性动态去 this 中查找 msg 和 test 属性，所以不需要手动加前缀。

Vue.js 3.0 在 Node.js 端的编译结果舍弃了 with，它会在 processExpression 过程中对表达式动态分析，给该加前缀的地方加上前缀。

processExpression 内部依赖了 @babel/parser 库去解析表达式生成 AST 节点，并依赖了 estree-walker 库去遍历这个 AST 节点，然后对节点分析去判断是否需要加前缀。

但 @babel/parser 这个库通常是在 Node.js 端用的，而且这库本身体积非常大，如果打包进 Vue.js 的话会让包体积膨胀 4 倍，所以我们并不会在生产环境的 Web 端引入这个库，Web 端生产环境下的运行时编译最终仍然会用 with 的方式。

用 with 的话就完全不需要对表达式做转换了，这也就回应了前面的注意点：只有在 Node.js 环境下的编译或者是 Web 端的非生产环境下才会执行 transformExpression。

> 我们在业务开发中 Web 端的开发环境是用 vue-loader 或者 vite 编译，所以不用担心体积。所以也可以执行 transformExpression。

#### Text 节点转换函数

文本节点的转换，会合并一些相邻的文本节点，然后为内部每一个文本节点创建一个代码生成节点。

只处理根节点、元素节点、 v-for 以及 v-if 分支相关的节点，它也会返回一个退出函数，因为 transformText 要保证所有表达式节点都已经被处理才执行转换逻辑。

### 静态提升

节点转换完毕后，接下来会判断编译配置中是否配置了 hoistStatic，如果是就会执行 hoistStatic 做静态提升：

```js
if (options.hoistStatic) {
  hoistStatic(root, context)
}
```

`静态提升是 Vue.js 3.0 的一个编译优化`，例如：

```html
<p>>hello {{ msg + test }}</p>
<p>static</p>
<p>static</p>
```

如果它配置了 hoistStatic，经过编译后，它的代码就变成了这样：

```js
import { toDisplayString as _toDisplayString, createVNode as _createVNode, Fragment as _Fragment, openBlock as _openBlock, createBlock as _createBlock } from "vue"
const _hoisted_1 = /*#__PURE__*/_createVNode("p", null, "static", -1 /* HOISTED */)
const _hoisted_2 = /*#__PURE__*/_createVNode("p", null, "static", -1 /* HOISTED */)
export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock(_Fragment, null, [
    _createVNode("p", null, "hello " + _toDisplayString(_ctx.msg + _ctx.test), 1 /* TEXT */),
    _hoisted_1,
    _hoisted_2
  ], 64 /* STABLE_FRAGMENT */))
}
```

_hoisted_1 和 _hoisted_2 这两个变量，它们分别对应模板中两个静态 p 标签生成的 vnode，把它的创建放到了 render 函数外部执行。

这样做的好处是，不用每次在 render 阶段都执行一次 createVNode 创建 vnode 对象，直接用之前在内存中创建好的 vnode 即可。

因为这些静态节点不依赖动态数据，一旦创建了就不会改变，所以只有静态节点才能被提升到外部创建。

### 创建根节点的代码生成节点

这一步就是为 root 这个虚拟的 AST 根节点创建一个代码生成节点。

```js
function createRootCodegen(root, context) {
  const { helper } = context;
  const { children } = root;
  const child = children[0];
  if (children.length === 1) {
    // 如果子节点是单个元素节点，则将其转换成一个 block
    if (isSingleElementRoot(root, child) && child.codegenNode) {
      const codegenNode = child.codegenNode;
      if (codegenNode.type === 13 /* VNODE_CALL */) {
        codegenNode.isBlock = true;
        helper(OPEN_BLOCK);
        helper(CREATE_BLOCK);
      }
      root.codegenNode = codegenNode;
    }
    else {
      root.codegenNode = child;
    }
  }
  else if (children.length > 1) {
    // 如果子节点是多个节点，则返回一个 fragement 的代码生成节点
    root.codegenNode = createVNodeCall(context, helper(FRAGMENT), undefined, root.children, `${64 /* STABLE_FRAGMENT */} /* ${PatchFlagNames[64 /* STABLE_FRAGMENT */]} */`, undefined, undefined, true);
  }
}
```

这里创建的 codegenNode 就是为了后续生成代码时使用。

createRootCodegen 完成之后，接着把 transform 上下文在转换 AST 节点过程中创建的一些变量赋值给 root 节点对应的属性

```js
root.helpers = [...context.helpers]
root.components = [...context.components]
root.directives = [...context.directives]
root.imports = [...context.imports]
root.hoists = context.hoists
root.temps = context.temps
root.cached = context.cached
```

`这样后续在代码生成节点时，就可以通过 root 这个根节点访问到这些变量了`。

## 3. 生成代码

生成代码阶段会通过 generate 函数，整个过程中生成的代码字符串会不断的更新到 context.code 中）

```js
return generate(ast, extend({}, options, {
  prefixIdentifiers
}))
```

生成代码阶段与前面的阶段相互对应，分为：

1. 创建代码生成上下文

2. 生成预设代码

3. 生成渲染函数

   ```js
   if (!ssr) {
   	push(`function render(_ctx, _cache) {`);
   }
   else {
   	push(`function ssrRender(_ctx, _push, _parent, _attrs) {`);
   }
   indent();
   ```

   这是 render 函数的开头片段，这个 render 函数就是我们老生常谈的每个组件内的 render 函数了

4. 生成资源声明代码

   这部分也就是不断的往 context.code 中更新 render 函数内的内容

   ```js
   // 生成自定义组件声明代码
   if (ast.components.length) {
     genAssets(ast.components, 'component', context);
     if (ast.directives.length || ast.temps > 0) {
       newline();
     }
   }
   // 生成自定义指令声明代码
   if (ast.directives.length) {
     genAssets(ast.directives, 'directive', context);
     if (ast.temps > 0) {
       newline();
     }
   }
   // 生成临时变量代码
   if (ast.temps > 0) {
     push(`let `);
     for (let i = 0; i < ast.temps; i++) {
       push(`${i > 0 ? `, ` : ``}_temp${i}`);
     }
   }
   ```

5. 生成创建 VNode 树的表达式

   这里就是最后一步了

   ```js
   // 生成创建 VNode 树的表达式
   if (ast.codegenNode) {
     genNode(ast.codegenNode, context);
   }
   else {
     push(`null`);
   }
   ```

   在前面转换的过程中，我们给根节点添加了 codegenNode，所以接下来就是通过 genNode 生成创建 VNode 树的表达式。

   主要就是一个递归的思想，`遇到不同类型的节点，执行相应的代码生成函数生成代码即可`。节点生成代码的所需的信息可以从节点的属性中获取，这完全得益于前面 transform 的语法分析阶段生成的 codegenNode，根据这些信息就能很容易地生成对应的代码了。

我们通过一个示例来看最后的结果：

```html
<div class="app">
  <hello v-if="flag"></hello>
  <div v-else>
    <p>hello {{ msg + test }}</p>
    <p>static</p>
    <p>static</p>
  </div>
</div>
```

最终生成的 render 函数：

```js
import { resolveComponent as _resolveComponent, createVNode as _createVNode, createCommentVNode as _createCommentVNode, toDisplayString as _toDisplayString, openBlock as _openBlock, createBlock as _createBlock } from "vue"
const _hoisted_1 = { class: "app" }
const _hoisted_2 = { key: 1 }
const _hoisted_3 = /*#__PURE__*/_createVNode("p", null, "static", -1 /* HOISTED */)
const _hoisted_4 = /*#__PURE__*/_createVNode("p", null, "static", -1 /* HOISTED */)
export function render(_ctx, _cache) {
  const _component_hello = _resolveComponent("hello")
  return (_openBlock(), _createBlock("div", _hoisted_1, [
    (_ctx.flag)
      ? _createVNode(_component_hello, { key: 0 })
      : (_openBlock(), _createBlock("div", _hoisted_2, [
          _createVNode("p", null, "hello " + _toDisplayString(_ctx.msg + _ctx.test), 1 /* TEXT */),
          _hoisted_3,
          _hoisted_4
        ]))
  ]))
}
```

## 编译优化的总结

最后我们总结一下 Vue3 中的编译优化点。

### 动态节点的收集与 patchFlag

传统 Diff 算法无法避免新旧虚拟 DOM 树间无用的比较操作，是因为运行时得不到足够的关键信息，从而无法区分动态内容和静态内容。 换句话说，只要运行时能够区分动态内容和静态内容，就可以实现极简的优化策略

Vue3 就在编译阶段会对动态节点作标记，并且会通过 patchFlag 的类型来具体标明这个动态节点的哪部分是动态的。我们上文提到的 Block 节点就是动态节点，Block 节点一般包括模版根节点、 带有 v-for、v-if/v-else-if/v-else 等指令的节点。

有了对应的 patchFlag 标志，在 diff 算法期间就可以针对性地完成靶向更新。

### hoistStatic 静态提升

静态提升可以减少更新时创建虚拟 DOM 带来的性能开销和内存占用。

静态提升是以树为单位的。包含动态绑定的节点本身不会被提升，但是该节点上的静态属性是可以被提升的。

```js
<div>
   <p foo="bar" a=b>{{ text }}</p>
</div>

// 静态提升的props对象
const hoistprop = { foo: 'bar', a: 'b'}
function render(ctx) {
  return (openBlock(), createBlock('div', null, [
    createVNode('p', hoistprop, ctx.text)
  ]))
}
```

### 预字符串化

基于静态提升，Vue3 进一步采用预字符串化进行优化。

采用预字符串化可以将多个静态节点序列化为字符串， 并生成一个 Static 类型的 VNode。

例如：

```js
const hoist1 = createVNode('p', null, null, PatchFlags.HOISTED)
const hoist2 = createVNode('p', null, null, PatchFlags.HOISTED)
 // ...20个 hoistx 变量
const hoist20 = createVNode('p', null, null, PatchFlags.HOISTED)

render(){
 return (openBlock(), createBlock('div', null, [
   hoist1, hoist2, /* ...20个变量 */， hoist20
 ]))
}
```

经过预字符串化后：

```js
const hoistStatic = createStatticVNode('<p></p><p></p>...20个...<p></p>')

render() {
  return (openBlock(), createBlock('div', null, [hoistStatic]))
}
```

带来的优势：

- 大块的静态内容可以通过 innerHTML 设置， 在性能上有一定优势
- 减少创建虚拟节点产生的性能开销
- 减少内存占用

### cacheHandler 缓存内联事件处理函数

内联函数在每次重新渲染时，都会为组件创建一个全新的 props 对象。同时，props 对象中的 onChange 等属性的值也会是全新的函数。这样会造成额外的性能开销。

例如：

```html
<Com @change="a+b"/>
```

在未经优化前的 render 函数：

```js
function render(ctx) {
  return h(Com, {
   // 内联事件处理函数
   onChange: () => (ctx.a + ctx.b)
  })
}
```

缓存优化后：

```js
function render(ctx, cache) { // 数组 cache 来自组件实例
  return h(Com, {
    // 将内联事件处理函数缓存到 cache 中
    onChange: cache[0] || cache[0] = ($event) => (ctx.a + ctx.b)
  })
}
```

### v-once

v-once 用来缓存全部或部分虚拟 DOM 节点，能够避免组件更新时重新创建虚拟 DOM 带来的性能开销， 也可以避免无用的 Diff 操作

示例：

```js
<section>
  <div v-once>{{ foo }}</div>
</section>

function render(ctx, cache) {
  return (openBlock(), createBlock('div', null, [
      cache[1] || (cache[1] = createVNode("div", null, ctx.foo, 1 /* TEXT */))
  ]))
}
```

由于节点被缓存，意味着更新前后的虚拟节点不会发生变化，因此也就不需要这些被缓存的虚拟节点参与 Diff 操作了。编译后的结果如下：

```js
render(ctx, cache) {
  return (openBlock(), createBlock('div', null, [
    setBlockTracking(-1), //阻止这段 VNode 被 Block缓存    
    cache[1] || (cache[1] = createVNode("div", null, ctx.foo, 1 /* TEXT */)),
    setBlockTracking(1), // 恢复
    cache[1] // 整个表达式的值
  ]))
}
```

这样，v-once 包裹的动态节点就不会被父级 Block 收集，因此不会参与 Diff 操作。

所以，v-once 指令通常用于不会发生改变的动态绑定中，例如绑定一个常量。

```html
<div v-once>{{ SOME_CONSTANT }}</div>
```

### SSR 优化

在 SSR 渲染中，静态节点会直接输出字符串，绕过了虚拟 DOM 的创建。但动态节点还是需要动态渲染的。

### 辅助函数的 tree-shaking

编译时，会根据不同的情况，引入不同的 API。上文中的辅助函数，会根据模版的写法，用到了哪些指令，插值等，来动态的引入 createVNode 等需要的 API