# Maven 基础

## 概述

> Maven 是 Java 的构建与依赖管理工具,对应前端的 npm/pnpm。它用 `pom.xml` 声明依赖和构建配置,通过「约定优于配置」统一项目结构。理解坐标、生命周期、依赖范围是上手关键。

## 核心知识点

### Maven 是什么

- **依赖管理**:声明 `groupId:artifactId:version`,自动下载传递依赖
- **构建自动化**:编译、测试、打包、部署一条命令完成
- **约定优于配置**:默认目录结构(src/main/java、src/test/java)不用手配

对应关系:前端 `package.json` ↔ `pom.xml`,npm 仓库 ↔ Maven 中央仓库,`node_modules` ↔ 本地仓库 `~/.m2/repository`。

### 坐标(Coordinates)

唯一定位一个构件,三段坐标:

```xml
<groupId>com.example</groupId>        <!-- 组织/公司域名倒序 -->
<artifactId>demo-service</artifactId> <!-- 项目/模块名 -->
<version>1.0.0</version>              <!-- 版本号 -->
<packaging>jar</packaging>            <!-- 打包类型:jar/war/pom -->
```

### 生命周期与命令

Maven 三大生命周期,各含多个**阶段(phase)**,按顺序执行:

```bash
mvn clean        # 清理 target
mvn compile      # 编译源码 → target/classes
mvn test         # 编译 + 运行单元测试
mvn package      # 打包成 jar/war → target/
mvn install      # 打包并装到本地仓库 ~/.m2
mvn deploy       # 打包并发布到远程仓库
```

调用某个阶段,会依次执行它之前的所有阶段(如 `mvn package` 会先 compile、test)。

常用附加参数:

```bash
mvn clean package -DskipTests        # 打包但跳过测试
mvn test -Dtest=UserServiceTest      # 只跑指定测试
mvn dependency:tree                  # 查看依赖树
```

### pom.xml 结构

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <properties>
        <java.version>17</java.version>      <!-- 统一版本号,便于引用 -->
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### 依赖范围 scope

| scope | 说明 |
|-------|------|
| `compile`(默认) | 编译/测试/运行都需要,会传递 |
| `provided` | 编译/测试需要,运行时由容器提供(如 servlet-api、lombok) |
| `runtime` | 编译不需要,运行时需要(如 JDBC 驱动) |
| `test` | 仅测试(如 JUnit) |
| `import` | 导入 BOM(见下一篇) |

### 仓库与镜像

- **本地仓库** `~/.m2/repository`:下载后的缓存
- **中央仓库** Maven Central:默认来源
- **私服/镜像**:公司内网 Nexus,或阿里云镜像加速

阿里云镜像(写入 `~/.m2/settings.xml`):

```xml
<mirrors>
  <mirror>
    <id>aliyun</id>
    <mirrorOf>central</mirrorOf>
    <name>aliyun</name>
    <url>https://maven.aliyun.com/repository/public</url>
  </mirror>
</mirrors>
```

## 示例

```bash
# 一条命令创建标准结构的 Spring Boot 项目(推荐用 start.spring.io 更直观)
mvn archetype:generate \
  -DgroupId=com.example -DartifactId=demo \
  -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

cd demo && mvn clean package
```

## 常见问题与坑

- **依赖下载慢**:换阿里云镜像或配置公司私服
- **`SNAPSHOT` 与 `RELEASE`**:开发用 `1.0-SNAPSHOT`(可重复覆盖),发布用 `1.0.0`(不可变),别把快照当正式版发
- **依赖冲突**:传递依赖版本不一致,用 `mvn dependency:tree` 排查(解决方案见下一篇)
- **`mvn` 命令找不到**:没装 Maven 或没进 PATH,`mvn -v` 验证
- **IDEA 里 Maven 和命令行不一致**:IDEA 可能用内置 Maven,确认 settings.xml 和 JDK 指向一致

## 参考资料

- [Maven 官方文档](https://maven.apache.org/guides/)
- [Maven 中央仓库](https://search.maven.org/)
