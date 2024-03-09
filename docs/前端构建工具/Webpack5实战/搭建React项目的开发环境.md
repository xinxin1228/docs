我们先看一下项目的整体目录：

```shell
├── package.json
├── pnpm-lock.yaml
├── postcss.config.js
├── public
│   └── index.html
├── src
│   ├── App.tsx
│   ├── components
│   │   ├── CreateImage
│   │   │   ├── index.tsx
│   │   │   └── style.ts
│   │   ├── CssCom
│   │   │   ├── index.css
│   │   │   └── index.tsx
│   │   ├── LessCom
│   │   │   ├── index.less
│   │   │   └── index.tsx
│   │   ├── SassCom
│   │   │   ├── index.sass
│   │   │   └── index.tsx
│   │   ├── ScssCom
│   │   │   ├── index.scss
│   │   │   └── index.tsx
│   │   └── index.ts
│   ├── img
│   │   ├── 01.jpg
│   │   └── 02.jpg
│   ├── index.less
│   ├── index.tsx
│   ├── pages
│   │   ├── About.tsx
│   │   ├── Article.tsx
│   │   └── Home.tsx
│   ├── router
│   │   └── index.tsx
│   ├── store
│   │   ├── features
│   │   │   ├── counter.ts
│   │   │   └── userInfo.ts
│   │   ├── hooks.ts
│   │   └── index.ts
│   └── types.d.ts
├── tsconfig.json
└── webpack
    ├── webpack.comm.config.js
    ├── webpack.dev.config.js
    └── webpack.prod.config.js
```

## 搭建 Webpack 基础

在之前的章节 [【搭建基础的前端工程化项目】](/docs/前端构建工具/Webpack5实战/搭建基础的前端工程化项目.md) 中，我们已经搭建了基本的开发环境，这里就不再重复。

接下里我们直接基于这个项目基础进行开发。

## 解析 React

安装：

```shell
$ pnpm add react react-dom
$ pnpm add @babel/preset-react -D
```

在 `webpack` 配置文件中进行配置，我们需要将原来的 `module.rules` 中 对 `js` 的解析改为 `jsx?`，然后配置对应的 babel 插件。

```js
// webpack/webpack.comm.config.js
module.exports = {
  // ...
  module: {
    rules: [
      // ...
      test: /\.jsx?$/,
      loader: 'babel-loader',
      exclude: /node_modules/,
      options: {
        presets: ['@babel/preset-env', '@babel/preset-react']
      }
    ]
  }
}
```

在入口文件编写：

```js
// src/index.js
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'

const root = ReactDOM.createRoot(document.getElementById('root'))

root.render(<App />)
```

## 引入 React-Router

安装：

```shell
$ pnpm add react-router-dom
```

在 `src/pages` 中创建几个 `jsx` 文件，用来当作路由页面。

<!-- tabs:start -->

#### **Home.jsx**

```jsx
import React, { memo } from 'react'

const Home = memo(() => {
  return <h1>Home</h1>
})

export default Home

```

#### **About.jsx**

```jsx
import React, { memo } from 'react'

const About = memo(() => {
  return <h1>About</h1>
})

export default About

```

#### **Article.jsx**

```jsx
import React, { memo } from 'react'

const Article = memo(() => {
  return <h1>Article</h1>
})

export default Article

```



<!-- tabs:end -->

在 `src/router` 目录下，新建 `index.js`  文件，用来存放路由相关的信息，之后导出在 `src/index.js` 中引入。`App.jsx`用作页面的布局文件，负责路由的跳转和渲染。

<!-- tabs:start -->

#### **router/index.js**

```js
import React, { lazy } from 'react'
import { useRoutes } from 'react-router-dom'
import App from '@/App'

// 路由懒加载
const lazyLoad = (path) => {
  const Comp = lazy(() => import(`@/pages/${path}`))
  return (
    <React.Suspense fallback={<>加载中...</>}>
      <Comp />
    </React.Suspense>
  )
}

export const GetRoutes = () => {
  const routes = useRoutes([
    {
      path: '/',
      element: <App />,
      children: [
        {
          index: true,
          element: lazyLoad('Home')
        },
        {
          path: 'about',
          element: lazyLoad('About')
        },
        {
          path: 'article',
          element: lazyLoad('Article')
        }
      ]
    }
  ])

  return routes
}

```

#### **src/index.js**

```js
import React from 'react'
import ReactDOM from 'react-dom/client'
import { BrowserRouter } from 'react-router-dom'
import { GetRoutes } from '@/router'

const root = ReactDOM.createRoot(document.getElementById('root'))

root.render(
  <BrowserRouter>
    <GetRoutes />
  </BrowserRouter>
)

```

#### **App.jsx**

```jsx
import React, { memo } from 'react'
import { NavLink, Outlet } from 'react-router-dom'
import './index.less'

const App = memo(() => {
  return (
    <div className="app">
      <h1>React + React-router + Redux + RTK</h1>
      <nav>
        <NavLink to="/">HOME</NavLink>
        <NavLink to="/about">ABOUT</NavLink>
        <NavLink to="/article">ARTICLE</NavLink>
      </nav>
      <main>
        <Outlet />
      </main>
    </div>
  )
})

export default App

```



<!-- tabs:end -->

这里需要注意的是：由于我们使用的是H5的 `history` 模式，因此在路由页直接刷新页面的话，浏览器会空白，这个时候需要我们去配置一下 `devServer` 。

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

## 引入 Redux

安装：

```shell
$ pnpm add react-redux @reduxjs/toolkit
```

`src/store`的目录结构如下：

```shell
store
├── index.js
├── counter.js
└── userInfo.js
```

`index.js` 文件，用于统一导出所有的 `store` 对象，`userInfo.js` 和 `counter.js` 是具体的 `store` 用例。

<!-- tabs:start -->

#### **counter.js**

```js
import { createSlice } from '@reduxjs/toolkit'

const initialState = {
  counter: 0
}

export const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    increment: (state) => {
      state.counter += 1
    },
    decrement: (state) => {
      state.counter -= 1
    }
  }
})

export const { increment, decrement } = counterSlice.actions
export default counterSlice.reducer

```

#### **userInfo.js**

```js
import { createSlice } from '@reduxjs/toolkit'

const initialState = {
  name: 'web',
  age: 24
}

export const userInfoSlice = createSlice({
  name: 'userInfo',
  initialState,
  reducers: {
    changeInfo: (state, { payload }) => {
      state.name = payload.name || state.name
      state.age = payload.age || state.age
    }
  }
})

export const { changeInfo } = userInfoSlice.actions
export default userInfoSlice.reducer

```

#### **index.js**

```js
import { configureStore } from '@reduxjs/toolkit'
import counterReducer from './counter'
import userInfoReducer from './userInfo'

const store = configureStore({
  reducer: {
    counter: counterReducer,
    userInfo: userInfoReducer
  }
})

export default store

```



<!-- tabs:end -->

然后在 `src/index.js` 入口文件中进行声明：

```js
// src/index.js
import React from 'react'
import ReactDOM from 'react-dom/client'
import { Provider } from 'react-redux'
import { BrowserRouter } from 'react-router-dom'
import { GetRoutes } from '@/router'
import store from './store'

const root = ReactDOM.createRoot(document.getElementById('root'))

root.render(
  <Provider store={store}>
    <BrowserRouter>
      <GetRoutes />
    </BrowserRouter>
  </Provider>
)
```

在各个组件中进行使用：

<!-- tabs:start -->

#### **CssCom**

```jsx
import React, { memo } from 'react'
import { useDispatch } from 'react-redux'
import { increment, decrement } from '@/store/counter'

import './index.css'

const CssDom = memo((props) => {
  const { type } = props

  const dispatch = useDispatch()

  return (
    <div>
      <h2 className="css">
        测试{type}文件是否<span>正常加载</span>
      </h2>
      <button onClick={() => dispatch(increment())}>count++</button>
      <button onClick={() => dispatch(decrement())}>count--</button>
    </div>
  )
})

export default CssDom

```

#### **LessCom**

```jsx
import React, { memo } from 'react'
import { useSelector, shallowEqual, useDispatch } from 'react-redux'
import { changeInfo as change } from '@/store/userInfo'
import './index.less'

const LessCom = memo(({ type }) => {
  const { name, age } = useSelector((state) => state.userInfo, shallowEqual)
  const dispatch = useDispatch()

  const changeInfo = () => {
    dispatch(
      change({
        age: age + 1
      })
    )
  }

  return (
    <>
      <h2 className="less">
        测试{type}文件是否<span>正常加载</span>
      </h2>
      <h3>
        name: {name} age: {age}
      </h3>
      <button onClick={changeInfo}>age++</button>
    </>
  )
})

export default LessCom

```

#### **SassCom**

```jsx
import React, { memo } from 'react'
import { useSelector, shallowEqual } from 'react-redux'
import './index.sass'

const SassCom = memo(({ type }) => {
  const { counter } = useSelector((state) => state.counter, shallowEqual)

  return (
    <>
      <h2 className="sass">
        测试{type}文件是否<span>正常加载</span>
      </h2>
      <h2>counter: {counter}</h2>
    </>
  )
})

export default SassCom

```

#### **ScssCom**

```jsx
import React, { memo } from 'react'
import { useSelector, shallowEqual, useDispatch } from 'react-redux'
import { changeInfo } from '@/store/userInfo'
import './index.scss'

const ScssCom = memo(({ type }) => {
  const userInfo = useSelector((state) => state.userInfo, shallowEqual)
  const dispatch = useDispatch()

  return (
    <>
      <h2 className="scss">
        测试{type}文件是否<span>正常加载</span>
      </h2>
      <h3>
        name: {userInfo.name} age: {userInfo.age}
      </h3>
      <button onClick={() => dispatch(changeInfo({ name: 'react' }))}>
        更改姓名
      </button>
    </>
  )
})

export default ScssCom

```



<!-- tabs:end -->

> 当前案例的完整代码地址：https://github.com/xinxin1228/webpack-actual/tree/main/05_%E6%90%AD%E5%BB%BAReact%E9%A1%B9%E7%9B%AE%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83_js%E7%89%88

## JS版 webpack 配置项总结

<!-- tabs:start -->

#### **webpack.comm.config.js**

```js
const path = require('node:path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

/**
 * @type {import('webpack').Configuration}
 */
module.exports = {
  entry: './src/index.js',
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        loader: 'babel-loader',
        exclude: /node_modules/,
        options: {
          presets: ['@babel/preset-env', '@babel/preset-react']
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
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, '../public/index.html')
    })
  ],
  cache: {
    type: 'filesystem'
  },
  resolve: {
    extensions: ['.js', '.jsx'],
    alias: {
      '@': path.resolve(__dirname, '../src')
    }
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
    port: 8080,
    hot: true,
    historyApiFallback: true
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader', 'postcss-loader']
      },
      {
        test: /\.less$/,
        use: ['style-loader', 'css-loader', 'postcss-loader', 'less-loader']
      },
      {
        test: /\.s[ac]ss$/,
        use: ['style-loader', 'css-loader', 'postcss-loader', 'sass-loader']
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
    clean: true,
    path: path.resolve(__dirname, '../dist'),
    filename: 'js/[name].[contenthash:8].js',
    chunkFilename: 'js/chunk-[name].[contenthash:8].js'
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
    new MiniCssExtractPlugin({
      filename: 'css/[name].[contenthash:8].css'
    })
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

## 添加 TypeScript

安装：

```shell
$ pnpm add typescript ts-loader @types/react @types/react-dom -D
```

生成 `tsconfig.json` 文件，直接在命令行中输入：

```shell
$ tsc --init
```

然后根据自己的需求进行保留和更改配置项，这里是我用的配置项内容：

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "esnext",
    // "strict": true,
    "jsx": "react",
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
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx"],
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
        test: /\.tsx?$/,
        loader: 'ts-loader',
        exclude: /node_modules/
      }
    ]
  }
}
```

然后在 `src` 目录下创建一个 `types.d.ts` 类型声明文件。

```js
// 支持各种图片
declare module '*.png'
declare module '*.jpg'
declare module '*.jpeg'
declare module '*.gif'
declare module '*.svg'
```

### 重构 Store 结构

目录如下：

```js
// store
├─ index.ts     // 导出store文件
├─ hooks.ts     // 给useDispatch 和 useSelector 加上类型定义，方便组件直接使用
├─ features     // 模块 分模块使用
├───── userInfo.ts   // home 自带 action和 reducer
├───── counter.ts
```

<!-- tabs:start -->

#### **userInfo.ts**

```ts
import { createSlice } from '@reduxjs/toolkit'

const initialState = {
  name: 'web',
  age: 24
}

export const userInfoSlice = createSlice({
  name: 'userInfo',
  initialState,
  reducers: {
    changeInfo: (state, { payload }) => {
      state.name = payload.name || state.name
      state.age = payload.age || state.age
    }
  }
})

export const { changeInfo } = userInfoSlice.actions
export default userInfoSlice.reducer

```

#### **counter.ts**

```ts
import { createSlice } from '@reduxjs/toolkit'

const initialState = {
  counter: 0
}

export const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    increment: (state) => {
      state.counter += 1
    },
    decrement: (state) => {
      state.counter -= 1
    }
  }
})

export const { increment, decrement } = counterSlice.actions
export default counterSlice.reducer

```

#### **hooks.ts**

```ts
import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux'

import { StoreTypes, AppDispatch } from '.'

// use hook 节约每次引入type的工作
// useSelector: 节约配置RootState type

export const useAppDispatch = () => useDispatch<AppDispatch>()

export const useAppSelector: TypedUseSelectorHook<StoreTypes> = useSelector

```

#### **index.ts**

```ts
import { configureStore } from '@reduxjs/toolkit'
import UserInfoReducer from './features/userInfo'
import CounterReducer from './features/counter'

// 导出store
const store = configureStore({
  reducer: {
    userInfo: UserInfoReducer,
    counter: CounterReducer
  }
})
export default store

export type StoreTypes = ReturnType<typeof store.getState>
// AppDispatch 作用是拿到Store的dispatch方法的类型 function
export type AppDispatch = typeof store.dispatch

```



<!-- tabs:end -->

项目中使用如下：

<!-- tabs:start -->

#### **CssCom**

```tsx
import React, { memo } from 'react'
import { increment, decrement } from '@/store/features/counter'
import { useAppDispatch } from '@/store/hooks'
import './index.css'

interface Props {
  type: string
}

const CssDom: React.FC<Props> = memo((props) => {
  const { type } = props

  const dispatch = useAppDispatch()

  return (
    <div>
      <h2 className="css">
        测试{type}文件是否<span>正常加载</span>
      </h2>
      <button onClick={() => dispatch(increment())}>count++</button>
      <button onClick={() => dispatch(decrement())}>count--</button>
    </div>
  )
})

export default CssDom

```

#### **LessCom**

```tsx
import React, { memo } from 'react'
import { shallowEqual } from 'react-redux'
import { changeInfo as change } from '@/store/features/userInfo'
import { useAppDispatch, useAppSelector } from '@/store/hooks'
import './index.less'

interface Props {
  type: string
}

const LessCom: React.FC<Props> = memo(({ type }) => {
  const { name, age } = useAppSelector((state) => state.userInfo, shallowEqual)
  const dispatch = useAppDispatch()

  const changeInfo = () => {
    dispatch(
      change({
        age: age + 1
      })
    )
  }

  return (
    <>
      <h2 className="less">
        测试{type}文件是否<span>正常加载</span>
      </h2>
      <h3>
        name: {name} age: {age}
      </h3>
      <button onClick={changeInfo}>age++</button>
    </>
  )
})

export default LessCom

```

#### **SassCom**

```tsx
import React, { memo } from 'react'
import { shallowEqual } from 'react-redux'
import { useAppSelector } from '@/store/hooks'
import './index.sass'

interface Props {
  type: string
}

const SassCom: React.FC<Props> = memo(({ type }) => {
  const { counter } = useAppSelector((state) => state.counter, shallowEqual)

  return (
    <>
      <h2 className="sass">
        测试{type}文件是否<span>正常加载</span>
      </h2>
      <h2>counter: {counter}</h2>
    </>
  )
})

export default SassCom

```

#### **ScssCom**

```tsx
import React, { memo } from 'react'
import { shallowEqual } from 'react-redux'
import { changeInfo } from '@/store/features/userInfo'
import { useAppDispatch, useAppSelector } from '@/store/hooks'
import './index.scss'

interface Props {
  type: string
}

const ScssCom: React.FC<Props> = memo(({ type }) => {
  const userInfo = useAppSelector((state) => state.userInfo, shallowEqual)
  const dispatch = useAppDispatch()

  return (
    <>
      <h2 className="scss">
        测试{type}文件是否<span>正常加载</span>
      </h2>
      <h3>
        name: {userInfo.name} age: {userInfo.age}
      </h3>
      <button onClick={() => dispatch(changeInfo({ name: 'react' }))}>
        更改姓名
      </button>
    </>
  )
})

export default ScssCom

```



<!-- tabs:end -->



## TS版 webpack 配置项总结

<!-- tabs:start -->

#### **webpack.comm.config.js**

```js
const path = require('node:path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

/**
 * @type {import('webpack').Configuration}
 */
module.exports = {
  entry: './src/index.tsx',
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env', '@babel/preset-react']
        }
      },
      {
        test: /\.tsx?$/,
        exclude: /node_modules/,
        loader: 'ts-loader'
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
      }
    ]
  },
  resolve: {
    extensions: ['.js', '.ts', '.tsx', '.jsx'],
    alias: {
      '@': path.resolve(__dirname, '../src')
    }
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, '../public/index.html')
    })
  ],
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
    port: 8080,
    open: true,
    historyApiFallback: true,
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader', 'postcss-loader'],
      },
      {
        test: /\.less$/,
        use: ['style-loader', 'css-loader', 'postcss-loader', 'less-loader'],
      },
      {
        test: /\.s[ac]ss$/,
        use: ['style-loader', 'css-loader', 'postcss-loader', 'sass-loader'],
      },
    ],
  },
  devtool: 'eval-cheap-module-source-map',
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
    filename: 'js/[name].[contenthash:8].js',
    chunkFilename: 'js/chunk-[name].[contenthash:8].js',
    clean: true,
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader'],
      },
      {
        test: /\.less$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader',
          'postcss-loader',
          'less-loader',
        ],
      },
      {
        test: /\.s[ac]ss$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader',
          'postcss-loader',
          'sass-loader',
        ],
      },
    ],
  },
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
          minChunks: 1,
        },
        common: {
          chunks: 'all',
          priority: -20,
          filename: 'commons.[contenthash].js',
          minChunks: 2, //被至少2个chunk引/用时才会被提取
          reuseExistingChunk: true, //如果模块已经被打包进某个chunk，则复用该chunk
        },
      },
    },
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: 'css/[name].[contenthash:8].css',
    }),
  ],
  devtool: 'nosources-source-map',
})

```



<!-- tabs:end -->

> 当前代码的完整地址：https://github.com/xinxin1228/webpack-actual/tree/main/06_%E6%90%AD%E5%BB%BAReact%E9%A1%B9%E7%9B%AE%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83_ts%E7%89%88