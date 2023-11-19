> 所有【JavaScript实战练习】对应的 GitHub 仓库： https://github.com/xinxin1228/js-combat-practice

## 需求描述

> 现在主流框架，无论是`vue`还是`react`都采用了虚拟 DOM 技术

## 采用虚拟 DOM 的优势

- 简单方便：如果使用手动操作真实 **DOM** 来完成页面，繁琐又容易出错，在大规模应用下维护起来也很困难
- 性能方面：使用**Virtual** **DOM**，能够有效避免真实**DOM** 频繁更新，减少多次引起重绘与回流，提高性能
- 跨平台：React 借助**虚拟 DOM**，带来了跨平台的能力，一套代码多端运行

既然**虚拟 DOM**现在已然是前端的热门话题，而且面试时，时常要求手写出一个真实 DOM 的虚拟 DOM，那么本节的主题就是**手写一个 class 来实现从真实 DOM 抽象为虚拟 DOM**

## 效果展示

### 真实 DOM

```html
<div id="app">
  <h2 class="my-title">我是h2标签</h2>
  我是一段文本
  <ul class="my-ul">
    <li class="my-li">标签一</li>
    <li class="my-li">标签二</li>
  </ul>
  <button type="button" onclick="alert('hello')">我是一个按钮</button>
</div>
```

### 虚拟 DOM

> 本效果只展示虚拟 DOM 的应该有的结构和数据，实际生成的虚拟 DOM 可以不完全参考该案例

```js
// 数据结构
{
  type: 'div', // 对应真实DOM的类型，比如diva,ul,li等
  props: {     // 对应真实DOM的attribute属性和自定义属性等
    id: 'app',
    onclick: 'alert("hello")'
  },
  children: []   // 对应真实DOM的子节点，如果是文本，那么该子节点的类型应该是string，并且值就是该文本内容。如果该元素的子节点是其他节点，那么该节点的类型应该是array，内容是对应子节点的vDOM
}


// 比如上面的真实DOM转化之后的vDOM
[
  {
    type: 'div',
    props: {
      id: '#app'
    },
    children: [
      {
        type: 'h2',
        props: { class: 'my-title' },
        children: '我是h2标签'
      },
      {
        type: 'text',
        props: {},
        children: '我是一段文本'
      },
      {
        type: 'ul',
        props: { class: 'my-ul' },
        children: [
          {
            type: 'li',
            props: { class: 'my-li' },
            chilren: '标签一'
          },
          {
            type: 'li',
            props: { class: 'my-li' },
            chilren: '标签二'
          },
        ]
      },
      {
        type: 'button',
        props: {
          type: 'button',
          onclick: 'aleart("hello")'
        },
        children: '我是一个按钮'
      }
    ]
  }
]
```

## 解题思路

### 一：我们需要几个类来管理 VDOM？

经过我们上面的写的`VDOM`抽象，发现`VDOM`其实主要分为两大类，分别是

- 管理非文本节点类型元素的`VElement`
- 管理文本节点类型的`VText`

为了更好的符合面向对象的逻辑和写法，我们将再**抽象**出一个公共父类——`VNode`。

`VNode`的主要目的是存储渲染之前的`DOM`节点信息，以及处理`VElement`与`VText`公共的**逻辑部分**，尽管目前在本例并没有具体过多的实现，但这依旧是一种很好的面向对象编程思想。

目前的几个类型的关系与说明如下图所示：

![类关系说明](../../image/前端笔记/26.png)

### 二：真实 DOM 转 VDOM 的入口应该是哪个类？

- 首先 VNode 是抽象类，因此它并不能做真实 DOM 的入口来使用
- VText 是专门处理文本类型的 VDOM 类，因此并不适合当作真实 DOM 的入口来使用
- VElement 是处理非文本节点的类，而真实的 DOM 节点（上述案例）是`DIV`节点，因此我们可以使用 VElement 作为入口

## 仓库地址

https://github.com/xinxin1228/js-combat-practice/tree/main/%E7%94%9F%E6%88%90%E8%99%9A%E6%8B%9FDOM