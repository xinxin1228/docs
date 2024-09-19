> 当前案例完整的完整代码地址：[https://github.com/xinxin1228/docs_develop_tips/tree/main/05_%E4%BB%8E%E9%9B%B6%E6%90%AD%E5%BB%BA%E4%B8%80%E4%B8%AAMonorepo%E9%A1%B9%E7%9B%AE](https://github.com/xinxin1228/docs_develop_tips/tree/main/05_%E4%BB%8E%E9%9B%B6%E6%90%AD%E5%BB%BA%E4%B8%80%E4%B8%AAMonorepo%E9%A1%B9%E7%9B%AE)

## 什么是 Monorepo?

一个 `Monorepo` 是一个单一的存储库，包含**多个不同的项目**，和**明确的关系**。

现代的前端工程越来越多的都会选择 `Monorepo` 的方式进行开发，比如 `Vue`、`React` 等知名开源项目都采用的 `Monorepo` 的方式进行管理。

`Monorepo` 的组织方式如下：

```shell
├── packages
|   ├── pkg1
|   |   ├── package.json
|   ├── pkg2
|   |   ├── package.json
├── package.json

```

在 `packages` 目录下存放多个子项目，当然也可以叫其他名字，每个子项目都有自己的 `package.json` 文件，用来管理子包。

## Monorepo 能解决什么问题？

不仅仅在于减少了维护和管理多个代码库的成本，同时还有以下优点：

1. **代码复用**：因为多个项目共享一个代码库，所以避免了在不同项目中重复编写相同功能代码的问题，提高了开发效率。
2. **提升协作效率**：多个项目在同一个代码库中进行开发，可以方便地共享代码和文档，避免不同项目之间的沟通和协调成本。
3. **集中管理**：`Monorepo` 架构中，不同的应用程序都在同一个代码库中，方便管理和监控。这一点非常重要，特别是在需要同时对多个版本进行修改和维护的情况下。
4. **统一构建**：`Monorepo` 的一个重要特点是可以共用一套构建系统和**工具链进行构建和部署**，提升了构建的效率。
5. **可以快速定位问题**：由于所有的代码都在同一个代码库中进行开发，`debugger` 可以很快找出问题所在的代码文件和行数，便于开发人员调试问题。
6. **一个版本**：无需担心因为项目依赖于第三方库的冲突版本而导致的不兼容问题。

> 总结来说，就是如果多个项目中的编码规范、风格一致，并且使用了很多相同的 npm 包和相同的自定义组件或方法，那么可以将这个项目改装为 Monorepo 项目，这样可以在不同的项目中复用相同的组件或方法。
>
> 比如在开发多个后台，我们自己封装了一套通用的 xxxForm、xxxTable组件，这样无需 cv 即可在多个项目中直接使用。

## 使用 pnpm 搭建 Monorepo 项目

### 选型理由

`monorepo` 的单仓分模块的要求，使得仓库内的模块不仅要处理与外部模块的关系，还要处理内部之间相互的依赖关系。因此我们需要选择一个强大的包管理工具帮助处理这些任务。

目前前端包管理的根基是 [npm](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.npmjs.com%2F)，在其基础上衍生出了 [yarn](https://link.juejin.cn/?target=https%3A%2F%2Fyarnpkg.com%2F)、[pnpm](https://link.juejin.cn/?target=https%3A%2F%2Fpnpm.io%2Fzh%2Fmotivation)。在 2022 年以后，我们推荐使用 [pnpm](https://link.juejin.cn/?target=https%3A%2F%2Fpnpm.io%2Fzh%2Fmotivation) 来管理项目依赖。`pnpm` 覆盖了 `npm`、`yarn` 的大部分能力，且多个维度的体验都有大幅度提升。来搭建。

`pnpm` 具有以下优势：

- 速度快：多数场景下，安装速度是 `npm/yarn` 的 2 - 3 倍。
- 基于内容寻址：硬链接节约磁盘空间，不会重复安装同一个包，对于同一个包的不同版本采取增量写入新文件的策略。
- 依赖访问安全性强：优化了 `node_modules` 的扁平结构，提供了限制**依赖的非法访问(幽灵依赖)**的手段。
- 支持 `monorepo`：自身能力就对 `monorepo` 工程模式提供了有力的支持。在轻量场景下，无需集成 [lerna](https://link.juejin.cn/?target=https%3A%2F%2Flerna.js.org%2F)、[Turborepo](https://link.juejin.cn/?target=https%3A%2F%2Fturbo.build%2Frepo%2Fdocs) 等工具。

### workspace 模式

`pnpm` 支持 `monorepo` 模式的工作机制叫做 [workspace(工作空间)](https://link.juejin.cn/?target=https%3A%2F%2Fpnpm.io%2Fzh%2Fworkspaces)。

它要求在代码仓的根目录下存有 `pnpm-workspace.yaml` 文件指定哪些目录作为独立的工作空间，这个工作空间可以理解为一个子模块或者 `npm` 包。

例如以下的 `pnpm-workspace.yaml` 文件定义：`a` 目录、`b` 目录、`c` 目录下的所有子目录，都会各自被视为独立的模块。

```yaml
packages:
  - a
  - b
  - c/*
```

> 注意：`pnpm` 并**不是通过目录名称，而是通过目录下 `package.json` 文件的 `name` 字段来识别仓库内的包与模块的。**

### 中枢管理操作

在 `workspace` 模式下，代码仓根目录通常不会作为一个子模块或者 `npm` 包，而是**主要作为一个管理中枢，执行一些全局操作，安装一些共有的依赖。**下面介绍一些常用的中枢管理操作。

- 安装项目公共开发依赖，声明在根目录的 `package.json - devDependencies` 中。`-w` 选项代表在 `monorepo` 模式下的根目录进行操作。
- 每个子包都能访问根目录的依赖，适合把 `TypeScript`、`Vite`、`eslint` 等公共开发依赖装在这里。

```shell
$ pnpm install -wD xxx
```

- 卸载公共依赖，在根目录的 `package.json - devDependencies` 中删去对应声明。

```shell
$ pnpm uninstall -w xxx
```

- 执行根目录的 package.json 中的脚本。

```shell
$ pnpm run xxx
```

### 子包管理操作

在 `workspace` 模式下，`pnpm` 主要通过 `--filter` 或 `-F` 选项过滤子模块，实现对各个工作空间进行精细化操作的目的。

**1. 为指定模块安装外部依赖。**

- 下面的例子指为 `app1` 包安装 `lodash` 外部依赖。
- 同样的道理，`-S` 和 `-D` 选项分别可以将依赖安装为正式依赖(`dependencies`)或者开发依赖(`devDependencies`)。

```shell
# 为 app1 包安装 lodash
$ pnpm --filter app1 i -S lodash
$ pnpm --filter app1 i -D lodash

# 或
$ pnpm -F app1 i -S lodash
$ pnpm -F app1 i -D lodash
```

**2. 指定内部模块之间的互相依赖。**

- 指定模块之间的互相依赖。下面的例子演示了为 `app1` 包安装内部依赖 `b`，需要加上 `--workspace`。

```shell
$ pnpm --filter app1 add b --workspace
```

`pnpm workspace` 对内部依赖关系的表示不同于外部，它自己约定了一套 [Workspace 协议 (workspace:)](https://link.juejin.cn/?target=https%3A%2F%2Fpnpm.io%2Fzh%2Fworkspaces%23workspace-%E5%8D%8F%E8%AE%AE-workspace)。下面给出一个内部模块 `app1` 依赖同是内部模块 `b` 的例子。

```json
// app1/package.json

{
  "name": "app1",
  // ...
  "dependencies": {
    "b": "workspace:^"
  }
}

```

在实际发布 `npm` 包时，`workspace:^` 会被替换成内部模块 `b` 的对应版本号(对应 `package.json` 中的 `version` 字段)。替换规律如下所示：

```json
{
  "dependencies": {
    "a": "workspace:*", // 固定版本依赖，被转换成 x.x.x
    "b": "workspace:~", // minor 版本依赖，将被转换成 ~x.x.x
    "c": "workspace:^"  // major 版本依赖，将被转换成 ^x.x.x 建议使用
  }
}
```

> 补充：包版本的依赖关系。
>
> MAJOR.MINOR.PATCH
>
> **MAJOR**（主版本号）：发生了不兼容的API更改。
>
> **MINOR**（次版本号）：添加了向后兼容的新功能。
>
> **PATCH**（修补版本号）：进行向后兼容的问题修复。
>
> 例如，`1.2.3`表示主版本号1，次版本号2，修补版本号3。
>
> **版本范围符号**：
>
> **精确匹配版本**：`"express": "4.17.1"`
> 只允许安装指定版本 `4.17.1`，不会安装其他版本。
>
> **波浪号（`~`）**：`"express": "~4.17.1"`
> 表示可以安装 `4.17.x`，但不包括`4.18.0`，允许修补版本的更新。
>
> **插入号（`^`）**：`"express": "^4.17.1"`
> 表示可以安装 `4.x.x` 版本，允许次版本和修补版本的更新，兼容性较高。
>
> **星号（`*`）**：`"express": "*"`
> 表示可以安装任何版本，不限制。
>
> **大于、小于等号（`>`, `<`, `>=`, `<=`）**：`"express": ">=4.16.0 <4.18.0"`
> 表示指定一个版本范围，在此范围内选择适合的版本。

### 开始搭建

接下来，我们开始搭建 `pnpm` + `monorepo` 项目。先总览一下项目目录：

```shell
├── packages
|   ├── common
|   |   ├── index.js
|   |   ├── package.json
|   ├── app1
|   |   ├── index.js
|   |   ├── package.json
|   ├── app2
|   |   ├── index.js
|   |   ├── package.json
├── package.json
├── pnpm-workspace.yaml
```

首先，先创建一个工程根目录并初始化 `pnpm`。

```shell
$ mkdir monorepo && cd $_
$ pnpm init
```

在项目根目录中生成了 `packages.json` 文件，但是根目录并不是任何一个模块，它将作为整个组件库 `monorepo` 项目的管理中枢。

之后，我们在根目录下创建 `pnpm-workspace.yaml` 文件，这个文件的存在本身，会让 `pnpm` 要使用 `monorepo` 的模式管理这个项目，他的内容告诉 `pnpm` 哪些目录将被划分为独立的模块，这些所谓的独立模块被包管理器叫做 `workspace`(工作空间)。我们在这个文件中写入以下内容。

```yaml
packages:
  - packages/*
```

然后我们创建 `packages` 目录用来存在各个项目以及公共的项目。

```shell
$ mkdir packages
```

我们创建 `packages/common` 目录用来存放其它项目公共的组件和方法。并且进行 `pnpm` 初始化后，暴露一个对外的 `sum` 函数。

```shell
$ mkdir packages/common && cd $_ && pnpm init
```



<!-- tabs:start -->

#### **index.js**

```js
const sum = (a, b) => a + b

module.exports = {
  sum,
}
```

#### **package.json**

注意： `name` 是 `@monorepo/common`，因此安装和导入的依赖名称也是 `@monorepo/common` ，而不是 `common`。

```json
{
  "name": "@monorepo/common",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}

```

<!-- tabs:end -->

然后我们创建 `packages/app1` 和 `packages/app2` 目录，分别表示两个不同的项目。然后分别进行 `pnpm` 初始化。

```shell
$ mkdir packages/app1 && cd $_ && pnpm init
$ mkdir packages/app2 && cd $_ && pnpm init
```

然后分别在 `app1` 和 `app2` 中安装 `@monorepo/common`：

```shell
$ pnpm --filter app1 add @monorepo/common --workspace
$ pnpm --filter app2 add @monorepo/common --workspace
```

> 在使用 `pnpm --filter` 或 `-F` 时，后面跟的是包的 `package.json` 中的 `name` 字段，而不是包的目录名。

然后更改 `app1` 和 `app2` 下的 `index.js`  和 `package.json`。

<!-- tabs:start -->

#### **index.js**

```js
// app1[app2]/index.js
const { sum } = require('@monorepo/common')

console.log('app1 sum', sum(1, 2))
```

#### **package.json**

<!-- tabs:end -->

```json
{
  "name": "app1",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "node index.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@monorepo/common": "workspace:^"
  }
}

```

然后运行 `dev` 来测试一下：

```shell
# 在根目录下运行
pnpm --filter app1 dev

# 在 packages/app1 下运行
pnpm dev
```

> 最后提醒一下，改完 package.json 中的 name 之后，使用 `pnpm store prune` 清除缓冲。

