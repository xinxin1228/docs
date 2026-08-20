# MyBatis 与 MyBatis-Plus

## 概述

> MyBatis 是「半自动」ORM,SQL 完全自己写(灵活性高),负责把 SQL 结果映射到对象;MyBatis-Plus 是它的增强,用 `BaseMapper` 一行不写 SQL 就完成单表 CRUD、条件构造、分页。适合 SQL 复杂、追求可控性的场景。

## 核心知识点

### MyBatis vs JPA

| | MyBatis | JPA(Hibernate) |
|---|---|---|
| SQL 控制 | 手写 SQL,完全可控 | 框架生成,复杂查询要 JPQL |
| 学习成本 | 低(会 SQL 就行) | 高(要懂实体状态、缓存、懒加载) |
| 灵活度 | 高,适合复杂报表/多表 | 单表 CRUD 快 |
| 适用 | SQL 复杂、性能敏感 | 领域模型驱动、CRUD 为主 |

### 基本使用

实体 + Mapper 接口:

```java
public class User {
    private Long id;
    private String username;
    private String email;
    // getter/setter
}

@Mapper
public interface UserMapper {
    User findById(@Param("id") Long id);
    List<User> findByCondition(@Param("kw") String kw, @Param("age") Integer age);
    int insert(User user);
    int update(User user);
    int deleteById(Long id);
}
```

XML 映射(或注解):

```xml
<mapper namespace="com.example.mapper.UserMapper">
    <select id="findById" resultType="com.example.entity.User">
        select * from t_user where id = #{id}
    </select>

    <select id="findByCondition" resultType="com.example.entity.User">
        select * from t_user
        <where>
            <if test="kw != null and kw != ''">
                and username like concat('%', #{kw}, '%')
            </if>
            <if test="age != null">
                and age = #{age}
            </if>
        </where>
    </select>
</mapper>
```

### #{} 与 ${} 的区别(安全关键)

```xml
<!-- #{id}:预编译占位符,参数绑定,防 SQL 注入 ✅ -->
select * from t_user where id = #{id}

<!-- ${id}:字符串拼接,有 SQL 注入风险,仅用于表名/列名等无法预编译的场景 ❌ -->
select * from ${tableName}
```

**规则:能用 `#{}` 就用 `#{}`,永远不要用 `${}` 拼接用户输入。**

### 动态 SQL

```xml
<where>          <!-- 自动处理多余 and/or -->
<set>            <!-- 更新时自动处理逗号 -->
<if test="..."> <foreach collection="list" item="id" open="(" separator="," close=")"> #{id} </foreach>
<choose>/<when>/<otherwise>   <!-- 类似 if-else -->
```

### MyBatis-Plus 增强

引入 `mybatis-plus-boot-starter` 后,`BaseMapper` 自带单表能力:

```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
    // 继承即获得:insert / deleteById / updateById / selectById / selectList / selectPage ...
}
```

```java
// 零 SQL 完成条件查询
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
wrapper.eq(User::getAge, 20)
       .like(User::getUsername, "张")
       .orderByDesc(User::getCreatedAt)
       .last("limit 10");
List<User> users = userMapper.selectList(wrapper);

// 分页(需配置分页插件 MybatisPlusInterceptor + PaginationInnerInterceptor)
Page<User> page = new Page<>(1, 20);
userMapper.selectPage(page, wrapper);
```

还提供 `IService` / `ServiceImpl`(通用 Service 层)、代码生成器、逻辑删除、乐观锁、字段自动填充等。

### 与 Spring Boot 集成

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
    <version>3.5.5</version>   <!-- Spring Boot 3 用 boot3-starter -->
</dependency>
```

```yaml
mybatis-plus:
  mapper-locations: classpath*:/mapper/**/*.xml
  configuration:
    map-underscore-to-camel-case: true   # 下划线列名 ↔ 驼峰字段自动映射
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl  # 开发期打印 SQL
```

## 示例

```java
// 条件构造器:等价于
// SELECT * FROM t_user WHERE (age > 18 AND vip = 1) OR username LIKE '%张%'
LambdaQueryWrapper<User> w = new LambdaQueryWrapper<>();
w.gt(User::getAge, 18).eq(User::getVip, true)
 .or()
 .like(User::getUsername, "张");
List<User> result = userMapper.selectList(w);
```

## 常见问题与坑

- **用 `${}` 拼用户输入**:SQL 注入漏洞,改用 `#{}`
- **resultType 写错 / 字段映射不上**:开启 `map-underscore-to-camel-case`,或写 `resultMap`
- **MyBatis-Plus 版本与 Boot 不匹配**:Boot 2 用 `mybatis-plus-boot-starter`,Boot 3 用 `mybatis-plus-spring-boot3-starter`
- **分页不生效**:没注册 `PaginationInnerInterceptor` 分页插件,`selectPage` 退化成全量查询
- **`selectList` 查大数据量**:没分页,内存爆掉,记得 `Page` 或 `last("limit N")`
- **实体字段和表字段名不一致**:用 `@TableField` 或统一驼峰映射

## 参考资料

- [MyBatis 官方文档](https://mybatis.org/mybatis-3/zh_CN/)
- [MyBatis-Plus 文档](https://baomidou.com/)
