## Js读取文件的宽高信息 支持url和本地文件

> 本案例的完整代码地址：https://github.com/xinxin1228/docs_develop_tips/tree/main/01_js%E8%AF%BB%E5%8F%96%E5%9B%BE%E5%83%8F%E6%96%87%E4%BB%B6%E5%AE%BD%E9%AB%98

在`Web`开发中，有时我们需要获取图片的宽度和高度以便进行一些操作，比如动态设置图片的大小或者进行布局调整。本文将介绍如何使用`JavaScript`来获取图片的宽度和高度。

### 项目搭建

接下来我们简单搭建一下项目。

> 我们使用`webpak`进行项目的构建，有不懂的可以查看本站的有关`Webpack`介绍的文章。

项目的目录如下：

```shell
├── node_modules
├── package.json
├── pnpm-lock.yaml
├── public
│   └── index.html
├── src
│   ├── index.js
│   └── style.css
└── webpack
    ├── webpack.comm.config.js
    ├── webpack.dev.config.js
    └── webpack.prod.config.js
```

`index.html`的文件内容如下：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Js读取图像文件的宽高</title>
  </head>
  <body>
    <div class="upload">
      <p>点击或拖拽进行上传</p>
      <p>——————————</p>
      <p>查看图片文件的宽高信息</p>
      <input type="file" id="input" multiple />
    </div>
    <input
      type="text"
      id="url"
      value="https://webpack.docschina.org/site-logo.1fcab817090e78435061.svg"
    />
    <button id="btn">查看</button>
  </body>
</html>
```

### 使用Image对象获取宽高

要获取图片的宽度和高度，我们可以使用JavaScript中的`Image`对象。这个对象表示一个HTML图像元素（`<img>`），然后可以在它的实例对象上读取到宽高。

```js
const img = new Image()
img.src = 'path/a.jpg'

img.addEventListener('load', e => {
  const width = e.target.width
  const height = e.target.height
  
  // 或
  const width = img.width
  const height = img.height
})
```

### 获取本地图像文件的宽高

我们先对`html`中的`.upload`元素进行美化和添加功能，使其可以实现拖拽上传和点击上传的功能。

css代码如下：

```css
* {
  margin: 0;
  padding: 0;
}

.upload {
  width: 400px;
  height: 260px;
  margin: 20px;
  border: 2px dashed #d9d9d9;
  border-radius: 10px;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  transition: border-color 0.3s;
  user-select: none;
  &.focus,
  &:hover {
    border-color: #1890ff;
  }
  p {
    font-size: 14px;
    color: #999;
  }
  #input {
    display: none;
  }
}
#url {
  display: inline-block;
  width: 300px;
}

```

js代码如下：

```js
const $ = selector => document.querySelector(selector)

const uploadEl = $('.upload')
const inputEl = $('#input')
const urlEl = $('#url')
const btnEl = $('#btn')

// 点击上传
uploadEl.addEventListener('click', () => {
  inputEl.click()
})

// 拖拽上传
uploadEl.addEventListener('dragover', e => {
  e.preventDefault()
  uploadEl.classList.add('focus')
})
uploadEl.addEventListener('drop', e => {
  e.preventDefault()
  // 拖拽选择的文件
  console.log(e.dataTransfer.files)
})
uploadEl.addEventListener('dragleave', e => {
  uploadEl.classList.remove('focus')
})

// 点击上传所选择的文件列表
inputEl.addEventListener('change', e => {
  console.log(e.target.files)
})

```

当前界面实现的效果如图所示：

![点击或拖拽上传演示](../image/开发技巧/01.gif)

我们上面获取图片信息的方法，需要先拿到 `img.src`，也就是图片的路径。那么本地的文件如何获取路径呢？我们可以使用 `URL.createObjectURL`，这个方法用于创建一个指向浏览器内部表示指定 `Blob` 或 `File` 对象的 `URL`。不过这个 `URL` 是临时性的，浏览器关闭之后或手动释放之后，该 `URL` 就会被释放。我们可以手动释放来避免内存泄漏。

```js
// 根据拿到读取的文件创建一个 Blob 链接
const url = URL.createObjectURL(file)

// 释放资源
URL.revokeObjectURL(url)
```

接下里我们封装一个方法 `getImageWidthHeightByFile`，用于根据本地读取的 `File` 对象来获取图片的宽高。

```js
/**
 * 根据本地读取的file对象，获取宽高
 * @param {File} file 本地读取的file对象
 * @returns { width, height, errTip? }
 */
function getImageWidthHeightByFile(file) {
  return new Promise(resolve => {
    const img = new Image()
    const url = URL.createObjectURL(file)
    img.src = url
    img.addEventListener('error', () => {
      resolve({ errTip: '该文件不是图片类型！' })
      URL.revokeObjectURL(url)
    })
    // 兼容处理 如果上传的不是图片文件，给出对应的错误提示
    img.addEventListener('error', () => {
      resolve({ errTip: '该文件不是图片类型！' })
    })
  })
}
```

### 获取url图像文件的宽高

有了上面获取本地图像文件宽高的基础，我们可以轻松写出获取 `url` 图像文件的宽高。代码逻辑一致，只是少了本地文件转 `url` 的步骤。

```js
/**
 * 根据url获取宽高
 * @param {string} url 图像url地址
 * @returns { width, height, errTip? }
 */
function getImageWidthHeightByUrl(url) {
  return new Promise(resolve => {
    const img = new Image()
    img.src = url
    img.addEventListener('load', e => {
      resolve({ width: e.target.width, height: e.target.height })
    })
    img.addEventListener('error', () => {
      resolve({ errTip: '该文件不是图片类型！' })
    })
  })
}
```

然后将上述方法嵌入到对应的`DOM`上即可。

```js
// 拖拽上传文件
uploadEl.addEventListener('drop', e => {
  e.preventDefault()
  uploadEl.classList.remove('focus')

  Promise.allSettled(
    Array.from(e.dataTransfer.files).map(async file => {
      const info = await getImageWidthHeightByFile(file)

      return {
        name: file.name,
        ...info,
      }
    })
  ).then(res => {
    console.log(res.map(n => n.value))
  })
})

// 点击上传文件
inputEl.addEventListener('change', e => {
  Array.from(e.target.files).forEach(async file => {
    const { width, height } = await getImageWidthHeightByFile(file)
    console.log(file.name, `${width}X${height}`)
  })
})
```

### 方法的优化

由于两个方法中的逻辑和代码基本一致，因此我们可以将两个方法进行合并：

```js
/**
 * 获取图像的宽高
 * @param {string|File} param url地址或File对象
 * @returns { width, height, errTip? }
 */
function getImageWidthHeight(param) {
  let url = param

  if (param instanceof File) {
    url = URL.createObjectURL(param)
  }

  return new Promise(resolve => {
    const img = new Image()
    img.src = url
    img.addEventListener('load', e => {
      resolve({ width: e.target.width, height: e.target.height })
      URL.revokeObjectURL(url)
    })
    img.addEventListener('error', () => {
      resolve({ errTip: '该文件不是图片类型！' })
    })
  })
}
```

