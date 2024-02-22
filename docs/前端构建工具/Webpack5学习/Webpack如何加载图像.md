> 本节完整的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/08-%E5%8A%A0%E8%BD%BD%E5%9B%BE%E5%83%8F

在 `Webpack5` 之前，我们需要使用 `file-loader`、`url-loader` 等 `Loader` 加载图片或其它多媒体资源文件，这些加载器各有侧重点，需要根据实际场景择优选用；而 `Webpack5` 之后引入了 `Asset Module`模型，自此我们只需要设置适当的 `module.rules.type` 配置即可，不需要为多媒体资源专门引入 `Loader`。


> `Asset Modules` 的相关配置 [查阅官方资料](https://webpack.docschina.org/guides/asset-modules/)。

`Asset Modules` 翻译成中文为 **“资源模块”** 的意思，他是 **Webpack5** 内置的模块类型，它允许 `Webpack` 处理资源文件（字体，图标等）而无需配置额外 `loader`。相当于 `Webpack5` 已经内置了`file-loader`与`url-loader`对图片、字体等这一些型文件的处理能力。

我们只需要在 `Webpack` 的`module.rules`配置选项中配置相关的信息，就可以完成对图片，字体等这一类型文件的处理。

`Asset Modules` 是 `Webpack` 的未来，随着 `Asset Modules` 功能的不断完善，未来我们将不会再使用之前提到的`file-loader`、`url-loader`等对文件资源处理的 `loader`，而完全用 `Asset Modules` 来取代。

```js
rules: [
  {
    test: /\.(jpe?g|png|gif|webp|svg)$/i,
    type: "asset",
    generator: {
      filename: "images/[hash][ext]",
    },
    parser: {
      dataUrlCondition: {
        maxSize: 10 * 1024, // 10KB 小于10k的直接转base64
      },
    },
  },
]
```

- `type`： 采用那种方式来处理 test 中匹配到的资源模块，他的值有很多，这里主要使用的与图片相关的 3 种类型，[详细查阅资料](https://webpack.docschina.org/configuration/module/#ruletype)。

- `parser.dataUrlCondition.maxSize` 用来设置文件的大小，Webpack 会根据这个大小来决定打包的图片资源是以文件输出还是替换成 Base64 编码格式的 DataURL。

- `generator.filename` 用来设置文件的名称。默认文件名格式是：`[hash][ext][query]`。也可以通过`output.assetModuleFilename`来设置，

| 类型值         | 说明                                                         |
| :------------- | :----------------------------------------------------------- |
| asset/resource | 这种类型用来替换之前`file-loader`的操作。它处理文件导入地址并将其替换成访问地址，同时把文件输出到相应位， |
| asset/inline   | 这种类型用来替换之前`url-loader`的操作。它处理文件导入地址并将其替换成 Base64 编码格式的 DataURL 地址。 |
| asset          | 在单独导出文件和生成 DataURL 间自动选择。用来替换之前通过`url-loader`，并配置资源体积来实现的功能。 |

接下来我们分别加载一下`jpg`、`png`、`svg`和`gif`的资源。项目的目录结构如下：

```js
// 项目目录
├── node_modules
├── package.json
├── pnpm-lock.yaml
├── public
│   ├── img  // 资源文件夹
│   │   ├── 01.jpg
│   │   ├── gif.gif
│   │   ├── note.png
│   │   └── note.svg
│   └── index.html
├── src
│   └── index.js
└── webpack.config.js
```

配置`webpack.config.js`如下：

```js
// webpack.config.js
const path = require('node:path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

/**
 * @type {import('webpack').Configuration}
 */
module.exports = {
  mode: 'development',
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    clean: true,
  },
  devServer: {
    port: 8080,
    hot: true,
  },
  module: {
    rules: [
      {
        test: /\.(jpe?g|png|svg|webp|gif)$/i,
        type: 'asset',
        generator: {
          filename: 'imgs/[hash][ext]',
        },
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024, // 10KB  少于10k转base64
          },
        },
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, 'public/index.html'),
    }),
  ],
}
```

如何在文件中引入使用：

```js
// src/index.js
import svg from '../public/img/note.svg'
import png from '../public/img/note.png'
import jpg from '../public/img/01.jpg'
import gif from '../public/img/gif.gif'

createImage([svg, png, jpg, gif])

function createImage(imgList) {
  const domEl = document.createDocumentFragment()

  imgList.forEach(img => {
    const imgEl = document.createElement('img')
    imgEl.src = img
    domEl.append(imgEl)
  })

  document.body.append(domEl)
}

```

