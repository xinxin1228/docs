`webpack`中的配置项目非常多，一个一个学完很难，关键又很难记住，所以我们先只学会使用较高的配置项即可。至于其他的配置，什么时候用到了什么时候再去[webpack官网](https://webpack.js.org/concepts/)去查吧。

接下来看一下`webpakc中常用的核心配置`有哪些:

- `entry`：声明项目入口文件，`webpack `会从这个文件开始递归找出所有文件依赖。
- `output`：输出，声明构建结果的存放位置。
- `target`：用于配置编译产物的目标运行环境，支持 `web`、`node`、`electron` 等值，不同值最终产物会有所差异。
- `mode`：控制`webpack`的工作模式，影响优化策略和开发工具的行为。支持 `development`、`production` 和`none`，`webpack`会根据该属性推断默认配置。
- `optimization`：用于控制如何优化产物包体积，内置 `Dead Code Elimination`、`Scope Hoisting`、代码混淆、代码压缩等功能。
- `module`：用于声明模块加载规则，例如针对什么类型的资源需要使用哪些`loader`进行处理。
- `plugin`：`webpack`插件列表，它们在构建流程中的各个阶段发挥作用，如压缩代码、生成资源映射表、提取`CSS`文件、注入环境变量等。

- `resolve`：定义了`webpack`在查找模块时如何解析模块请求，例如自动补全扩展名、`alias`别名、模块搜索范围等。
- `devtool`：控制`Source Map`的生成方式，有助于在开发模式对代码进行调试。
- `devServer`：在开发阶段提供本地静态和热更新服务。

## entry

`webpack` 的基本运行逻辑是从 **「入口文件」** 开始，递归加载、构建所有项目资源，所以项目必须使用 `entry` 配置项明确声明项目入口。`entry` 配置规则比较复杂，支持如下形态：

- **字符串【单入口】：**表示单入口打包，打包后，默认生成`main.js`文件，保存在当前目录的`dist`目录。
- **对象【多入口】：**对象形态功能比较完备，除了可以指定入口文件列表外，还可以指定入口依赖、`runtime `打包方式等。
- **数组【单入口】：**表示单入口打包，数组的最后一个文件为打包的入口文件，在打包前会先把数组的其余文件预先加载到入口文件中
  打包后，默认生成`main.js`文件，保存在当前目录的`dist`目录。
- **函数【多入口】：**如果入口文件的设置涉及到一些逻辑处理，可以把值设为函数。 函数的返回值可以是**字符串、数组、对象**中的任意一种形式。最后打包的结果与上面三种形式单独写的效果是致的。

```js
// webpack.config.js
module.exports = {
    // ...
    
    // 单入口 字符串形式
    entry: './src/a.js',
    // 单入口 数组形式 这样打包两个文件会融合到一个文件中
    entry: ['./src/a.js', './src/b.js'],
    // 多入口 对象形式 这样打包会单独打包出两个文件 
    entry: {
        // 打包出的成品是 a.js
        a: './index/a.js', 
        // 打包出的成品是 bbb.js
        b: {
            import: "./index/b.js",
            filename: 'bbb.js'
        }
    },
    // 多入口 函数形式 可返回上面的几种形式
    entry() {
      // 返回字符串
      return './src/a.js',
      
      // 返回数组
      return ['./src/a.js', './src/b.js']
    
      // 返回对象
      return {
        a: './src/a.js',
        b: './src/b.js',
      }
    }
}
```

这其中，**「对象」** 形态的配置逻辑最为复杂，支持如下配置属性：

- `import`：声明入口文件，支持路径字符串或路径数组(多入口)。
- `dependOn`：声明该入口的前置依赖 `Bundle`。
- `runtime`：设置该入口的 `Runtime Chunk`，若该属性不为空，`webpack` 会将该入口的运行时代码抽离成单独的 Bundle。
- `filename`：效果与 `output.filename` 类同，用于声明该模块构建产物路径。
- `library`：声明该入口的 `output.library` 配置，一般在构建 `NPM Library` 时使用。
- `publicPath`：效果与 `output.publicPath` 相同，用于声明该入口文件的发布 URL。
- `chunkLoading`：效果与 `output.chunkLoading` 相同，用于声明异步模块加载的技术方案，支持 `false/jsonp/require/import` 等值。
- `asyncChunks`：效果与 `output.asyncChunks` 相同，用于声明是否支持异步模块加载，默认值为 `true`。

> 本案例的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/03-%E5%B8%B8%E7%94%A8%E7%9A%84%E6%A0%B8%E5%BF%83%E9%85%8D%E7%BD%AE/entry

## output

`webpack` 的 `output` 配置项用于声明：如何输出构建结果，比如产物放在什么地方、文件名是什么、文件编码等。`output` 支持许多子配置项，包括：

- `output.path`：声明产物放在什么文件目录下。
- `output.filename`：声明产物文件名规则，支持 `[name]/[hash]` 等占位符。
- `output.chunkFilename`：表示的是入口文件打包过程中生成的其它的模块名。
- `output.publicPath`：文件发布路径，在 Web 应用中使用率较高。
- `output.clean`：是否自动清除 `path` 目录下的内容，调试时特别好用。
- `output.library`：`NPM Library` 形态下的一些产物特性，例如：`library` 名称、模块化(UMD/CMD 等)规范；
- `output.chunkLoading`：声明加载异步模块的技术方案，支持 `false/jsonp/require` 等方式。

### filename和chunkFilename的区别

比如`index.js`文件里有`import('./a.js')`，其中`a.js`就被单独打包成了`85.bundle.js`，这个名称就是默认的`output.chunkFilename`。

需要注意的是：`import('./a.js')`是动态导入，而`import './a.js'`是静态导入。

### hash、fullhash、chunkhash、contenthash 的区别

> 在前面学习`output.filename`和`output.chunkFilename`属性的配置，我们提到了下面这种写法。

```js
output.filename = "[name].js";
output.chunkFilename = '[id].js';
```

> 我们知道`[]`方括号表示一个占位符，`name`和`id`表示一个特定的动态值。
>
> 其实，占位符中还有`hash、fullhash、chunkhash、contenthash`值，这些值都是根据文件内生成的唯一的 hash 值。

| hash 值     | 说明                                                       |
| :---------- | :--------------------------------------------------------- |
| hash        | 根据打包中所有文件计算出的 hash 值。                       |
| fullhash    | webpack5 提出的，它用来替代之前的 hash                     |
| chunkhash   | 根据打包的当前模块（JS)计算出来的 hash 值。                |
| contenthash | 他与 chunkhash 很像，不过他主要用于计算 CSS 文件的 hash 值 |

使用演示：

```js
// webpack.config.js
module.exports = {
  // ...
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: '[name][fullhash:8].js',
    chunkFilename: "[chunkhash:8].js",
  }
}
```

接下里我们重点关注一下`libaray`的用法，比如要发布一个`npm`包：

```js
// webpack.config.js
const path = require('node:path')

module.exports = {
  // ...
  output: {
    path: path.resolve(__dirname, './src/index.js'),
    filename: 'bundle.js',
    // 指定打包的产物信息
    library: {
      name: '_', // 浏览器下，使用_作为全局变量访问
      type: 'umd' // 采用umd方式打包
    },
    
    // 上面的写法也可以改写成下面的写法
    library: '_',
    libraryTarget: 'umd',
  },
  // ...
}
```

> 本案例的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/03-%E5%B8%B8%E7%94%A8%E7%9A%84%E6%A0%B8%E5%BF%83%E9%85%8D%E7%BD%AE/output

## mode

`webpack` 的 `mode` 配置项是一个核心选项，它用于指定当前的构建环境。通过设置不同的模式，`webpack` 会自动应用相应的优化策略、内置插件和默认配置，以适应开发（`development`）或生产（`production`）等不同场景的需求。

以下是 `webpack` 中 `mode` 属性的详细说明：

- **development**：
  - 当`mode`设置为`'development'`时，`webpack` 将启用一套有利于快速迭代和调试的配置：
    - 不会进行代码压缩和优化，保持源码易于阅读和调试。
    - `Source Maps` 被默认启用，允许开发者在浏览器中直接查看原始源代码而不是打包后的代码，便于定位错误。
    - `HMR`（`Hot Module Replacement`，热模块替换）在配合 `DevServer` 使用时可以实现模块热更新，无需刷新页面即可看到修改效果。
- **production**：
  - 当`mode`设置为`'production'`时，`webpack` 会采取一系列性能优化措施：
    - 开启代码压缩：使用如 `TerserWebpackPlugin` 等工具对 `JavaScript` 和 `CSS` 进行压缩，减少文件大小。
    - 删除无用的代码：例如删除未使用的变量、函数等。
    - `Tree Shaking`：基于 `ES6` 模块静态分析，去除未被引用的 `export` 导出内容。
    - 压缩并提取 `CSS`：如果使用了相关插件，`CSS` 文件会被压缩并可能提取到单独文件中。
    - 强化代码分割：更激进地拆分代码，生成更小的 `chunks`，利于加载速度。
- **none** 或自定义模式：
  - 如果 `mode` 设置为 `'none'` 或者没有设置，则不会应用任何默认优化，需要手动配置所有所需的插件和优化策略。
  - 可以自定义 `mode`，并根据自定义模式编写对应的配置逻辑，但通常情况下，大部分项目只会使用预设的 `'development' `和 `'production'` 模式。

可以将本案例的代码`clone`下来，分别执行不同的命令来查看`js`文件的变化。

> 本案例的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/03-%E5%B8%B8%E7%94%A8%E7%9A%84%E6%A0%B8%E5%BF%83%E9%85%8D%E7%BD%AE/mode

## module

`webpack` 的 `module` 配置项是其配置文件 (`webpack.config.js`) 中的一个核心部分，它主要负责处理和转换不同类型的模块。在 `webpack` 中，一切皆模块，包括 `JavaScript`、`CSS`、图片等资源都可以被当作模块来处理。`module` 部分的配置主要关注如何使用加载器（`loaders`）对这些模块进行预处理或转译。

`loader` 是 `webpack` 构建流程中的一个重要环节，它们是链式调用的转换器，可以将一种类型的文件转换成另一种类型，或者对某种类型的文件进行特定的编译、压缩等操作。

以下是对 `module` 配置的详细解析：

- **rules**:

  - `module.rules` 是一个数组，其中包含多个规则对象。每个规则对象定义了针对不同类型的模块应该如何去处理。

  - 规则通常会包含以下属性：

    - `test`: 用于匹配模块的文件路径或者文件内容的正则表达式。当模块与该正则表达式匹配时，对应的加载器将被应用到这个模块上。

    - `use`: 指定一个或多个加载器，可以是一个字符串或数组。如果是数组，则从右到左执行（即从后往前）。加载器按照顺序处理模块内容，例如先通过 `Babel` 转译 `ES6` 语法，然后通过 `css-loader` 处理 `css` 文件。可以这样理解：

      ```js
      funa(funb(x))
      ```

      肯定是`funb`先执行，那个也是同样的道理。

    - `include/exclude`: 可选，用于指定哪些文件应当包含在规则内或排除在外，它们也是正则表达式或绝对路径。

    - `type`: 可以用来指定模块的类型，比如 `'javascript/auto'` 或 `'javascript/esm'`，但大多数情况下此属性可不显式设置，`webpack` 会自动根据文件扩展名推断模块类型。

- **noParse**:

  - 可选配置项，用于告诉 `webpack` 不要尝试解析某些大型库的模块依赖关系。这在大型且内部已做优化不需要再进一步处理的库中很有用，能够提高构建速度。

- **parser**:

  - 在一些高级场景下，可以自定义模块的解析器，控制如何解析模块中的导入导出语句。

接下来我们详细介绍和使用一下常用的`loader`都有哪些。

为了不打断进度，避免造成学习和阅读上的心智负担，我将常用的`loader`单独划分出一个章节，避免在这里嵌套太深。

[Webpack中常用的Loader]()（点击即可前往阅读）

## plugins

`webpack` 的 `plugins` 是构建过程中执行特定任务的扩展点，它们参与到整个编译生命周期中，可以在多个关键阶段（如初始化、编译开始前、模块编译时、生成资源时、压缩资源时、输出资源后等）添加自定义处理逻辑。插件可以用来优化构建结果、注入环境变量、生成额外的文件、实现代码分割、资源管理等功能。

我将常用的`plugin`单独划分出一个章节，点击下方列表即可前往阅读。

[Webpack中常用的Plugin]()（点击即可前往阅读）

## devtool

`webpack` 的 `devtool` 配置选项是用于控制是否生成以及如何生成 `Source Map`（源代码映射）的。`Source Map` 是一种将打包后的混淆、压缩或转换过的代码映射回原始源代码的技术，它极大地帮助开发者在浏览器的开发者工具中调试经过` Webpack` 打包处理后的 `JavaScript` 代码。

`devtool`默认值在生产环境下，也就是`mode`为`production`，值是`false`。在开发环境下也就是`mode`为`development`，值是`eval`。

在选择 `devtool` 时，需要平衡以下几个因素：

- **调试需求**：更详尽的映射有助于定位问题，但可能牺牲速度。
- **构建速度**：某些选项在开发过程中能实现热更新的快速重建。
- **输出大小**：内联 `Source Map` 会影响最终的 `bundle` 体积，而外链则会产生额外的 `.map` 文件。
- **安全性考虑**：在生产环境中，有时并不希望向客户端公开原始源代码。

那么在不同的环境下，应该怎么选择呢？

### production

线上环境官方推荐的 devtool 有4种：

- none
- source-map
- hidden-source-map
- nosources-source-map 线上环境没有绝对的最优选择一说，根据自己业务需要去选择即可，很多项目也是选择除上述4种之外的 `cheap-module-source-map` 选项。

### development

开发环境选择就比较容易了，只需要考虑打包速度快、调试方便，官方推荐以下4种：

- eval
- eval-source-map
- eval-cheap-source-map
- eval-cheap-module-source-map 大多数情况下我们选择 `eval-cheap-module-source-map` 即可。

因此最后我们选择：

- 本地开发我们配置`eval-cheap-module-source-map`使用最详细的`SourceMap`，既能保证速度，也可以得到详细的信息。

- 测试环境我们有可能需要在线调试错误，所以配置`source-map`。

- 生产环境直接禁用`SourceMap`或设置为`nosources-source-map`，一是能缩小构建后包的体积，其次能防止别人窃取源码。在报错时也可以得到文件的位置信息，但得不到文件内容。

> 本案例的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/03-%E5%B8%B8%E7%94%A8%E7%9A%84%E6%A0%B8%E5%BF%83%E9%85%8D%E7%BD%AE/devtool

## devServer

`webpack-dev-server` 是一个基于 `Node.js` 的小型 `Express` 服务器，用于在开发阶段提供本地静态和热更新服务。这个工具与 `webpack` 配合使用时，能够显著提升前端开发效率，因为它可以实时编译、打包资源，并且支持自动刷新浏览器（`live reloading`）和热模块替换（`Hot Module Replacement, HMR`）等功能。

使用之前需要先进行安装：

```shell
$ pnpm add webpack-dev-server -D
```

以下是`devServer`常用的配置，完整的配置参考[Webpack-dev-server官网文档](https://webpack.js.org/configuration/dev-server/)。

```js
// webpack.config.js
module.exports = {
  // ...
  devServer: {
    static: path.resolve(__dirname, "./devdist"), // 静态文件目录
    port: 8080, // 端口号
    open:true,  // 自动打开浏览器
    compress: true, // 启用gzip压缩
    hot: true, // 启用热模块替换
    proxy: {
      '/api': 'http://localhost:3000', // 请求/api/users将把请求代理到http://localhost:3000/api/users
      secure: false, // 接受在 HTTPS 上运行且证书无效的后端服务器
    },
  }
  // ...
}
```

在`npm scripts`中使用:

```js
// package.json

// ...
scripts: {
  "dev": "webpack-dev-server",
  // 或者下面的方式
  "dev2": "webpack serve"
},
// ...
```

在`cli`中使用

```shell
$ npx webpack serve
# 或
$ npx webpack-dev-server
```

> 本案例的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/03-%E5%B8%B8%E7%94%A8%E7%9A%84%E6%A0%B8%E5%BF%83%E9%85%8D%E7%BD%AE/devServer 

## resolve

`webpack` 中的 `resolve` 配置对象用于配置模块如何解析。在 `webpack` 的构建过程中，当遇到 `import` 或 `require` 等模块导入语句时，`webpack` 会根据 `resolve` 配置来确定如何查找和解析这些模块。以下是 `resolve` 配置中常用的关键选项详解：

### **alias**

通过别名设置模块导入路径的映射关系。例如，你可以创建一个快捷方式，将长路径或者频繁引用的第三方库路径映射到一个简短的别名上。

```js
// webpack.config.js
module.exports = {
  // ...
  resolve: {
  	alias: {
      '@': path.resolve(__dirname, '@'),
    	'@components': path.resolve(__dirname, 'src/components'),
  	}
	}
}
```

然后在代码中可以使用：

```js
- import { sum } from './components/js/sum'
+ import { sum } from '@components/js/sum'
```

 这样可以代替原本冗长的相对路径。

### **extensions**

指定导入模块时自动尝试添加的扩展名顺序。这允许开发者省略文件扩展名。它的默认值是 ：

```javascript
resolve: {
  extensions: ['.wasm', '.mjs', '.js', '.json'],
}
```

这意味着当导入一个模块时没有明确指定扩展名，`webpack` 将按照 `.wasm`、`.mjs`、`.js` 和 `.json` 的顺序依次尝试添加这些扩展名来查找文件是否存在。

我们项目中存在后缀为`.jsx`的文件，默认的`extensions`中并没有`.jsx`选项，我们将其添加。

```js
// webpack.config.js
module.exports = {
  // ...
  resolve: {
  	extensions: ['.js', '.jsx']
	}
}
```

然后在代码中可以使用：

```js
- import Hello from './components/jsx/Hello.jsx' 
+ import Hello from './components/jsx/Hello'
```

> 本案例的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/03-%E5%B8%B8%E7%94%A8%E7%9A%84%E6%A0%B8%E5%BF%83%E9%85%8D%E7%BD%AE/resolve