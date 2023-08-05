## 	webpack四大核心

```js
// 入口 entry

// 出口 output

// 装载器 loader

// 插件 plugin

// 我们以豆浆机举例，入口即是放置未处理的黄豆、大豆等一些原材料，装载器就是刀片，对未处理的成品进行打磨转换，插件就是帮助我们实现一些我们想要的功能，比如给我们自动加热和加糖等，出口就是将打磨好的豆浆进行倒出的出口
```

## 开发时依赖和生产时依赖的关系

```js
// 在我们常见的安装包的时候，有多种模式，一种是安装开发依赖，一种是安装生产依赖，还有的是安装全局依赖

// 全局依赖
全局依赖一般指的是给本地计算机全局安装的一些`工具`， 比如webpack yarn cnpm等

// 开发依赖   -D
是指开发的时候使用的到，但是当我们项目发布到服务器时不再需要的依赖，比如less，sass、html-webpack-plugin

// 生产依赖  -S
是指无论是在本地环境还是在线上环境都会使用到的依赖 最熟悉的就是axios，因为发送网络请求，无论是在本地还是在线上，都会使用的到


```



## webpack安装和使用

```js
// 全局安装
npm install webpack webpack-cli -g


// 局部安装
// 使用与全局webpack版本与当前项目webpack版本不一致时，用项目里面的webpack进行打包，以达到正确的使用
// 1. 配置json文件
npm init -y
// 2. 在生产环境环境安装webpack和webpack-cli
npm install webpack webpack-cli -D (生产环境)
// 3. 使用
npx webpack (npx会优先查找当前项目里面的webpack，而不是使用全局的webpack版本)

// 指定入口打包 (默认为src下面的index.js)
npx webpack --entry ./src/main.js

// 安装可以解析 css 文件的loader
npm install style-loader -D
npm install css-loader -D
// 然后再webpack.config.js中配置

// 解析less
npm install less -D
npm install less-loader -D

```



## webpack配置文件

```js
const path = require("path")
// html模板
const htmlwebpckplugin = require("html-webpack-plugin")
// 单独css文件
const MiniCssEctractPlugin = require("mini-css-extract-plugin")
// css压缩
const CssMini = require("css-minimizer-webpack-plugin")
// js压缩
const JSMini = require("terser-webpack-plugin")
const webpack = require("webpack")

module.exports = {
  optimization: {
    // 压缩
    minimizer: [new CssMini(), new JSMini()],
  },
  // 入口
  entry: "./src/index.js",
  // 模式  development:开发模式  production 生产模式(默认)
  mode: "production",
  // 出口
  output: {
    path: path.resolve(__dirname, "build"),
    filename: "js/index.js",
  },
  // 配置webpakc-dev-server
  devServer: {
    // 自动打开浏览器
    open: true,
    // web服务器端口
    port: 5000,
    // 开启热更新
    hot: true,
    // 配置跨域
    proxy: {
      "/admin": {
        target: "http://huangjianxin.cn",
      },
    },
  },
  // 配置loader
  module: {
    rules: [
      {
        test: /\.html$/,
        use: ["html-withimg-loader"],
      },
      {
        test: /\.css$/,
        // use: ["style-loader", "css-loader"],
        // 使用单独css文件
        use: [
          {
            loader: MiniCssEctractPlugin.loader,
            options: {
              esModule: false,
              // 给css所有图片添加相对路径
              publicPath: "../images/",
            },
          },
          "css-loader",
        ],
      },
      {
        test: /\.less$/,
        use: [MiniCssEctractPlugin.loader, "less-loader"],
      },
      {
        test: /\.(jpg|jpeg|png)/,
        use: {
          loader: "file-loader",
          options: {
            // 图片加载到目录下
            outputPath: "images",
          },
        },
      },
    ],
  },
  // 插件
  plugins: [
    // html模板
    new htmlwebpckplugin({
      template: "./src/index.html",
    }),
    // 单独css文件
    new MiniCssEctractPlugin({
      filename: "css/index.css",
    }),
    // 使用wbepack 全局注入
    new webpack.ProvidePlugin({
      $: "jquery",
    }),
  ],
  // 忽略打包 比如第三方包
  externals: {
    jquery: "$",
  },
}

```

## 分类存储

```js
// 默认打包完成后，所有的文件 css文件 js文件 以及html文件都在同一层目录下，想要css文件在css文件夹下，js文件在js文件夹下，需要进行分类存储

// 只需要在出口的时候进行设置
js/index.js   css/index.css

output: {
  path: path.resolve(__dirname, './build'),
  filename: 'js/index.js'
}
```

## 避免缓存

```js
// 浏览器对于文件名相同的文件会进行缓存策略，下次加载的时候就会优先使用缓存好的文件，因此有些时候需要避免缓存

output: {
  path: path.resolve(__dirname, './build'),
    // 动态加载名字，并且添加哈希值避免缓存 哈希的位数
  filename: 'js/[name].[hash:8].js'
}
```

## 注入全局变量

```js
// 比如在js文件中使用jquery的时候，我么需要在每一个js文件中，对jquery进行引用，` import $ from 'jquery' `

// 但是利用webpack可以进行全局导入，这样就不要在每一个文件中进行导入

// webpack.config.js
cosnt webpack = require('webpack')

module.exports = {
  plugins: [
    // 给每一个组件注入 $ 变量
    new webpack.ProvidePlugin({
      $: "jquery"
    })
  ]
}
```

## 忽略打包

```js
// 比如我们在项目中引入了第三方的包，但是在本项目打包的时候，不想一起将第三方的包进行打包，可以使用 `externals` 进行忽略打包

module.exports = {
  // 忽略打包
  externals: {
    jquery: "$"
  }
}

// 最佳实现方案是利用 CDN（内容发布网络） 代替 第三方包下载
```

## 图片加载处理

```js
// 图片加载需要file-loader / url-loader

// file-loader 默认使用 ES6模块化
import imgSrc from './图片路径'
1. 使用js导入图片

// 安装
npm install file-loader -D

// 使用
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(jpg|jpeg|png)/,
        use: {
          loader: "file-loader".
          options: {}
        }
      }
    ]
  }
}



2. 使用css|less方法导入图片
	css文件在根目录，没有任何问题
  css文件在css目录中，需要打包时将图片加上../前缀
	图片加载到指定路径

// webpack.config.js
module.exports = {
  rules: [
    {
      test: /\.(jpg|png|jpeg)/,
      // 2.1 css文件在根目录
      use: [
        MiniCssEctractPlugin.loader,
        "css-loader	"
      ]
      // 2.2 css文件在css目录下
      use: [
        {
          loader: MiniCssEctractPlugin.loader,
          options: {
            // 给css所有图片添加相对路径
            publicPath: "../",
          },
        },
    		"css-loader"
      ]
    }
  ]
}

3. 在html上使用img加载图片
```

## 配置跨域

```js
目标网络地址： http://huangjianxin.cn/admin/get/hotArticle
当前本地地址： http://localhost:5000

// index.js
new Promise((resolve, reject) => {
  $.ajax({
    url: "/admin/get/hotArticle",
    type: "GET",
    success: resolve,
    error: reject,
  })
})
  .then((res) => {
    console.log(res)
  })
  .catch((err) => {
    console.log(err)
  })

// webpack.config.js
module.exports = {
  devServer: {
    proxy: {
      // 匹配所有以admin开头的
      "/admin": {
        target: 'http://huangjianxin.cn'
      }
    }
  }
}
```



## plugin插件

### `html-webpack-plugin`

```js
// 当我们采用默认打包时，会发现它只会自动打包src/index.js 不会对html进行打包，我们还需要手动书写html并且进行引入，这个时候我们可以借助 `html-webpack-plugin` 插件进行完成

// 安装
npm i html-webpack-plugin -D

// 使用  webpack.config.js
// 导入
const htmlwebpackplugin = require('html-webpack-plugin')

module.exports = {
  // 使用插件
  plugins: [
    new htmlwebpackplugin({
      // 指定输出的模板
      template: "./src/index.html",
      // 别名
      filename: 'main.html'
    })
  ]
}

```

### `webpack-dev-server`

```js
// 用来在开发过程中，起服务，这样无需我们手动打开index.html文件，

// 前提
需要先安装 webpack 和 webpack-cli
npm install webpack webpack-cli -D

// 安装 
npm install webpack-dev-server -D

// 配置指令 package.json
scripts: {
  "dev": "webpack serve"
}

// 配置dev配置 webpack.config.js
module.exports = {
  // 配置 webpack-config.js
  devServer: {
    // 是否自动打开
    open: true,
    // web服务器端口
    port: 5000
  }
}


```

### `mini-css-extract-plugin`

```js
// 用来生成css文件

作用：我们使用style-laoder时，会将css直接放置在html中的style内，但是我们有时并不需要将css内容放置在style中，而是将其放置在单独的css文件

// 安装
npm install mini-css-extrct-plugin -D

// 使用	
const MiniCssExtrctPlugin = require("mini-css-extrct-plugin")

module.exports = {
  module: {
    // 使用MiniCssExtrctPlugin代替style-loader
    rules: [
      {
        test: /\.css$/,
        use: [
          MiniCssExtructPlugin.loader,
          "css-loader"
        ]
      }
    ]
  },
  // 使用插件
  plugins: [
    new MiniCssExtructPlugin({
      // 打包之后的css文件名称
      filename: 'index.css'
    })
  ]
}

```

### `css-minimizer-webpack-plugin`

```js
// 将生产的css文件进行压缩，仅在生产环境中生效

// 安装
npm install css-minimizer-webpack-plugin -D

// 使用
const cssMini = require("css-minimizer-webpack-plugin")

module.exports = {
  // 压缩
  optimization: {
    minimizer: [
      // 使用css压缩
      new CssMini()
    ]
  }
}
```

### `terser-webpack-plugin`

```js
// 将生产的js文件进行压缩，仅在生产环境中生效

// 安装
npm install terser-webpack-plugin -D

// 使用
const JsMini = require("terser-webpack-plugin")

module.exports = {
  // 压缩
  optimization: {
    minimizer: [
      // 使用css压缩
      new CssMini(),
      // 使用js压缩
      new JsMini()
    ]
  }
}
```



## loader装载器

#### `css-loader` 和 `style-loader`

```js
// 用来加载css样式并且添加到页面上

// 安装
npm install style-loader css-loader -D

// 在js文件中引入css文件
// index.js
import ('./index.css')

// 在webpack.config.js中配置
modulex.exports = {
  module: {
    // 匹配规则 可以是多个 因此是数组
    rules: [
      {
        // 匹配规则，以.css为后缀的
        test: /\.css$/,
        // 使用的loader 执行的顺序为：从下到上执行
        use: [
          "style-loader",
          "css-loader"
        ]
      }
    ]
  }
}
```

### `less-loader`

```js
// 当我们使用less、sacc或者scss时候，也需要对应的loader进行转化

// 安装
npm install less less-loader -D

// 使用 webpack.config.js
module.exports = {
  // loader配置
  module: {
    // 匹配规则
    rules: [
      {
        test: /\.less$/,
        use: [
          'less-loader'
        ]
      }
    ]
  }
}

```

### `file-loader`

```js
// 用来加载图片资源 

// 安装
npm install file-loader -D

// 使用
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(jpg|jpeg|png)/,
        use: {
          loader: "file-loader".
          options: {}
        }
      }
    ]
  }
}


```

### `url-loader`

```js
// 推荐使用url-loader代替file-loader

// 安装
npm install url-laoder -D

// 使用
// webpack.config.js
module.exports = {
  rules: [
    {
      test: /\.(jpg|png|jpeg)/,
      use: {
        loader: 'url-loader',
        options: {
          // 不采用Esmodule
          esModule: false,
          // 输出位置
          outputPath: 'images',
          // 最大限制，低于该限制，转化为Base64格式 100k
          limit: 100 * 1024
        }
      }
    }
  ]
}
```



### `html-withimg-loader`

```js
// 用来加载html中的图片

// 安装
npm install html-withimg-loader -D

// 使用
// webpack.config.js
module.exprots = {
  rules: [
    {
      test: /\.html$/,
      use: [
        "html-withimg-loader"
      ]
    }
  ]
}
```

## 实战

### 构建Vue开发环境
