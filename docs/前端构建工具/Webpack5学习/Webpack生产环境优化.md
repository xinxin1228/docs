## 摇树优化 Tree Shaking

在实际开发中，我们通常会用到一些第三方的工具函数，如果没有经过任何特殊处理的话，打包时会引入整个包。但实际开发中我们可能只用到了里面极小部分的功能（API），这样将整个工具函数的代码都打包进来，体积就太大了。

`Tree Sharking`可以帮我们检测模块中没有用到的那块代码，并在 Webpack 打包时将没有使用到的代码块移除，减小打包后的资源体积。它的名字也非常形象，通过摇晃树，把树上干枯无用的叶子摇掉。

`Webpack5` 中**内置**了这个功能，在**生产环境打包**时，Webpack 会自动开启`Tree Shaking`功能，将没有用到的那部分代码给删除。

我们简单创建项目来演示一下。

目录结构如下：

```shell
├── node_modules
├── package.json
├── pnpm-lock.yaml
├── src
│   └── index.js
└── webpack.config.js
```

我们在`index.js`中编写如下内容：

```js
// src/index.js
const sum = (a, b) => a + b
const sub = (a, b) => a - b

console.log(1)
```

`webpack.config.js`文件内容如下：

```js
// webpack.config.js
module.exports = {
  mode: 'production',
  entry: './src/index.js',
}
```

接下来，在当前根目录下执行`npx webpack`打包，打包后生成的`main.js`文件内容如下：

```js
console.log(1)
```

> 本节完整的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/10-%E7%94%9F%E4%BA%A7%E7%8E%AF%E5%A2%83%E4%BC%98%E5%8C%96/01_tree-shaking

## 资源压缩

资源压缩的主要目的是减小文件体积，以提升页面加载速度或降低带宽消耗等。资源压缩通常发生在生产环境打包的最后一个环节，开发环境打包是不需要进行资源压缩处理的。

资源压缩主要是对 HTML、JS、CSS 文件、图片进行压缩，通常的做法就是整个文件或大段的代码压缩成一行，比如：去掉空格，空行，删除注释，把较长的变量名替换成较短的变量名等。

详情压缩内容请看：[Webpack如何压缩代码](/docs/前端构建工具/Webpack5学习/Webpack如何压缩代码.md)

## 缩小查找范围

在 `Webpack` 打包时，我们可以通过以下几个方面来减少打包所需要的时间：

- 排除那些不需要被预处理器解析的模块。
- 帮助 `Webpack` 快速找到需要的模块。
- 帮助 `Webpack` 识别不带后缀名的文件。

### 配置预处理器的 exclude 与 include

在使用预处理器解析模块的时候，可以通过配置 `exclude` 和 `include` 来减少需要预除理的模块：

- exclude： 排除不需要使用当前预处理器来处理的文件夹目录。
- include：指定当前预处理器只对哪些目录中匹配的文件做预处理。

> 注：当 exclude 与 include 同时指定时，以 exclude 的设置为主。

```js
module.exports = {
  // ....{
  module: {
    rules: [
      // 预处理器
      {
        test: /\.js$/,
        exclude: /node_modules/,
        include: /src/,
        use: {
          loader: "babel-loader",
        },
      },
    ],
  },
};
```

### resolve.modules

- `resolve.modules` 用来配置 Webpack 如何搜寻第三方模块的路径。
- `resolve.modules` 的值可以是相对路径，也可以是绝对路径，他的默认值是`['node_modules']`这是一个相对路径。

当我们使用 import 等语句来加载第三方模块时，如下：

```js
import axios from 'axios'
```

上面代码表示，Webpack 在打包时，默认会在当前目录的`./node_modules`下搜索 axios 模块，如果没有找到，则会到上一级目录`../node_modules`下找，如果还没找到，则会再去`../../node_modules`下找，以此类推，直到找到为止。如果找不到，最后就会抛出错误。

我们的第三方模块都是保存在项目根目录的`node_modules`下，因此没有必要一级一级向上找。主要考虑的时，当我们的地址写错时，一层一层向上找，最后找不到再抛出错误，浪费了大量的时间。所以，我们可以指定第三方模块的搜索地址为绝对地址，就是我们当前项目目录下的`node_modules`文件夹中找，找不到就直接抛错，

以下是`resolve.modules`的配置

```js
// webpack.config.js
module.exports = {
  // ...
  resolve: {
    modules: [path.resolve(__dirname, "node_modules")],
  },
};
```

### resolve.extensions

在我们引入（导入）自定义模块时，我们通常会采用以下简写形式

```js
import { username } from "./login";
```

以上代码并没有指定 login 文件的后缀名是以`.js`还是`.json`等其它格式。那 Webpack 是如何知道这个文件最后是以`.js`结尾的后缀名呢？

`resolve.extensions`就是 Webpack 识别不带后缀名文件的关键。他的默认值是：`['.js', '.json', '.wasm']`。

Webpack 在解析到这些不带后缀名的文件后，会按数组`['.js', '.json', '.wasm']`中的元素从头到尾的顺序尝试找到对应的文件，如果找到了，后面的格式就不匹配了，如果没找到一直往后匹配，直到找到为止。如果都匹配完还没找到就会抛出错误。

如果`resolve.extensions`指定的数组中的项越多，那 Webpack 尝试搜寻的次数就越多，这会影响 Webpack 的解析速度。所以我们可以根据我们的实际项目来合理的配置`resolve.extensions`的值。

**配置的规则：**

- 出现频率高的放在最前，以免能最快找到对应文件
- 没有用到的后缀名，不要出现，这样在找不到对应文件后，会尽快抛出错误

配置代码如下：

```js
// webpack.config.js
module.exports = {
  //...
  resolve: {
    extensions: [".js", ".jsx", ".ts", '.tsx'],
  },
};
```

> 本节完整的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/10-%E7%94%9F%E4%BA%A7%E7%8E%AF%E5%A2%83%E4%BC%98%E5%8C%96/02_resolve
