> 前言：最新工作中要用到阿里 OSS 进行文件的云存储，因此这里记录一下所踩的坑和如何使用。本案例中，前端所使用的 SDK 是 `Browser.js`，后端使用的 SDK 是 `Node.js`。

这里先简述一下 OSS 在业务中的地址和交互流程：我们使用阿里云 OSS 来存储用户所上传的文件，当用户上传文件时，需要先记录用户的上传记录，然后使用 OSS 进行文件上传，之后根据上传之后的返回数据，比如 Version_ID（用来记录同名文件的不同版本 id）等，再进行本地业务的入库记录。

## OSS 是什么

简单来说，阿里云 OSS（以下简称为 OSS）是由阿里云提供的大规模、安全、低成本、高可靠的云存储服务。OSS 主要用于存储和管理各种类型的非结构化数据，如图片、音频、视频、文档等，同时也支持日志和其他类型的二进制或文本数据。

OSS 的官方开发文档地址为：https://help.aliyun.com/zh/OSS/developer-reference/installation?spm=a2c4g.11186623.0.i75

## 上传

前端使用 OSS 上传的方式主要由：**API上传**和 **SDK上传**。

### API 上传

顾名思义，就是直接发起 `REST API` 请求携带文件传输至 OSS，使用 `REST API` 发起请求适用于对程序自定义要求较高的场景。但相对 SDK 的使用来说，较为繁琐。
比如一个分片上传文件，使用 SDK 的某个方法可以一步到位，而使用 API 的话，需要先将发送一个初始化切片的请求，然后拿着该请求返回的全局唯一的 `Upload ID`，然后前端进行文件的切片并携带该 `Upload ID` 再去发送切片传输请求，之后等所有的切片上传成功之后，再携带该 `Upload ID` 发送一个合并切片的请求。

为了让 OSS 识别上传者身份，需要添加签名鉴权，比如在 Header 或 URL 中包含签名，具体的添加方法以及签名的获取方式这里不再赘述，有兴趣可以移步至[使用 REST API 发起请求](https://help.aliyun.com/zh/OSS/developer-reference/use-the-restful-API-to-initiate-requests/?spm=a2c4g.11186623.0.0.61f81b45W4Fzq4)。

这里我们着重介绍 SDK 的方式进行上传。

### SDK上传

这里先总结一下 SDK 使用的流程：用户上传文件时，查看本地有无记录的 OSS 用于鉴权的 **临时访问凭证** 信息，如果本地存在 **临时访问凭证** 信息，那就直接上传，如果本地不存在 **临时访问凭证** 或已过期，那么需要先向自己的服务端（这里指的是 node）的 SDK 获取临时访问凭证，之后用 **临时访问凭证** 调用前端的 SDK 来上传等一系列操作。

> 疑问：为什么不直接在前端的 SDK 上使用从控制台上获取到的密钥？
>
> 由于Browser.js SDK通常在浏览器环境下使用，为避免暴露阿里云账号访问密钥（AccessKey ID和AccessKey Secret），强烈建议您使用临时访问凭证的方式执行OSS相关操作。
>
> 临时访问凭证包括临时访问密钥（AccessKey ID和AccessKey Secret）和安全令牌（SecurityToken）。
>
> 您可以通过调用STS服务的AssumeRole接口或者使用各语言STS SDK来获取临时访问凭证。

首先就是准备工作[准备工作](https://help.aliyun.com/zh/OSS/developer-reference/installation?spm=a2c4g.11186623.0.i29)，需要先去阿里云控制台去设置 RAM 用户，获取 `accessKeyId`、`accessKeySecret`、`roleArn`。

#### 1.node使用STS进行临时授权

需要先[创建RAM用户](https://help.aliyun.com/zh/OSS/developer-reference/authorized-access-3?spm=a2c4g.11186623.0.i11#section-zkq-3rq-dhb)，并且设置权限。

```js
const { STS } = require('ali-OSS')
const express = require('express')
const app = express()

app.get('/STS', (req, res) => {
  const client = new STS({
    accessKeyId: 'yourAccessKeyId', // 准备工作中获取到的accessKeyId
    accessKeySecret: 'yourAccessKeySecret',
    bucket: 'yourBucket', // 存储桶名称 比如你的存储桶是test 那这里就是test
  });

  const result = await client.assumeRole(
    'yourRoleArn', // 准备工作中创建的RAM角色
    null,
    60 * 60 // token过期时间 单位秒 15min/1h 最少15min，最大1h
  );
  res.set('Access-Control-Allow-Origin', '*');
  res.set('Access-Control-Allow-METHOD', 'GET');
  const { credentials = {} } = result;

  //返回参数
  const params = {
    AccessKeyId: credentials.AccessKeyId,
    AccessKeySecret: credentials.AccessKeySecret,
    SecurityToken: credentials.SecurityToken,
    Expiration: credentials.Expiration,
    bucket: 'yourBucket', // 你的存储桶名称
    region: 'yourRegion', // 你当前OSS服务器所在的区别 比如 OSS-cn-beijing
    host: `http://${'yourBucket'}.${'yourRegion'}.aliyuncs.com`, // 根据根据存储桶和的确拼接出host
  }
  
  // 获取STS信息
  res.send(params)
})
```

针对上面的代码，个人来说不太喜欢直接把上面的密钥暴漏出来，而是更倾向于把这些密钥抽取成环境变量，然后在项目中使用。

不懂环境变量抽取的可以参考我之前写的这篇文章：[Node中如何使用环境变量](/docs/开发技巧/Node中如何使用环境变量.md)。

首先先在项目的根目录上创建 `.env` 文件。

```shell
OSS_ACCESS_KEY_ID=xxxxxxxx
OSS_ACCESS_KEY_SECRET=xxxxxxxx
OSS_BUCKET=test
OSS_REGION="OSS-cn-beijing"
OSS_ROLEARN=xxx
```

然后替换上面的变量。

```js
const client = new STS({
  accessKeyId: process.env.OSS_ACCESS_KEY_ID, // 从环境变量中读取
  accessKeySecret: process.env.OSS_ACCESS_KEY_SECRET,
  bucket: process.env.OSS_BUCKET, 
});

const result = await client.assumeRole(
  process.env.OSS_ROLEARN, // 准备工作中创建的RAM角色
  null,
  60 * 60 // token过期时间 单位秒 15min/1h 最少15min，最大1h
);
```



#### 2.前端使用获取到的临时凭证进行上传

##### 获取STS凭证
```js
const getOssSTS = async () => {
  const res = await axios.get('/api/oss/sts')
  const data = res.data

  if (!data.success) return message.error(message.msg)

  setSTSData(data.data) // 将获取到的STS凭证存储起来，可以存储在全局的store中，也可以只存储在当前页面上。
  return data.data // 将STS凭证返回出去
}
```

#### 文件上传
文件上传分为简单上传和分片上传，我们可以在前端手动判断不同的文件大小来调用不同的方式上传。比如文件大于1MB我们调用分片上传，文件小于等于1MB时我们调用简单上传。

> 需要注意的是，简单上传不支持进度显示，因此，为了有更好的用户体验，我这里都采用了分片上传方式。

```js
/**
* 单文件上传
* @param {File} file 要上传的文件
*/
const uploadOssByAliOss = (file) => {
  try {
    // 获取STS凭证
    let signInfo = STSData

    // 过期重新获取
    if (!signInfo.Expiration || dayjs(signInfo.Expiration).valueOf() < Date.now()) {
      signInfo = await getOssSTS()
    }

    // 实例化OSS的配置项信息
    const config = {
      accessKeyId: signInfo.AccessKeyId,
      accessKeySecret: signInfo.AccessKeySecret,
      bucket: signInfo.bucket,
      stsToken: signInfo.SecurityToken,
      region: signInfo.region,
      secure: true, // 是否开启https
      // 过去自动刷新凭证
      refreshSTSToken: async () => {
        // 获取STS
        const info = await getOssSTS()
        return {
            accessKeyId: info.AccessKeyId,
            accessKeySecret: info.AccessKeySecret,
            stsToken: info.SecurityToken,
        }
      },
      // 刷新临时访问凭证的时间间隔，单位为毫秒。
      refreshSTSTokenInterval: 300000,
    }

    // 初始化OSS上传实例
    const client = new OSS(config) 
    
    // 分片上传OSS的配置信息
    const optionsInfo = {
      progress: async (p, cpt) => {
        // 在这里显示上传的进度，也可以记录上传的位置，方便断点续传或者取消上传
        console.log(`当前文件的上传进度：${p * 100 |0}，当前文件上传的位置：${cpt}`)
      },
      parallel: 4, // 设置并发上传的分片数量。
      partSize: 1024 * 1024, // 1MB
      timeout: 120000, //设置超时时间
    }
   
    /**
    * 开始分片上传
    * @param {String} url 上传的地址：[host+filename] 还记得node中返回的host地址吗？ 假设host地址是：http://test.OSS-cn-beijing.aliyuncs.com ,文件名称是 1.jpg，那url就是 http://test.OSS-cn-beijing.aliyuncs.com/1.jpeg
    * @param {File} file 上传的文件
    * @param {Object} optionsInfo 上传的配置项
    */
    const res = await client.multipartUpload(url, file, optionsInfo)
  } catch(err) {
    console.log('上传失败', err)
  }
}

```

### 多文件上传

多文件上传其实就是多次调用单文件上传，比如有一个要上传的文件列表是`fileList`。
```js
Promise.allSettled(fileList.map(file => uploadOssByAliOss(file))).finally(() => {
  console.log('无论成功失败 所有文件上传完成')
})

```

### 取消上传
> 踩坑点：切记这里要重新生成STS凭证信息，不能复用之前的，否则会产生问题。

```js
const cancelOssUpload = async fileObj => {
  // 重新生成OSS实例
  const signInfo = await getOssSTS()

  const config = {
    accessKeyId: signInfo.AccessKeyId,
    accessKeySecret: signInfo.AccessKeySecret,
    bucket: signInfo.bucket,
    dir: signInfo.dir,
    stsToken: signInfo.SecurityToken,
    region: signInfo.region,
    // secure: true, // 是否开启https
  }

  const client = new OSS(config)
  
  /**
  * 取消上传
  * 参数分别是：
  * name：之前在上传进度progress中保存的cpt.name
  * uploadId: 之前在上传进度progress中保存的cpt.uploadId
  */
  await client.abortMultipartUpload(cpt?.name, cpt?.uploadId)
}

```

#### 全部取消上传
全部取消上传其实就是先将已经上传完成的进行一下过滤，然后将正在上传的文件列表分别调用取消文件上传，比如有一个要上传的文件列表是`fileList`。

``` js
Promise.allSettled(fileList.map(file => uploadCancelFileList(file))).finally(() => {
  console.log('全部取消完成！')
})

````


## 获取文件地址

根据文件的相对路径来获取文件的完整路径。这部分我们在Node端实现。图片处理相关的参数[见这里](https://help.aliyun.com/zh/OSS/developer-reference/img-5?spm=a2c4g.11186623.0.0.65e9dc77bPhOLS)。

```js
const OSS = require('ali-OSS')
const express = require('express')
const app = express()

app.get('/url', (req, res) => {
  // 要获取完整路径的相对url
  const fileUrl = ['README.md', '/images/2.jpg']

  // 确保已设置好对应的环境变量
  const client = new OSS({
    region: progress.env.OSS_REGION, // Bucket所在地域
    accessKeyId: process.env.OSS_ACCESS_KEY_ID, // 密钥
    accessKeySecret: process.env.OSS_ACCESS_KEY_SECRET,
    bucket: progress.env.OSS_BUCKET, // 存储桶名称
    secure: true // 开启https
  })

  return fileUrl.map(url => {
    const original = client.signatureUrl(n.url, {
      expires: 300, // 链接过期时长
      // process: "image/resize,m_fixed,w_100,h_100", 图片处理
    })

    return original 
  })
})

```

预览指定version_id的文件：[相关文档地址](https://help.aliyun.com/zh/OSS/developer-reference/authorized-access-3?spm=a2c4g.11186623.0.i63)。

> 踩坑点：这里需要注意的是，如果带上图片处理参数，那么版本就不会生效，永远返回的是最新版本处理后的图像，这个咨询阿里支持，暂没有解决方案。
>
> 问题记于：2024-02-26

```js
const OSS = require('ali-OSS')
const express = require('express')
const app = express()

app.get('/url', (req, res) => {
  // 要获取完整路径的相对url和version信息
  const fileUrl = [
    {
      url: 'README.md',
      oss_version: 'xxxxxx'
    },
    {
      url: '/images/2.jpg',
      oss_version: 'xxxxxxx'
    } 
  ]

  // 确保已设置好对应的环境变量
  const client = new OSS({
    region: progress.env.OSS_REGION, // Bucket所在地域
    accessKeyId: process.env.OSS_ACCESS_KEY_ID, // 密钥
    accessKeySecret: process.env.OSS_ACCESS_KEY_SECRET,
    bucket: progress.env.OSS_BUCKET , // 存储桶名称
    secure: true // 开启https
  })

  return fileUrl.map(n => {
    const original = client.signatureUrl(n.url, {
      expires: 300, // 链接过期时长
      subResource: {
        versionId: n.oss_version
      }
    })

    return original 
  })
})
```

### 补充：如何获取到图像文件的宽高

需要先利用 `signatureUrl` 方法来获取到图像文件的地址，然后去请求该地址来得到图像的信息。

```js
const OSS = require('ali-OSS')
const express = require('express')
const axios = require('axios')
const app = express()

app.get('/url', (req, res) => {
  // 要获取完整路径的相对url
  const fileUrl = ['README.md', '/images/2.jpg']

  // 确保已设置好对应的环境变量
  const client = new OSS({
    region: progress.env.OSS_REGION, // Bucket所在地域
    accessKeyId: process.env.OSS_ACCESS_KEY_ID, // 密钥
    accessKeySecret: process.env.OSS_ACCESS_KEY_SECRET,
    bucket: progress.env.OSS_BUCKET , // 存储桶名称
    secure: true // 开启https
  })

  const list = await Promise.allSettled(fileUrl.map(n => {
    const original = client.signatureUrl(n.url, {
      expires: 300, // 链接过期时长
    })

    const { data } = await axios.get(original)
    
    n.width = data.ImageWidth.value
    n.height = data.ImageHeight.value

    return n 
  }))

  res.send(list)
})
```