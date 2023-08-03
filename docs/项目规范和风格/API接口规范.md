## RESTful 风格 API 设计指南

> restful 是一种软件架构风格、设计风格，而不是标准，只是提供了一组设计原则和约束条件。它主要用于客户端和服务器交互类的软件。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。 ——百度百科

### 协议

客户端在通过 `API` 与后端服务通信的过程中，`应该` 使用 `HTTPS` 协议。

### 域名

应该尽量将 API 部署在专用域名之下。

> https://api.example.com

如果确定 API 很简单，不会有进一步扩展，可以考虑放在主域名下。

> https://example.org/api/

### 版本

应该将 API 的版本号放入 URL。

> https://api.example.com/v1/
>
> https://example.com/api/v1/

另一种做法是，将版本号放在 HTTP 头信息中，但不如放入 URL 方便和直观。[Github](https://developer.github.com/v3/media/#request-specific-version)采用这种做法。

### HTTP 动词

对于资源的具体操作类型，由 `HTTP` 动词表示。常用的 `HTTP` 动词有下面五个（括号里是对应的 `SQL` 命令）。

- GET（SELECT）：从服务器取出资源（一项或多项）。
- POST（CREATE）：在服务器新建一个资源。
- PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
- PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。
- DELETE（DELETE）：从服务器删除资源。

其中

1 删除资源 `必须` 用 `DELETE` 方法

2 创建新的资源 `必须` 使用 `POST` 方法

3 更新资源 `应该` 使用 `PUT` 方法

4 获取资源信息 `必须` 使用 `GET` 方法

针对每一个端点来说，下面列出所有可行的 `HTTP` 动词和端点的组合

| 请求方法 | URL                              | 描述                                              |
| -------- | -------------------------------- | ------------------------------------------------- |
| GET      | /zoos                            | 列出所有的动物园(ID 和名称，不要太详细)           |
| POST     | /zoos                            | 新增一个新的动物园                                |
| GET      | /zoos/{zoo}                      | 获取指定动物园详情                                |
| PUT      | /zoos/{zoo}                      | 更新指定动物园                                    |
| DELETE   | /zoos/{zoo}                      | 删除指定动物园                                    |
| GET      | /zoos/{zoo}/animals              | 检索指定动物园下的动物列表(ID 和名称，不要太详细) |
| GET      | /animals                         | 列出所有动物(ID 和名称)。                         |
| POST     | /animals                         | 新增新的动物                                      |
| GET      | /animals/{animal}                | 获取指定的动物详情                                |
| PUT      | /animals/{animal}                | 更新指定的动物                                    |
| GET      | /animal_types                    | 获取所有动物类型(ID 和名称，不要太详细)           |
| GET      | /animal_types/{type}             | 获取指定的动物类型详情                            |
| GET      | /employees                       | 检索整个雇员列表                                  |
| GET      | /employees/{employee}            | 检索指定特定的员工                                |
| GET      | /zoos/{zoo}/employees            | 检索在这个动物园工作的雇员的名单(身份证和姓名)    |
| POST     | /employees                       | 新增指定新员工                                    |
| POST     | /zoos/{zoo}/employees            | 在特定的动物园雇佣一名员工                        |
| DELETE   | /zoos/{zoo}/employees/{employee} | 从某个动物园解雇一名员工                          |

### 过滤

> 如果记录数量很多，服务器不可能都将它们返回给用户。API `应该` 提供参数，过滤返回结果。下面是一些常见的参数。

- ?limit=10：指定返回记录的数量，返回 10 条数据

- ?offset=10：指定返回记录的开始位置，从第 10 条数据的位置返回

- ?limit=10&offset=10：制定返回的记录和起始位置，从第 10 条数据的位置返回 10 条数据

### 鉴权

`应该` 使用 `OAuth2.0` 的方式为 API 调用者提供登录认证。`必须` 先通过登录接口获取 `Access Token` 后再通过该 `token` 调用需要身份认证的 `API`。

客户端在请求需要认证的 `API` 时，`必须` 在请求头 `Authorization` 中带上 `access_token`。

> Authorization: Bearer token...

### 响应

响应格式必须是`JSON` 格式，下面以最常见的用户请求为例：

#### 用户登录成功（200）

```js
HTTP/1.1 200 ok
Content-Type: application/json
Server: example.com

{
    mes: "登录成功！",
  	token: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6Nywibmlja25hbWUiOiJodWFuZ2ppYW54aW4iLCJuYW1lIjoi6buE5bu65pawIiwicGFzcyI6ImQzNjc3OWNkOTc3NmQ5MjM5YzEzMzlkODUzMThmMGE0Zjc1YmJkMWEiLCJyb2xlIjo1LCJzdGF0dXMiOjEsImNyZWF0ZV90aW1lIjoiMjAyMi0xMS0xNlQxNDoyMjo1Ni4wMDBaIiwidXBkYXRlX3RpbWUiOiIyMDIyLTExLTE2VDE0OjIyOjU2LjAwMFoiLCJleHAiOjE2OTE2NzQ4NzAsImlhdCI6MTY5MTA3MDA3MH0.m3ezoXAoWMLlrynSDQoYbhtv-0_VqmsKLrwSW8UNfz0",
  userInfo: {
    id: 1,
    name: '黄建新',
    profession: "前端全栈开发工程师"
  }
}
```

#### 添加新用户成功（201）

```js
HTTP/1.1 201 ok
Content-Type: application/json
Server: example.com

{
    mes: "添加成功！",
}
```

#### 删除用户成功（204）

```js
HTTP/1.1 204 ok
Content-Type: application/json
Server: example.com

{
    mes: "删除成功！",
}
```

#### 用户登录失败（400）

```js
HTTP/1.1 400 Bad Request
Server: nginx/1.11.9
Content-Type: application/json
Transfer-Encoding: chunked
Cache-Control: no-cache, private
Connection: keep-alive

{
    error: "用户名或密码错误！",
}
```

#### 用户未登录（401）

```js
HTTP/1.1 401 Unauthorized
Server: nginx/1.11.9
Content-Type: application/json
Transfer-Encoding: chunked
WWW-Authenticate: JWTAuth
Cache-Control: no-cache, private
Connection: keep-alive

{
    error: "请先登录！",
}
```

#### 用户无权限（403）

```js
HTTP/1.1 403 Forbidden
Server: nginx/1.11.9
Content-Type: application/json
Transfer-Encoding: chunked
Cache-Control: no-cache, private
Connection: keep-alive

{
    error: "您没有该处理权限！",
}
```

#### 用户不存在（404）

```js
HTTP/1.1 404 Bad Request
Server: nginx/1.11.9
Content-Type: application/json
Transfer-Encoding: chunked
Cache-Control: no-cache, private
Date: Sun, 24 Jun 2018 13:22:36 GMT
Connection: keep-alive

{
    error: "该用户不存在或已被删除！",
}
```

## 状态码

### 2xx

#### 200

所代表的含义：请求成功，更新成功，替换成功

#### 201

新增成功，添加成功

#### 204

删除成功

### 4xx

#### 400

- 用户传输的数据校验不通过，比如缺少某一个必传参数
- 添加新用户时，用户填写的邮箱或用户已存在

#### 401

- `Token`验证失败，用户`未登录`情况下请求一些需要登录才能看的数据
- 用户登录时，用户账号或密码错误

#### 403

用户`已登录`，但是不具有某一权限

#### 404

请求的内容不存在，比如请求、修改或删除的数据`不存在`或`删除不可见`

### 5xx

#### 500

服务器异常，通常是后端服务代码逻辑出现问题时，直接抛出该状态码

## 其他规范

- 统一以 /api 为请求前缀

- URI 结尾不应包含（/）

- 正斜杠分隔符（/）必须用来指示层级关系

- 应使用连字符（-）来提高 URI 的可读性，不得在 URI 中使用下划线（\_）

  > /api/users/workflow-action

- URI 路径中全都使用小写字母， 但是一个请求地址含有多个 params 参数时间，参数格式为 请求信息+ID

  > 比如：/api/users/:userID/shops/:shopID

## 配合 Axios 使用

### 请求拦截

```js
// 添加请求拦截器
axios.interceptors.request.use((config: AxiosRequestConfig) => {
  const token: string | null = cache.getItem('token-admin')
  if (!config.headers) return
  // 添加token
  config.headers.Authorization = 'Bearer admin' + token
  return config
})
```

### 响应拦截

```js
// 添加响应拦截器
axios.interceptors.response.use(
  (response: AxiosResponse) => {
    // 对响应数据做点什么

    return response
  },
  (err: any) => {
    // 对响应错误做点什么
    // 关闭loading加载动画

    if (err.message === 'Network Error') message.warning('网络连接异常！')
    if (err.code === 'ECONNABORTED') message.warning('请求超时，请重试')
    message.warning(err.response.data.error)
    return Promise.reject(err)
  }
)
```

## 具体写法

> 下面以`Axios`和`Node`为例，实现对商品的增删改查

```js
// 增 post
axios.post('/goods', {
  id: 10,
})
// node
const { id } = req.body

// 删 delete
axios.delete('/goods', {
  data: {
    id: 20,
  },
})
// node
const { id } = req.body

// 改 put
axios.put('/goods', {
  id: 30,
})
// node
const { id } = req.body

// 查 get
axios.get('/goods', {
  params: {
    id: 40,
  },
})
// node
const { id } = req.query
```
