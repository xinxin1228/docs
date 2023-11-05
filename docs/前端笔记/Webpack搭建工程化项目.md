## 安装依赖、初始化配置文件

### 初始化项目

```shell
# 这里采用 pnpm
pnpm init
```

### 安装`webpack`依赖

```shell
# 这里仅安装webpack必备插件的包，其他的包根据需求自行安装
pnpm add webpack webpack-dev-server webpack-cli webpack-merge html-webpack-plugin -D
```

### 安装`eslint`、`prettier`

```shell
pnpm add eslint prettier -D

# 由于eslint和prettier会有冲突，因此需要安装依赖并且配置eslint来解决冲突
pnpm add eslint-config-prettier eslint-plugin-prettier -D
```

### 在根目录创建`.editorconfig`文件

```js
# http://editorconfig.org

root = true

[*] # 表示所有文件适用
charset = utf-8 # 设置文件字符集为 utf-8
indent_style = space # 缩进风格（tab | space）
indent_size = 2 # 缩进大小
end_of_line = lf # 控制换行类型(lf | cr | crlf)
trim_trailing_whitespace = true # 去除行首的任意空白字符
insert_final_newline = true # 始终在文件末尾插入一个新行

[*.md] # 表示仅 md 文件适用以下规则
max_line_length = off
trim_trailing_whitespace = false
```

在根目录创建`.prettierrc`文件

```js
{
  "useTabs": false,
  "tabWidth": 2,
  "printWidth": 80,
  "singleQuote": true,
  "trailingComma": "none",
  "semi": false
}
```

### 配置`eslint`

```shell
npx eslint --init

# 然后根据你的需求选择, 之后配置eslint
```

`eslint`配置文件

```json
{
  "env": {
    "browser": true,
    "es2021": true
  },
  "extends": ["eslint:recommended", "prettier"],
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "plugins": ["prettier"],
  "rules": {
    "prettier/prettier": "error",
    "no-undef": "off"
    // 添加需要处理的规则，"error" 代表错误， "warn" 代表警告 “off” 代表关闭该错误
  }
}

```

### 编写`webpack`配置文件

```js
// 在根目录创建 config 文件夹，用来存放webpack相关的配置
mkdir config

// 在 config 文件中创建文件 结构如下：
config
├── webpack.comm.config.js  // 公共文件
├── webpack.dev.config.js   // 开发专用文件
└── webpack.prod.config.js  // 构建专用文件
```

#### `webpack.comm.config.js` 

```js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

/**
 * @type {import('webpack').Configuration}
 */
module.exports = {
  entry: path.resolve(__dirname, '../src/index.js'),
  module: {
    rules: []
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, '../public/index.html')
    })
  ]
}

```

#### `webpack.dev.config.js`

```js
const webpack = require('webpack')
const { merge } = require('webpack-merge')
const webpackCommConfig = require('./webpack.comm.config')

/**
 * @type {import('webpack').Configuration}
 */
module.exports = merge(webpackCommConfig, {
  mode: 'development',
  devServer: {
    port: 5200,
    hot: true,
    open: true
  },
  plugins: [new webpack.HotModuleReplacementPlugin()],
  devtool: 'cheap-module-eval-source-map'
})

```

#### `webpack.prod.config.js`

```js
const path = require('path')
const { merge } = require('webpack-merge')
const webpackCommConfig = require('./webpack.comm.config')

/**
 * @type {import('webpack').Configuration}
 */
module.exports = merge(webpackCommConfig, {
  mode: 'production',
  output: {
    path: path.resolve(__dirname, '../dist'),
    clean: true
  }
  
  // 额外补充，当项目为一个库时，比如axios，
  filename: 'axios.min.js', // 输出的文件名
  sourceMapFilename: 'axios.min.map', // map文件名
  libraryTarget: 'umd', // 定义打包方式Universal Module Definition,同时支持在CommonJS、AMD和全局变量使用
  library: 'axios', // 类库的命名空间，如果通过网页的方式引入，则可以通过window.axios访问它
  libraryExport: 'default',// 对外暴露default属性，就可以直接调用default里的属性
  globalObject: 'this', // 定义全局变量,兼容node和浏览器运行，避免出现"window is not defined"的情况
})
```

### 初始化`git` 

这步不可忽略，后面要配置`git`提交规范

```js
git init
```

编写`.gitignore`

```js
node_modules
dist
```

### 配置`git`提交规范

1. 使用 `husky-init` 命令快速在项目初始化一个 husky 配置。在配置 husky 之前必须初始化 git，否则可能会配置不成功

```shell
# 切记先初始化git
npx husky-init && pnpm install
```

2. 安装`cz-conventional-changelog`，并且初始化`cz-conventional-changelog`(提交规范)

```shell
pnpm add commitizen cz-conventional-changelog @commitlint/cli @commitlint/config-conventional -D
```

3. 在package.json 中添加

```js
"config": {
  "commitizen": {
    "path": "./node_modules/cz-conventional-changelog"
  }
}
```

4. 在根目录创建`commitlint.config.js`文件

```js
module.exports = {
  extends: ['@commitlint/config-conventional']
}
```

4. 构建和初始化提交规范之后，`git add`之后，不再使用`git commit`提交，而是使用 `npx cz`或在`script`脚本中添加`commit: cz`

6. 强制提交风格，不然规则提交报错

```shell
npx husky add .husky/commit-msg '
echo "🚀 ~ 请使用提交规范来提交代码"

npx --no-install commitlint --edit "$1"'
```

## 编写`scripts`

```json
"scripts": {
  // 开发
  "dev": "webpack-dev-server --config ./config/webpack.dev.config.js",
  // 构建
  "build": "webpack --config ./config/webpack.prod.config.js",
  // 格式化
  "prettier": "prettier --write ./src",
  // 校验eslint，并修复
  "lint": "eslint ./src/**.js --fix",
  // 初始化husky 
  "prepare": "husky install",
  // 可以使用 pnpm commit 来提交代码
  "commit": "cz"
}
```



