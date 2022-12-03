# mini-vue3

> 基于 Vue3.2.10，采用 yarn workspace + lerna 构建 monorepo 项目。

## 项目初始化

### 初始化 git

```zsh
git init
git config --local user.name xxx
git config --local user.email xxx
```

### 搭建 monorepo

```zsh
yarn init -y
```

在 package.json 中写入：

```json
{
  "private": true, // 防止包被作为第三方库上传到 npm 中
  "workspaces": [
    "packages/*" // 读取 packages 文件下所有包
  ]
}
```

在根目录下创建 packages/reactivity、packages/shared 等文件夹，同样以 `yarn init -y` 的方式在每个文件夹下生成 package.json 文件。

初始化两个子包，修改每个项目的 package.json：

```json
// reactivity/package.json
{
  "name": "@vue/reactivity",
  "main": "src/index.ts",
}

// shared/package.json
{
  "name": "@vue/shared",
  "main": "src/index.ts",
}

```

根据 package.json 的 main 字段创建入口文件。

#### 支持 typescript

在根目录下执行 `yarn add -D -W typescript`，将 typescript 添加到主包的 package.json 中。

在根目录下执行 `npx tsc --init`，调用 node_modules/bin/tsc 命令，自动生成 tsconfig.json 文件。

#### 添加其他包

`yarn add -D -W rollup rollup-plugin-typescript2 @rollup/plugin-node-resolve @rollup/plugin-json execa`

#### 代码格式化

安装第三方包：

```zsh
yarn add -D -W yorkie lint-staged prettier chalk
```

其中，yorkie 用于实现 gitHooks，lint-staged 用于实现对所有暂存文件进行操作，prettier 用于对代码进行检查或格式化，chalk 用于修改终端文字显示样式（注意最新版本不支持 require 导入，需切换成 ^4.1.0）。

修改 package.json 文件：

```json
  "gitHooks": {
    "pre-commit": "lint-staged", // 提交之前，操作暂存文件
    "commit-msg": "node scripts/verifyCommit.js" // 提交时，校验 commit message
  },
  "lint-staged": {
    "*.js": [
      "prettier --write" // 格式化所有 js 文件
    ],
    "*.ts?(x)": [
      "prettier --parser=typescript --write" // 格式化所有 ts、tsx 文件，使用 typescript 解析器
    ]
  },
```

创建 .prettier.json 文件，自定义代码风格：

```json
{
  "semi": false,
  "singleQuote": true,
  "printWidth": 80,
  "trailingComma": "none",
  "arrowParens": "avoid"
}
```

创建 scripts/verifyCommit.js 文件，验证提交信息：

```js
// Invoked on the commit-msg git hook by yorkie.

const chalk = require('chalk')
const msgPath = process.env.GIT_PARAMS
const msg = require('fs').readFileSync(msgPath, 'utf-8').trim()

const commitRE =
  /^(revert: )?(feat|fix|docs|dx|style|refactor|perf|test|workflow|build|ci|chore|types|wip|release)(\(.+\))?: .{1,50}/

if (!commitRE.test(msg)) {
  console.log()
  console.error(
    `  ${chalk.bgRed.white(' ERROR ')} ${chalk.red(
      `invalid commit message format.`
    )}\n\n` +
      chalk.red(
        `  Proper commit message format is required for automated changelog generation. Examples:\n\n`
      ) +
      `    ${chalk.green(`feat(compiler): add 'comments' option`)}\n` +
      `    ${chalk.green(
        `fix(v-model): handle events on blur (close #28)`
      )}\n\n` +
      chalk.red(`  See .github/commit-convention.md for more details.\n`)
  )
  process.exit(1)
}
```

#### 添加 EsLint

安装包：`yarn add -D -W eslint @typescript-eslint/parser`

在根目录下创建配置文件 .eslintrc.js 文件，写入配置：

```js
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
  },
  extends: ['airbnb-base'], // 使用 airbnb 风格
  rules: {
    // 覆盖规则
    semi: 0,
  },
}
```

安装包：`yarn add -D -W eslint-config-airbnb-base`

查找 peer 包：`yarn info eslint-config-airbnb-base peerDependencies`

安装依赖列表即可（这里需要额外安装 eslint-plugin-import 包）

#### 添加 lint 脚本

在 package.json 中添加：

```json
"scripts": {
  "lint": "eslint --ext .ts packages/**/*.ts"
},
```

其中，`**` 表示匹配所有目录（包括子目录），`*` 表示匹配所有文件，所以 `packages/**/*.ts` 表示匹配 packages 文件夹下所有 ts 文件。
