

### ajax与axios之间数据交互

```js
// 由于后端没有jQuery，因此后端只能使用axios，但是前端既可以使用axios，也可以使用ajax

// 当前端发送get请求 

// 情况一：通过axios发送 
// 前端：   数据放在params
$(".get").click(() => {
  let name = $(".name").val()
  let pass = $(".pass").val()
  axios.get("/login", {
    	params: {
      name, pass
    }
  })
    .then(({ data }) => {
    console.log(data)
  })
    .catch(err => {
    console.log(err)
  })
})

// 后端：
router.get("/login",async ctx=>{
  // 接受前端发送的数据 （转为字符串的形式） name=123&pass=123
  // let data = ctx.querystring
  // 接受前端数据，转化为对象的形式  [Object: null prototype] { name: '123', pass: '123' }
  let data = ctx.query
  console.log(data)
  let body = {
    stste:200,
    mes:"登录成功，正在跳转。。",
    data
  }
  // 后端返回给前端的数据
  ctx.body = body
})

// 情况二：通过ajxa发送 
// 前端   数据放在data
$(".get").click(() => {
      let name = $(".name").val()
      let pass = $(".pass").val()
      $.ajax({
        type: "get",
        url: "/login",
        data: {
          name, pass
        },
        success: res => {
          console.log(res)
        }
    })
  	
  // 或者下面方法
   $.get("/login", {
        data: {
          name, pass
        }
      }).then(res => {
        console.log(res)
      })
  
 })
// 后端于上一样



// 当前端发送post请求

//情况一：使用axios发送  
// 前端   数据放在data中
$(".post").click(() => {
      let name = $(".name").val()
      let pass = $(".pass").val()
      axios.post("/login", {
        data: {
          name, pass
        }
      })
        .then(({ data }) => {
          console.log(data)
        })
        .catch(err => {
          console.log("数据获取失败", err)
        })
    })

//后端  koa2
router.post("/login",async ctx=>{
  // 获取前端发送的数据
  let {name,age} = ctx.request.body.data
  
  ctx.body = {
    state:200,
    mes:"恭喜你，已经成功登录",
    data:{name,age}
  }
})

// 情况二：通过ajax发送请求
// 前端  数据放在data中
 $(".post").click(() => {
      let name = $(".name").val()
      let pass = $(".pass").val()
      $.ajax({
        type: "post",
        url: "/login",
        data: {
          name, pass
        },
        success: res => {
          console.log(res)
        }
      })
   
   	// 或者下面代码
   $.post("/login", {
        data: {
          name, pass
        }
      }).then(res => {
        console.log(res)
      })
   
   
   
})

// 后端： koa2
router.post("/login",async ctx=>{
  // 获取前端发送的数据
  let data = ctx.request.body
  console.log(data)
  
  ctx.body = {
    state:200,
    mes:"恭喜你，已经成功登录",
    data
  }
})

```




### Koa使用

```js
// 安装koa  npm i koa
// 管理路由  npm i koa-router
// 使用静态资源  npm i koa-static 比如css js
// 使用html模板文件  npm i koa-views 处理html类文件
// 解决post请求    npm i koa-body 获取前端传来的数据
// 解决跨域请求   npm i @koa/cors
// 给前端返回数据  ctx.body = {}
// 接受前端的数据  ctx.request.body
// 渲染views里的页面  app.use((views("./路径")))
// 加载静态资源   app.use(Static("./路径"))


const koa = require("koa")
const router = require("koa-router")()
const app = new koa()
const Static = require("koa-static")
const viesw = require("koa-views")
const body = require("koa-body")
const {resolve} = require("path")
const userInfo = require("./user") // 获取数据库数据

app.use(async (ctx,next)=>{ 
  ctx.body = "这里是在前端输出的内容"
})

app
  .use(body())  // 加载前端发送的数据
	.use(views(resolve(__dirname,"./views"))) // 加载html文件
	.use(Static(resolve(__dirname,"./static"))) // 加载静态资源 css js 利用path中的拼接路径 （当前文件路径+目标路径） 得到绝对路径

// 当前端访问跟路由触发
router.get("/",async ctx=>{
  ctx.body = "这里通过get请求路由"
  ctx.redirect("/index")  //路由重定向 
  ctx.status = 304   //修改状态码 重定向状态码为304
  ctx.type = "text/plain"  //定义返回类型 （返回纯文本形式）
  ctx.render("index") // 渲染views中的index页面
})

// 通过post访问跟路由时触发
router.post("/",async ctx=>{
  // 返回给前端的数据
  ctx.body = "这里通过post请求路由"
  // 获取前端发送的数据
  let data = ctx.request.body
})

// 获取前端通过get发送的数据，并返回结果
router.get("/login",async ctx=>{
  // 接受前端发送的数据 （转为字符串的形式） name=123&pass=123
  // let data = ctx.querystring
  // 接受前端数据，转化为对象的形式  [Object: null prototype] { name: '123', pass: '123' }
  let data = ctx.query
  console.log(data)
  let body = {
    stste:200,
    mes:"登录成功，正在跳转。。",
    data
  }
  // 后端返回给前端的数据
  ctx.body = body
})


//获取前端通过post发送的数据，并返回结果(登录 使用数据库)
router.post("/login",async ctx=>{
  let {name,pass} = ctx.request.body
  // 数据库查询
  let res = await userInfo.find({userNmae:name})
  if(!res.length){
    ctx.body = {
      message:"该账号还未注册",
      state:0
    }
    return
  }
  if(res[0].pass === pass){
    ctx.body = {
      message:"登录成功",
      state:1
    }
  }else{
    ctx.body = {
      message:"密码错误",
      state:0
    }
  }
})

// koa请求动态路由
//请求方式   http://域名/product/123
router.get('/product/:aid',async (ctx)=>{
   console.log(ctx.params); //{ aid: '123' }  //获取动态路由的数据
   ctx.body='这是商品页面';
})

//固定写法
app
  .use(router.routes())
  .use(router.allowedMethods())
	.listen(3000,()=>{
  console.log("3000端口已监听")
})
```





#### 

### koa2+typescript使用

```ts
// 安装koa  pnpm add koa
// 管理路由  pnpm add @koa/router @types/koa__router
// 使用静态资源  npm i koa-static 比如css js
// 使用html模板文件  npm i koa-views 处理html类文件
// 解决post请求    npm i koa-body 获取前端传来的数据
// 解决跨域请求   npm i @koa/cors
// 给前端返回数据  ctx.body = {}
// 接受前端的数据  ctx.request.body
// 渲染views里的页面  app.use((views("./路径")))
// 加载静态资源   app.use(Static("./路径"))


// 加载静态资源
import Static from "koa-static";
app.use(Static(__dirname + `/public`)) // 切记 不能携程 app.use(Static('public')) 无法生效
// 上传文件配置，和接受delete请求时，获取参数
app.use(
    koaBody({
      multipart: true, // 允许上传文件
      strict: false, // 获取delete请求的参数
      formidable: {
        maxFieldsSize: 200 * 1024 * 1024 // 上传文件的最大限制
      }
    })
  )
```

### koa2+ts上传文件

```ts
// 文件上传
import type { Context } from "koa";
import fs from "fs";
import path from "path";
import dayjs from "dayjs";

import { throwError } from "./feedbackClient";

/**
 *
 * @param ctx 执行上下文
 * @param uploadPath 上传的路径 相对于 src/public/
 * @returns 上传后的文件名
 */
export const upload = async (ctx: Context, uploadPath: string) => {
  const file = ctx.request.files?.file;

  if (!file) throw throwError(400, "文件上传不能为空");

  // @ts-ignore
  const reader = fs.createReadStream(file.filepath);
  const name = Math.random().toString(36).slice(2, 8);
  // @ts-ignore
  const ext = path.extname(file.originalFilename);
  const imgSrc = `/${dayjs().format("YYMMDD")}_${name}${ext}`;
  // @ts-ignore
  const filePath =
    // @ts-ignore
    path.join(__dirname, `../public/${uploadPath}`) + imgSrc;
  // 创建可写流
  const upStream = fs.createWriteStream(filePath);
  // 可读流通过管道写入可写流
  reader.pipe(upStream);

  return `${uploadPath}${imgSrc}`;
};


// 调用
await upload(ctx, 'file')
```

### Koa2+ts下载文件

编写一个接口，要求后端将本地文件以流式的方式传给前端，然后前端调用浏览器的下载进行下载，假设文件全部存放在后端的 `/static` 目录下。

```js
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

### koa2+ts打zip下载

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



```js
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





























