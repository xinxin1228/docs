# IO 与 NIO

## 概述

> Java 的 IO 体系:传统 BIO(阻塞 IO,面向流)、NIO(非阻塞,面向缓冲区与通道)、以及 `Files` 工具类与序列化。理解 IO/NIO 的区别,是理解 Netty、Tomcat 高并发模型的起点。

## 核心知识点

### BIO:面向流

按数据单位分字节流 / 字符流,按方向分输入 / 输出:

```
字节流(处理二进制):InputStream / OutputStream
  └── FileInputStream、BufferedInputStream、ObjectInputStream
字符流(处理文本):Reader / Writer
  └── InputStreamReader、BufferedReader、FileWriter
```

```java
// 按行读文件(BufferedReader 是装饰器,包一层缓冲提升性能)
try (BufferedReader br = new BufferedReader(
        new InputStreamReader(new FileInputStream("a.txt"), StandardCharsets.UTF_8))) {
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

`InputStreamReader` 负责**字节→字符**的编码转换,必须显式指定 `StandardCharsets.UTF_8`,否则用平台默认编码易乱码。

### NIO:面向缓冲区与通道

三大核心:

| 组件 | 作用 |
|------|------|
| Buffer | 数据容器(`ByteBuffer`、`CharBuffer`...) |
| Channel | 数据传输通道(FileChannel、SocketChannel、ServerSocketChannel) |
| Selector | 多路复用:一个线程监听多个 Channel 的事件 |

```java
// 用 NIO 读文件
try (FileChannel channel = FileChannel.open(Path.of("a.txt"))) {
    ByteBuffer buf = ByteBuffer.allocate(1024);
    channel.read(buf);
    buf.flip();   // 切换读写模式
    String content = StandardCharsets.UTF_8.decode(buf).toString();
    System.out.println(content);
}
```

**BIO vs NIO 的本质区别**:BIO 一个连接一个线程,连接多了线程爆炸;NIO 用 Selector 一个线程管理成千上万连接(事件驱动),是 Netty/Tomcat NIO 模式的基础。

### Files 工具类(推荐)

JDK 7 的 `java.nio.file.Files` 一行搞定常见操作:

```java
List<String> lines = Files.readAllLines(Path.of("a.txt"));      // 读所有行
Files.writeString(Path.of("b.txt"), "hello", StandardCharsets.UTF_8);  // 写字符串
Files.copy(Path.of("a.txt"), Path.of("c.txt"), StandardCopyOption.REPLACE_EXISTING);
Files.move(Path.of("a.txt"), Path.of("d.txt"));
Files.deleteIfExists(Path.of("c.txt"));
Files.walk(Path.of("."))        // 递归遍历目录(流式)
     .filter(Files::isRegularFile)
     .forEach(System.out::println);
```

### 对象序列化

让对象可写盘/网络传输,实现 `Serializable`:

```java
public class User implements Serializable {
    private static final long serialVersionUID = 1L;   // 版本号,防反序列化不匹配
    private String name;
    private transient String password;   // transient 不序列化(敏感字段)
}

// 写
try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("u.obj"))) {
    oos.writeObject(new User());
}
// 读
try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("u.obj"))) {
    User u = (User) ois.readObject();
}
```

## 示例

```java
import java.nio.file.*;
import java.util.stream.Stream;

public class IoDemo {
    public static void main(String[] args) throws Exception {
        Path dir = Path.of("src");
        // 递归统计所有 .java 文件行数
        try (Stream<Path> walk = Files.walk(dir)) {
            long total = walk.filter(Files::isRegularFile)
                             .filter(p -> p.toString().endsWith(".java"))
                             .mapToLong(p -> {
                                 try { return Files.readAllLines(p).size(); }
                                 catch (Exception e) { return 0; }
                             })
                             .sum();
            System.out.println("Java 代码总行数: " + total);
        }
    }
}
```

## 常见问题与坑

- **中文乱码**:没指定 UTF-8,字符流默认用平台编码,务必显式传 `StandardCharsets.UTF_8`
- **流没关闭**:资源泄漏,用 `try-with-resources` 自动关闭
- **`Buffer` 忘 flip**:NIO 读之前要 `flip()`,否则读到的是脏数据
- **序列化 `serialVersionUID` 不写**:类改动后反序列化可能 `InvalidClassException`
- **大文件一次性 `readAllLines`**:内存爆掉,大文件要按行流式读或分块读
- **NIO 直接上手复杂**:日常文件操作用 `Files` 就够,只有高并发网络编程才需要 Selector/Netty

## 参考资料

- [Java IO 教程](https://docs.oracle.com/javase/tutorial/essential/io/)
- [NIO 包文档](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/nio/package-summary.html)
