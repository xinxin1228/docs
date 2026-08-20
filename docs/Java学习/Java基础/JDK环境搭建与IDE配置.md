# JDK 环境搭建与 IDE 配置

## 概述

> 记录在 macOS 上搭建 Java 开发环境:装 JDK、配 `JAVA_HOME`、装 IntelliJ IDEA、配置 Maven 与 IDEA 联动。我已具备前端环境,重点在于理解 JDK 与 JRE 的区别,以及多个 JDK 版本如何切换。

## 核心知识点

### JDK vs JRE vs JVM

| 概念 | 作用 |
|------|------|
| JVM | 运行字节码的虚拟机,一次编译到处运行的关键 |
| JRE | JVM + 核心类库,只够「运行」Java 程序 |
| JDK | JRE + 编译器(javac)+ 调试/监控工具,开发必须装 JDK |

开发装 **JDK**,部署线上很多只装 **JRE**(或直接用 JDK 精简版)。JDK 9 之后不再单独发行 JRE,`jlink` 可以自裁剪运行时。

### 装 JDK(推荐 brew)

```bash
# 查看可用版本
brew search openjdk

# 装 LTS 版本(推荐 17 或 21)
brew install openjdk@17
brew install openjdk@21
```

### 配置 JAVA_HOME

brew 装的 openjdk 是 keg-only,不会自动链接,需手动配置环境变量:

```bash
# 以 17 为例,写入 ~/.zshrc
echo 'export JAVA_HOME=$(/usr/libexec/java_home -v 17)' >> ~/.zshrc
echo 'export PATH="$JAVA_HOME/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

验证:

```bash
java -version
# openjdk version "17.0.x" ...
javac -version
echo $JAVA_HOME
```

### 多版本切换

macOS 自带 `/usr/libexec/java_home` 命令,可列出并切换:

```bash
/usr/libexec/java_home -V   # 列出所有已装 JDK
/usr/libexec/java_home -v 21  # 切到 21
```

团队里常见做法是配一个别名,或直接用 IDE 的 Project SDK 覆盖。另一个方案是 [jenv](https://www.jenv.be/) 做目录级版本管理(类比前端 nvm)。

### IDEA 配置要点

1. **Project SDK**:`File → Project Structure → SDKs`,选择上面装的 JDK 17/21
2. **Maven 集成**:`Settings → Build → Maven`,把 Maven home 指向本机 Maven(或 IDEA 内置),`User settings file` 指向 `~/.m2/settings.xml`(配了阿里云镜像后生效)
3. **编码**:`Settings → Editor → File Encodings`,全设为 UTF-8(Java 源码默认 UTF-8,避免中文注释乱码)
4. **代码格式化**:团队统一用 Google Java Format 或 IDEA 默认风格,配合 `.editorconfig`

## 示例

```bash
# 一次性验证整个链路是否通
mkdir -p ~/tmp/hello && cd ~/tmp/hello
cat > Hello.java <<'EOF'
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello Java " + System.getProperty("java.version"));
    }
}
EOF
javac Hello.java   # 编译出 Hello.class
java Hello         # 运行,输出 Hello Java 17.x
```

## 常见问题与坑

- **`java` 可用但 `javac` 不可用**:说明装的是 JRE 或 JDK 没进 PATH,检查 `JAVA_HOME/bin` 是否在 PATH 里
- **`JAVA_HOME` 改了但 `java -version` 还是旧的**:终端缓存了旧 PATH,重开终端或 `source ~/.zshrc`;用 `which java` 看实际解析到哪
- **IDEA 里报 `SDK not found`**:项目 SDK 选了已卸载的版本,重新指向新 JDK
- **brew openjdk 不生效**:忘记 `export JAVA_HOME`,keg-only 包不会自动进 PATH
- **编译中文报 `编码 GBK 不可映射字符`**:`javac -encoding UTF-8 Hello.java`,或统一 UTF-8

## 参考资料

- [Homebrew openjdk](https://formulae.brew.sh/formula/openjdk@17)
- [jenv](https://www.jenv.be/)
- [IntelliJ IDEA 文档](https://www.jetbrains.com/help/idea/)
