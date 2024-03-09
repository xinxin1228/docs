> 本节完整的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/06-%E4%BB%A3%E7%A0%81%E5%8E%8B%E7%BC%A9

资源压缩的主要目的是减小文件体积，以提升页面加载速度或降低带宽消耗等。资源压缩通常发生在生产环境打包的最后一个环节，开发环境打包是不需要进行资源压缩处理的。

资源压缩主要是对 HTML、JS、CSS 文件、图片进行压缩，通常的做法就是整个文件或大段的代码压缩成一行，比如：去掉空格，空行，删除注释，把较长的变量名替换成较短的变量名等。

图片这类静态资源，一般在项目上线后会交由服务端来处理，不需要我们处理。

## JS压缩

> 默认在 `Webpack5` 中，当我们**开启生产环境**打包时，`JS` 代码会自动做压缩处理，并不需要做任何配置。

是因为在安装 `Webpack5` 时，会自动安装`terser-webpack-plugin`插件，同时在生产环境下打包时，`Webpack5` 会自动使用这个插件来对`JS` 做压缩处理。

如果我们不想在生产环境打包时，对 `JS` 做压缩处理，我们只需要`webpack.config.js`文件中添加如下配置，就可以。

```js
// webpack.config.js
module.exports = {
  // ...
  // 相关的优化配置，在这里配置
  optimization: {
    // 不使用插件压缩代码
    // 默认值为true，表示使用terser-webpack-plugin 插件来对JS代码压缩。
    minimize: false,
  },
};
```

## CSS压缩

> 默认配置下，`Webpack5` 在生产环境打包下，并不会对 `CSS` 做压缩处理。

我们需要借助第三方插件：`css-minimizer-webpack-plugin`来帮我们对 CSS 做压缩处理。

> 注意：必须结合 `MiniCssExtractPlugin` 插件才能发挥作用，使用 `style-loader` 是不会发挥作用。

安装：

```shell
$ pnpm add css-minimizer-webpack-plugin -D
```

接下来，按以下步骤修改`webpack.config.js`文件，来完成相关插件配置即可。

- 使用`require`方法，来加载插件
- 在`optimization`选项中通过`minimize`和`minimizer`属性，告诉 Webpack 需要使用`minimizer`中的插件来压缩代码。

```js
// webpack.config.js
module.exports = {
  // ....
  optimization: {
    // 使用插件压缩代码
    minimize: true,
    // 提供一个或多个压缩插件，来对打包后的文件作相关压缩，不过会覆盖默认的插件
    // 也就是，这样设置后，CSS代码确实压缩，但是JS代码确没有被压缩了
    minimizer: [new CssMinimizerPlugin()],
  },
}
```

但是 JS 代码确没有被压缩了。是因为`minimizer:[new CssMinimizerPlugin()]`设置，覆盖了默认的 JS 压缩插件。

如果想要 JS 和 CSS 都能被压缩，则需要在`minimizer`对应数组中添加 JS 压缩插件。官方说明文档中提到，可以在 `optimization.minimizer` 中可以使用 `'...'` 来访问默认值（JS 压缩插件）。

接下来我们把`webpack.config.js`配置文件中`optimization`项的内容，修改如下：

```js
// webpack.config.js
module.exports = {
  // ...
  optimization: {
    minimize: true,
    // ...表示，webpack5自带的JS压缩插件
    minimizer: [new CssMinimizerPlugin(), '...'],
  },
}
```

## HTML压缩

> 在使用`html-webpack-plugin`插件时，在这个插件中通过配置`minify`参数，就可以控制打包后的 HTML 代码是否需要做压缩处理。
>
> 如果`minify`的值为`false`表示不对打包后的 HTML 压缩，`true`表示做压缩处理。他的默认值就是 `true`，所以如果要做压缩处理，也可以不用写这个配置。

如果没有使用`html-webpack-plugin`这个插件，也可以使用另一个插件：`html-minimizer-webpack-plugin`。

`html-minimizer-webpack-plugin`是一个基于 `JavaScript` 实现的、高度可配置的 HTML 压缩器，支持一系列压缩特性。

安装：

```shell
$ pnpm add html-minimizer-webpack-plugin -D
```

使用：

```js
// webpack.config.js
const HtmlWebpackPlugin = require("html-webpack-plugin");
const HtmlMinimizerPlugin = require("html-minimizer-webpack-plugin");

module.exports = {
  // ...
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
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, 'public/index.html')
    }),
  ],
}
```

