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
