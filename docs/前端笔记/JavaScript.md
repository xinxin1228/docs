## DOM

### 概念

- 文档对象模型，（Document Object Model）
- 将页面所有的内容表示为可以修改的对象
- 浏览器会对我们编写的 HTML、CSS 进行渲染，同时它又要考虑我们可能会对 JavaScript 进行操作
- 于是浏览器将我们编写在 HTML 中的每一个元素都抽象成一个个对象
- 所有这些对象都可以通过 JavaScript 来对其进行访问，那么我们可以通过 JavaScript 来操作页面
- 所以，我们将这个抽象过程称为文档对象模型

![浏览器文档对象继承图](../image/前端笔记/09.jpg)

### 节点和元素

![节点和元素的区别](../image/前端笔记/10.jpg)

#### 节点和元素之间的导航

![节点导航图](../image/前端笔记/11.jpg)

![元素导航图](../image/前端笔记/12.jpg)

#### 代码说明

```js
<body>
  <!-- 注释 -->
  我是一段文本

  <div class="box">box元素</div>
  <ul>
    <li>1</li>
    <li>2</li>
    <li>3</li>
    <li>4</li>
    <li>5</li>
  </ul>

  <script>
    // 获取body全部信息
    const doc = document.body
    // 获取body内部所有的节点
    const docNodes = doc.childNodes // 包含注释节点以及 换行的 文本节点
    // 获取body内部所有的元素
    const docElements = doc.children // 只包含html标签元素的 元素节点 (html标签元素包含 script 标签)

    // 获取box元素
    const box1 = docElements[0] // 通过body子元素列表索引查找
    const box2 = doc.firstElementChild // 根据body元素第一个子元素查找

    // 获取ul元素
    const ul1 = box1.nextElementSibling // 通过box元素的下一个元素来查找
    const ul2 = docElements[1] // 通过body子元素列表第二个来查找

    // 获取li元素
    const li1 = ul1.children // 获取ul下面的所有子元素

    console.log(li1)
  </script>
</body>
```

#### children 和 childNodes 的区别

```js
<html>
  <div id="#app">
    <p>我是p标签</p>
    <span>我是span标签</span>
    <a>我是a标签</a>
  </div>

  <script>
    const app = document.querySelector('#app') console.log(app.children) //
    HTMLCollection(3) [p, span, a] console.log(app.childNodes) //
    NodeList(7) [text, p, text, span, text, a, text] //
    他们都是类数组，用于存储可以按位置存取的有序节点
  </script>
</html>
```

### 访问 html 元素

```js
// 可以使用多种方式访问html，但是推荐第一种

// 第一种
document.documentElement

// 第二种
document.children.item(0)

// 第三种
document.querySelector('html')
```

### 获取 dom 上绑定事件的 event 对象

```js
 <button οnclick="btnFunc()"></button>
<script>
function btnFunc(){
   var e = window.event || arguments.callee.caller.arguments[0];
   console.log(e);
 }
</script>
```

### 获取属性

```js
// 每个元素都有零个或多个属性，通常用于为元素或其内容附加更多信息。与属性相关的 DOM 方法主要有 3 个：getAttribute()、setAttribute()和 removeAttribute()。这些方法主要用于操纵属性，包括在 HTMLElement 类型上定义的属性。

;<div class="dome" id="do" my-data="自定义" title="这是title" data-age="23">
  自定义元素属性
</div>

const domeEl = document.querySelector('.dome')

// 元素上的自带属性可以直接获取 包括data属性 但是不能获取自定义属性
with (domeEl) {
  console.log(className) // dome
  console.log(id) // do
  console.log(title) // 这是title
  console.log(dataset.age) // 23
}

// 使用 getAttribute() 获取
console.log(domeEl.getAttribute('class')) // dome
console.log(domeEl.getAttribute('id')) // do
console.log(domeEl.getAttribute('title')) // 这是title
console.log(domeEl.getAttribute('data-age')) // 23
console.log(domeEl.getAttribute('my-data')) // 自定义

// 使用 setAttribute() 使用
domeEl.setAttribute('id', '我是ID')

// 使用 removeAttribute() 删除
domeEl.removeAttribute('title')
```

### 动态引入 js 标签和 css 标签

```js
// 动态引入js标签 放置在body标签的最后面
const scriptEl = document.createElement('script')
scriptEl.src = './1.js'
document.body.append(scriptEl)

// 动态引入css标签，放在head标签最后面
const linkEl = document.createElement('link')
linkEl.href = './1.css'
linkEl.rel = 'stylesheet'
document.head.append(linkEl)
```

### data 属性使用

```js
<body>
  <div class="box" data-name="xinxin" data-age="22"></div>

  <script>
    const box = document.querySelector('.box') console.log(box.dataset.name) //
    xinxin console.log(box.dataset.age) // 22
  </script>
</body>
```

### 插入元素

![插入元素](../image/前端笔记/13.jpg)

```js
// 在node子元素的最后面插入节点或字符串
node.append(...nodes or string)
// 在node开头插入节点或字符串
node.prepend(...node or string)
// 在node元素前面插入节点和字符串
node.before(...node or string)
// 在node元素后面插入节点和字符串
node.after(...node or string)
// 替换node元素
node.replaceWith(...node or string)
// 移除node元素
node.remove()
// 克隆node元素 deep: 是否深拷贝 克隆子元素
node.cloneNode(deep?: Boolean)

// 提示： 使用上面最新的方面，放弃使用比如：insertBefore， appendChild这些旧的方法

```

### innerHTML 和 outerHTML 的区别

```js
// innerHTML是给指定DOM里添加DOM元素，outerHTML是替换指定DOM元素，也就是说使用outerHTML之后，指定的DOM将不负存在

<html>
	<ul class='ul1'></ul>
	<ul class='ul2'></ul>
</html>

// innerHTML
const ulEl1 = document.querySelector('.ul1')
ulEl1.innerHTML = `
	<li>我是li1</li>
	<li>我是li2</li>
	<li>我是li3</li>
`
// 替换之后的ul为：
<ul class='ul1'>
	<li>我是li1</li>
  <li>我是li2</li>
  <li>我是li3</li>
</ul>


// outerHTML
const ulEl2 = document.querySelector('.ul2')
ulEl2.outerHTML = `
	<li>我是li1</li>
	<li>我是li2</li>
	<li>我是li3</li>
`
// 替换的ul2直接被取代了，成为
<li>我是li1</li>
<li>我是li2</li>
<li>我是li3</li>
```

### 元素大小

![元素大小](../image/前端笔记/14.jpg)

### 元素样式

```html
<html>
  <head>
    <style>
      #app {
        width: 100px;
        height: 100px;
        background-color: red;
      }
    </style>
  </head>
  <body>
    <div #app style="width: 300px;height: 200px;background-color: blue;"></div>

    <script>
      const appEl = document.querySelector('#app')

      console.log(appEl.style.cssText) // width: 300px;height: 200px;background-color: blue;
    </script>
  </body>
</html>
```

#### style 对象

```js
// style
// 这个 style 属性是 CSSStyleDeclaration 类型的实例，其中包含通过 HTML style 属性为元素设置的所有样式信息，但不包含通过层叠机制从文档样式和外部样式中继承来的样式。

// 以下为 style 方法下的属性介绍

// cssText，包含 style 属性中的 CSS 代码。
// length，应用给元素的 CSS 属性数量。
// item(index)，返回索引为 index 的 CSS 属性名。

// cssText属性可以存取和读取样式的字符串形式的css代码，设置cssText 是一次性修改元素多个样式最快捷的方式，因为所有变化会同时生效

// 案例一：给一个元素设置宽度为200px，高度为100px，背景颜色为red
const divEl = document.querySelector('#app')
// 方法一：
divEl.style.width = '200px'
divEl.style.height = '100px'
divEl.style.backgroundColor = 'red'

// 方法二：
divEl.style.cssText = 'width: 200px;height: 100px; background-color: red'

// 注意，以下方法不可行
divEl.style = {
  width: '200px',
  height: '100px',
  backgroundColor: 'red',
}
```

#### getComputedStyle()对象

```js
// style 对象中包含支持 style 属性的元素为这个属性设置的样式信息，但不包含从其他样式表层叠继承的同样影响该元素的样式信息。DOM2 Style在document.defaultView上增加了getComputedStyle()方法。这个方法接收两个参数：要取得计算样式的元素和伪元素字符串（如":after"）。如果不需要查询伪元素，则第二个参数可以传 null。getComputedStyle()方法返回一个 CSSStyleDeclaration对象（与 style 属性的类型一样），包含元素的计算样式。

// 一般情况下，我们不用document.defaultView.getComputedStyle()使用，只需直接getComputedStyle()即可

document.defaultView.getComputedStyle === getComputedStyle // true

<html>
	<head>
    <style>
      #app {
        width: 100px;
        height: 100px;
        background-color: red;
        font-size: 19px
      }
    </style>
  </head>
  <body>
    <div #app style='width: 300px;height: 200px;background-color: blue;'></div>

    <script>
    	const appEl = document.querySelector('#app')

      const style = getComputedStyle(appEl)

      console.log(style.fontSize) // 19px
			console.log(style.width) // 300px
			console.log(style.backgroundColor) // blue
    </script>
  </body>
</html>

// 注意：在所有浏览器中计算样式都是只读的，不能修改 getComputedStyle()方法返回的对象。而且，计算样式还包含浏览器内部样式表中的信息。因此有默认值的 CSS 属性会出现在计算样式里。

```

### event 事件对象

#### 常见属性

- `type`：事件的类型
- `target`：事件真正发生的对象
- `currentTarget`：当前处理事情的对象
- `eventPhase`：事件所处的阶段
- `offsetX`、`offsetY`：事件发生在元素内的位置
- `clientX`、`clientY`：事件发生在客户端的位置
- `pageX`、`pageY`：事件发生在客户端相对于 document 的位置
- `screenX`、`screenY`：事件发生在相对于屏幕的位置

#### 常见方法

- `preventDefault`：取消事件的默认行为
- `stopPropagation`：阻止事件的进一步传递（冒泡和捕获都可以阻止）

### 常用的鼠标事件

![常用的鼠标事件](../image/前端笔记/15.jpg)

### 常用的键盘事件

![常用的键盘事件](../image/前端笔记/16.jpg)

#### 修饰键

```js
// 键盘上的修饰键 Shift、Ctrl、Alt 和 Meta 经常用于修改鼠标事件的行为。DOM 规定了 4 个属性来表示这几个修饰键的状态：shiftKey、ctrlKey、altKey 和 metaKey。这几属性会在各自对应的修饰键被按下时包含布尔值 true，没有被按下时包含 false。
// 注意：在mac系统上同样也是这几个修饰键
command => metaKey
option => altKey
control => ctrlKey
shift => shiftKey
```

### 常用的浏览器事件

```js
// 1.contextmenu事件
// 也就是浏览器点击右键的菜单
// 隐藏默认的菜单，显示自定义
window.addEventListener('contextmenu', e => {
  // 阻止默认的菜单
  e.prevenDefault()
  // 展示自己的菜单
})
// 点击鼠标左键 隐藏菜单
docunment.addEventListener('click', e => {
  // 隐藏菜单
})

// 2.beforeunload事件
//beforeunload 事件会在 window 上触发，用意是给开发者提供阻止页面被卸载的机会。这个事件会在页面即将从浏览器中卸载时触发，如果页面需要继续使用，则可以不被卸载。这个事件不能取消，否则就意味着可以把用户永久阻拦在一个页面上。相反，这个事件会向用户显示一个确认框，其中的消息表明浏览器即将卸载页面，并请用户确认是希望关闭页面，还是继续留在页面上 弹窗演示如下图所示

window.addEventListener("beforeunload", (event) => {
 event.returnValue = '即将关掉弹窗';
});

// 3.hashchange事件
// HTML5 增加了 hashchange 事件，用于在 URL 散列值（URL 最后#后面的部分）发生变化时通知开发者。这是因为开发者经常在 Ajax 应用程序中使用 URL 散列值存储状态信息或路由导航信息。onhashchange 事件处理程序必须添加给 window，每次 URL 散列值发生变化时会调用它。event
// 对象有两个新属性：oldURL 和 newURL。这两个属性分别保存变化前后的 URL，而且是包含散列值的完整 URL。下面的例子展示了如何获取变化前后的 URL：
window.addEventListener("hashchange", (event) => {
 console.log(`Old URL: ${event.oldURL}, New URL: ${event.newURL}`);
});
如果想确定当前的散列值，最好使用 location 对象：
window.addEventListener("hashchange", (event) => {
 console.log(`Current hash: ${location.hash}`);
});

```

![beforunload事件弹窗](../image/前端笔记/17.jpg)

### 常用的表单事件

![常用的表单事件](../image/前端笔记/18.jpg)

### 拖拽事件

```js
每个可拖动的元素，在拖动过程中，都会经历三个过程：
拖动开始=>拖动过程中=>拖动结束

// 拖拽的元素
const drapEl = document.querySelector('.drap')
// 存放拖拽元素的目标容器
const targetEl = document.querySelector('.target')

// 拖拽要被拖拽的元素
// 在元素开始被拖动时候触发
drapEl.addEventListener('dragstart', function () {
  this.classList.add('hold')
  setTimeout(() => (this.className = 'xx'), 0)
  // 这里的this指当前被拖拽的元素
  this.parentNode.classList.add('leaved')
  oldEl = this.parentNode
})
// 在拖动操作完成时触发
drapEl.addEventListener('dragend', function () {
  this.classList.add('img')
  this.classList.remove('xx')
  console.log(this.parentNode)
  oldEl.classList.remove('leaved')
})

// 当被拖动元素在目标元素内时触发
drag.addEventListener('dragover', e => {
  // 注意:dragenter和dragover事件的默认行为是拒绝接受任何被拖放的元素。因此，我们必须阻止浏览器这种默认行为。e.prevenDefault();
  e.preventDefault()
})
// 当被拖动元素没有放下就离开目标元素时触发
drag.addEventListener('dragleave', e => {
  e.target.classList.remove('hovered')
})
// 当被拖动元素没有放下就离开目标元素时触发
drag.addEventListener('dragenter', e => {
  e.preventDefault()
  e.target.classList.add('hovered')
})
// 当被拖动元素在目标元素里放下时触发
drag.addEventListener('drop', function () {
  this.append(imgEl)
  this.classList.remove('hovered')
})
```

| 对象             | 事件名称  | 描述                     |
| ---------------- | --------- | ------------------------ |
| 被拖动的元素对象 | dragstart | 在元素开始被拖动时候触发 |
|                  | drag      | 在元素被拖动时反复触发   |
|                  | dragend   | 在拖动操作完成时触发     |

| 对象             | 事件名称  | 描述                                                             |
| ---------------- | --------- | ---------------------------------------------------------------- |
| 目标对象（拖动） | dragenter | 当被拖动元素进入目标元素占据的屏幕空间时触发                     |
| （拖动）         | dragover  | 当被拖动元素在目标元素内时触发                                   |
| （拖动）         | dragleave | 当被拖动元素没有放下就离开目标元素时触发                         |
| （释放）         | drop      | 当被拖动元素在目标元素里放下时触发，一般需要取消浏览器的默认行为 |

## BOM

- 浏览器对象模型，（Browser Object Model）
- 由浏览器提供的用于处理文档之外的所有内容的其他对象

### windows 对象

#### 窗口大小和视口位置

```js
// BOM 的核心是 window 对象，表示浏览器的实例。window 对象在浏览器中有两重身份，一个是ECMAScript 中的 Global 对象，另一个就是浏览器窗口的 JavaScript 接口。这意味着网页中定义的所有对象、变量和函数都以 window 作为其 Global 对象，都可以访问其上定义的 parseInt()等全局方法。

// window的width和height
  `innerWith`和`innerHeight`：获取window窗口的宽度和高度（包含滚动条）
  `outerWith`和`outerHeight`：获取window窗口的整个宽度和高度
  `documentElement.clientHeight`和`documentElement.clientWith`：获取html宽度和高度
// window的滚动位置
  `scrollX`：X轴滚动的位置，也就是横向滚动条滚动的位置，或者 `document.documentElement.scrollLeft`
  `scrollY`：Y轴滚动的位置，也就是纵向滚动挑滚动的位置，或者`document.documentElement.scrollTop`
// 滚动方法
  `scrollBy(x, y)`：将页面滚动至相对于当前位置的(x, y)位置
  `scrollTo(x, y)`将页面滚动至绝对坐标

// 平滑滚动条距离浏览器顶部1000px的位置
window.scrollTo({
  top: 1000,
  left: 0,
  behavior: 'smooth', // 平滑滚动
})
```

#### 导航与打开新窗口

```js
window.open() // 可以用于导航到指定 URL
// 接受四个参数
// 要加载的 URL
// 目标窗口
// 特性字符串
// 表示新窗口在浏览器历史记录中是否替代当前加载页面的布尔值。

window.open('/login') // 新窗口打开并跳转到login
windwo.open('/login', '_parent') // 当前页跳转到login

// 关闭打开的新窗口
const info = window.open('/login') // 获取新打开窗口的信息
// 关闭窗口 方法一
info.close()
// 关闭窗口 方法二
window.close(info)
// 注意： 以上窗口关闭方法只能用于 window.open()创建的弹出窗口。
// 并且， 弹出窗口可以调用 top.close()来关闭自己
```

### 定时器

```js
// JavaScript 在浏览器中是单线程执行的，但允许使用定时器指定在某个时间之后或每隔一段时间就执行相应的代码。setTimeout()用于指定在一定时间后执行某些代码，而 setInterval()用于指定每隔一段时间执行某些代码。

// 案例一：
// 控制台依次打印1～10，之后清除定时器

// 使用 setInterval() 实现
const max = 11
let num = 0,
  timer = null

function cycle() {
  num++
  if (num >= max) {
    clearInterval(timer)
  } else {
    console.log('当前打印的数字', num)
  }
}

timer = setInterval(caycle, 100)

// 使用 setTimeout() 实现
const max = 11
let num = 0

function cycle() {
  num++
  if (num < max) {
    setTimeout(cycle, 100)
  }
}

setTimeOut(cycle, 100)

// 总结：注意在使用 setTimeout()时，不一定要手动清除定时器，因为它会在条件满足时自动停止，否则会自动设置另一个超时任务。这个模式是设置循环任务的推荐做法。

// setIntervale()在实践中很少会在生产环境下使用，因为一个任务结束和下一个任务开始之间的时间间隔是无法保证的，有些循环定时任务可能会因此而被跳过。而像前面这个例子中一样使用 setTimeout()则能确保不会出现这种情况。一般来说，最好不要使用 setInterval()。

// 案例二：
// 使用定时器轮询请求一个接口。直到该接口返回值中的 done 为 true的时候，停止轮询请求

// 模拟网络请求，发送数据
const getData = () => {
  return new Promise((resolve, reject) => {
    const num = (Math.random() * 10000) | 0
    console.log(
      `开始轮询请求数据，预计返回时间为:${num}, 是否继续发起请求：${
        num % 2 ? '是' : '否'
      }`
    )

    setTimeout(() => {
      console.log('数据请求完成', num)
      resolve({ data: num, done: num % 2 ? false : true })
    }, num)
  })
}

// 使用 setTimeout() 实现
;(async function () {
  const data = await getData()
  if (!data.done) {
    setTimeout(arguments.callee) // arguments.callee用来获取当前的匿名函数
  }
})()

// 使用 setInterval() 实现
let timer
;(async function () {
  clearInterval(timer) // 默认先清除一次定时器，防止循环
  const data = await getData()

  if (!data.done) {
    timer = setInterval(arguments.callee)
  }
})()
```

### location 对象

![location 对象](../image/前端笔记/19.jpg)

#### 路径参数说明

路径： `http://foouser:barpassword@www.wrox.com:80/WileyCDA/?q= javascript#content`

|       属性        |                                     值                                     |
| :---------------: | :------------------------------------------------------------------------: |
|   location.hash   |                                  #content                                  |
|   location.host   |                              www.wrox.com:80                               |
| location.hostname |                                www.wrox.com                                |
|   location.href   | http://foouser:barpassword@www.wrox.com:80/WileyCDA/?q= javascript#content |
| location.pathname |                                 /WileyCDA/                                 |
|   location.port   |                                     80                                     |
| location.protocol |                                   http:                                    |
|  location.search  |                               ?q=javascript                                |
| location.username |                                  foouser                                   |
| location.password |                                barpassword                                 |
|  location.origin  |                            http://www.wrox.com                             |

#### 获取搜索函数

```js
// 方法一：使用 URL 解析
const url = new URL(location.href)

// 获取对象的值， 获取不到时为 null
console.log(url.searchParams.get('name')) // web
console.log(url.searchParams.get('age')) // 22

// 设置值
url.searchParams.set('name', 'xx')

// 追加
url.searchParams.append('namex', '11')

// 判断
console.log(url.searchParams.has('name')) // true


// 方法二：（推荐） 使用 URLSearchParams 解析
假设当前 URL 为 http://127.0.0.1:59573/BOM/location.html?q=java&name=web&age=23
const searchParams = new URLSearchParams(lcoation.search)

console.log(searchParams) // { size: 3 }
console.log(searchParams.toString()) // q=java&name=web&age=23

// 返回的实例对象可迭代
for (const params of searchParams.entries()) {
	console.log(params)
}
// ['q', 'java']
// ['name', 'web']
// ['age', '23']


console.log('获取name的值', searchParams.get('name')) // web
console.log('获取name11的值', searchParams.get('name11')) // null

console.log('判断name是否存在', searchParams.has('name')) // true
console.log('判断name11是否存在', searchParams.has('name11')) // false

console.log('设置name的值为 react', searchParams.set('name', 'react')) // undefined
console.log(
  '设置name11的值为 angular',
  searchParams.set('name11', 'angular')
) // undefined
console.log(
  '设置之后的值',
  new URLSearchParams(location.search).get('name'), // web 修改上一个对象的时候，不会影响新的对象取值
  searchParams.get('name'), // react
  searchParams.get('name11') // angular
)

 console.log('检测是否删除name', searchParams.has('name')) // true
console.log('删除name', searchParams.delete('name')) // undefined
console.log('检测是否删除name', searchParams.has('name')) // false
```

### history 对象

![history 对象](../image/前端笔记/20.jpg)

### JSON、Storage

- `JSON.stringify()` 将一个 JavaScript 对象或值转换为 JSON 字符串``
- `JSON.parse()` 将 JSON 字符串转化为 JavaScript 对象

![JSON.stringify()](../image/前端笔记/21.jpg)

```js
const obj = {
  name: 'web',
  age: 22,
}

const objString = JSON.stringify(obj, (key, value) => {
  if (key === 'age') return 20
  return value
})
console.log(objString) //{"name":"web","age":20}

// JSON.parse() 也是如此
```

## 日常总结

### 判断奇偶数

```js
奇数
num & 1 == 1

偶数
num & 1 == 0

判断数组是否含有某个数
(arr.indexOf(123) === -1) => !~arr.indexOf(123)
```

### 匿名函数写法

```js
// 写法一
;(function () {
  console.log('我是匿名函数')
})()

// 写法二：
;(function () {
  console.log('我是匿名函数')
})() +
  // 写法三：
  (function () {
    console.log('我是匿名函数')
  })()
```

### 判断数组中是否含有某个值

```js
// 判断数组是否含有4
let arr = [1, 4, 2, 4, 2, 3, 2, 1]
// 有就返回数据 没有返回undefined
let bol = arr.find(item => item === 4)
let bol1 = arr.filter(item => item === 4)
let bol2 = arr.some(item => item === 4)
let bol3 = new Set(arr)
let bol4 = arr.inclueds(4)
console.log(bol) // 4
console.log(bol1.length > 0) // true
console.log(bol2) // true
console.log(bol3.has(4)) // true
console.log(bol4) // true

// 简写方式
console.log(arr.includes(3))
console.log(!!arr.find(item => item === 3))
console.log(arr.findIndex(item => item === 3) >= 0)
console.log(new Set(arr).has(3))
console.log(arr.filter(item => item === 3).length > 0)
console.log(arr.some(item => item === 3))
```

### 数组拼接

```js
let arr = [1, 4, 5, 4, 2]
let s = [8, 7, 6]
console.log(arr.push(...s)) //[1,4,5,4,2,8,7,6]
console.log([...arr, ...s]) //[1,4,5,4,2,8,7,6] 推荐
console.log(arr.concat(s)) //[1,4,5,4,2,8,7,6]
```

### 深拷贝与浅拷贝

```js
// 栈
// 基本数据类型常放置于栈中，有固定的大小

// 堆
// 引用数据类型由于大小不确定，因此存储与堆中

// 什么是深拷贝与浅拷贝
// 假设B复制了A，当我们修改A的时候，如果B跟着一起改变，那么就是浅拷贝，如果B没有跟着一起改变，那么就是深拷贝

// 基本数据类型不存在深拷贝和浅拷贝 如果非要纠结 就是深拷贝 存在于栈中
// 基本类型 => 深拷贝（也叫赋值，存在栈里，新建一块空间）
// 浅拷贝：
// 1.针对引用类型数据 对象 数组
// 2.指针、内存地址

// 实现深拷贝
// 利用new Object()和Object.assign()的方式都不能实现深拷贝，他们只是实现了一级深拷贝

// 利用 JSON.parse(JSON.stringify(obj)) 可以实现深拷贝 但是不支持function

// 手写深拷贝
function deep(dist: object, obj: any): any {
  let o = dist
  for (let i in obj) {
    if (typeof obj[i] === 'object') {
      o[i] = obj[i].constructor === Array ? [] : {}
      deep(o[i], obj[i])
    } else {
      o[i] = obj[i]
    }
  }
  return o
}
```

### 求元素宽高及边距

```js
//在原生中，求元素的宽度和高度，以及元素距离父级元素的距离
box.offsetHeight  //求元素的高度
box.offsetWidth   //求元素的宽度
box.style.left    //求元素距离父级元素的左边距
box.style.top     //求元素距离父级元素的上边距
box.offsetTop     //求元素距离顶部的距离，必须是在最外层元素
box.offsetWidth   //求元素距离左边的距离，必须是最外层
box.style.left = number   //设置元素距离父级的左边距
box.style.width = number  //设置元素的宽度
window.innerWidth  // 浏览器的宽度
window.innerHeight  // 浏览器的高度
document.documentElement.scrollTop   // 滚动条滚动的距离

//在jQuery中，求元素的宽度和高度，以及元素的距离父级元素的距离
$(box).height()  //求元素的高度
$(box).width()   //求元素的宽度
$(box).offset().top   //获取元素距离父级元素的上边距
$(box).offset().left  //获取元素距离父级元素的左边就
$(box).css({
  left:number     //设置元素距离父级元素的左边距
  top:number      //设置元素距离父级元素的上边距
})
$(window).scrollTop() // 获取滚动条滚动距离
```

### 添加或移动类名

```js
//在原生中，删除元素的某个类名或者是添加某个类名
box.className = 'ss' //替换元素的类名
box.className += ' ss' //添加元素的类名
box.classNmae = '' //清空类名
box.classList.add('ss') //添加元素的类名
box.classList.remove('ss') //删除元素的类名（有就删除，没有的话不执行 不会报错）
box.classList.toggle('ss') //有这个类名就删除，没有就添加
box.classList.contains('ss') //判断是否这个类名

//在jQuery中，删除元素的某个类名或者是添加某个类名
$(box).removeClass('s') //删除元素的类名（有就删除，没有的话不执行 不会报错）
$(box).addClass('s') //添加元素的类名
$(box).toggleClass('ss') //有这个类名就删除，没有就添加
$(box).attr('class', '') //清空类名
$(box).attr('class', 'a') //覆盖里面所有的类名
$(box).hasClass('a') //判断是否含有这个类名
```

### 网站上设置随着滚动条滚动，页面内容以动画的形式出来

```js
原理：先获取元素距离顶部的距离，记为boxTop，然后获取浏览器的高度，记为winH，然后监听浏览器滚动条滚动事件，记滚动条距离顶部为scrollTop，当scrollTop + winH > boxTop，给该元素添加对应的动画。

公式： scrollTop + winH > boxTop

代码：
// 获取元素
const box = document.querySelector('.box')
// 获取元素距离顶部的距离
const boxTop = box.offsetTop
// 监听滚动条滚动事件
window.addEventListener('scroll', () => {
  // 获取浏览器的高度
  let winH = window.innerHeight
  // 获取滚动条滚动距离
  let scrollTop = document.documentElemet.scrollTop
  if(scrollTop + winH > boxTop) {
    // 添加动画类名
    const cls = ['animated', 'Fading', 'Entrances', 'fadeInLeft']
    queueMicrotask(() => {
      five.classList.add(...cls)
    })
  } else {
    // 这里可以选择移除动画
    const cls = ['animated', 'Fading', 'Entrances', 'fadeInLeft']
    queueMicrotask(() => {
      five.classList.remove(...cls)
    })
  }
}, false)
```

### 切换图片路径

```js
//在原生中，切换图片的路径
img.src = ''

//在jQuery中，切换图片的路径
$(img)[0].src = imgArr[index]
$(img).attr('src', `${imgArr[index]}`)
```

### 几种 for 循环的区别

```js
//for(;;)与for(i in arr)的区别
当他们操作元素时，效果时一样的，
但是当自我遍历的时候，for(;;)只能遍历自身的元素
然而for(i in arr)则会遍历内部的本地元素 （遍历对象使用只能使用for...in）
因此，当要利用循环绑定事件的时候，优先选择使用for...in
利用循环计算某个数组的和时，使用普通的for循环 ，reduce函数
let arr = [1,5,4]
//for(let s of arr) ES6
这种方法可以快速获取数组元素的值
其中arr可以配置三个参数
keys() 变量接受下标
for(let s of arr.keys())  // 0,1,2
values() 变量接受值 （默认）
for(let s of arr.values()) // 1,5,4
 entries() 下标和值
 for(let s of arr.entries()) //[0,1] [1,5] [2,4]

具有length 和symbol才是可迭代的

可枚举意味着 可以使用for in 遍历
可迭代意味着 可以使用for of 遍历


// forEach没有返回值
let s = forEach(item => {
  return item
})
console.log(s)  // undefined

// map 映射函数  返回一个数组
let arr = [{id:0,name:"web"},{id:1,name:"web2"},{id:3,name:"web3"}]
arr.map(item=>{
  return arr.id  // [0,1,2]
})

// filter 过滤函数   返回一个数组
let arr = [1,4,2,4,2]
arr.filter(item=>{
	return item > 3   // [4,4]
})

// some  条件函数  返回boolean 有一个满足 返回true
let arr = [1,4,2,3]
let s = arr.some(item=>{
  return item>3
})
console.log(s)  // true

//every 条件函数  返回boolean  有一个不满足  返回false
let arr = [1, 4, 2, 3]
let s = arr.every(item => {
  return item>3
})
console.log(s)  // false

// reduce 汇总函数  返回一个值
let arr = [1, 2, 4, 2, 5]
let s = arr.reduce((prev, next) => {
  return prev + next
},0)
console.log(s)  // 14


```

### ES6 中的数组操作

```js
// Array.of()  用于将一组值转化为数组
Array.of(1,2,3,4)  // [1,2,3,4]

// Array.from()  可以将类数组和具备迭代的接口对象转化为数组
Array.from(new Set(arr))

// Set 类型 可以用来做去重
let set = new Set([1,2,3,2,3])
console.log(set)  // {1,2,3}
console.log([...set])  // [1,2,3]


// 大于20 并且去重
let ss = [1, 2, 13, 15, 20, 35, 123, 45, 1, 624, 465, 23, 23, 23, 45, 42]
let r = Array.of(...new Set(ss.filter(item => item > 20)))
console.log(r) // [35, 123, 45, 624, 465, 23, 42]

// find
// 方法的回调函数可以接受三个值  依次为当前值，当前的位置和原数组  用来找第一个符合要求的并返回的值 和filter有所不同 未找到返回 undefined
let arr = [2,1,4,3]
let newArr = arr.find(item => item > 2)  // 4
let newArr = arr.filter(item => item > 2) // [4,3]
let newArr = arr.find(item => item < 0) // undefined
let newArr = arr.filter(item => item < 0) // []

// findIndex()
// 用法与find类型，只不过该方法返回的是找到第一个值的下标 未找到返回 -1
let newArr = arr.findIndex(item => item > 2)  // 2

// map类型 它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。
const m = new Map(); m.set('name','value').set('age',18).set('a','a');
var a = {id:2};
m.set(a,'value');
// size属性  成员个数
const m = new Map();
m.set('a','a').set('b','b');
m.size //2
// get属性  读取 key 对应的键值，如果找不到 key ，返回 undefined
const m = new Map();
var a = {id:2};
m.set(a,'value');
m.set('name','jindu');
m.get(a); //'value' 通过变量a来获取
m.get('name'); //jindu 通过'name'属性获取
// has(key) 返回一个布尔值，表示某个键是否在当前 Map 对象之中。
const m = new Map();
var a = {id:2};
m.set(a,'value');
m.set('name','jindu');
m.has('name') //true
m.has('a') //false
m.has(a) //true
// delete(key) 删除某个键，返回 true 。如果删除失败，返回 false
const m = new Map();
var a = {id:2};
m.set(a,'value');
m.set('name','jindu');
m.delete('name')
m.has('name') //false
// clear() 清除所有成员，没有返回值。
const m = new Map();
var a = {id:2};
m.set(a,'value');
m.set('name','jindu');
m.clear();

// 综合用法 给定两个数组 求两个数组的并集  交集  差集
let a1 = [1, 2, 3, 4, 5, 6, 4, 5, 6];
let a2 = [4, 5, 6, 7, 8, 9, 7, 8, 9];

// 并集
let arr = [...new Set([...a1, ...a2])]  // [1, 2, 3, 4, 5, 6, 7, 8, 9]
// 交集
let arr = a1.filter(item => a2.includes(item))
// 方法二
let arr = al.filter(item => new Set(a2).has(item)}
return [...new Set(arr)]  //[4, 5, 6]
// 差集
let arr1 = [...new Set(a1.filter(item => {
  if (!a2.includes(item)) {
    return item
  }
}))]
let arr2 = [...new Set(a2.filter(item => {
  if (!a1.includes(item)) {
    return item
  }
}))]
console.log(arr1,arr2)  //[1,2,3]  [7,8,9]

```

### js 中类型转换

```js
let map = new Map([["a",1],["b",2],["c",3],["d",4]])
let obj = {"a":1,"b":2,"c":3}

// 类数组转数组
arguments => array
1. arguments = Array.from(arguments)
2. arguments = Array.prototype.slice.call(arguments) // 调用数组的slice方法，使用call指向被调用的类数组
3. arguments = [].slice.call(arguments) // 类似上面的方法，只是把Array.prototype用[]代替
4. arguments =  [...arguments]

// 对象转数组
Object.keys(obj)  // ["a","b","c"]
Object.values(obj)  // [1,2,3]

// map类型转数组
[...map.keys()]  // ["a", "b", "c", "d"]
[...map.values()]  // [1,2,3,4]
[...map]   // [["a",1],["b",2],["c",3],["d",4]]

// map转化为object
let o2 = {}
for(let [k,v] of map){
  o2[k] = v
}
console.log(o2)  //{a: 1, b: 2, c: 3, d: 4}

// object转map
let map = new Map()
for(let i in obj){
  map.set(i, obj[i])
}
console.log(map)  // {"a" => 1, "b" => 2, "c" => 3}

// object转map 第二种方式
new Map(Object, entries(obj))

```

### Async/Await 使用

```js
// 要完全理解 await 关键字，必须知道它并非只是等待一个值可用那么简单。JavaScript 运行时在碰到 await 关键字时，会记录在哪里暂停执行。等到 await 右边的值可用了，JavaScript 运行时会向消息队列中推送一个任务，这个任务会恢复异步函数的执行。因此，即使 await 后面跟着一个立即可用的值，函数的其余部分也会被异步求值。

// 个人理解 碰见await就立即暂停，不止暂定后面的代码，也暂停await那行代码，如下面的 代码顺序执行案例：

async function foo() {
  console.log(2)
  console.log(await Promise.resolve(8))
  console.log(9)
}
async function bar() {
  console.log(4)
  console.log(await 6)
  console.log(7)
}
console.log(1)
foo()
console.log(3)
bar()
console.log(5)
// 1
// 2
// 3
// 4
// 5
// 8
// 9
// 6
// 7

// async 函数返回的是一个promise 对象，如果要获取到promise 返回值，应该用then 方法
async function fn() {
  return 100
}
console.log(fn()) //Promise {<resolved>: 100}
let f = fn()
f.then(function (d) {
  console.log(d) //100
})

// await操作符用于等待一个Promise对象，它只能在异步函数async function内部使用。
// 返回值：返回promise对象的处理结果，如果待等的不是promise对象，则返回该值本身
// 如果一个promise被传递给一个await操作符，await将等待promise正常处理完成并返回其处理结果

function f2() {
  console.log(4)
}
async function f() {
  console.log(1)
  await f2() // 阻塞f2()执行，会先执行f2()-执行同步任务-await后面的代码
  console.log(2)
}
f()
console.log(3) // 1 4 3 2

setTimeout(() => {
  // 宏任务
  console.log(1)
}, 0)
new Promise((resloved, rejected) => {
  resloved()
  console.log(2) // 同步
}).then(() => {
  // 异步
  console.log(3) // 微任务
})
console.log(4) // 同步
// 2 4 3 1

function p() {
  return new Promise(resolve => {
    resolve()
    console.log(1)
  })
}
async function fn() {
  console.log(2)
  await p()
  console.log(3)
}
fn() // 2 1 3
```

### jQuery 中的 each 循环

```js
//原生与jQuery中的循环的区别
for(i in arr)
{
  arr[i].index = i   ？？？
  arr[i].onclick = function(){
    this.index        //此时，this指向当前操作的元素，this.index表示当前操作元素的下标
  }
}


$.each(arr, function(index, value){  //index表示每个元素的下标，value表示每个元素  （在Vue中，index和value与jQuery相反）
  $(value).click(function(){
    value   //此时，value表示当前操作的元素
    index   //此时，index表示当前操作元素的下标
  })

})
```

### jQuery 中常用的 dom 操作

```js
//$(a).before(b)  在a元素的前面添加b元素
//$(a).after(b)   在a元素的后面添加b元素
//$(a).append(b)  在a元素里面的子元素最后添加b元素
//$(a).prepend(b)  在a元素里面的子元素最前面添加b元素
//$(a).replaceWith(b)  将a使用b进行替换

// find()  用来找某父元素下面的子元素  可以链式调用
$('.bar').show().find('.user span').text(user)
// 让父元素bar显示，并且使其里面的user的span元素的内容变为user内容

let p = $('p') //获取所有的p标签，然后给除了第三个标签外的所有标签添加类名为 goudan
// 方法一：p.addClass("goudan").eq(2).removeClass("goudan")
// 方法二：
p.eq(2).removeClass('goudan').siblings().addClass('goudan')
```

### 滚动条事件

```js
//利用jQuery绑定滚动条事件   （推荐）
$(window).scroll(function () {
  //触发滚轮事件时，监听滚轮事件属性
  console.log($(window).scrollTop())
  if ($(window).scrollTop() >= 200) {
    //当滚动条滚动一定距离处理事件
    $(show).show(200) //显示元素
  } else {
    $(show).hide(200) //隐藏元素
  }
})

//利用原生js绑定滚动条事件  (不推荐)
window.onscroll = function () {
  console.log(document.documentElement.scrollTop) //获得当前滚动条的高度
  //当滚动条滚动一定距离处理事件
  if (document.documentElement.scrollTop > 400) {
    box.style.display = 'none'
  } else {
    box.style.display = 'block'
  }
}
```

### 点击事件控制滚动条滚动指定位置

```js
//原生js的方法    (了解即可，不推荐使用)
//先获取当前的滚动条位置，并计算出目标的上边距，如果当前大于目标，那么设置定时器或者请求动画帧，使其当前的滚动条上边距不变减减，知道小于等于指定位置的时候，关闭定时器或者请求动画帧
//方法一：定时器控制滚动
let btn = document.querySelector(".p") //假设指定位置为p
btn.onclick = ()=>{
  let a = btn.offsetTop
  let now = document.documentElement.scrollTop
  let time = setInterval(() => {
    if(now>a){
    now -= 10
    window.scrollTo(0,now)
    if(now<=a){
      clearInterval(time)
    }
  }else{
    now += 10
    window.scrollTo(0,now)
    if(now<=a){
      clearInterval(time)
    }
  }
  },1)
}
//方法二：使用请求动画帧控制滚动条
let btn = document.querySelector(".p")
btn.onclick = ()=>{
  let a = btn.offsetTop
  let now = document.documentElement.scrollTop
  let time = null
  function fn(){
    if(now>a){
    now -= 50
    window.scrollTo(0,now)
    time = requestAnimationFrame(fn)  //调用请求动画帧
    if(now<=a){
      cancelAnimationFrame(time)  //清除请求动画帧
    }
  }else{
    now += 50
    window.scrollTo(0.now)
     time =  requestAnimationFrame(fn)  //调用请求动画帧
    if(now<=a){
       cancelAnimationFrame(time)  //清除请求动画帧
    }
  }
  }
  fn()
}


//jQuery方法    （推荐写法）
//jQuery中不用计算当前的滚动条位置，并且可以使用animate动画自动完成，不需要关闭。
$('.btn').click(()=>{
  $("html,body").animate({
   scrollTop:$(".top").offset().top
  },600)
})
```

### jQuery 控制视频播放与暂停

```js
//利用jQuery控制video视频的播放与暂停
$(video).trigger('play') //开始播放视频
$(video).trigger('pause') //暂停播放视频
```

```js
//在video标签中的几种常见熟悉
autoplay //是否自动播放
loop //是否循环播放
controls //是否显示播放控件
muted //是否静音 （当网页打开时，视频无法自动播放可以加上这个属性）
```

### 数组 sort()的几种用法

```js
//数组的几种排序方法
//1.只对数组进行排序
let s = [1, 3, 5, 7, 2, 4]
s.sort((a, b) => a - b) //[1,2,3,4,5,7]  从小到大排序
s.sort((a, b) => b - a) //[7,5,4,3,2,1]  从大到小排序
s.sort(() => Math.random() - 0.5) //[3,5,1,2,4,7]  随机排序
//2.对数组里面的某个值进行排序，进而使整体排序
let ss = [
  {
    name: '《银行家算法》',
    price: 100,
  },
  { name: '《代码大全》', price: 114 },
  { name: '《编程珠玑》', price: 60 },
  { name: '《编程艺术》', price: 130 },
  { name: '《java从入门到入土》', price: 30 },
]
ss.sort((a, b) => a.price - b.price) //按照图书的价格使整体从小到大排序
ss.sort((a, b) => b.price - a.price) //按照图书的价格使整体从大到小排序
ss.sort(() => Math.random() - 0.5) //随机排序
```

### ES6 中解构和剩余参数

```js
// 结构中的别名
let {a: q, b, c} = {a: 1, b: 2, c: 3}
// 相当于q是a的别名，但是匹配后面的值的时候，还是会以a进行匹配 以q进行展示
q = 1 , b=2, c=3

let {a, b: y} = {a: 1, y: 2} // a = 1, y = undefined

// 默认值
let {x=3} = {x:10}  // x = 10
let {x,y=10} = {x:10,y=null} // x=10,y=null
let {x=1,y=10} = {x:undefined} // x=1,y=10
let {x:n=1,y:a=2} = {x:10,y:20} //n = 10,a = 20

function move({x=0,y=0}={}){
  return [x,y]
}
move({x:3,y:8}) // 相当于 => {x=0,y=0}={x:3,y:8}  [3,8]
move({x:3})   // 相当于 => {x=0,y=0}={x:3}   [3,0]
move({})   // 相当于 => {x=0,y=0}={}    [0,0]
move()    // 相当于 => {x=0,y=0}={} (没传值，使用默认值) [0,0]

function move({x,y}={x:0,y:0}){
  return [x,y];
}
move({x:3, y:8)};   //  [3, 8]
move({x:3})   // [3, undefined]
move({})    // [undefined, undefined]
move()     //  [0, 0]


// 剩余参数
let arr1 = [0, 1, 2]
let arr2 = [3, 4, 5]
let arr3 = [...arr1,...arr2] //[0, 1, 2, 3, 4, 5]
...arr1 =>  0,1,2

function f(...res){
  console.log(res)    //  [-1,0,1,2,3]
  console.log(...res) //	-1,0,1,2,3
}
let args = [0,1]
f(-1,...args,2,...[3])  // f(-1,0,1,2,3)

let s = "hello"
console.log(s.split(""))
// es6写法
[..."hello"] // ["h","e","l","l","o"]


let o1 = {id:1}
let o2 = {name:2}
let o3 = {...o1, ...o2}  // {id:1, name:2}
let o4 = {...o1, o2}  // {id:1, o2:{name:2}}

let fun = () => 123
let obj4 = {...o3,fun}  // {id:1,name:2,fun}

//解构并不要求变量必须在解构表达式中声明。不过，如果是给事先声明的变量赋值，则赋值表达式必须包含在一对括号中：
let personName, personAge;
let person = {
 name: 'Matt',
 age: 27
};
({name: personName, age: personAge} = person);
console.log(personName, personAge); // Matt, 27


```

![ES6语法](../image/前端笔记/22.jpg)

### 常见的三种高阶函数

```js
//常见并且好用的三种高阶函数  filter / map / reduce
//用法如下：

let ss = [1, 4, 5, 6, 6, 3]
//需求一：要求筛选出所有小于5的数字
//碰见这种带有过滤类型的要求时，我们可以使用filter()过滤函数
console.log(ss) // [1,4,5,6,6,3]
ss = ss.filter(function (n) {
  return n < 5 //返回值为Boolean值，也就是为真时，打印输出
}) //filter过滤函数传入是一个回调函数
console.log(ss) // [1,4,3]

//需求二：要求将筛选后所有的数字乘2
//碰见这种改变原来数值的要求，我们可以时map()函数
ss = ss.map(function (n) {
  return n * 2 //返回值为改变后的数值
}) //map函数传入的是一个回调函数
console.log(ss) // [2,8,6]

//需求三：要求将改变后的数组进行求和
//碰见这种汇总问题，我们使用reduce()函数
ss = ss.reduce(function (a, b) {
  return a + b
}, 0) //第一个传入为回调函数，第二个传入为起始值，想加/减默认为0，相乘/除默认为1
console.log(ss) //16
//汇总总和：  return a+b  后面参数默认为0
//汇总总积：  return a*b  后面参数默认为1
//汇总总差：  return a-b  后面参数默认为0
//汇总总商：  return a/b  后面参数默认为1

//这三种函数合起来的写法为:
let ss = [1, 4, 5, 6, 6, 3]
let sum = ss
  .filter(function (n) {
    return n < 5
  })
  .map(function (n) {
    return n * 2
  })
  .reduce(function (a, b) {
    return a + b
  })
//ES6写法为：
let sum = ss
  .filter(n => n < 5)
  .map(n => n * 2)
  .reduce((a, b) => a + b)
```

### 显示当前时间、新年倒计时、过去多长时间

```js
//显示当前时间和新年倒计时
//1.显示当前时间
function fn(){
  let data = new Date()
  let h = data.getHours()<10?"0"+data.getHours():data.getHours()
  let f = data.getMinutes()<10?'0'+data.getMinutes():data.getMinutes()
  let s = data.getSeconds()<10?'0'+data.getSeconds():data.getSeconds()									console.log(h+":"+f+":"+s)
  requestAnimationFrame(fun)
}
fn()
//2.新年倒計時
function fn(){
  let data = new Date("2022,2,1") - new Date()
  let day = data/(24*60*60*1000)|0 //显示剩余天数
  let h = data/(60*60*1000)%24<10?"0"+(data/(60*60*1000)%24|0):data/(60*60*1000)%24|0 //显示一天中剩余的小时
  let f = data/(60*1000)%60>10?"0"+(data/(60*1000)%60|0):data/(60*1000)%60  //显示剩余的分钟
  let s = data/1000%60<10?'0'+(data/1000%60|0):data/1000%60|0  //显示剩余的秒数
  console.log(dat+":"+h+":"+f+":"+s)  requestAnimationFrame(fn)
}
fn()
//过去多久
function fn(){
  let data = new Date() - new Date("2019,5,1")
  let day = data/(24*60*60*1000)|0
  let h = data/(60*60*1000)%24<10?'0'+(data/(60*60*1000)%24|0):data/(60*60*1000)%24|0
  let f = data/(60*1000)%60<10?"0"+(data/(60*1000)%60|0):data/(60*1000)%60|0
  let m = data/1000%60<10?'0'+(data/1000%60|0):(data/1000%60|0)
  console.log(day+"天"+h+"小时"+f+"分"+s+"秒")
  requestAnimationFrame(fn)
}
fn()
```

### 请求动画帧

```js
//请求动画帧
//在js中，有两种可以做一直改变的方法，分别是定时器和请求动画帧，在关闭请求动画帧时，要先定义动画帧
let a = 1
function fn() {
  a++
  console.log(a, '请求动画帧')
  let time = requestAnimationFrame(fn) //开启请求动画帧
  if (a > 10) {
    cancelAnimationFrame(time) //关闭请求动画帧
  }
}
fn()
```

### 解除鼠标绑定

```js
//在原生中，解除鼠标绑定事件
box.onmousedown = function(){  //鼠标按下时操作
  document.onmousemove = function(){    //鼠标移动时的操作
  }
  document.onmouseup = function(){
    document.onmousedown = null
    document.onmousemove = null    //当鼠标松开时，解除鼠标的摁下和移动事件
  }
}

//在jQuery中，解除鼠标绑定事件
$(".box").mousedown(function(){  //鼠标摁下事件
  $(window).mousemove(function(){    //鼠标移动事件
  }
)
$(window).mouseup(()=>{
  $(window).unbind("mousedown").unbind("mousemove")
  }
)  //鼠标解除事件})
```

### 浏览器窗口事件

```js
//原生方法中，获取窗口的大小
let winw = window.innerWidth
let winw = document.documentElement.clientWidth
//获取窗口的宽度
let winh = window.innerHeight
let winh = document.documentElement.clientHeight
//获取窗机的高度

//jQuery中获取窗口的大小
let winw = $(window).width()
let winh = $(window).height()
//原生方法中，改变窗口大小动态获取当前窗口的宽和高
window.onresize = function () {
  let winw = window.innerWidth
  let winh = window.innetHright
}
//在jQuery方法中，改变窗口大小动态获取当前窗口的宽和高
$(window).resize(function () {
  let winw = $(window).width()
  let winh = $(window).height()
})
```

### 文本代码片段

```js
//文本代码片段 常见用法为给某个容器里面添加多个重复的元素，如果使用for循环对其进行遍历添加，会使网页一直重排，使得性能降低，但是使用文本代码片段，只需要重排一次，原理为，先将添加的所有代码插入到文本代码片段，然后再将文本代码片段插入到需要插入的地方
document.createDocumentFragment() //文本代码片段 使用appendChild()来添加其他元素，然后再使用appendChild将其添加到其他的元素上面
let box = document.querySelector('.box')
//判断可以添加的个数
let len = ((box.offsetWidth / 80) | 0) * ((box.offsetHeight / 50) | 0)
let meet = document.createDocumentFragment() //声明代码块片段
for (let i = 0; i < len; i++) {
  let p = document.createElement('p')
  meet.appendChild(p)
}
box.appendChild(meet)
```

### 事件源对象

```js
//事件源对象，用于获取在同一个容器下多个子元素的dom事件 ，例如有一个div元素，里面有多个p标签，我们需要点击每一个p标签，为其添加一个类名，从而使它改变样式，我们之前一直使用querySelectorAll来获取多个元素，使用大量的dom获取会影响性能，因此我们可以使用事件源对象，绑定给他们的父级，然后找到点击的那一个元素即可
// 利用 nodeName 可以精准定位到某一元素
let box = document.querySelector('.box')
box.addEventListener('click', fun)
function fun(even) {
  if (even.target.nodeName === 'P') {
    even.target.classList.add('active')
  }
}
```

### 获取弧度制计算三角函数

```js
let s = Math.cos((num * Math.PI) / 180) * r
```

### 二级事件对象

```js
//dom事件分为，0级事件，2级事件和3级事件
box.onclick = function () {
  console.log('0级事件')
}
box.addEventListener(
  'click',
  function () {
    console.log('2级事件') //2级事件和3级事件是一样的写法，只不过3级事件是2级事件的优化
  },
  false
)
//2级事件中第一个传参为事件名，不需要添加on，第二个参数为执行的方法，第三个参数为布尔值，当值为true时，代表是捕获阶段，当值为false时，代表为冒泡阶段true //捕获阶段  false //冒泡阶段  （默认）
box.addEventListener(
  'click',
  function (even) {
    even.stopPropagation() //阻止冒泡事件
  },
  false
) //冒泡事件的布尔值为false，可不写
box.addEventListener(
  'click',
  function (even) {
    even.stopImmediatePropagation() //阻止捕获事件
  },
  true
) //冒泡事件的布尔值为true，可不写
even.preventDefault() //阻止默认事件
even.returnValue = false //阻止默认事件
even.stopPropagation() //阻止冒泡事件
even.stopImmediatePropagation() //阻止捕获事件
```

### 事件解除绑定

```js
//事件的解除绑定 （鼠标拖拽事件，当抬起时停止移动）
//原生js 0级事件解除绑定
box.onmousedown = function () {
  console.log('鼠标摁下了')
  document.onmousemove = function () {
    console.log('开始移动')
  }
}
document.onmouseup = function () {
  console.log('鼠标抬起')
  document.onmousedown = null //解除事件绑定
  document.onmousemove = null //解除事件绑定
}
//原生js 2级事件解除绑定
box.addEventListener(
  'mousedown',
  function () {
    //清除函数
    let removeDom = null
    console.log('鼠标摁下')
    document.addEventListener(
      'mousemove',
      function (even) {
        //获取自身匿名函数
        removeDom = arguments.callee
        console.log('鼠标移动')
      },
      false
    )
    document.addEventListener(
      'mouseup',
      function () {
        //清除鼠标移动事件
        document.removeEventListener('mousemove', removeDom)
        console.log('鼠标已抬起')
      },
      false
    )
  },
  false
)
// jQuery事件解除绑定
$('box').mousedown(() => {
  $(window).mousemove(() => {
    $(window).mouseup(() => {
      $(window).unbind('mousedown').unbind('mousemove')
    })
  })
})
```

### this 绑定

```js
//this绑定对象，从而改变this的指向
//三种绑定分别为：
函数.call(this)
函数.apply(this)
函数.bind(this)
let a = {
  name: 'web前端工程师',
}
document.addEventListener(
  'click',
  function () {
    console.log(this) //{name:'web前端工程师'}
  }.bind(a)
)

// 三种绑定的区别
//call 和 apply会立即执行，然后bind不会立即执行。
//bind call 可以传任何数据  apply只能传递数组类型
//例如，给上面的点击函数绑定了call和apply之后，函数在不点击的时候就已经执行完毕，这样点击之后就没有任何的效果了。而bind绑定会在点击时才会触发

// 立即执行函数的this指向为window (正常情况下)
// 但是下面这种情况指向并非window
const ss = {
  name: 'WEB架构师',
  age: 20,
}
function sum() {
  ;(() => {
    console.log(this) // ss对象
  })()
}
sum.call(ss)
```

### 常见的兼容问题

```js
//常见的几种需要兼容的问题
// 问题一： 兼容事件对象event
// 如下：box.onclick = function(even){ even = even || window.even }
// 注意：在二级事件中，不要兼容event，因为二级事件IE8及以下的浏览器本来就不支持，所以无须多此一举
// 问题二： 兼容获取浏览器窗口大小
// 将此封装为成一个方法为：
function winsize() {
  if (window.innerWidth) {
    return {
      winw: window.innerWidth,
      winh: window.innerHeight,
    }
  } else {
    return {
      winw: document.documentElement.clientWidth,
      winh: document.documentElement.clientHeight,
    }
  }
}
```

### 固定元素水平居中

```css
position: fixed;
top: 0;
left: 50%;
transform: translateX(-50%);
```

### js 检测类型的三种方法

```js
// 判断js类型的三种方法，分别是：typeof instanceof  Object.prototype.toString.call
// typeof使用
// typeof （要判断的值）  返回类型
let arr = [1, 2, 3]
let str = 'web前端'
let obj = { name: 'web工程师' }
let num = 1
let fun = function () {}
typeof arr //object    (💛)
typeof str //string
typeof obj //object
typeof num //number
typeof null //object    (💛)
typeof 12n //bigint
typeof undefined //undefined
typeof true //boolean
typeof fun //function

// instanceof使用
// （要判断的值） instanceof 类型 返回布尔
arr instanceof Array //true
arr instanceof Object //true
arr instanceof String //false
fun instanceof Function //true

// Object.prototype.toString.call()使用
// Object.prototype.toString.call(要判断的值)
Object.prototype.toString.call(arr) //"[object Array]"
Object.prototype.toString.call(true) //"[object Boolean]"
Object.prototype.toString.call(null) //"[objcet Null]"       (💛)
Object.prototype.toString.call(obj) //"[objcet Object]"
Object.prototype.toString.call(num) //"[objcet Number]"
Object.prototype.toString.call(undefined) //"[objcet Undefined]"
Oject.prototype.toString.call(fun) //"[object Function]"
```

### 数组降维

```js
// 数组将维，arr.flat() 里面写传入数字参数，对应维数，当为Infinity时，降为一维
let arr = [1, [2, 4], 23, 4, [1, 23, [1, 4, 5]], 1, 24, 2]
console.log(arr) //[1, Array(2), 23, 4, Array(3), 1, 24, 2]
arr = arr.flat(Infinity)
console.log(arr) //[1, 2, 4, 23, 4, 1, 23, 1, 4, 5, 1, 24, 2]
//递归降维
let arr = [
  1,
  2,
  [1, [4, 6, [2, 5, 1], 3, 3], 5],
  4,
  0,
  [2, 3],
  7,
  23,
  7,
  2,
  8,
  67,
  23,
]
let newArr = []
function fun(arr) {
  if (!Array.isArray(arr)) return
  while (arr.length) {
    let value = arr.shift()
    if (Array.isArray(value)) {
      fun(Value)
    } else {
      newArr.push(value)
    }
  }
}
fun(arr)
console.log(newArr) //[1, 2, 1, 4, 6, 2, 5, 1, 3, 3, 5, 4, 0, 2, 3, 7, 23, 7, 2, 8, 67, 23]
```

### es6 中的按需加载，节省性能

```js
//es6中的按需加载，可以节省性能，不会造成没必须的浪费，例如需要使用Math对象中的abs()方法，但是如果全部直接使用，会加载一些并不用到的方法。
let { abs } = Math
console.log(abs(-1)) //1
```

### 获取元素的样式

```js
//原生js方法：
//假如我们需要获取输入一断话的长度，为这段话的字体大小*这段话的字数
window.getComputedStyle(box)  => box:要获取的元素
//获取字体大小：
window.getComputedStyle(box).fontSize   //16px
//变为纯数字为:
parseInt(window.getComputedStyle(box).fontSize) //16
//获取元素其他的css属性类似
//jQuery方法：
$(dom).css("要获取的样式");  dom => 要获取的元素
//获取字体大小：
$(dom).css("font-size")
```

### 函数节流和防抖

```js
//节流：在规定时间内，只执行第一次
//防抖:在规定时间内，只执行最后一次 （多用于键盘事件）
//节流：
//封装函数写法：
function s(fn,time){
  let start = 0;
  return function(){
    let data = new Date;
    if(data - start>time){
      fn();
      start = data;
    }
  }
}
let fun = s(function(){
  console.log("这里是执行代码")}，1000);
document.addEventListener('click',fun,false);
//简易写法(推荐):
let start = 0;
document.addEventListener('click',function(){
  let data = new Date;
  if(data - start>1000){
    fun();
    start = data;
  }
},false)
function fun(){
  console.log("这里是执行函数");
}
//防抖
//函数写法：
function fun1(fn,time){
  let start = null;
  return function(){
    clearTimeout(start);
    start = setTimeout(fn,time)
  }
}
let fun = fun1(function(){
  console.log("这里是执行函数");
},1000)
document.addEventListener('click',fun,false);
//简易写法:
let start = null;
document.addEventListener('click',function(){
  clearTimeout(start);
  start = setTimeout(function(){
    console.log("这里是执行代码")
  },1000)
},false);
```

### 正则表达式

```js
//includes用法，str.includes("要匹配的内容");
let str = "下面介绍正则表示式的用法。"
console.log(str.includes("正则")) // true
console.log(str.includes("java"))// false

<!--------------------------------------------------------->
let str = "下面介绍正则表达式的用法。"
// 正则匹配的两种写法：字面量和构造函数
// 方法一： 字面量 (推荐写法)
let reg = /正则表达式/;

//方法二：构造函数写法：
let reg1 = new RegExp("正则表达式");
//正则方法：
console.log(reg.test(str))  //true
//检测字符串中是否含有正则里面的内容 ，返回 true 和 falseconsole.log(reg.exec(str))  //['正则表达式']//检测字符串中是否含有正则匹配的内容，有就返回匹配的内容，没有就返回null//字符串方法使用正则匹配console.log(str.match(reg))  //['正则表达式']//查找字符串中是否含有正则匹配的内容，有返回匹配结果数组，没有返回nullconsole.log(str.split(reg))  //['下面介绍','的用法']  //按照正则匹配规则进行切割  console.log(str.replace(reg,"web前端")) //下面介绍web前端的用法  //找到匹配的正则并替换为对应的字符串console.log(str.search(reg))  //4//返回正则匹配的位置，没有找到则返回-1//修饰符i   //不区分大小写进行匹配m   //换行匹配g   //全局匹配，查找整个字符串//范围匹配简单类   /[里面任意值]/   //匹配里面所有的字符范围类   /[a-z]/i   //到z的所有字符,不区分大小写中文类   /[\u4e00-\u9fa5 ]/   //匹配所有的中文组合类   /[a-z0-9]/i       //匹配所有的字母和数字负向类   /[^]/       //取反，取不在里面的内容//界定符^   /^1[0-9]/g //匹配以1开头的数字$   /[0-9]5$/g //匹配以5结束的数字.   /./  //匹配任意字符//元字符  \d    [0-9]    //数字字符\D    [^0-9]   //非数字字符\w    [a-zA-Z0-9]  //数字字母\W    [^a-zA-Z0-9] //非数字字母\s    [\r\t\n\f]   //空白字符\S    [^\r\t\n\f]  //非空白字符//包含范围 `{1,}`  可由 `+` 代替`{0,}`  可由 `*` 代替`{0,1}`  可由 `?` 代替  let str = '2021年Web前端工程师在学习Js'let reg  = /[a-z0-9]/ig; //匹配所有的数字和字母，并且全局匹配，不区分大小写console.log(str.match(reg)) //['2','0','2','1','W','e','b','J','s']let reg1 = /[\u4e00-\u9fa5 ]/g; //匹配所有的中文汉字，并且全局匹配console.log(str.match(reg1)) //["年", "前", "端", "工", "程", "师", "在", "学", "习"]let reg2 = /[^a-z]/ig;  // 取反，取不在正则里面的内容，如全局匹配非字母的字符集console.log(str.match(reg2)) //["2", "0", "2", "1", "年", "前", "端", "工", "程", "师", "在", "学", "习"]// /^.*/  贪婪匹配 匹配所有内容
```

### 分别获取形参个数和实参个数

```js
arguments.length // 获取实参个数
arguments.callee.length // 获取形参个数
fun.length // 获取形参的个数
function fun(a, b) {
  console.log(arguments.length) //3
  console.log(arguments.callee.length) //2
}
fun(1, 2, 3)
```

### 原型链继承

```js
function Fater() {
  this.name = '我是父类'
  this.eat = () => {
    return this.name + '喜欢吃汉堡'
  }
}
function Son() {
  this.name = '我是子类'
  this.eat = () => {
    return this.name + '喜欢吃零食'
  }
}

Son.prototype = new Fater()
let s = new Son()
console.log(s, '子类信息')
console.log(s.__proto__, '父类信息')
console.log(s.name, '调用子类的信息')
console.log(s.eat(), '调用子类的方法')
console.log(s.__proto__.name, '调用父类的信息')
console.log(s.__proto__.eat(), '调用父类的方法')
```

### 全屏显示

```js
box.requestFullScreen() //元素全屏显示
document.CancelFullScreen() //取消全屏显示
document.webkitIsFullScreen() //判断元素是否全屏显示
```

### 网络状态

```js
//online  从断网到联网时触发
//onoffline  从联网到断网时触发
window.online = () => {
  console.log('网络已重新连接', navigator.onLine) // true
}
window.onoffline = () => {
  console.log('网络连接已断开', navigator.offLine) // false
}

// navigator.onLine 判断网络是否连接，
```

### H5 读取文件

```js
;<input type="file"></input>
box.change = function () {}
```

### css 去除父级继承的样式

```js
all: initial //去除父级继承来的样式
```

### 移动端事件

```js
//移动端事件分为四类：分别是click事件、touch类事件、tap类事件、swipe事件。// click事件（单击事件）：click //类似于PC端的click事件,但在移动端中，连续click的触发有200ms ~ 300ms的延迟//touch类事件（触摸事件）：touchstart 	// 手指触摸到屏幕会触发touchmove  	//当手指在屏幕上移动时触发touchend   	//当手指离开屏幕时触发touchcancel //可由系统进行的触发，比如手指触摸屏幕的时候，突然alert了一下，或者系统中其他打断了touch的行为，则可以触发该事件//tap类事件（触碰事件）：tap     		//手指碰一下屏幕会触发longTap  		//手指长按屏幕会触发singleTap  	//手指碰一下屏幕会触发doubleTap   //手指双击屏幕会触发//swipe类事件（滑动事件）：swipe       //手指在屏幕上滑动时触发swipeLeft   //手指在屏幕上向左滑动时会触发swipeRight  //手指在屏幕上向右滑动时会触发swipeUp     //手指在屏幕上向上滑动时会触发swipeDown   //手指在屏幕上向下滑动时会触发
```

### 移动端视口处理

```js
//用来解决不同设备下，展现出一样的效果
;(() => {
  //采用IIFE，防止变量污染
  function winSize() {
    //获取视口宽度
    let width = window.screen.width
    //获取文档
    let dom = document.documentElement
    //设置字体大小,以iphone6为样板，设置倍率为100
    dom.style.fontSize = width / 3.75
  }
  winSize() //浏览器大小改变时，字体跟着改变
  window.addEventListener(
    'resize',
    () => {
      winSize()
    },
    false
  )
})()
```

### 移动端左右拖拽临界问题

```js
// 左右拖拽  有拖拽 （最大值）
//拖拽长度为视口宽度两倍时，最大拖拽限制为视口的一倍，拖拽长度为视口宽度的三倍时，最大拖拽限制为视口的两倍(假设视口宽度为3.75rem)
//获取元素
let dom = document.querySelector("dom")
//视口宽度
let winW = window.screen.width
//动态变化
window.onresize = ()=>{
  winW = window.screen.width
}
//视口宽度比例
let domW = dom.offsetWidth/winW
//设置临界状态
if(initMoveX().result<-(domW-1)*width){
  dom.style.transform = `translateX(${-(domW-1)*width}px)`
}
拖拽元素长度        最大限制拖拽元素长度
7.5(2winW)  rem     winW11.25(3winW)  rem
2winW15(4winW)    rem       3winWx/w   rem
(x/w-1)winW...       ...
//滚动条移动
//滚动条移动比例： 拖拽元素长度/(滚动条总宽度+滚动元素的宽度)
//滚动条移动距离：比例*移动距离
//获取滚动条
let barW = (barBanner.clientWidth-bar.clientWidth) / width
let pro = banner.clientWidth / (barBanner.clientWidth + bar.clientWidth)
bar.style.transform = `translateX(${movex.barresult-s/pro}px)`
//临界处理
if(movefunX().result>0){
  banner.style.transform = `translateX(0px)`
  bar.style.transform = `translateX(0px)`
}  if(movefunX().result<-(bannerW-1)*width){
  banner.style.transform = `translateX(${-(bannerW-1)*width}px)`
  bar.style.transform = `translateX(${(barW)*width}px)`
}
```

### unicode 编码互相转换

```js
//将文字转化为unicode编码
encodeURIComponent('新新小朋友') //"%E6%96%B0%E6%96%B0%E5%B0%8F%E6%9C%8B%E5%8F%8B"
//将unicode编码转化为文字
decodeURIComponent('%E6%96%B0%E6%96%B0%E5%B0%8F%E6%9C%8B%E5%8F%8B') //新新小朋友
//en编码  de解码
```

### 本地存储

```js
//本地存储分别长期存储和短期存储
//不同点：长期存储指的是手动将其删除的时候才会被清除，短期存储是指页面关闭的时候或者是手动删除的时候会被删除。
//localStorage存储在浏览器中的local Storage中
//sessionStorage存储在浏览器中的session Storage中

//共同点：
//它们都是一个键值对，存储对象和数组的时候需要将它们转化为JSON格式。如下：
//存值：
JSON.stringify(value)
//localStorage.setItem("arr", JSON.stringify([1,2,3]))
//取值：
JSON.parse(value)
//JSON.parse(localStorage.getItem("arr"))
//可以不同的key值存储多个，如果一个key存储多个的话，后面将会把前面的进行覆盖

1 // 长期存储localStorage
localStorage.setItem('name', '新新小朋友')
//存储数据
localStorage.getItem('name')
//取值 （新新小朋友）
localStorage.removeItem('name') //删除名为name的数据
localStorage.clear() //将本地的所有数据进行清空
2 //短期存储
sessionStoragesessionStorage.setItem('key', 'value') //存储数据
sessionStorage.getItem('key') //取值
sessionStorage.clearItem('key') //删除名为key的数据
sessionStorage.clear() //将本地的所有数据进行清空
// 本地存储的数据自动登录
// 获取所有的本地存储的数据
let data = localStorage
if (data.length) {
  // 获取登录的最后一个用户名
  let name = data.key(data.length - 1) // 获取密码
  let pass = localStorgae.getItem(name)
}
```

### Cookie

```js
// 添加cookie值 第一个值为属性，第二个为到期时间
document.cookie = 'name=新新小朋友;max-age=3600'

// 获取cookie值
function fn(key) {
  let cookie = document.cookie
  let reg = new RegExp(`(^|\\s)${key}=([^;]+)(;|$)`)
  reg.test(cookie)
  return RegExp.$2
}
fn('age')

// 删除cookie
document.cookie = 'name=1;max-age=-1'

// 整体封装写法
const cookie = {
  // 批量添加cookie值  默认窗口关闭时清除
  set(obj, day) {
    for (let [value, key] of Object.entries(obj)) {
      console.log(value, key)
      document.cookie = `${value}=${key};max-age=${day * 3600 * 24}`
    }
  },
  // 获取cookie的值
  get(key) {
    new RegExp(`(^|\\s)${key}=([^;]+)(;|$)`).test(document.cookie)
    return RegExp.$2
  },
  // 清除cookie
  remove(key) {
    this.set(
      {
        [key]: '',
      },
      -1
    )
  },
}
```

### 常见的浏览器事件

```js
// 前进
window.location.href = history.go(1)
window.history.go(1)
// 后退
window.location.href = history.go(-1)
window.history.go(-1)
window.location.href = history.back()
window.history.back()
// 网页刷新
loaction.reload()
// 网页关闭
window.close()
//禁止网页前进
window.history.forward(1)
// 禁止网页后退
window.history.forward(-1)
// 新页面打开
window.open('url')
```

### Buffer 操作

```js
Buffer.from(str)  //将字符串转化为
bufferBuffer.alloc(size) //创建一个指定大小的
bufferBuffer.alloUnsafe(size)  //创建一个指定大小的
buffer，但里面可能包含数据buf.toString()   //将缓冲区的数据转化为字符串
```

## 高级 JavaScript

### this

```js
1. 函数在调用时，JavaScript会默认给this绑定一个值
2. this的绑定和定义的位置（编写的位置）没有关系
3. this的绑定和调用方式以及调用的位置有关
4. this时运行时被绑定的

// this规则之外
如果在显示绑定中，传入一个null或者undefined，那么这个显示绑定会被忽略，使用默认规则
foo.call(null) // window

// 箭头函数没有this，因此它会先从自己的作用域查找this，自己作用域没有this时，它会找父级作用域，直到找到全局的作用域，需要注意的是，obj是一个对象，对象没有作用域，只有 块级 和 函数 存在作用域
 const obj = {
   name: 'web',
   foo: function () {
     return function () {
       console.log('this', this) // window
     }
   },
   foo1: function () {
     return () => {
       console.log('this', this) // obj
     }
   },
   foo2: () => {
     return () => {
       console.log('this', this) // window
     }
   },
 }

 let a = obj.foo1()
 a()
```

### 回流和重绘

#### 回流

- 回流 reflow，也被称为重排
  - 第一次确定节点的大小和位置，称之为布局
  - 之后对节点的大小、位置修改重新计算称之为回流
- 什么情况下引起回流？
  - 比如 DOM 结构发生了改变（添加新的节点或移除节点）
  - 比如改变了布局（修改了 width、height、padding、font-size 等）
  - 比如窗口 resize（修改了窗口的尺寸）
  - 比如调用 getComputedStyle 方法获取尺寸、位置信息
- 回流会引起特别大的开销，比较浪费性能
- 回流一定会引起重绘

#### 重绘

- 第一次渲染内容称为绘制

- 之后重新渲染称之为重绘

- 什么情况会引起重绘

  - 比如修改背景色、文字颜色、边框颜色、样式等

### 类型比较隐士转换

```js
1. NaN和任何人都不相等，包括自身
2. bigInt,Symbol，首先比较是不是同一个类型，不是同一个类型肯定不相等
3. null,undefined，它两只跟自身或者对方相等
4. 布尔值和其他类型比较，布尔根值会转化为数字
5. 数字类型和字符串类型比较，字符串会转化为数字
6. 对象类型和原始类型比较，对象类型会转化为原始类型，两个对象比较，比较的是他们的引用

null == undefined   (√)
null == null        (√)
null === undefined  (×)
null === null       (√)
"0" == false        (√)
"0" === false       (×)
NaN == NaN          (×)
NaN == "NaN"        (×)
NaN == '213a'       (×)
NaN != NaN          (√)
NaN !== NaN         (√)
undefined === undefined (√)
undefined == 0      (×)
null == 0           (×)
```

### 函数式编程和面向对象编程

```js
// 函数式
优点：JavaScript中函数是一等公民，便于拆分组合和扩展性好，方便tree-shaking
缺点：管理难度大，复杂逻辑难以组织，模块难以划分

//  面向对象式
优点：模块分明，逻辑清晰，方便组织庞大业务
缺点：不好配合tree-shaking，JavaScript对于面向对象实现不完美
```

### 迭代器与生成器

```js
// 迭代器
// 在JavaScript中，计数循环就是一种最基本的迭代
for (let i = 0; i < 4; i++) {
  console.log(i)
}
// 循环是迭代机制的基础，这是因为它可以指定迭代的次数，以及每次迭代要执行什么操作。每次循环都会在下一次迭代开始之前完成，而每次迭代的顺序都是事先定义好的。
// 在 ECMAScript 中，这意味着必须暴露一个属性作为“默认迭代器”，而且这个属性必须使用特殊的 Symbol.iterator 作为键。这个默认迭代器属性必须引用一个迭代器工厂函数，调用这个工厂函数必须返回一个新迭代器。

// 内置Iterable接口的类型：
// 字符串
// 数组
// Map
// Set
// agritainment对象
// NodeList等DOM集合类型

// 检查是否存在默认迭代器属性可以暴露这个工厂函数：
obj[Symbol.iterator]

let num = 1;
let obj = {};
// 这两种类型没有实现迭代器工厂函数
console.log(num[Symbol.iterator]); // undefined
console.log(obj[Symbol.iterator]); // undefined

let str = 'abc';
let arr = ['a', 'b', 'c'];
let map = new Map().set('a', 1).set('b', 2).set('c', 3);
let set = new Set().add('a').add('b').add('c');
let els = document.querySelectorAll('div');
// 这些类型都实现了迭代器工厂函数
console.log(str[Symbol.iterator]); // f values() { [native code] }
console.log(arr[Symbol.iterator]); // f values() { [native code] }
console.log(map[Symbol.iterator]); // f values() { [native code] }
console.log(set[Symbol.iterator]); // f values() { [native code] }
console.log(els[Symbol.iterator]); // f values() { [native code] }
// 调用这个工厂函数会生成一个迭代器
console.log(str[Symbol.iterator]()); // StringIterator {}
console.log(arr[Symbol.iterator]()); // ArrayIterator {}
console.log(map[Symbol.iterator]()); // MapIterator {}
console.log(set[Symbol.iterator]()); // SetIterator {}
console.log(els[Symbol.iterator]()); // ArrayIterator {}

// 实现可迭代协议的所有类型都会自动兼容接收可迭代对象的任何语言特性。接收可迭代对象的原生语言特性包括：
// for-of 循环
// 数组解构
// 扩展操作符
// Array.from()
// 创建集合
// 创建映射
// Promise.all()接收由期约组成的可迭代对象
// Promise.race()接收由期约组成的可迭代对象
// yield*操作符，在生成器中使用

// 由于正常情况下 对象和number类型是不支持迭代，也就是不支持for...of循环，但是我们可以通过往它们身上添加 Symbol.iterator 方法来实现迭代效果

// 对象实现 迭代
const obj = {
  name: 'web',
  age: 23,
  // 方案二：
  [Symbol.iterator]() {
    let len = Object.keys(obj).length
    let i = 0
    return {
      next() {
        if (i < len) {
          const key = Object.keys(obj)[i]
          i++
          return { done: false, value: obj[key] }
        } else {
          return { done: true, value: undefined }
        }
      }
    }
  },
  // 方案二：
  [Symbol.iterator]() {
    let i = 0
    const keys = Object.keys(this)
    return {
      next: () => {
        return {
          value: { key: keys[i], value: this[keys[i]] },
          done: i++ > keys.length - 1 ? true : false
        }
      }
    }
  }
}

// 迭代遍历
for (const item of obj) {
  console.log(item) // { name: 'web' }, { age: 23 }
}


// Number类型实现迭代
const num = new Number(12)
num[Symbol.iterator] = function() {
  let i = 0
  return {
    next() {
      if (i < num) {
        return { done: false, value: ++i }
      } else {
        return { done: true, value: undefined }
      }
    }
  }
}

for (const n of num) {
  console.log(n) // 1~12
}



// 生成器
// 生成器是 ECMAScript 6 新增的一个极为灵活的结构，拥有在一个函数块内暂停和恢复代码执行的能力。这种新能力具有深远的影响，比如，使用生成器可以自定义迭代器和实现协程。
// 生成器的形式是一个函数，函数名称前面加一个星号（*）表示它是一个生成器。只要是可以定义函数的地方，就可以定义生成器。

// 生成器函数声明
function* generatorFn() {}
// 生成器函数表达式
let generatorFn = function* () {}
// 作为对象字面量方法的生成器函数
let foo = {
 * generatorFn() {}
}
// 作为类实例方法的生成器函数
class Foo {
 * generatorFn() {}
}
// 作为类静态方法的生成器函数
class Bar {
 static * generatorFn() {}
}
·
// 箭头函数不能用来定义生成器函数

// yield 关键字有点像函数的中间返回语句，它生成的值会出现在 next()方法返回的对象里。
// 通过 yield 关键字退出的生成器函数会处在 done: false 状态；通过 return 关键字退出的生成器函数会处于 done: true 状态。
function* generatorFn() {
 yield 'foo';
 yield 'bar';
 return 'baz';
}
let generatorObject = generatorFn();
console.log(generatorObject.next()); // { done: false, value: 'foo' }
console.log(generatorObject.next()); // { done: false, value: 'bar' }
console.log(generatorObject.next()); // { done: true, value: 'baz' }

// 生成器函数内部的执行流程会针对每个生成器对象区分作用域。在一个生成器对象上调用 next()不会影响其他生成器：
function* generatorFn() {
 yield 'foo';
 yield 'bar';
 return 'baz';
}
let generatorObject1 = generatorFn();
let generatorObject2 = generatorFn();
console.log(generatorObject1.next()); // { done: false, value: 'foo' }
console.log(generatorObject2.next()); // { done: false, value: 'foo' }
```

### v8 引擎

```ts
// 64位下是1.4G
// 32位下是700MB
// 但是根据浏览器不同，有些许扩容，node情况下会有一些c++内存扩容

// 内存分为两块 新生代 老生代

// 新生代
短期存活的变量的会存在新生代中，新生代的内存量极小，64位下大概是32MB

// 老生代
生存时间比较长的变量，会转存到老生代，老生代占据了几乎所有内存，64位下大概是1400MB


新生代的回收算法： 复制-清空，牺牲空间换时间

老生代回收分为：
1. 标记已死变量
2. 清除已死变量
3. 整理磁盘

什么时候触发回收
1. 执行完一次代码
2. 内存不够的时候

全局变量会直到程序执行完毕，才会回收。
普通变量，就是当他们失去引用的时候，才会回收

// 检测内存
// 浏览器端
window.performance.memory
// node端
process.memoryUsage()
```

```js
JavaScript代码执行顺序：先执行同步代码，然后执行微任务里面的代码，最后执行宏任务里面的代码。在执行任何的宏任务之前，都需要保证微任务队列已被清空。

宏任务： 定时器、 ajax、 DOM操作、 UI Rendering
微任务： Promise.then、 queueMicrotask


常用的GC算法——引用计数和标记清除

高阶函数：把一个函数如果接受另外一个函数作为参数，或者该函数会返回另外一个函数作为返回值的函数，那么这个函数就被成为高阶函数

this指向优先级：new绑定 > 显示绑定 > 隐式绑定 > 默认绑定
1. let p1 = new Person("新新",123)
2. sum.call(obj,123)
3. obj.doIt()
4. sum()
new绑定不能和apply/call一起使用

// 类数组转数组
arguments => array
1. arguments = Array.from(arguments)
2. arguments =   Array.prototype.slice.call(arguments) // 调用数组的slice方法，使用call指向被调用的类数组
3. arguments = [].slice.call(arguments) // 类似上面的方法，只是把Array.prototype用[]代替
4. arguments =  [...arguments]

在严格模式下 碰见错误会直接抛出或者中止程序，而在非严格模式下，有些情况下并不会抛出异常，只是会静默（不做任何处理）

// 打印该对象上的所有原型对象
console.log(Object.getOwnPropertyDescriptors(Object.prototype))
```

### Object 上的方法

```js
let obj1 = {
  name:"obj1对象"
}
let obj2 = {
  name:"obj2对象"
}
obj1.__proto__.info = "这是obj1对象"
obj2.__proto__.info = "这是obj2对象"

function Person(name, age){
  this.name = name
  this.age = age
}

Person.prototype.eating = function(){
  console.log(`${this.name}正在吃东西`)
}

const person1 = new Person("web",22)


// Object.getPrototypeOf()
// 获取对象上的原型 （[[prototype]]属性的值）
Object.getPrototypeOf(obj1) // {info:"这是obj1对象"}
Object.getPrototypeOf(person1) // {eating:f()}


// Object.getOwnPropertyDescriptor()
// 获取对象某一属性的 属性描述符
Object.getOwnPropertyDescriptor(obj, 'name')
{
  configurable: true, // 是否可以删除 默认为false
  value: 22,         // 该属性的值 默认为undefined
  enumerable: true,  // 是否可枚举，默认为false，就是既不会被for in识别，也不会显示在obj属性中
  writable: false,  // 是否可以更改，默认是false，重新赋值也无法更改
}

// Object.getOwnPropertyDescriptors()
// 获取对象上的所有*自身*属性的描述符
Object.getOwnPropertyDescriptors(obj1)
// {name:{value:"obj1对象",configurable:true,enumerable:true,writable:true}}
Object.getOwnPropertyDescriptors(person1)
// {name:{value:"web",configurable:true,enumerable:true,writable:true},age:{value:22,configurable:true,enumerable:true,writable:true}}


// Object.preventExtensions()
// 对象变得不可拓展，也就是永远不能再添加新的属性 (对象不可扩展)
Object.preventExtensions(obj)
// 新增属性无效
obj.ss = "xxx"

// Object.isExtensible()
// 判断一个对象是否可以被拓展添加新属性
Object.preventExtensions(obj)
Object.isExtensible(obj) // false


// Object.seal()
// 阻止添加新属性 + 属性标记为不可配置 （对象的封闭）
// 不可拓展，不可删除
Object.seal(obj)
obj.age = 22 // 新增无效
delete obj.name // 删除无效


// Object.isSealed()
// 判断一个对象是否是封闭对象
Object.seal(obj)
Object.isSealed(obj) // false

// Object.freeze()
// 不可添加 + 不可配置 + 不能修改 （对象的冻结）
Object.freeze(obj)
obj.age = 22 // 新增无效
obj.name = '123' // 修改无效
delete obj.name // 删除无效

// Object.isFrozen()
// 检查一个对象是否被冻结
Object.freeze(obj)
Object.isFrozen(obj) // true


// Object.setPrototypeOf()
// 设置对象原型到另一个对象  (将obj2的原型设置给obj1)
Object.setPrototypeOf(obj1,obj2)
console.log(obj1) // {name:"obj1对象"}
console.log(Object.getPrototype(obj1)) //{name:"obj2对象"}

Object.setPrototypeOf(person1,obj2)
console.log(Object.getPrototypeOf(obj1)) //{name:"obj2对象"}


// Object.create()
// 创建一个空的对象，并将现有对象的原型赋值给该对象
const obj3 = Object.create(obj2,{
	// 第二个参数传入的是属性描述符
  text:{
    value:"测试",
    enumerable: true
  }
})
console.log(obj3) // {text:"测试"}
console.log(Object.getPrototypeOf(obj3)) // {name:"obj2对象"}


// Object.keys()
// 获取该对象所有的key值
console.log(Object.keys(obj1)) // ["name"]


// Object.values()
// 获取该对象的所有value值
console.log(Object.values(obj1)) // ["obj1对象"]


// Object.entries()
// 可以获取到一个数组，数组中会存放可枚举属性的键值对数组
console.log(Object.entries(obj1)) [["name","obj1对象"]]


// Object.fromEntries()
// 将entries转为对象，也可以用来读取 formData 中的所有数据
console.log(Object.fromEntries(Object.entries(obj1))) // {name:"obj1对象"}

const formData = new FormData()
formData.append('name', 'web')
formData.append('age', 23)
console.log(Object.fromEntries(formData)) // { name: 'web', age: 23 }


// Object.assign()
// 用于将所有可枚举属性的值从一个或多个源对象分配到目标对象。它将返回目标对象，可以做浅拷贝，无法用来深拷贝，因为它只能将原来数组的所有属性和值复制了一份到新的空对象中 假如源值是一个对象的引用，它仅仅会复制其引用值
const obj = {
  name: "WEB架构师",
  age: 22,
  info: {
    name: "info"
  } (0x100), // 对象引用类型复制为地址值
  arr: [1, 2, 3] (0x200)
}
const obj1 = Object.assign({}, obj)
obj.name = "Vue"
obj.info.name = "info改变"
console.log(obj1.name) // WEB架构师
console.log(obj1.info.name) // info改变
```

![Object方法](../image/前端笔记/23.jpg)

### 寄生组合式继承

```js
// subType:子类（继承类） superType:父类（基类）
function inheritPrototype(subType, superType) {
  // 将父类的原型赋给子类的原型  原型链继承
  subType.prototype = Object.create(superType.prototype)
  // 将子类的constructor指向指向自己 默认会指父类
  Object.defineProperty(subType, 'constructor', {
    enumerable: true,
    writable: true,
    configurable: true,
    value: subType,
  })
}

// 基类
function Person(name, age) {
  this.name = name
  this.age = age
}
// 基类的公共方法
Person.prototype.runing = function () {
  console.log(`${this.name}正在跑步`)
}

// 继承类  老师类
function Teacher(name, age, title) {
  // 利用call绑定将Person的属性绑定到子类属性上
  Person.call(this, name, age)
  this.title = title
}
// 继承
inheritPrototype(Teacher, Person)
// 老师类添加方法
Teacher.prototype.loading = function () {
  console.log(`${this.title}${this.name}正在授课`)
}

// 继承类 学生类
function Student(name, age, num) {
  // 利用call绑定将Person的属性绑定到子类属性上
  Person.call(this, name, age)
  this.num = num
}
// 继承
inheritPrototype(Student, Person)
// 学生类添加方法
Student.prototype.studying = function () {
  console.log(`学号为${this.num}的${this.name}正在学习`)
}

// 实例化
const stu1 = new Student('学生1', 22, 111)
const tea1 = new Teacher('老师1', 42, '高级讲师')

console.log(stu1) // Student {name:"学生",age:22,num:111}
console.log(tea1) // Teacher {name:"老师1",age:42,title:"高级讲师"}
stu1.runing() // 学生1正在跑步
tea1.loading() // 高级讲师老师1正在授课
console.log(Object.getOwnPropertyDescriptors(stu1)) // {name:{value:"学生1",writable:true,enumerable:true,configurable:true},age:{value:22,writable:true,enumerable:true,configurable:true},num:{value:111,writable:true,enumerable:true,configurable:true}}
```

### 对象方法补充

```js
const obj = {
  name: 'obj',
  age: 22,
}

obj.__proto__ = {
  type: '原型链',
}

const info = Object.create(obj)

// 基类
function Fun() {}
// 继承类
function Son() {}
// 调用上面的继承方法
inheritPrototype(Son, Fun)
const son = new Son()

// hasOwnProperty
// 检测对象上是否含有某一个属于自己的属性（不是在原型上的属性）
console.log(obj.hasOwnProperty('name')) // true
console.log(obj.hasOwnProperty('type')) // false

// in
// 检测对象上是否含有某一属性，包括原型上的属性
console.log('age' in obj) // true
console.log('type' in obj) // true

// instanceof
// 用于检测*构造函数*的prototype,是否出现在某个实例对象的原型链上
console.log(son instanceof Son) // true
console.log(son instanceof Fun) // true
console.log(son instanceof Array) // false

// isPrototyoeOf
// 用于检测*某个对象*，是否出现在某个实例对象的原型链上
console.log(obj.isPrototyoeOf(info)) // true
console.log(obj instanceof info) // TypeError
```

### 对象的隐式转换

```js
// 对象的隐式转化，需要看里面是否含有valueOf属性和toString属性，如果有的话，优先使用里面的值
// 对于字符串转换，优先调用toString
// 对于数学运算，优先调用valueOf

// 默认情况下，普通对象具有 toString 和 valueOf 方法：
// toString 方法返回一个字符串 "[object Object]"
// valueOf 返回对象自身  obj.valueOf() === obj

xxx[obj] // 优先取obj.toString()的值，没有则返回 [object Object]
`${obj}` // 优先取obj.toString()的值，没有则返回 [object Object]
obj + 1 // 优先取obj.valueOf()的值，然后取obj.toString()的值，没有则返回 [object Object]
obj * 1 // 优先取obj.valueOf()的值，然后取obj.toString()的值，没有则返回 [object Object]
obj == 1 // 优先取obj.valueOf()的值，然后取obj.toString()的值，没有则取 [object Object]

const obj = {
  value: 10,
  toString() {
    return this.value + 10
  },
  valueOf() {
    return this.value
  },
}

obj[obj] = obj.value
console.log('keys:', Object.keys(obj)) // ['20', value, toString, valueOf]
console.log('${obj}:', `${obj}`) // '20'
console.log('obj + 1:', obj + 1) // 11
console.log('obj+ "":', obj + '') // '10'

const obj = {
  name: 'WEB架构师',
  age: 22,
}

console.log(obj + 1) // [object Object]1
console.log(1 + obj) // 1[object Object]
console.log(obj * 1) // NaN
console.log(obj - 1) // NaN
console.log(obj.toString()) // [object Object]
console.log(`${obj}`) // [object Object]

const obj = {
  name: 'WEB架构师',
  age: 22,
  valueOf() {
    return this.age
  },
}

console.log(obj + 1) // 23
console.log(1 + obj) // 23
console.log(obj * 1) // 22
console.log(obj - 1) // 21
console.log(obj.toString()) // [object Object]
console.log(`${obj}`) //  [object Object]

const obj = {
  name: 'WEB架构师',
  age: 22,
  toString() {
    return this.name
  },
}

console.log(obj + 1) // "WEB架构师1"
console.log(1 + obj) // "1WEB架构师"
console.log(obj * 1) // NaN
console.log(obj - 1) // NaN
console.log(obj.toString()) // "WEB架构师"
console.log(`${obj}`) //  "WEB架构师"

const obj = {
  name: 'WEB架构师',
  age: 22,
  valueOf() {
    return this.age
  },
  toString() {
    return this.name
  },
}

console.log(obj + 1) // 23
console.log(1 + obj) // 23
console.log(obj * 1) // 22
console.log(obj - 1) // 21
console.log(obj.toString()) // "WEB架构师"
console.log(`${obj}`) //  "WEB架构师"
```

### 数据属性描述符

```js
let obj = {
  name:"WEB架构师",
  age:21
}
// 可以通过 Object.getOwnPropertyDescriptors(obj) 来获取该对象身上的所有属性的属性描述符
console.log(Object.getOwnPropertyDescriptors(obj))
// name
name: {
  value: name,
  // 通过字面量方法添加的属性 默认为true
  configurable: true,
  // 通过字面量方法添加的属性 默认为true
  enumerable: true,
  // 通过字面量方法添加的属性 默认为true
  writable: true
}

// 数据属性描述符
// 第一个参数为要操作的对象 第二个参数为要操作的属性 第三个参数为操作方式
Object.defineProperty(obj, "skill", {
  // 该特性为该属性的值 默认新添加的属性值为undefined
  value: "js,vue,react", //
  // 该特性不可删除，也不可以重新定义属性描述符
  configurable: false, // 默认为false
  // 该特性是配置对应的属性是否可枚举
  enumerable: false,// 默认为false
  // 该特性是属性是否可以赋值
  writable: false // 默认为false
})
```

### 存取属性描述符

```js
// 访问器属性不包含数据值。相反，它们包含一个获取（getter）函数和一个设置（setter）函数，不过这两个函数不是必需的。在读取访问器属性时，会调用获取函数，这个函数的责任就是返回一个有效的值。在写入访问器属性时，会调用设置函数并传入新值，这个函数必须决定对数据做出什么修改。访问器属性有 4 个特性描述它们的行为。
[[Configurable]]：表示属性是否可以通过 delete 删除并重新定义，是否可以修改它的特
性，以及是否可以把它改为数据属性。默认情况下，所有直接定义在对象上的属性的这个特性
都是 true。  [[Enumerable]]：表示属性是否可以通过 for-in 循环返回。默认情况下，所有直接定义在对
象上的属性的这个特性都是 true。  [[Get]]：获取函数，在读取属性时调用。默认值为 undefined。  [[Set]]：设置函数，在写入属性时调用。默认值为 undefined。

// 1. 隐藏某个私有属性希望被外界直接访问和赋值
let obj = {
  name:"WEB架构师",
  age:21,
  _skill:"JavaScript,PHP"
}

// 单个定义属性描述符
Object.defineProperty(obj,"skill",{
  enumerable: true,
  configurable: true,
  get(){
    console.log("访问了该属性")
  	return this._skill
  },
  set(num){
    console.log("修改了该属性")
  	this._skill = num
  }
})

// 同时定义多个属性描述符
Object.defineProperties(stuObj, {
  obj1: {
    configurable: false,
    writable: false,
    value: "定义多个属性描述符",
    enumerable: true
  },
  obj2: {
    configurable: true,
    enumerable: false,
    get: function () {

    },
    set: function () {

    }
  }
})

// 监听一个对象里面的所有属性
Object.keys(obj).forEach(item => {
  // 获取value值
  let val = obj[item]
  Object.defineProperty(obj, item, {
    get(){
      console.log(`${item}被使用了`)
    },
    set(newVal){
      val = newVal
      console.log(`${item}被设置了`)
    }
  })
})


// 案例：
const proto = {
  name: "p_parent",
  type: "p_object",
  [Symbol.for("p_address")]: "地球",
}

const ins = Object.create(proto)
Object.defineProperty(ins, "age", {
  value: 22,
})
ins.sex = 1
ins[Symbol.for("say")] = function () {
  console.log("say")
}

const inKeys = []
for (let p in ins) {
  inKeys.push(p)
}

// 可以遍历原型数据，但是不可以遍历不可枚举和symbol
console.log(inKeys) // ['sex', 'name', 'type']
console.log(Reflect.ownKeys(ins)) // ['sex', 'age', 'p_say']
// 可以遍历symbol和不可枚举的属性 不可以遍历原型数据
```

### Class 类

```js
// Es6推出的class类是ES5构造函数的语法糖，但是在继承等方面却得到了极大的方便
// 基类
class Person {
  // 构造器，用来接受传递的参数，有且只能有一个
  constructor(name, age) {
    this.name = name
    this.age = age
  }
  runing() {
    console.log(`${this.name}正在跑步`)
  }
  // 静态方法，就是通过类名直接调用的方法，但是不能通过实例来调用，Person.eating()，应用场景中，比如：Promise.all()
  static eating(name) {
    console.log(`${name}吃饭饭`)
  }
}

// 继承类
// 在ES5中，我们通过寄生组合式继承方法来实现继承，在Es6类中，我们只需要extends关键字即可实现继承
class Teacher extends Person {
  constructor(name, age, title) {
    // 通过super调用父类构造函数，传递参数
    super(name, age)
    this.title = title
  }
  studying() {
    console.log(`${this.name}正在教书`)
  }
  // 如果子类方法名和父类方法名一样，会重新父类方法，但是可以通过super.方法()的方法先调用父类的方法
  runing() {
    super.runing()
    console.log(`${this.name}正在听歌`)
  }
}

const tea = new Teacher('老师', 32, '高级讲师')

console.log(tea) // Teacher {name: "老师", age: 32, title: "高级讲师"}
tea.runing() // 老师正在跑步 老师正在听歌
```

### Symbol 使用

```js
// Symbol是Es6出现的唯一标识
// 使用
let a1 = Symbol()
let a2 = Symbol()
console.log(a1 === a2) // false

// 添加描述
let name = Symbol('这是name属性')
let age = Symbol('这是age属性')
// 获取属性
console.log(name.description, age.description) // 这是name属性 这是age属性

// 在使用Symbol作为key的属性名，在遍历或者Object.keys是获取不到Symbol值的，但是可以使Object.getOwnPropertySymbols（对象名）来获取所有的Symbol
let name = Symbol()
let age = Symbol()
let obj = {
  // 使用 [Symbol值] 加入
  [name]: 'WEB架构师',
  [age]: 22,
}

// 使用getOwnPropertySymbols遍历
const symbols = Object.getOwnPropertySymbols(obj)
for (let s of symbols) {
  console.log(obj[s]) // "WEB架构师" 22
}
```

### Set 和 WeakSet

```js
// 在E6中新推出的数据结构，Set和Map
// Set是新增的数据结构，用来用来保存数据，类似于数组，但是和数组不同的是元素不能重复，因此可以用来去重。

// 使用
const arr = [1, 2, 3, 4]
const set = new Set()
const set1 = new Set(arr)

// 查看元素个数
set.size
// 添加元素
set.add(value)
// 删除元素
set.delete(value)
// 判断是否存在某个元素，返回boolean
set.has(10)
// 清空set
set.clear()

// 遍历Set forEach for...of
set.forEach(item => {
  console.log(item)
})
for(let item of set){
  console.log(item)
}

// Set转数组
1. [...set]
2. Array.from(set)

// 数组转Set
new Set(arr)


// WeakSet
// 和set类似，元素不能重复
// 和set的区别：
区别一：WeakSet只能存放对象类型，不能存放基本数据类型
区别二：WeakSet对元素的引用是弱引用，弱引用和强引用比起来，它们虽然都指针指向一个地址，但是GC会回收弱引用指向的地址。

// 使用
const obj = {name:"web"}
const weak = new WeakSet()

// 添加
weak.add(obj)
// 删除
weak.delete(obj)
// has 判断是否存在某个对象 返回boolean
weak.has(obj)

// WeakSet不能遍历
// 因为WeakSet只是对对象的弱引用，因此是没办法获取的

```

### Map 和 WeakMap

```js
// Es5中我们无法用对象类型作为key来使用的，但是在Es6中的Map，允许我们我们对象类型来作为key的

// 使用
const obj1 = { name:"obj1" }
const obj2 = { name:"obj2" }
const map = new Map()
const map1 = new Map([obj1,"aaa"],[obj2,"bbb"],[1,"ccc"])
// 添加
map.set(obj1,"aaa")
map.set(obj2,"bbb")
map.set(123,"ccc")

console.log(map) // Map(3) { {name:"obj1"} => "aaa",{name:"obj2"} => "bbb",123 => "ccc" }

// 添加
map.set(key,value)
// 根据key获取value
map.get(key)
// 判断是否包含某一个key 返回boolean
map.has(key)
// 根据key删除键值对，返回boolean
map.delete(key)
// 清空
map.clear()

// Map可以遍历 forEach for...of
map.forEach((value,key)=>{
  console.log(value,key)
})

for(let [key,value] of map){
  console.log(key,value)
}


// WeakMap
// 和Map类似，都是以键值对的形式存在的
// 区别：
1. WeakMap的key只能使用对象，不接受其他类型的key
2. WeackMap的key对对象是弱引用，因此可以被GC回收

const weakMap = new WeakMap()
const obj = { name: "web" }
weakMap.set(obj,"aaa")

// 常用方法
// 添加
weakMap.set(key,value)
// 根据key获取value
weakMap.get(key)
// 判断是否含有key 返回boolean
weakMap.has(key)
// 根据key删除键值对
weakMap.delete(key)

// WeakMap不能遍历，没有forEach方法，也不支持for...of遍历

// 应用场景 Vue3响应式原理 结合Proxy
Map用来保存响应式数据的属性和数据改变处理的函数，WeakMap保存响应式数据名和上面生成的Map数据，通过Proxy可以获取改变的数据对象和其属性，依次通过WeakMap和Map获取发生变化的数据对象和属性，并自动调用发生改变时的对应函数

// 响应式对象 data
const data = {
	name: "WEB架构师",
  age: 22
}

// 页面渲染
document.querySelector("#mes").innerHTML = `${data.name},${this.age}`

// data对象属性改变时，对应属性的方法
const altDataName = () => {
	document.querySelector("#mes").innerHTML = `${data.name},${this.age}`
}
const altDataAge = ()=> {
  document.querySelector("#mes").innerHTML = `${data.name},${this.age}`
}

// 声明WeakMap对象和Map对象
const weakMap = new WeakMap()
const map = new Map()

// 添加Map和WeakMap方法
map.set("name", [altDataName])
map.set("age", [altDataAge])
weakMap.set(data, map)

// 通过Proxy监听属性值变化
const proxy = new Proxy(data, {
  // 对象源 属性
  get(target, key){
    return target[key]
  },
  // 对象源 属性 新值
  set(target, key, newVal){
    // 赋新值
    target[key] = newVal
    // 当发现值改变时 通过WeakMap传入改变的对象 返回键值对 以确保是哪个对象发生了改变
    const obj = weakMap.get(target)
    // 通过该对象获取key来确认是哪一个值发生了改变，并获取该数组的所有函数
    const arr = map.get(key)
    // 通过遍历依次执行该方法
    arr.forEach(item => item())
  }
})


// Map与数组和对象之间的转换
const map = new Map()
map.set('key1', 'value1').set('key2', 'value2')

console.log('map', map) // map Map(2) {'key1' => 'value1', 'key2' => 'value2'}

// Map转数组
console.log('Map转数组', [...map]) // Map转数组 [['key1', 'value1'], ['key2', 'value2']]

// 数组转Map
const arr = [1, 2, 3, 4]
console.log('数组转Map', new Map(arr.entries())) // 数组转map Map(4) {0 => 1, 1 => 2, 2 => 3, 3 => 4}

// Map转对象
// 方法一
const obj = {}
map.forEach((value, key) => {
  obj[key] = value
})
console.log('Map转对象', obj) // Map转对象 {key1: 'value1', key2: 'value2'}
// 方法二
console.log(Object.fromEntries(map))


// 对象转Map
const obj = {
  name: 'web',
  age: 22
}
// 方法一
console.log('对象转Map 方法一：', new Map(Object.entries(obj))) // 对象转Map 方法一： Map(2) {'name' => 'web', 'age' => 22}
// 方法二
const map2 = new Map()
for (const [key, value] of obj) {
  map2.set(key, value)
}
console.log('对象转Map 方法二：', new Map(Object.entries(obj))) // 对象转Map 方法二： Map(2) {'name' => 'web', 'age' => 22}
```

### flatMap 使用

```js
// ES10(ES2019)中推出了flatMap，相比于map，它可以遍历后直接转化为一维数组。
let str = ['hello word', '你好 WEB', 'JavaScript Node Webpack']
let newStr = str.flatMap(item => item.split(' ')) // ["hello", "word", "你好", "WEB", "JavaScript", "Node", "Webpack"]
let newStr1 = str.map(item => item.split(' ')) //  (3) [Array(2), Array(2), Array(3)]
```

### !!和??

```js
// !!
1. 将一个其他类型转化为boolean类型
2. 类似于Boolean(num)的方式

// ??
逻辑运算符，当操作符的左侧是null或undefined时，返回右侧操作数，否则返回左侧操作数


// 在平时开发中，我们需要将一个内容转化为Boolean
// 常见的转化方式
const num = 123
const bol = Boolean(num)

// 使用 !! 转化  机制： !num会将一个数字转化为取反的boolean，!!num则会得到正常的boolean类型
const bol = !!num
```

### 空值运算合并符??使用

```js
// ES11推出的？？用来解决空值问题
// ?? 更好的解决了 || 的问题，相对于||来说，让我们传入 0 或者是 "" ，它就失去了判断的意义，然而 ?? 则很好的弥补了这一点

function fun(value) {
  let val1 = value ?? 'default'
  let val2 = value || 'default'
  console.log(val1, val2)
}

fun(0) // 0 default
fun(10) // 10 10
fun('') // "" default
```

### 可选链（Optional Chaining）

```js
// 在ES11（ES2010）中推出了可选链，方便了我们在进行undefined和null的判断
const obj = {
  name: 'web架构师',
  age: 22,
  friend: ['js', 'vue', 'react'],
  info: {
    sex: '男',
    like: '火锅',
    more: {
      data: '要获取的数据',
    },
  },
}

// 需求：假设我们要获取obj中的data数据
// 以往的方法
if (obj6 && obj6.info && obj6.info.more) {
  console.log(obj6.info.more.data)
}
// ES11可选链
console.log(obj6.info?.more?.data)
```

### Global This

```js
// 获取全局对象，在浏览器环境下获取的是window，在Node环境中获取的是Global,这样可以在不同的环境下获取全局对象

// 浏览器环境
console.log(globalThis) //window

// Node环境
console.log(globalThis) // global
```

### ES12 logical assignment operators

```js
// 1.逻辑或运算符
let mes = ""
mes ||= "defalut"
console.log(mes) // default

**类似于以前的**
mes = mes || "defalut"

// 2. 逻辑与操作符
let obj = {
  name:"WEB架构师"
}
obj &&= obj.name
console.log(obj) // WEB架构师

**类似于以前的**
obj = obj && obj.name


// 3. 逻辑空运算符
let foo = 0
foo ??= "default"
console.log(foo) // default

**类似于以前的**
foo = foo ?? "default"
```

### Proxy 代理对象

```js
// 在ES6中，新增Proxy类，帮助我们创建和监听对象的
// 用法
const objProxy = new Proxy(target, handler)

// 案例
const obj = {
  name:"WEB架构师",
  age:22
}
const objProxy = new Proxy(obj, {
  // 获取属性捕获器  (目标对象、监听的对象(obj) ， 监听的属性，调用的代理对象(objProxy))
  get(target, key, receiver){
    console.log(`获取了${key}属性`)
    return target[key]
  },
  // 设置属性捕获器  (目标对象、监听的对象 ， 监听的属性 ， 设置的新值 ，调用的代理对象(objProxy))
  set(target, key, newValue, receiver){
    console.log(`${key}属性设置了新值`)
    target[key] = newValue
  },
  // in 捕获器  (目标对象、监听的对象 ， 监听的属性)
  has(target,key){
    console.log(`in 属性捕获器`)
    return key in target
  },
  // delete 捕获器   (目标对象、监听的对象 ， 监听的属性)
  deleteProperty(target,key){
    console.log("delete属性捕获器")
  	delete target[key]
  }
})

// 获取值捕获器
console.log(objProxy.name)
// 设置值捕获器械
objProxy.name = "WEB开发者"
// 查看属性是否存在
console.log("name" in objProxy)
// 删除属性
delete objProxy[name]

// 上面的写法可以简写为如下：
const objPro = new Proxy(obj, {
  get(...rest) {
    return Reflect.get(...rest)
  },
  set(...rest) {
    return Reflect.set(...rest)
  }
})
// 或
const objPro - new Proxy(obj, {
  get: Reflect.get,
  set: Reflect.set
})
```

### ES7

```js
// Array Includes
// 判断数组是否含有某一指定元素
const arr = [1, 2, 3]
console.log(arr.includes(2)) //true
console.log(arr.includes(4)) // false

// 指数(乘方) exponentiation运算符
// 2的3次方
// ES7之前
console.log(Math.pow(2, 3)) // 8
// ES7
console.log(2 ** 3)
```

### ES8

```js
const obj = {
  name: 'WEB架构师',
  age: 22,
}

// Object values
// 可以获取对象中所有的value，而Object.keys()是获取对象中所有的key
console.log(Object.values(obj)) //  ["WEB架构师", 22]

// Object entries
// 可以获取到一个数组，数组中会存放可枚举属性的键值对数组
console.log(Object.entries(obj)) // [ ["name","WEB架构师"],["age","22"]]

// String Padding
// 分别是对字符串的首尾进行填充的，返回一个新值，不会影响原来的字符串
const mes = 'WEB架构师'
// 首部填充 第一个参数为填充后的长度 第二个参数为填充的内容，默认为空
console.log(mes.padStart(12, 'a')) // aaaaaaWEB架构师
// 尾部填充 第一个参数为填充后的长度 第二个参数为填充的内容，默认为空
console.log(mes.padEnd(12, 'a')) // WEB架构师aaaaaa
// 案例
// 现有一串银行卡号 要求只显示后四位 前面的用星号代替
const nums = '123234234124235123'
const lastNums = nums.slice(-4)
const newNums = lastNums.padStart(nums.length, '*')

// Object Descriptors
// Object.getOwnPropertyDescriptors()
// 获取对象上的所有*自身*属性的描述符
const obj = {
  name: 'WEB架构师',
  age: 22,
}
console.log(Object.getOwnPropertyDescriptors(obj)) // {name:"WEB架构师",age:22}

// async/await
```

### Reflect

```js
Reflect是ES6新增的类，它是一个对象，字面的意思是反射。
作用： 提供了很多操作JavaScript对象的方法，用来代替Object进行操作。
Reflect通常结合Proxy一起使用

const obj1 = {
  name:"obj1",
  get name(){
    return this.name
  }
}
const obj2 = {
  name:"obj2"
}

// Reflect上的方法 与 Object上的方法对照


// 获取对象身上某个属性的值，类似于 target[key]
Reflect: Reflect.get(target,  key,  receiver)
原始：    target[key]
// target：需要取值的目标对象
// key：需要获取的键值
// receiver：如果target对象中指定了getter，receiver则为getter调用时的this值，如果没有getter，则this指向毫无意义
console.log(Reflect.get(obj1,"name")) // obj1
console.log(Reflect.get(obj1,"name",obj2)) // obj2

// 数据属性描述符
Reflect: Reflect.defineProperty(target, property, descriptor)
原始：    Object.defineProperty(target, property, descriptor)
// target：目标对象。
// property：引用的目标对象上的字符串键属性。
// descriptor：包含可选的 enumerable、configurable、writable、value、get 和 set定义的对象。

// 获取属性描述符
Reflect: Reflect.getOwnPropertyDescriptor(target, property)
原始：    Object.getOwnPropertyDescriptor(target, property)
// target：目标对象。
// property：引用的目标对象上的字符串键属性。

// 删除对象上的属性
Reflect: Reflect.deleteProperty(target, property)
原始：    delete target[property]
// target：目标对象。
// property：引用的目标对象上的字符串键属性。

// 获取对象上的所有 可枚举的key
Reflect: Reflect.ownKeys(target)
原始：    Object.keys(tarhey)
// target：目标对象。

// 获取对象的原型
Reflect: Reflect.getPrototypeOf(target)
原始1:    Object.getPrototypeOf(target)
原始2:    target.__proto__
// target：目标对象。

// 设置对象的原型
Reflect: Reflct.setPrototypeOf(target, prototype)
原始1:    Object.setPrototypeOf(target, prototype)
原始2:    target.__proto__ = prototype
// target：目标对象。
// prototype：target 的替代原型，如果是顶级原型则为 null。

// 检测对象是否可拓展
Reflect: Reflect.isExtensible(target)
原始:     Object.isExtensible(target)
// target：目标对象。

// apply调用
Reflect: Reflect.apply(target, thisArg, argumentList)
原始1:    Function.prototype.apply(thisArg, argumentList)
原始2:    Function.prototype.call(thisArg, argumentList)
// target：目标对象。
// thisArg：调用函数时的 this 参数。
// argumentsList：调用函数时的参数列表
```

### Promise

```js
// 用法
const promise = new Promise((resolve,reject) => {
  resolve("成功的回调")
})

promise.then(res => {
  console.log(`res: ${res}`)
}).catch(err => {
  console.log(`err: ${err}`)
})

.then() 和 .catch() 默认都返回一个resolve的Promise，值为undefined


// 类方法Promise.resolve
const promise = Promise.resolve({ name: "web" })
// 相当于创建一个Promise对象，并返回resolve方法
const promise1 = new Promise((resolve, reject) => {
	resolve({ name: "web" })
})

// 类方法Promise.reject
const promise = Promise.reject("rejectd message")
// 相当于 创建一个Promise对象，并且返回reject方法
const promise1 = new Promise((resolve, reject) => {
  reject("rejected message")
})


// 类方法Promise.all
// 所有的Promise都变成fulfilled时，再拿到结果，非Promise的会被Promise.resolve()转为Promise，如果传入的参数有一个是reject，那么整个Promise都是reject

// 这个静态方法接收一个可迭代对象，返回一个新的Promise:
let p1 = Promise.all([
 Promise.resolve(),
 Promise.resolve()
]);

// 可迭代对象中的元素会通过 Promise.resolve()转换为期约
let p2 = Promise.all([3, 4]);

// 空的可迭代对象等价于 Promise.resolve()
let p3 = Promise.all([]);

// 无效的语法
let p4 = Promise.all();
// TypeError: cannot read Symbol.iterator of undefined

const p1 = new Promise((resolve, reject) => {
	resolve("p1成功返回")
})
const p2 = new Promise((resolve, reject) => {
	reject("p2失败返回")
})
Promise.all([p1,p2]).then(res => {
  console.log(res)
}).catch(err => {
  console.log(err)  // p2失败返回
})


// 类方法Promise.allSettled
// all方法有一个缺陷：当有其中一个Promise变成reject状态时，新Promise就会立即变成对应的reject状态。该方法会在所有的Promise都有结果（settled），无论是fulfilled，还是reject时，才会有最终的状态

// 这个静态方法接收一个可迭代对象，返回一个新的Promise:
Promise.allSettled([p1,p2]).then(res => {
  console.log(res) // [{status: "fulfilled", value: "p1成功返回"}, {status: "rejected", reason: "p2失败返回"}]
}).catch(err => {
  console.log(err)
})

// 类方法Promse.race()
// 多个Promise相互竞争，谁先有结果，那么就使用谁的结果
const p1 = new Promise((resolve, reject) => {
	setTimeout(() => {
    resolve("p1成功返回")
  }, 5000)
})
const p2 = new Promise((resolve, reject) => {
	setTimeout(() => {
    reject("p2失败返回")
  }, 2000)
})
const p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("p3成功返回")
  }, 3000)
})
Promse.race([p1, p2, p3]).then(res => {
  console.log(res)
}).catch(err => {
  console.log(err)  // p2失败返回
})


// 类方法 Promise.any()
// any方法是ES12中新增的方法，和race方法是类似的 any方法会等到一个fulfilled状态，才会决定新Promise的状态；如果所有的Promise都是reject的，那么也会等到所有的Promise都变成rejected状态；如果所有的Promise都是reject的，那么会报一个AggregateError的错误。
const p1 = new Promise((resolve, reject) => {
	setTimeout(() => {
    resolve("p1成功返回")
  }, 500)
})
const p2 = new Promise((resolve, reject) => {
	setTimeout(() => {
    reject("p2失败返回")
  }, 100)
})
const p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("p3成功返回")
  }, 300)
})
Promse.race([p1, p2, p3]).then(res => {
  console.log(res)  // p3成功返回
}).catch(err => {
  console.log(err)
})
```

### 异常抛出

```js
// 可以通过thorw关键字，抛出一个异常

// JavaScript内置了一个异常类，Error类，可以直接创建这个类的对象
throw new Error("Err Message")

// Error有一些自己的子类
RangeError : 下标值越界时使用的错误类型
SyntaxError : 解析语法错误时使用的错误类型
TypeError : 出现类型错误时，使用的错误类型
// 使用
throw new TypeError("类型错误")
```

### JSON 序列化

```js
// 在目前的开发中，JSON是一种非常重要的数据格式，它并不是编程语言，而是一种可以在服务器和客户端之间传输的数据格式

// 规则
	· undefined、任意函数、symbol
   - 在作为属性值的时候，会被自动忽略，
 	 - 作为数组，序列化返回null
	 - 单独序列化，返回undefined

// 将一个普通对象转为JSON字符串保存
localStorage.setItem("obj", JSON.stringify(obj))

// 将JSON字符串转回对象 转对对应的JavaScript类型
JSON.parse(localStorage.getItem("obj"))

JSON.stringify(target, ?:replacer, ?:space)
// 如果replacer是一个函数，可以选择性替换值，如果replacer是数组，则会保留仅包含数组指定的属性，space会如果是 2 ，会转为直观的JavaScript对象类型，数字2代表两个空格，如果是其他字符串，则会将空格替换为字符串j
const obj = {
  name: "WEB架构师",
  age: 22,
  info: "WEB高级架构师"
}
const obj1 = JSON.stringify(obj, ["name"]) // {"name":"WEB架构师"}

// key为属性 value为值
const obj2 = JOSN.stringify(obj, (key, value) => {
  // age年龄++
  if(key === "age") return ++value
	// 隐藏info属性
  if(key === "info") return undefined
  return value
}, 2) // 保留两个空格 更直观


JSON.parse(target, ?:reviver)
// JSON.parse()方法用来解析JSON字符串，构造由字符串描述的JavaScript对象， reviver和stringify的方法一样
const obj1 = JSON.parse(JSON.stringify(obj), (key, value) => {
  // 年龄--
  if(key === "age") return --value
	// 隐藏info
  if(key === "info") return undefined
  return value
})


// 可以使用JOSN序列化实现深拷贝，但是如果对象里面有函数或者Symbol，就无能为力了
const newObj = JSON.parase(JSON.stringify(obj))
```

### localStorage 和 sessionStorage

```js
// localStorage会永久保存，除非用户手动清除，而sessionStorage会在浏览器关闭的时候销毁

// localStorage和sessionStorage用着相同的方法
// Storage常见的方法和属性
// 属性
Storage.length // 返回一个整数，表示存储Storage的数据项数量

// 方法
Storage.key(n:number) //  该方法接受一个数值n作为参数，返回存储中的第n个key名称
Storage.getItem(key) // 该方法接受一个key作为参数，并且返回key对应的value
Storage.setItem(key, value) // 该方法接受一个key和value，并且将会把key和value添加到存储中 如果key相同，则会更新
Storage.removeItem(key) // 该方法接受一个key作为参数，并把该key从存储中删除
Storage.clear()  // 该方法的作用是清空存储中的所有key

```

### generator 函数

```js
// ES6新出的语法，是一个生成器函数，返回一个迭代器对象

// 定义 普通函数多一个 *
function* fun() {
  yield 'vue'
  yield 'react'
  yield 'angular'
}

// iterator 迭代器
const result = fun()
console.log(result) // fun {<suspended>}

// 使用迭代器
console.log(result.next()) // {value: 'vue', done: false} value代表当前yield的值，done代表迭代器是否消耗完
console.log(result.next()) // {value: 'react', done: false} value代表当前yield的值，done代表迭代器是否消耗完
console.log(result.next()) // {value: 'angular', done: false} value代表当前yield的值，done代表迭代器是否消耗完
console.log(result.next()) // {value: undefined, done: true}value代表当前yield的值，done代表迭代器是否消耗完

// generator和Promise一起使用
function* fun() {
  const val = yield new Promise(resolve => {
    setTimeout(() => {
      resolve('hello')
    }, 3000)
  })
  console.log(val)
}

const obj = fun()
obj.next().value.then(res => {
  obj.next(res)
})
```

### fetch

```js
// 一个让处理http pipline更容易的工具
特点：
	- 返回一个Pormise
	- Resolve发生在网络通信正常（404，500也是resolve）
  - Reject发生在网络通信异常
	- 默认不接受cookie（需要设置）


// get
fetch('http://localhost:3000')
.then(res => res.json())
.then(res => console.log('获取的对象', text))
.catch(err => console.log(err))

// post
fetch("http://localhost:3000", {
  methods: "POST",
  headers: {
    "Content-Type": "application/x-www-form-urlencoded",
    body: 'report=2',
    mode: 'cors' // 设置跨域
  }
})
.then(res => res.json())
.then(res => console.log(res))
.catch(err => console.log(err))
```

## 手写

### 数组手写

#### 手写 push

```js
// 将自定义的myPush挂在到Array的原型方法上
Array.prototype.myPush = function (...thisArr) {
  // 获取拼接后的数组
  let self = [...this, ...thisArr]
  // 将拼接后的数组赋值给旧数组
  self.forEach((item, index) => (self[i] = item))
  // 返回新数组的长度
  return self.length
}
```

#### 手写 pop

```js
// 将自定义的myPop方法挂载到Array的原型上
Array.prototype.myPop = function () {
  // 获取到被删除的数组的最后一个值
  let self = this[this.length - 1]
  // 数组的长度减一，最后一个值被删除
  this.length--
  // 返回被删除的值
  return self
}
```

#### 手写 shift

```js
// 将自己的myShift挂载到Array的原型方法上
Array.prototype.myShift = function () {
  // 获取将要删除的数组第一个元素的值
  let self = this[0]
  // 数组的元素前移1
  this.forEach((item, index) => (this[index] = this[index + 1]))
  // 数组的长度减少
  this.length--
  // 返回被删除的第一个值
  return self
}
```

#### 手写 unshift

```js
// 将自己的myunshift挂载到Array的原型方法上
Array.prototype.myunshift = function(..thisArr){
  // 获取拼接后的数组
  let self = [...thisArr,...this]
  // 将拼接后的数组赋值到旧数组
  self.forEach((item,index) => this[i] = item )
  // 返回新数组的长度
  return self.length
}
```

#### 手写 flat

```js
// 分为两种方法 一种是普通的递归函数 一种是使用reduce函数迭代

// 1. 普通函数递归
Array.prototype.myFlat = function (depth = 1) {
  let newArr = []

  this.forEach(item => {
    if (Array.isArray(item) && depth > 0) {
      newArr = newArr.concat(item.myFlat(depth - 1))
    } else {
      newArr.push(item)
    }
  })

  return newArr
}

// 2. 使用reduce函数迭代
Array.prototype.myFlat = function (depth = 1) {
  return this.reduce((prev, next) => {
    if (Array.isArray(next) && depth > 0) {
      return prev.concat(next.myFlat(depth - 1))
    } else {
      return [...prev, next]
    }
  }, [])
}
```

#### 手写 flatMap

```js
// 方法一：利用flat展开一层
Array.prototype.myFlatMap = function (thisFun) {
  if (typeof thisFun !== 'function') throw TypeError('期望传入函数')

  const result = []

  for (let i = 0; i < this.length; i++) {
    result.push(fn(this[i], i, this))
  }

  return result.flat()
}

// 方法二：利用contact的特性自动展开
Array.prototype.myFlatMap = function (callback) {
  let arr = []

  for (let i = 0; i < this.length; i++) {
    const result = callback(this[i], i, this)

    arr = arr.concat(result)
  }

  return arr
}
```

#### 手写 forEach

```js
Array.prototype.myForEach = function (thisFun) {
  if (typeof thisFun !== 'function') throw TypeError('期望传入函数')

  const _self = [...this]
  for (let i = 0; i < _self.length; i++) {
    const val = _self[i]
    thisFun.call(this, val, i, _self)
  }
}
```

#### 手写 map

```js
Array.prototype.myMap = function (thisFun) {
  if (typeof thisFun !== 'function') throw TypeError('期望传入函数')

  const _self = [...this]
  const arr = []
  for (let i = 0; i < _self.length; i++) {
    const val = _self[i]
    arr.push(thisFun.call(this, val, i, _self))
  }
  return arr
}
```

#### 手写 reduce

```js
Array.prototype.myReduce = function (thisFun, init) {
  const _self = [...this]
  // 当前遍历的索引
  let currentIndex = 0
  if (init === undefined) {
    init = _self[0]
    currentIndex = 1
  }

  for (currentIndex; currentIndex < _self.length; currentIndex++) {
    init = thisFun.call(this, init, this[currentIndex], currentIndex, _self)
  }
  return init
}
```

#### 手写 filter

```js
Array.prototype.myFilter = function (thisFun) {
  const _self = [...this]
  const len = _self.length

  const arr = []
  for (let i = 0; i < len; i++) {
    const val = _self[i]
    if (thisFun.call(this, val, i, _self)) {
      if (typeof val === 'object') {
        arr.push(Object.assign({}, val))
      } else {
        arr.push(val)
      }
    }
  }

  return arr
}
```

#### 手写 find

```js
Array.prototype.myFind = function (thisFun) {
  if (typeof thisFun !== 'function') throw TypeError('期望传入类型为函数')
  const _self = [...this]
  for (let i = 0; i < _self.length; i++) {
    const val = thisFun(_self[i], i, _self)
    if (val)
      return typeof _self[i] === 'object'
        ? Object.assign({}, _self[i])
        : _self[i]
  }
}
```

#### 手写 findIndex

```js
Array.prototype.myFindIndex = function (thisFun) {
  if (typeof thisFun !== 'function') throw TypeError('期望传入参数为函数')
  const _self = [...this]
  for (let i = 0; i < _self.length; i++) {
    const result = thisFun(_self[i], i, _self)
    if (result) return i
  }
}
```

#### 手写 includes

```js
// 注意：
arr.includes(valueToFind[, fromIndex])
// 从fromIndex 索引处开始查找 valueToFind。如果为负值，则按升序从 array.length + fromIndex 的索引开始搜（即使从末尾开始往前跳 fromIndex 的绝对值个索引，然后往后搜寻）。默认为 0。

Array.prototype.myIncludes = function (valueToFind, fromIndex = 0) {
  const self = [...this]
  const len = self.length
  if (fromIndex < 0) fromIndex += len
  for (fromIndex; fromIndex < len; fromIndex++) {
    if (valueToFind === self[fromIndex]) return true
  }
  return false
}
```

#### 手写 some

```js
Array.prototype.mySome = function (thisHoc) {
  if (typeof thisHoc !== 'function')
    throw TypeError(`${thisHoc} is not a function`)

  const self = [...this]
  const len = self.length
  for (let i = 0; i < len; i++) {
    const val = thisHoc(self[i], i, self)
    if (val) return true
  }
  return false
}
```

#### 手写 every

```js
Array.prototype.myEvery = function (thisHoc) {
  if (typeof thisHoc !== 'function')
    throw TypeError(`${thisHoc} is not a function`)

  const self = [...this]
  const len = self.length
  for (let i = 0; i < len; i++) {
    const val = thisHoc(self[i], i, self)
    if (!val) return false
  }
  return true
}
```

### 深拷贝手写

#### 手写深拷贝

```js
// 支持深拷贝Symbol类型（包括key或value为Symbol）

function deepCopy(obj) {
  if (typeof obj !== 'object') return obj
  if (obj === null) return null
  
  let newObj = Array.isArray(obj) ? [] : {}

  for (const key in obj) {
    const val = obj[key]

    if (typeof val === 'object') {
      newObj[key] = deep(val)
    } else if (typeof val === 'symbol') {
      newObj[key] = Symbol(val.description)
    } else {
      newObj[key] = val
    }
  }

  // 处理key为symbol类型
  const syms = Object.getOwnPropertySymbols(obj)

  syms.forEach(sym => {
    const newSym = Symbol(sym.description)
    newObj[newSym] = deep(obj[sym])
  })

  return newObj
}

let obj1 = deepCopy(obj)
```

### 函数手写

#### 手写 call

```js
// 将自己的myCall方法挂载到Function的原型上
// 第一个参数为this要绑定的对象 第二个参数为接受的参数
Function.prototype.myCall = function (thisArg, ...thisArr) {
  // 获取当前的this 调用该方法的函数
  let self = this
  // 判断要绑定的this指向值，存在将其对象化，不存在默认指向window
  // thisArg = thisArg !== undefined && thisArg !== null ? Object(thisArg) : window
  // 上面和下面这种写法相同
  thisArg = Object(thisArg ?? window)
  // 使用Symbol 防止篡改this绑定对象上的内容
  let sym = Symbol('this绑定')
  // 利用隐式绑定 绑定this
  thisArg[sym] = self
  // 调用该函数 并将值传入
  const result = thisArg[sym](...thisArr)
  // 删除上面添加的属性
  delete thisArg[sym]
  
  return result
}

fun.myCall('123', 1, 2)
```

#### 手写 apply

```js
// 将自定义的myApply挂载到Function的原型中
// 第一个参数 要挂载this的对象  第二个对象 传入的数据，apply默认以Array的方式传参，赋值默认值[]
Function.prototype.myApply = function (thisArg, thisArr = []) {
  // 获取当前调用该方法的函数 this
  let self = this
  // 判断要绑定的this指向值，存在将其对象化，不存在默认指向window
  thisArg = Object(thisArg ?? window)
  // 使用Symbol 防止篡改this绑定对象上的内容
  let sym = Symbol('this绑定对象')
  // 利用显示绑定绑定this
  thisArg[sym] = self
  // 调用该函数 并传入参数
  const result = thisArg[sym](...thisArr)
  delete thisArg[sym]
  
  return result
}

sum.myApply('123', 123, 2)
```

#### 手写 bind

```js
// 将自定义的myBind挂载到Function的原型中
// 第一个参数 要挂载this的对象  第二个对象 传入的数据
Function.prototype.myBind = function (thisArg, ...thisArr) {
  // 获取调用该方法的函数 this
  let self = this
  // 判断要绑定的this指向 存在将其对象化 不存在默认指向window
  thisArg = Object(thisArg ?? window)
  // 使用Symbol 防止篡改this绑定对象上的内容
  let sym = Symbol('this绑定')
  // 利用隐式绑定绑定this
  thisArg[sym] = self
  // 返回并调用该函数 并传入参数 返回箭头函数 防止this被再次改变
  return (...rest) => {
    // 将两者传入的参数合并传入
    const result = thisArg[sym](...thisArr, ...rest)
    delete thisArg[sym]
    return result
  }
}

nums.myBind('123', 1, 2)(3, 4)
```

#### 手写 once

```js
// once函数 表示只执行一次的函数
// 参数为要执行的函数
function myOnce(fun) {
  // 是否执行完成
  let flag = false
  
  return function (...rest) {
    // 当第一次执行完之后，falg的值会变为true，原因是利用了闭包的缓存
    if (!falg) {
      flag = true
      return fun.apply(this, rest)
    }
  }
}

// 进阶 可控制可执行次数的函数
function myOnceNum(fun, num = 1) {
  let performedNum = 0

  return function (...rest) {
    if (performedNum < num) {
      performedNum++
      return fun.apply(this, rest)
    }
  }
}
```

### 方法手写

#### 手写柯里化函数

```js
// 传入要柯里化的函数
function curringFun(fun) {
  // 柯里化函数之后返回一个新函数 接受传入的参数
  return function curring(...thisArr) {
    // 判断传入的实参个数与原函数形参个数
    if (thisArr.length >= fun.length) {
      // 返回并调用函数
      return fun.apply(this, thisArr)
    } else {
      // 传入的参数不够 返回一个函数接受接下来的参数
      return function (...rest) {
        // 递归调用curring函数，将接下来的值传入
        return curring.call(this, ...thisArr, ...rest)
      }
    }
  }
}

// 原函数
const fun = (a, b, c, d, e) => a + b + c + d + e
// 柯里化函数
const cur1 = curringFun(fun)
// 调用
console.log(cur1(1, 2, 3)(4)(5))
console.log(cur1(1)(2)(3)(4)(5))
console.log(cur1(1, 2, 3, 4, 5))
```

#### 手写组合函数

```js
// 接受传入的函数
function composeFun(...fun) {
  // 判断传入的参数是否都为函数，不为函数则抛出异常
  fun.forEach(item => {
    if (typeof item !== 'function') {
      // 期望传入的参数都为函数
      throw new TypeError('Expected arguments are functions')
    }
    // 获取传入的函数个数
    let length = fun.length
    // 将组合后的函数以函数的形式返回 并接受参数
    return function (...rest) {
      // 初始调用第一个函数
      let index = 0
      // 如果未传入函数 直接返回传入的参数
      let result = length ? fun[index].apply(this, rest) : rest
      // 依次调用函数
      while (++index < length) {
        result = fun[index].call(this, result)
      }
      return result
    }
  })
}

// 需求： 假设有三个函数，我们需要依次调用这三次函数 每个函数的值来源于上一个函数的返回值，第一个函数的值来源于传入的值
const fun1 = a => a + 2
const fun2 = a => a * 2
const fun3 = a => a - 10
// 分别调用
console.log(fun3(fun2(fun1(3)))) // 5
// 将函数组合
let com1 = composeFun(fun1, fun2, fun3)
console.log(com(3)) // 5

// 方案二 利用reduce递归
const pomose = (...Hoc) => {
  if (!Hoc.length) return Hoc
  return Hoc.reduce(
    (prev, next) =>
      (...rest) =>
        next(prev(...rest))
  )
}

const fun = pomose(fun1, fun2, fun3)
console.log(fun(3)) // 5
```

#### 手写 new 操作符

```js
// 手写new操作符
// 第一个参数为构造函数 第二个参数为构造函数接受的参数
function myNew(fun, ...rest) {
  // 因为构造函数返回的是一个对象 因此需要先声明一个对象
  let obj = {}
  // 将构造函数的显示原型赋给obj的隐式原型上
  obj.__proto__ = fun.prototype
  // 为构造函数赋值
  let result = fun.apply(obj, rest)
  // 这一步为了当构造函数内部有新的返回对象时， 可以调用
  return result instacnceof Object ? result : obj
}

// 使用
function Person(name, age) {
  this.name = name
  this.age = age
}
const per1 = myNew(Person, 'web', 22)
console.log(per1)  // Person {name: 'web', age: 22}

function Student(name, age) {
  this.name = name
  this.age = age
  return {
    info: '新的对象'
  }
}
const per2 = myNew(person, 'js', 20)
console.log(per2) // {info: '新的对象'}
```

#### 手写基于 thunk 的 Generator 自动执行函数

```js
// 传入一个Generator函数

function run(gen) {
  const fn = typeof gen === 'function' ? gen() : gen
  function next(val) {
    const result = fn.next(val)
    if (result.done) return result.value
    Promise.resolve(result.value).then(res => {
      next(res)
    })
  }
  next()
}
```

#### 手写顺序执行函数的方法（reduce）

```js
const pomose = (...Hoc) => {
  if (!Hoc.length) return Hoc
  return Hoc.reduce(
    (prev, next) =>
      (...rest) =>
        next(prev(...rest))
  )
}
const fun = pomose(fun1, fun2, fun3)
// 类似组合函数的 fun3(fun2(fun1(3)))
console.log(fun(3))
```

#### 手写顺序执行 promise 的方法（reduce）

```js
// 假设有三个promise方法，分别是登录login,获取用户信息getInfo，以及获取用户订单getOrder,要求用户登录后，依据上个方法的返回值以此执行这三次promise

/**
 *
 * @param promiseList promise方法列表
 * @param initData  初始化参数
 * @returns
 */
function promiseCompose(promiseList, initData) {
  if (!Array.isArray(promiseList))
    throw TypeError(`${promiseList} is not an Array`)

  try {
    return promiseList.reduce(
      (prev, next) => prev.then(res => next(res)),
      Promise.resolve(initData)
    )
  } catch (error) {
    console.warn(error)
  }
}

const promise = promiseCompose([login, getInfo, getOrder], {
  name: '新新小朋友',
  pass: 123123,
})
promise.then(res => {
  console.log(res)
})
```

### Vue 手写

#### 手写 Vue3 响应式原理

```js
// 全局用来收集响应式函数
let activeReactiveFn = null
// 存储全局依赖
const weakMap = new WeakMap()

// 创建一个依赖类，为响应式的每个对象设置依赖
class Depend {
  constructor() {
    // 存放的响应式数据不同属性的依赖
    this.activeReactiveArr = new Set()
  }
  // 收集依赖
  addDepend() {
    if (activeReactiveFn) {
      this.activeReactiveArr.add(activeReactiveFn)
    }
  }
  // 依次执行相应的依赖
  notify() {
    this.activeReactiveArr.forEach(item => item())
  }
}

// 响应式函数  收集要被响应式处理的依赖
function watchFn(fn) {
  activeReactiveFn = fn
  fn()
  activeReactiveFn = null
}

// 响应式对象的数据对象的属性依赖
function getDepend(target, key) {
  // 获取目标响应式对象
  let map = weakMap.get(target)
  // 第一次访问不存在 添加
  if (!map) {
    map = new Map()
    weakMap.set(target, map)
  }
  // 获取目标响应式对象的响应式属性依赖
  let depend = map.get(key)
  if (!depend) {
    depend = new Depend()
    map.set(key, depend)
  }
  // 返回响应式对象数据对应的属性依赖
  return depend
}

// 封装函数 普通对象转为响应式对象
function conversionResponsive(target) {
  return new Proxy(target, {
    get(target, key, receiver) {
      // 获取该属性的对应依赖
      const depend = getDepend(target, key)
      // 添加依赖
      depend.addDepend()
      return Reflect.get(target, key, receiver)
    },
    set(target, key, newVal, receiver) {
      Reflect.set(target, key, newVal, receiver)
      // 获取该属性的对应依赖
      const depend = getDepend(target, key)
      // 执行依赖
      depend.notify()
    },
  })
}

// 普通对象
const obj = {
  name: 'WEB架构师',
  info: 'JavaScript',
}

// 转为响应式对象
const objProxy = conversionResponsive(obj)

// 初始化页面渲染
function render() {
  document.querySelector('#h1').innerHTML = reactiveProxy.name
  document.querySelector('#h2').innerHTML = reactiveProxy.info

  // ... 1000行
}

render()

// 响应式数据变化之后的依赖
watchFn(() => {
  render()
  //  ...1000行
})

// 响应式数据变化
reactiveProxy.name = '123'
reactiveProxy.info = 'asdas'
```

#### 手写 vue2 响应式原理

```js
// 其他与上面的Vue3思维和过程一样 只是在原理处理上不一样
function conversionResponsive(target) {
  Object.keys(target).forEach(key => {
    let val = Reflect.get(target, key)
    Object.defineProperty(target, key, {
      set(newVal) {
        val = newVal
        // 依赖
        const depend = getDepend(target, key)
        depend.notify()
      },
      get() {
        const depend = getDepend(target, key)
        depend.addDepend()
        return val
      },
    })
  })
  return target
}
```

### 手写 Promise

```js

```

### 节流和防抖

#### 手写节流

```js
// 节流 就是在规定时间内只触发一次 比如轮播图左右切换
// 传入两个参数 第一个为目标函数 第二个为节流延时时间
function throttle(target, timer) {
  let time = null
  return function (event) {
    if (Date.now() - time > timer) {
      target.call(this, event)
      time = timer
    }
  }
}

// 普通要处理的函数
function btnClick(event) {
  console.log('点击了按钮', this, event)
}

// 调用节流函数
btn.addEventListener('click', throttle(btnClick, 2000), false)

// 手写节流函数Promise版 对于取值来说，并非单纯调用函数 使用Promise版本合适
function throttle(target, timeout = 1000) {
  let time = null
  return function (event) {
    const newTime = Date.now()
    return new Promise(resolve => {
      if (newTime - time >= timeout) {
        const result = target.call(this, event)
        time = newTime
        resolve(result)
      }
    })
  }
}

// 使用
function IIFE() {
  let a = 10
  let b = 20
  return a + b
}
// 使用节流
const newFun = throttle(IIFE, 2000)
setInterval(() => {
  newFun().then(res => console.log(res))
}, 10)
```

#### 手写防抖

```js
// 防抖 比如搜索框搜索内容 每次输入都要发送ajax请求，这会浪费服务器资源，因此等用户在规定时间内最后一次输入完 才发送ajax请求， 并且当用户首次输入的时候 会立即发送请求（升级版）
function shakePlus(target, timer, immediately = false) {
  // 声明定时器函数
  let time = null
  // 立即执行函数是否本次已经执行	 这里也使用到了闭包的缓存性
  let execute = false
  return function (event) {
    if (immediately && !execute) {
      target.call(this, event)
      execute = true
    }
    clearTimeout(time)
    time = setTimeout(() => {
      target.call(this, event)
      execute = false
    }, timer)
  }
}

// 普通执行函数
const changeInput = function (event) {
  console.log('输入了内容', this, event)
}

// 监听输入框函数 立即执行
input.addEventListener('input', shakePlus(changeInput, 1000, true), false)
```

### 手写 Mini-Vue

#### 概述

```js
// vue源码主要分为三大系统，分别式编译系统，响应式系统和渲染系统。

编译系统
编译系统负责将`template`结构转化为vnode对象

渲染系统
渲染系统赋值将vnode转化未真实的dom并且挂载到页面上

响应式系统
负责将普通数据对象转化未响应式数据对象

// 我们将分别模拟渲染系统和响应式系统，在渲染系统中，手写h(vnode)函数和mount(挂载)函数，以及patch(diff算法)函数比较新旧节点。在响应式系统中，手写响应式原理(Vue3)

// 分为一个HTML文件和三个JS文件
index.html  // 项目文件
index.js    // 入口文件
render.js   // 渲染文件
reactive.js // 响应式文件
```

#### reactive.js

```js
// 收集响应式函数
let reactiveFn = null
// WeakMap结构
const weakMap = new WeakMap()

// 依赖类，收集某对象某属性的依赖关系
class Depend {
  constructor() {
    this.reactiveArr = new Set()
  }
  // 收集依赖
  add() {
    if (!reactiveFn) return
    this.reactiveArr.add(reactiveFn)
  }
  // 通知依赖
  nofity() {
    this.reactiveArr.forEach(fn => fn())
  }
}

// 收集响应式函数
const watchFn = fn => {
  reactiveFn = fn
  fn()
  reactiveFn = null
}

// 获取某对象对应属性的依赖
const getDepend = (target, key) => {
  let map = weakMap.get(target)
  if (!map) {
    map = new Map()
    weakMap.set(target, map)
  }
  let depend = map.get(key)
  if (!depend) {
    depend = new Depend()
    map.set(key, depend)
  }
  return depend
}

// 封装响应式 普通对象转为响应式
const ProxyTarget = obj => {
  return new Proxy(obj, {
    get(target, key, receiver) {
      const depend = getDepend(target, key)
      depend.add()
      return Reflect.get(target, key, receiver)
    },
    set(target, key, newVal, receiver) {
      Reflect.set(target, key, newVal, receiver)
      const depend = getDepend(target, key)
      depend.nofity()
    },
  })
}
```

#### render.js

```js
// 编写h函数 模拟vnode
// tag: HTML元素的类型 'div' string
// props: HTML元素的属性 class='aaa' {}|null
// children: HTML元素的内容或子节点 string|Array<h>
const h = (tag, props, children) => {
  return {
    tag,
    props,
    children,
  }
}

// 编写mount函数，挂载函数，将h函数生成的vnode渲染到页面上
// vnode: 虚拟节点
// container: 挂载的容器
const mount = (vnode, container) => {
  const el = (vnode.el = document.createElement(vnode.tag))
  // 渲染Props
  if (vnode.props !== null) {
    Object.keys(vnode.props).forEach(key => {
      const val = Reflect.get(vnode.props, key)
      // 判断是不是事件
      if (key.startsWith('on')) {
        el.addEventListener(key.slice(2).toLowerCase(), val, false)
      } else {
        el.setAttribute(key, val)
      }
    })
  }
  // 判断是否有子节点
  if (typeof vnode.children === 'string') {
    el.textContent = vnode.children
  } else {
    vnode.children.forEach(key => render(key, el))
  }
  container.appendChild(el)
}

// diff算法 打补丁 比较新旧节点
const patch = (n1, n2) => {
  // 判断两个节点类型是否一致
  if (n1.tag !== n2.tag) {
    // 获取旧节点挂载的父节点
    const parentElement = n1.el.parentElement
    // 删除旧节点
    parentElement.removeChild(n1.el)
    // 挂载新节点
    mount(n2, parentElement)
  } else {
    // 获取el元素
    const el = (n2.el = n1.el)
    // 旧节点props
    const oldProps = n1.props ?? {}
    // 新节点Props
    const newProps = n2.props ?? {}
    // 将所有的新节点添加到el
    Object.keys(newProps).forEach(key => {
      const val = Reflect.get(newProps, key)
      // 判断是不是事件
      if (key.startsWith('on')) {
        el.addEventListener(key.slice(2).toLowerCase(), val, false)
      } else {
        el.setAttribute(key, val)
      }
    })
    // 将所有的旧节点移除
    Object.keys(oldProps).forEach(key => {
      const val = Reflect.get(oldProps, key)
      // 移除所有事件
      if (key.startsWith('on')) {
        el.removeEventListener(key.slice(2).toLowerCase(), val, false)
      }
      // 移除无用的属性
      if (!(key in newProps)) {
        el.removeAttribute(key)
      }
    })

    // 判断新节点是否为字符串
    if (typeof n2.children === 'string') {
      // 判断旧字符串是否为字符串
      if (typeof n1.children === 'string') {
        // 判断是否一样 不一致更新
        if (n1.children !== n2.children) {
          el.textContent = n2.children
        }
      } else {
        // 添加新的内容
        el.innerHTML = n2.children
      }
    } else {
      // 当旧节点为字符串
      if (typeof n1.children === 'string') {
        el.textContent = ''
        n2.children.forEach(key => render(key, el))
      } else {
        // 当新旧节点都为子节点
        // 旧节点的长度
        const oldChildLen = n1.children.length ?? 0
        const newChildLen = n2.children.length ?? 0
        const { min } = Math
        // 获取新旧节点最小长度
        const minLen = min(oldChildLen, newChildLen)
        for (let i = 0; i < minLen; i++) {
          patch(n1.children[i], n2.children[i])
        }
        // 当新节点长度大于旧节点长度
        if (newChildLen > oldChildLen) {
          n2.children.slice(oldChildLen).forEach(key => {
            mount(key, el)
          })
        }
        // 当新节点长度小于旧节点长度
        if (newChildLen < oldChildLen) {
          n1.children.slice(newChildLen).forEach(key => {
            el.removeChild(key.el)
          })
        }
      }
    }
  }
}
```

#### index.js

```js
// 程序的入口函数
// App: 挂载的数据和vnode data: ProxtTarget({})  render(return h())
const creatApp = App => {
  return {
    // 挂载容器
    mount(container) {
      const dom = document.querySelector(container)
      // 是否为首次挂载 第一次为挂载 第二次为节点更新
      let isMount = false
      // 旧节点
      let oldVnode = null
      // 加入响应式系统
      watchFn(() => {
        if (!isMonut) {
          oldVnode = App.render()
          mount(oldVnode, dom)
          isMount = true
        } else {
          const newVnode = App.render()
          patch(oldVnode, newVnode)
          oldVnode = newVnode
        }
      })
    },
  }
}
```

#### index.html

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
  </head>

  <body>
    <h2>vue源码主要分为三大系统，分别是编译系统，响应式系统，渲染系统</h2>

    <div id="app"></div>

    <!-- 引入响应式系统 -->
    <script src="./reactive.js"></script>
    <!-- 引入渲染系统 -->
    <script src="./render.js"></script>
    <!-- 程序入口 -->
    <script src="./index.js"></script>
    <script>
      const App = {
        data: ProxtTarget({
          name: 'WEB架构师',
          age: 22,
          text: '文本框输入',
        }),
        render() {
          return h('div', null, [
            h('h2', { class: 'h2' }, `name: ${this.data.name}`),
            h('h2', { class: 'h2' }, `age: ${this.data.age}`),
            h(
              'button',
              {
                class: 'btn',
                onClick: () => {
                  this.data.age++
                },
              },
              'age ++'
            ),
            h(
              'input',
              {
                class: 'input',
                onInput: $event => {
                  this.data.text = $event.target.value
                },
                value: `${this.data.text}`,
              },
              ``
            ),
            h('h3', null, `${this.data.text}`),
          ])
        },
      }

      const app = creatApp(App)
      app.mount('#app')
    </script>
  </body>
</html>
```
