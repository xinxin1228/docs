在 <u>[初体验](/docs/前端构建工具/webpack5学习/Webpack配置项总览以及初体验.md)</u> 中，我们见识到了`webpack.config.js`写法中的一种方式：就是导出**单个配置对象**的方法实现。

```js
// webpack.config.js
module.exports = {
    mode: 'development',
    entry: './src/index.js',
    // ...
}
```

实际上，`webpack`还支持以**数组**、**函数**方式来配置运行函数，以适配不同的应用场景和需求，它们的大致上的区别如下：

- **单个配置对象**：比较常用的一种方式，逻辑简单，适合大多数业务项目；
- **配置对象数组**：每个数组项都是一个完整的配置对象，每个对象都会触发一次单独的构建，通常用于需要为同一份代码构建多种产物的场景，如 `Library`；
- **配置对象函数**：`webpack` 启动时会执行该函数获取配置，我们可以在函数中根据环境参数(如 `NODE_ENV`)动态调整配置对象。

下面我们一一展示：

## 单个配置对象写法

```js
// webpack.config.js
module.exports = {
    entry: './src/index.js',
    // ...
}
```

单个配置对象的写法是我认为在日常开发中最常用，也是最直观的写法。

> 本案例的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/02-%E9%85%8D%E7%BD%AE%E7%BB%93%E6%9E%84/01-%E5%8D%95%E4%B8%AA%E9%85%8D%E7%BD%AE%E5%AF%B9%E8%B1%A1%E5%86%99%E6%B3%95

## 配置数组写法

导出数组的写法如下：

```js
// webpack.config.js
module.exports = [
    {
        entry: './src/index.js',
        // ...
    },
    {
        entry: './src.index.js',
        // ...
    }
]
```

使用数组方式时，`webpack` 会在启动后创建多个 `Compilation` 实例，并行执行构建工作，但需要注意，`Compilation` 实例间基本上不作通讯，这意味着这种并行构建对运行性能并没有任何正向收益，例如某个 Module 在 `Compilation` 实例 A 中完成解析、构建后，在其它 `Compilation` 中依然需要完整经历构建流程，无法直接复用结果。

数组方式主要用于应对“同一份代码打包出多种产物”的场景，例如在构建 `Library` 时，我们通常需要同时构建出 `ESM/CMD/UMD` 等模块方案的产物，如：

```js
// webpack.config.js
module.exports = [
  {
    mode: 'development',
    entry: './src/index.js',
    name: 'commonjs', // 可以便于指定名字执行
    output: {
      filename: 'fun-commonjs.js',
      libraryTarget: 'commonjs',
    },
  },
  {
    mode: 'development',
    entry: './src/index.js',
    name: 'umd', 
    output: {
      filename: 'fun-umd.js',
      libraryTarget: 'umd',
      library: '_', // 浏览器下，使用_作为全局变量访问
    },
  },
]
```

使用配置数组时，还可以通过 `--config-name` 参数指定需要构建的配置对象，例如上例配置中若执行 `npx webpack --config-name=umd` 或 `pnpm build --config-name=umd`，则仅使用数组中 `name='umd'` 的项做构建。

我们发现上述配置中有许多重复配置项，我们可以使用`webpack-merge`来合并简化配置。首先需要先安装该工具。

```shell
$ pnpm add webpack-merge -D
```

然后修改配置即可：

```js
const path = require("node:path")
const { merge } = require("webpack-merge")

const commConfig = {
  mode: "development",
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, './dist')
  }
};

module.exports = [
  merge(commConfig, {
    name: 'commonjs',
    output: {
      filename: 'fun-commonjs.js',
      libraryTarget: 'commonjs',
    },
  }),
  merge(commConfig, {
    name: 'umd',
    output: {
      filename: 'fun-umd.js',
      libraryTarget: 'umd',
      library: '_', // 浏览器下，使用_作为全局变量访问
    },
  }),
]
```

> 本案例的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/02-%E9%85%8D%E7%BD%AE%E7%BB%93%E6%9E%84/02-%E6%95%B0%E7%BB%84%E5%86%99%E6%B3%95

## 配置函数写法

配置函数方式要求在配置文件中导出一个函数，并在函数中返回 Webpack 配置对象，或配置数组，或 `Promise` 对象，如：

```js
// webpack.config.js
module.exports = function(env, argv) {
  // ...
  return {
    mode: 'development',
    entry: './src/index.js'
    // ...
  }
}
```

运行时，`webpack`会传入两个环境参数对象：

- `env`：通过 `--env` 传递的命令行参数，适用于自定义参数，例如：

| 命令：                                                       | `env` 参数值：                                |
| ------------------------------------------------------------ | --------------------------------------------- |
| npx webpack --env prod                                       | { prod: true }                                |
| npx webpack --env prod --env min                             | { prod: true, min: true }                     |
| npx webpack --env platform=app --env production              | { platform: "app", production: true }         |
| npx webpack --env foo=bar=app                                | { foo: "bar=app"}                             |
| npx webpack --env app.platform="staging" --env app.name="test" | { app: { platform: "staging", name: "test" }} |

- `argv`：命令行 `Flags` 参数，支持 `entry`/`output-path`/`mode`/`merge` 等。其中有个参数是`env`，其值就是函数第一个参数的值。

| 命令：                                    | `env` 参数值： | `argv`参数值：                      |
| ----------------------------------------- | -------------- | ----------------------------------- |
| npx webpack --env prod --mode development | { prod: true } | { env: {...}, mode: 'development' } |
| npx webpack --mode production             | {  }           | { env: {},  mode: 'development' }   |

“**配置函数**”这种方式的意义在于，允许用户根据命令行参数动态创建配置对象，可用于实现简单的多环境治理策略。

执行命令为：

```shell
$ npx webpack --env dev --mode production
```

`webpack.config.js`为：

```js
// webpack.config.js
module.exports = function (env, args) {
  return {
    mode: env.dev ? 'development' : 'production',
    entry: './src/index.js',
    devtool: env.dev ? 'eval-cheap-module-source-map' : 'nosources-source-map',
    plugins: [
      new HtmlWebpackPlugin({
        template: './public/index.html',
      }),
    ],
    devServer: {
      port: 8080,
    },
  }
}
```

这样可以根据命令行不同的参数实现不同的效果。

但是这种方式**并不是很推荐使用**，一方面是需要在命令添加大量的参数，二是对于配置内部需要做大量的逻辑判断，使得可读性和维护性下降。因此这种写法了解即可。下面介绍我个人推荐并且在日常中使用的写法。

> 本案例的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/02-%E9%85%8D%E7%BD%AE%E7%BB%93%E6%9E%84/03-%E5%87%BD%E6%95%B0%E5%86%99%E6%B3%95

## 根据不同环境使用不同的配置文件

在现代前端工程化实践中，通常需要将同一个应用项目部署在不同环境(如生产环境、开发环境、测试环境)中，以满足项目参与各方的不同需求。这就要求我们能根据部署环境需求，对同一份代码执行各有侧重的打包策略，例如：

- 开发环境需要使用 `webpack-dev-server` 实现 `Hot Module Replacement`；
- 测试环境需要带上完整的 `Soucemap `内容，以帮助更好地定位问题；
- 生产环境需要尽可能打包出更快、更小、更好的应用代码，确保用户体验。

这种方法的做法是将`webpack`的配置文件分成多个，将多个配置文件公共的部分单独抽取出去，使用`webpack-merge`进行合并。然后在使用的时候配和`--config`来指定配置文件。

文件结构如下：

```shell
|- webpack
└─── webpack.comm.config.js # 公共配置
└─── webpack.dev.config.js # 开发使用
└─── webpack.prod.config.js # 生产使用
```

现在公共配置文件中写入通用配置：

```js
// webpack/webpack.comm.config.js
const path = require('node:path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  entry: './src/index.js',
  module: {
    rules: [
      {
        test: /\.js$/,
        use: ['babel-loader'],
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, '../public/index.html'),
    }),
  ],
}
```

然后编写不同环境下的配置文件内容，并使用`webpack-merge`进行配置合并。

在开发环境中，我们需要考虑开发效率。因此会使用`webpack-dev-server`实现`HMR`，并且设置`sourceMap`。配置文件如下：

```js
// webpack/webpack.dev.config.js
const { merge } = require('webpack-merge')
const commConfig = require('./webpack.comm.config')

module.exports = merge(commConfig, {
  mode: 'development',
  devServer: {
    port: 8080,
    hot: true,
  },
  devtool: 'eval-cheap-module-source-map',
})

```

在生产环境下，我们需要考虑性能和代码安全性，配置文件如下：

```js
const path = require('node:path')
const { merge } = require('webpack-merge')
const commConfig = require('./webpack.comm.config')

module.exports = merge(commConfig, {
  mode: 'production',
  output: {
    path: path.resolve(__dirname, '../dist'),
    clean: true,
  },
  devtool: 'nosources-source-map',
})

```

编写完不同环境下的配置文件后，我们只需要在执行命令的时候，使用不同的命令行参数即可。

在开发环境下启动：

```shell
$ webpack-dev-server --config webpack/webpack.dev.config.js
```

在生产环境下打包：

```shell
$ webpack --config webpack/webpack.prod.config.js
```

> 本案例的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/02-%E9%85%8D%E7%BD%AE%E7%BB%93%E6%9E%84/04-%E6%A0%B9%E6%8D%AE%E7%8E%AF%E5%A2%83%E9%80%89%E6%8B%A9%E9%85%8D%E7%BD%AE%E5%86%99%E6%B3%95