> 本练习对应的GitHub仓库： https://github.com/xinxin1228/js-combat-practice

## 面向对象

### 渲染自定义 HTML 组件

#### 需求描述

- 在`html`模版里直接编写**自定义、非`html`**的标签元素，并且支持多种写法。

  比如`<my-title></my-title>`、`<my-button></my-button>`或`<MyTitle></MyTitle>`、`<MyButton></MyButton>`

- 不影响正常的`html标签元素`的展示

- 能绑定事件，可以处理自定义标签特有的属性，并且支持在自定义标签上写`class`

  `<my-title level='1'></my-title>`中，`level`代表自定义元素的等级，比如默认为`h1`，根绝`level`的值自动生成`h1~h6`之间的标签元素

  `<my-button type='primary' onClick='handleClick'></my-button>`中，`type`指定按钮的状态类型，比如`primary`、`error`等，`onClick`代表给该自定义元素绑定的事件类型

#### 准备工作

自定义标签`html`代码

```html
<div id="app">
  <h1>我是标准的h1标签</h1>
  <my-button onclick="handleClick" class="submit xx my-button" type="primary">
    我是my-button按钮
  </my-button>
  <my-title class="blue">我是my-title</my-title>
  <MyButton onclick="handleClick2" type="error">
    支持多种写法 以及事件绑定
  </MyButton>
  <MyTitle level="3">支持h1～h6的多种标签，默认为h1 </MyTitle>
</div>
```

#### 效果展示

渲染自定义标签之前的页面样式和控制台的中`html`结构

##### 渲染之前

![渲染之前](../image/前端笔记/24.jpg)

##### 渲染之后

![渲染之后](../image/前端笔记/25.jpg)

#### 仓库地址

https://github.com/xinxin1228/js-combat-practice/tree/main/%E6%B8%B2%E6%9F%93%E8%87%AA%E5%AE%9A%E4%B9%89%E7%BB%84%E4%BB%B6

### Store 实现多个 class 之间数据共享

#### 需求描述

- 封装一个高阶类，可以获取数据和存储数据，提供一下的方法

  ```js
  // 获取存储库中的 [key] 的值
  get(key)
  
  // 往存储库中存储 [key]: [value]
  set(key, value)
  
  // 判断是 [key] 是否存在存储库
  has(key)
  
  // 获取存储库中的所有数据
  show()
  
  // 删除存储库中的 [key]
  deleteByKey(key)
  ```

  

- 通过该高阶类，可以做到在多个类中数据进行流通

#### 所用的知识

**HOC 高阶类**

1. 在父类使用子类才有的东西
2. 类写完不直接用，而是进行包裹

#### 效果展示

```js
import { store } from './Store.js'

const A = stroe.connect(class {...})
const B = store.connect(class {...})

const a = new A()
const b = new B()

console.log(a.get('name')) // undefined
console.log(a.get('name')) // undefined
a.set('name', 'web')
console.log(a.get('name')) // 'web'
console.log(b.get('name')) // 'web'
```

#### 仓库地址

https://github.com/xinxin1228/js-combat-practice/tree/main/Store
