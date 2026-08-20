# MySQL 与 Java 集成

## 概述

> Java 操作 MySQL 的分层:JDBC(底层)→ 连接池(HikariCP)→ ORM(JPA / MyBatis)。本笔记聚焦 JDBC 基础、SQL 注入防护、连接池与事务,ORM 细节见《Spring Data JPA》《MyBatis》两篇。

## 核心知识点

### JDBC 基础

JDBC 是 Java 访问数据库的标准 API,五步:加载驱动 → 建连接 → 执行 SQL → 处理结果 → 释放资源。

```java
// 纯 JDBC(了解即可,实际都用连接池 + ORM)
String url = "jdbc:mysql://localhost:3306/demo?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai";
try (Connection conn = DriverManager.getConnection(url, "root", "123456");
     PreparedStatement ps = conn.prepareStatement("select * from t_user where id = ?")) {
    ps.setLong(1, 1L);
    try (ResultSet rs = ps.executeQuery()) {
        while (rs.next()) {
            System.out.println(rs.getString("username"));
        }
    }
}
```

### PreparedStatement 与 SQL 注入防护

```java
// ❌ Statement 字符串拼接:SQL 注入漏洞
String sql = "select * from t_user where username = '" + input + "'";
// 输入 ' OR '1'='1 会查出全部数据

// ✅ PreparedStatement:参数化,驱动自动转义
PreparedStatement ps = conn.prepareStatement(
    "select * from t_user where username = ?");
ps.setString(1, input);
```

**规则:所有用户输入都必须通过 `?` 占位符绑定,绝不字符串拼接 SQL。** 在 ORM 层面,MyBatis 的 `#{}`、JPA 的 `@Query` 参数都是参数化的。

### 连接池 HikariCP

创建数据库连接开销大,连接池复用连接。Spring Boot 默认集成 **HikariCP**(性能最好):

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/demo?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    driver-class-name: com.mysql.cj.jdbc.Driver   # MySQL 8 驱动
    hikari:
      maximum-pool-size: 10         # 最大连接数
      minimum-idle: 5               # 最小空闲
      connection-timeout: 30000     # 获取连接超时(ms)
      max-lifetime: 1800000
```

连接池配置要点:连接数别拍脑袋设大,过大反而压垮数据库,一般 `核数 * 2 + 磁盘数` 起步再压测调优。

### 事务

```java
@Transactional
public void transfer(Long from, Long to, BigDecimal amount) {
    accountMapper.decrease(from, amount);
    // 若这里抛异常,前面的操作回滚
    accountMapper.increase(to, amount);
}
```

事务要点:

- `@Transactional` 默认**运行时异常回滚**(`RuntimeException`),受检异常默认不回滚,可用 `rollbackFor` 指定
- **自调用不生效**:同类内 `this.method()` 绕过代理,事务不生效(同 AOP 自调用问题)
- 事务要**短小**,别在事务里做远程调用/发消息
- 隔离级别、传播行为按需配置:`@Transactional(isolation = ..., propagation = ...)`

### ORM 选型

- **JPA(Hibernate)**:自动建表、单表 CRUD 快,适合模型驱动
- **MyBatis / MyBatis-Plus**:手写 SQL 灵活,适合复杂查询

详见对应两篇笔记。

## 示例

```java
// Spring Boot + JdbcTemplate(轻量场景,不引入完整 ORM)
@Repository
public class UserDao {
    private final JdbcTemplate jdbc;
    public UserDao(JdbcTemplate jdbc) { this.jdbc = jdbc; }

    public User findById(Long id) {
        return jdbc.queryForObject(
            "select id, username, email from t_user where id = ?",
            (rs, rowNum) -> new User(rs.getLong("id"),
                                     rs.getString("username"),
                                     rs.getString("email")),
            id);
    }
}
```

## 常见问题与坑

- **`serverTimezone` 报错**:MySQL 8 驱动必须带 `serverTimezone`,否则抛时区异常
- **中文乱码**:URL 加 `characterEncoding=utf8`,表/库字符集用 `utf8mb4`(支持 emoji)
- **SQL 注入**:唯一防线是参数化,别信任何「过滤字符串」的土办法
- **连接池耗尽**:连接没释放(忘了关/事务过长),`connection-timeout` 后大量报错,排查连接泄漏
- **事务不回滚**:方法内 `try-catch` 吞了异常,或用了受检异常没配 `rollbackFor`
- **大数据量 `selectList`**:没分页,内存爆掉

## 参考资料

- [JDBC 教程](https://docs.oracle.com/javase/tutorial/jdbc/)
- [HikariCP](https://github.com/brettwooldridge/HikariCP)
- [MySQL 笔记](/docs/数据库/MySQL.md)
