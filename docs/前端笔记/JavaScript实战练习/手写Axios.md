> 所有【JavaScript实战练习】对应的 GitHub 仓库： https://github.com/xinxin1228/js-combat-practice

## 手写 webpack-axios 版本

### 所用的知识

- webpack

- XMLHttpRequest

- axios 使用

- Proxy

- Promise

### 实现的功能

- 发送请求 （`GET`、`POST`、`PUT`、`DELETE`）
- 可多种方式调用传参（`axios[method](url[,data[,config]])`）
- 自定义配置（`{baseURL, transformRequest...}`）
- 进度监控（`{onDownProgress, onUploadProgress}`）
- 拦截器（`axios.interceptors.[request,response].use`）

### 项目运行

```shell
# 安装依赖
pnpm i

# 运行
pnpm dev

# 打包
pnpm build
```

### 步骤详解

#### 一、编写类文件`Axios`

由于`axios`在调用时，既可以`axios()`调用，可以`axios[method]()`调用，因此这里我使用`proxy`来实现该功能。

```js
class Axios {
	constructor() {
    return new Proxy(this.request, {
      // 这一步是为了 axios() 的时候，调用this.request方法
      apply: (fn, _, argArray) => {
        return fn.apply(this, argArray)
      },
      // 这一步是为了 axios.get() 的时候，调用this.get方法
      get: (_, propName) => {
        return this[propName]
      },
      // 这一步的目的是为了给axios实例赋值，比如axios.defaults = ...
      set: (_, propName, newVal) => {
        this[propName] = newVal
        return true
      }
    })
  }

  // axios() 默认调用使用这个方法，axios === axios.request
  request() {}

  get() {}
  post() {}
  put() {}
  delete() {}

  create() {
    const axios = new Axios()

    // 默认
   	axios.defaults = { ... }

    return axios
  }
}

// 这一步的目的是将 axios实例上的create方法复制给Axios的静态方法上，这样方便用户可以 Axios.create() 或 axios.create() 创建一个新的axios实例
Axios.create = Axios.prototype.create

export default Axios.create()
```

到了这一步，我们可以引入我们写的文件并且调用 axios，比如

```js
import axios from './axios'

axios()

axios.get()

axios.post()

axios.put()

axios.delete()

// 创建新的axios实例
axios.create()

axios.defaults.baseURL = 'http://localhost:3000'
```

#### 二、编写默认配置文件，并且在每次`axios`实例后赋值

```js
// defaultConfig.js
export default {
  baseURL: '', // 请求主地址
  method: 'get', // 请求方式
  headers: {
    common: {
      Accept: 'application/json',
      'Content-Type': 'application/json',
    },
    get: {},
    post: {},
    put: {},
    delete: {},
  },
  // 请求转化，可以用于修改和添加配置信息，优先级高级默认配置，但低于拦截器
  transformRequest(config) {
    return config
  },
  // 响应的数据转化，用于修改 response.data
  transformResponse(data) {
    try {
      return JSON.parse(data)
    } catch {
      return data
    }
  },
}

// axios每次实例后赋值
import defaultConfig from './defaultConfig'

class Axios {
  // ...  同第一步

  // config 外面传入的配置
  create(config) {
    const axios = new Axios()

    // PS：暂时我们先以简单浅拷贝的形式进行对象之前的合并，但是这种方式有明显的缺点。例如： { a: { name: 'web' } } 和 { a: { age: 22 } }合并时，后者的a会替换前者的a，成为 { a: { age: 22 } }，而并非我们所期望的 { a: { name: 'web', age: 22 } }
    axios.defaults = { ...defaultConfig, ...config }

    return axios
  }
}
```

既然发现上面代码中，`利用对象的浅拷贝合并对象的缺点后`，我们来编写一个方法用来对象之间的合并。

编写`utils/mergeConfig.js`

```js
import deepCopy from './deepCopy' // 深拷贝
import { isArray, isTypeToString } from './check' // 类型校验

/**
 * 对象的合并
 * @param {object} target 默认对象
 * @param  {Array<object>} mergeObj 需要合并的对象，合并优先级根据编写顺序，越后的优先级越高.PS: mergeConfig(target, obj1, obj2, obj3)
 * 优先级：obj3 > obj2 > obj1
 * @returns
 */
export default function mergeConfig(target, ...mergeObj) {
  // 处理 单个合并
  if (mergeObj.length === 1) {
    // 取出第一项 也就是唯一一项
    mergeObj = mergeObj[0]
    // 这里进行深拷贝，防止副作用 篡改传入的对象
    target = deepCopy(target, isArray(target) ? [] : {})
    mergeObj = deepCopy(mergeObj, isArray(mergeObj) ? [] : {})

    for (const key in mergeObj) {
      let oldVal = target[key]
      let newVal = mergeObj[key]

      if (typeof newVal === 'object') {
        // 针对null情况
        if (!newVal) {
          target[key] = newVal
          continue
        }

        // 原对象与新对象类型不一致，直接将原对象清空，赋值为新对象的类型
        if (isTypeToString(newVal) !== isTypeToString(oldVal)) {
          oldVal = isArray(newVal) ? [] : {}
        }

        target[key] = mergeConfig(oldVal, newVal)
      } else {
        target[key] = newVal
      }
    }
  } else {
    // 只要mergeObj数组中还有对象，就递归的取出mergeObj的第一项，然后进行合并
    while (mergeObj.length) {
      let obj = mergeObj.shift()

      target = mergeConfig(target, obj)
    }
  }

  return target
}
```

这里就不列出深拷贝以及工具函数的代码，具体可以查看仓库中的完整代码。

那么利用上面的`mergeConfig`，默认赋值的写法可以改为：

```js
create(config) {
  const axios = new Axios()

  // 合并配置对象
  axios.default = mergeConfig(defaultConfig, config)

  return axios
}
```

#### 三、针对不同的请求方式、传参方式，汇总出一份配置文件

我们先来总结一下不同的`axios`调用方式以及传参方式

`axios`的不同传参方式

- `axios(url)` ： 只传入`url`，默认是`get`请求

- `axios(url, config)`：`url`为请求地址，`config`为请求配置项

- `axios(config)`：`config`为请求配置项

`axios.get`的不同传参方式

- `axios.get(url)`：`url`为请求地址

- `axios.get(url, config)`：`url`为请求地址，`config`为请求配置项

`axios.post`的不同传参方式

- `axios.post(url)`：`url`为请求地址
- `axios.post(url, data)`：`url`为请求地址，`data`为携带的数据
- `axios.post(url, data, config)`：`url`为请求地址，`data`为携带的数据，`config`为请求配置项

`axios.put`的不同传参方式

- `axios.put(url)`：`url`为请求地址
- `axios.put(url, data)`：`url`为请求地址，`data`为携带的数据
- `axios.put(url, data, config)`：`url`为请求地址，`data`为携带的数据，`config`为请求配置项

`axios.delete`的不同传参方式

- `axios.delete(url)`：`url`为请求地址
- `axios.delete(url, config)`：`url`为请求地址，`config`为请求配置项

```js
class Axios {
  // ...

  // 预处理，处理所有请求方式相同的方法
  // 比如 axios[method](url)
  _preprocessing(method, rest) {
    let config = null

    if (rest.length === 1 && isString(rest[0])) {
      config = { url: rest[0], method }
    } else {
      config = undefined
    }

    return config
  }

  get(...rest) {
    let config = this._preprocessing('get', rest)

    // 但 !config = true 时，说明没有匹配到上面的 预处理 中的参数处理方式
    if (!config) {
      if (rest.length === 2 && isObject(rest[1])) {
        config = { ...rest[1], url: rest[0], method: requestMethod.GET }
      } else {
        throwError(true, '参数不合法')
      }
    }

    // 将传入的配置文件整理后传入 统一的合并参数方法中
    return this._mergeConfig(config)
  }

  post(...rest) {
    let config = this._preprocessing('post', rest)

    if (!config) {
      if (rest.length === 2) {
        throwError(!isString(rest[0]), 'url必须是string类型')
        throwError(
          !isObject(rest[1]) && !isArray(rest[1]),
          'data必须是object类型'
        )

        config = { url: rest[0], data: rest[1], method: requestMethod.POST }
      } else if (rest.length === 3) {
        throwError(!isString(rest[0]), 'url必须是string类型')
        throwError(
          !isObject(rest[1]) && !isArray(rest[1]),
          'data必须是object类型'
        )
        throwError(!isObject(rest[2]), 'config必须是json类型')

        config = {
          ...rest[2],
          url: rest[0],
          data: rest[1],
          method: requestMethod.POST,
        }
      } else {
        throwError(true, '参数不合法')
      }
    }

    return this._mergeConfig(config)
  }

  put(...rest) {
    let config = this._preprocessing(requestMethod.PUT, rest)

    if (!config) {
      if (rest.length === 2) {
        throwError(!isString(rest[0]), 'url必须是string类型')
        throwError(
          !isObject(rest[1]) && !isArray(rest[1]),
          'data必须是object类型'
        )

        config = { url: rest[0], data: rest[1], method: requestMethod.PUT }
      } else if (rest.length === 3) {
        throwError(!isString(rest[0]), 'url必须是string类型')
        throwError(
          !isObject(rest[1]) && !isArray(rest[1]),
          'data必须是object类型'
        )
        throwError(!isObject(rest[2]), 'config必须是json类型')

        config = {
          ...rest[2],
          url: rest[0],
          data: rest[1],
          method: requestMethod.PUT,
        }
      } else {
        throwError(true, '参数不合法')
      }
    }

    console.log('put', config)
    return this._mergeConfig(config)
  }

  delete(...rest) {
    let config = this._preprocessing(requestMethod.DELETE, rest)

    if (!config) {
      if (rest.length === 2 && isObject(rest[1])) {
        config = { ...rest[1], url: rest[0], method: requestMethod.DELETE }
      } else {
        throwError(true, '参数不合法')
      }
    }

    console.log('delete', config)
    return this._mergeConfig(config)
  }
}
```

我们将不同的请求的参数进行了处理，然后将处理好的参数进行统一处理合并

```js
import urlLib from 'url'

// 合并参数 调用发起请求
  _mergeConfig(config) {
    let { method, url } = config
    let headers = {}

    method = method.toLowerCase()

    // 计算headers
    headers = mergeConfig(
      defaultConfig['headers'],
      defaultConfig['headers']['common'],
      defaultConfig['headers'][method],
      config['headers'],
      config['headers']?.['common'],
      config['headers']?.[method]
    )
    // 删除header无用项
    for (const key in headers) {
      if (['get', 'post', 'put', 'delete', 'common'].includes(key)) {
        const val = headers[key]
        const mergeVal = mergeConfig(
          defaultConfig['headers'][key],
          config['headers']?.[key]
        )

        if (JSON.stringify(val) === JSON.stringify(mergeVal)) {
          delete headers[key]
        }
      }
    }

    // url
    const baseURL = this._getConfigValue(config, 'baseURL', '')

    config.url = urlLib.resolve(baseURL, url)
    config.headers = headers

    conosle.log('config', config)
  }
```

#### 四、发起网络请求

`request.js`

```js
import { isFunction, isObject } from '../utils/check'
import throwError from '../utils/throwError'
import requestMethods from './requestMethods'

// 发送ajax请求
export default function request(config) {
  let { url, method, headers, params, data } = config

  throwError(!url, '请输入url地址')
  throwError(
    !Object.values(requestMethods).includes(method?.toLowerCase()),
    'method不合法'
  )
  throwError(headers && !isObject(headers), 'headers必须是json格式')
  throwError(params && !isObject(params), 'params必须是json格式')

  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest()

    // 挂载params
    // 去除hash
    const searchIndex = url.indexOf('?')
    const hashIndex = url.indexOf('#')

    if (!!~hashIndex) {
      url = url.substring(0, hashIndex)
    }

    if (!~searchIndex) {
      url += '?'
    } else {
      if (url.slice(-1) !== '&') {
        url += '&'
      }
    }

    let searchParams = ''
    for (const key in params) {
      searchParams += `${key}=${params[key]}&`
    }
    if (searchParams.slice(-1) === '&') {
      searchParams = searchParams.slice(0, -1)
    }
    url += searchParams

    xhr.open(method, url)

    // 挂载headers
    for (const key in headers) {
      xhr.setRequestHeader(key, headers[key])
    }

    xhr.send(JSON.stringify(data))

    xhr.addEventListener('load', () => {
      console.log('xhr', xhr)

      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(xhr)
      } else {
        reject(xhr)
      }
    })
    xhr.addEventListener('error', err => {
      reject(err)
    })
    xhr.addEventListener('progress', e => {
      if (isFunction(config.onDownloadProgress)) {
        config.onDownloadProgress(e)
      }
    })
    xhr.upload.addEventListener('progress', e => {
      if (isFunction(config.onUploadProgress)) {
        config.onUploadProgress(e)
      }
    })
  })
}
```

#### 五、处理`transformRequest`和`transformResponese`

```js
_mergeConfig(config) {
    // ...

    // transformRequest
    this.transformRequest = this._getConfigValue(
      config,
      'transformRequest',
      () => {}
    )
    // transformResponse
    this.transformResponse = this._getConfigValue(
      config,
      'transformResponse',
      () => {}
    )

    if (!this.transformRequest(deepCopy(config)))
      console.warn('transformRequest必须返回config！否则不生效！')

  	// 处理 transformRequest
    config = this.transformRequest(deepCopy(config)) || config

 return new Promise((r, j) => {
    request(config).then(
      (res) => {
        const data = response(res, config)

        if (!_this.transformResponse(data.data))
          console.warn('transformResponse必须返回res！否则不生效！')

        // 处理transformResponse
        data.data = _this.transformResponse(data.data)

        r(data)
      },
      (err) => {
        j(err)
      }
    )
  })
}
```

#### 六、拦截器

首先先了解一下`axios`拦截器的用法

##### 请求拦截器

> 注意：请求拦截器的执行顺序是：后添加的先执行，先添加的后执行

```js
// 请求拦截器
axios.interceptors.request.use(
  config => {
    console.log('use1')
    return config
  },
  err => {
    console.log('err')
    return Promise.reject(err)
  }
)

axios.interceptors.request.use(
  config => {
    console.log('use2')
    return config
  },
  err => {
    console.log('err')
    return Promise.reject(err)
  }
)

axios.interceptors.request.use(
  config => {
    console.log('use3')
    return config
  },
  err => {
    console.log('err')
    return Promise.reject(err)
  }
)

// 执行顺序:
use3 => use2 => use1
```

##### 响应拦截器

> 注意：响应拦截器的执行顺序是：先添加的先执行，后添加的后执行

```js
axios.interceptors.response.use(
  data => {
    console.log('use11')
  },
  err => {
    console.log(err)
    return Promise.reject(err)
  }
)

axios.interceptors.response.use(
  data => {
    console.log('use22')
  },
  err => {
    console.log(err)
    return Promise.reject(err)
  }
)

axios.interceptors.response.use(
  data => {
    console.log('use22')
  },
  err => {
    console.log(err)
    return Promise.reject(err)
  }
)

// 执行顺序：
use11 => use22 => use33
```

因为拦截器的调用是**链式调用**的，因此我们可以想到`Promise`的`then`链式调用，下面是使用`Promise`链式调用的案例：

```js
// 合并参数 调用发起请求
_mergeConfig(config) {
 // ...

  // transformRequest
  this.transformRequest = this._getConfigValue(
    config,
    'transformRequest',
    () => {}
  )
  // transformResponse
  this.transformResponse = this._getConfigValue(
    config,
    'transformResponse',
    () => {}
  )

  if (!this.transformRequest(deepCopy(config)))
    console.warn('transformRequest必须返回config！否则不生效！')

  config = this.transformRequest(deepCopy(config)) || config

  // 请求拦截器调用链 [默认的请求发送]
  const requestList = [this.dispatchRequest()]
  this.interceptors.request.forEach((item) => {
    requestList.unshift(item)
  })
  this.interceptors.response.forEach((item) => {
    requestList.push(item)
  })

  let requestPromise = Promise.resolve(config)

  while (requestList.length) {
    const { resolve, reject } = requestList.shift()

    requestPromise = requestPromise.then(resolve, reject)
  }

  return requestPromise
}

// 发起请求
dispatchRequest() {
  let _this = this

  return {
    resolve(config) {
      return new Promise((r, j) => {
        request(config).then(
          (res) => {
            const data = response(res, config)

            if (!_this.transformResponse(data.data))
              console.warn('transformResponse必须返回res！否则不生效！')

            data.data = _this.transformResponse(data.data)

            r(data)
          },
          (err) => {
            j(err)
          }
        )
      })
    },
    reject(err) {
      return Promise.reject(err)
    }
  }
}
```

### 仓库地址

https://github.com/xinxin1228/js-combat-practice/tree/main/%E6%89%8B%E5%86%99axios/webpack-axios

## 手写 vite-axios 版本

### 说明

除了将`webpack`替换为`vite`，`axios`的核心代码和步骤与`手写webpack-axios版本`的代码一致，这里不在赘述。

另：如果对`vite`搭建工程化项目不太熟悉的话，可以去参考我之前写的[Vite 搭建工程化项目](/docs/前端笔记/Vite搭建工程化项目.md)这篇文章

### 仓库地址

https://github.com/xinxin1228/js-combat-practice/tree/main/%E6%89%8B%E5%86%99axios/vite-axios
