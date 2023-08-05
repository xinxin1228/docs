## 基础知识

### 初始化文件

> 先创建一个data文件夹，然后在data文件里面再创建两个文件夹，一个为db文件夹，一个为logs文件夹，在logs文件夹里创建一个mongodb.log的日志文件。大致如下：

- data

  ​	-- db

  ​	-- logs

  ​		-- mongodb.log

```js
然后在data层进入cmd，添加下面连接语句：
mongod --dbpath ./data/db --logpath ./data/logs/mongodb.log
```

### 开启服务

上述步骤过于繁琐，因此我们直接在任务管理器中打开mongodb的服务即可，就无需上述的操作

### 安装引入

```js
npm i mongoose -S  // 安装mongodb
const mongoose = require("mongoose")  // 引用mongoose
```

### 连接数据库

```js
const mongoose = require("mongoose")
// 连接数据库  路由后面跟着库名 如果没有 则自动创建
mongoose.connect("mongodb://127.0.0.1:27017/user",{
  useNewUrlParser: true,    // 新版
  useUnifiedTopology: true
}，err=>{
	if(err){
  	console.log("数据库连接失败")
  	return
	} 
	console.log("数据库连接成功")
})

// 或者
;(async ()=>{
  await mongoose.connect("mongobd://127.0.0.1:27017/user",{
  useNewUrlParser: true,
  useUnifiedTopology: true
})
  .then(res=>{
    	console.log("数据库连接成功",err)
  })
  .catch(err=>{
    console.log("数据库连接失败",err)
  })
})();
```

### Schema架构、模板

```js
// 设置数据库的模板、架构、规范
const UserSchema = new mongoose.Schema({
  name:{
    type:String,  //类型
    required:true  //默认不为空
  },
  post:{
    type:String,
    defult:"学生"  //默认值
  },
  age:Number,
  sex:String,
  // 存储数据的时间 ,默认时间戳
  time:{
    type:Number,
    default:Date.now  // 注意 这里不能加括号，不然加了括号就是固定值了
    
  }
},{
  versionKey:false  //取消每项的版本信息 (_v)
})

//Schema中常用三个选项：
type  // 数据类型
required  //是否必须
default  //默认值 

//Schema的数据类型：
String Number Date Boolean Buffer Mixed ObjectId Array Decimal128

// 针对 type:String 时 
minlength：数字   // 最小长度
maxlength：数字   // 最大长度
match:正则        // 正则判断
trim:true       // 去除字符串两边的空格

// enum枚举 --存储的值必须是选项中的某一个
enum:[xx,xx,xx]

// 针对 type:Number 时
min:number   //最小值
max:number   //最大值

// 存数组
arr:[
  // 这里写数组里面所需要存储的数据规则
  {
    type:String
  }
]


```

### 建表

```js
// 创建一个名为 users 的 表
const userInfo = mongoose.model("users",UserSchema)
//第一个参数为表名，没有则自动创建，最好带上s，不然它会自动添加s，影响后期维护 第二个参数为架构，由前面已经设计的架构
```

### 添加数据

```js
// 第一种方式：
// 定义一个用户
let user1 = new userInfo({
  name:"typeScript",
  age:20,
  sex:"女",
  post:"web开发者"
})

// 添加用户
;(async ()=>{
  let a = awawt user1.save()
  console.log("数据添加成功",a)
})();

// 第二种方式
await user.create({
   name:"typeScript",
   age:20,
   post:"优秀的前端工程师"
})
.then(res=>{
  console.log("数据添加成功",res)
})
.catch(err=>{
  console.log("数据添加失败",err)
}))
```

### 数据查找

```js
;(async ()=>{
  const userInfo = mongoose.model("users",UserSchema)
  
  {}    //基础条件块，eg:  {name:"afei"}
	
	$or $nor   //或者 或者取反，eg:  {$or:[{name:"afei"},{age:"20"}]}
	
	$gt $gte $lt $lte $ne   //大于 大于等于 小于 小于等于 不等于，eg:  {age:{$lt:20}}
	$in $nin     // 在/不在 指定的多个字段之类，eg:  {name:{$in:["afei","zhuque"]}}
	$exists   // 存在某属性，eg:  {age:{$exists:true}}
	
	$size     //数组长度匹配，eg:  {arr:{$size:2}}
	$all    //数组中是否存在指定的所有项，eg:  {arr:{$all:[123,456]}}
	
	$where    //可以使用JavaScript代码或函数，eg:  {$where:"this.age===18"}
	正则   //使用正则匹配，eg:  {name:/afei/}
  
  // 获取所有数据
  let res = await userInfo.find({})
  
  // 根据ID查找
  let res = await userInfo.findById({"60c5c2f7247120461c40f63b"})
  
  // 获取满足要求的第一条数据
  let res = await userInfo.findOne({age:20})
  
  // 获取满足要求的所有数据
  let res = await userInfo.find({age:30})
  
  // 获取满足要求的前10条数据
  let res = await userInfo.find({age:30}).limit(10) 
  
  // 获取年龄大于20并且小于30所有数据
  // 第一种 适合查找多个条件
  let res = await userInfo.find({
    age:{
      $gt:20,
      $lt:30
    }
  })
  // 第二种 适合多个
  let res = await userInfo.find().where("age").gt(20).lt(30)
  
  // 获取大于20小于30的女性
  let res = await userInfo.find({
    age:{
      $gt:20,
      $lt:30
    },
    sex:"女"
  })
  
  console.log(res)
  
  // 随机获取十条数据
  let doc = await articleDB.aggregate([
    { $sample: { size: 10 }}  
  ])  
  
})();
```

### 数据删除

```js
;(async ()=>{
  const userInfo = mongoose.model("users",UserSchema)
  
// 查找并删除第一条数据  返回数据
let res = await userInfo.findOneAndDelete({age:30})
 
// 删除第一条数据   返回详情
let res = await userInfo.deleteOne({age:30})

// 删除所有的数据  返回详情
let res = await userInfo.deleteMany({age:30})

// 删除所有的数据 返回详情
let res = await 	userInfo.remove({age:30})
  
console.log(res)
})();
```

###  数据更新

```js
;(async ()=>{
const userInfo = mongoose.model("users",UserSchema)

// 将年龄为30的第一条数据改为32  返回详情
let res = await userInfo.updateOne({
 	age:30
},{
  age:32
})

// 将年龄所有大于30小于31的所有数据改为32  返回详情
let res = await userInfo.updateMany({
  age:{
    $lt:31,
    $gt:30
  }
},{
  age:32
})

// 将所有人的年龄+1
let res = awiat userInfo.updateMany({},{$inc:{age:1}})


console.log(res)
  
})();



// 更常见
const userDB =  mongoose.model("texts",new mongoose.Schema({
  author:String,    //作者
  friends:[String],  // 朋友
  likes:[String],    // 点赞人数
  info:{      // 详细信息
    name:String,   // 姓名
    age:Number,    // 年龄
    warry:Boolean，  // 是否结婚
    likes:[String]  // 关注数
  }
}))

// 存取一条数据
userDB.create({
  author:"xinxin",
  friends:["小王","小张","小李"],
  likes:["000","111"],
  info:{
    name:"新新小朋友",
    age:22,
    warry:false,
    likes:["1111",'2222','3333']
  }
})
       
// 测试案例
;(async ()=>{
  // 需求一：往xinxin朋友里面添加 小可爱 ()
	await userDB.updateOne(
    {
      author:"xinxin"
    },{
      $push:{
        friends:"小可爱"
      }
    }
  )
  
  // 需求二：使xinxin 年龄+1
  await userDB.updateOne(
    {author:"xinxin"},
    {
    $inc:{
      "info.age":1
    }
  })
  
  // 需求三：往xinxin里面添加a,b,c三个朋友
  await userDB.updateOne(
    {
      author:"xinxin"
    },
    {
      $push:{
        friends:{
          $each:["a","b","c"]
        }
      }
    }
    )
  
  // 需求四：删除xinxin中的好友a
  await userDB.updateOne(
    {
      author:"xinxin"
    },
    {
      $pull:{
        friends:"a"
      }
    }
    )
  
 // 需求五：往info里面的likes添加 'haha'
  await userDB.updateOne(
    {author:"xinxin"},
    {
      $push:{
        "info.likes":"haha"
      }
    }
  )
  
  // 需求六：删除info里面的2222
  await userDB.updateOne(
    {author:"xinxin"},
    {
      $pull:{
        "info.likes":"2222"
      }
    }
  )
})();

// 加上firends存放着姓名和id 根据id判断friends里面是否含有某个人的
	// 需求七 : 判断info 是否含有id为Cid的用户

方法一：用some遍历循环判断 为true表示有 反子则无
let doc = 当前的数据 
let isHas = doc.friends.some(item=>{
    return item._id+"" == Cid
 })

方法二： 用mongodb自带属性判断 有为对象 没有为null
let isHas = doc.friends.id(Cid)
	
   // 需求八 : 判断info 是否含有id为Cid的用户，如果有删除
let isHas = doc.friends.id(Cid)
await messageDB.findByIdAndUpdate(doc,{$pull:{friends:isHas}})



// 在使用findByIdAndUpdate进行相关操作时，需要在连接数据库时，添加一条语句"useFindAndModify:false"
```

### 留言与子留言

```js
// 留言点赞
// 检验留言是否存在
  let doc = await meesageDB.findById(mesId)
// 当前用户的id
  let {_id} = req.session.userInfo
// 检验是否已经点赞
  if(doc.likes.includes(_id)){
    // 已点赞
    await meesageDB.findByIdAndUpdate(mesId,{
      $pull:{
        likes:_id
      }
    })
    return res.send({
      code:0,
      mes:"取消赞成功",
      data:doc
    })
  }
  // 未点赞
  await meesageDB.findByIdAndUpdate(mesId,{
    $push:{
      likes:req.session.userInfo._id
    }
  })


// 子留言
// 添加子留言
    await meesageDB.findByIdAndUpdate(mesID,{
      $push:{
        childMes:{
          author:_id,
          text,
          replayUser:replayUserID
        }
      }
    })

// 留言删除
await meesageDB.findByIdAndRemove(mesID)

// 子留言删除
// 检测父留言是否存在
  let doc = await meesageDB.findById(itemID)
  if(!doc){
    return res.send({
      code:3,
      mes:"该留言已被删除，请刷新后重试"
    })
  }
// 检测子留言是否存在
  let isHas = doc.childMes.id(childID)
   // 删除子留言
  await meesageDB.findByIdAndUpdate(itemID,
    {
      $pull:
      {
        childMes:isHas
      }
    }
  )
  

// 子留言点赞
// 判断父留言是否存在
  let parent = await meesageDB.findById(pID)
  if(!parent){
    return res.send({
      code:9,
      mes:"该留言已被删除，请刷新后重试"
    })
  }
// 判断子留言是否存在
  let child = parent.childMes.id(cID)
  if(!child){
    return res.send({
      code:11,
      mes:"该回复已被删除，请刷新后重试"
    })
  }
// 是否点赞
if(child.likes.includes(id)){
    // 已点赞
    await meesageDB.findByIdAndUpdate(pID,{
      $pull:{
        [`childMes.${index}.likes`]:id
      }
    })
    res.send({
      code:0,
      mes:"已取消点赞"
    })
  }else{
    //未点赞  
    await meesageDB.findByIdAndUpdate(pID,{
      $push:{
        [`childMes.${index}.likes`]:id
      }
    })
    res.send({
      code:0,
      mes:"点赞成功"
    })
  }


// 检测文章下的评论是否存在 
// itemID：评论id
 let mess = doc.comment.id(itemID)
 
 // 文章添加评论
 await articleDB.findByIdAndUpdate(_id,{
   $push:{
     comment:{
       // 评论者id
       author:userInfo._id,
       // 评论内容
       text,
     }
   }
 })
// 文章删除评论
await articleDB.findByIdAndUpdate(_id,{
  $pull:{
    comment:mess
  }
})

// 第二种删除方法
await articleDB.findByIdAndUpdate(_id, {
  $pull: {
    comment: { _id: mess._id },
  },
})



// 文章评论回复
// index：当前评论的下标
 await articleDB.findByIdAndUpdate(_id,{
   $push:{
     // 第n条评论下的回复添加信息
     [`comment.${index*1}.reply`]:{
       // 回复的id
       author:userInfo._id,
       // 回复内容
       text
     }
   }
 })

// 删除文章评论下的回复
// 检测评论是否存在
let mess = doc.comment.id(item._id)
// 检测回复是否存在 第n条评论下的回复查询回复id
let isHas = doc.comment[index].reply.id(childItem._id)
await articleDB.findByIdAndUpdate(_id,{
  $pull:{
    // 删除第n条评论下的回复
    [`comment.${index*1}.reply`]:isHas
  }
})


```



### 表关联

```js
// 如果实现网站的留言功能
// 要求：登录不同的用户，可以查看所有的留言，并且在当前用户提交之后刷新留言，实现留言的永久存储

// 创建一个message.js文件，用来保存留言内容
const mogoose = require("mongoose")
const Schema = momgoose.Schema
const messageSchema = new Schema({
  // 留言内容
  message:{
    type:String,
    required:true
  },
  // 留言者
  author:{
    // 类型 ObjectId
    type:Schema.Types.ObjectId,
    // 关联哪个表
    ref:"users",
    required:true
  },
  // 留言时间，默认 不需要用户添加
  time:{
    type:String,
    default:Date.now
  }
})

module.exprots = mongoose.model("message",messageSchema)

// 前端
// 发送留言
$("#message").click(async()=>{
  let message = $("#mes").val()
  let {data} = await axios.post("/message",{
    data:{message}
  })
  showMess()
  console.log(data)
})

// 渲染留言
async function showMess(){
 	let ss = await axios.get("/showMes")
  $.each(ss,(index,value)=>{
    //.... 省略
  })
}
showMess()

// 后端
const messageDb = require("../db/message")
// 处理留言 
router.post("/message",async(req,res)=>{
  let {message} = req.body.data
	await messageDb.create({
    message,
    // 获取登陆后存储在cookie中的userInfo信息
    author:req.session.userInfo._id
  })
  res.send("留言成功")
})

// 显示留言
roter.get("/showMes",async(req,res)=>{
  // 把author的id信息转化为表关联有用的数据信息，使用populate("",{}),第一个参数是本地要替换的数据，第二个参数是不要显示出来的数据
  let userMess = await messageDb.find().pupulate("author",{pass:0,__v:0})
  res.send(userMess)
})

// 可以获取留言的作者 以及 子留言的作者
articleDB.findById(id).populate("author",{__v:0,pass:0}).populate("comment.author",{__v:0,pass:0})
```

#### 

## 高级查询

###  模糊查找 多条件查询 包含多个状态查找 关联查询 等

```js
// 比如查询 settledDB表 的条件有：
1. 农家乐名称查询 模糊查询
2. 联系人 模糊查询
3. 联系方式 模糊查询
4. 申请世家 包括开始时间和结束时间
5. 审核进度 可以选择多个进度
6. 申请状态 可以选择多个状态
并且 关联了 user表，字段为 createor

方法为：
const { agritainment_name, contacts, phone, create_time, progress, is_delete } = params;

const findName = new RegExp(`${agritainment_name || ""}`);
const findContacts = new RegExp(`${contacts || ""}`);
const findPhone = new RegExp(`${phone || ""}`);
const startTime = create_time?.split(",")[0];
const endTime = create_time?.split(",")[1];


return await settledDB
  .find({
  agritainment_name: findName, // 利用正则进行模糊查找
  contacts: findContacts,
  phone: findPhone,
  progress: {
    $in: (progress || "0,1,2,3")?.split(",") // 查询多个状态
  },
  is_delete: {
    $in: (is_delete || "0,1")?.split(",")
  },
  create_time: {
    $gte: startTime ?? 0,   // 查询时间的起始 如果不传 默认为0
    $lte: endTime ?? Number.MAX_VALUE // 查询时间的结束 如果不传 默认最大值
  }
})
  .populate("createor", { name: 1, _id: 1 }); // 将关联的字段列出来，并且将 createor 字段转化为 包含 name和_id 的对象

// 最后一行可以这样写
.populate('createor', ['name'])
```

### 转换多级嵌套

```js
// 转换多级嵌套
1. user表 中存放了 business 表的 _id，关联在 business
2. order表 order表里存放user的_id字段，关联在createor字段
// 需求：在order表中查询 createor 字段中的 business 字段

mongoDB.find({}, {}, {})
  .populate({
    path: "createor", // order表 表关联的字段
    select: ["agritainment_id", "score", "comment"], // 选择展示user表的字段
    populate: { // 继续表关联 user 下的 buiness 字段
      path: "business", // 关联business 
      select: ["name"]  // 展示 business 中的name
    }
  })
```

### 查找数组类型数据下包含某一数组任一值的数据

```js
// 类型
tags: { type: [String], required: true }


return await travelsDB.find(
  {
    tags: {
      $elemMatch: {
        $in: ['数据一', '数据二']
      }
    }
  },
  {},
  {
    sort: {
      views: -1
    }
  }
);
```

