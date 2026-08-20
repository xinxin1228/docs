# Lambda 与 Stream

## 概述

> Java 8 引入的函数式编程核心:Lambda 表达式、函数式接口、方法引用、Stream 流式操作、Optional。这是从「命令式写集合循环」到「声明式写数据处理」的关键转变,后端代码里几乎无处不在。

## 核心知识点

### 函数式接口与 Lambda

函数式接口 = 只有一个抽象方法的接口,可用 `@FunctionalInterface` 标注。Lambda 是它的匿名实现简写:

```java
// 传统匿名内部类
Runnable r1 = new Runnable() {
    @Override public void run() { System.out.println("hi"); }
};
// Lambda 简化
Runnable r2 = () -> System.out.println("hi");
```

常用函数式接口(`java.util.function`):

| 接口 | 方法 | 用途 |
|------|------|------|
| `Predicate<T>` | `boolean test(T)` | 判断过滤 |
| `Function<T,R>` | `R apply(T)` | 转换映射 |
| `Consumer<T>` | `void accept(T)` | 消费 |
| `Supplier<T>` | `T get()` | 供给 |
| `BiFunction<T,U,R>` | `R apply(T,U)` | 双参转换 |

### 方法引用 `::`

```java
list.forEach(System.out::println);       // 对象::实例方法
list.stream().map(String::toUpperCase);  // 类::实例方法
list.stream().map(User::getName);        // 类::实例方法
```

### Stream 流式操作

```java
List<User> users = ...;

List<String> names = users.stream()                 // 创建流
    .filter(u -> u.getAge() > 18)                   // 中间:过滤
    .map(User::getName)                             // 中间:映射
    .distinct()                                     // 中间:去重
    .sorted()                                       // 中间:排序
    .collect(Collectors.toList());                  // 终结:收集

long count = users.stream().filter(u -> u.isVip()).count();
boolean any = users.stream().anyMatch(u -> u.getAge() > 30);
```

- **中间操作惰性**:`filter`/`map` 等不触发执行,只有遇到**终结操作**(`collect`/`count`/`forEach`)才真正跑
- **一次流只能消费一次**:终结后流关闭,不能再复用

### 常用 Collectors

```java
Collectors.toList();
Collectors.groupingBy(User::getDept);                // 分组
Collectors.partitioningBy(User::isVip);              // 二分组
Collectors.joining(",");                             // 拼接字符串
Collectors.toMap(User::getId, u -> u);               // 转 Map
Collectors.summarizingInt(User::getAge);             // 统计
```

### Optional 防空

```java
Optional<User> opt = Optional.ofNullable(findById(1));
String name = opt.map(User::getName)      // 有值才映射
                 .orElse("未知");          // 空则默认
User u = opt.orElseThrow(() -> new BizException("用户不存在"));
opt.ifPresent(u -> System.out.println(u.getName()));
```

`map` 返回 `Optional`,嵌套结构(如 `Optional<Optional>` )要用 `flatMap` 拍平。

## 示例

```java
record User(String name, int age, boolean vip) {}

public class StreamDemo {
    public static void main(String[] args) {
        List<User> users = List.of(
            new User("张三", 20, true),
            new User("李四", 16, false),
            new User("王五", 25, true),
            new User("赵六", 30, false)
        );

        // 找出 18 岁以上 VIP 的名字,按年龄降序,拼成字符串
        String result = users.stream()
            .filter(u -> u.age() > 18)
            .filter(User::vip)
            .sorted((a, b) -> b.age() - a.age())
            .map(User::name)
            .collect(Collectors.joining("、"));
        System.out.println(result);   // 王五、张三
    }
}
```

## 常见问题与坑

- **流复用**:一个 Stream 终结后就关闭,复用会抛 `IllegalStateException`,需要时重新 `stream()`
- **`toMap` 重复 key**:key 重复会抛异常,要提供合并函数 `toMap(k, v, (a,b) -> a)`
- **在 Lambda 里修改外部变量**:只能捕获**事实 final** 的变量(不能改变其引用),要累加用 `AtomicInteger` 或 collect
- **`parallelStream` 别乱用**:并行流在 CPU 密集、数据量大时才有收益,且要注意线程安全与共享可变状态
- **`Optional` 别到处套**:它适合「可能为空」的返回值,不适合当字段或方法参数

## 参考资料

- [Lambda 教程](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
- [Stream 文档](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Stream.html)
