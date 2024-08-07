> 本案例完整代码地址：https://github.com/xinxin1228/docs_develop_tips/tree/main/03_%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F

## 什么是环境变量？

这里指的环境环境变量并非是系统层面中那个环境变量，而是指项目开发和运行时所需的环境变量。

比如我们经常在工程化项目中可以通过 `process.env.NODE_ENV` 来区分当前项目运行环境（开发，测试，生产等），以便我们进行相应的项目配置，比如端口的切换、api 请求地址的切换等。

那为什么`process.env.NODE_ENV` 能用来区分环境呢？它是如何来的？

- `process` 对象是一个 `global`（全局变量），提供有关信息，控制当前 Node.js 进程。作为一个对象，它对于 `Node.js` 应用程序始终是可用的，故无需使用 require()。
  
- `process.env` 属性返回一个包含用户环境信息的对象。

在 `node` 环境中，当我们打印 `process.env` 时，发现它并没有 `NODE_ENV `这一个属性。实际上 `process.env.NODE_ENV` 是在 `package.json` 的 `scripts` 命令中注入的，也就是 `NODE_ENV` 并不是 `node` 自带的，而是由用户定义的，至于为什么叫 `NODE_ENV`，应该是约定俗成的吧。

在不同操作系统中设置环境变量的语法是有差异的，例如在 `Unix-like` 系统中（比如 Mac）通常使用 `export` 命令，而在 `Windows` 中则是使用 `set` 命令。

<!-- tabs:start -->

#### **Window下的设置**

```json
{
  // ...
  "scripts": {
    "dev": "set NODE_ENV=development && node src/index.js"
  }
}
```

#### **Mac下的设置**

```json
{
  // ...
  "scripts": {
    "dev": "export NODE_ENV=development node src/index.js" // 这里不是手误 是类Unix-like系统下不需要 &&
  }
}
```



<!-- tabs:end -->

> 在Mac上设置环境变量不需要使用 `&&` 的原因是因为Mac和Linux系统中的命令是按顺序执行的，不像Windows系统中需要使用 `&&` 来连接多个命令。Mac 中的 `&&`  的含义是，前一个命令执行成功了才会执行后一个命令。

这时我们想到，可以设置不同的 `npm scripts` 来在不同的平台进行运行。

```json
{
  // ...
  "scripts": {
    "dev:win": "set NODE_ENV=development && node src/index.js",
    "dev:mac": "export NODE_ENV=development node src/index.js"
  }
}
```

- 在 `Window` 平台下执行：`pnpm dev:win`

- 在 `Mac` 平台下执行：`pnpm dev:mac`

运行的命令数量少的时候还好说，一旦运行的命令数量多，那将是一坨。因此这个时候请出 `corss-env`。

## cross-env

`cross-env` 是一个 Node.js 工具包，主要用于在不同操作系统（如Windows、Linux和macOS）上设置和使用环境变量，尤其是当你需要在命令行中设置环境变量并且期望跨平台兼容时。

`cross-env` 的作用在于提供了一个统一的接口，使得无论在哪种操作系统上运行命令，都能正确设置环境变量。这样，开发者就不需要关心具体的操作系统差异，只需按照一种方式编写命令，`cross-env` 会负责在后台转换为对应的系统命令。

安装：

```shell
$ pnpm add cross-env
```

然后我们可以将上面的两种命令整合成一条语句。

```json
// package.json
{
  "scripts": {
    "dev": "cross-env NODE_ENV=development nodemon src/index.js"
  }
}
```

> 注意这里 设置变量 与 运行语句 之间，没有 `&&`。

在`cross-env`中，可以一次性设置多个环境变量。

```json
// package.json
{
  "scripts": {
    "dev": "cross-env NODE_ENV=development PORT=5231 nodemon src/index.js"
  }
}
```

在这段脚本中，`cross-env` 会设置两个环境变量：

1. `NODE_ENV` 被设置为 `development`
2. `PORT` 被设置为 `5231`



## dotenv

如果需要配置的环境变量太多，全部设置在`scripts`命令中既不美观也不容易维护，此时将环境变量配置在`.env`文件中，然后使用`dotenv`插件来加载`.env`配置文件。

安装：

```shell
$ pnpm add dotenv -D
```

在项目的根目录上创建 `.env` 文件：

```shell
NODE_ENV=development
PORT=3000
API_URL=http://localhost:5000/api
```

> 注意：这里 等于（=）两边有无空格都可以，但是为了和 `linux` 下设置环境变量保持一致，因此都不保留空格。也没有必要用引号来包裹字符串。`dotenv` 会自动为你做这个。

尽早的在项目入口文件中引入和配置 `dotenv`，在 config 函数中可以配置 `.env` 文件的路径。默认情况下会加载项目根目录的 `.env` 文件。

```js
// src/index.js
const path = require('node:path')

// 默认使用 会加载根目录的 .env
require('dotenv').config()

// 指定加载 如果找不到指定文件，不会报错，因此可以放心在生产环境中使用
require('dotenv').config({ path: path.resolve(__dirname, '../env/.env.pord')})
```

这样就可以在程序中使用环境变量了。

需要注意的是，一旦你创建了这个文件，请记住，你不应该把它推送到远程仓库（比如 GitHub 和 GitLab 等），因为它可能包含敏感数据，如认证密钥和密码。将该文件添加到 `.gitignore` 中，以避免不小心将其推送到公共仓库。

```shell
# .gitignore
.env*
```

在生产环境中设置和使用环境变量通常不推荐使用 `dotenv` 这类工具（这也是我为什么把它装入开发依赖的原因），而是直接在服务器或云服务提供商提供的环境中设置环境变量。例如：

###  Linux服务器设置环境变量

在Linux服务器上，通常通过修改系统的环境变量文件或在启动脚本中设置环境变量。例如，在 `bash shell` 中，可以编辑`/etc/environment`文件，或者在启动脚本（如`/etc/init.d/myapp`）中使用`export`命令设置：

```bash
export API_URL=postgres://user:pass@host:port/dbname
```

### Docker容器设置环境变量

在Dockerfile中通过`ENV`指令设置环境变量，或者在运行容器时通过`-e`或`--env`选项传入：

```dockerfile
# 在Dockerfile中设置
ENV API_URL=postgres://user:pass@host:port/dbname

# 运行容器时设置
docker run -d -e API_URL=postgres://user:pass@host:port/dbname my_image
```

## cross-env 和 dotenv 结合使用

在实际项目中，由于项目运行的环境较多，比如开发、测试、生成等，我们需要为不同的环境编写不同的环境变量。我们会在 `npm scripts` 中通过 `cross-env` 设置 `NODE_ENV`，然后根据不同的 `NODE_ENV` 来加载不同的 `.env` 文件。

简单的目录如下：

```shell
├── env
   ├── .env.dev
   ├── .env.test
   └── .env.pord
```

在不同的 `.env.*` 编写不同的环境变量

<!-- tabs:start -->

#### **.env.dev**

```shell
PORT=3000
TITLE=开发环境dev
```



#### **.env.test**

```shell
PORT=8080
TITLE=测试环境test
```



#### **.env.pord**

```shell
PORT=443
TITLE=生产环境pord
```



<!-- tabs:end -->

然后我们编写不同的 `npm scripts`：

```json
// package.json
{
  "scripts": {
    "start": "node src/index.js",
    "start:dev": "cross-env NODE_ENV=dev npm start",
    "start:test": "cross-env NODE_ENV=test npm start",
    "start:pord": "cross-env NODE_ENV=pord npm start"
  }
}
```

最后在入口文件中根据不同的 `process.env.NODE_ENV` 来切换不同 的 `.env.*` 文件。

```js
// src/index.js
const path = require('noed:path')

require('dotenv').config({
  path: path.resolve(__dirname, `../env/.env.${process.env.NODE_ENV}`),
})

console.log('NODE_ENV', process.env.NODE_ENV)
console.log('PORT', process.env.PORT)
console.log('TITLE', process.env.TITLE)
```



## 总结

- 一般来说，如果需要设置的不同环境下的变量少（只有一个或两个），那么使用**单独** `cross-env` 无疑是合适的。
- 如果只在开发环境下设置多个变量，那么**单独**使用 `dotenv` 的方法也很不错。
- 如果运行的环境和变量都很多的话，那么采用 `cress-env` 和 `dotenv` 组合的方式最合适。

