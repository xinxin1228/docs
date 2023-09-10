### node基础

### 服务器默认端口

```js
http协议默认端口  80
https协议默认端口  443
```

### path (路径)

```js
// 引用
const path = reqeire("path") 

__dirname  //当前文件夹的路径

__filename  //当前文件的路径

path.join()   //拼接路径

path.resolve()  //拼接路径，但是会解析/ 开头的路径

path.relative()  //得到两个路径的相对路径

path.parse()  //解析一个路径，得到一个包含路径信息的对象

console.log(path.parse("C:/Users/admin/node/01.js"))
{
  root:"C:/",    //根目录
  dir:"C:/Users/admin/node", // 文件夹目录
  base:"01.js",   // 文件名称
  ext:".js",  // 文件后缀
  name:"01"   // 文件名称
}


path.extname() //获取文件扩展名
```

### url （url地址）

```js
// 引入URL
const URL = require("url").URL
// 引入URL 解构方法
const {URL} = require("url") 

let x = new URL("地址")
console.log(x)
```

### querystring （解析字符串）

```js
// 引入模块
const querystring = require("querystring")

querystring.parse()  //解析成查询字符串

querystring.stringift({a:10,b:20},"$",":") 

a:10$b:20

//将前端的数据转化格式发给后端，第一个参数：对象，第二个参数：对象之间的连接符，第三个对象：键值对的替换符
```

### fs （文件系统）

```js
// 引入模块
const fs = require("fs")
const {resolve} = require("path")

fs.readFile()   //读取文件 

fs.readFileSync()  //读取文件  （同步方法）

fs.writerFile()  //写入文件

fs.readdir("./text",err=>{})   //读取目录

fs.stat("./text",(err,res)=>{   //读取文件/文件夹对应信息
  if(res) return
  console.log(res)
  console.log(res.isFile())    // 判断是不是文件
  console.log(res.isDirectory())  // 判断是不是目录
})     

// 读取目录+判断类型
fs.readdir("./text",(err,file)=>{
  if(err) return
  file.forEach(item=>{
    fs.stat(resolve("./text",item),(err,res)=>{
      if(err) return
      console.log(item+"是不是文件":res.isFile())
    })
  })
})

fs.mkdir("./text",err=>{})    // 创建目录

fs.rename("./dir/text","./dir/txt",err=>{})  //重命名

fs.rename(path.join(__dirname,"../../text.txt"),"./text",err=>{})  // 移动位置

fs.rm(path.join(__dirname,"../../text.txt"),{recursive:true},err=>{})   // 删除目录和文件  支持递归删除

const os = require("os")
//换行  
os.EOL  类似于 \r \n
// 复制api
fs.copyFile("被复制的文件","复制的文件",()=>{
  console.log("文件复制成功")
})

// 数据硬链接  数据会同步修改
fs.link("被复制的文件","复制的文件",()=>{
  console.log("数据硬链接创建成功")
})

// 数据追加
fs.appendFile("追加的文件","追加的内容",err=>{
  if(!err){
    console.log("数据追加成功")
  }
})

// 判断路径是否存在 （这里使用同步版本）
console.log(fs.eistsSync("要判断的文件"))


```

### Buffer操作

```js
Buffer.from(str)  //将字符串转化为buffer
Buffer.alloc(size) //创建一个指定大小的buffer
Buffer.alloUnsafe(size)  //创建一个指定大小的buffer，但里面可能包含数据
buf.toString()   //将缓冲区的数据转化为字符串

```

### 项目初始化

```js
// 项目初始化 
npm init

// 快速完成项目初始化
npm init -y

// 安装依赖
npm i 包名 -S  （通用）  （当前项目的 生产环境 -- 后续上线之后也要用到的，比如axios）
npm i 包名 -D  （当前项目的1 开发环境 -- 只在开发过程中使用，上线之后就不用的，比如less，sass等css预处理器）
npm i 包名 -g  （全局安装 -- 所有在当前电脑上创建的项目都能用这个包）
```

### 

### 后端允许他人跨域访问

```js
// 安装cors
npm i cors -S

// 引入cors
const cors = require("cors")
module.exports = cors({
  // 可以设置多个跨域
origin:['http://localhost:8080','http://localhost:8081'],
credentials:true // 设置跨域
})

// 跨域中间件
app.use(cors)

// 前端访问会出现跨域问题 后端不存在跨域问题 因此访问别人网址跨域时，可以先通过后端作为中转站，然后前端访问自己的后端
```

### 编码转换

```js
// 编码 将字符转化为 unicode编码
encodeURI(str)   // 处理一般的字符
encodeURIComponent(str)  // 处理特殊复杂字符

// 解码 将unicdeo编码转化为字符串
decodeURL(str)   
decodeURIComponent(str)


// en 编码     de 解码
```



### 文件写入

```js
//简单文件写入  （异步） 使用于小型文件
const fs = require("fs")
fs.writeFile("text.txt","这里是写入的内容",{flag:"w"},err=>{
  if(!err){
    console.log("文件写入成功")
  }else{
    console.log("文件写入失败")
  }
})
// flag取值：w 可写  r 可读  a 追加
// 封装写法为：
function wrFile(path,Content){
  return new Promise((resovled,rejected)=>{
    fs.writeFile(path,Content,{flag:"w"},err=>{
      if(!err){
        resovled("创建成功")
      }else{
        rejected("创建失败")
      }
    })
  })
}
// 1
async function wrListFile(){
  await wrFile("text.txt","1111")
  await wrFile("text.txt","2222")
  await wrFile("text.txt","3333")
  await wrFile("text.txt","4444")
  await wrFile("text.txt","5555")
}
wrListFile()
// 2
(async ()=>{
	await wrFile("text.txt","1111")
  await wrFile("text.txt","2222")
  await wrFile("text.txt","3333")
  await wrFile("text.txt","4444")
  await wrFile("text.txt","5555")
})();


// 流式文件写入   （异步 推荐写法） 适用于大型
const fs = require("fs")
// 创建可写流
let ws = fs.createWriteStream("text.txt")
// 写入流
ws.write("今天天气不错")
ws.write("吃了烧烤，喝了奶茶")
// 关闭流
ws.end()
// 检测可写流是否开启
ws.once("open",()=>{
	console.log("可写流开启")
})
// 检查可写流是否关闭
ws.once("close",()=>{
	console.log("可写流关闭")
})
//封装写法为：
function reFile(path){
  return new Promise((resovled,rejected)=>{
    fs.readFile(path,(err,data)=>{
      if(!err){
        resovled(data)
        console.log(data.toString())
      }else{
        rejected(err)
      }
    })
  })
}
(async ()=>{
  await reFile("text.txt")
})();
```

### 文件读取

```js
// 简单文件读取  （异步）
const fs = require("fs")
fs.readFile("bg.jpg",(err,data)=>{
  if(!err){
    console.log(data)
  }else{
    console.log("数据读取失败")
  }
})
// 封装写法为：
function reFile(path){
  return new Promise((resovled,rejected)=>{
    fs.readFile(path,(err,data)=>{
      if(!err){
        resovled(data)
        // wrFile(path,data)
      }else{
        rejected(err)
      }
    })
  })
}
(async ()=>{
  await reFile("text.txt")
})();

// 流式文件读取  （异步，推荐写法）
const fs = require("fs")
// 创建可读取流
let wr = fs.createReadStream("bg.jpg")
wr.on("data",data=>{
  console.log(data)
})
wr.once("open",()=>{
  console.log("读取流开启")
})
wr.once("close",()=>{
  console.log("读取流关闭")
})
```

### 读取文件并写入

```js
// 简单文件读取并写入
const fs = require("fs")
fs.readFile("bg.jpg",(err,data)=>{
  console.log(data)
  fs.writeFile("newbg.jpg",data,err=>{
    if(!err){
      console.log("文件写入成功")
    }
  })
})

//流式文件读取与写入
const fs = require("fs")
// 创建可读流
let wr = fs.creatReadStream("bg.jpg")
// 创建可写流
let ww = fs.creatWriteStream("newbg.jpg")
wr.once("open",()=>{
  console.log("可读流打开")
})
wr.once("close",()=>{
  console.log("可读流关闭")
})
ww.once("open",()=>{
  console.log("可写流打开")
})
ww.once("close",()=>{
  console.log("可写流关闭")
})
// 将读取的数据写入  (不推荐写法)
wr.on("data",data=>{
  ww.write(data)
})
// 将读取的数据写入  （推荐写法）
wr.pipo(ww)
```

### 遍历文件列表，读取某一项

```js
const fs = require("fs")
//用法：fs.readdir(目录,(err,res)=>{}) 
fs.readdir("../node",(err,res)=>{
  if(!err){
    res.forEach((item,index)=>{
      if(item === "index.html"){
        (async ()=>{
         	let s = awiat reFile(item)
          await wrFile("newIndex/html",s)
        })();
      }
    })
  }else{
    console.log("数据获取失败")
  }
})
```

### 读取文件列表，并判断里面每一项是不是文件

```js
const fs = require("fs")
const {resolve} = require("path")

function readFile(path){
  return new Promise((resolved,rejected)=>{
    fs.readdir(path,(err,file)=>{
      if(err){
        rejected(err)
        return
      }
      resolved(file)
    })
  })
}

function statFile(path){
  return new Promise((resolved,rejected)=>{
    fs.stat(resolve("./text",path),(err,res)=>{
      if(err){
        rejected(err)
        return
      }
      resolved(path+"是不是文件:"+res.isFile())
    })
  })
}

;(async ()=>{
  let a =await readFile("./text")
  console.log(a)
  a.forEach(async item => {
    let s = await statFile(item)
    console.log(s)
  })
})();
```



### node输入输出

```js
// 引入readline模块
const readline = require("readline") 
// 创建readline接口实例
let r1 = readline.createInterface({
  input:process.stdin,
  output:process.stdout
})
// question方法
r1.question("晚上吃什么好吃的?",res=>{
  console.log("我想吃:",res)
  r1.close()
})
// close事件监听
r1.on("close",()=>{
  // 结束程序
  process.exit(0)
})
// 封装写法为：
const readline = require("readline")
function userQue(req){
  return new Promise((resolved,rejected)=>{
    r1.question(req,res=>{
      resolved(res)
    })
  })
}
(async ()=>{
  let name = await userQue("你要创建的文件的名字")
  r1.close()
})();
```

### 路径模块和系统模块

```javascript
// 获取路径信息的扩展名
const path = require("path")
let url = "https://www.baidu.com"
let a = path.extname(url)
console.log(a)  // .com
```



### redaline按行读取

```js
// 可以输出打印每一行的内容 
const fs = require("fs")
const readline = require("readline")
let r1 = readline.createInterface({
  input: fs.createReadStream("./text.txt"), // 入口
  crlfDelay: Infinity  //指定windows的分隔符
})
// 读取并输出内容
r1.on('line',line=>{
  console.log(line)
})
```

### node中哈希加密

```js
// 引入加密模块
const crypto = require("crypto")
const fs = require("fs")
const os = require("os")

let pass = "新新小朋友123"
const hash = crypto.createHash("sha1") //加密的类型
// 加密的类型有： md5 sha1 sha256 sha512
hash.update(pass) //加密处理
let res = hash.digest("hex") // 输出格式
// 输出格式分为base64 和 十六进制(hex)
console.log(res)

// 浏览器自带的hash加密命令 （命令提示符内输出）
Certutil -hashfile "要加密的文件" sha1("加密的格式") 

```

### 流式写入大文件

```js
// 生成文件名字
let name = "/article/"+userInfo._id+Date.now()+".md"
// 存储文章 并且生成地址
// 创建一个可以写入的流，写入到文件 output.txt 中
var ws = fs.createWriteStream(path.join(__dirname,"../../static"+name));
// 使用 utf8 编码写入数据
ws.write(text);
// 处理流事件 --> finish、error
ws.once('finish', async function() {
  let nowTime = Date.now()
  if(nowTime - time > 2000){
    time = nowTime
    // 存入数据库
    await articleDB.create({
      author:userInfo._id,
      title,
      des,
      mdUrl:name
    })
    res.send({
      code:0,
      mes:"文章发表成功"
    })
  }
});

ws.once('error', function(err){
  console.log(err.stack);
});

// 标记文件末尾
ws.end();

```



### 

## Express

### Express使用

```js
// 下载并引入express模块
npm i express -S 
const express = require("express")

//实例化 使用
const app = express()

// 引用静态资源
app.use(express.static("静态资源目录"))

//中间件  
app.use((req,res,next)=>{
  // ...这里写的是操作
  next()
  //  结束之后使用next来接下一个
})

// 中间件处理数据（方面后面的数据处理）  app 可以链式调用
app.use(express.json())
   .use(express.urlencoded({extended: true}))
   .listen(3000,()=>{
      console.log("3000端口已开始监听")
    })

// get方式路由监听 并返回前端数据
app.get("/login",(req,res)=>{
  // 获取前端get发送的数据
  let query = req.query
  // 后端发送给前端的数据
  let data = {
    state:200,
    mes:"登录成功，正在跳转页面",
    query
  }
  // 后端发送给前端的数据
  res.send(data)
})

// 监听post路由
app.post("/login",(req,res)=>{
  // post方式 获取前端发送给后端的数据
  let data = req.body 
  let ss = {
    state:200,
    mes:"恭喜你，已成功登录",
    data
  }
  // 后端发送给前端数据
  res.send(ss)
})
```

### 路由

```js
// 在主文件中引入
const express = require("express")
const app = express()
//引用子路由
app.use("/",require("./router/index"))

// 在router文件夹创建index.js子路由
const express = require("express")
// 创建子路由
const router = express.Router()
// 使用子路由
router.get("/",async ()=>{...})
// 导出子路由
module.exprots = router

```

### 动态理由

```js
 const express = require("expres")
 const router = express.Router()
 router.get("/artircle/:id",async(req,res)=>{
   console.log(req.params.id) // 获取对应的id
 })
 module.exprots = router
```

### session持久化存储

```js
// 安装 session
npm i express-session -S

// 安装 connect-mongo 使登录信息存储到数据库，实现持久化存储
npm i connect-mongo -S

// 导入
const session = require("express-session")
const MongoStore = require("connect-mongo")

// session 存储在内存中	 

// 一般情况下，我们为了入口文件的可读性，会将session另外存到一个单独的文件里，入口文件直接导入即可
// 加载session中间件
app.use(require("./middleware/session"))

// session.js 文件
// 引入
const session = require("express-session")
const MongoStore = require("connect-mongo")
// 导出
module.exports = session({
    // 密钥字符串，服务端生成session的签名，可以随意写
  secret:"hjx",
  // 给前端设置cookie相关的设置，一般配置maxAge即可,也就是存储的时间
  cookie:{maxAge:7*24*60*60*1000},
  // 向服务发送请求后，是否重置cookie时间，建议为true
  rolling:true,
  // 是否强制重新保存session，即使它没有发生变化，建议为false
  resave:false,
  // 是否在session还未初始化时就存储，有利于前后鉴权，建议true
  saveUninitialized:true,
  // 让session存储到数据库中
  store:MongoStore.create({
    mongoUrl:"mongodb://localhost:27017/xinxin"
  })
})

// 此时，在路由监听里面，如果用户登录成功，我们只需要将用户的个人信息添加到cookie即可
if("登录成功"){
   req.session.userInfo = userInfo
}

// 自动登录
// 首先，页面通过ajax请求"/check"路由
// 前端：
$.get("/check")
.then(res=>{
  if(!res.code){
    // 已登录
    // ... 已登录后的操作
  }else{
    // 未登录
    // ... 未登录后的操作
  }
})
// 后端
router.get("/check",async(req,res)=>{
  if(!req.session.userInfo){
    // 未登录
    res.send({
      code:3,
      mes:"用户未登录"
    })
  }else{
   	// 已经登录
    res.send({
      code:0,
      mes:"登录成功",
      data:{
       	name:req.session.userInfo.name,
        uerImg:req.session.userInfo.userImg,
        .....
      }
    })
  }
})

// 退出登录 销毁cookie即可
// 前端
$("#exit").click(async ()=>{
  await $.get("/exit")
  //  。。。 清空vuex
})
// 后端
router.get("/exit",async (req,res)=>{
  // 销毁session
  req.session.destroy()  
})
```

## Express实战

### 上传头像

```js
// 安装 multer
npm i multer -S

// 引入multer
const multer = require("multer")

// 存储到磁盘
let upload = multer({
  storage:multer.diskStorage({
    // 文件存储的目录
    destination(req,file,cb){
      cb(null,join(__dirname,"../../../public/file/cover"))
    },
    // 文件的名字
    filename(req,file,cb){
      let name = req.session.userInfo._id
      let ext = path.extname(file.originalname)
      req.coverFile = name+Date.now()+ext
      cb(null,req.coverFile)  //名字
    }
  })
}).single("file") //name名字 elemetUI默认为file

// 发起请求时上传
router.post("/phone",async(req,res)=>{
  // 调用提前声明好的上传
  upload(req,res,async err=>{
    // 上传失败
    if(err){
      return res.send({
        code:7,
        mes:"上传失败，请稍后再试~"
      })
    }
    // 上传成功 
    // 修改数据库
    let {_id} = req.session.userInfo
    let userPhone = `/file/phone/${req.coverFile}`
     await userDB.findByIdAndUpdate(_id,{
      phone:userPhone
    })
    //改变session
    req.session.userInfo.phone = userPhone
    
    res.send({
      code:0,
      mes:"头像修改成功",
      data:req.session.userInfo //发给前端数据 改变vuex
    })
  })
})


// 上传多个和上传一个一样 
```

### 结合Token上传

```js
const express = require("express")
const router = express.Router()
const multer = require("multer")
const path = require("path")
const jwt = require("jsonwebtoken")
const adminDB = require("../../db/admin")

// 密钥
const cort = "coderxinxin2022-04-15"

// 用户id
let userID = null

// 上传先校验用户
// 检测是否登录
router.use(async (req, res, next) => {
  try {
    // 校验token
    // 获取请求头里面的token
    let token = (req.headers.authorization || "").slice(7)
    let info = null
    // 捕获错误
    try {
      info = jwt.verify(token, cort)
      const doc = await adminDB.findById(info._id)
      if (!doc)
        return res.send({
          code: 4,
          mes: "系统检测到该账号已被注销",
        })
      // 检测密码是否更新
      if (info.pass !== doc.pass)
        return res.send({
          code: 4,
          mes: "密码已更新,请重新登录",
        })
      userID = doc._id
      next()
    } catch (error) {
      console.log(error)
      return res.send({
        code: 4,
        mes: "登录信息已过期,请重新登录",
      })
    }
  } catch (error) {
    res.send({
      code: 5,
      mes: "服务器异常，请稍后再试",
    })
  }
})

// 上传用户背景封面
router.post("/cover", async (req, res) => {
  console.log("开始上传")
  // 调用提前声明好的上传
  upload(req, res, async (err) => {
    // 上传失败
    if (err) {
      return res.send({
        code: 7,
        mes: "上传失败，请稍后再试~",
      })
    }
    // 上传成功
    console.log("上传成功")
    res.send({
      code: 0,
      mes: "头像修改成功",
    })
  })
})
```

### 鉴权

```js
// 常用于鉴权用户是否登录
router.use((req,res,next)=>{
  if(!req.session.userInfo){
  	// 未登录
    return res.send({
      code:3,
      mes:"请先登录"
    })
  }
  // 已登录
  next()
})
```

#### 

### log4js

```js
// 安装
npm install log4js

// 配置
// 日志配置文件
const path = require("path")
const log4js = require("log4js");

// log4js默认的日志级别如下：ALL < TRACE < DEBUG < INFO < WARN < ERROR < FATAL < MARK < OFF
log4js.configure({
  appenders: {
    // 配置打印输出源
    trace: {
      type: "console", // 控制台打印日志
      // type: "file", // 表示日志输出为普通文件，在此种配置下日志会输出到目标文件夹的目标文件中，并会随着文件大小的变化自动份文件
      // type: "dateFile", // 表示是输出按时间分文件的日志，在此种配置下,日志会输出到目标目录下，并以时间格式命名，随着时间的推移，以时间格式命名的文件如果尚未存在，则自动创建新的文件.
      // compress: true, //（默认为false） - 在滚动期间压缩备份文件（备份文件将具有.gz扩展名）
      // maxLogSize: 10000000, // 文件最大存储空间,单位是字节,只在type: file模式有效,表示文件多大时才会创建下一个文件（ xxx.log .1 之类）
      filename: path.join(__dirname, "/logger/logs", "trace", "trace"), // 写入日志文件的路径
      pattern: "yyyy-MM-dd.log", //确定何时滚动日志的模式,只在type: dateFile模式有效,(默认为.yyyy-MM-dd0),表示一个文件的时间命名模式,格式:.yyyy-MM-dd-hh:mm:ss.log,在生成文件中会依照pattern配置来在filename的文件结尾追加一个时间串来命名文件。
      encoding: "utf-8", // default "utf-8"，文件的编码
      alwaysIncludePattern: true, //将模式包含在当前日志文件的名称以及备份中,只在type: dateFile模式有效,(默认为false),配置为ture即最终的日志路径文件名为filename + pattern
      //backups： 只在type: file模式有效,表示备份的文件数量,如果文件过多则会将最旧的删除。
    },
    file: {
      type: "file",
      filename: path.join(__dirname, "/logger/logs", 'log'),
      pattern: "yyyy-MM-dd.log",
      encoding: "utf-8",
      alwaysIncludePattern: true,
    },
    console: {
      type: "console",
    }
  },
  categories: {
    default: { appenders: ["console"], level: "trace" },
    cheese: { appenders: ["console", "file"], level: "warn" },
  }
});

module.exports.logger = log4js.getLogger();
module.exports.loggerGet = log4js.getLogger('GET');
module.exports.loggerPost = log4js.getLogger("POST")
module.exports.loggerPut = log4js.getLogger("PUT")
module.exports.loggerDelete = log4js.getLogger("DELETE")



// 使用
const { logger, loggerGet } = require('../log4js')

 logger.trace(mes)
    logger.debug('debug');
    logger.info('info');
    logger.warn('warn');
    logger.error('error');
    logger.fatal('fatal');

    loggerGet.trace("trace");
    loggerGet.debug('debug');
    loggerGet.info('info')
    loggerGet.warn('warn');
    loggerGet.error('error');
    loggerGet.fatal('fatal');
```

### 通过后端判断是否登录 并渲染

```js
// 处理那种如果用户没有登录访问一些指定的页面或者链接的时候，跳转到登录页面，如果已经登录，那么就正常跳转

// 判断是否登录
let isCheck = false; 

// 将判断的发送请求封装为一个async函数
async function isLogin() {
    let res = await $.get("/check",)
    if (res.state === 200) {
      // 已登录
      $(".user_success").show()
      $(".user-error").hide()
      isCheck = true
    } else {
      isCheck = false
    }
  }

// 数据渲染
;(async()=>{
  // 判断是否已经登录
  await isLogin()
  if (isCheck) {
      $(".yyds")[0].href = './more.html'
    } else {
      $(".yyds")[0].href = './dl.html'
    }
})();

```

### token验证

```js
// token和session类似，都是校验用户身份的一种手段，不同的是session存储与服务端内存或者数据库以及前端的cooking中，而token属于无状态的，只存在与前端的本地存储中，发送请求的时候添加到发送头，然后后端对其校验。

// 安装
npm i jsonwebtoken -S

// 在需要的地方引入
const jwt = require("jsonwebtoken")

// 创建一个密钥
const cort = 'xinxin123'

// 生成token
let token = jwt.sign({
  // 传入的数据
   _id,
  name,
  pass,
  // 有效时间
  exp: Math.floor(Date.now() / 1000) + 60*60*24,  // 一天时间过期
	},cort)

// 校验token
// 获取请求头里面的token
let token = (req.headers.authorization || "").slice(7)
let info = null
    // 捕获错误
    try {
      info =  jwt.verify(token,cort)
    } catch (error) {
      return res.send({
        code:4,
        mes:"无效的token，请重新登陆"
      })
    } 
	console.log(info) 
	// 返回格式
//{
  _id: '234sdfsasd123',
  name: 'web',
  pass: '123123',
  exp: 1636431734,
  iat: 1636431614
}

// 根据获取的数据 进行下一步操作
    if(info._id !== "234sdfsasd123"){
      return res.send({
        code:3,
        mes:"token失效或不存在"
      })
    }
```

### 邮件发送

```js
// 安装
npm i nodemailer -S

const email = require("nodemailer")

// 发送邮件定时器
let time = null
// 设置验证码
let yzm =`ahscaihlashfalsbfaljfba`

// 发送邮件
router.get("/email",async (req,res)=>{
  try {
    // 获取用户邮箱
    let userInfo = {
      email:"2927748798@qq.com"
    }
   let timer = Date.now()
   setTimeout(()=>{
      yzm =`ahscaihlashfalsbfaljfba`
   },60*1000)
   if(timer - time > 3000 ){
    time = timer
     // 获取验证码
     yzm =`${Math.random()*10|0}${Math.random()*10|0}${Math.random()*10|0}${Math.random()*10|0}${Math.random()*10|0}${Math.random()*10|0}`

     console.log(yzm*1)
   }else{
    console.log("请一分钟后再试")
    res.send({
      code:4,
      mes:"请一分钟后再试"
    })
    return
   }
    // 发送邮件
    let transporter = email.createTransport({
       host: "smtp.qq.com",
      port: 587,
      secure: false,
      auth: {
        user: "coderhjx@foxmail.com",
        pass: "opsbmqdckzriddjg",
      },
    })
  
    let opt = {
      from:'"新新小朋友科技有限有限公司"<2719382032@qq.com>',
      to:userInfo.email,
      subject:"爱分享官网修改密码邮箱信息",  //主题  
      text:"修改账号密码",  //文本内容
      html:`
        您的验证码为：<span style="color:red;">${yzm}</span>(有效期为30分钟)<br>
        若该操作并非您本人操作，说明该账号存在密码泄露风险，请前往修改密码。
      `,
      // attachments:[  //用来发送文件
      //   {
      //     filename:"packjson.json",  // 文件名称
      //     path:"./package-lock.json"       // 文件路径
      //   }
      // ]
    }  
    let info = await transporter.sendMail(opt)
    res.send({
      code:0,
      mes:"发送邮件成功",
      data:yzm
    })
  } catch (error) {
    res.send({
      code:5,
      mes:"服务器异常，请稍后再试"
    })
  }
})
```

### 验证码

```js
// 安装
npm install --save svg-captcha

const svgCaptcha = require('svg-captcha')

const captcha = svgCaptcha.create({
  width: 100,
  height: 30,
  noise: 2,
  color: true,
})

```



## TypeScript

### express结合typescript使用

```ts
// 目录
|- public
|- src
|--- router
|------ index.ts
|---- index.ts 项目入口
|- package.json
|- package-lock.json
|- tsconfig.json


// 初始化项目
npm init -y

// 新建tsconfig.json
{
  "compilerOptions": {
    // 编译选项
    "target": "es6", // 编译目标代码的版本标准
    "module": "commonjs", // 编译目标使用的模块化标准
    "lib": ["es6"], // 指定ts环境
    "outDir": "./dist", // 编译结果位置
    "removeComments": true, // 编译结果移除注释
    "strictNullChecks": true, // 在严格的null检查模式下，null和undefined值不包含在任何类型里，只允许赋值给void和本身对应的类型
    "declaration": true, // 生成 d.ts
    "esModuleInterop": true, // 这个选项允许我们默认导出的时候使用*代替导出的内容。
    "strict": true //允许严格类型检查。
  },
  "include": ["./src"], // 指定tsc编译的范围
  // "files": ["./src/index.ts"] // 指定编译文件，须删除"include"配置
  "exclude": ["node_modules"]
}

// 安装依赖
npm install typescript ts-node-dev @types/node @types/express --save-dev
npm install express --save

// 配置package.json
"scripts": {
  "dev": "ts-node-dev src/index.ts",
  "test": "echo \"Error: no test specified\" && exit 1",
      "build": "tsc",
   "build": "tsc"
}

// 引入和导出改为import 和 export

// 使用
// index.ts
import express from 'express'
const app = express()
import routeApi from './router'

app.use('/api', routeApi)

app.listen(3000, () => {
  console.log('3000端口已开启')
})

// router/index.ts
import express from 'express'
const router = express.Router()

router.get('/ts', (req, res) => {
  res.send({
    code: 0,
    mes: 'ts真帅',
    data: {
      info: req.query,
    },
  })
})

router.post('/ts', (req, res) => {
  res.send('POst请求')
})

export default router


```

### node+ts+express+RESTful API

```ts
// 目录
|- src  
|--- index.ts  // 入口
|- router
|--- user.ts 
|--- index.ts  // 路由入口
|- controllers // 存放不同的接口实例
|--- user.ts  
|--- menu.ts
|- package.json
|- ....

// router/index.ts
import express from 'express'
const router = express.Router()
import user from './user'

// 用户
router.use('/user', user)

export default router


// router/user/index.ts
import express from 'express'
import User from '../../controllers/user'

const router = express.Router()

// 用户  RESTful API风格
router
  .route('/')
  .get(User.findUser)
  .post(User.publishUser)
  .delete(User.deleteUser)
  .put(User.updateUser)

export default router


// controllers/user.ts
import type { Request, Response } from 'express'
import Mes from '../utils/format-mes'

class User {
  // 查找用户
  static findUser = (req: Request, res: Response) => {
    try {
      const { name } = req.query
      if ((name as string)?.includes('web')) throw '该用户已被注册'
      res.send(Mes.failed('该用户不存在，可以注册'))
    } catch (error) {
      res.send(Mes.failed(error as any))
    }
  }

  // 添加用户
  static publishUser = (req: Request, res: Response) => {
    try {
      const { name, pass } = req.body
      if (name === 'web') throw '该用户名已被注册'
      res.send(Mes.success('注册成功'))
    } catch (error) {
      res.send(Mes.failed(error as any))
    }
  }

  // 删除用户
  static deleteUser = (req: Request, res: Response) => {
    try {
      const { id } = req.body
      if (!(id as string).includes('1')) throw '该用户不存在'

      res.send(Mes.success('删除成功'))
    } catch (error) {
      res.send(Mes.failed(error as any))
    }
  }

  // 更新用户信息
  static updateUser = (req: Request, res: Response) => {
    try {
      const { name, newName } = req.body

      res.send(
        Mes.success('修改成功', {
          旧用户名: name,
          新用户名: newName,
        })
      )
    } catch (error) {
      res.send(Mes.failed(<any>error))
    }
  }
}

export default User



```

### 在typescript中使用jsonwebtoken

```js
pnpm install jsonwebtoken --save
pnpm install jsonwebtoken --save-dev
```

