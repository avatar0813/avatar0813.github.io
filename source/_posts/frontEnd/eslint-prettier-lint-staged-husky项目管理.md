---
title: eslint+prettier+lint-staged+husky项目管理
date: 2023-07-30 19:31:41
tags: [project]
index_img: /img/projectManage.jpg
categories: [学习]
---

# 使用 `eslint`,`prettier`,`lint-staged`,`husky` 做项目管理

## 添加`eslint`，`prettier`做代码校验

### 下载依赖项

`pnpm install eslint prettier -D`

### 初始化`eslint`

运行命令：`pnpm init @eslint/config`
cli会提醒让你选择你的配置项,生成配置文件。

[![pPpRAl4.png](https://s1.ax1x.com/2023/07/30/pPpRAl4.png)](https://imgse.com/i/pPpRAl4)

### 配置`prettier`

```js
module.exports = {
  printWidth: 80,
  tabWidth: 2,
  semi: false,
  singleQuote: true,
};
```

### 设置忽略文件`.eslintignore.cjs`, `.prettierrc.cjs`

### 为了更好的结合`eslint` 与 `prettier` 还需添加 `eslint-config-prettier`, `eslint-plugin-prettier`

`eslint-config-prettier`的作用是为了关闭`eslint`中不必要或可能与 `Prettier` 冲突的规则。
需要在`eslint`配置文件中添加

```js
module.exports = {
  ...
  extends: [...ohterExtends,'prettier'],
}
```

`eslint-plugin-config`的作用则是为了将`prettier`的校验转化为`eslint`的规则，从而做到编辑器的错误提示.

```js
module.exports = {
  ...
  plugin: [...otherPlugins,'prettier'],
  rules: {
    'prettier/prettier': 'error'
  }
}

```

## 添加`husky`,`lint-staged`, 规范`commits`

`husky`的作用是调用`git hook`做提交校验
`lint-staged`的作用是只校验`git add .`的内容

### 下载`lint-staged`

`pnpm install -D lint-staged`

### 配置信息

```json
// package.json
"lint-staged": {
    "src/**/*.{js,jsx,ts,tsx,vue}": "eslint"
}
```

### 下载`husky`

`pnpm install -D husky`

### 添加`git hook`

**⚠️注意** 添加`hook`时需要有`.git`文件夹，如果是初始化的项目需先`git init`, 然后再运行`npx husky install` 创建`.husky`目录结构

```js
cmd: npx husky install
fatal: not a git repository (or any of the parent directories): .git
husky - git command not found, skipping install
```

运行命令

`npx husky add .husky/pre-commit "npx --no-install lint-staged"`

其中`"npx --no-install lint-staged"` 表示调用`lint-staged`

此时项目中 `husky` 配置完毕
```bash
##  。husky/precommit
. "$(dirname -- "$0")/_/husky.sh"

npx --no-install lint-staged

```

### 测试

[![pPp2OSg.png](https://s1.ax1x.com/2023/07/30/pPp2OSg.png)](https://imgse.com/i/pPp2OSg)

[![pPp2bY8.png](https://s1.ax1x.com/2023/07/30/pPp2bY8.png)](https://imgse.com/i/pPp2bY8)

**拦截提交表示成功**

## 对于使用`prettier`不同人也有不同的声音

大佬`Anthony Fu` 在他的个人[eslint-config](https://github.com/antfu/eslint-config)中就抛弃使用`prettier`.
以及有相关文章介绍他个人为什么不愿意再使用`prettier`[为什么我不使用 Prettier](https://antfu.me/posts/why-not-prettier-zh)