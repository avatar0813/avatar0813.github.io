# 从一次下载失败理解npm、yarn install策略

## 工具版本

- `nodejs`: v14.21.2
- `npm`: 6.14.17
- `yarn`: 1.22.19

## 问题现象

在一次删除`lock`文件重新下载依赖时，原本能够正常下载的依赖，现在却报错：**node版本太低**，导致下载依赖失败。特记录此次问题出现的原因。

假设现在项目有两个依赖 `marked`, `simplemde`

```json
"dependencies": {
    "marked": "^0.3.19",
    "simplemde": "^1.11.2"
}
```

而`simplemde` 内部又依赖了`marked`

```json
  "dependencies": {
    ...
    "marked": "*"
  },
```

发现`marked`有两个依赖版本`marked@*`, `marked@^0.3.19`, 也可以说项目添加`marked`依赖的目的就是为了控制`simplemde`中的依赖`marked`的版本。然而`yarn`在下载依赖的时候，将依赖中的子依赖也平铺开了下载。

此时：`marked@*` 指定的依赖版本是 `5.0.4` 而 自2023-05-02发布的`marked@5.0.0`开始， 对`node`的版本要求直接从 12 升至 18，导致`engines.node`版本校验不通过，下载报错。

```json
// marked/v5.0.0/package.json
{
  ...
  "engines": {
    "node": ">= 18"
  }
}
```

### 对比 `npm` 和 `yarn` 下载依赖

> 通过`npm install`下载依赖 (npm version: 6.14.17)

```json
...
// package-lock.json
"marked": {
    "version": "0.3.19",
    ...
},
"simplemde": {
    "version": "1.11.2",
    ...
    "requires": {
    ...
    "marked": "*"
    }
},
...
```

在`package-lock.json`中, 将`marked` 锁在 `version: 0.3.19`
> 通过 `yarn` 下载依赖 (yarn version: 1.22.19)

```json
// yarn.lock
...
marked@*:
  version "5.0.4"
  ...

marked@^0.3.19:
  version "0.3.19"
  ...

simplemde@^1.11.2:
  version "1.11.2"
  ...
  dependencies:
    ...
    marked "*"
...
```

`yarn.lock` 中确有两个`marked`依赖版本

## 问题原因

而一开始在项目中指定`marked: ^0.3.19`也就是为了限制子依赖版本在`0.3.*`, 但是现在使用`yarn`下载会校验子依赖中的`marked@*`,导致下载中断, 但是使用`npm`下载却不会有这个问题。

虽然可以通过配置忽略版本校验，但是**根本原因**还是`yarn`重复校验下载了依赖。

## 了解`yarn`的[install流程](https://yarnpkg.com/cli/install)

- `Resolving packages`: 分析包的依赖关系及版本信息
- `Fetching packages`: 下载依赖项，存储到缓存中
- `Linking dependencies`: 将缓存中的包扁平化的安装到项目当中去
- `Building fresh packages`: 构建安装, 执行install阶段的scripts

### 测试校验

- 首先清空缓存

`yarn` 查看缓存路径：`yarn cache dir`, 清空缓存：`yarn cache clean --force`

`npm` 查看缓存路径：`npm config get cache`, 清空缓存：`npm cache clean --force`

使用`yarn`下载，下载失败

![pPiXFjH.png](https://s1.ax1x.com/2023/08/03/pPiXFjH.png)

![pPiXAud.png](https://s1.ax1x.com/2023/08/03/pPiXAud.png)

> 这里倒是可以通过忽略`engines`来通过下载

```bash
# 忽略engines校验
yarn config set ignore-engines true
```

## `npm install`的过程发生了什么

当存在嵌套依赖和根级依赖冲突时，npm会根据以下规则来确定使用哪个依赖项：

- 直接依赖优先： 根级依赖的优先级更高，它们将覆盖任何嵌套依赖中的相同包。
- 版本范围解析： 如果根级依赖和嵌套依赖都有对同一个包的依赖，并且它们的版本范围不冲突，npm会尽量满足两者的依赖关系，并使用符合两者版本要求的最高版本。
- 版本冲突解析： 如果根级依赖和嵌套依赖对同一个包有不兼容的版本要求，npm会尝试解决版本冲突，通常会选择满足所有依赖关系的最高版本，并通过符号链接或软链接来确保正确的依赖关系。

### `npm`如何解决有版本冲突的依赖包

现在我一个项目有两个依赖, `marked@^0.3.19`, 和 自己发的包 `avatar0813-pkg-t`

其中`avatar0813-pkg-t`包依赖 `marked@^5.1.2`

![pPiOQmR.png](https://s1.ax1x.com/2023/08/03/pPiOQmR.png)

![pPiOKX9.png](https://s1.ax1x.com/2023/08/03/pPiOKX9.png)

![pPiOu6J.md.png](https://s1.ax1x.com/2023/08/03/pPiOu6J.md.png)

> 问题: node_modules 中的`avatar0813-pkg-t` 包中间还有个node_modules

**这正是npm install解决版本冲突策略，因为两个依赖`marked@^0.3.19`,`marked@^5.1.2`版本不兼容导致的，两个依赖都要，但是不能扁平化安装**

![pPiOnl4.png](https://s1.ax1x.com/2023/08/03/pPiOnl4.png)

> 如果将`avatar0813-pkg-t` 包中依赖改为的`marked@*`以兼容外部版本则不会出现这个问题。

> 亦或这指定`resolutions`依赖从而解决冲突

![pPixWGV.md.png](https://s1.ax1x.com/2023/08/03/pPixWGV.md.png)
