# 从零搭建 SpringBoot 项目

## 概述

> 从零搭一个可运行的 Spring Boot 后端工程:创建项目 → 目录结构 → 依赖 → 配置 → 接 MySQL + Flyway + JPA → 跑通第一个接口。这是一条完整的最小闭环,后续所有功能都在这套骨架上扩展。

## 核心知识点

### 1. 创建项目

方式一:[start.spring.io](https://start.spring.io) 网页勾选(推荐,直观)。选择:

- Project: Maven
- Language: Java
- Spring Boot: 3.x(最新稳定版)
- Group: `com.example`,Artifact: `demo`
- 依赖:Spring Web、Spring Data JPA、Validation、MySQL Driver、Flyway、Lombok(可选)

方式二:IDEA 新建 → Spring Initializr(与网页等价)。

下载解压后用 IDEA 打开,或命令行生成:

```bash
curl https://start.spring.io/starter.zip \
  -d dependencies=web,data-jpa,validation,mysql,flyway \
  -d javaVersion=17 -d groupId=com.example -d artifactId=demo \
  -o demo.zip
```

### 2. 目录结构

```
demo/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/com/example/demo/
    │   │   ├── DemoApplication.java      # 启动类(包根)
    │   │   ├── controller/               # 控制层:接收请求
    │   │   ├── service/                  # 业务层:业务逻辑
    │   │   │   └── impl/
    │   │   ├── repository/               # 数据访问层:DAO
    │   │   ├── entity/                   # 实体类
    │   │   ├── dto/                      # 传输对象
    │   │   ├── config/                   # 配置类
    │   │   ├── common/                   # 通用:Result、异常
    │   │   └── exception/                # 全局异常处理
    │   └── resources/
    │       ├── application.yml
    │       └── db/migration/             # Flyway SQL
    └── test/java/com/example/demo/
```

分层约定:Controller → Service → Repository,单向依赖,不越层调用。

### 3. 依赖(pom.xml 关键片段)

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>
</parent>

<properties>
    <java.version>17</java.version>
</properties>

<dependencies>
    <dependency><groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId></dependency>
    <dependency><groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId></dependency>
    <dependency><groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId></dependency>
    <dependency><groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId><scope>runtime</scope></dependency>
    <dependency><groupId>org.flywaydb</groupId>
        <artifactId>flyway-mysql</artifactId></dependency>
</dependencies>
```

### 4. 配置 application.yml

```yaml
server:
  port: 8080

spring:
  application:
    name: demo
  datasource:
    url: jdbc:mysql://localhost:3306/demo?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    username: ${DB_USERNAME:root}
    password: ${DB_PASSWORD:123456}
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: validate          # 交给 Flyway 管表结构,JPA 只校验不建表
    open-in-view: false           # 关闭 OSIV,避免长事务
  flyway:
    enabled: true
    baseline-on-migrate: true
```

### 5. 建表(Flyway)

`src/main/resources/db/migration/V1__init_schema.sql`:

```sql
CREATE TABLE t_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100),
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### 6. 实体 + Repository + 接口

```java
@Entity
@Table(name = "t_user")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String email;
    // getter/setter
}

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}

@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserRepository repo;
    public UserController(UserRepository repo) { this.repo = repo; }

    @GetMapping
    public List<User> list() { return repo.findAll(); }
}
```

### 7. 启动与验证

```bash
# 启动(启动时自动执行 Flyway 建表)
mvn spring-boot:run

# 验证接口
curl http://localhost:8080/api/users
# 健康检查(需引 actuator)
curl http://localhost:8080/actuator/health
```

## 示例

完整跑通的最小代码已在上面给出,按 1→7 顺序操作即可得到「启动即建表 + 可访问接口」的后端骨架。

## 常见问题与坑

- **启动报 `Failed to configure a DataSource`**:没配数据源 url/驱动,或 MySQL 没启动
- **`ddl-auto` 与 Flyway 冲突**:不要用 `ddl-auto: update/create`,让 Flyway 全权管表结构,否则两套机制打架
- **Flyway 报数据库不支持**:MySQL 8 漏了 `flyway-mysql` 依赖
- **实体字段映射不上**:列名 `created_at` vs 字段 `createdAt`,开启驼峰映射或写 `@Column`
- **启动类位置不对**:`@SpringBootApplication` 要在包根,否则扫描不到 controller
- **端口占用**:`8080` 被占,改 `server.port`

## 参考资料

- [Spring Initializr](https://start.spring.io/)
- [Spring Boot 快速开始](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html)
