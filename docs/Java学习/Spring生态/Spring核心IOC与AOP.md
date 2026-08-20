# Spring 核心 IOC 与 AOP

## 概述

> Spring 的两大核心:IOC(控制反转,对象的创建与依赖交给容器管理)和 AOP(面向切面编程,把横切逻辑如日志、事务从业务中剥离)。理解 IOC 容器与 AOP 代理,是看懂整个 Spring 生态的前提。

## 核心知识点

### IOC 与依赖注入

传统方式:类自己 `new` 依赖对象(控制在自己手里)。IOC:依赖由容器创建并注入(控制反转给容器),类只需声明需要什么。

```java
// 传统:自己 new,强耦合
public class OrderService {
    private UserService userService = new UserService();
}

// IOC:声明依赖,由容器注入
@Service
public class OrderService {
    private final UserService userService;
    public OrderService(UserService userService) {   // 构造器注入
        this.userService = userService;
    }
}
```

**构造器注入是首选**(依赖明确、便于测试、可加 `final`),避免字段注入 `@Autowired` 带来的隐式依赖和循环依赖隐患。

### Bean 与容器

`ApplicationContext` 是 IOC 容器,管理所有 **Bean**(由 Spring 创建、装配、管理的对象)。

三种注册 Bean 的方式:

```java
// 1. 注解扫描:类上加组件注解,Spring 扫描包自动注册
@Component        // 通用
@Service          // 业务层
@Repository       // 数据访问层
@Controller       // 控制层

// 2. Java 配置类:方法返回对象并标注 @Bean
@Configuration
public class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

三种注入方式:

```java
// 构造器注入(推荐,可省略 @Autowired,单构造器时自动注入)
public OrderService(UserService userService) { ... }

// Setter 注入
@Autowired
public void setUserService(UserService s) { ... }

// 字段注入(不推荐)
@Autowired
private UserService userService;
```

多个同类型 Bean 时用 `@Qualifier` 指定,或用 `@Primary` 指定默认。

### Bean 作用域与生命周期

```java
@Scope("singleton")   // 默认,单例(一个容器一个实例)
@Scope("prototype")   // 原型,每次获取都新建
// web 环境还有:request、session
```

**注意单例 Bean 不要存可变状态**(线程不安全),业务状态放方法局部变量或 request 作用域。

### AOP 面向切面

把「日志、事务、鉴权、性能统计」等横切关注点与业务逻辑分离:

```java
@Aspect
@Component
public class LogAspect {

    // 切点:拦截 com.example.service 包下所有方法
    @Pointcut("execution(* com.example.service..*(..))")
    public void serviceLayer() {}

    // 环绕通知:方法执行前后都织入
    @Around("serviceLayer()")
    public Object log(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            return pjp.proceed();               // 执行原方法
        } finally {
            System.out.println(pjp.getSignature() + " 耗时 " +
                (System.currentTimeMillis() - start) + "ms");
        }
    }
}
```

通知类型:

| 通知 | 注解 | 时机 |
|------|------|------|
| 前置 | `@Before` | 方法执行前 |
| 后置 | `@After` | 方法执行后(无论成败) |
| 返回 | `@AfterReturning` | 正常返回后 |
| 异常 | `@AfterThrowing` | 抛异常后 |
| 环绕 | `@Around` | 前后都可,最灵活 |

### AOP 底层:动态代理

- 目标类**有接口** → JDK 动态代理(`java.lang.reflect.Proxy`)
- 目标类**无接口** → CGLIB 字节码代理(Spring Boot 2.x+ 默认全用 CGLIB)

关键结论:**AOP 只在通过 Spring 容器获取的 Bean 上生效**;类内部自调用(`this.method()`)不会走代理,切面不生效。

## 示例

```java
// 用 AOP 实现一个简单的性能监控
@Aspect
@Component
public class MetricsAspect {

    @Around("@annotation(Timed)")          // 拦截带 @Timed 注解的方法
    public Object measure(ProceedingJoinPoint pjp) throws Throwable {
        long t0 = System.nanoTime();
        Object result = pjp.proceed();
        long cost = (System.nanoTime() - t0) / 1_000_000;
        System.out.println(pjp.getSignature().getName() + " 耗时 " + cost + "ms");
        return result;
    }
}
```

## 常见问题与坑

- **字段注入 `@Autowired` 泛滥**:难测试、易循环依赖,改构造器注入
- **AOP 内部自调用不生效**:`this.xxx()` 绕过代理,改成从容器拿自己或拆成两个 Bean
- **单例 Bean 存状态**:多线程共享可变字段会出并发 bug
- **循环依赖**:A 注入 B、B 注入 A,构造器注入下直接报错(这是好事,逼着重构),字段注入则可能用「三级缓存」勉强解开但埋隐患
- **切点表达式写错**:`execution` 表达式作用域(public/protected)、包路径要精确,多打印验证

## 参考资料

- [Spring 核心文档](https://docs.spring.io/spring-framework/reference/core.html)
- [Spring AOP 文档](https://docs.spring.io/spring-framework/reference/core/aop.html)
