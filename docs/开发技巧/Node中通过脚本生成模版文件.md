> 当前案例完整的完整代码地址：[https://github.com/xinxin1228/docs_develop_tips/tree/main/04_Node%E4%B8%AD%E9%80%9A%E8%BF%87%E8%84%9A%E6%9C%AC%E7%94%9F%E6%88%90%E6%A8%A1%E7%89%88%E6%96%87%E4%BB%B6](https://github.com/xinxin1228/docs_develop_tips/tree/main/04_Node%E4%B8%AD%E9%80%9A%E8%BF%87%E8%84%9A%E6%9C%AC%E7%94%9F%E6%88%90%E6%A8%A1%E7%89%88%E6%96%87%E4%BB%B6)

## 应用场景

在日常开发中，我们经常会碰到的一种场景是：在固定的文件夹下创建固定的文件，然后彼此互相导入。

以我真实的日常开发为例：`Node`端采用的是三层架构来实现 `Restful API`，分为了 `Controller` 层，`Service` 层和 `Dao` 层。主要的文件结构如下：

```shell
src
├── controller
│   ├── UserController.js
│   └── index.js
├── dao
│   ├── UserDao.js
│   └── index.js
├── index.js
└── service
    ├── UserService.js
    └── index.js
```

比如创建一个用户的接口。

首先需要在 `controller`目录下新建文件 `UserController.js` ， 用来处理客户端的请求和返回响应。

然后在 `service` 目录下新建文件 `UserService.js`  ，用来负责业务逻辑的处理，处理数据、规则和流程的编排。

最后在 `dao` 目录下新建文件 `UserDao.js` ，负责与数据库进行交互。

然后先分别在各自目录下的 `index.js` 文件中导出，然后在  `UserController.js` 中导入 `UserService.js` ，在 `UserService.js` 中导入 `UserDao.js`。

如此可见，当有非常多的功能时，手动创建和导入是一件非常耗时和繁琐的过程。因此我们需要一个脚本来自动创建这些相关文件，以及互相导入依赖。

## 编写脚本

在项目的根目录新建 `scripts` 文件夹，并在里面创建 `create.js` 脚本文件。

```shell
$ mkdir scripts && touch scripts/create.js
```

配置 `package.json` 加入该运行脚本。

```json
// package.json
{
  // ...
  "scripts": {
    "create": "node scripts/create.js"
  },
}
```

这样我们可以通过以下命令来自动创建上面所说的文件，并且通过参数来控制是否覆盖或者是否使用 `typescript`。

```shell
$ npm run create user -- --force --typescript
```

- `user` ：要生成的模块名称，会自动生成 `UserController` 等。
- `--force` 或 `-f` ：如果有同名文件，强制覆盖替换。
- `--typescript` 或 `-t` ：生成 `ts` 模版文件。

### 1. 解析脚本参数

新建 `utils/parseProcessArgv.js` 文件来解析命令行参数。

```js
// utils/parseProcessArgv.js
import Log from './log.js'

class ParseProcessArgv {
  constructor() {
    this._argv = process.argv.slice(2)
    this.argvInfo = this._argv.reduce((prev, next) => {
      // 选项参数
      if (next.startsWith('-')) {
        prev.options ||= {}
        prev.options[next.replace(/^-+/g, '')] = true
      } else {
        prev.args ||= []
        prev.args.push(next)
      }

      return prev
    }, {})
  }
  get argv() {
    return this._argv
  }
  set argv(info) {
    Log.error('请忽修改process.argv！')
  }

  /**
   * 获取参数
   * @param {String|Number} param 获取指定索引的参数或查询参数是否存在
   */
  getParam(param) {
    if (typeof param === 'string') {
      return this.argvInfo.args?.includes(param)
    } else if (typeof param === 'number') {
      return this.argvInfo.args?.at(param)
    } else {
      Log.error('参数类型有误！')
    }
  }

  /**
   * 获取所有的参数
   */
  getParams() {
    return this.argvInfo.args
  }

  /**
   * 获取选项参数是否存在
   * @param {String} param 选项参数
   */
  getOptionsParam(param) {
    return this.argvInfo.options?.[param]
  }

  /**
   * 获取所有的选项参数
   */
  getOptionsParams() {
    return this.argvInfo.options
  }
}

export const parseProcessArgv = new ParseProcessArgv()
```

### 2. 创建模版

在 `scripts` 目录下创建 `template` 文件夹，用来存放不同模版的文件。目录如下：

```shell
scripts
└── template
    ├── Controller.js.tmpl
    ├── Controller.ts.tmpl
    ├── Dao.js.tmpl
    ├── Dao.ts.tmpl
    ├── Service.js.tmpl
    └── Service.ts.tmpl
```

文件内容如下：

<!-- tabs:start -->

#### **Controller.js.tmpl**

```js
import { <<%= name %>>Service } from '../service/index.<<%= lang %>>'

export default class <%= name %>>Controller {
  static <<%= name %>>Service = new <<%= name %>>Service()

  hi() {
    const result = <<%= name %>>Controller.<<%= name %>>Service.hi()
    console.log(result)
  }
}
```

#### **Controller.ts.tmpl**

```ts
import { <<%= name %>>Service } from '../service/index.<<%= lang %>>'

export default class <<%= name %>>Controller {
  static <<%= name %>>Service: <<%= name %>>Service = new <<%= name %>>Service()

  hi() {
    const result = <<%= name %>>Controller.<<%= name %>>Service.hi()
    console.log(result)
  }
}

```

#### **Service.js.tmpl**

```js
import { <<%= name %>>Dao } from '../dao/index.<<%= lang %>>'

export default class <<%= name %>>Service {
  static <<%= name %>>Dao = new <<%= name %>>Dao()

  hi() {
    return <<%= name %>>Service.<<%= name %>>Dao.hi()
  }
}

```

#### **Service.ts.tmpl**

```ts
import { <<%= name %>>Dao } from '../dao/index.<<%= lang %>>'

export default class <<%= name %>>Service {
  static <<%= name %>>Dao: <<%= name %>>Dao = new <<%= name %>>Dao()

  hi(): string {
    return <<%= name %>>Service.<<%= name %>>Dao.hi()
  }
}

```

#### **Dao.js.tmpl**

```js
export default class <<%= name %>>Dao {
  hi() {
    return 'hi'
  }
}

```

#### **Dao.ts.tmpl**

```ts
export default class <<%= name %>>Dao {
  hi(): string {
    return 'hi'
  }
}

```

<!-- tabs:end -->

编写一个替换模版占位符的函数。详细思路为：

1. 先根据不同的类型拼接出模版的地址。
2. 查看模版地址是否存在，不存在提示，存在则读取文件拿到字符串。
3. 根据传入的数据来替换字符串中的占位符，并返回最终的字符串。

```js
/**
 * 根据模版生成文件内容字符串
 * @param {String} type 模版类型 Controller | Service | Dao
 * @param {Object} data 模版数据
 * @returns
 */
function generatedContent(type, data) {
  if (!data || typeof data !== 'object') {
    Log.error('data类型必须是对象！')
    process.exit(1)
  }
  const { lang = 'js' } = data

  // 1. 先根据不同的类型拼接出模版的地址。
  const templatePath = resolve(__dirname, `./template/${type}.${lang}.tmpl`)

  // 2. 查看模版地址是否存在，不存在提示，存在则读取文件拿到字符串。
  const isExit = existsSync(templatePath)
  if (!isExit)
    Log.error(`模版文件 /template/${type}.${lang}.tmpl 不存在，请先创建！`)
  const content = readFileSync(templatePath, 'utf-8')

  // 3. 根据传入的数据来替换字符串中的占位符，并返回最终的字符串。
  return content.replace(/<<%=\s*(\w+)\s*%>>/g, (match, p1) => data[p1] || match)
}

```

### 3. 新增文件

最后，我们根据命令行传入的参数来创建对应的文件以及修改导入导出。详细思路为：

1. 先根据前面写的[解析脚本参数](#_1-解析脚本参数)的获取的要创建模块的名称和选项参数。
2. 判断是否有 `-f, --force` 强制覆盖参数，如果有的话直接创建文件，反之则判断文件是否已存在。

```js
function init() {
  let name = parseProcessArgv.getParam(0)

  if (!name) Log.error('请输入要添加的模版名称！')
  name = name.split(/_|-/).map(startStrToUpperCase).join('')
  const lang = parseProcessArgv.getOptionsParam(['t', 'typescript'])
    ? 'ts'
    : 'js'
  const fileName = `${name}$.${lang}`

  const CONTROLLER_URL = resolve(__dirname, '../src/controller')
  const SERVICE_URL = resolve(__dirname, '../src/service')
  const DAO_URL = resolve(__dirname, '../src/dao')

  // 路径映射
  const path_list = [
    {
      type: FILETYPE.CONTROLLER, // 文件类型
      path: resolve(CONTROLLER_URL, getFileName(fileName, FILETYPE.CONTROLLER)), // 文件路径
      entry: resolve(CONTROLLER_URL, `index.${lang}`), // 入口文件
    },
    {
      type: FILETYPE.SERVICE,
      path: resolve(SERVICE_URL, getFileName(fileName, FILETYPE.SERVICE)),
      entry: resolve(SERVICE_URL, `index.${lang}`), // 入口文件
    },
    {
      type: FILETYPE.DAO,
      path: resolve(DAO_URL, getFileName(fileName, FILETYPE.DAO)),
      entry: resolve(DAO_URL, `index.${lang}`), // 入口文件
    },
  ]

  // 非强制覆盖，检查是否一个某个文件已经存在
  const existFiles = path_list.filter(({ path }) => {
    return existsSync(path)
  })
  if (!parseProcessArgv.getOptionsParam(['f', 'force'])) {
    if (existFiles.length) {
      existFiles.forEach(({ type }) => {
        Log.error(
          `文件 ${type.toLowerCase()}/${getFileName(
            fileName,
            type
          )} 已经存在！`,
          false
        )
      })
      process.exit(1)
    }
  }

  // 批量创建
  path_list.forEach(({ type, path, entry }) => {
    // 创建文件
    writeFileSync(path, generatedContent(type, { name, lang }), {
      flag: 'w',
    })
    if (existFiles.find(n => n.type === type)) {
      Log.success(
        `文件 ${type.toLowerCase()}/${getFileName(fileName, type)} 覆写成功！`
      )
    } else {
      Log.success(
        `文件 ${type.toLowerCase()}/${getFileName(fileName, type)} 创建成功！`
      )
    }

    // 处理导入导出
    writeFileSync(
      entry,
      `\nexport { default as ${name}${type} } from './${getFileName(
        fileName,
        type
      )}'`,
      { flag: 'a+' }
    )
  })

  process.exit(0)
}
```

