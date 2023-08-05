

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

































