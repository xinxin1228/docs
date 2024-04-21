文件的下载或将多文件打成一个 Zip 压缩包进行下载是一个常见的需求，今天我们实现一下该功能。

本案例后端采用的是 `Node+Koa+Typescript`。

我们需要编写两个接口，一个是用来下载单文件的下载接口，另外一个是可以下载多文件或文件夹的 Zip 下载接口。

前端使用 `axios` 调用这些接口来使用浏览器的下载进行下载。

## 文件下载接口

```ts
import Koa from 'koa';
import Router from 'koa-router';
import bodyParser from 'koa-bodyparser';
import fs from 'node:fs';
import path from 'node:path';

const app = new Koa();
const router = new Router();

app.use(bodyParser());

router.get('/download/:name', async (ctx) => {
   try {
     // 获取要下载的文件名称
     const { name } = ctx.params
     // 获取文件的真实路径
     const filePath = path.resolve(__dirname, `../static/${name}`)
     // 查看文件是否存在
     const exist = fs.existsSync(filePath);
     if (!exist) throw '文件不存在！'
     
     // 添加响应头告诉浏览器，接收到的响应内容应当作为附件下载，而不是直接在浏览器窗口内打开或嵌入。
     ctx.set({
        "Content-Disposition": `attachment;`
      });
     
     ctx.status = 200
     // 流式读取文件并返回给前端
     ctx.body = fs.createReadStream(filePath) 
   } catch(error) {
     ctx.status = 500
     ctx.body = { error }
   }
});

app.use(router.routes());
app.use(router.allowedMethods());

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

## 压缩为Zip文件并下载

> 需要安装依赖 `adm-zip` 。

编写一个接口，要求后端将本地文件按照不同的目录结打成一个 ZIP 压缩包，然后返回给前端，如果某个文件不存在，忽略该文件即可，不会中止打包。假设后端的文件存放在 `/static` 目录下。这里的所有文件都是平级存放在 `/static` 目录下，不存在嵌套目录结构。

这里我们分目录进行打包，假设目录结构如下：

```shell
 down
├── 素材文件夹
│   ├── 1.jpg
│   ├── 2.jpg
│   ├── 2.jpg
│   │   └── 动漫
│   │         ├── 01.jpg
│   │         ├── 02.jpg
│   ├── web.jpg
```

```ts
import Koa from 'koa';
import Router from 'koa-router';
import bodyParser from 'koa-bodyparser';
import fs from 'node:fs';
import path from 'node:path'
import AdmZip from 'adm-zip';
import { promisify } from "util";

const app = new Koa();
const router = new Router();

// 使用 promisify 将 fs.unlink 转换为 promise 形式
const unlink = promisify(fs.unlink);

app.use(bodyParser());

router.post('/zip', async (ctx) => {
  // 要打zip压缩包的文件列表 格式为：
  // list: [{ folder: '/a', name: '1.jpg', url: 'xas.jpg' }]
  // folder: 打包后的文件夹路径
  // name: 打包后的文件名称
  // url: 文件的路径
  const { list } = ctx.request.body;
  const zip = new AdmZip();
  
  // 获取每个文件真实的地址
  const getFilePath = name => path.resolve(__dirname, `../static/${name}`)

  // 循环处理每个文件
  for (const file of list) {
    const { url, name, folder } = file;
    const filePath = getFilePath(name)
    
    // 如果该文件不存在，跳过该文件
    if (!fs.existsSync(filePath)) return

    // 将文件添加到 zip，并指定自定义的路径 以及打包后的名称
    zip.addLocalFile(filePath, `/download${folder}`, name)
  }

  // 保存 zip 文件到临时文件夹
  const tempZipPath = getFilePath('./temp.zip');
  zip.writeZip(tempZipPath);

  // 设置响应头，告诉浏览器返回的是 zip 文件
  ctx.set("Content-Type", "application/zip");
  ctx.set("Content-Disposition", "attachment");

  // 读取并发送 zip 文件
  ctx.body = fs.createReadStream(tempZipPath);
	// 使用 fs.unlink会在文件被关闭后再尝试删除。fs.createReadStream 返回的流会在读取完毕后自动关闭，这样就确保了文件被关闭后再进行删除操作，从而避免了出现文件仍被占用的情况
   await unlink(tempZipPath);
});

app.use(router.routes());
app.use(router.allowedMethods());

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

## 前端下载

前端下载的过程可以简述为：先发起 `Ajax` 请求获取文件数据，然后将该文件数据转为 `Blob` 格式，之后创建一个 `a` 标签调用浏览器下载进行下载。

封装一个下载 `Blob` 格式的数据的方法。

```ts
/**
 * 下载文件
 * @param blob Blob数据
 * @param name 文件名称
 */
export const download = (blob: Blob, name?: string) => {
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  if (name) link.download = name
  link.href = url
  link.style.display = 'none'
  link.click()
  URL.revokeObjectURL(url)
  link.remove()
}
```

调用下载：

<!-- tabs:start -->

#### **下载文件**

```js
const { data } = await axios.get('/download/1.jpg', {
    responseType: 'blob'
  }
)

download(data, 'download.zip')
```

#### **下载Zip**

```js
const { data } = await axios.post('/zip', {
    list: [
      { name: '1.jpg', url: 'adsa.jpg', folder: '/a' },
      { name: '2.jpg', url: 'adsxa.jpg', folder: '/a/b' },
      { name: '3.jpg', url: 'adsaa.jpg', folder: '/a' },
      { name: '12.jpg', url: 'ads1a.jpg', folder: '/c' },
    ]
  },
  {
  	responseType: 'blob'
  }
)

download(data, 'download.zip')
```



<!-- tabs:end -->

