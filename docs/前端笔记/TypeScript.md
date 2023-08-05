## 安装`TypeScript`

```js
// 全局安装
npm i -g typescript

// 检查安装的ts的版本好
tsc -v 

// ts编译为js
tsc text.ts
```

## VsCode自动编译TS代码

```ts
// 生成配置文件tsconfig.json
 tsc --init
// 修改tsconfig.json配置
"outDir": "./dist",
"strict": false,    
// 启动监视任务: 
终端 -> 运行任务 -> 监视tsconfig.json
```

## 基本用法

```ts
// 在变量后面加上 :数据类型即可
const a: number = 10

// 如果没有添加数据类型， 默认为any类型
const a = 10

// 如果变量是多个类型 可以使用 | 联合使用
const a: number | string = 10

// 设置数组类型
const arr: number[] = [1, 4, 23, 4]
const arr1: Array<string> = ["23", "1"]

// 接口 用来定义对象的类型
interface Person{
  name: string,
  age: number,
  // 设置可选属性 使用时不添加不会报错 :前面加上？ 可选参数必须接在必选参数的后面
  sex?: string
  // 设置添加任意类型
  [propName:string]: any,
  // 设置只读类型
  readonly read: string
}

const obj1:Person = {
  name:"web高级开发工程师",
  age:20
}

// 接口生成函数
interface searchFn {
  (text:string,type:string):string
}

const ss: searchFn = function (text,type) {
  return "string"
} 

ss("web","工程师")

// jquery 全局声明
declare var $:(slector:string) => any
$("#abc")


// 安装其他包
npm i @types/包名 -S

npm i @types/jquery -S
npm i @types/node -S


// 断言 ts告诉编译器，相信我 我知道我是什么类型 这样不会报错
// 方式一  变量 as 类型  
function getString(str:number|string):number{
  if((str as string).length){
    return (str as string).length
  }else{
    return str.toString().length
  }
}
// 方式二  <类型>变量
 function getString(str:number|string):number{
    if ((<string>str).length) {
      return (<string>str).length
    }else{
      return str.toString().length
    }
  }

// 函数可选参数 
// 注意：可选参数必须在必填参数的后面
function fun (x:number=1,y?:number,z?:number): number {
  return x+y+z
}

```

### 接口在函数中的运用

```js
// 接口约束 约束函数参数中的类型和值
interface Info{
  id: number,
  name: string,
  phone: string
}

function userInfo(info:Info):string{
  return info.name
}

```

### 泛型使用

```js
// 未使用泛型前
function fn1(x:nunber,y:number):Array<number>{
  return [x,y]
}
function fn2(x:string,y:string):Array<string>{
  return [x,y]
}
function fn3(x:boolean,y:boolean):Array<Boolean>{
  return [x,y]
}

// 使用泛型后 <T> 泛型变量T 表示任何类型
function fn<T>(x:T,y:T):Array<T>{
  return [x,y]
}

fn<String>("a","v")

// 函数声明
function f1<T>(x:T):T{
  return x
}
fn<number>(1)
// 函数表达式
var f2 = function<N>(x:N):N{
  return x
}
f2<string>("1")
// 箭头函数
let f3 = <T>(x:T):T =>x
f3<number>(1)

```

### 接口

```ts
// 定义一个用户信息接口，要必须有姓名和年龄和自我介绍，其他可能会有
interface Info{
  name: string,
  age: number,
  show(): string,
  [proName:string]: any
}

// 强制指定类型
let user1:Info = {
  name:"新新小朋友",
  age:20,
  show(){
     return `我是${this.name},今年${this.age}岁了`
  }
}

// 类实现接口
class user2 implements Info{
  name:string;
  age:number;
  constructor(name:string,age:number){
    this.name = name,
    this.age = age
  }
  show(){
    return "这是类实现接口"
  }
}
let userInfo = new user2("web架构师",22)
```

## 配置文件

```js
// 文件名称为 tsconfig.json 
ts的配置文件，ts编译器可以根据它的信息来对代码进行编译
{
  // 指定编译成js的版本
  "target": "ES5",
  // outDir  用来指定编译后文件所在的目录
  "outDir":"./dist",
  // rootDir 指定TS文件源码目录
   "rootDir": "./src",
  // module 编译之后的js代码导入导出规范，默认是commonjs，采用默认即可
   "module": "CommonJS",
   // lib 指定编译期间使用的库
   "lib": ["DOM", "ES2015"],
   // 所有严格检查的总开关 默认打开 
   "strict": false,
   // noImplicitAny 表达式或函数参数上有隐含的any类型时报错 注释 建议开启 
   "noImplicitAny": true,
   // strictNullChecks null和undefined即是类型也是值，只能赋值给any和unkndow以及他们各自的类型
   "strictNullChecks": true,
   // experimentalDecorators 使用装饰器
   "experimentalDecorators": true,
   // emitDecoratorMetadata 使用元数据
   "emitDecoratorMetadata": true
   // declaration 指定 TS 文件编译后生成相应的 .d.ts 文件 声明文件 书的目录
   "declaration": true,
   // 是否移除注释 默认为false
   "removeComments":true,
     
   // baseUrl  设置路径别名的根路径
   "baseUrl": 'src',
   // paths  设置对应文件夹的路径别名, 假设在任意文件使用src/controller下的文件，可以直接 import {} from '@/controller/...' 来使用
   "paths": {
     "@/controller/*": ["controller/*"],
     "@/service/*": ["service/*"],
     "@/dao/*": ["dao/*"]
   },
   // resolveJsonModule 启用导入 json 文件
   "resolveJsonModule": true,
   // esModuleInterop 这个选项允许我们默认导出的时候使用*代替导出的内容。
   "esModuleInterop": true,
   // sourceMap 为生成的js创建源映射文件
   "sourceMap": true,
   // declarationMap 为 .d.ts 生成源映射文件
   "declarationMap": true,
   // 默认catch子句变量为“unknown”而不是“any”
   "useUnknownInCatchVariables": false
     
  // 用来指定哪些ts文件需要tsc编译
  "include":[
    // src文件下面的所有文件夹下面的所有文件 
    "./src/**/*"
  ]
  // 不需要被编译的文件目录 默认值：["node_modules","bower-components","jspm_packages"]
  "exclude":[
    // src下面的hello目录里面的所有内容都不会被编译
    "./src/hello/**/*"
  ]
  // 是否对js文件进行编译 默认为false
  "allowJs":false,
  // 是否检测js文件语法 默认为false
  "checkJS":true,
  
  // 不生成编译后的文件 默认为false
  "noEmit":true,
  // 当有错误时，不生成编译后的文件 默认为false
  "noEmitOnError":false,
  // 使用设置编译后的文件开启严格模式 默认为false
  "alwaysStrict":true,
  
}
```

### 项目中的使用的配置文件格式

```json
{
  "compilerOptions": {
    "target": "ES5",
    "lib": ["ES2020", "DOM"],
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,

    "outDir": "./dist",
    "module": "commonjs",
    "baseUrl": "src",
    "paths": {
      "@/controller": ["controller"],
      "@/service": ["service"],
      "@/dao": ["dao"],
      "@/decorator": ["decorator"],
      "@/middleware": ["middleware/*"],
      "@/utils": ["utils/*"]
    },
    "resolveJsonModule": true,

    "allowJs": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "removeComments": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "useUnknownInCatchVariables": false
  },
  "include": ["./src"] // 这里不能使用rootDir，不然src外部的文件无法访问，这里指定tsc编译的范围
}

```

### ts配置完别名在编译运行和打包时依旧识别别名

```ts
// 接上，我们在使用上述的paths别名之后，在使用过程中无任何问题，但是在编译期间甚至打包期间会报找不到该 '@/xx' 模块的错误，因为ts在tsc编译期间不会去把路径别名替换成对应路径，因此就报出找不到模块的错误
import { UserController } from '@/controller';

let controller = new UserController(); // 报错 Error: Cannot find module '@/controller'

// 解决编译时的报错
pnpm add tsconfig-paths -D
// 修改运行命令
"dev": "ts-node-dev -r tsconfig-paths/register src/index.ts",
  
// 解决打包时错误
pnpm add tsc-alias -D
// 修改打包命令
"build": "tsc && tsc-alias"
```



## 声明文件

```tsx
// 关键字 declare 表示声明的意思，我们可以用它来做出各种声明

declare let/const // 声明全局变量
declare function  // 声明全局方法
declare class     // 声明全局类
declare enum      // 声明枚举
declare namespace // 声明（含有子属性）全局对象
interface/type    // 声明全局类型
declare module '' // 声明模块
```

## 装饰器

```ts
// 装饰器执行顺序
属性装饰器 => 参数装饰器 => 函数装饰器 => 类装饰器

@Controller('/user') // 类装饰器 ④执行
class UserController {
  @Autowired() // 属性装饰器 ①执行
  private readonly userService: UserService
  
  @Post('/login') // 方法装饰器 ③执行
  public login(@Info info){ // 参数装饰器 ②执行 
  }
}


// 属性装饰器
// 自动装配装饰器 依赖注入 创建和使用分离 
type PropertyDecorator = (targetPrototype: Object, propertyKey: string | symbol) => void
/**
 * @describe 自动装配类型实例到当前属性
 * @param dependencyid 依赖注入的id，默认为当前的属性名，非必要不写
 * @returns PropertyDecorator
 */
function Autowired(dependencyid: string | symbol): PropertyDecorator {
  // 类的原型  当前属性
  return (targetPrototype, propertyKey) => {
    // 如果没有传入 依赖id 那么使用当前属性名
    if (!dependencyid) dependencyid = propertyKey; 
    // 获取当前属性的类型，也就是类
    const ProClass = Reflect.getMetadata('design:type', targetPrototype, propertyKey);
    // 对该类进行实例化，然后利用数据描述符赋值给类中被装饰的参数
    const ProClassInstance = new ProClass();
    // 赋值
    Reflect.defineProperty(targetPrototype, propertyKey, { value: ProClassInstance });
  }
}

// 函数装饰器
// 请求方法装饰器 Get Post
type MethodDecorator = (targetPrototype: Object, propertyKey: string | symbol, descriptor: TypedPropertyDescriptor) => void
// 请求方式
function ReqMethod(reqType: ReqMethods): MethodDecorator {
  // 请求路径
  return (reqPath: string = '/') => {
    // 当前类的原型对象  方法名  方法的数据属性描述符
    return (targetPrototype, methodName, descriptor) => {
			// 将当前函数的请求类型和请求路径保存到元数据中 ，最后在controller中统一注册
      Reflect.defineMetadata('reqPath', reqPath, targetPrototype, methodName);
      Reflect.defineMetadata('reqPath', reqPath, targetPrototype, methodName);
    }
  }
}
// 枚举出请求类型
export enum ReqMethods {
  Get = 'get',
  Post = 'post',
  Put = 'put',
  Delete = 'delete'
}
// 依次导出请求方式
export const Get = Request(ReqMethods.Get);
export const Post = Request(ReqMethods.Post);
export const Put = Request(ReqMethods.Put);
export const Delete = Request(ReqMethods.Delete);


// 中间件装饰器
// 用来处理 到达真实请求函数之前的处理，比如判断是否登录 判断权限
/**
 * @describe 为当前路由注入中间件
 * @param middleware 中间件函数
 * @return MethodDecorator
 */
function middleware(middleware: RequestHandler): MethodDecorator {
  return (targetPrototype, methodName) => {
    const middlewares = Reflect.getMetaData('middlewares', targetPrototype, methodName) || []
    middlewares.push(middleware)
    Reflect.defineMetadata('middlewares', middlewares, targetPrototype, methodName)
  }
}


// 类装饰器 
// controller 用来自动注册当前类和方法的所有路由
/**
 * @describe 自动注册当前controller下的所有路由
 * @param rootPath 当前controller请求的根路径，不写默认为小写前半部分
 * @returns ClassDecorator
 */
function Controller(rootPath?: string): ClassDecorator{
  return (target) => {
    const targetPrototype = target.prototype
    // 如果根目录不写 自动获取该类的名称作为路由  UserController => /user
    if (!rootPath) rootPath = '/' + target.name.split('Controller')[0].toLowerCase();
    // 从当前类的原型上 遍历出所有的方法
    for(const methodName in targetPrototype){
      // 获取当前方法的请求路径
      const reqPath: string = Reflect.getMetadata('reqPath', targetPrototype, methodName);
      // 如果请求路径不存在 跳过本地执行
      if (!reqPath) continue;
      // 请求类型
      const reqType: ReqMethods = Reflect.getMetadata('reqType', targetPrototype, methodName);
      // 请求方法
      const reqHandler: RequestHandler = Reflect.get(targetPrototype, methodName);
      // 请求该方法的中间件
      const middlewares: Array<RequestHandler> = Reflect.getMetadata('middlewares', targetPrototype, methodName);
      // 自动注册路由
      if (reqPath && reqType)
        if(middlewares){
          router[reqType](rootPath + reqPath, ...middlewares, reqHandler)
        } else {
          router[reqType](rootPath + reqPath, reqHandler);
        }
    }
  }
}


```



## webpack打包ts代码

```js
// 创建一个新的文件夹
// 初始化项目
npm init -y
// 安装webpack
npm i -D webpack webpack-cli typescript ts-loader
// 配置文件

// 创建配置文件
webpack.config.js
// 开始配置
const path = require("path")

module.exports = {
  // 指定入口文件
  entry:"./src/index.ts",
  // 指定打包文件所在目录
  output:{
    // 指定打包文件的目录
    path:path.resolve(__dirname,"dist"),
    // 打包后文件的文件
    filename:"bundle.js"
  },
  // 指定webpack打包时要使用的模块
  module:{
    // 指定要加载的规则
    rules:[
      {
        // test指定的是规则生效的文件
        text:/\.ts$/,
        // 要使用的loader
        use:"ts-loader",
        // 要排除的文件
        exclude:/node-modules/
      }
    ]
  }
}
```

## TS继承

```js
// 什么是原型链查找
子对象首先在自己的对象空间中查找要访问的属性和方法，如果找到，就输出，如果没有找到，就沿着 [__proto__]属性指向的原型对象空间中查找有没有这个属性和方法，如果找到出就输出，没有找到就继续沿着原型对象空间中 【__proto__】去查找上一级的，直到找到Object.prototype的原型对象属性所指的原型对象空间位置，如果再找不到，就输出undefined

// 继承的作用
代码的复用，节省空间，提交效率，

TS 继承关键字 extends
class Person{
  constructor(public name: string, public age: number){}
  
  // 方法 eat 并不是 Person 对象中，而是在 parent 对象的原型空间上
  public eat(){
    console.log(`${this.name}正在吃饭`)
  }
}

class Student extends Person{
  constructor(public name: string, public age: number, public score: number){
    // 使用 super 调用父类的构造器，并传递参数
    super('web', 22, 100) // 类似于js继承中的 Person.call(this, 'web', 22, 100)
  }
  
  // 重写父类的方法
  public eat(){
    // 调用父类的方法
    super.eat() // 类似于 Person.prototyoe.eat()
    console.log(`吃的小火锅`)
  }
}

// 修饰符
// public 公共属性或方法，可以在本类中、子类中以及类的外部所使用的
// protected 受保护的类，可以在本类和子类中使用，但是不能在类的外部使用
// private 私有类，只可以在本类中使用
// 子类属性和方法的修饰符权限一定要大于等于父类的修饰符的权限，比如 父类 protected， 子类 public
```

### 什么是类

```js
定义： 类是拥有相同属性和方法的一系列对象的集合，类是一个模具，是从这该类包含的所有具体对象中抽象出来的一个概念，类定义了它所包含的全体对象的静态特征和动态特征

以人类为例
静态特征【属性】 高度、姓名、年龄
动态特征【方法】 吃饭、走路                                                                              
```

### 获取Function 和 class上的方法

```js
// 在ES6之前，function既可以被当作函数使用，也可以当作类来使用
function Person(name, age){
  this.name = name
  this.age = age
}
Person.prototype.eat = function(){
  console.log(`${this.name}正在吃饭`)
}

// 获取Perosn的所有方法
Person.prototype


// class
class Person {
  constructor(public name: string, public age: number) {}

  public eating() {}
  public walking() {}
  public xx() {}
}

const per = new Person('web', 22)
const prototype = Object.getPrototypeOf(per)

console.log(Reflect.ownKeys(prototype))
```

## interface 和 type区别

```js
type 和 interface 类似，都用来定义类型，但 type 和 interface 区别如下：
// type 也称为类型别名
// interface 称为接口

相同点一：
它们都可以定义类型

不同：
区别一：
type可以声明基本数据类型别名/联合类型/元组等

区别二：
interface可以使用extends实现继承，type无法实现继承，但是可以通过交叉类型实现相同的效果

区别三：
在一个文件中定义多个interface时，会互相合并，但是在一个文件中定义多个type时，会报错, 注意，后续申明的interface类型必须属于同一类型， 下面这样是错误的

interface Person {
  name: string | number
}
interface Person {
  name: number // 报错， 后续属性声明必须属于同一类型。属性“name”的类型必须为“string | number”，但此处却为类型“number”。
}
```

## 函数重载

```js
// 函数重载适用于完成项目中某种相同功能但细节又完全不相同的应用场景，

// 优势
1. 结构分明，
2. 各司其职，自动提示方法和属性
3. 更利于功能扩展

// 函数重载定义
1. 由一个实现签名+一个或多个重载签名合成
2. 在外部调用函数重载定义的函数时，只能调用重载签名，不能调用实现签名。 因为实现签名的函数体是给重载签名编写的，实现签名只是在定义时起到了统领所有重载签名的作用，在执行调用时就看不到实现签名了。
3. 调用重载签名时，会根据传递的参数来判断调用的是哪一个函数
4. 只要一个函数体，只有实现签名配备了函数体，所有的重载签名都只有签名，不具备函数体
5. 无论重载签名有几个参数，参数类型是何种类型，实现签名都可以是一个无参函数签名，实现签名个数可以小于重载签名的参数个数，但实现签名准备包含重载签名的某个位置的参数时，那实现签名就必须兼容所有重载签名该位置的参数类型。
6. 必须给重载签名提供返回值类型，TS无法默认推导，


// 要求根据id找出对应的单条消息，根据消息类型找到消息列表，
function findMes(id: number): MesInter | undefined
function findMes(type: MesTypes): Array<MesInter>
function findMes(params: number | MesTypes): any {
  if (isNumber(params)) {
    return messages.find(item => item.id === params)
  } else {
    return messages.filter(item => item.type === params)
  }
}
```

## 单件设计模式

```js
// 什么是设计模式
设计模式就是一种更好的编写代码的方案

// 单件设计模式就是一个类对外有且只有一个实例，这种编码方案就是单件设计模式

// 应用场景： 
1. vuex和redux作为全局状态管理容器，被设计为唯一的对象，也就是是单件设计模式
2. 自己封装的localStore和SessionStore也是单件设计模式

// 构造单件设计模式 【懒汉式单件设计模式，延迟调用】
1. 把构造器设置为私有的，不允许外部来创建类的实例
2. 至少应该提供一个外部访问的方法或属性，外部可以通过这个方法或属性来得到，所以应该把这个方法设置为应该方法
3. 外部调用第二步提供的静态方法来获取一个对象

// 构造单件设计模式 【饿汉式单件设计模式，立即创建对象】
1. 把构造器设置为私有的，不允许外部来创建类的实例
2. 建立一个静态引用属性，同时把这个静态引用属性直接指向一个对象
（ private static localStore: Cache = new Cache() ）
3. 外部调用第二步提供的静态方法来获取一个对象
饿汉式单件设计模式，是不管你有没有使用，一上来就给创建对象


// 采用 懒汉式 单件模式封装的local，比全部是静态方法的要好，节省内存开销
class Cache {
  // 这里改为静态属性，应该静态方法中只能访问静态属性和静态方法
  private static instance: Cache
  private constructor() {
    console.log('调用了Cache的构造函数')
  }

  public static getInstance() {
    if (!this.instance) {
      this.instance = new Cache()
    }
    return this.instance
  }

  public set(key: string, value: any) {
    localStorage.setItem(key, JSON.stringify(value))
  }

  public get(key: string) {
    let val = localStorage.getItem(key)
    return val ? JSON.parse(val) : null
  }

  public remove(key: string) {}
}
export default Cache.getInstance()

// 使用
import Cache from './cache'

Cache.clear()


// 饿汉式设计模式
class Cache {
  public static instance: Cache = new Cache()
  private constructor() {
    console.log('Cache 执行了')
  }

  public set() {}
  public get() {}
  public remove() {}
}

export default Cache.instance


// 继承式
abstract class Cache {
  constructor(public cacheType: CacheTypes) {}

  public abstract set(key: string, value: any): void
  public abstract get(key: string): any
  public abstract remove(key: string): void
}

class LocalCache extends Cache {
  public static localCache: LocalCache = new LocalCache()

  constructor() {
    super(CacheTypes.localhost)
  }

  public set(key: string, value: any) {
    localStorage.setItem(key, JSON.stringify(value))
  }
  public get(key: string) {
    const val = localStorage.getItem(key)
    return val ? JSON.parse(val) : val
  }
  public remove(key: string) {
    localStorage.removeItem(key)
  }
}

export = {
  LocalCache: LocalCache.localCache,
}

```

## 静态方法

```js
1. 带static关键字的方法就是一个静态方法
2. 静态方法和对象无关，外部的对象变量不能调用静态方法和静态变量
3. 外部可以通过类名来调用
4. 静态方法内部通过 this 不可以访问实例属性，只能方法静态方法和属性

1. 外部如何调用ts类的静态成员
答：类名.静态属性  类型.静态方法
2. TS类的一个静态方法如何调用其它的静态成员
答：使用 this 来获取静态成员
3. 静态方法是否可以访问实例方法和实例属性，反过来呢？
答：都不能
4. 对象变量是否可以访问静态成员
答：不能
5. 一个静态方法改变了某个静态属性，其它静态方法或类外部任何地方调用这个属性都会发生变化
6. 静态成员保存在内存哪里？何时分配的内存空间？
答：任何一个TS类中的静态成员存储在内存的静态区，运行一个TS类，TS首先会为静态成员开辟内存空间，静态成员的内存空间分配的时间要早于对象空间的分配，也就是任何一个对象创建之前TS就已经为静态成员分配好了空间。但一个静态方法只会分配一个空间，只要当前服务器不重启或者控制台没有结束之前，那么静态方法一直存在于空间，无论调用多少次这个静态方法，都是调用的同一块空间
7. 静态方法或属性和原型对象空间上的方法或属性有何区别？
答：原型对象空间上的方法和属性用来提供给所有类的对象变量公用的方法和属性，没有对象和对象变量，原型上的属性和方法就没有了用武之地，而静态方法或属性属于类，可以通过类直接调用
8. 静态方法是否可以接受一个对象变量来作为方法的参数？
答：可以，静态方法内部不能通过 this 来访问对象的实例属性和实例方法，但可以通过调用静态方法时把对象变量传递给静态方法来使用，比如：我们把js的Object构造函数想象成一个 TS 类，Object 类中 用户大量的静态方法，例如：keys, call等，比如 Object.keys() 就可以接受一个对象来作为参数
9. 何时应该把一个方法定义成静态方法或属性定义为静态属性
答： 应用一： 单件设计模式中的静态方法和静态属性就是很好的应用之一。
应用二：当类中的某个方法没有任何必要使用任何对象属性时，而且使用了对象属性反而让这个方法的逻辑不正确，既然如此，就应该禁止这个方法访问任何对象属性和其它的对象方法，这时就应该把这个方法定义为静态方法。
应用三：当一个类中某个方法只有一个或1~2个对象属性，而且更重要的是，你创建这个类的对象毫无意义，我们只需要使用这个类中的一个或多个方法就可以，你们这个方法就应该定义为静态方法，常见的工具类中的方法通常都应该定义为静态方法，


彩蛋: new 一个 TS 类的方法可以吗？能在TS类外部使用prototype为TS类增加属性和方法吗？
答：虽然在JS中可以 new 一个类内部定义的对象或静态方法，但 TS已经屏蔽了去new一个类中的方法【JS可以，因为JS函数具有双重性质，被new时会当成一个构造函数】，TS类可以访问prototype原型对象属性，但无法在prototype原型对象属性增加新的方法或属性，这么做，就是为了让我们只能在类的内部定义方法，防止回到ES5从前非面类和对象的写法。【但是可以覆盖类上已经存在的方法】

```

## 类型断言

```js
// 在某些情况下，我们比ts更清除这个数据的类型，这个时候，我们可以使用断言来告诉ts：相信我，我知道这元素的类型

// 断言主要分未两类
<>  // 尖括号   <number>num   
as  // 使用as   num as number

// 非空断言
// 在上下文中当类型检查器无法断定类型时，一个新的后缀表达式操作符 ! 可以用于断言操作对象是非 null 和非 undefined 类型。具体而言，x! 将从 x 值域中排除 null 和 undefined 。

obj!.name

// 把两种能有重叠关系的数据类型进行相互转化的一种TS语法，把其中的一种数据类型转化为另外一种数据类型。类型断言和类型转换产生的效果一样，但语法格式不同

// 断言语法
A as B 或 <B>A

// 案例
class Person {
  constructor(public name: string, public age: number) {}
  public eat() {
    console.log(`${this.name}正在吃饭`)
  }
}

class Student extends Person {
  constructor(public name: string, public age: number, public score: number) {
    super(name, age)
  }
  public study() {
    console.log(`${this.name}正在学习`)
  }
}

// 具有重叠关系的常见
1. 如果A，B都是类，并且有继承关系
【extends】关系，无论A，B谁是父类或子类，A的对象变量可以断言为B类型，B的对象变量可以断言为A类型。但注意一般情况下是把父类对象断言成子类
const per = new Person('web', 22)
const stu = new Student('web', 22, 100)
per as Student // 正确
stu as Person // 正确
<Student>per // 正确
<Person>stu // 正确

2. 如果A，B都是类，但是没有继承关系
两个类中的任意一个类的所有public实例属性【不包括静态方法】加上所有的public实例方法和另外一个类的所有public实例属性加上所有的public实例方法 完全相同 或者 是另外一个类的子类，则这两个类可以相互断言，否则这两个类不能相互断言


3. 如果A是类，B是接口，并且A类实现了B接口
则A的 对象 可以断言成 B 接口的类型，同样，B接口类型的对象变量也可以断言成A类型

4. 如果A是类，并且A类没有实现B接口，规则和 2 相同

5. 如果A是类，B是type定义的对象数据类型【就是引用数据类型，列如Array，对象，不能是基本数据类型，列如string,number,boolean】，并且A类实现了Btype定义的数据类型，则A的对象变量可以断言成Btype定义的对象数据类型，同样Btype定义的对象数据类型的对象变量也可以断言成A类型

6. 如果A是类，B是type定义的对象数据类型，并且A类没有实现Btype定义的数据类型，规则和 2 一样

7. 如果A是一个函数上参数变量的联合类型，例如string|number，那么在函数内部可以断言成string或number类型

8. 多个类组成的联合类型如何断言？
例如：let vechile: Car | Bus | Truck，vechile可以断言成其中任意一种数据类型，例如 vechile as Car, vechile as Bus, vechile as Truck

9. 任何数据类型都可以转换成 any 或 unknown，同样，any 和 unknown 类型也可以转换成任何其它数据类型
```

##  类型守卫

### 概念

```js
// 作用：
在语句的块级作用域，【if语句内或条目运算符表达式内】缩小变量的一种类型推断的行为

// 四种类型守卫
1. 类型判断： typepf
2. 属性或者方法或者函数判断：in
3. 实例判断：instanceof
4. 字面量相等判断 ==，===，!=， !==
 
// instancef 原理
A instanceof B  =>  A.__proto__ ... .__proto__ === B.protptype
  
```

### 自定义守卫

```ts
// 自定义守卫格式
function 函数名(形参： 参数类型【参数类型大多为any】)： 形参 is A类型{
  return true or false
}
// 返回布尔值的条件表达式赋予类型守卫的能力，只有当函数返回true时，形参被确定为 A类型 
// 例：
function isString(value: any): value is any {
  return typeof value === 'string'
}
```



## 多态

```js
// 定义：
父类的对象变量可以接受任何一个子类的对象，从而利用这个父类的对象变量来调用子类中重写的方法而输出不同的结果

// 产生多态的条件
1. 必须存在继承关系
2. 必须有方法重写

// 多态的好处
利用项目的扩展【从局部满足了 开闭原则-对修改关闭，对扩展开放】

// 多态的局限性
无法直接调用子类独有的方法，必须结合instanceof类型守卫来解决

```

## 抽象类

```js
一个在任何位置都不能被实例化的类就是一个抽象类 【abstract】

1. 什么样的类可以被定义为抽象类
从宏观上来说，任何一个实例后毫无意义的类都可以被定义为抽象类。比如：我们实例化一个玫瑰花类的对象变量，可以得到一个具体的 玫瑰花 对象，但是如果我们实例一个 花类 的对象变量，那毫无意义。因此抽象类 不可以 被 实例化

// 一个类定义为抽象类的样子
abstract class Flower {
  constructor(public name: string) {}
	// 可以有 0 或 多个抽象方法， 只有方法体，没有方法的实现体
  public abstract eat(): string
  // 也可以包含 0 或 多个 具体方法
  public walk(){
    console.log(`${this.name}正在散步`)
  }
}

// 抽象类的特点
可以包含只有方法声明的方法【和方法签名类似，就是多了abstract关键字】，也可以包含实现了具体功能的方法，可以包含构造器，但不能直接实例化一个抽象类，只能通过子类来实例化

// 抽象类比普通类充当父类给项目带来的好处
1. 提供统一名称的抽象方法，提高代码的可维护性：抽象类通常用来充当父类，当抽象类把一个方法定义为抽象方法，那么会强制在所有子类中实现它，防止不同子类的同功能的方法命名不相同，从而降低项目维护成本。
2. 防止实例化一个实例化后毫无意义的类
```

## 泛型

```js
// 定义：
1. 泛型是一种 参数化 的数据类型，定义时不明确，但在使用时明确的一种数据类型
2. 编译期间进行数据类型安全检查的数据类型

// 注意：
1. 类型安全检查发生在编译期间
2. 泛型是参数化的数据类型，使用时明确化后的数据类型就是参数的值

// 泛型的格式
// class 类名<泛型形参类型> 泛型形参类型一般有两种表示，1.A-Z的任何一个字母，2. 语义化的单词来表示，绝大多数的情况，泛型都是采用第一种形式表示，例如：
class Array<T>{
  public arr!: Array<T>
}

// 问题：
一. 为什么 object 不能代替类上的泛型？
1. 编译期间object无法进行类型安全检查，而泛型在编译期间可以进行类型安全检查
2. object类型数据无法接受非object类型的变量，只能接受object类型的变量，泛型能轻松做到
3. object类型数据获取属性和方法时无自动提示

二. 为什么 any 不能代替类上的泛型？
1. 编译期间 any 无法进行类型安全检查，而泛型在编译期间可以进行类型安全检查
2. any 扩大数据类型的属性后没有编译错误导致潜在风险，而泛型却有效避免了此类问题的发生
3. any类型数据获取属性和方法时无自动提示，泛型有自动提示

```

## 泛型约束

```js
// 泛型约束就是把泛型的具体化数据类型范围缩小

// 语法
T extends object
extends 表示具体化的泛型类型只能是object类型，某个变量如果能断言成object类型【变量 as object】，那么这个变量的类型就符合T extends object，就是说该变量的类型可以是T的具体化类型。

```

## 综合练习案例

### 综合练习案例一

```js
// 涉及 自定义类型守卫 泛型 泛型约束 继承 抽象类 方法重载
// 要求
/**
 *
 * 1. 创建一个ArrayList类，要求传入参数必须是Array<object>类型，具有 add(添加), get(获取， 可以根据下标获取对应的值， 根据值获取对应下标的功能)，delete(删除，输入下标可以删除，返回删除内容， 输入内容，返回删除的下标，)，size(获取当前列表的长度)， show（获取所有的具体，可选参数：获取参数指定的长度）。
 * 2. 抽象一个车类(Vechile)，包含抽象属性 total(租聘价格),vechType(车的种类),vechNo(车牌),day(租聘天数),deposit(押金)，以及抽象方法calculateRent(计算租聘价格)
 * 3. 根据抽象车类，派生出Car(小汽车类),Truck(大卡车类),Bus(公交车类)，派生类被实例化的时候无需输入total属性，小汽车增加属性carType(小汽车车型)和方法computedDeposit(得到不同车型车的价格)，大卡车增加属性weight(大卡车的重量)和方法computedDeposit(得到不同吨数的价格)，公交车增加属性place(公交车的座位数)和方法computedDeposit(得到不同座位的价格)
 * 4. 新建一个顾客类，包含私有属性bill(顾客账单， 该属性类型为ArrayList类型， 可以在实例顾客类时传递，也可以后续添加)，具有方法add(加入购物车 添加商品)， show(顾客账单), cost(计算所有商品的租聘费用，包含租金+押金)
 *
 */

const isNumber = (value: any): value is number => typeof value === 'number'

class ArrayList<T extends object> {
  private zIndex: number = 0
  constructor(private arr: Array<T> = []) {}

  public add(item: T) {
    this.arr[this.zIndex++] = item
  }

  public get(index: number): T | undefined
  public get(item: T): number
  public get(indexOrItem: number | T): any {
    if (isNumber(indexOrItem)) {
      return this.arr[indexOrItem]
    } else {
      return this.arr.findIndex(item => item === indexOrItem)
    }
  }

  public delete(index: number): T | undefined
  public delete(item: T): number
  public delete(indexOrItem: number | T): any {
    this.arr.filter((item, index) => {
      if (isNumber(indexOrItem)) {
        return index !== indexOrItem
      } else {
        return item !== indexOrItem
      }
    })
  }

  public size(): number {
    return this.zIndex
  }

  public show(limit?: number): Array<T> {
    return limit ? this.arr.slice(0, limit) : this.arr
  }
}

abstract class Vechile {
  public abstract total: number // 租聘价格
  public abstract vechType: string // 车的种类
  public abstract vechNo: string // 车牌
  public abstract day: number // 租聘天数
  public abstract deposit: number // 押金

  // 计算租聘价格
  public abstract calculateRent(): number
}

// 小汽车
class Car extends Vechile {
  public total: number = 0
  constructor(
    public vechType: string,
    public vechNo: string,
    public day: number,
    public deposit: number,
    public carType: string // 小汽车车型
  ) {
    super()
  }

  // 得到不同车型车的价格
  public computedDeposit(): number {
    let price
    switch (this.carType) {
      case '奥迪':
        price = 300
        break
      case '大众':
        price = 200
        break
      default:
        throw new Error('暂无该车型')
        break
    }

    return price
  }

  public calculateRent(): number {
    this.total = this.computedDeposit() * this.day
    return this.total
  }
}

// 大卡车
class Truck extends Vechile {
  public total: number = 0
  constructor(
    public vechType: string,
    public vechNo: string,
    public day: number,
    public deposit: number,
    public weight: number //大卡车的重量
  ) {
    super()
  }

  // 得到不同吨数的价格
  public computedDeposit(): number {
    let price
    if (this.weight < 100) {
      price = 50
    } else if (this.weight < 400) {
      price = 100
    } else {
      throw new Error('抱歉，暂没有该要求的卡车')
    }

    return price
  }

  public calculateRent(): number {
    return this.day * this.computedDeposit()
  }
}

// 公交车
class Bus extends Vechile {
  public total: number = 0
  constructor(
    public vechType: string,
    public vechNo: string,
    public day: number,
    public deposit: number,
    public place: number // 公交车的座位数
  ) {
    super()
  }

  // 得到不同座位的价格
  public computedDeposit(): number {
    let price
    if (this.place < 50) {
      price = 300
    } else if (this.place < 100) {
      price = 500
    } else {
      throw new Error('抱歉，暂没有该要求的公交车')
    }

    return price
  }

  public calculateRent(): number {
    return this.day * this.computedDeposit()
  }
}

const car1 = new Car('汽车', '京A123d', 5, 1000, '大众') // 1000+1000 2000
const truck1 = new Truck('卡车', '京B3412', 3, 2000, 200) // 300+2000  2300
const bus1 = new Bus('公交车', '京C12312', 4, 3000, 20) // 1200+3000 4200

// 顾客类
class Customer<T extends object> {
  // 顾客账单
  private bill: ArrayList<T> = new ArrayList<T>()

  // 加入购物车 添加商品
  public add(shop: T) {
    this.bill.add(shop)
  }

  // 顾客账单
  public show() {
    console.log(this.bill.show())
  }

  // 支付费用
  public cost(): number {
    let total = 0
    for (let i = 0; i < this.bill.size(); i++) {
      total +=
        (this.bill.get(i) as any).calculateRent() +
        (this.bill.get(i) as any).deposit
    }
    return total
  }
}

const cus = new Customer<Vechile>()
cus.add(car1)
cus.add(bus1)
cus.add(truck1)

cus.show()

console.log(cus.cost())
```



### typeof 和 keyof

```js
// typeof 使用
// 获取一个对象变量的类型
const obj = { name: string, age: 22 }
type objType = typeof obj // type typeofObj = { name: string; age: number; }

// keyof 使用
// 获取一个对象中的所有key组成的联合类型
type keyofObj = keyof objType // 'name' | 'age'
type keyofObj = keyof typeof obj // 'name' | 'age' 


// 综合应用
class ObjVal<T extends object, E extends keyof T> {
  constructor(public target: T, public key: E) {}

  // public getValue(): T[E] {
  //   return this.target[this.key]
  // }

  // public setValue(newVal: T[E]) {
  //   this.target[this.key] = newVal
  // }
  public get Value(): T[E] {
    return this.target[this.key]
  }

  public set Value(newVal: T[E]) {
    this.target[this.key] = newVal
  }
}

const obj1 = new ObjVal(obj, 'name')
obj1.Value = 'xx'
console.log(obj1.Value)
```

### 综合练习案例二

```js
/**
 * 综合案例
 * 排序
 *
 * 技术点：泛型函数 函数重载 类型守卫
 *
 * 1. 手写出泛型版本的快速排序，递归写法
 * 2. 写出中文排序
 * 3. 字符串自排序
 * 4. 中文+英文+数字 数组排序
 * 5. 中文+英文+数字数组+数组内部字符串自排序
 * 6. 字符串自排序 + 中文+英文、数字数组+数组内部字符串自排序
 */


// 快速排序 可以排序 数字 字母
function quickSort<T>(arr: Array<T>): Array<T> {
  if (arr.length < 2) return arr
  const middle = arr.splice((arr.length / 2) | 0, 1)[0]
  const leftArr: Array<T> = []
  const rightArr: Array<T> = []
  arr.forEach(item => {
    item > middle ? rightArr.push(item) : leftArr.push(item)
  })
  return [...quickSort(leftArr), middle, ...quickSort(rightArr)]
}

// 自定义守卫
const isString = (value: any): value is string => typeof value === 'string'
// 判断是否中文
const isChinese = (value: any): boolean => /[\u4e00-\u9fa5]+/g.test(value)

// 1.中文排序
function sortChinesse<T>(arr: Array<T>): Array<T> {
  return arr.sort((prev, next) => (prev as any).localeCompare(next, 'zh-CN'))
}

// 2. 字符串自排序
function strSelftSort(str: any): any {
  const _arr = [...str]
  return quickSort(_arr).join('')
}

// 3. 中文+英文+数字 数组排序
function sort<T>(arr: Array<T>): Array<T> {
  // 1. 判断是否为中文
  const isCh = arr.some(item => isChinese(item))
  return isCh ? sortChinesse(arr) : quickSort(arr)
}

// 4. 中文+英文+数字数组+数组内部字符串自排序
function sort1<T>(arr: Array<T>): Array<T> {
  // 1. 判断是否为中文
  const isCh = arr.some(item => isChinese(item))

  if (isCh) {
    return sortChinesse(arr)
  } else {
    return quickSort(arr).map(item => {
      if (isString(item)) {
        return strSelftSort(item)
      } else {
        return item
      }
    })
  }
}

// 5. 字符串自排序 + 中文+英文、数字数组+数组内部字符串自排序
function sort2(str: string): string
function sort2<T>(arr: Array<T>): Array<T>
function sort2<T>(strAndArr: string | Array<T>): any {
  if (isString(strAndArr)) {
    return strSelftSort(strAndArr)
  } else {
    return sort1(strAndArr)
  }
}


// 模拟数据
// 数字数据 用来保留两位小数
const numArr = [4, 6.34323, 2.23423, 7.34534, 2, 8.1, 19, 5, 0, 234]
// 中文排序
const chineseArr = ['武汉', '郑州', '太原', '济南', '沈阳', '大连']
// 字符串排序
const str: string = 'gksayopsmvr'
// 英文+自排序
const strArr: string[] = ['dfa', 'fwe', 'ser', 'sda', 'yuy', 'iot']
```

### object 和 Object的区别

```js
1. Object是所有类型的父类，相当于半个 any，而  object 只是对象数据类型的描述
2. object上没有任何方法，而Object自带很多熟悉和方法
```

## 高级类型

#### infer

```js
// infer表示在 extends 条件语句中以占位符出现的用来修饰数据类型的关键字，被修饰的数据类型等到使用时才能被推断出来

// infer作为占位符关键字出现的位置
1. infer出现在extends条件语句后的函数类型的参数类型位置上
2. infer出现在extends条件语句后的参数类型的返回值类型上
3. infer会出现在类型的泛型具体化类型上
```

## 装饰器

### 函数装饰器

### 属性装饰器

### 类装饰器

### 参数装饰器



