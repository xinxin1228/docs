# Spring Boot 快速上手

## 概述

> Spring Boot 是 Spring 生态的「一键启动」方案,通过自动配置(Auto-Configuration)和 starter 依赖,把繁琐的 XML 配置、版本管理、内嵌服务器全部搞定。本笔记覆盖核心注解、配置、起步依赖与常用组件。

## 核心知识点

### 为什么用 Spring Boot

| 痛点(传统 Spring) | Boot 的解决 |
|--------------------|-------------|
| 一堆 XML/Java 配置 | 自动配置,零配置起步 |
| 依赖版本难管理 | starter + BOM 锁定版本 |
| 要外置 Tomcat | 内嵌 Tomcat,`java -jar` 直接跑 |
| 监控/健康检查缺失 | Actuator 开箱即用 |

### 核心注解 @SpringBootApplication

```java
@SpringBootApplication   // 三个注解的合成
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

等价于:

```java
@Configuration                    // 标记配置类
@EnableAutoConfiguration          // 开启自动配置(核心)
@ComponentScan                    // 扫描当前包及子包的 @Component
```

**自动配置原理**:`@EnableAutoConfiguration` 根据 classpath 里的 jar(如引入了 web starter 就有 Tomcat)和项目配置,自动装配对应 Bean。核心是 `spring-boot-autoconfigure` 里 `META-INF/spring/...AutoConfiguration.imports` 的按条件装配。

### Starter 依赖

「一站式」依赖集合,一个 starter 引入一整组相关依赖:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>         <!-- Web MVC + 内嵌 Tomcat -->
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>    <!-- JPA + Hibernate -->
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>  <!-- 参数校验(Boot 3 起独立) -->
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>    <!-- 监控 -->
</dependency>
```

### 配置:application.yml

```yaml
server:
  port: 8080

spring:
  application:
    name: demo
  datasource:
    url: jdbc:mysql://localhost:3306/demo?useUnicode=true&characterEncoding=utf8
    username: root
    password: ${DB_PASSWORD}   # 敏感信息用环境变量

my:
  custom:
    enabled: true
    limit: 100
```

读取配置:

```java
// 方式一:单个值
@Value("${my.custom.limit}")
private int limit;

// 方式二:类型安全绑定(推荐)
@ConfigurationProperties(prefix = "my.custom")
@Validated
public class CustomProperties {
    private boolean enabled;
    private int limit;
    // getter/setter
}
```

### 常用组件

- **Actuator**:`/actuator/health`、`/actuator/metrics` 等监控端点,健康检查、指标采集必备
- **DevTools**:开发热重载,`spring-boot-devtools`(仅开发用)
- **CommandLineRunner / ApplicationRunner**:启动后执行一次初始化逻辑

## 示例

```java
// 一个最小可跑的 Spring Boot 应用
@SpringBootApplication
@RestController
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @GetMapping("/hello")
    public String hello(@RequestParam(defaultValue = "world") String name) {
        return "Hello, " + name;
    }
}
```

启动:

```bash
mvn spring-boot:run
# 或打包后运行
mvn clean package && java -jar target/demo-0.0.1-SNAPSHOT.jar
# 访问 http://localhost:8080/hello?name=Java
```

## 常见问题与坑

- **启动类位置**:`@SpringBootApplication` 要在包根目录,才能 `@ComponentScan` 扫到所有子包的 Bean
- **Bean 扫不到**:自定义类不在启动类包路径下,用 `@ComponentScan` 或 `scanBasePackages` 显式指定
- **端口冲突**:`Port already in use`,改 `server.port` 或 `--server.port=8081` 启动参数
- **依赖版本混乱**:别手动写 starter 版本,交给 `spring-boot-dependencies` BOM 管;混用不同 Boot 版本会踩冲突
- **`@Value` 读不到**:属性名/前缀拼错,或类没被 Spring 管理(没加 `@Component`)
- **配置密码硬编码**:用环境变量注入,别把密码写进 yml 提交仓库

## 参考资料

- [Spring Boot 参考文档](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring Initializr(快速生成项目)](https://start.spring.io/)
