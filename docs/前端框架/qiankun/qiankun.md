## react18作为主应用

### 注意：

子应用的挂载点一定不能和父应用的挂载点重复，比如react默认的挂载是root 那么子应用的挂载点应将html文件的挂载点改为其他，不能继续采用root

### 第一步

在 主应用上安装 qiankun

```js
npm i qiankun -S
```

在 src/index.ts替换成下面代码

```js
import React, { useEffect } from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import reportWebVitals from './reportWebVitals'
import { registerMicroApps, start } from 'qiankun'
import { BrowserRouter } from 'react-router-dom'

import { GetRoutes } from './router'

registerMicroApps([
  {
    name: 'qiankun-app1', // 项目名称 必须唯一
    entry: 'http://localhost:3000', // 项目路径
    container: '#container', // 挂在容器
    activeRule: '/react1', // 访问路由
  },
  {
    name: 'vue01',
    entry: 'http://localhost:8080', 
    container: '#container',
    activeRule: '/vue01',
  },
])

start()

const root = ReactDOM.createRoot(document.getElementById('root') as HTMLElement)
root.render(
  <React.StrictMode>
    <BrowserRouter>
      <GetRoutes />
    </BrowserRouter>
  </React.StrictMode>
)

reportWebVitals()

```

### 第二步：

导出子应用的声明周期

#### react子应用

```js
npm i react-app-rewired customize-cra -D
```

在 `src` 下新建 `public-path.js`文件

```js
if (window.__POWERED_BY_QIANKUN__) {
  // eslint-disable-next-line no-undef
  __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
}
```

在 `根目录`下新建 `config-overrides.js`

```js
const { name } = require('./package');
const path = require("path")
const { override, addWebpackAlias } = require("customize-cra");


module.exports = {
  webpack:
    override(
      addWebpackAlias({
        "@": path.join(__dirname, "./src"),
      }),
      (config) => {
        config.output.library = `${name}-[name]`;
        config.output.libraryTarget = 'umd';
        // config.output.jsonpFunction = `webpackJsonp_${name}`;
        config.output.globalObject = 'window';

        return config;
      }),

  devServer: (_) => {
    const config = _;

    config.headers = {
      'Access-Control-Allow-Origin': '*',
    };
    config.historyApiFallback = true;
    config.hot = false;
    config.watchContentBase = false;
    config.liveReload = false;

    return config;
  },
};

```

在 `index.tsx` 中导出 `qiankun` 声明周期钩子

```tsx
import './public-path'
import React from 'react'
import ReactDOM from 'react-dom/client'
import { Provider } from 'react-redux'
import { BrowserRouter } from 'react-router-dom'

import reportWebVitals from './reportWebVitals'
import { GetRoutes } from './router'
import store from './redux'
import './index.css'

//@ts-ignore
const BASE_NAME = window.__POWERED_BY_QIANKUN__ ? '/business' : '/'

let root: any = null

function render(props: { container?: any }) {
  let { container } = props

  container = container
    ? container.querySelector('#business1')
    : document.querySelector('#business1')
  root = ReactDOM.createRoot(container)

  root.render(
    <Provider store={store}>
      <BrowserRouter basename={BASE_NAME}>
        <GetRoutes />
      </BrowserRouter>
    </Provider>
  )
}

//@ts-ignore
if (!window.__POWERED_BY_QIANKUN__) {
  render({})
}

export async function bootstrap() {}

export async function mount(props: { container?: any }) {
  render(props)
}

//@ts-ignore
export async function unmount(props) {
  // root?.unmount()
  // root = null
  const { container } = props
  //@ts-ignore
  root.unmount(
    container
      ? container.querySelector('#business1')
      : document.querySelector('#business1')
  )
}

reportWebVitals()

```

#### Vue3子应用

在 `src` 下新建 `public-path.js`文件

```js
if (window.__POWERED_BY_QIANKUN__) {
  // eslint-disable-next-line no-undef
  __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
}
```

覆盖 根目录文件 `vue.config.js`

```js
const { name } = require('./package')

const port = 6000

module.exports = {
  outputDir: 'dist',
  assetsDir: 'static',
  filenameHashing: true,
  devServer: {
    hot: true,
    port,
    headers: {
      'Access-Control-Allow-Origin': '*'
    }
  },
  // 自定义webpack配置
  configureWebpack: {
    output: {
      // 把子应用打包成 umd 库格式
      library: `${name}-[name]`,
      libraryTarget: 'umd',
      chunkLoadingGlobal: `webpackJsonp_${name}`
    }
  }
}

```

覆盖 `src/main.ts`的内容

```ts
import './public-path'
import { createApp } from 'vue'
import { createRouter, createWebHistory } from 'vue-router'
import App from './App.vue'
import routes from './router'
import store from './store'

let router = null
let instance: any = null
let history: any = null

function render(props = {}) {
  // @ts-ignore
  const { container } = props
  // 当为微服务主节点情况下访问，会设置二级路径，而直接访问时没有二级路径，此处需要根据实际情况修改
  // @ts-ignore
  history = createWebHistory(window.__POWERED_BY_QIANKUN__ ? '/vue01' : '/')
  router = createRouter({
    history,
    routes
  })

  instance = createApp(App)
  instance.use(router)
  instance.use(store)
  instance.mount(container ? container.querySelector('#app') : '#app')
}

// @ts-ignore
if (!window.__POWERED_BY_QIANKUN__) {
  render()
}

export const bootstrap = async (): Promise<void> => {
  // console.log('%c ', 'color: green ', 'vue3.0 app bootstraped')
}

export const mount = async (props: any): Promise<void> => {
  render(props)
  instance.config.globalProperties.$onGlobalStateChange = props.onGlobalStateChange
  instance.config.globalProperties.$setGlobalState = props.setGlobalState
}

export const unmount = async (): Promise<void> => {
  instance.unmount()
  instance._container.innerHTML = ''
  instance = null
  router = null
  history.destroy()
}

```

