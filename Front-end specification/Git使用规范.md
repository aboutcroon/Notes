# 1. 分支命名与保护

## 1.1 分支命名

### 1.1.1 master（主分支）

- 主分支，线上环境
- 该分支为只读分支，只能从 `test` 分支合并，不能直接在此分支上进行修改
- 所有在 `master` 分支的推送应该打标签做记录，方便追溯

### 1.1.2 dev（主开发分支）

- 主开发分支，基于 `master` 分支克隆
- 该分支为只读分支，只能从其他分支（`feature/hotfix`）合并，不能直接在此分支上进行修改
- 包含所有要发布到下一个 `test` 分支的代码
- feature 功能分支开发完成时，合并到 `dev` 分支

### 1.1.3 release（预发布分支）

- 预发布分支，基于 `dev` 分支克隆
- 当有一组 feature 开发完成，首先会合并到 dev 分支，进入提测时，会基于 dev 分支创建 release 分支进行测试
- 如果测试过程中若存在 bug 需要修复，则直接由开发者在 release 分支修复并提交，修复完成之后合并到 dev 分支

### 1.1.4 feature （功能开发分支）

- 功能开发分支，基于 `dev` 分支克隆
- 开发新功能时，需要从 `dev` 分支拉取 `feature` 分支，分支命名为 feature-功能名称/模块名称
- 开发完成后，将代码合并到 `dev` 分支，并删除该分支

### 1.1.5 hotfix（bug 修复分支）

- 补丁分支，基于 `master` 分支克隆，主要用于对线上版本进行 bug 修复
- 命名为 hotfix-bug模块名称，修复完毕后合并到 `dev` 分支
- 属于临时分支，补丁修复上线后可删除该分支

## 1.2 分支保护

设置 `master` 分支、`test` 分支、`dev` 分支只能通过 merge request 的方式合并，不可以直接 push 代码。

由负责人进行 `code review` 之后再合并到相应分支。





# 2. Commit message 规范

参考于 Angular 定义的 commit 基本格式

[参考文档](https://github.com/angular/angular/blob/master/CONTRIBUTING.md)

[具体示例](https://github.com/angular/angular/commits/master)

## 2.1 基本格式

基本格式如下：

```html
<type>(<scope>): <subject>
 
<body>
 
<footer>
```

第一行的 `Header` 部分为必填，`Body` 和 `Footer` 部分为可选。注意 Header，Body，Footer 中间有空白行分割。

## 2.2 Header

Header 部分只有一行，包括三个字段：`type`(必需)、`scope`(可选)、`subject`(必需)。

### type：

`type` 用于说明 commit 的类别，只允许使用以下 7 个标识

- `feat`：新功能（feature）
- `fix`：修补 bug
- `docs`：文档（documentation）
- `style`：格式（不影响代码运行的变动）
- `refactor`：重构（不是新增的功能，也不是修改 bug 的代码变动）
- `test`：增加测试
- `chore`：构建过程或辅助工具的变动

通常 feat 和 fix 会被放入 changelog（变更日志） 中，其他类型则不会。

### scope：

`scope` 用于说明 commit 影响的范围，为可选值。通常是文件、路径、功能等

### subject：

`subject` 是 commit 目的的简短描述，不超过 50 个字符，一般用英文表示

- 以动词开头，使用第一人称现在时，比如 `change`，而不是 `changed` 或 `changes`
- 第一个字母小写
- 结尾不加句号 (.)

## 2.3 Body

`Body` 部分是对本次 commit 的详细描述，为可选项，可以分成多行（换行用 \n），示例：

```
More detailed explanatory text, if necessary. 
Further paragraphs come after blank lines. 

- Bullet points are okay, too 
- Use a hanging indent
```

## 2.4 Footer

`Footer` 部分只用于两种情况：

### 不兼容的变动

如果当前代码与上一个版本不兼容，则 Footer 部分以 `BREAKING CHANGE` 开头，后面加上对变动的描述、变动理由以及迁移方法。

### 关闭问题

如果当前 commit 针对某个问题，那么可以在 Footer 部分关闭这个问题：

```
Closes #234
```

也可以一次关闭多个问题：

```
Closes #123, #245, #992
```





# 3. 自动化工具处理 commit message

如 commit message 规则所示，若我们手写上述的 commit 基本格式很明显并不方便，而且还有出错的可能，那么可以使用自动化工具生成上述规范的commit message。

## 3.1 Commitizen

我们可以使用 [commitizen](https://github.com/commitizen/cz-cli) 工具来规范化 git commit 的 message。

commitizen 本质上是一个通用的规范化 git commit 的框架，可以通过各种 [Adapters](https://github.com/commitizen/cz-cli#adapters) 来格式化对应的格式。我们上述使用的规则是 Angular 的 commit 格式，那么需要额外安装 cz-conventional-changelog 这个 Adapter。目前 commitizen 实现的 Adaptor 有 Angular、ESlint、mapbox、Jira 等等，我们这里只使用 cz-conventional-changelog 来实现 Angular 的 commit 格式。

## 3.2 具体使用

全局安装 commitizen

```nginx
npm install commitizen -g
```



使用 commitizen 命令初始化当前工程

```nginx
commitizen init cz-conventional-changelog --save-dev
```



执行完毕后，在 package.json 文件中的 devDependencies 配置中多了 cz-conventional-changelog 依赖，也增加了以下配置段

```json
"config": {
  "commitizen": {
    "path": "./node_modules/cz-conventional-changelog"
  }
}
```



此时，使用 `git cz` 命令来替换 git commit 命令即可。





# 4. Git hook 设置 commit message 校验

我们可以在 git commit 的 hook 中加入 [commitlint](https://github.com/conventional-changelog/commitlint) 校验，这样可以使不符合 commit 规范的本地提交无法完成。

## 4.1 具体使用

在项目中安装 husky，commitlint 相关依赖

```nginx
npm install husky @commitlint/config-conventional @commitlint/cli --save-dev
```



在 package.json 文件中添加 husky 的 git hook 配置

```json
{
  "husky": {
    "hooks": {
      "commit-msg": "commitlint -x @commitlint/config-conventional -E HUSKY_GIT_PARAMS"
    }
  }
}
```



此时，如果提交的 commit message 不符合规范，该 hook 就会报错。

## 4.2 配置规则

我们可以生成 `commitlint.config.js` 或 `.commitlintrc.js` 文件来配置自定义的规则：

```
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
```



commitlint.config.js 文件的配置如下：

```js
module.exports = {
  extends: [
    "@commitlint/config-conventional"
  ],
  rules: {
    'type-enum': [2, 'always', [
      'upd', 'feat', 'fix', 'refactor', 'docs', 'chore', 'style', 'revert'
     ]],
    'type-case': [0],
    'type-empty': [0],
    'scope-empty': [0],
    'scope-case': [0],
    'subject-full-stop': [0, 'never'],
    'subject-case': [0, 'never'],
    'header-max-length': [0, 'always', 72]
  }
}
```

rules 由 name 和配置数组组成，如：" 'name': [0, 'always', 72] "，数组中第一位为 level，可选 0，1，2，0 为 disable，1 为 warning，2 为 error；第二位为应用与否，可选 always 或 never；第三位为该 rule 的值。

更多配置规则可参照 commitlint 官方网站：[commitlint rules](https://commitlint.js.org/#/reference-rules)





# 5. Git hook 设置 eslint 校验及修复

我们也可以在 git commit 的 hook 中加入 [lint-staged](https://github.com/okonet/lint-staged#readme) 校验，用来对 git 暂存的文件执行 lint 修复。

## 5.1 具体使用

安装 husky、lint-staged 和 eslint 依赖

```nginx
npm install husky lint-staged eslint --save-dev
```



在 package.json 文件中增加 husky 和 lint-staged 配置

```json
"husky":{
  "hooks": {
    "pre-commit": "lint-staged"
  }
},
"lint-staged": {
  "src/**": [
    "eslint --fix"
  ]
}
```

## 5.2 配置规则

上述 lint 修复会根据 .eslintrc.js 文件中的配置进行。

eslint 配置规则详见：[eslint](http://10.10.1.21:8050/display/FRON/eslint)

