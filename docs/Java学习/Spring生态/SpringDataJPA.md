# Spring Data JPA

## 概述

> Spring Data JPA 基于 Hibernate,把「实体类 ↔ 数据库表」做 ORM 映射,提供 `JpaRepository` 一行接口就获得 CRUD、分页、派生查询等能力。适合以领域模型为中心、写 SQL 较少的场景。核心难点是关联映射与 N+1 问题。

## 核心知识点

### 实体映射

```java
import jakarta.persistence.*;

@Entity
@Table(name = "t_user")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)   // 自增主键
    private Long id;

    @Column(name = "username", nullable = false, length = 50)
    private String username;

    @Column(unique = true)
    private String email;

    @Enumerated(EnumType.STRING)      // 枚举存字符串,别存序号
    private Gender gender;

    // 时间字段
    @CreatedDate
    private LocalDateTime createdAt;

    // 无参构造器(JPA 必需)
    public User() {}
    // getter/setter...
}
```

### Repository 接口

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // 开箱即用:save / findById / findAll / deleteById / count ...

    // 派生查询:按方法名自动生成 SQL
    List<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
    List<User> findByAgeGreaterThanAndVipTrue(int age);
    List<User> findByUsernameContaining(String keyword);
    Page<User> findByAgeGreaterThan(int age, Pageable pageable);

    // 自定义 JPQL(面向实体,不是表)
    @Query("select u from User u where u.age > :age order by u.createdAt desc")
    List<User> findAdult(@Param("age") int age);

    // 原生 SQL
    @Query(value = "select * from t_user where age > :age", nativeQuery = true)
    List<User> findAdultNative(@Param("age") int age);

    // 更新/删除要加 @Modifying + @Transactional
    @Modifying
    @Query("update User u set u.vip = true where u.id = :id")
    int makeVip(@Param("id") Long id);
}
```

### 关联关系与懒加载

```java
@Entity
public class Order {
    @Id private Long id;

    @ManyToOne(fetch = FetchType.LAZY)   // 多对一,默认建议 LAZY
    @JoinColumn(name = "user_id")
    private User user;
}

@Entity
public class User {
    @Id private Long id;

    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    private List<Order> orders;
}
```

- `@ManyToOne` 默认 EAGER(性能隐患),建议显式改 `LAZY`
- `@OneToMany` 默认 LAZY
- `mappedBy` 表示「关系由对方的 `user` 字段维护」,避免双向关系生成两张中间表
- `cascade` 级联:保存父时连带保存子

### N+1 问题与解决

查询「用户列表 + 每个用户的订单」时,若懒加载会触发「1 次查用户 + N 次查订单」:

```java
// 解决一:JOIN FETCH 一次查回
@Query("select distinct u from User u left join fetch u.orders where u.age > :age")
List<User> findWithOrders(@Param("age") int age);

// 解决二:@EntityGraph 声明式抓取
@EntityGraph(attributePaths = "orders")
List<User> findByAgeGreaterThan(int age);
```

**规律:需要返回关联数据时,用 JOIN FETCH / EntityGraph 一次性抓取,避免 N+1。**

### 分页与事务

```java
Page<User> page = userRepository.findAll(PageRequest.of(0, 20, Sort.by("createdAt").descending()));
long total = page.getTotalElements();
List<User> list = page.getContent();

@Transactional   // 写操作加事务
public void updateUser(Long id, UserDto dto) { ... }
```

## 示例

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserRepository repo;
    public UserController(UserRepository repo) { this.repo = repo; }

    @GetMapping
    public Result<Page<User>> page(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size) {
        return Result.ok(repo.findAll(PageRequest.of(page, size)));
    }

    @PostMapping
    public Result<User> create(@Valid @RequestBody UserDto dto) {
        User u = new User();
        u.setUsername(dto.getUsername());
        u.setEmail(dto.getEmail());
        return Result.ok(repo.save(u));   // save:有 id 则更新,无则插入
    }
}
```

## 常见问题与坑

- **`@ManyToOne` 默认 EAGER**:不经意间查一次带一堆关联,显式 `LAZY`
- **N+1 查询**:列表页尤其明显,用 `JOIN FETCH` 或 `@EntityGraph`
- **`save` 判断更新还是插入**:`save` 对有 id 的实体先 select 再 insert/update,批量更新性能差,用 `@Modifying` 批量 SQL 或 `saveAll`
- **实体作为 DTO 直接返回**:关联懒加载在序列化时抛 `LazyInitializationException`,用 DTO 投影或 DTO 转换
- **`@Modifying` 忘加 `@Transactional`**:报 `TransactionRequiredException`
- **双向关联的 `equals`/`hashCode`**:避免用懒加载字段做 hash,否则踩坑

## 参考资料

- [Spring Data JPA 文档](https://docs.spring.io/spring-data/jpa/reference/)
- [Hibernate 官方文档](https://hibernate.org/orm/documentation/)
- [JPA N+1 问题详解](https://vladmihalcea.com/n-plus-1-query-problem/)
