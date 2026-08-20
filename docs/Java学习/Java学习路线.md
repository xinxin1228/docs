# Java 学习路线

## 概述

> 记录从纯前端转全栈的 Java 学习路线。前端(JS/TS)与 Java 的差异是理解重点:Java 是**静态强类型、编译型、面向对象**语言,没有原型链、没有 `===`、没有「真值/假值」短路,思维要切到「先定义类型、再谈逻辑」。

## 核心知识点

### 六个阶段(建议按序推进)

| 阶段 | 目标 | 对应笔记 | 关键产出 |
|------|------|----------|----------|
| 1. Java 基础 | 语法 + 面向对象 + 集合 | Java 基础 | 能写工具类、看懂业务代码 |
| 2. Java 进阶 | JVM / 并发 / IO / 设计模式 | Java 进阶 | 能排查性能、写并发安全代码 |
| 3. 构建工具 | Maven 依赖与多模块 | 构建工具 | 能搭工程、管依赖 |
| 4. Spring 生态 | IOC/AOP/Boot/MVC/JPA/MyBatis/Security | Spring 生态 | 能写完整后端接口 |
| 5. 数据库集成 | Flyway / MySQL / Redis | 数据库集成 | 能管库版本、做缓存 |
| 6. 项目实战 | 从零搭项目 + 完整案例 | 项目实战 | 能独立交付一个模块 |

### 与前端技能的衔接(已有笔记复用)

- **SQL 基础**:直接复用本站《数据库》章节的 MySQL 笔记,Java 侧只补「JDBC / ORM 映射 / 事务」这一层
- **HTTP / RESTful**:复用《计算机网络》和《API 接口规范》,Java 侧只补 Spring MVC 的注解映射
- **工程化思维**:我熟 npm/pnpm 的依赖管理,Maven 的 `dependency`/`scope` 概念几乎一一对应,迁移成本低
- **Nginx / Docker / Jenkins**:已在《服务器相关》《Docker 学习》,后端部署直接套用

### 转岗最容易踩的三个思维差

1. **值语义 vs 引用语义**:Java 基本类型传值、对象传引用;`==` 比引用,`equals()` 比内容——和 JS 的 `===` 完全不同
2. **空值**:Java 没有 `undefined`,只有 `null`,但 `null` 调用方法直接 `NullPointerException`,要习惯用 `Optional` / 前置校验
3. **同步/异步**:Java 默认同步阻塞,异步靠线程池 / `CompletableFuture`,不像 JS 天生事件循环

## 示例

```java
// JS 里的 a === b,在 Java 里要分场景:
String a = "hello";
String b = new String("hello");
System.out.println(a == b);        // false,比的是引用地址
System.out.println(a.equals(b));   // true,比的是内容
```

## 常见问题与坑

- **急着上 Spring**:没有集合/泛型/反射基础,Spring 的自动装配、泛型注入会看得一头雾水,阶段 1 不要跳
- **只看不写**:Java 是「编译期」语言,很多错误(类型不匹配、泛型擦除)不写出来体会不到,每篇笔记的示例建议手敲一遍
- **版本混用**:笔记以 **Java 17 + 21** 为主(我的技术栈是 17-26),老教程里的 Java 8 写法(匿名内部类)要主动识别并替换成 Lambda

## 参考资料

- [Java SE 官方教程](https://docs.oracle.com/javase/tutorial/)
- [Spring Boot 官方文档](https://docs.spring.io/spring-boot/)
- [MyBatis-Plus 官方文档](https://baomidou.com/)
- [MySQL 笔记](/docs/数据库/MySQL.md)
- [Redis 笔记](/docs/数据库/Redis.md)
