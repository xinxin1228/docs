## 基础知识

### 安装使用

#### HTML 中引入 React

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="author" content="新新小朋友" />
    <link
      rel="icon"
      href="http://huangjianxin.cn/favicon.ico"
      type="image/x-icon"
    />
    <title>新新小朋友</title>
    <script
      src="https://unpkg.com/react@17/umd/react.development.js"
      crossorigin
    ></script>
    <script
      src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"
      crossorigin
    ></script>
    <script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
  </head>
  <body>
    <!-- 挂载的容器 -->
    <div id="root"></div>
    <!-- type='text/bable' 可以编译转化jsx语法 -->
    <script type="text/babel">
      class App extends React.Component {
        state = {
          val: '我是React',
          moves: ['星际穿越', '大话西游', '盗梦空间', '少年派'],
          num: 0,
        }

        handleClick = () => {
          this.setState({
            val: 'Hello Word',
          })
        }

        addClick = () => {
          this.setState({ num: (this.state.num += 1) })
        }

        downClick = () => {
          this.setState({ num: (this.state.num -= 1) })
        }

        render() {
          return (
            <div>
              <h1>{this.state.val}</h1>
              <button onClick={this.handleClick}>改变内容</button>
              <ul>
                {this.state.moves.map(item => (
                  <li key={item}>{item}</li>
                ))}
              </ul>
              <h2>当前计数：{this.state.num}</h2>
              <button onClick={this.addClick}>+1</button>
              <button onClick={this.downClick}>-1</button>
            </div>
          )
        }
      }

      ReactDOM.render(<App />, document.getElementById('root'))
    </script>
  </body>
</html>
```

#### 脚手架安装

```js
npx create-react-app 项目名称
// 项目名称不能包含大写字母

// 脚手架安装react
npx create-react-app my-react
```

#### React18 初始化脚手架

```js
import React from 'react'
import ReactDOM from 'react-dom/client'

import App from './App'

const root = ReactDOM.createRoot(document.getElementById('root'))
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

### css 用法

##### 内联样式

> 顾名思义，就是在元素内部书写的内联样式，

- 内联样式是官方推荐的一种 css 样式的写法：
  - style 接受一个采用小驼峰命名属性的 JavaScript 对象，而不是 CSS 字符串：
  - 井且可以引用 state 中的状态来设置相关的样式：
- 内联样式的优点：
  - 1.内联样式，样式之间不会有冲突
  - 2.可以动态获取当前 state 中的状态
- 内联样式的缺点：
  - 1.写法上都需要使用驼峰标识
  - 2.某些样式没有提示
  - 3.大量的样式，代码混乱
  - 4.某些样式无法编写（比如伪类/伪元素）
- 所以官方依然是希望内联合适和普通的 css 来结合编写：

代码如下：

```jsx
import React, { PureComponent } from 'react'

export default class Apps extends PureComponent {
  render() {
    return (
      <>
        <header
          style={{
            width: '100%',
            display: 'flex',
            justifyContent: 'space-between',
            margin: 'auto',
          }}
        >
          <div className="logo">Huangjianxins</div>
          <div className="navList" style={{ display: 'flex' }}>
            <div className="list" style={{ margin: '0 20px' }}>
              首页
            </div>
            <div className="list" style={{ margin: '0 20px' }}>
              文章
            </div>
            <div className="list" style={{ margin: '0 20px' }}>
              留言
            </div>
            <div className="list" style={{ margin: '0 20px' }}>
              有链
            </div>
            <div className="list" style={{ margin: '0 20px' }}>
              关于
            </div>
          </div>
          <div className="userInfo">个人信息</div>
        </header>
      </>
    )
  }
}
```

##### 普通的 css

- 普通的 css 我们通常会编写到一个单独的文件，之后再进行引入。
- 这样的编写方式和普通的网页开发中编写方式是一致的：
  - 如果我们按照普通的网页标准去编写，那么也不会有太大的问题。
  - 但是组件化开发中我们总是希望组件是一个独立的模块，即便是样式也只是在自己内部生效，不会相互影响。
  - 但是普通的 css 都属于全局的 css，样式之间会相互影响。
- 这种编写方式最大的问题是样式之间会相互层叠掉；

代码如下：

```jsx
import React, { PureComponent } from 'react'

// 书写普通样式
import './index.css'

export default class App extends PureComponent {
  render() {
    return (
      <>
        <header>
          <div className="logo">Huangjianxin</div>
          <div className="navList">
            <div className="list">首页</div>
            <div className="list">文章</div>
            <div className="list">留言</div>
            <div className="list">有链</div>
            <div className="list">关于</div>
          </div>
          <div className="userInfo">个人信息</div>
        </header>
      </>
    )
  }
}
```

##### css modules

- css modules 井不是 React 特有的解决方案，而是所有使用了类似于 webpack 配置的环境下都可以使用的。
  - 但是，如果在其他项目中使用个，那么我们需要自己来进行配置，比如配置 webpack.configjs 中的 modules:true 等。
- React 的脚手架已经内置了 css modules 的配置：
  - .css/.less/.scss 等样式文件都修改成 .module.css/.module.less/.module.scss 等
  - 之后就可以引用并且进行使用了；
- css modules 确实解决了局部作用域的问题，也是很多人喜欢在 React 中使用的一种方案。 但是这种方案也有自己的缺陷；
  - 引用的类名，不能使用连接符(.home-title)，在 JavaScript 中是不识别的；
  - 所有的 className 都必须使用(style.className) 的形式来编写；
  - 不方便动态来修改某些样式，依然需要使用内联样式的方式；
- 如果你觉得上面的缺陷还算 OK，那么你在开发中完全可以选择使用 css modules 来编写，并且也是在 React 中很受欢迎的一种方
  式。

代码如下：

```jsx
import React, { PureComponent } from 'react'

// 像普通css一样去书写，然后再使用模块化的方式引入和使用
import style from './index.module.css'

export default class index extends PureComponent {
  render() {
    return (
      <>
        <header>
          <div className={style.logo}>Huangjianxin</div>
          <div className={style.navList}>
            <div className={style.list}>首页</div>
            <div className={style.list}>文章</div>
            <div className={style.list}>留言</div>
            <div className={style.list}>有链</div>
            <div className={style.list}>关于</div>
          </div>
          <div className={style.userInfo}>个人信息</div>
        </header>
      </>
    )
  }
}
```

##### css in js

- 实际上，官方文档也有提到过 CSS in JS 这种方案：
  - “CSS-in-JS" 是指一种模式，其中 CSS 由 JavaScript 生成而不是在外部文件中定义；
  - 注意此功能并不是 React 的一部分，而是由第三方库提供。React 对样式如何定义井没有明确态度；
- 在传统的前端开发中，我们通常会将结构（HTML)、样式（CSS）、逻辑(JavaScript ) 进行分离。
  - 但是在前面的学习中，我们就提到过，React 的思想中认为逻辑本身和 UI 是无法分离的，所以才会有了 JSX 的语法。
  - 样式呢？样式也是属于 UI 的一部分；
  - 事实上 CSS-in-JS 的模式就是一种将样式（CSS）也写入到 JavaScript 中的方式，并且可以方便的使用 JavaScript 的状态：
  - 所以 React 有被人称之为 All in Js；

代码如下：

```jsx
// 使用 styled-components
注意：
当前使用yarn add styled-components会出现报错，不兼容问题（2022-05-11）
替换为：
npm intsall styled-components -S

// 定义 同层新建style.js文件
// 编写代码
import styled from "styled-components"

export const AppStyled = styled.div`
  h3 {
    color: ${(props) => props.color};
  }
`

// attrs可以定义变量或者设置一些其他的属性
export const InputStyled = styled.input.attrs({
  placeholder: "输入密码",
  bgColor: "pink",
})`
  // 使用上面定义的值
  background-color: ${(props) => props.bgColor};
  // 使用state中的值
  color: ${(props) => props.textColor};
`

// 使用
import React, { PureComponent } from "react"

import { AppStyled, InputStyled } from "./style"

export default class App extends PureComponent {
  constructor(props) {
    super(props)

    this.state = {
      currIndex: 0,
      navList: ["首页", "文章", "留言", "友链", "关于"],
      color: "green",
    }
  }

  render() {
    const { currIndex, navList, color } = this.state
    return (
      <AppStyled>
        <header>
          <div className="navList">
            {navList.map((item, index) => (
              <div
                className={"list " + (currIndex === index ? "active" : "")}
                key={item}
                onClick={() => this.handleClick(index)}
              >
                {item}
              </div>
            ))}
          </div>
        </header>
        <InputStyled type="password" color={color} />
      </AppStyled>
    )
  }

  handleClick(index) {
    this.setState({
      currIndex: index,
    })
  }
}



// 用法二： 样式继承
const ButtonStyle = styled.button`
  width: 100px;
  height: 30px;
  color: red;
`

// styled括号里面跟上需要继承的样式
const PrimaryButtonStyle = styled(ButtonStyle)`
  background-color: blue;
`



// 用法三： 主题
// 在App根组件上进行设置
import React, { PureComponent } from "react"

import { ThemeProvider } from "styled-components"

import Home from "./home"

export default class App extends PureComponent {
  render() {
    return (
      // 使用ThemeProvider替换原来的div，并且传入props为theme的主题颜色
      <ThemeProvider theme={{ color: "red", bgcColor: "pink" }}>
        App
        <Home />
      </ThemeProvider>
    )
  }
}

// 使用
// Home/style.js中
import styled from "styled-components"

export const HomeStyle = styled.div`
  h2 {
    color: red;
  }
  p {
  // 在props中的theme存放刚才定义的主题信息
    color: ${(props) => props.theme.color};
  }
  ul {
    background-color: ${(props) => props.theme.bgcColor};
    li {
    }
  }
`
```

### JSX

#### JSX 用法

```js
// JSX是JavaScript XML 的简写，表示在JavaScript代码中写HTML格式的代码，是JavaScript的语法扩展
// 优势：
 1.声明式语法更加直观
 2.与HTML结构相同
 3.降低了学习成本
 4.提升开发效率

// 为什么可以在脚手架中使用JSX语法
JSX 不是标准的ECMAScript语法，它是ECMAScript的语法拓展
- 需要使用babel编译处理后，才能在浏览器环境中使用
- create-react-app脚手架中已经默认有该（babel）配置，无需手动配置
- 编译JSX语法的包： @bable/preset-react

const jsx = (
	<div className="div">
  	<h1>这是我的第一个React项目</h1>
  	<h2>这是h2元素</h2>
  </div>
)

// React18挂载渲染方式
const root = ReactDOM.createRoot(document.getElementById("root"))
root.render(jsx)
```

#### JSX 绑定样式

```jsx
class App extends React.Component {
  state = {
    domClass: 'red',
    flag: false,
  }
  render() {
    return (
      <div className="app">
        // 绑定变量
        <p className={this.state.domClass}>绑定变量</p>
        // 原来类名，添加新的
        <p className={'blue grren ' + (this.state.flag ? 'red' : '')}>
          原来类名，添加新的
        </p>
        // 绑定style
        <p
          style={{
            color: 'red',
            fontSize: '30px',
            backgroundColor: 'yellow',
          }}
        >
          jsx绑定style内联样式
        </p>
      </div>
    )
  }
}
```

#### JSX 中函数使用 this

```jsx
// 在jsx中函数使用this

// 使用bind绑定 函数中
<button onClick={this.handleClick.bind(this)}></button>

// 使用this绑定 constructor
constructor(){
  super()

  this.handleClick = this.handleClick.bind(this)
}

// 箭头函数 无参
handle = () => {
  console.log(this)
}

<button onClick={this.handle}></button>

// 有参
handle = (name) => {
  return () => {
    console.log(this, name)
  }
}

<button onClick={this.handle('react')}></button>


// jsx中使用箭头函数表达式 （推荐写法） 特别是在有参数的情况下，会变得特别方便

handle(name){
  console.log(this, name)
}
// 无参
<button onClick={() => { this.handle() }}></button>
// 有参
<button onClick={() => { this.handle('react') }}></button>
```

#### JSX 的条件渲染

```js
// 场景：loading效果
// 条件渲染：根据不同的条件来渲染不同的JSX结构
import React from 'react'
import ReactDOM from 'react-dom/client'

const isLoading = true

let loading = () => {
  return isLoading ? <div>Loading...</div> : <div>加载完成</div>
}

const root = ReactDOM.createRoot(document.getElementById('root'))
root.render(jsx)
```

#### JSX 列表渲染

```react
// 类型Vue中的v-for，只不过这里不再是作用于template，而是作用于JSX

// 如果需要渲染一组数据，我们应该使用数组的 map () 方法 filter(){ 只要返回值是一个数组的方法就可以 }  渲染列表的时候需要添加key属性，key属性的值要保证唯一


const lists = ['WEB', 'JavaScript', 'React', 'Vue']

const render = (
	<ul>
  	{ lists.map(item => <li key={ item }>{ item }</li>) }
  </ul>
)

const root = ReactDOM.createRoot(document.getElementById('root'))
root.render(root)
```

### React 组件

#### 实现类似 Vue 中的插槽

```jsx
// React中没有类似Vue的插槽，是因为React太灵活了，它不需要插槽，但是在开发中又需要插槽的类似功能，因此可以使用两种方式来实现插槽功能

// 方法一： 使用 props.children
// 缺点： 无法实现具名插槽的功能，而且传入的内容必须按照顺序来传，因为要按索引来取

// 父组件
<ChildrenCom>
	<span>左边的数据</span>
   <input type="text" placeholder="输入搜索内容" />
   <div className="right">搜索</div>
</ChildrenCom>

// 子组件 类组件
export default class NavBar extends Component {
  render() {
    return (
      <div className="nav">
        <div className="nav-left">
          {this.props.children[0] || "默认内容"}
        </div>
        <div className="nav-center">{this.props.children[1] || "默认内容"}</div>
        <div className="nav-right">{this.props.children[2] || "默认内容"}</div>
      </div>
    )
  }
}


// 方法二： 使用 props

// 父组件
<NavBar
  leftSlot={<div className="left">返回</div>}
  centerSlot={<input type="text" placeholder="输入搜索内容" />}
  rightSlot={<div className="right">搜索</div>}
/>

// 子组件
export default class NavBar extends Component {
  render() {
    const { leftSlot, centerSlot, rightSlot } = this.props
    return (
      <div className="nav">
        <div className="nav-left">{leftSlot || "默认内容"}</div>
        <div className="nav-center">{centerSlot || "默认内容"}</div>
        <div className="nav-right">{rightSlot || "默认内容"}</div>
      </div>
    )
  }
}

```

#### React 组件-函数创建

```react
// 约定1： 函数名称必须大写
// 约定2： 函数组件必须有返回值，表示当前组件的结构，如果返回值为null，表示不渲染内容

function Header(){
  return (
    <div className='header'>
      <h1>这是Hedaer组件</h1>
    </div>
  )
}
```

#### React 组件-类创建

```react
// 使用ES6语法的class进行创建
// 类组件必须提供 render 方法
// render方法必须有返回值
class Header extends React.Component {
  render(){
    return (
    	<div className="header">
      <h1>这是使用Class类创建的Header组件</h1>
      </div>
    )
  }
}
```

#### 状态组件

```react
// 创建组件的方式有两种，一种是使用函数，一种是使用类，使用函数是无状态的，使用类是有状态的

// 当组件需要保存状态，比如计数器案例时，我们需要来存储数据并监听和改变数据

class Dome extends React.Component {
  // 构造器
  constructor(){
    // 因为是extends继承而来，因为需要super调用父类方法
    super()
    // 用来保存状态
    this.state = {
      num: 0
    }
  }

  // 上面可采用下面方法进行替换
  state = {
    num: 0
  }


  // add 这里需要使用箭头函数改变this指向
  add = () => {
    // 使用
    // 改变state里面的数据时间，不能直接修改，需要使用setState()
    this.setState({ num: this.state.num + 1 })
  }

  // down
  down = () => {
    this.setState({
      num: this.state.num - 1
    })
  }

  // 渲染
  render(){
    return (
    	<div>
        <h2>num: {this.state.num}</h2>
        <button onClick={add}>+1</button>
        <button onClick={down}>-1</button>
      </div>
    )
  }
}
```

#### 组件通讯

```react
- 组件是封闭的，要接受外部数据应该通过props来实现
- props的作用：接收传递给组件的数据
- 传递数据：给组件标签添加属性

- 接收数据：函数组件通过 参数 props接收数据，类组件通过 this.props接收数据

- 可以给组件传递任意类型的数据
- props是只读属性，不能对值进行修改
- 注意：使用类组件时，如果写了构造函数，应该将props传递给super(),否则，无法在构造函数中获取到props，其他的地方是可以拿到的
```

#### 父组件传递数据给子组件

```react
// 和Vue的Props类似
// 父组件在调用子组件时，可以将要传递的数据直接写在子组件上

// 父组件
import Chilren from './component/01'

class Father extends React.Component {
  state = {
    title: '传递给子组件数据'
  }

  render(){
    return (
      <div className="father">
        <h2>这是父组件</h2>
        <Chilren title={this.state.title} />
      </div>
    )
  }
}



// 子组件 类组件
import React from 'react'
class Chilren extends React.Component {
  constructor(props){
    super(props)
  }

  render(){
    return (
      <div className="father">
        <h3>接受父组件传过来的数据</h3>
        {this.props.title}
      </div>
    )
  }
}

export default Chilren

// 子组件 函数组件
function Children(props){
  return (
  	<div className="father">
      <h3>接受父组件传过来的数据</h3>
      {props.title}
    </div>
  )
}

```

#### 子组件传递数据给父组件

```react
// 由于React是单向数据流，只能子组件给父子间，并且父子间的数据改变，子组件的数据响应式变化，但是子组件无法直接修改父组件传递的值。因为所有 React 组件都必须像纯函数一样保护它们的 props 不被更改
// 但是父组件可以设置一个回调函数，并且传递给子组件，当组件想要改变父组件数据时，调用该函数即可，类似于Vue的Emits

// 父组件
import Chilren from './component/01'

class Father extends React.Component {
  state = {
    num: 0
  }
  // 传给子组件，用来子组件调用改变父组件的值 num: 接受子组件传递的值
  changeNum(num){
    console.log('子组件传递的值为：', num)
    // 更新父组件数据
    this.setState({ num })
  }

  render(){
    return (
      <div className="father">
        <h2>这是父组件num: {this.state.num}</h2>
        <Chilren title={this.state.num} changeNum={this.changeNum} />
      </div>
    )
  }
}


// 子组件
class Children extends React.Component {

  handleClick = (num) => {
    return () => {
      this.props.changeNum(10)
    }
  }

  render(){
    return (
      <div className="father">
        <h2>这是父组件num: {this.state.num}</h2>
        <button onClick={handleClick(10)}>改变父组件的num值</button>
      </div>
    )
  }
}

```

#### 兄弟组件数据传递

```react
// 类似于Vue的兄弟间数据传递，需要一个公共的父组件，然后进行传递，先传递给父组件，然后再通知另外的子组件

// 兄弟组件传递的方式有很多，如果redux，event bus（事件总线），公共父级传递

// 父组件
import BoderComOne from './component/01'
import BoderComTwo from './component/02'

class Parent extends React.Component {
  state = {
    boderNum: 20
  }

  changBorderNum = (borderNum) => {
    this.setState({ borderNum })
  }

  render(){
    return (
      <h2>测试兄弟组件传值</h2>
    	<BoderComOne boderNum={this.state.borderNum} change={changBorderNum} />
      <BoderComTwo boderNum={this.state.borderNum} change={changBorderNum} />
    )
  }
}

// 兄弟一组件
class BorderComOne extends React.Component {
  handleClick = (borderNum) => {
    return () => {
      this.props.change(borderNum)
    }
  }

  render(){
    return (
      <h3>这是组件一，值为: {this.props.borderNum}</h3>
      <button onClick="handleClick(30)">改变值为30</button>
    )
  }
}



// 兄弟二组件
class BorderComTwo extends React.Component {
  handleClick = (borderNum) => {
    return () => {
      this.props.change(borderNum)
    }
  }

  render(){
    return (
      <h3>这是组件二，值为: {this.props.borderNum}</h3>
      <button onClick="handleClick(50)">改变值为50</button>
    )
  }
}


```

#### 事件总线 EventBus

```tsx
// 事件总线 用来用来解决兄弟组件之间传值问题
// 安装
npm install events -D

// 声明
// event.tsx
import { EventEmitter } from 'events'

// 事件总线
const events = new EventEmitter()
export default events



// 兄弟组件1
// index.tsx
import React, { useState } from 'react'
import events from './event'

const App: React.FC = () => {
  const [num, setNum] = useState<number>(0)
  const handleClick = () => {
    // 发送事件， 第一个为事件名，后面为参数
    event.emit('tiggerEvent', num + 1)
    setNum(num + 1)
  }

  return (
    <div className="emits">
      <h2>事件总线——发送 num: {num}</h2>
      <button onClick={handleClick}>发送事件</button>
    </div>
  )
}



// 兄弟组件2
// index2.tsx
import React, { useEffect, useState } from "react"
import event from './events'

const App2: React.FC = () => {
  const [num, setNum] = useState<number>(0)
  // 处理事件
  const handleEventChange = (num: number) => {
    console.log('接受到了消息', setNum(num))
  }

  // 监听改变
  useEffect(() => {
    // 监听的事件名，监听之后的处理方案
    events.addListener('tiggerEvent', handleEventChange)
  })

  return (
    <div className="app2">
      <h2>事件总线——接受 num: {num}</h2>
    </div>
  )
}

```

#### 爷孙组件数据传递

```tsx
// Context提供了两个组件：Provider 和 Consumer
// 类似于Vue的Provide 和 Inject

// 创建需要传递的数据
const defaultValue = {
  userName: '新新小朋友'
}

// 创建Context对象
const appContext = React.createContext(defaultValue)

// Provider组件： 提供数据 包裹需要传递的数据组件
<appContext.Provider value={ defaultValue }>
   <Child />
</appContext.Provider>
// Consumer组件： 接受数据
<appContext.Consumer>
	{
    (value) => (
      <div className="chilren">
        <h3>传递的数据为： {value.name}</h3>
      </div>
    )
  }
</appContext.Consumer>

// 使用
// 父组件
// 创建需要传递的内容
const defaultValue = {
  userName: '新新小朋友'
}
// 创建Context上下文对象, 并使用export导出
export const appContext = React.createContext(defaultValue)

const App: React.FC = () => {
  return (
    <div className="app">
      <h3>这是父组件</h3>
      <appContext.Provider value={ defaultValue }>
        <Child />
      </appContext.Provider>
    </div>
  )
}


// 孙组件
// 导入上级组件导出的context
import { appContext } from '../../app'

const Child: React.FC = () => {
  return (
  	<div className="child">
    	<h3>这是孙组件</h3>
      <appContext.Consumer>
      	{
          (value) => (
            <h3>userName: { value.userName }</h3>
          )
        }
      </appContext.Consumer>
    </div>
 	)
}
```

#### 组件优化

```tsx
// react中组件肯定是存在嵌套关系：当我们使用Component的时候，父组件的state或者props更新的时候，无论子组件的state、props是否更新，都会触发子组件的更新，会形成很多没必要的render，浪费很多性能

shouldComponentUpdate方法：

// nextProps: 最新的Props
// nextState: 最新的State
shouldComponentUpdate(nextProps, nextState){
  // 如果最新的props或state不一致，则更新页面，如果一致 则不更新页面
  if(nextProps.name !== this.props.name){
    // 更新
    return true
  }
  // 不更新
  return false
}

react里边：PureComponent（实现了shouldComponentUpdate）通过props和state的浅比较

浅比较是react源码中的内置的一个函数，它替代了shouldComponentUpdate的工作。它只比较外层数据结构，只要外层相同，则认为没有变化，不会深层次去比较数据

原理：当组件更新时，如果组件的props和state都没发生改变的时候，render方法就不会被触发，省去了virtual DOM的生成和对比的过程，达到提升性能的目的

PureComponent提供了一个具有浅比较的shouldComponentUpdate方法，PureComponent和Component基本上相同的

PureComponent真正起作用的时候，只是在一些纯展示组件上

PureComponent存在的问题：

1. 不可以在在函数式组件上使用的
2. 进行的是浅比较，深层次的数据不一致，有可能会导致页面得不到更新
3. 不太适合使用在含有多层嵌套对象的state和prop中的


// 函数组件优化 使用memo

const MemoHeader = memo(function Header(){
  console.log('Header调用')
  return <h2>我是Header调用</h2>
})
```

### setState

#### setState 使用

```jsx
// 数据合并
this.state = {
  name: 'WEB架构师',
  age: 22
}

this.setState({
  name: 'React工程师'
})

// 问题：使用setState后，会不会将新的数据覆盖旧的数据，也就是说age将不存在，答案是否，使用setState并不会覆盖，而是数据的合并
// 原理：
Object.assign({}, this.state,  { name: 'React工程师' })
// 解释：
后两个对象将赋值给前面的空对象，因此name的值会发送覆盖更新，而age的值将不会变化
```

#### setState 的同步异步

```tsx
异步更新，同步执行

// 为什么使用setState
 因为我们修改了状态state的时候，希望react根据最新的state来重新渲染界面。直接修改的方式，react并不会知道状态发生了改变，react没有实现类似于vue2 中Object.defineProperty或者是vue3 proxy的方式来监听数据的改变，必须通过setState来告知react状态的改变。setState是继承自Component。当我们调用setState的时候，会重新执行render方法。

// setState 是异步的
1. 生命周期里面是异步的
2. 合成事件是异步的 （react中是合成事件机制，react并不是直接将事件绑定在dom上面，采用的是事件冒泡的形式冒泡到document上，然后react将这个事件封装给正式的函数去处理）

// 获取异步更新之后的数据
// 方法一：setState的第二个参数是回调函数，可以获取更新之后的最新值
this.setState({
  message: 'Hello React'
}, () => {
  console.log(this.state.message)
})

//方法二：setState更新之后会触发render函数，之后会调用componentDidUpdate()生命周期
componentDidUpdate(){
  console.log('获取最新的状态', this.state.message)
}



// setState 是同步的
1. 在setTimeout是同步的
2. 在原生事件里面，setState也是同步的

1.在setTimeout是同步的
changeText(){
  setTimeout(() => {
    this.setStaet({
      message: 'React'
    })
    console.log(this.state.message)
  }, 0)
}

2. 在原生事件里面，setState也是同步的
render(){
  return (
  	<div>
    	<button id='btn'></button>
    </div>
  )
}

componentDidMount(){
  document.querySelector('#btn').addEventListener('click', () => {
    this.setStaet({
      message: 'React'
    })
    console.log(this.state.message)
  }, false)
}





// 总结
setState并不是由异步代码实现，执行的过程和顺序都是同步的，只是生命周期和合成事件调用顺序是在更新之前，导致生命周期和合成事件没有办法立即拿到更新后的值，形成了所谓的异步

```

### 获取元素

#### 获取元素 ref

```tsx
// 获取dom元素的ref
// 有些时候我们需要操作DOM元素，因此可以设置ref来获取DOM元素，这和Vue类似

// 引入
import React, { createRef, useEffect } from 'react'

// 使用
const Refs: React.FC = () => {
  // 声明输入框的ref元素
  const inputRef = createRef<HTMLInputElement>()
  // 初始化生命周期
  useEffect(() => {
    // 获取焦点
    inputRef.current?.focus()
  }, [])
  return (
    <div className="ref">
    	<input type="text" ref={inputRef} />
    </div>
  )
}

缺点： 每次重新渲染都会重新创建一个ref对象

// 补充
// 可以使用createRef来获取类组件，但是不能获取函数组件，后续可以使用Hooks

class App extends Component{
  constructor(props){
    super(props)

    this.childRef = creatRef()
  }

  render(){
    return (
      <div className='app'>
      	// 子组件
        <Child ref={this.childRef} />
        <button onClick={() => this.handleClick()}>点击按钮调用子组件的方法</button>
      </div>
    )
  }

  handleClick(){
    this.childRef.current.add()
  }

}
```

#### 使用`ref`获取子组件内的元素

```jsx
// 使用 React.forwardRef()包裹 可以向子组件传入ref对象，以便获取子组件的元素

// 子元素
/**
 * props 父组件传递的props
 * ref 父组件传递的ref对象
 */
const ChildrenCom = React.forwardRef((props, ref) => {
  return (
    <div>
      <input type="text" ref={ref} />
    </div>
  )
})

// 父组件
export default function Home(){
  const inputRef = useRef()
  const [info, dispatch] = useReducer((state, action) => {
    switch(action.type){
      default:
        return state
    }
  }, {
    name: '新新'
  })

  return (
  	<div>
    	<button onClick={() => console.log(inputRef)}>
        获取ChilreneCom组件的搜索框
      </button>
      <MoreCom info={info} ref={inputRef} />
    </div>
  )
}


// 但是上面的做法不是很推荐，因为使用到了整个ref对象，当我们想要获取某些子组件向外暴漏的事件方法，可以使用useImperativeHandle

// 子组件
/**
 * props 父组件传递的props
 * ref 父组件传递的ref对象
 */
const MoreCom = React.forwardRef((props, ref) => {
  const inputRef = useRef()

  // 接受两个参数，第一个参数是接受父组件传递过来的props，并且将后面第二个参数返回的对象绑定到ref.current，也就是说将父组件用到的ref操作这里统一导出
  useImperativeHandle(ref, () => ({
    focus: () => {
      console.log("执行这边")
      inputRef.current.focus()
    },
    shows() {
      console.log("我是show函数")
    },
  }))

  return (
    <div>
      <input type="text" ref={inputRef} />
    </div>
  )
})


// 父组件
export default function Home(){
   const comRef = useRef()

   <button onClick={() => comRef.current.focus()}>
  	打印子组件暴漏的focus方法
	</button>
	<button onClick={() => comRef.current.shows()}>
  	打印子组件暴漏的shows方法
	</button>
   <MoreCom info={info} ref={comRef} />
}

```

![使用`ref`获取子组件内的元素](../../image/前端框架/react/01.jpg)

#### 受控组件和非受控组件

```jsx
// 受控组件 和 非受控组件
// 就是受到React的状态(state)约束和控制的组件。同理，非受控组件就是不受到React状态（state）越苏的组件，要获取它们的信息，可以采用ref的方式来获取

// 受控组件
export default class App extends PureComponent {
  constructor(props) {
    super(props)

    this.state = {
      value: '',
      pass: '',
      eat: '',
    }
  }
  render() {
    return (
      <div>
        <input
          type="text"
          placeholder="输入用户名"
          value={this.state.value}
          name="value"
          onChange={e => this.handleChange(e)}
        />
        <h4>输入的内容： {this.state.value}</h4>
        <br />
        <input
          type="text"
          placeholder="输入密码"
          value={this.state.pass}
          name="pass"
          onChange={e => this.handleChange(e)}
        />
        <h4>输入的密码： {this.state.pass}</h4>
        <select
          value={this.state.eat}
          name="eat"
          onChange={e => this.handleChange(e)}
        >
          <option value="apple">苹果</option>
          <option value="banana">香蕉</option>
          <option value="lizhi">荔枝</option>
          <option value="huoguo">火锅</option>
        </select>
      </div>
    )
  }

  // 将每个输入框添加一个name属性，其name属性的值和保存的变量信息一致，这样可以封装一个公共的函数
  handleChange(e) {
    this.setState({
      [e.target.name]: e.target.value,
    })
  }
}
```

### 高阶组件

```jsx
import React, { PureComponent } from 'react'

// 组件一
class Home extends PureComponent {
  render() {
    return (
      <div className="home">
        <h2>Home组件</h2>
        <h3>name: {this.props.name}</h3>
        <h3>pass: {this.props.pass}</h3>
        <h3>address: {this.props.address}</h3>
      </div>
    )
  }
}

// 组件二
class About extends PureComponent {
  render() {
    return (
      <div className="about">
        <h2>About组件</h2>
        <h3>name: {this.props.name}</h3>
        <h3>pass: {this.props.pass}</h3>
        <h3>address: {this.props.address}</h3>
      </div>
    )
  }
}

// 高阶组件 拦截原组件，处理之后返回新组件
function NewCompon(OldCompon) {
  // 函数组件方式
  // return (props) => <OldCompon {...props} address={"大同"} />

  // 类组件方式
  return class extends PureComponent {
    render() {
      return <OldCompon {...this.props} address={'大同'} />
    }
  }
}

const NewComponHome = NewCompon(Home)
const NewComponAbout = NewCompon(About)

export default class App extends PureComponent {
  render() {
    return (
      <div>
        <NewComponHome name={'web'} pass={22} />
        <NewComponAbout name={'React'} pass={20} />
      </div>
    )
  }
}
```

#### 组件的生命周期

**注意**： 只有类组件才有声明周期，函数组件没有生命周期（Hooks 除外）

render 函数里面不能执行`setState`，不然会造成递归渲染，导致栈溢出

##### 创建时（挂载阶段）

执行时机：组件创建时（页面加载时）

执行顺序：`constructor()` ---> `render()` ---> `conponentDidMount`

conponentDidMount 作用： 获取 dom 或者发送 ajax 请求

##### 触发时机

`constructor`: 创建组件时，最先指向

`render`：每次组件渲染都会触发

`componentDidMount`：组件挂载后（完成 DOM 渲染）

##### 更新时

`componentDidUpdate()`

执行时机：`setState()、 forceUpdate()、 组件接收到新的props`

说明：以上三者任意一种变化，组件就会重新渲染

执行顺序：render() ---> conponentDidMount

##### 卸载时

`componentWillUnmount()`

执行时机：组件从页面中消失

作用：用来做清理操作

## Hooks

> Hooks 是什么：
> 可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。
> 之前组件划分为状态组件(class)和无状态组件(function), 因为函数是 js 的第一公民，因此 Hooks 的出现使得函数组件不再是无状态组件，而是和 class 组件一样

> 注意： 只能在函数组件中使用 Hooks，在类组件中不能使用 Hooks

### `useState`

```tsx
// 用来存储当前组件的状态，类似于类组件中的this.state = {} 或 state = {}

// API
const [变量, 更改变量的函数(以set开头，变量为主体，比如:set变量)] = useState<变量类型>(变量值)

// 用法
// 1 引入
improt React, { useState } from 'react'

const HooksCom: React.FC = () => {
  // 2 定义变量和改变变量值的方法
  const [count, setCount] = useState<number>(0)

  // 改变count
  const add = (): void => {
    setCount(count + 1)
  }
  const down = (): void = {
    setCount(count - 1)
  }

  // 3 使用
  return (
    <div className="hooks">
      <h2>使用State Hooks</h2>
      当前计数： { count }
      <button onClick={}>+1</button>
      <button onClick={}>-1</button>
    </div>
  )
}
```

### `useEffect`

```tsx
// 你可以把 useEffect Hook 看做 componentDidMount，componentDidUpdate 和 componentWillUnmount 这三个函数的组合。

useEffect(() => {})  =====> componentDidMount 和 componentDidUpdate 组合
useEffect(() => {}, [])  ======> componentDidMount
useEffect(() => {}, [变量1, 变量2])  ======> componentDidUpdate

// 清除定时器 卸载
useEffect(() => {
    const timer = setInterval(() => {
      console.log("定时器开始执行")
    }, 1000)
    return () => {
    	clearInterval(timer)
      console.log('组件卸载了')
    }
  }, [])
```

### `useContext`

```tsx
接收一个 context 对象（React.createContext 的返回值）并返回该 context 的当前值。当前的 context 值由上层组件中距离当前组件最近的 <MyContext.Provider> 的 value prop 决定
别忘记 useContext 的参数必须是 context 对象本身：
正确： useContext(MyContext)
错误： useContext(MyContext.Consumer)
错误： useContext(MyContext.Provider)

// 也就是说 它简化了 在孙组件接受爷组件传值时的嵌套关系， 类似于 Context

// 使用
import React, { useContext } from 'react'
// 导入上级组件导出的context
import { appContext } from '../../app'

// 使用
const Child: React.FC = () => {
  // 使用useContext
  const value = useContext(appContext)
  return (
    <div className="child">
      <h3>上级组件传递的内容：{ value.userName }</h3>
    </div>
  )
}
```

### `useRef`

```tsx
// useRef和createRef类似，但是useRef性能更高，更推荐使用

createRef: 	每一次重新渲染的时候都会创建一个新的ref对象
useRef: 第一次渲染的时候创建一个对象之后，之后重新渲染的时候，如果发现这个对象已经创建过了，不再创建新的ref对象

// 引入
import React, { useRef, useEffect } from 'react'

// 使用
const App = () => {
  // 获取DOM元素
  const inputRef = useRef()
  // 初始化生命周期
  useEffect(() => {
    inputRef.current.focus()
  }, [])
  return (
    <div className="input">
    	<input type="text" ref={inputRef} />
    </div>
  )
}
```

### `useReducer`

```jsx
// 在hook中，如果使用useState处理过于复杂的数据，可以使用useReducer,并不是来替代redux

引入useReducer以后，useReducer接收一个reducer函数作为参数，reducer函数接收两个参数，一个参数state，另外一个参数是action。然后返回一个状态state，dispath，state是返回状态中的值，dispath是一个可以发布事件来更新state的。

const [state, dispatch] = ((state, action) => { return 改变后的state }, 初始值)

官方提供的两种state管理：useState ，useReducer

在useReducer传入了reducer函数根据action来更新state，如果action为add则增加

在button中调用dispath来发布了 add事件，发布add事件会在reducer函数根据这个类型对state进行对象的操作，然后更新state

useReducer：

使用场景：

1. 如果说你的state是一个数组或者是对象
2. 如果你的state变化很复杂，经常一个操作需要修改很多的state
3. 如果你的应用比较大。希望UI和业务能够分开维护的时候

理解为阉割版：redux

useState好像用useReducer去实现

// 引入
import React, { useReducer } from 'react'

// 使用
const App = () => {
  // 使用useReducer
  const [state, dispatch] = useReducer((state, action) => {
    switch(action) {
      case "add":
        return state + 1
      case: "down":
        return state - 1
      default:
        return state
    }
  }, 10)

  return (
    <div className="reducer">
      <h2>useReducer使用</h2>
      <h2>{ state }</h2>
      <button onClick={() => dispatch('add')}>+1</button>
      <button onClick={() => dispatch("down")></button>
    </div>
  )
}


// 提取出来使用
import React, { useReducer } from "react"

const renducer = (state, action) => {
  switch (action) {
    case "add":
      console.log("点击了加", state.num)
      return { num: state.num + 1 }
    case "down":
      return { num: state.num - 1 }
    default:
      return { num: 1000 }
  }
}
const init = {
  num: 0,
}

const App = () => {
  const [state, dispatch] = useReducer(renducer, init)

  return (
    <div className="reducer">
      <h2>useReducer使用</h2>
      <h3>{state.num}</h3>
      <button onClick={() => dispatch("add")}>+1</button>
      <button onClick={() => dispatch("down")}>-1</button>
    </div>
  )
}

export default App
```

### `useCallback`

```jsx
useCallback实际目的是为了进行性能的优化，
useCallback会返回一个函数的 memoized(记忆的) 值，在依赖不变的情况下，多次定义的时候，返回的值是相同的

useCallback是对函数做优化的，使用useCallback的目的是不希望子组件进行多次渲染，并不是为了函数进行缓存

使用场景：就是父组件给子组件传递一个函数，但父组件的数据更新时，函数会重新定义，因此子组件也会被重新渲染，但是当子组件并没有发生变化，没必要进行渲染的时候，这样就会浪费性能，因此我们需要对传递给子组件的函数进行优化，使得子组件不会被多次渲染

// 例子：子组件接受父组件传递的count和改变count的方法，当与父组件的message变量没有关系，因此当父组件改变message的值时，我们不希望子组件一起更新
// 父组件
const App = memo(() => {
  const [count, setCount] = useState(0)
  const [message, setMessage] = useState('hha')

  // 这样将函数依赖设置为 count 只有当count改变时，这个函数才会改变，子组件才会被重新渲染
  const add = useCallback(() => {
    setCount(count + 1)
  }, [count])

  return <div>
    <h2>父组件的值：{count}</h2>

    {/* 子组件 */}
    <Com1 count={count} handleAdd={add} />
  </div>
})

// 子组件
interface PropsTypes {
  count: number
  handleAdd: () => void
}
const Num = memo((props) => {
  const { count, handleAdd } = props

  return (
    <div>
    	父级传递的值： {count}
    </div>
  )
})

```

### `useMemo`

```jsx
useMemo的目的也是为了进行性能的优化， 它优化的是返回值，而不是函数。
一般也可用与做计算属性

// 用来模糊搜索
const Home = memo(() => {
  const [originUrl, setOriginUrl] = useState<Array<any>>([])
  const [searchText, setSearchText] = useState<string>('')

  useEffect(() => {
    setOriginUrl([
      {
        key: '/',
        label: '首页',
        icon: '<></>',
        module_parent: null
      },
      {
        key: '/admin/module',
        label: '模块管理',
        icon: '<></>',
        module_parent: 'admin'
      }
    ])
  }, [])

  const renderUrl = useMemo(() => {
    if (!searchText) return originUrl
    return originUrl.filter((item) => item.label.includes(searchText))
  }, [originUrl, searchText])

  return (
    <>
      <input
        type="text"
        value={searchText}
        onInput={(e) => setSearchText(e.currentTarget.value)}
      />
      <ul>
        {renderUrl.map((item) => (
          <li key={item.key}>{item.label}</li>
        ))}
      </ul>
    </>
  )
})






// useCallback 和 useMemo 的区别

// useCallback 对函数进行优化
const fun1 = useCallback(() => {
  conosle.log('执行了fun1函数')
}, [count])

// useMemo 对函数返回值进行优化
const fun2 = useMemo(() => {
  return () => {
    console.log('执行了fun2函数')
  }
}, [count])
```

### `useImperativeHandle`

```jsx
// 当我们父组件想要使用子组件里面的元素和方法时，直接获取整个子组件并不是最好的选择，而是在子组件要使用 useImperativeHandle 统一暴漏出去

// 子组件
/**
 * props 父组件传递的props
 * ref 父组件传递的ref对象
 */
const MoreCom = React.forwardRef((props, ref) => {
  const inputRef = useRef()

  // 接受两个参数，第一个参数是接受父组件传递过来的props，并且将后面第二个参数返回的对象绑定到ref.current，也就是说将父组件用到的ref操作这里统一导出
  useImperativeHandle(ref, () => ({
    focus: () => {
      console.log("执行这边")
      inputRef.current.focus()
    },
    shows() {
      console.log("我是show函数")
    },
  }))

  return (
    <div>
      <input type="text" ref={inputRef} />
    </div>
  )
})


// 父组件
export default function Home(){
   const comRef = useRef()

   <button onClick={() => comRef.current.focus()}>
  	打印子组件暴漏的focus方法
	</button>
	<button onClick={() => comRef.current.shows()}>
  	打印子组件暴漏的shows方法
	</button>
   <MoreCom info={info} ref={comRef} />
}
```

### `useLayoutEffect`

```jsx
useLayoutEffect看起来和useEffect非常相似，事实上他们也只有一个区别而已
useEffect会在渲染的内容更新到DOM 后 执行，不会阻塞DOM的更新
useLayoutEffect会在渲染的内容更新到DOM 之前 执行，会阻塞DOM的更新
```

## 路由

### react-router 使用

```jsx
// 安装
npm install react-router-dom --save

// 使用
// 根文件 index.js
import { BrowserRouter } from 'react-router-dom'

// 将整个App组件使用 BrowserRouter 来包裹
const root = ReactDOM.createRoot(document.getElementById("root"))
root.render(
  <BrowserRouter>
    <Provider store={store}>
      <App />
    </Provider>
  </BrowserRouter>
)


// App.js
import { Link, NavLink, Route, Routes } from "react-router-dom"

import Home from "./views/home"
import About from "./views/about"

function App() {
  return (
    <div className="App">
      // 类似于vue的 vue-link
      <Link to="/">首页</Link>
      <Link to="/about">关于</Link>

      // NavLink 当前被选中的路由会加上active类，因此可以在css里进行设置样式
      <NavLink to="/">首页</NavLink>
      <NavLink to="/about">关于</NavLink>


      // Routes是一组路由匹配组
      <Routes>
        // Route是匹配的每一个路由，path是路径 element是组件信息
        <Route path="/" element={<Home />}></Route>
        <Route path="/about" element={<About />}></Route>
        // 配置404页面
        <Route path="*" element={<Notfound />} />
      </Routes>
    </div>
  )
}

export default App

```

### react-router 二级路由

```jsx
// 比如在关于页面 存在其他详细关于页面 只需要在 <Route >下面继续存放<Route >路由即可

// index 代表默认匹配的路由， 子路由路径可以使用相对路径 company 也可以使用绝对路径 /about/company
;<Route path="/about" element={<About />}>
  <Route index element={<Default />}></Route>
  <Route path="company" element={<Company />}></Route>
  <Route path="we" element={<We />}></Route>
</Route>

// 如果显示子路由的内容 在About组件内 使用 Outlet展示即可
import { Outlet } from 'react-router-dom'

// 需要展示子路由的地方
;<Outlet />
```

### react-router 分包书写

```jsx
// 在src下新建router/index.js木匠

import { useRoutes } from 'react-router-dom'

// 前台布局
import ClientLayout from '@/layout/client'

export const GetRoutes = () => {
  const routes = useRoutes([
    // 前台路由
    {
      path: '/',
      element: <ClientLayout />,
      children: [
        { path: '', element: <div>发现音乐</div> },
        { path: 'my', element: <div>我的音乐</div> },
        { path: 'friend', element: <div>关乎</div> },
      ],
    },
    // 后台路由
    {
      path: '/admin',
      element: <div>后台页面</div>,
      children: [
        {
          path: '',
          element: (
            <div>
              <h2>后台首页</h2>
            </div>
          ),
        },
      ],
    },
    // 404
    { path: '*', element: <div>404</div> },
  ])
  return routes
}

// 根 index.js 中
import React from 'react'
import ReactDOM from 'react-dom/client'

import '@/assets/css/reset.css'

import { BrowserRouter } from 'react-router-dom'

import { GetRoutes } from '@/router'

const root = ReactDOM.createRoot(document.getElementById('root'))
root.render(
  <BrowserRouter>
    <GetRoutes />
  </BrowserRouter>
)
```

### react-router 懒加载

```jsx
import React, { lazy } from 'react'

// 路由懒加载
const lazyLoad = path => {
  const Comp = lazy(() => import(`@/views/client${path}`))
  return (
    <React.Suspense fallback={<>加载中...</>}>
      <Comp />
    </React.Suspense>
  )
}

export const ClientRoutes = [
  // 发现音乐
  {
    path: '',
    element: lazyLoad('/my'),
  },
  // 我的音乐
  {
    path: 'my',
    element: lazyLoad('/my'),
  },
]
```

### 路由跳转

```jsx
import React from 'react'
import { useNavigate } from 'react-router-dom'

export default function Login() {
  const navigate = useNavigate()

  const handleClick = () => {
    navigate('/')
  }

  return (
    <div>
      <h2>Login</h2>
      <button onClick={() => handleClick()}>登录</button>
    </div>
  )
}
```

### 动态路由

```jsx
// 声明需要加载的动态路由组件  访问方式 /me/details/1
<Route path="me" element={<Me />}>
  <Route path="details/:id" element={<Details />} />
</Route>

// 访问动态路由
export default function Me() {
  return (
    <div>
      <h2>用户列表</h2>
      <ul>
        <li>
          <NavLink to="/me/details/1">用户1</NavLink>
        </li>
        <li>
          <NavLink to="/me/details/2">用户2</NavLink>
        </li>
      </ul>
      {/* 加载动态路由模块 */}
      <Outlet />
    </div>
  )
}

// 获取动态路由的传参
import React from "react"
import { useParams } from "react-router-dom"

export default function Details() {
  const params = useParams()

  return (
    <div>
      <h3>当前查看的是 用户的详细资料: {params.id}</h3>
    </div>
  )
}

```

## Redux

### Redux 使用

```jsx
// 安装
npm install redux -S
yarn add redux

state 存储全局的状态
reducer 在state和action中建立连接
action 派发的状态 state状态即将修改后的值
// 组件使用action 派发action
store.dispatch(action1)
// 订阅store的修改，更新render
store.subscribe(() => {
  // 获取存储在store的值
  this.setState({
    num: store.getState().info
  }}
})

// 在store下新建四个文件
├─store
│ └─ index store的声明和导出
│ └─ constant 存放action的名称
│ └─ reducer 将state和action联系在一起
│ └─ actionCreator 存放action


// index.js
import * as redux from "redux"

import reducer from "./reducer"

const store = redux.createStore(reducer)

export default store


// constant.js
export const ADD_NUMBER = "ADD_NUMBER"
export const SUB_NUMBER = "SUB_NUMBER"


// reducer.js
// 注意：reducer必须要是一个纯函数，用来连接state和action
import { ADD_NUMBER, SUB_NUMBER } from "./constant"

const defaultValue = {
  author: "新新小朋友",
  num: 10,
}

/**
 *
 * @param state 当前state的值，第一次赋值为初始化state
 * @param action 需要派发的action
 * @returns 更新之后的state
 */
const reducer = (state = defaultValue, action) => {
  switch (action.type) {
    case ADD_NUMBER:
      return { ...state, num: state.num + action.num }
    case SUB_NUMBER:
      return { ...state, num: state.num - 1 }
    default:
      return state
  }
}

export default reducer


// actionCreateors.js
import { ADD_NUMBER, SUB_NUMBER } from "./constant"

// 增加
export const addAction = (num) => ({ type: ADD_NUMBER, num })

// 减少
export const subAction = () => ({ type: SUB_NUMBER })



// 封装 connect函数
import React, { PureComponent } from "react"

import store from "../store1"

/**
 *
 * @param mapStateToProps 组件需要使用的state
 * @param mapDispatchToProps 组件需要调用的dispatch
 */
export function connect(mapStateToProps, mapDispatchToProps) {
  return function enhanceHOC(WrappedComponent) {
    return class extends PureComponent {
      constructor(props) {
        super(props)

        this.state = {
          storeState: mapStateToProps(store.getState()),
        }
      }

      componentDidMount() {
        this.unSubscribe = store.subscribe(() => {
          this.setState({
            storeState: mapStateToProps(store.getState()),
          })
        })
      }

      componentWillUnmount() {
        this.unSubscribe()
      }

      render() {
        return (
          <WrappedComponent
            {...this.props}
            {...mapStateToProps(store.getState())}
            {...mapDispatchToProps(store.dispatch)}
          />
        )
      }
    }
  }
}


// 使用
import React from "react"

import { Button } from "antd"

import { connect } from "../../../utils/connect"

import { addAction, subAction } from "../../../store1/actionCreateors"

function Nums(props) {
  return (
    <div>
      <h2>这是Nums组件， 使用store1</h2>
      <h3>当前num: {props.num}</h3>
      <Button onClick={() => props.addAction(3)}> +3 </Button>
      <Button onClick={() => props.subAction(1)}> -1 </Button>
    </div>
  )
}

// 该组件所使用的state的映射
const mapStateToProps = (state) => ({
  num: state.num,
})

// 该组件所使用的dispatch映射
const mapDispatchToProps = (dispatch) => ({
  addAction(num) {
    dispatch(addAction(num))
  },
  subAction(num) {
    dispatch(subAction(num))
  },
})

export default connect(mapStateToProps, mapDispatchToProps)(Nums)

```

####

### react-redux 使用

```jsx
// 安装
# NPM
npm install react-redux -S
# Yarn
yarn add react-redux

// index.js 包裹 App 组件
import store from '@/store'
import { Provider } from 'react-redux'

const root = ReactDOM.createRoot(document.getElementById("root"))
root.render(
  <Provider store={store}>
    <App />
  </Provider>
)


// 目标组件中
import React from "react"

import { connect } from "react-redux"
import { addAction, subAction } from "@/store1/actionCreateors"

import { Button } from "antd"

function Nums(props) {
  return (
    <div>
      <h2>这是Nums组件， 使用store1</h2>
      <h3>当前num: {props.num}</h3>
      <Button onClick={() => props.handleAddAction(3)}> +3 </Button>
      <Button onClick={() => props.handleSubAction(1)}> -1 </Button>
    </div>
  )
}

// 该组件所使用的state的映射 返回一个对象
const mapStateToProps = (state) => ({
  num: state.num,
})

// 该组件所使用的dispatch映射 返回一个对象
const mapDispatchToProps = (dispath) => ({
  handleAddAction(num) {
    dispath(addAction(num))
  },
  handleSubAction(num) {
    dispath(subAction(num))
  },
})

export default connect(mapStateToProps, mapDispatchToProps)(Nums)


```

### Redux 中异步操作

```jsx
使用中间件 redux-thunk
默认情况下的dispatch(action), action需要是一个JavaScript的对象
redux-chunk可以让dispatch(action函数)，action可以是一个函数，该函数会被调用，并且会传给这个函数一个dispatch函数

注意： 该中间件是将原来在生命周起（componentDidMount）中的发送异步请求放到了action中，因为action可以传入函数，并且拿到dispatch

// 安装
npm install redux-thunk -S

// 第一步，在store/index中应用中间件
import * as redux from "redux"
import thunkMiddleware from "redux-thunk"

import reducer from "./reducer"

// 应用中间件
const storeenhancer = redux.applyMiddleware(thunkMiddleware)

const store = redux.createStore(reducer, storeenhancer)

export default store


// 第二走，在store/action中添加对应网络请求的action
// redux-thunk 中 定义的action函数
/**
 *
 * @param dispatch 获取当前dispatch对象，调用其他action
 * @param getState 获取上一次store的数据，可以用来分页
 */
export const getHomeMultidataAction = () => {
  return (dispatch, getState) => {
    axios.get("http://123.207.32.32:8000/home/multidata").then(({ data }) => {
      // 调用上面的更改数据的action
      dispatch(updateBanners(data.banner.list))
  	})
  }
}


// 第三步，在组件中使用
将原来在声明周期发送异步请求直接更换为调用action

componentDidMount() {
  // axios.get("http://123.207.32.32:8000/home/multidata").then(({ data }) => {
  //   this.props.handleUpdate(data.banner.list)
  // })
  this.props.getShopLists()
}


const mapStateToProps = (state) => ({
  banners: state.banners,
})

const mapDispatchToProps = (dispatch) => ({
  handleUpdate(banners) {
    dispatch(updateBanners(banners))
  },
  handleChangeNum(num) {
    dispatch(addAction(num))
  },
  getShopLists() {
    // 这里的 getHomeMultidataAction 加括号
    dispatch(getHomeMultidataAction())
  },
})

export default connect(mapStateToProps, mapDispatchToProps)(Shop)

```

### Redux 调试工具的使用

```jsx
// Redux调试工具并不像Vuex的调试工具一样默认开启，需要手动配置来完成Redux调试工具的开启

// store/index
import * as redux from 'redux'
import thunkMiddleware from 'redux-thunk'

import reducer from './reducer'

// 创建 composeEnhancers函数
const composeEnhancers =
  window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({ trace: true }) || redux.compose

// 应用中间件
const storeenhancer = redux.applyMiddleware(thunkMiddleware)

// const store = redux.createStore(reducer, storeenhancer)
// 用新创建的composeEnhancers函数包裹原来的中间件
const store = redux.createStore(reducer, composeEnhancers(storeenhancer))

export default store
```

### redux 分模块使用

```jsx
// 目前，我们所有的redux数据都是存放在store/reducer/defaultValue中，在我们网站开发过程中，redux数据会变的非常庞大，reducer中的数据会变得难以阅读和维护，因此我们需要按照不同的页面或者不同的功能将redux拆解成不同的文件夹

// 未分模块前的结构路径
├─store
│ └─ index
│ └─ constant
│ └─ reducer
│ └─ actionCreator

// 分模块之后的结构路径
│  index.js
│  reducer.js
│
├─shopInfo
│      actionCreateors.js
│      constant.js
│      index.js
│      reducer.js
│
└─userInfo
        actionCreateors.js
        constant.js
        index.js
        reducer.js

// 原来的reducer代码
import * as Action from "./constant"

// 默认初始化值
const defaultValue = {
  author: "新新小朋友",
  age: 22,
  shops: [],
}

export default function reducer(state = defaultValue, action) {
  switch (action.type) {
    case Action.UPDATE_AGE:
      return { ...state, age: state.age + action.num }
    case Action.UPDATE_SHOP:
      return { ...state, shops: action.shop }
    default:
      return state
  }
}


// shopInfo下的reducer.js
import * as Action from "./constant"

const initialShopInfo = {
  shops: [],
}
export default function shopInfoReducer(shopInfo = initialShopInfo, action) {
  switch (action.type) {
    case Action.UPDATE_SHOP:
      return { ...shopInfo, shops: action.shop }
    default:
      return shopInfo
  }
}

// userInfo下的reducer.js
import * as Action from "./constant"

const initialUserInfo = {
  author: "新新小朋友呀",
  age: 21,
}

export default function userInfoReducer(userInfo = initialUserInfo, action) {
  switch (action.type) {
    case Action.UPDATE_AGE:
      return { ...userInfo, age: userInfo.age + action.num }
    default:
      return userInfo
  }
}


// store/reducer.js
import { combineReducers } from "redux"

import shopInfoReducer from "./shopInfo"
import userInfoReducer from "./userInfo"

export default function reducer(state = {}, action) {
  return {
    userInfo: userInfoReducer(state.userInfo, action),
    shopInfo: shopInfoReducer(state.shopInfo, action),
  }
}

// 利用redux自带的combinReducers进行合并 和上面自己合并的功能一样
const reducer = combineReducers({
  userInfo: userInfoReducer,
  shopInfo: shopInfoReducer,
})

export default reducer


// 组件使用
import React, { PureComponent } from "react"

import { connect } from "react-redux"

import { updateAge } from "../../store2"

class Me extends PureComponent {
  constructor(props) {
    super(props)

    this.state = {
      authoe: props.author,
    }
  }

  componentDidMount() {}

  render() {
    console.log(this.props)
    const { author, age, friends } = this.props
    return (
      <div>
        <h2>author: {author}</h2>
      </div>
    )
  }
}

const mapStateToProps = (state) => ({
  author: state.userInfo.author,
  age: state.userInfo.age,
  friends: state.userInfo.friends,
  banner: state.shopInfo.banner,
})

const mapDispatchToProps = (dispatch) => ({
  handleAddAge(num) {
    dispatch(updateAge(num))
  },
  handleSubAge(num) {
    dispatch(updateAge(num))
  },
})

export default connect(mapStateToProps, mapDispatchToProps)(Me)

```

### redux 中 Hooks 使用

```jsx
上面的方法我们不难看出，通过一个 connect 函数，并且使用mapSatetToProps 和 mapDispatchToProps 对当前组件和redux进行关联，但是这种方法需要额外编写内容，比较繁琐，因此Redux给我们提供给了对应的Hooks

// 子组件结果Redux使用无论就是两件事，一是获取Redux保存的数据，二是通过dispatch操作Redux

// 发现音乐 - 推荐
import React, { memo, useEffect } from "react"
import { useDispatch, useSelector, shallowEqual } from "react-redux"

import { getBannerDataAction } from "@/store/recommend"

const DiscoverCom = memo(() => {
  // 通过useSelector Hooks 可以获取Redux的数据，第二个参数是为了性能优化，进行每次的值进行浅比较，如果本组件没有依赖Redux其他值，那么其他值改变时，本组件不重新渲染
  const { banners } = useSelector((state) => ({
    banners: state.recommend.banners,
  }), shallowEqual)
  // 通过useDispatch Hooks可以获取dispatch函数，用来调用Redux的action，进行操作修改Redux
  const dispatch = useDispatch()

  useEffect(() => {
    dispatch(getBannerDataAction())
  }, [dispatch])
  return <div>DiscoverCom推荐: {banners?.length}</div>
})

export default DiscoverCom




```

### Immutable 结合 Redux 使用

```jsx
// 什么是Immutable
immutable是一种持久化数据结构，immutable数据就是一旦创建，就不能更改的数据，每当对immutable对象进行修改的时候，就会返回一个新的immutable对象，以此来保证数据的不可变。

// 优点：
节省内存，提高性能

// 用法：
对象浅层比较采用Map， 数组采用List，对象深层比较采用fromJS
修改对象数据采用 im.set('name', '新新')
获取对象数据采用 im.get('name')

修改深层对象数据采用 deepIm.setIn(['info', 'more'], '更多') // 修改 obj.info.more
获取深层对象数据采用 deepIm.getIn(['info', 'more']) // 相当于 obj.info.more

修改数组第一个 im.set(0, '2')
获取数据第二个 im.get(1)


// 结合Redux在项目中使用

// 安装
npm install immutable -S
npm install redux-immutable -S

// store目录
├─ index.js
├─ reducer.js
│
├─shopInfo
├───── actionCreateors.js
├───── constant.js
├───── index.js
├───── reducer.js

// 第一步 修改 shopInfo 下的 reducer.js

// 修改前：
import * as Constant from "./constant"
const recommendValue = {
  // 轮播图数据
  banners: [],
}
const reducer = (state = recommendValue, action) => {
  switch (action.type) {
    // 更新轮播图数据
    case Constant.UPDTAE_BANNERS:
      return { ...state, banners: action.banners }
    default:
      return state
  }
}
export default reducer

// 修改后
// 采用Map进行包裹，以及在修改时，采用set进行修改
import { Map } from "immutable"
import * as Constant from "./constant"
const recommendValue = Map({
  // 轮播图数据
  banners: [],
})
const reducer = (state = recommendValue, action) => {
  switch (action.type) {
    // 更新轮播图数据
    case Constant.UPDTAE_BANNERS:
      return state.set("banners", action.banners)
    default:
      return state
  }
}
export default reducer



// 第二步 修改 store 下的 index.js
// 之前从 redux 中取出combineReducers，现在从 redux-immutable中取出
import { combineReducers } from "redux-immutable"

import recommendReducer from "./recommend"

const siteInfo = {
  author: "新新小朋友",
  time: "2022-05-20",
}

const reducer = combineReducers({
  recommend: recommendReducer,
})

export default reducer



// 第三步 组件使用
// 发现音乐 - 推荐
import React, { memo, useEffect } from "react"
import { useDispatch, useSelector, shallowEqual } from "react-redux"
import { getBannerDataAction } from "@/store/recommend"
const DiscoverCom = memo(() => {
  const { banners } = useSelector(state => ({
    banners: state.get('shopInfo').get('banners')
    // 推荐使用下面这种方法
    banners: state.getIn(['shopInfo', 'banners'])
  }), shallowEqual )
  const dispatch = useDispatch()
  console.log(banners)

  useEffect(() => {
    dispatch(getBannerDataAction())
  }, [dispatch])
  return <div>DiscoverCom推荐: {banners?.length}</div>
})
export default DiscoverCom



```

### Redux 的综合使用

```jsx
// 安装依赖
npm install redux react-redux redux-thunk immutable redux-immutable -S

// 在src目录下新建store文件夹，里面新建 index.js 和 reducer.js 以及其他模块的redux, 目录如下：
├─ index.js
├─ reducer.js
│
├─shopInfo
├───── actionCreateors.js
├───── constant.js
├───── index.js
├───── reducer.js
│
├─userInfo
├─────  actionCreateors.js
├─────  constant.js
├─────  index.js
├─────  reducer.js


// 根 index.js 中
imort { Provider } from 'react-redux'
import store from '@/store'
const root = ReactDOM.createRoot(document.getElementById("root"))
root.render(
  <Provider store={store}>
    <App />
  </Provider>
)



// store/index.js
import { createStore, compose, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'

import reducer from './reducer'

// 调式工具
const composeEnhancers =
  window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({ trace: true }) || compose

// 应用中间件
const storeEnhancer = applyMiddleware(thunk)

const store = createStore(reducer, composeEnhancers(storeEnhancer))

export default store


// reducer.js
import { combineReducers } from "redux-immutable"

import recommendReducer from "./recommend"

const reducer = combineReducers({
  recommend: recommendReducer,
})

export default reducer


//  shopInfo 下的 reducer.js
// 这是发现音乐-推荐下的store
import { Map } from "immutable"
import * as Constant from "./constant"
const recommendValue = Map({
  // 轮播图数据
  banners: [],
})
const reducer = (state = recommendValue, action) => {
  switch (action.type) {
    // 更新轮播图数据
    case Constant.UPDTAE_BANNERS:
      return state.set("banners", action.banners)
    default:
      return state
  }
}
export default reducer


// 组件中使用
// 发现音乐 - 推荐
import React, { memo, useEffect } from "react"
import { useDispatch, useSelector, shallowEqual } from "react-redux"

import { getBannerDataAction } from "@/store/recommend"

const DiscoverCom = memo(() => {
  const { banners } = useSelector(
    (state) => ({
      banners: state.getIn(["recommend", "banners"]),
    }),
    shallowEqual
  )
  const dispatch = useDispatch()
  console.log(banners)

  useEffect(() => {
    dispatch(getBannerDataAction())
  }, [dispatch])
  return <div>DiscoverCom推荐: {banners?.length}</div>
})
export default DiscoverCom

```

## 开发经验

### 配置别名

```jsx
类似于Vue中的 src => @

需要借助 craco 来完成
// 安装
yarn add @craco/craco
或者 npm install @craco/craco -S
npm install craco-less -S

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

// 原理
__dirname： 获取当前文件的路径
等价于： G:\react.js\react学习\dome1\node

// src下
resolve("src") => path.resolve(__dirname, 'src')
等价于：  G:\react.js\react学习\dome1\node\src


// 修改packjson下的script内容
"scripts": {
  "start": "craco start",
  "build": "craco build",
  "test": "craco test",
  "eject": "react-scripts eject"
},



```

### 使用 moment 库

```jsx
// 安装moment
npm install moment -S
yarn add monment

// 在react中使用moment而不是使用dayjs，只因为Antdesign这个库很多时间转化采用的是moment

import moment from 'moment'
// 格式化为中文
import "moment/locale/zh-cn"

// 获取当前时间戳
moment().valueOf()

// 格式化当前时间 和dayjs类似
moment().format("YYYY-MM-DD HH:mm:ss")
// 时间戳格式需要转化为13位
moment(1651373878000).format("YYYY-MM-DD HH:mm:ss")

// 相对时间
moment().fromNow()
moment(1651373878000).fromNow()

```

### 使用 classnames 库

```jsx
// 一个方便动态添加动态class的库

// 安装
npm install classnames -S

// 使用
// 导入
import classNames from 'classnames'

// 使用
<h2
	// 基本类 aaa, bbb 需要动态添加的类名： active
	className={classNames("aaa", "bbb", { active: this.state.isActive })}
>
  这是Home组件
</h2>

```

### 跨域处理

```js
// 目标访问地址： http://huangjianxin.cn/admin/get/hotArticle

// 本地地址: http://localhost:3000

// package.json 添加下面一句话即可
"proxy": "http://huangjianxin.cn",

// 然后在发送网络请求时，去掉前缀
axios.get("/admin/get/hotArticle")

```

### React 过渡动画

![](https://img-blog.csdnimg.cn/61cda6a5faeb484487b7eb012c306275.png)

- CSSTransition是基于Transition组件构建的
- CSSTransition执行过程中，有三个状态：appear、enter、 exit:
  它们有三种状态，需要定义对应的CSS样式：
  - 第一类，开始状态：对于的类是-appear -enter、 exit：
  - 第二类：执行动画：对应的类是-appear-active、 -enter-active、 -exit-active:
  - 第三类：执行结束：对应的类是-appear-done. -enter-done、 -exit-done
- CSSTransition常见对应的属性：
- in：触发进入或者退出状态
    - 如果添加了unmountOnExit=(true)，那么该组件会在执行退出动画结東后被移除掉；
    -  当in为true时，触发进入状态，会添加-enter、-enter-acitve的class开始执行动画，当动画执行结束后，会移除两个class，并且添加-enter-done的class；
    - 当in为false时，触发退出状态，会添加-exit. -exit-active的class开始执行动画，当动画执行结束后，会移除两个class，并且添加-enter-done的class；

```jsx
// 一共方便我们做过渡效果的库

// 安装
npm install react-transition-group -S
yarn add react-transition-group

// 使用


// CSSTransition
// 用来做显示和隐藏
import { CSSTransition } from "react-transition-group"

export default () => {
  return (
    <div>
    	<button>测试显示和隐藏</button>
      {/* 需要动画的组件 用CSSTransition来包裹 */}
      <CSSTransition
        // 动画的类名前缀
        classNames="card"
        // 显示和隐藏
        in={isShow}
        // 多久切换类名，并非动画时长
        timeout={500}
        // 退出后卸载组件
        unmountOnExit
        // 初次进入添加动画，需要设置对应appear的类名
        appear
        onEnter={() => console.log("开始进入的钩子函数")}
        onEntering={() => console.log("正在进入的钩子函数")}
        onEntered={() => console.log("进入完成的钩子函数")}
        onExit={() => console.log("开始退出的钩子函数")}
        onExiting={() => console.log("正在退出的钩子函数")}
        onExited={() => console.log("退出完成的钩子函数")}
        >
        <Components />
      </CSSTransition>
    </div>
  )
}

// style.css 对应过渡动画css文件
/* 准备进入 */
.card-enter {
  opacity: 0;
  height: 0;
  transform: scale(0.6);
}

/* 进入过程 */
.card-enter-active {
  opacity: 1;
  height: 326px;
  transform: scale(1);
  transition: all 0.5s;
}

/* 进入完成 */
.card-enter-done {
  height: 326px;
  opacity: 1;
  transform: scale(1);
}

/* 准备离开 */
.card-exit {
  height: 326px;
  opacity: 1;
  height: 0;
  transform: scale(1);
}

/* 离开过程 */
.card-exit-active {
  opacity: 0;
  height: 0;
  transform: scale(0.6);
  transition: all 0.5s;
}

/* 离开完成 */
.card-exit-done {
  opacity: 0;
  transform: scale(0.6);
}

/* 初始化进入 */
.card-appear {
  opacity: 0;
  transform: scale(0.6);
}

/* 进入过程 */
.card-appear-active {
  opacity: 1;
  transform: scale(1);
  transition: all 0.5s;
}

/* 进入完成 */
.card-appear-done {
  opacity: 1;
  transform: scale(1);
}



// SwitchTransition
// 两个组件之间切换的炫酷动画
 {/* 两个组件之间切换显示 */}
<SwitchTransition
  // 控制方式 是先进还是先出
  mode="out-in"
  >
  <CSSTransition
    // 由于这里不再是显示和隐藏，而是切换，因此不能使用 in，而是采用 key
    key={isShow}
    timeout={500}
    classNames="btn"
    >
    {isShow ? (
      <Button onClick={() => this.handleChange()}>点击隐藏按钮</Button>
    ) : (
      <Button onClick={() => this.handleChange()}>点击显示按钮</Button>
    )}
  </CSSTransition>
</SwitchTransition>

/* SwitchTransition 动画 */
.btn-enter {
  opacity: 0;
  transform: translateY(-60px);
}

.btn-enter-active {
  opacity: 1;
  transform: translateY(0);
  transition: all 0.5s;
}

.btn-exit {
  opacity: 1;
  transform: translateY(0);
}

.btn-exit-active {
  transition: all 0.5s;
  opacity: 0;
  transform: translateY(60px);
}



 {/* 给一组元素添加动画 */}
<Button type="primary" onClick={() => this.handleAdd()}>
  添加新元素
</Button>
{/* 需要使用 TransitionGroup 包裹列表 */}
<TransitionGroup>
  {lists.map((item, index) => (
    {/* 列表使用 CSSTransition 包裹 */}
    <CSSTransition key={index} timeout={500} classNames="btn">
      <h3>{item}</h3>
    </CSSTransition>
  ))}
</TransitionGroup>


```
