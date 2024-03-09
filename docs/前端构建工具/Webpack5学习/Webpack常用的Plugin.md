> 本节完整的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/05-plugin

## 什么是plugin

`webpack` 的插件（Plugin）是扩展其功能的核心机制之一。在 `webpack` 构建过程中，插件能够参与到整个构建流程的不同阶段，并执行特定任务，比如打包优化、资源处理、环境变量注入、生成额外文件等。

插件并不直接对模块内容进行转换操作，而是与 `loader` 互补，`loader` 主要负责单个文件的编译和加载，而插件则可以全局地改变输出结果或影响构建过程的行为。

以下是一些常见的插件及其作用：

## HtmlWebpackPlugin

根据模板文件自动生成 `html` 文件，并将编译后的 `bundle` 文件注入到 HTML 中。

安装：

```shell
$ pnpm add html-webpack-plugin -D
```

使用：

```js
// webpack.config.js
const path = require('node:path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  // ...
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, 'public/index.html'),
      // 压缩html文件
      minify: {
        removeComments: true, // 移除HTML中的注释
        collapseWhitespace: true, // 删除空白符与换行符
        minifyCSS: true, // 压缩内联css
      }
    }),
  ]
}
```

**多页应用打包**

有时，我们的应用不一定是一个单页应用，而是一个多页应用，那么如何使用 `webpack` 进行打包呢。

```js
// webpack.config.js
const path = require('node:path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  // ...
  entry: {
    index: './src/index.js',
    login: './src/login.js'
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, 'public/index.html'),
      filename: 'index.html', // 打包后的文件名
    }),
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, 'public/login.html'),
      filename: 'login.html'
    })
  ]
}
```

如果需要配置多个 `HtmlWebpackPlugin`，那么 `filename` 字段不可缺省，否则默认生成的都是 `index.html`。

但是有个问题，`index.html`和`login.html`会发现，都同时引入了`index.js`和`login.js`，通常这不是我们想要的，我们希望&nbsp;`index.html`只引入`index.js`，`login.html`只引入`login.js`。

`HtmlWebpackPlugin`提供了一个`chunks`的参数，可以接受一个数组，配置此参数仅会将数组中指定的`js`引入到`html`中。

```js
// webpack.config.js
const { resolve } = require('node:path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  // ...
  plugins: [
    new HtmlWebpackPlugin({
      template: resolve(__dirname, 'public/index.html'),
      filename: 'index.html', // 打包后的文件名
      chunks: ['index']
    }),
    new HtmlWebpackPlugin({
      template: resolve(__dirname, 'public/login.html'),
      filename: 'login.html',
      chunks: ['login']
    })
  ]
}
```

> 本案例完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/05-plugin/HtmlWebpackPlugin

## MiniCssExtractPlugin

从 `JavaScript` 文件中提取 `CSS` 样式，生成单独的`CSS` 文件。用于分离 `CSS` 和 `JavaScript`，提高加载性能。

安装：

```shell
$ pnpm add mini-css-extract-plugin -D
```

使用：

```js
// webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader'],
      },
    ],
  },
  plugins: [new MiniCssExtractPlugin()],
}
```

> 本案例完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/05-plugin/MiniCssExtractPlugin

## CssMinimizerWebpackPlugin

代码压缩`css`代码。这个插件需要结合上面的`MiniCssExtractPlugin`插件一起使用。

安装：

```shell
$ pnpm add css-minimizer-webpack-plugin -D
```

使用：

```js
// webpack.config.js
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

module.exports = {
  //...
  module: {
    rules: [
      {
        test: /.css$/,
        // 注意，这里用的是 `MiniCssExtractPlugin.loader` 而不是 `style-loader`
        use: [MiniCssExtractPlugin.loader, "css-loader"],
      },
    ],
  },
  optimization: {
    minimize: true,
    minimizer: [
      // Webpack5 之后，约定使用 `'...'` 字面量保留默认 `minimizer` 配置，因为默认开启了js代码的压缩，直接写的话会覆盖，因此需要保留
      "...",
      new CssMinimizerPlugin(),
    ],
  },
  // 需要使用 `mini-css-extract-plugin` 将 CSS 代码抽取为单独文件
  // 才能命中 `css-minimizer-webpack-plugin` 默认的 `test` 规则
  plugins: [new MiniCssExtractPlugin()],
}
```

这里的配置逻辑，一是使用 `mini-css-extract-plugin` 将 `CSS` 代码抽取为单独的 `CSS` 产物文件，这样才能命中 `css-minimizer-webpack-plugin` 默认的 `test` 逻辑；二是使用 `css-minimizer-webpack-plugin`。

> 本案例完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/05-plugin/CssMinimizerWebpackPlugin

## HtmlMinimizerWebpackPlugin

压缩`html`代码。

安装：

```shell
$ pnpm add html-minimizer-webpack-plugin -D
```

使用：

```js
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin')
const HtmlMinimizerPlugin = require('html-minimizer-webpack-plugin')

module.exports = {
  //...
  optimization: {
    minimize: true,
    minimizer: [
      // Webpack5 之后，约定使用 `'...'` 字面量保留默认 `minimizer` 配置
      "...",
      new HtmlMinimizerPlugin({
        minimizerOptions: {
          // 折叠 Boolean 型属性
          collapseBooleanAttributes: true,
          // 使用精简 `doctype` 定义
          useShortDoctype: true,
          // ...
        },
      }),
    ],
  },
  plugins: [new HtmlMinimizerPlugin({
    template: 'public/index.html'
  })],
}
```

> 本案例完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/05-plugin/HtmlMinimizerWebpackPlugin

## DefinePlugin

定义某些变量的值，在打包时用变量值替代变量。它允许您在编译时将预定义的变量或表达式替换为实际值。这对于根据环境变量注入不同的配置信息（例如 API URL、构建版本号等）非常有用，这样可以避免硬编码这些值，并在不同环境下有不同的行为。

`DefinePlugin`定义变量的方法如下：

```js
// webpack.config.js
const path = require('node:path')
const { DefinePlugin } = require('webpack')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  // ...
  plugins: [
    new DefinePlugin({
      API_URL: JSON.stringify('https://api.example.com'),
      VERSION: JSON.stringify('1.0.0'),
    }),
  ],
}

```

使用变量的方法如下：

```js
// src/index.js
console.log('process.env', process.env.NODE_ENV) // 这个是系统自带的
console.log('API_URL', API_URL)
console.log('VERSION', VERSION)
```

> 本案例完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/05-plugin/DefinePlugin

