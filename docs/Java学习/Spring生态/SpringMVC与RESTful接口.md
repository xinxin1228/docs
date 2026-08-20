# Spring MVC 与 RESTful 接口

## 概述

> Spring MVC 是 Spring Boot Web 的核心,负责把 HTTP 请求映射到处理器并返回响应。本笔记覆盖请求处理流程、常用注解、参数绑定、响应封装与 RESTful 设计规范。

## 核心知识点

### 请求处理流程

```
客户端请求
  → DispatcherServlet(前端控制器,统一入口)
  → HandlerMapping(找到对应 Controller 方法)
  → HandlerAdapter(执行方法)
  → 返回 ModelAndView / 响应体
  → 视图渲染 / 序列化为 JSON
```

我们只需关注 Controller 层,其余由框架完成。

### 常用注解

```java
@RestController                  // = @Controller + @ResponseBody,每个方法返回 JSON
@RequestMapping("/users")        // 类级路径前缀
public class UserController {

    @GetMapping("/{id}")                       // GET 查询单个
    public User getUser(@PathVariable Long id) { ... }

    @PostMapping                              // POST 创建
    public User create(@RequestBody UserDto dto) { ... }

    @PutMapping("/{id}")                      // PUT 全量更新
    public User update(@PathVariable Long id, @RequestBody UserDto dto) { ... }

    @DeleteMapping("/{id}")                   // DELETE 删除
    public void delete(@PathVariable Long id) { ... }
}
```

参数绑定:

| 注解 | 来源 |
|------|------|
| `@PathVariable` | URL 路径 `/users/{id}` |
| `@RequestParam` | 查询参数 `?page=1` |
| `@RequestBody` | 请求体 JSON(自动反序列化) |
| `@RequestHeader` | 请求头 |
| `@RequestParam` 绑定对象 | `?name=&age=` 自动绑定到 POJO |

### 响应与状态码

```java
@GetMapping("/{id}")
public ResponseEntity<User> get(@PathVariable Long id) {
    User u = userService.findById(id);
    return ResponseEntity.ok(u);               // 200
    // return ResponseEntity.status(404).body(null);  // 自定义状态码
    // return ResponseEntity.created(uri).body(u);    // 201 + Location 头
}
```

`@RestController` 返回的对象由 Jackson 序列化为 JSON;`null` 字段默认序列化(可配 `@JsonInclude(NON_NULL)` 忽略)。

### RESTful 设计规范

| 约定 | 示例 |
|------|------|
| 用名词复数表示资源 | `/users`、`/orders` |
| HTTP 方法表达动作 | GET 查 / POST 增 / PUT 改 / DELETE 删 |
| 层级资源 | `/users/{id}/orders` |
| 状态码语义化 | 200 成功 / 201 创建 / 400 参数错 / 401 未认证 / 403 无权限 / 404 不存在 / 500 服务错 |
| 统一响应结构 | `{ code, message, data }` |

### 过滤器和拦截器

- **Filter**:Servlet 规范,容器级,可处理请求/响应(编码、CORS、鉴权)
- **HandlerInterceptor**:Spring 层,能拿到 Handler 信息,做权限、日志

```java
// 拦截器实现
public class AuthInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse resp, Object handler) {
        String token = req.getHeader("Authorization");
        if (token == null) {
            resp.setStatus(401);
            return false;    // 中断请求
        }
        return true;
    }
}
```

## 示例

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;
    public UserController(UserService userService) { this.userService = userService; }

    @GetMapping
    public Result<List<User>> list(
            @RequestParam(defaultValue = "1") int page,
            @RequestParam(defaultValue = "20") int size) {
        return Result.ok(userService.page(page, size));
    }

    @PostMapping
    public Result<User> create(@Valid @RequestBody UserDto dto) {
        return Result.ok(userService.create(dto));
    }
}
```

## 常见问题与坑

- **`@RestController` 返回字符串却想跳转页面**:`@RestController` 永远返回响应体,页面跳转用 `@Controller` + 模板
- **GET 请求带 `@RequestBody`**:GET 一般用查询参数,别用 body(部分客户端/网关不支持)
- **JSON 反序列化失败**:字段类型不匹配、缺无参构造器、`@JsonProperty` 映射错
- **路径参数类型不匹配**:`/users/{id}` 传 `abc` 到 `Long`,抛 `MethodArgumentTypeMismatchException`
- **CORS 跨域**:前后端分离要配 CORS(`@CrossOrigin` 或全局 CorsFilter)
- **状态码乱用**:错误也返回 200,前端无法区分成败,统一异常处理配合语义化状态码(见下一篇)

## 参考资料

- [Spring MVC 文档](https://docs.spring.io/spring-framework/reference/web/webmvc.html)
- [RESTful API 设计最佳实践](https://restfulapi.net/)
