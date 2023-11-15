> 所有【JavaScript实战练习】对应的 GitHub 仓库： https://github.com/xinxin1228/js-combat-practice

## webpack-ts版本

### 需求描述

事件队列的作用和用途在这里不再赘述，本案例的目标是在于手写一个类似于`pubsub-js`这样的事件队列库，实现该库中的以下方法：

- `subscribe` 订阅
- `publish` 发布
- `unsubscribe`取消订阅
- `clearAllSubscriptions` 清除所有的订阅
- `getSubscriptions` 获取订阅
- `countSubscriptions` 统计订阅
- 错误处理

实现事件队列库后，分别在`vue`、`react`中引用该库，实现**兄弟组件通信**和**跨级组件通信**，组件之间的关系架构图如下：

![组件关系架构图](../../image/前端笔记/27.png)

要求如下：

- `App`是根组件，在`App`组件中，引入`子组件1`和`子组件2`。
- 在`App`组件中，包含了数据 `count`、`mes`以及方法`addCount`。其中`count`数据类型为`Number`，通过`addCount`方法可以改变`count`的值。
- 在`子组件1`中，包含了自身的数据`a`、`b`以及方法`addA`和`addB`，分别用来改变`a`和`b`的值。并且引用了`App`中的数据`a`和方法`addCount`。
- 在`子组件2`中，要求可以通过点击按钮调用子组件1的`addA`和`addB`方法。并且在`子组件2`中，引入`子组件3`。
- 在`子组件3`中，要求可以通过点击按钮调用子组件1的`addA`和`addB`方法。

### 所用的知识

- vue
- react
- typescript

### 效果展示

![pubsub使用场景演示](../../image/前端笔记/06.gif)

### 步骤详解

关于`webpack`如何搭建`typescript`、`vue`以及`react`环境，由于并不是该节内容的主要目标，因此这里不再详解，不懂或感兴趣的可以参考[本节代码地址](https://github.com/xinxin1228/js-combat-practice/tree/main/%E6%89%8B%E5%86%99%E7%B1%BB%E4%BC%BCpubsub-js%E7%9A%84%E4%BA%8B%E4%BB%B6%E9%98%9F%E5%88%97%E5%BA%93)或往期文章[Webpack搭建工程化项目](/docs/前端笔记/Webpack搭建工程化项目)

#### 一、先熟悉`pubsub-js`的用法

```js
// 导入pubsub
import pubsub from 'pubsub'

/**
 * 订阅主题 
 * 订阅主题为 add 的主题，并且传入该主题变化之后的处理函数
 * @param topic 主题
 * @param callback 主题下的事件处理函数
 * @returns {string} uid 返回该主题的uid
 */
const uid = pubsub.subscribe('add', (...rest) => {
  console.log('add主题发生了变化', rest)
})

/**
 * 发布主题
 * 出发主题为 add 的主题，并且传入想传入的参数
 * @param topic 主题
 * @param rest 参数
 * @returns {boolean} result 是否发布成功
 */
pubsub.publish('add', { id: 1, name: 'web' })

/**
 * 取消订阅 支持通过uid和mes（前置匹配）来清除
 * 匹配顺序  mes > uid
 * 如果匹配到了前置匹配，那么优先清除前置匹配到，不清除uid
 * @param params 需要清除的uid | uid数组 | mes | mes数组
 * @param deep 针对mes,是否深度匹配，比如 mes为 'a'时，清除'a'和'a.*'的所有
 */
pubsub.unsubscribe(uid) // 取消uid
pubsub.unsubscribe(['a', 'b']) // 取消所有以 a 和 b 开头的主题

/**
 * 统计订阅 仅支持订阅名称（精准匹配）不支持uid
 * @param topic 订阅的名称
 */
countSubscriptions('aa')

// 清除所有的订阅
clearAllSubscriptions()

/**
 * 查看当前类型下的所有订阅主题 （前置匹配） 不支持uid
 * @param topic 订阅主题的名称 比如 topic='x'时，可以查看到['x', 'xx', 'x1']
 */
getSubscriptions('x')
```

#### 二、根据数据存放方式来确定数据结构

首先，我们需要记录**某一个主题（topic）**下的所有**处理函数（callback）**，这时我们想到了`JSON`的键值对格式，如下所示：

```js
// 主题与主题对应处理函数的关系
const queue = {
  addA: [fn, fn, fn],
  addB: [fn, fn, fn]
}
```

但是我们不能直接将处理函数放入对应主题的函数列表中，因为在`unsubscribe`方法中，需要根据`uid`或`topic`来取消订阅主题，也就是删除某一个主题函数列表中的`fn`，但是如果按照上面的结构，我们很难知道具体是要删除哪一个`fn`。

因此我们需要改变一下数据结构：主题列表中不再存放事件处理函数`fn`，而是存放**订阅主题后返回的uid**。并且使用其他的变量来记录`uid`和`fn`之间的关系。如下所示：

```js
// 主题与订阅uid的关系
const queue = {
  addA: [uid1, uid2, uid3],
  addB: [uid4, uid5, uid6]
}

// uid与主题对应处理函数的关系
const uidList = {
  uid1: { topic: addA, callback: fn1 },
  uid4: { topic: addB, callback: fn4 },
  uid3: { topic: addA, callback: fn3 },
  ...
}
```

这样我们想要取消订阅`uid3`这个主题，我们只需要先在`uidList`中找到`uid3`对应的值`{ topic: addA, callback: fn3 }`，然后从`uidList`中移除`uid3`，再根据 `topic: addA` 去`queue`中移除`addA`中的`uid3`即可。逻辑代码如下：

``` js
// PS: 以下代码为逻辑演示代码，并非实际运行代码，重点在于思想

// 假设我们要取消订阅 uid3

// 这里我们将 object类型替换为map类型、array类型替换为set类型
const queue = new Map()
const uidList = new Map()

queue.set('addA', uid1)
queue.set('addA', uid2)
queue.set('addB', uid3)
uidList.set(uid1, { topic: 'addA', callback: fn1 })
uidList.set(uid2, { topic: 'addA', callback: fn2  })
uidList.set(uid3, { topic: 'addB', callback: fn3 })

// 第一步 先从 uidList 中找到uid3的topic信息
const topicInfo = uidList.get(uid3)  // { topic: 'addB', callbcak: fn3 }

// 第二步 从 uidList 中移除uid3
uidList.delete(uid3)

// 第三步 从 queue 中删除 addB 中的uid3，并重新赋值 addB
const queueInfo = queue.get(topicInfo.topic)

queueInfo.delete(uid3)
queue.set(topicInfo.topic, queueInfo)
```

#### 三、确定设计模式

我们期望提供一个唯一实例，这样在整个应用程序中共享相同状态或资源。因此我们采用**单例模式**。

```js
// pubsub.ts 导出
class Pubsub {
  ...
}
  
// 在内部实例化，这样全局都调用的是同一个实例
const pubsub = new Pubsub()

export default pubsub


// a.ts文件 导入
import pubsub from './pubsub'

pubsub.subscribe('addA', () => {})

// b.ts文件 导入
import pubsub from './pubsub'

pubsub.publish('addA', 11)
```

### 代码编写

本案例采用`TypeScript`编写，除了`函数重载`部分，别的与`JavaScript`一致

#### `Pubsub类`

```js
class Pubsub {
  private queue: Map<string, Set<string>>
  private uidList: Map<string, any>
  private uid: number
  
  constructor() {
    this.queue = new Map()
    this.uidList = new Map()
    this.uid = 0
  }
}

const pubsub = new Pubsub()

export default pubsub
```

#### `subscribe(订阅主题)`

```js
/**
 * 订阅主题
 * @param topic 主题名称
 * @param callback 该主题对应的处理函数
 * @returns {string} uid
 */
subscribe(topic: string, callback: () => any): string {
  const topicInfo = this.queue.get(topic) || new Set()
  const uid = `uid_${this.uid}`

  topicInfo.add(uid)
  this.uidList.set(uid, {
    topic,
    callback
  })
  this.queue.set(topic, topicInfo)
  this.uid++

  return uid
}
```

#### `publish(发布主题)`

```js
/**
 * 发布主题
 * @param topic 主题
 * @param rest 参数
 * @returns {boolean} result 是否发布成功
 */
publish(topic: string, ...rest: any[]): boolean {
  const topicInfo = this.queue.get(topic) //map

  if (!topicInfo) return false
  if (!topicInfo.size) return false

  topicInfo.forEach((uid) => {
    const callback = this.uidList.get(uid)?.callback
    callback?.(topic, ...rest)
  })

  return true
}
```

#### `unsubscribe(取消订阅)`

```js
/**
 * 取消订阅 支持通过uid和topic（主题名称前置匹配）来清除
 * 匹配顺序  topic > uid
 * 如果匹配到了前置匹配，那么优先清除前置匹配到，不清除uid
 * @param params 需要清除的uid | uid数组 | topic | topic数组
 * @param deep 针对topic,是否深度匹配，比如 topic为 'a'时，清除'a'和'a.*'的所有
 * @returns {string | boolean} 如果传入的是uid，则返回boolean,如果传入的是topic，则返回topic
 */
unsubscribe(uid: string | string[]): string | boolean
unsubscribe(topic: string | string[], deep: boolean): string | boolean
unsubscribe(params: string | string[], deep = true) {
  // 是否匹配到前置匹配 如果匹配到之后 不再继续删除uid
  let isDelete = false

  if (Array.isArray(params)) {
    params.forEach((u) => {
      // topic
      this.queue.forEach((list, key) => {
        // deep ? 深度匹配 ：全等
        if (deep ? key.startsWith(u) : key === u) {
          list.forEach((uid) => {
            this.uidList.delete(uid)
          })
          this.queue.delete(key)
          isDelete = true
        }
      })

      if (isDelete) return true

      // uid
      const isHas = this.uidList.has(u)
      if (!isHas) return false

      const info = this.uidList.get(u)
      const queueInfo = this.queue.get(info.topic)
      if (!queueInfo || !queueInfo.size) return true

      queueInfo.delete(u)
      this.uidList.delete(u)
      this.queue.set(info.topic, queueInfo)
      !queueInfo.size && this.queue.delete(info.topic)

      return params
    })
  } else {
    // topic
    this.queue.forEach((uidList, key) => {
      // deep ? 深度匹配 ：全等
      if (deep ? key.startsWith(params) : key === params) {
        uidList.forEach((uid) => {
          this.uidList.delete(uid)
        })
        this.queue.delete(key)
        isDelete = true
      }
    })

    if (isDelete) return true

    // uid
    const isHas = this.uidList.has(params)
    if (!isHas) return false

    const info = this.uidList.get(params)
    const queueInfo = this.queue.get(info.topic)
    if (!queueInfo) return true

    queueInfo.delete(params)
    this.uidList.delete(params)
    this.queue.set(info.topic, queueInfo)
    !queueInfo.size && this.queue.delete(info.topic)

    return params
  }
}
```

#### `clearAllSubscriptions(清除所有的订阅)`

```js
clearAllSubscriptions(): void {
  this.queue.clear()
  this.uidList.clear()
  this.uid = 0
}
```

#### `countSubscriptions(统计订阅)`

```js
/**
 * 统计订阅 仅支持订阅名称（精准匹配）不支持uid
 * @param topic 订阅的名称
 */
countSubscriptions(topic: string | string[]): number | object {
  if (Array.isArray(topic)) {
    const topicInfo = {} as any

    topic.forEach((m) => {
      topicInfo[m] = this.queue.get(m)?.size || 0
    })

    return topicInfo
  } else {
    return this.queue.get(topic)?.size || 0
  }
}
```

#### `getSubscriptions(查看当前类型下的所有订阅主题)`

```js
/**
 * 查看当前类型下的所有订阅主题 （前置匹配） 不支持uid
 * @param topic 订阅主题的名称 比如 topic='x'时，可以查看到['x', 'xx', 'x1']
 */
getSubscriptions(topic: string | string[]): string[] {
  const topicList: Array<string> = []

  if (Array.isArray(topic)) {
    topic.forEach((item) => {
      this.queue.forEach((_, key) => {
        if (key.startsWith(item)) {
          topicList.push(key)
        }
      })
    })
  } else {
    this.queue.forEach((_, key) => {
      if (key.startsWith(topic)) {
        topicList.push(key)
      }
    })
  }

  return topicList
}
```

### 仓库地址

https://github.com/xinxin1228/js-combat-practice/tree/main/%E6%89%8B%E5%86%99%E7%B1%BB%E4%BC%BCpubsub-js%E7%9A%84%E4%BA%8B%E4%BB%B6%E9%98%9F%E5%88%97%E5%BA%93/webpack

## vite-ts版本

### 说明

除了将`webpack`替换为`vite`，`pubsub`的核心代码和步骤与`webpack-ts版本`的代码一致，这里不在赘述。

另：由于`vite`天然支持`typescript`，因此在`vite-ts版本`中，`vue`和`react`均结合`typescript`进行编写。

另：如果对`vite`搭建工程化项目不太熟悉的话，可以去参考我之前写的[Vite 搭建工程化项目](http://localhost:3000/#/docs/前端笔记/Vite搭建工程化项目)这篇文章

### 仓库地址

https://github.com/xinxin1228/js-combat-practice/tree/main/%E6%89%8B%E5%86%99%E7%B1%BB%E4%BC%BCpubsub-js%E7%9A%84%E4%BA%8B%E4%BB%B6%E9%98%9F%E5%88%97%E5%BA%93/vite