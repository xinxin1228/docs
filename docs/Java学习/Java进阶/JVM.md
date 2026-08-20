# JVM

## 概述

> JVM(Java 虚拟机)是 Java「一次编译、到处运行」的底座。掌握内存结构、垃圾回收、类加载与常用排查工具,是定位内存泄漏、OOM、Full GC 频繁等线上问题的前提。

## 核心知识点

### 内存结构(运行时数据区)

| 区域 | 内容 | 是否线程私有 |
|------|------|--------------|
| 堆 Heap | 对象实例、数组(GC 主战场) | 共享 |
| 方法区(元空间 Metaspace) | 类元信息、常量、静态变量 | 共享 |
| 虚拟机栈 | 方法调用帧、局部变量表、操作数栈 | 私有 |
| 本地方法栈 | native 方法 | 私有 |
| 程序计数器 | 当前执行字节码行号 | 私有 |

- JDK 8 起「方法区」实现由永久代(PermGen)改为**元空间(Metaspace)**,用本地内存,不再受 `-XX:MaxPermSize` 限制
- 堆分为年轻代(Eden + 两个 Survivor)和老年代;对象先在 Eden,存活多次 Minor GC 后晋升到老年代

### 垃圾回收 GC

判断对象可回收用**可达性分析**:从 GC Roots(栈引用、静态变量、常量、JNI 引用)出发,不可达的对象即可回收。

主要收集器:

| 收集器 | 特点 |
|--------|------|
| Serial | 单线程,小堆/客户端 |
| Parallel | 吞吐优先,JDK 8 默认 |
| CMS | 低延迟,已废弃(JDK 14 移除) |
| **G1** | JDK 9+ 默认,区域化,可控停顿 |
| ZGC / Shenandoah | 超低延迟,大堆 |

G1 把堆划分为多个 Region,优先回收垃圾最多的 Region,目标停顿可控。

### 类加载机制

三个阶段:加载 → 链接(验证/准备/解析)→ 初始化。

**双亲委派模型**:类加载请求先交给父加载器,父加载不了才自己加载,避免核心类被篡改:

```
Bootstrap(加载 rt.jar 核心类)
  ↑
Extension/Platform(加载扩展类)
  ↑
Application(加载 classpath 下的类)
```

破坏双亲委派的典型场景:Tomcat(每个 webapp 独立加载)、SPI(如 JDBC 驱动,JDK 用「线程上下文类加载器」反向下加载)。

### 常用 JVM 参数

```bash
-Xms512m              # 初始堆大小
-Xmx2g                # 最大堆大小(两者通常设成一样,避免动态扩容)
-Xss1m                # 每个线程栈大小
-XX:MaxMetaspaceSize=256m
-XX:+UseG1GC          # 使用 G1
-XX:MaxGCPauseMillis=200
-XX:+PrintGCDetails   # 打印 GC 详情(JDK 9+ 用 -Xlog:gc*)
```

## 示例

```bash
# 排查一条命令:定位占用内存最高的 Java 进程
jps                       # 列出 Java 进程和 PID
jmap -heap <pid>          # 查看堆配置与使用
jstat -gcutil <pid> 1000  # 每秒打印 GC 占用率(FGC 次数猛增 = 要调优)
jstack <pid>              # 打印线程栈,定位死锁/阻塞
jmap -dump:format=b,file=heap.hprof <pid>  # 导出堆转储,用 MAT 分析
```

### 案例:一个内存泄漏如何一步步暴露

```java
// 典型泄漏:静态集合持续增长,元素永不释放
public class LeakDemo {
    private static final List<byte[]> CACHE = new ArrayList<>();
    public static void main(String[] args) throws Exception {
        while (true) {
            CACHE.add(new byte[1024 * 1024]);   // 每次加 1MB,永不清理
            Thread.sleep(100);
        }
    }
}
```

程序启动后用 `jstat -gcutil <pid> 1000` 每秒观察,会看到老年代 O 区占用持续上涨、FGC 越来越频繁,最终抛 `OutOfMemoryError: Java heap space`。定位流程:

1. `jps` 找到 PID
2. `jmap -dump:format=b,file=heap.hprof <pid>` 导出堆
3. 用 MAT 打开,看「支配树」里 `LeakDemo` 的 `CACHE`(ArrayList)持有海量 `byte[]` → 锁定泄漏点

分析 OOM 的通用思路:导出 hprof → 用 Eclipse MAT 或 IDEA Profiler 看「支配树」里谁持有大量对象 → 定位泄漏点。

## 常见问题与坑

- **OOM:Java heap space**:堆不够,要么加 `-Xmx`,要么有内存泄漏(对象一直被引用无法回收)
- **OOM:Metaspace**:类加载过多(动态生成类、热部署),调大 `-XX:MaxMetaspaceSize`
- **StackOverflowError**:递归太深或栈太小
- **Full GC 频繁**:老年代持续增长,可能是内存泄漏或对象过早晋升,需要 dump 分析
- **`-Xms` ≠ `-Xmx` 导致频繁扩容**:生产建议两者一致
- **把 GC 调优当性能银弹**:先优化代码(减少对象创建、缓存、分页),再谈 JVM 参数

## 参考资料

- [JVM 内存管理白皮书](https://docs.oracle.com/en/java/javase/17/gctuning/)
- [美团 JVM 调优实战](https://tech.meituan.com/tags/jvm.html)
