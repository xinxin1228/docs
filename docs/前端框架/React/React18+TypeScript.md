## React18-TypeScript结合使用

#### 安装

```tsx
npx create-react-app 项目名称 --template typescipt 
```

#### 安装Prettier

```shell
// 安装prettier  并且版本锁定，安装开发依赖
npm install --save-dev --save-exact prettier

// 在根目录创建 .prettierrc 规则文件
{
  "useTabs": false,
  "tabWidth": 2,
  "printWidth": 80,
  "singleQuote": true,
  "trailingComma": "none",
  "semi": false
}

// 在根目录创建 .prettierignore 忽略规则文件
build
coverage

// 安装Pre-commit Hook，实现每次代码提交前自动格式化

npx mrm@ lint-staged
// 这一行会同时安装husky和lint-stage，并自动生成.husky/pre-commit文件

// 由于React项目自带Eslint, 安装eslint-config-prettier ，解决eslint和prettier规则冲突
npm install --save-dev eslint-config-prettier

// 在package.json中增加
"eslintConfig": {
  "extends": [
    "react-app",
    "react-app/jest",
    "prettier" //新增加内容，替换部分eslint规则
  ]
 },
```

#### 配置别名

```tsx
// 安装
npm install @craco/craco --save-dev
npm install craco-less --save-dev

// 在根目录上新建craco.config.js
const CracoLessPlugin = require("craco-less")
const path = require("path")

const resolve = (dir) => path.resolve(__dirname, dir)

module.exports = {
  webpack: {
    alias: {
      "@": resolve("src"),
      "components": resolve("src/components")
    },
  },
}

// 修改packjson下的script内容
"scripts": {
  "start": "craco start",
  "build": "craco build",
  "test": "craco test",
  "eject": "react-scripts eject"
},

// 由于这是ts文件，配置完 @ 之后还无法在项目中直接使用，需要配置
  
1. 配置tsconfig.json 让vscode和React识别@
// tsconfig.json
{
    "compilerOptions": {
        "baseUrl": "./",
        "paths": {
          "@/*": ["src/*"]
        }
    },
    "exclude": [
        "node_modules"
    ]
}
```

#### 安装顶部进度条nprogress

```js
// 安装
npm install nprogress --save && npm install @types/nprogress --save-dev

// 使用
// 顶部进度条
import React, { memo, useEffect } from 'react'
import nprogress from 'nprogress'
import 'nprogress/nprogress.css'

//进度条配置
nprogress.configure({
  easing: 'ease', // 动画方式
  speed: 500, // 递增进度条的速度
  showSpinner: true, // 是否显示加载ico
  trickleSpeed: 200, // 自动递增间隔
  minimum: 0.3 // 初始化时的最小百分比
})

const TopLoading: React.FC = memo(() => {
  useEffect(() => {
    nprogress.start()

    return () => {
      nprogress.done()
    }
  }, [])

  return <></>
})

export default TopLoading

```



#### 安装react-router

```tsx
npm install react-router-dom --save
npm instal @types/react-router-dom --save-dev


// 使用
// router
├─ index.tsx
├─ client
├───── index.tsx
├─ admin
├───── index.tsx


// index.tsx
import { useRoutes, RouteObject } from "react-router-dom"

import ClientLayout from "../layout/client"
import AdminLayout from "../layout/admin"

import clientRoutes from "./client"
import adminRoutes from "./admin"

export const GetRoutes = () => {
  const routes: RouteObject[] = [
    // 前台路由
    {
      path: "/",
      element: <ClientLayout />,
      children: clientRoutes,
    },
    // 后台路由
    {
      path: "/admin",
      element: <AdminLayout />,
      children: adminRoutes,
    },
  ]

  return useRoutes(routes)
}


// client/index.tsx
import { RouteObject } from "react-router-dom"

import { clientLazyLoad } from "../../utils/lazy-router"

import Home from "../../views/client/home"

const clientRoutes: RouteObject[] = [
  // 首页
  {
    path: "",
    element: <Home />,
  },
  // 关于
  {
    path: "about",
    element: clientLazyLoad("/about"),
  },
]

export default clientRoutes


// 懒加载路由
export const clientLazyLoad = (path: string) => {
  const Comp = lazy(() => import(`../views/client${path}`))
  return (
    <React.Suspense fallback={<>加载中...</>}>
      <Comp />
    </React.Suspense>
  )
}


// index.tsx
import { BrowserRouter } from "react-router-dom"

import { GetRoutes } from "@/router"

const root = ReactDOM.createRoot(document.getElementById("root"))
root.render(
  <BrowserRouter>
    <GetRoutes />
  </BrowserRouter>
)

```

#### 使用Redux

```tsx
// react18中发生了重大的改革，不再推荐从redux中导出createStore，而是改用使用工具 @reduxjs/toolkit，并且在redux模块的书写方式也发生了巨大变化，不再书写reducer和action文件，而是将它们合在了一起，只需一个文件，并且内置了immutable，无须手动引入

// 安装
npm install react-redux @reduxjs/toolkit --save

// 目录
// store
├─ index.ts     // 导出store文件
├─ hooks.ts     // 给useDispatch 和 useSelector 加上类型定义，方便组件直接使用
├─ features     // 模块 分模块使用
├───── home.ts   // home 自带 action和 reducer
├───── link.ts
├───── about.ts

// index.ts
import { configureStore } from "@reduxjs/toolkit"

import HomeReducer from "./features/home"
import CountReducer from "./features/count"

import type { InitialStateTypes as HomeTypes } from "./features/home"
import type { InitialStateTypes as CountType } from "./features/count"

// 导出store
const store = configureStore({
  reducer: {
    home: HomeReducer,
    count: CountReducer,
  },
})
export default store

// export type StoreTypes = {
//   home: HomeTypes
//   count: CountType
// }

// 类似于上面的方法
// StoreTypes作用是返回store的方法getState的类型 function
// store.getState就是store里面的reducer类型
export type StoreTypes = ReturnType<typeof store.getState>

// AppDispatch 作用是拿到Store的dispatch方法的类型 function
export type AppDispatch = typeof store.dispatch
                                    

// hooks.ts
import { useDispatch, useSelector, TypedUseSelectorHook } from "react-redux"

import { StoreTypes, AppDispatch } from "."

// use hook 节约每次引入type的工作
// useSelector: 节约配置RootState type

export const useAppDispatch = () => useDispatch<AppDispatch>()

export const useAppSelector: TypedUseSelectorHook<StoreTypes> = useSelector


// 在 index.tsx 根组件引用
import React from "react"
import ReactDOM from "react-dom/client"
import App from "./App"
import { Provider } from "react-redux"

import store from "./store"

const root = ReactDOM.createRoot(document.getElementById("root") as HTMLElement)
root.render(
  <Provider store={store}>
    <App />
  </Provider>
)


// home.ts 每个模块处理
import { createSlice, createAsyncThunk } from "@reduxjs/toolkit"

import { getData } from "../../server/home"

export interface InitialStateTypes {
  title: string
  movesList: Array<any>
}

const initialState: InitialStateTypes = {
  title: "默认标题",
  movesList: [],
}

export const homeSlice = createSlice({
  // 命名空间
  name: "home",
  // 初始值
  initialState,
  // reducer和action结合体
  reducers: {
    // 相当于action,
    // 第一个参数是immutable之后的state
    // 第二个参数是 { type, payload }结合体
    updateTitle(state, { payload }) {
      state.title = payload
    },
  },
  extraReducers: {
    [asyncUpdateTitle.fulfilled.type](state, { payload }) {
      state.movesList = payload.data.list
    },
  },
  // 目前推荐下面这种写法，这样就可以将action异步操作代码放到下面
  extraReducers: (builder) => {
    builder.addCase(asyncUpdateTitle.fulfilled, (state, { payload }) => {
      state.movesList = payload.data.list
    })
  },
})

// 导出action
export const { updateTitle, updateBanner, updateMovesList } = homeSlice.actions

// 异步操作
export const asyncUpdateTitle = createAsyncThunk(
  "home/asyncUpdateTitle",
  async () => {
    return (await getData()).data
  }
)

// 导出reducer
export default homeSlice.reducer



// 组件中使用
import React, { memo, useEffect, useState } from "react"
import { shallowEqual } from "react-redux"

import { useAppDispatch, useAppSelector } from "./store/hooks"

import {
  updateTitle,
  asyncUpdateTitle,
  updateBanner,
} from "./store/features/home"
import { addNum, subNum, asyncGetMoves } from "./store/features/count"

const App: React.FC = memo(() => {
  const dispatch = useAppDispatch()
  const { title, banner, movesList } = useAppSelector((state) => state.home, shallowEqual)
  const { num } = useAppSelector((state) => state.count, shallowEqual)

  useEffect(() => {
    dispatch(asyncGetMoves())
  }, [dispatch])

  const handleClick = () => {
    dispatch(updateTitle("web前端"))
  }

  const addBanner = () => {
    dispatch(updateBanner(text))
    setText("")
  }

  return (
    <div>
      <h2>当前title：{title}</h2>
      <button onClick={handleClick}>更新title为web前端</button>
      <br />
      <ul>
        {movesList.map((item) => (
          <li key={item.title}>{item.title}</li>
        ))}
      </ul>
    </div>
  )
})

export default App

```

#### RTK中在extraReducers中使用reducers

```tsx
export const noticeSlice = createSlice({
  name: 'notice',
  initialState,
  reducers: {
    // 改变消息数量
    updateNoticeAction(state, { payload }) {
      state.unreadPreSaleNum = payload
    }
  },
  extraReducers(builder) {
    builder.addCase(asyncGetAllNoticeAction.fulfilled, (state, { payload }) => {
      // 调用noticeSlice.caseReducers上调用
      noticeSlice.caseReducers.updateNoticeAction(state, {
        payload: 1000,
        type: 'updateNoticeAction'
      }) //
      // 数据过滤
    })
  }
})

```

#### 使用styled-components

```shell
npm install styled-components --save
npm install @types/styled-components --save-dev
```

#### 使用 animate.css实现后台系统中页面之间的切换

```js
npm install react-transition-group -S
npm install @types/react-transition-group -D
npm install animate.css --save
 
```



#### 使用styled-components组件传递的props

```tsx
// index.tsx
<ListStyled key={item.id} cover={item.cover}>
  <div className="info"></div>
  <div className="link"></div>
</ListStyled>


// style.ts
interface ListType {
  cover: string
}
export const ListStyled = styled('div')<ListType>`
  width: 377px;
  height: 216px;
  background-color: red;
  background: url(${(props) => props.cover});
`



```



#### 二次封装axios

```tsx
// axios.tsx
// axios二次封装
import * as ReactDOM from 'react-dom/client'

import axios, { AxiosRequestConfig, AxiosResponse } from 'axios'

import Loading from '@/pages/global/loading'

import { ECHO_DURATION } from '@/global.config'

import { message } from 'antd'

export interface AxiosDefaultTypes<T> {
  author: string
  time: string
  des: string
  data: T
}

// 当前正在请求的数量
let requestCount = 0

// 显示loading
function showLoading() {
  if (requestCount === 0) {
    const oWrapper = document.createElement('div')
    oWrapper.setAttribute('id', 'loading')
    document.body.appendChild(oWrapper as HTMLElement)
    const root = ReactDOM.createRoot(oWrapper as HTMLElement)
    root.render(<Loading />)
  }
  requestCount++
}

// 关闭loading
function hideLoading() {
  requestCount--
  if (requestCount === 0) {
    setTimeout(() => {
      document.body.removeChild(
        document.getElementById('loading') as HTMLElement
      )
    }, ECHO_DURATION)
  }
}

export const baseURL: string =
  process.env.NODE_ENV === 'development' ? 'http://localhost:3000' : ''

axios.defaults.baseURL = baseURL as string
axios.defaults.withCredentials = true as boolean
axios.defaults.timeout = 5000

// 添加请求拦截器
axios.interceptors.request.use((config: AxiosRequestConfig) => {
  // 默认开启loading加载动画
  if (config.headers?.isLoading !== false) showLoading()
  const token: string | null = localStorage.getItem('token')
  if (!config.headers) return
  // 添加token
  config.headers.Authorization = 'Bearer ' + token
  return config
})

// 添加响应拦截器
axios.interceptors.response.use(
  (response: AxiosResponse<AxiosDefaultTypes<any>>) => {
    // 对响应数据做点什么
    // 关闭loading加载动画
    if (response.config.headers?.isLoading !== false) hideLoading()
    return response
  },
  (err) => {
    // 对响应错误做点什么
    // 关闭loading加载动画
    if (err.config.headers?.isLoading !== false) hideLoading()
    if (err.message === 'Network Error') message.warning('网络链接异常！')
    if (err.code === 'ECONNABORTED') message.warning('请求超时，请重试')
    return Promise.reject(err)
  }
)

export default axios

```

#### 配置跨域

```tsx
// 方法一：
// package.json
"proxy": "http://localhost:5000/api"

本地请求方式： /getData
目标地址： http://localhost:5000/api/getData

缺点： 只能配置一个跨域


// 方法二：
```

#### react-router-dom6使用动画路由

```tsx
// 第一步 改变 src/router/index.tsx 文件


// 改变方式如下，给每一个routes 下的 route 列表添加 nodeRef 字段， 

import { createRef } from 'react'
import { useRoutes } from 'react-router-dom'

import { lazyLoadRouter } from '@/utils/lazy-router'
import Layout from '../layout'

export const routes = [
  {
    path: '/',
    element: <Layout />,
    nodeRef: createRef(),
    children: [
      {
        path: '',
        element: lazyLoadRouter('home'),
        nodeRef: createRef()
      },
      {
        path: '/admin/question',
        element: lazyLoadRouter('admin/question'),
        nodeRef: createRef()
      },
      {
        path: '*',
        element: lazyLoadRouter('notFound'),
        nodeRef: createRef()
      }
    ]
  }
]

export const GetRoutes = () => {
  return useRoutes(routes)
}

// 第二步 在 src/index.tsx 的引入方式和上面的一样

// 第三步 不再 使用 Outlet 组件显示内容 而是改为 useOutlet
import React, { memo, useState } from 'react'
import { useOutlet, useNavigate, useLocation } from 'react-router-dom'
import { SwitchTransition, CSSTransition } from 'react-transition-group'
import { Layout } from 'antd'
import {
  WindowsOutlined,
  MenuUnfoldOutlined,
  MenuFoldOutlined
} from '@ant-design/icons'

import { SiderMenu, Crumbs, UserInfo } from '@/components'
import { LayoutStyled } from './style'
import { routes } from '@/router'

const { Header, Sider } = Layout

const LayoutCom = memo(() => {
  const location = useLocation()
  const currentOutlet = useOutlet()

  const { nodeRef } =
    routes.find((route) => route.path === location.pathname) ?? {}

  return (
    <LayoutStyled>
      <SwitchTransition>
        <CSSTransition
          key={location.pathname}
          nodeRef={nodeRef as any}
          timeout={300}
          classNames="page"
          unmountOnExit
          appear
          >
          {(state) => (
            <div ref={nodeRef as any} className="page">
              {currentOutlet}
            </div>
          )}
        </CSSTransition>
      </SwitchTransition>
    </LayoutStyled>
  )
})

export default LayoutCom


// 动画样式如下：
/* 动画 */
.page-enter,
.page-appear {
    opacity: 0;
    transform: translateX(-25px);
  }
.page-enter-active,
  .page-appear-active {
    opacity: 1;
    transition: opacity 300ms, transform 300ms;
    transform: translateX(0px);
  }
.page-exit {
  opacity: 1;
  transform: translateX(0px);
}
.page-exit-active {
  opacity: 0;
  transform: translateX(-25px);
  transition: opacity 300ms, transform 300ms;
}


```
