> 本节完整的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/11-%E7%BC%96%E5%86%99loader

如何扩展 `Webpack`？有两种主流方式，一是 `Loader` —— 主要负责将资源内容翻译成 `Webpack` 能够理解、处理的 `JavaScript` 代码；二是 `Plugin` —— 深度介入 `Webpack` 构建过程，**重塑** 构建逻辑。

## 为什么需要 Loader？

为什么 Webpack 需要设计出 Loader 这一扩展方式？本质上是因为计算机世界中的文件资源格式实在太多，不可能一一穷举， 那何不将"**解析**"资源这部分任务开放出去，由第三方实现呢？Loader 正是为了将文件资源的“读”与“处理”逻辑解耦，Webpack 内部只需实现对标准 JavaScript 代码解析/处理能力，由第三方开发者以 Loader 方式补充对特定资源的解析逻辑。

实现上，Loader 通常是一种 mapping 函数形式，接收原始代码内容，返回翻译结果，如：

```js
module.exports = function(source) {
  // 执行各种代码计算
  return modifySource
}
```

在 Webpack 进入构建阶段后，首先会通过 IO 接口读取文件内容，之后调用 [LoaderRunner](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fwebpack%2Floader-runner) 并将文件内容以 `source` 参数形式传递到 Loader 数组，`source` 数据在 Loader 数组内可能会经过若干次形态转换，最终以标准 JavaScript 代码提交给 Webpack 主流程，以此实现内容翻译功能。

`Loader` 函数签名如下：

```js
module.exports = function(source, sourceMap?, data?) {
  return source;
};
```

`Loader` 接收三个参数，分别为：

- `source`：资源输入，对于第一个执行的 `Loader` 为资源文件的内容；后续执行的 `Loader` 则为前一个 `Loader` 的执行结果，可能是字符串，也可能是代码的 AST 结构；
- `sourceMap`: 可选参数，代码的 `sourcemap` 结构；
- `data`: 可选参数，其它需要在 Loader 链中传递的信息，比如 `posthtml/posthtml-loader` 就会通过这个参数传递额外的 AST 对象。

其中 `source` 是最重要的参数，大多数 Loader 要做的事情就是将 `source` 转译为另一种形式的 `output` 。

## Loader简单示例

接下来我们开发一个简单的`Loader`，要求可以在`loader`调用源文件中的方法，并且打印出其他的信息。

目录如下：

```shell
├── node_modules
├── package.json
├── pnpm-lock.yaml
├── loader  # 自定义loader
│   └── index.js
├── public
│    └── index.html
├── src
│    └── index.js
└── webpack.config.js
```

我们着重编写`loader/index.js`文件：

```js
// loader/index.js
module.exports = function (source) {
  const newSource = `
    ${source}
    console.log(sum(1, 2))
    console.log(3333)
    console.log('version', ${version})
    console.log('webpack', ${webpack})
  `
  
  return newSourc
}
```

代码逻辑很简单，核心功能只是在原来 `source` 上拼接了一些文本。

我们的代码入口文件如下：

```js
// src/index.js
const sum = (a, b) => a + b
const sub = (a, b) => a - b

console.log(111)
```

然后在`webpack.config.js`引入该文件：

```js
// webpack.config.js
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [{
          loader: path.resolve(__dirname, "../dist/index.js")
        }]
      }
    ]
  }
}
```

然后在命令行执行打包：

```shell
$ npx webpack
```

打包后的`main.js`如下：

```js
// dist/main.js
console.log(111)
console.log(3)
console.log(3333)
console.log("version",2)
console.log("webpack",!0);
```

除了第一条打印，后面的打印的都是我们在loader中添加的。

## 使用上下文接口

除了作为内容转换器外，Loader 运行过程还可以通过一些[上下文接口](https://link.juejin.cn/?target=https%3A%2F%2Fwebpack.js.org%2Fapi%2Floaders%2F%23thisaddcontextdependency)，**有限制**地影响 Webpack 编译过程，从而产生内容转换之外的副作用。上下文接口将在运行 Loader 时以 `this` 方式注入到 Loader 函数

Webpack 官网对 [Loader Context](https://webpack.docschina.org/api/loaders#the-loader-context) 已经有比较详细的说明，这里简单介绍几个比较常用的接口：

- `fs`：Compilation 对象的 `inputFileSystem` 属性，我们可以通过这个对象获取更多资源文件的内容；
- `resource`：当前文件路径；
- `resourceQuery`：文件请求参数，例如 `import "./a?foo=bar"` 的 `resourceQuery` 值为 `?foo=bar`；
- `callback`：可用于返回多个结果；
- `getOptions`：用于获取当前 Loader 的配置对象；
- `async`：用于声明这是一个异步 Loader，开发者需要通过 `async` 接口返回的 `callback` 函数传递处理结果；
- `emitWarning`：添加警告；
- `emitError`：添加错误信息，注意这不会中断 Webpack 运行；
- `emitFile`：用于直接写出一个产物文件，例如 `file-loader` 依赖该接口写出 Chunk 之外的产物；
- `addDependency`：将 `dep` 文件添加为编译依赖，当 `dep` 文件内容发生变化时，会触发当前文件的重新构建；

## 取消 Loader 缓存

需要注意，Loader 中执行的各种资源内容转译操作通常都是 CPU 密集型 —— 这放在 JavaScript 单线程架构下可能导致性能问题；又或者异步 Loader 会挂起后续的加载器队列直到异步 Loader 触发回调，稍微不注意就可能导致整个加载器链条的执行时间过长。

为此，Webpack 默认会缓存 Loader 的执行结果直到模块或模块所依赖的其它资源发生变化，我们也可以通过 `this.cacheable` 接口显式关闭缓存：

```js
module.exports = function(source) {
  this.cacheable(false);
  // ...
  return output;
};
```

