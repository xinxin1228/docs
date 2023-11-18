> 所有【JavaScript实战练习】对应的 GitHub 仓库： https://github.com/xinxin1228/js-combat-practice

### 需求描述

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

### 所用的知识

**HOC 高阶类**

1. 在父类使用子类才有的东西
2. 类写完不直接用，而是进行包裹

### 效果展示

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

### 仓库地址

https://github.com/xinxin1228/js-combat-practice/tree/main/Store