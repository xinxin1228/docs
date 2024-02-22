> 本节完整的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/09-npm%E5%8C%85

虽然 `Webpack` 多数情况下被用于构建 `Web` 应用，`Webpack` 同样具有完备的构建 `NPM `库的能力。与一般场景相比，构建 `NPM`库时需要注意：

- 正确导出模块内容。
- 不要将第三方包打包进产物中，以免与业务方环境发生冲突。
- 将 `CSS` **抽离**为独立文件，以方便用户自行决定实际用法。
- 始终生成 `Sourcemap` 文件，方便用户调试。

接下来我们搭建一个简单的`npm`包，要求兼容 `import/require`和`浏览器`环境下使用。

## 使用方式

`import`的方式使用：

```js
import { sum, deepCopy } from './utils.js'

sum(1, 2)

// 或以下方式引用

import * as _ from './utils.js'

_.sum(1, 2)
```

`require`的方式使用：

```js
const { sum, deepCopy } = require('./utils.js')

sum(1, 2)

// 或以下方式引用

const _ = require('./utils.js')

_.sum(1, 2)
```

`浏览器`下使用：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <script src='./utils.js'></script>
    <title>webpack</title>
  </head>
  <body>
    <h1>webpack npm包测试</h1>

    <script>
      console.log(_)
    </script>
  </body>
</html>

```

## 目录结构

目录结构如下：

```shell
├── node_modules
├── package.json
├── pnpm-lock.yaml
├── public
│   └── index.html
├── src
│   ├── index.js
│   ├── index.test.js # 测试 import/require 环境
│   └── utils
│       └── check.js
├── test
│   └── index.html  # 测试浏览器环境
└── webpack
    ├── webpack.comm.config.js
    ├── webpack.dev.config.js
    └── webpack.prod.config.js

```

## 配置weback.config.js

接下来，我们需要将上例构建为适合分发的产物形态。我们需要修改 `output.library` 配置，以适当方式导出模块内容：

```js
// webpack.config.js
module.exports = {
  // ...
  output: {
    filename: "[name].js",
    path: path.join(__dirname, "./dist"),
    library: {
      name: "_",
      type: "umd",
    },
  },
  // ...
};
```

这里用到了两个新配置项：

- [output.library.name](https://webpack.docschina.org/configuration/output#outputlibrary)：用于定义模块名称，在浏览器环境下使用 `script` 加载该库时，可直接使用这个名字调用模块，例如：

```js
// webpack.config.js
module.exports = {
  // …
  entry: './src/index.js',
  output: {
    library: 'MyLibrary',
  },
};
```

假设你在 `src/index.js` 的入口中导出了如下函数：

```js
export function hello(name) {
  console.log(`hello ${name}`);
}
```

此时，变量 `MyLibrary` 将与你的入口文件所导出的文件进行绑定，下面是如何使用 webpack 构建的库的实现：

```html
<script src="https://example.org/path/to/my-library.js"></script>
<script>
  MyLibrary.hello('webpack');
</script>
```

- [output.library.type](https://webpack.docschina.org/configuration/output#outputlibrarytype)：用于编译产物的模块化方案，可选值有：`commonjs`、`umd`、`module`、`jsonp` 等，通常选用兼容性更强的 `umd` 方案即可。

> 提示： library.name 和 library.type也可以写成下面的方式。

```js
// webpack.config.js
module.exports = {
  // ...
	output: {
  -	 library: {
  -    	name: "_",
  -    	type: "umd",
  -  	},
  +  	library: '_',
  +	  libraryType: 'umd'
	},
}
```

## 排除第三方包依赖包

假设我们在项目中使用[`lodash`](https://www.lodashjs.com/)。

安装：

```shell
$ pnpm add lodash
```

使用：

```js
// src/index.js
import lodash from 'lodash'

console.log('lodash', lodash.VERSION)
```

此时执行编译命令 `npx webpack`，我们会发现产物文件的体积非常大。

这是因为 Webpack 默认会将所有第三方依赖都打包进产物中，这种逻辑能满足 Web 应用资源合并需求，但在开发 NPM 库时则很可能导致代码冗余。以 `test-lib` 为例，若使用者在业务项目中已经安装并使用了 `lodash`，那么最终产物必然会包含两份 `lodash` 代码！

为解决这一问题，我们需要使用 [externals](https://webpack.docschina.org/configuration/externals/) 配置项，将第三方依赖排除在打包系统之外：

```js
// webpack.config.js
module.exports = {
  // ...
+  externals: {
+   lodash: {
+     commonjs: "lodash",
+     commonjs2: "lodash",
+     amd: "lodash",
+     root: "_",
+   },
+ },
  // ...
};
```

> 提示： Webpack 编译过程会跳过 `externals` 所声明的库，并假定消费场景已经安装了相关依赖，常用于 NPM 库开发场景；在 Web 应用场景下则常被用于优化性能。
>
> 例如，我们可以将 React 声明为外部依赖，并在页面中通过 `<script>` 标签方式引入 React 库，之后 Webpack 就可以跳过 React 代码，提升编译性能。

至此，Webpack 不再打包 `lodash` 代码，我们可以顺手将 `lodash` 声明为 `peerDependencies`：

```js
// package.json
{
  // ...
+ "peerDependencies": {
+   "lodash": "^4.17.21"
+ }
}
```

> 在`package.json`文件中，`peerDependencies`是一个特殊的依赖项列表，它用来表示当前包（库或插件）需要与宿主项目共享的其他包的特定版本范围。这些共享依赖通常是在运行时被宿主项目直接引用，并且对于当前包的功能至关重要。

!> 注：以下方法为网上通用的方法，但我并没有实现成功。

实践中，多数第三方框架都可以沿用上例方式处理，包括 React、Vue、Angular、Axios、Lodash 等，方便起见，可以直接使用 `webpack-node-externals` 排除所有 `node_modules` 模块，使用方法：

```js
// webpack.config.js
const nodeExternals = require('webpack-node-externals');

module.exports = {
  // ...
+  externalsPresets: { node: true }
+  externals: [nodeExternals()]
  // ...
};
```

## 抽离 CSS 代码

假设我们开发的 NPM 库中包含了 CSS 代码 —— 这在组件库中特别常见，我们通常需要使用 `mini-css-extract-plugin` 插件将样式抽离成单独文件，由用户自行引入。

这是因为 Webpack 处理 CSS 的方式有很多，例如使用 `style-loader` 将样式注入页面的 `<head>` 标签；使用 `mini-css-extract-plugin` 抽离样式文件。作为 NPM 库开发者，如果我们粗暴地将 CSS 代码打包进产物中，有可能与用户设定的方式冲突。

为此，需要在前文基础上添加如下配置：

```js
// webpack.config.js
module.exports = {  
  // ...
+ module: {
+   rules: [
+     {
+       test: /\.css$/,
+       use: [MiniCssExtractPlugin.loader, "css-loader"],
+     },
+   ],
+ },
plugins: [new MiniCssExtractPlugin()],
};
```

## 生成 Sourcemap

`Sourcemap` 是一种代码映射协议，它能够将经过压缩、混淆、合并的代码还原回未打包状态，帮助开发者在生产环境中精确定位问题发生的行列位置，所以一个成熟的 NPM 库除了提供兼容性足够好的编译包外，通常还需要提供 `Sourcemap` 文件。

接入方法很简单，只需要添加适当的 `devtool` 配置：

```js
// webpack.config.js
module.exports = {  
  // ...
+ devtool: 'source-map'
};
```



