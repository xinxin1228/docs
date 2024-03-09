我们先看一下项目的整体目录：

```shell
├── package.json
├── pnpm-lock.yaml
├── postcss.config.js
├── public
│   └── index.html
├── src
│   ├── App.vue
│   ├── components
│   │   ├── CreateImage.vue
│   │   ├── CssCom.vue
│   │   ├── Layout.vue
│   │   ├── LessCom.vue
│   │   ├── SassCom.vue
│   │   ├── ScssCom.vue
│   │   └── index.ts
│   ├── img
│   │   ├── 01.jpg
│   │   └── 02.jpg
│   ├── index.ts
│   ├── pages
│   │   ├── About.vue
│   │   ├── Article.vue
│   │   ├── ArticleDetail.vue
│   │   └── Home.vue
│   ├── router
│   │   └── index.ts
│   ├── store
│   │   ├── counter.ts
│   │   ├── index.ts
│   │   └── userInfo.ts
│   └── types.d.ts
├── tsconfig.json
└── webpack
    ├── webpack.comm.config.js
    ├── webpack.dev.config.js
    └── webpack.prod.config.js
```

## 搭建 Webpack 基础

在上一章 [【搭建基础的前端工程化项目】](/docs/前端构建工具/Webpack5实战/搭建基础的前端工程化项目.md) 中，我们已经搭建了基本的开发环境，这里就不再重复。

接下里我们直接基于这个项目基础进行开发。

## 解析 Vue 文件

安装依赖：

```shell
$ pnpm add vue vue-loader
```

在 `webpack` 配置文件中进行配置：

```js
// webpack/webpack.config.comm.js
const { VueLoaderPlugin } = require('vue-loader')

module.exports = {
  // ...
  module: {
    rules: [
      // ...
      {
        test: /\.vue$/,
        use: ['vue-loader']
      }
    ]
  },
  plugins: [
    // ...
    new VueLoaderPlugin()
  ]
}
```

在入口文件进行编写：

```js
// src/index.js
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)
app.mount('#app') // 需要在 public/index.html 编写这个元素
```

## 引入 Vue-Router

安装依赖：

```shell
$ pnpm add vue-router
```

在 `src/pages` 中创建几个 `vue` 文件，用来当作路由页面。

<!-- tabs:start -->

#### **Home.vue**

```vue
<template>
  <h1>这是 Home 页面</h1>
</template>

<script setup></script>

<style scoped lang="less"></style>

```

#### **About.vue**

```vue
<template>
  <h1>这是 About 页面</h1>
</template>

<script setup></script>

<style scoped lang="less"></style>

```



#### **Article.vue**

```vue
<template>
  <h1>这是 Article 页面</h1>
</template>

<script setup></script>

<style scoped lang="less"></style>

```



<!-- tabs:end -->

在 `src/router` 目录下，新建 `index.js` 文件，用来存放路由相关的信息，之后导出在 `src/index.js` 中引入。在 `components/pages` 目录下新建 `Layout.vue` 文件，用作页面的布局文件，之后在 `App.vue` 中引入。

<!-- tabs:start -->

#### **router/index.js**

```js
import { createRouter, createWebHistory } from 'vue-router'
import Home from '@/pages/Home.vue'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('@/pages/About.vue')
  },
  {
    path: '/article',
    name: 'Article',
    component: () => import('@/pages/Article.vue')
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router

```



#### **src/index.js**

```js
import { createApp } from 'vue'
import router from './router'
import App from './App.vue'

const app = createApp(App)
app.use(router).mount('#app')

```



#### **Layout.vue**

```vue
<template>
  <div class="layout">
    <nav>
      <router-link to="/">Home</router-link>
      <router-link to="/about">About</router-link>
      <router-link to="/article">Article</router-link>
    </nav>

    <router-view></router-view>
  </div>
</template>

<script lang="ts" setup></script>

<style scoped lang="less">
.layout {
  nav {
    display: flex;
    a {
      margin: 0 10px;
      font-size: 16px;
      text-decoration: none;
      color: purple;
      font-weight: bold;
      &.router-link-active {
        color: green;
      }
    }
  }
}
</style>

```



#### **App.vue**

```vue
<template>
  <div class="vue">
    <h1>Hello Vue</h1>
    <div class="count">
      <button @click="count++">+1</button>
      <h2>{{ count }}</h2>
      <button @click="count--">-1</button>
    </div>
    <CssCom type="css" />
    <LessCom type="less" />
    <SassCom type="sass" />
    <ScssCom type="scss" />
    <Layout />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { CssCom, LessCom, SassCom, ScssCom, Layout } from '@/components'

const count = ref(10)
</script>

<style scoped lang="less">
.vue {
  .count {
    height: 30px;
    display: flex;
    align-items: center;
    h2 {
      margin: 0 10px;
    }
  }
}
</style>

```



<!-- tabs:end -->

!> 这里需要注意的是：由于我们使用的是H5的 `history` 模式，因此在路由页直接刷新页面的话，浏览器会空白，这个时候需要我们去配置一下 `devServer` 。

配置 `webpack.dev.config.js` 文件：

```js
// webpack/webpack.dev.config.js
module.exports = {
  // ...
  devServer: {
    port: 8080,
    hot: true,
    historyApiFallback: true, // 添加这行代码
  }
}
```

## 引入 pinia

安装

```shell
$ pnpm add pinia
```

`src/store`的目录结构如下：

```shell
store
├── index.js
├── counter.js
└── userInfo.js
```

 `index.js` 文件，用于统一导出所有的 `store` 对象，`userInfo.js`  和 `counter.js` 是具体的 `store` 用例。

<!-- tabs:start -->

#### **counter.js**

```js
import { ref } from 'vue'
import { defineStore } from 'pinia'

const useCounterStore = defineStore('counter', () => {
  const counter = ref(0)

  const increment = () => {
    counter.value++
  }

  const decrement = () => {
    counter.value--
  }

  return {
    counter,
    increment,
    decrement
  }
})

export default useCounterStore

```



#### **userInfo.js**

```js
import { ref } from 'vue'
import { defineStore } from 'pinia'

const useUserInfoStore = defineStore('userInfo', () => {
  const userInfo = ref({
    name: 'web',
    age: 24
  })

  const updateInfo = ({ name, age }) => {
    userInfo.value = {
      name: name || userInfo.value.name,
      age: age || userInfo.value.age
    }
  }

  return { userInfo, updateInfo }
})

export default useUserInfoStore

```



#### **index.js**

```js
export { default as useUserInfoStore } from './userInfo'
export { default as useCounterStore } from './counter'
```





<!-- tabs:end -->

然后在 `src/index.js` 入口文件中进行声明：

```js
// src/index.js
import { createApp } from 'vue'
import router from './router'
import { createPinia } from 'pinia'
import App from './App.vue'

const app = createApp(App)
app.use(createPinia()).use(router).mount('#app')
```

在各个组件中进行使用：

<!-- tabs:start -->

#### **CssCom.vue**

```vue
<template>
  <button @click="increment">count++</button>
  <button @click="decrement">count--</button>
</template>

<script setup>
import { useCounterStore } from '@/store'
  
const counterStore = useCounterStore()
const { increment, decrement } = counterStore
</script>
```



#### **LessCom**

```vue
<template>
  <h3>name: {{ userInfo.name }} age: {{ userInfo.age }}</h3>
  <button @click="changeInfo">age++</button>
</template>

<script setup>
import { storeToRefs } from 'pinia'
import { useUserInfoStore } from '@/store'

const userInfoStore = useUserInfoStore()
const { userInfo } = storeToRefs(userInfoStore)

const changeInfo = () => {
  userInfo.value.age++
}
</script>
```



#### **SassCom**

```vue
<template>
  <h2>counter: {{ counter }}</h2>
</template>

<script setup>
import { storeToRefs } from 'pinia'
import { useCounterStore } from '@/store'
  
const counterStore = useCounterStore()
const { counter } = storeToRefs(counterStore)
</script>
```



#### **ScssCom**

```vue
<template>
  <h3>name: {{ userInfo.name }} age: {{ userInfo.age }}</h3>
  <button @click="updateInfo({ name: 'vue' })">更改姓名</button>
</template>

<script setup>
import { storeToRefs } from 'pinia'
import { useUserInfoStore } from '@/store'

const userInfoStore = useUserInfoStore()
const { userInfo } = storeToRefs(userInfoStore)
const { updateInfo } = userInfoStore
</script>
```

<!-- tabs:end -->

> 当前案例的完整代码地址：https://github.com/xinxin1228/webpack-actual/tree/main/03_%E6%90%AD%E5%BB%BAVue3%E9%A1%B9%E7%9B%AE%E7%9A%84%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83_js%E7%89%88

## JS版 webpack 配置项总结

<!-- tabs:start -->

#### **webpack.comm.config.js**

```js
const path = require('node:path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { VueLoaderPlugin } = require('vue-loader')

/**
 * @type {import('webpack').Configuration}
 */
module.exports = {
  entry: './src/index.js',
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader',
        exclude: /node_modules/,
        options: {
          presets: ['@babel/preset-env']
        }
      },
      {
        test: /\.(jpe?g|png|gif|webp|svg)$/i,
        type: 'asset',
        generator: {
          filename: 'images/[hash][ext]'
        },
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024 // 10KB 小于10k的直接转base64
          }
        }
      },
      {
        test: /\.vue$/,
        use: ['vue-loader']
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, '../public/index.html')
    }),
    new VueLoaderPlugin()
  ],
  resolve: {
    extensions: ['.js'],
    alias: {
      '@': path.resolve(__dirname, '../src')
    }
  },
  cache: {
    type: 'filesystem'
  }
}

```

#### **webpack.dev.config.js**

```js
const { merge } = require('webpack-merge')
const webpackCommConfig = require('./webpack.comm.config')

module.exports = merge(webpackCommConfig, {
  mode: 'development',
  devServer: {
    historyApiFallback: true,
    port: 8080,
    hot: true
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: ['autoprefixer']
              }
            }
          }
        ]
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
                plugins: ['autoprefixer']
              }
            }
          },
          'less-loader'
        ]
      },
      {
        test: /\.s[ac]ss$/,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: ['autoprefixer']
              }
            }
          },
          'sass-loader'
        ]
      }
    ]
  },
  devtool: 'eval-cheap-module-source-map'
})

```

#### **webpack.prod.config.js**

```js
const path = require('node:path')
const { merge } = require('webpack-merge')
const webpackCommConfig = require('./webpack.comm.config')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin')

module.exports = merge(webpackCommConfig, {
  mode: 'production',
  output: {
    path: path.resolve(__dirname, '../dist'),
    filename: 'js/[name][contenthash:8].js',
    chunkFilename: 'js/chunk-[name][contenthash:8].js',
    clean: true
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: ['autoprefixer']
              }
            }
          }
        ]
      },
      {
        test: /\.less$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: ['autoprefixer']
              }
            }
          },
          'less-loader'
        ]
      },
      {
        test: /\.s[ac]ss$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: ['autoprefixer']
              }
            }
          },
          'sass-loader'
        ]
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: 'css/[name][contenthash:8].css'
    })
  ],
  optimization: {
    minimize: true,
    minimizer: [`...`, new CssMinimizerPlugin()],
    splitChunks: {
      cacheGroups: {
        chunks: 'all',
        // 抽离node_modules中的公共模块
        vendors: {
          test: /[\/]node_modules[\/]/,
          filename: 'vendors.[contenthash].js',
          priority: -10, // 设置优先级，确保先处理这个规则
          enforce: true,
          minChunks: 1
        },
        common: {
          chunks: 'all',
          priority: -20,
          filename: 'commons.[contenthash].js',
          minChunks: 2, //被至少2个chunk引/用时才会被提取
          reuseExistingChunk: true //如果模块已经被打包进某个chunk，则复用该chunk
        }
      }
    }
  },
  devtool: 'nosources-source-map'
})

```

<!-- tabs:end -->

> 当前案例完整的完整代码地址：https://github.com/xinxin1228/webpack-actual/tree/main/03_%E6%90%AD%E5%BB%BAVue3%E9%A1%B9%E7%9B%AE%E7%9A%84%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83_js%E7%89%88

## 添加 TypeScript

安装依赖：

```shell
$ pnpm add typescript ts-loader -D
```

生成 `tsconfig.json` 文件，直接在命令行中输入：

```shell
$ tsc --init
```

然后根据自己的需求进行保留和更改配置项，这里是我用的配置项内容：

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "esnext",
    // "strict": true,
    "jsx": "preserve",
    "importHelpers": true,
    "moduleResolution": "node",
    "experimentalDecorators": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "useDefineForClassFields": true,
    "sourceMap": true,
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"]
    },
    "lib": ["esnext", "dom", "dom.iterable", "scripthost"]
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.vue"],
  "exclude": ["node_modules"]
}
```

配置 `webpack`。

```js
// webpack/webpack.comm.config.js
module.exports = {
  // ...
  module: {
    rules: [
      // ...
      {
        test: /\.ts$/,
        loader: 'ts-loader',
        options: {
          appendTsSuffixTo: [/\.vue$/] // 识别vue文件中的ts
        },
        exclude: /node_modules/
      }
    ]
  }
}
```

然后在 `src` 目录下创建一个 `types.d.ts` 类型声明文件。

```js
// src/types.d.ts
declare module '*.vue' {
  /* eslint-disable */
  import type { DefineComponent } from 'vue'
  const component: DefineComponent<{}, {}, any>
  export default component
}

// 支持各种图片
declare module '*.png'
declare module '*.jpg'
declare module '*.jpeg'
declare module '*.gif'
declare module '*.svg'

```

## TS版 webpack 配置项总结

<!-- tabs:start --->

#### **webpack.comm.config.js**

```js
const path = require('node:path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { VueLoaderPlugin } = require('vue-loader')

/**
 * @type {import('webpack').Configuration}
 */
module.exports = {
  entry: './src/index.ts',
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader',
        exclude: /node_modules/,
        include: path.resolve(__dirname, '../src'),
        options: {
          presets: ['@babel/preset-env']
        }
      },
      {
        test: /\.(jpe?g|png|gif|webp|svg)$/i,
        type: 'asset',
        generator: {
          filename: 'images/[hash][ext]'
        },
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024 // 10KB 小于10k的直接转base64
          }
        }
      },
      {
        test: /\.vue$/,
        use: ['vue-loader']
      },
      {
        test: /\.ts$/,
        loader: 'ts-loader',
        options: {
          appendTsSuffixTo: [/\.vue$/] // 识别vue文件中的ts
        },
        exclude: /node_modules/
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, '../public/index.html')
    }),
    new VueLoaderPlugin()
  ],
  resolve: {
    extensions: ['.ts', '.js'],
    alias: {
      '@': path.resolve(__dirname, '../src')
    }
  },
  cache: {
    type: 'filesystem'
  }
}

```



#### **webpack.dev.config.js**

```js
const { merge } = require('webpack-merge')
const webpackCommConfig = require('./webpack.comm.config')

/**
 * @type {import('webpack').Configuration}
 */
module.exports = merge(webpackCommConfig, {
  mode: 'development',
  devServer: {
    port: 8080,
    hot: true
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.less$/,
        use: ['style-loader', 'css-loader', 'less-loader']
      },
      {
        test: /\.s[ac]ss$/,
        use: ['style-loader', 'css-loader', 'sass-loader']
      }
    ]
  },
  devtool: 'eval-cheap-module-source-map'
})

```



#### **webpack.prod.config.js**

```js
const path = require('node:path')
const { merge } = require('webpack-merge')
const webpackCommConfig = require('./webpack.comm.config')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin')

/**
 * @type {import('webpack').Configuration}
 */
module.exports = merge(webpackCommConfig, {
  mode: 'production',
  output: {
    path: path.resolve(__dirname, '../dist'),
    filename: 'js/[name].[contenthash:8].js',
    chunkFilename: 'js/chunk-[name].[contenthash:8].js',
    clean: true
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader']
      },
      {
        test: /\.less$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader',
          'postcss-loader',
          'less-loader'
        ]
      },
      {
        test: /\.s[ac]ss$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader',
          'postcss-loader',
          'sass-loader'
        ]
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({ filename: 'css/[name].[contenthash:8].css' })
  ],
  optimization: {
    minimize: true,
    minimizer: ['...', new CssMinimizerPlugin()],
    splitChunks: {
      cacheGroups: {
        chunks: 'all',
        // 抽离node_modules中的公共模块
        vendors: {
          test: /[\/]node_modules[\/]/,
          filename: 'vendors.[contenthash].js',
          priority: -10, // 设置优先级，确保先处理这个规则
          enforce: true,
          minChunks: 1
        },
        common: {
          chunks: 'all',
          priority: -20,
          filename: 'commons.[contenthash].js',
          minChunks: 2, //被至少2个chunk引/用时才会被提取
          reuseExistingChunk: true //如果模块已经被打包进某个chunk，则复用该chunk
        }
      }
    }
  },
  devtool: 'nosources-source-map'
})

```



<!-- tabs:end -->

> 当前案例完整的完整代码地址：https://github.com/xinxin1228/webpack-actual/tree/main/04_%E6%90%AD%E5%BB%BAVue3%E9%A1%B9%E7%9B%AE%E7%9A%84%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83_ts%E7%89%88
