## 结构化理解 Webpack 配置项

`Webpack` 原生提供了上百种配置项，这些配置最终都会作用于 `Webpack` 打包过程的不同阶段，因此我们可以从流程角度更框架性、结构化地了解各项配置的作用。

`Webpack` 构建的流程化大致为：

![Weback构建流程](../../../docs/image/前端构建工具/Webpack5学习/02.png)

1. **初始化（Initialization）**
   - 读取配置：`Webpack` 加载并解析 `webpack.config.js` 或其他指定的配置文件。
   - 创建 Compiler 实例：Compiler 是整个构建过程的核心对象，负责协调和执行打包任务。
2. **编译准备（Pre-Compilation）**
   - 缓存检查：`Webpack` 可能会查看缓存以决定是否需要重新构建。
   - 插件预处理：触发插件生命周期中的相关钩子方法，如 `compiler.hooks.beforeCompile`，允许插件在正式编译前进行准备工作。
3. **开始编译（Compilation）**
   - 入口模块解析：从配置中的 `entry` 配置项找到项目入口点，并递归解析所有依赖模块，形成一个模块依赖图。
   - 模块加载与转换：`Loader` 被应用到每个模块上，对源代码进行语法转换、编译等操作。
   - 生成依赖关系图：`Webpack` 根据模块间的相互引用关系构建出完整的依赖树。
4. **后处理（Optimization）**
   - 所有模块递归处理完毕后开始执行后处理，包括模块合并、注入运行时、产物优化等。
   - 代码分割（`Code Splitting`）：根据配置和模块之间的动态导入关系，将代码拆分为多个 `chunk`。
   - `Tree Shaking`：通过静态分析去除未使用的模块和变量，减少输出文件体积。
   - `Scope Hoisting`：提升代码复用性，减少函数作用域包裹，优化生成代码。
5. **生成产物（Emitting Assets）**
   - 输出资源：`Webpack` 根据编译结果生成最终的 `bundle` 文件，包括 `JavaScript`、`CSS`、图片等各类资源。
   - 写入文件系统：将打包好的 `bundle` 文件以及任何额外的资源写入到指定的输出目录中。
   - 清理旧文件：如果配置了清理功能，`Webpack` 会删除之前构建遗留下来的不需要的文件。
   - 构建结束通知：调用 `compiler.hooks.done` 等钩子，表示打包过程已完成。

我们可以再进一步将上述的构建流程简化为：

![进一步简化后的Weback构建流程](../../../docs/image/前端构建工具/Webpack5学习/03.png)

我们可以将这些流程与`Webpack`中的配置相结合：

- 输入输出：
  - `entry`：用于定义项目入口文件，`Webpack` 会从这些入口文件递归找出所有项目文件。
  - `context`：项目执行上下文路径。
  - `output`：配置产物输出路径、名称等。
- 模块处理：
  - `resolve`：用于配置模块路径解析规则，可用于帮助 `Webpack` 更精确、高效地找到指定模块。
  - `module`：用于配置模块加载规则，例如针对什么类型的资源需要使用哪些 `Loader` 进行处理。
  - `externals`：用于声明外部资源，`Webpack` 会直接忽略这部分资源，跳过这些资源的解析、打包操作。
- 后处理：
  - `optimization`：用于控制如何优化产物包体积，内置 `Dead Code Elimination`、`Scope Hoisting`、代码混淆、代码压缩等功能。
  - `target`：用于配置编译产物的目标运行环境，支持 `web`、`node`、`electron` 等值，不同值最终产物会有所差异。
  - `mode`：编译模式短语，支持 `development`、`production` 等值，可以理解为一种声明环境的短语。

除了核心的打包功能之外，`Webpack` 还提供了一系列用于提升研发效率的工具，大体上可划分为：

- 开发效率类：
  - `watch`：用于配置持续监听文件变化，持续构建。
  - `devtool`：用于配置产物 `Sourcemap` 生成规则。
  - `devServer`：用于配置与 `HMR` 强相关的开发服务器功能。
- 性能优化类：
  - `cache`：`Webpack 5` 之后，该项用于控制如何缓存编译过程信息与编译结果。
  - `performance`：用于配置当产物大小超过阈值时，如何通知开发者。
- 日志类：
  - `stats`：用于精确地控制编译过程的日志内容，在做比较细致的性能调试时非常有用。
  - `infrastructureLogging`：用于控制日志输出方式，例如可以通过该配置将日志输出到磁盘文件。
- 等等

逻辑上，每一个工具类配置都在主流程之外提供额外的工程化能力，例如 `devtool` 用于配置产物 `Sourcemap` 生成规则，与 `Sourcemap` 强相关；`devServer` 用于配置与 `HMR` 相关的开发服务器功能；`watch` 用于实现持续监听、构建。

## Webpack初体验

接下来，我们编写一个简单的项目来体验一下`Webpack`。默认的`webpack.config.js`在编写时没有语法提示，为了获得最佳的编写感受，我们可以在头部加上**类型标注**。

```js
// webpack.config.js

/**
 * @type {import('webpack').Configuration}
 */
module.exports = {
  // ...
}
```

然后开始我们的简单项目，要求可以识别`js`和`css`，并应用到`public/index.html`上。

文件的目录结构如下：

```shell
├── package.json
├── pnpm-lock.yaml
├── public
│   └── index.html
├── src
│   ├── index.css
│   └── index.js
└── webpack.config.js
```

首先先安装`webpack`工具的相关依赖：

```shell
$ pnpm add webpack webpack-cli webpack-dev-server -D
```

- `webpack`： 是核心的打包工具，负责项目的构建工作。
- `webpack-cli`： 用来方便地通过命令行执行 `webpack` 的构建任务。
- `webpack-dev-server`： 为开发环境提供了一个实时编译和刷新的本地服务器，提高了开发体验和效率。

因为我们需要一个`html`文件来存放我们页面相关的内容，并且将我们的`js`和`css`插入到页面中，因此我们需要安装相关依赖：

```shell
$ pnpm add html-webpack-plugin -D
```

然后安装对应的`loader`，至于`loader`的用处，后面会介绍到：

```shell
$ pnpm add css-loader style-loader babel-loader @babel/core @babel/preset-env -D
```

编写`webpack.config.js`：

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
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
      {
        test: /\.js$/,
        use: [
          {
            loader: 'babel-loader',
            options: {
              presets: ['@babel/preset-env'],
            },
          },
        ],
      },
    ],
  },
  devServer: {
    port: 8080,
    hot: true,
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, './public/index.html'),
    }),
  ],
}


```

这样我们可以在当前目录的终端中直接启动或打包：

```shell
# 启动
npx webpack serve

# 打包
npx webpack --mode production
```

这样每次启动都要拼写很长一段，我们可以将这样命令写到`npm scripts`中：

```js
// package.json
{
  // ...
  "scripts": {
    "dev": "webpack serve",
    "build": "webpack --mode production"
  },
}

```

> 本案例完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/01-%E5%88%9D%E4%BD%93%E9%AA%8C
