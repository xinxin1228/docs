> 本节完整的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/04-loader

## 什么是loader

在`webpack` 配置中，`loader` 是一系列可配置的函数链，用于对源代码文件进行转换和处理。比如`webpack`默认不识别`.css`文件和`.less`文件，我们可以利用对应的`css-loader`和`less-loader`来处理转换对应的文件。

下面介绍几种日常使用到的`loader`：

## JavaScript 和 TypeScript 相关的 Loader

### babel-loader

可以将最新或实验性 `JavaScript` 语法（如 `ES6+`）以及 `JSX`的代码转换为浏览器和旧版 `Node.js` 环境都能理解和执行的语法。

将`ES6+`的语法转化为向后兼容的`javascript`代码。

先安装对应的loader：

```shell
$ pnpm add babel-loader @babel/core @babel/preset-env -D
```

然后在`module.rules`中使用：

```js
// webpack.config.js
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env'],
        },
      },
    ],
  },
  // ...
}

```

识别并转换`.jsx`文件，需要额外安装`@babel/preset-react`

```shell
$ pnpm add @babel/preset-react -D
```

然后在`module.rules`中使用：

```js
// webpack.config.js
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env', '@babel/preset-react'],
        },
      },
    ],
  },
  // ...
}

```

### ts-loader

用于将`.typescript`的文件编译为`.javascript`：

安装：

```shell
$ pnpm add typescript ts-loader -D
```

使用：

```js
const path = require('node:path')

module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: 'ts-loader'
      },
    ],
  },
  resolve: {
    extensions: ['.ts', '.js'],
  }
}
```

## css样式转换loader

### css-loader

该 `loader` 会将 `CSS` 等价翻译为形如 `module.exports = '${css}'` 的`JavaScript` 代码，使得 `webpack` 能够如同处理 `JavsScript` 代码一样解析 `CSS` 内容与资源依赖。

安装：

```shell
$ pnpm add css-loader -D
```

使用：

```js
// webpack.config.js
module.exports = {
  // ...
  module: {
     rules: [
      {
        test: /\.css$/,
        use: ['css-loader'],
      },
    ],
  }
}
```

经过 `css-loader` 处理后，`CSS` 代码会被转译为等价 `JS` 字符串，但这些字符串还不会对页面样式产生影响，需要继续接入 `style-loader` 。

### style-loader

该 `loader` 将在产物中注入一系列 `runtime` 代码，这些代码会将 `CSS` 内容注入到页面的 `<style>` 标签，使得样式生效。

安装：

```shell
$ pnpm add style-loader -D
```

使用

```js
// webpack.config.js
module.exports = {
  // ...
  module: {
     rules: [
      {
        test: /\.css$/,
        use: ['style-loader' ,'css-loader'],
      },
    ],
  }
}
```

上述配置语义上相当于 `style-loader(css-loader(css))` 链式调用。

经过 `style-loader` + `css-loader` 处理后，样式代码最终会被写入 `Bundle` 文件，并在运行时通过 `style` 标签注入到页面。这种将 `JS`、`CSS` 代码合并进同一个产物文件的方式有几个问题：

- `JS`、`CSS` 资源无法并行加载，从而降低页面性能；

- 资源缓存粒度变大，`JS`、`CSS` 任意一种变更都会致使缓存失效。

因此，**生产环境**中通常会用 `mini-css-extract-plugin`插件替代 `style-loader`，将样式代码抽离成单独的 `CSS` 文件。

安装：

```shell
$ pnpm add mini-css-extract-plugin -D
```

使用：

```js
// webpacl/webpack.prod.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

module.exports = {
    module: {
        rules: [{
            test: /\.css$/,
            use: [
              - 'style-loader', // 移除
              +	MiniCssExtractPlugin.loader， // 新增
                'css-loader'
            ]
        }]
    },
    plugins: [
        new MiniCssExtractPlugin(), // 这一步不可忽略
    ]
}
```

需要注意的是：

- `mini-css-extract-plugin` 库同时提供 `Loader`、`Plugin` 组件，需要同时使用。

- `mini-css-extract-plugin`不能与`style-loader` 一起使用，否则报错。

- `mini-css-extract-plugin`需要与 `html-webpack-plugin` 同时使用，才能将产物路径以 `link` 标签方式插入到`html` 中。

### less-loader

将 `.less` 文件转换为 `.css`文件。

安装：

```shell
$ pnpm add less less-loader -D
```

使用：

```js
// webpack/webpack.dev.config.js
module.exports = {
    module: {
        rules: [
          {
            test: /\.less$/,
            use: ['style-loader', 'css-loader', 'less-loader']
        	}
        ]
    }
}
```

需要注意的是 `less-loader`只会将`.less`转化为`.css`，至于页面上显示则需要和处理`css`文件一致。在**开发环境**下使用`style-loader`和`css-loader`，在**生产环境**下使用`mini-css-extract-plugin`和`css-loader`。

### sass-loader

将 `.sass` 或`.scss`文件转换为 `.css`文件。

安装：

```shell
$ pnpm add sass sass-loader -D
```

使用：

```js
// webpack/webpack.dev.config.js
module.exports = {
    module: {
        rules: [
          {
            test: /\.s[ac]ss$/,
            use: ['style-loader', 'css-loader', 'sass-loader']
        	}
        ]
    }
}
```

需要注意点与`less-loader`一致，这里不再赘述。

### postcss-loader

与上面介绍的 `Less/Sass/Scss` 这一类预处理器类似，但并不是预处理器，但可以将其视为一个“后处理器”或“转换器”。`PostCSS` 也能在原生 `CSS` 基础上增加更多表达力、可维护性、可读性更强的语言特性。两者主要区别在于预处理器通常定义了一套` CSS` 之上的超集语言；`PostCSS` 并没有定义一门新的语言，而是与 `@babel/core` 类似，只是实现了一套将 `CSS` 源码解析为 `AST` 结构，并传入 `PostCSS` 插件做处理的流程框架，具体功能都由插件实现。

安装：

```shell
$ pnpm add postcss postcss-loader -D
```

使用：

```js
// webpack/webpack.dev.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          "style-loader", 
          "css-loader", 
          "postcss-loader"
        ],
      },
    ],
  }
}
```

不过，这个时候的 `PostCSS` 还只是个空壳，下一步还需要使用适当的 `PostCSS` 插件进行具体的功能处理，例如我们可以使用 **autoprefixer**(自动补充浏览器前缀)。

安装：

```shell
$ pnpm add autoprefixer -D
```

使用：

```js
// webpack/webpack.dev.config.js
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          "style-loader", 
          'css-loader', 
          {
            loader: "postcss-loader",
            options: {
              postcssOptions: {
                // 添加 autoprefixer 插件
                plugins: [require("autoprefixer")],
              },
            },
          }
        ],
      },
    ],
  }
}
```

需要注意的是，如果项目中使用到了`less`或`sass`，需要再专门针对`less`或`sass`单独设置，并不能和`css`共享。

比如在项目中同时使用了`css`和`less`，那么写法如下：

```js
// webpack/webpack.dev.config.js
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          "style-loader", 
          'css-loader', 
          {
            loader: "postcss-loader",
            options: {
              postcssOptions: {
                // 添加 autoprefixer 插件
                plugins: [require("autoprefixer")],
              },
            },
          }
        ],
      },
      {
        test: /\.less$/,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                // 添加 autoprefixer 插件
                plugins: [require('autoprefixer')],
              },
            },
          },
          'less-loader',
        ],
      },
    ],
  }
}
```

`PostCSS` 最大的优势在于其简单、易用、丰富的插件生态，基本上已经能够覆盖样式开发的方方面面。实践中，经常使用的插件有：

- `autoprefixer`：自动添加浏览器前缀。
- `postcss-preset-env`：将最新 `CSS` 语言特性转译为兼容性更佳的低版本代码的插件。
- `postcss-less`：兼容 `Less` 语法的 `PostCSS` 插件。
- `postcss-sass`：兼容 `sass` 语法的 `PostCSS` 插件。
- `stylelint`：一个现代 `CSS` 代码风格检查器，能够帮助识别样式代码中的异常或风格问题。
