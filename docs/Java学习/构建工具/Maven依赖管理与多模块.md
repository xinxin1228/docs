# Maven 依赖管理与多模块

## 概述

> 进阶主题:传递依赖与版本仲裁、`dependencyManagement` 与 BOM、依赖排除、以及多模块(parent/aggregator)工程。这是中大型后端项目的标配——统一版本、拆分模块、管理依赖冲突。

## 核心知识点

### 传递依赖与版本仲裁

A 依赖 B,B 依赖 C,那么 A 也间接依赖 C(传递依赖)。当多个传递依赖带来 C 的不同版本时,仲裁规则:

- **路径最短优先**:离根最近的那条路径的版本胜出
- **路径相同时,先声明优先**:pom 里先声明的那个版本胜出

排查依赖树:

```bash
mvn dependency:tree           # 打印完整依赖树
mvn dependency:tree -Dincludes=com.google.guava   # 只看某个依赖
```

### dependencyManagement:统一版本(不实际引入)

`dependencyManagement` 只**声明版本**,不真正引入依赖;子模块要使用时,在 `dependencies` 里写坐标但**不写版本**,继承父 pom 声明的版本:

```xml
<!-- 父 pom -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>33.0.0-jre</version>
        </dependency>
    </dependencies>
</dependencyManagement>

<!-- 子模块:无需写版本 -->
<dependencies>
    <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
    </dependency>
</dependencies>
```

好处:所有模块版本统一,升级只改父 pom 一处。

### BOM:现成的版本清单

Spring Boot 的 `spring-boot-dependencies` 就是一个 BOM(用 `<scope>import</scope>` 引入),里面锁定了所有 starter 的版本:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>3.2.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

所以 Spring Boot 项目里引 starter 通常不写版本,版本由 BOM 管。

### 排除与可选依赖

```xml
<!-- 排除传递依赖中的某个包(如去掉自带日志,换 log4j2) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### 多模块工程(parent + 聚合)

一个父 pom(`<packaging>pom</packaging>`)管理公共依赖,多个子模块各自实现:

```
my-project/                 (父工程,pom)
├── pom.xml                 (packaging=pom,声明 modules)
├── common/                 (公共模块,jar)
├── service/                (业务服务,jar,依赖 common)
└── web/                    (Web 入口,jar,依赖 service)
```

父 pom 关键配置:

```xml
<packaging>pom</packaging>

<modules>
    <module>common</module>
    <module>service</module>
    <module>web</module>
</modules>
```

子模块 pom 通过 `<parent>` 继承父工程,并声明对兄弟模块的依赖:

```xml
<parent>
    <groupId>com.example</groupId>
    <artifactId>my-project</artifactId>
    <version>1.0.0</version>
</parent>

<artifactId>service</artifactId>

<dependencies>
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>common</artifactId>
        <version>1.0.0</version>
    </dependency>
</dependencies>
```

在父工程根目录执行 `mvn clean package`,会按依赖顺序构建所有模块。

### 版本属性管理

用 `<properties>` 集中声明版本,便于引用和升级:

```xml
<properties>
    <java.version>17</java.version>
    <spring-boot.version>3.2.0</spring-boot.version>
    <guava.version>33.0.0-jre</guava.version>
</properties>
```

## 示例

```bash
# 多模块工程目录(常见后端分层)
my-backend/
├── pom.xml                 # 父 pom:统一 Spring Boot 版本、Java 版本
├── my-backend-common/      # 通用:DTO、工具类、异常、枚举
├── my-backend-domain/      # 领域:Entity、Repository 接口
├── my-backend-service/     # 业务逻辑:Service 实现
└── my-backend-web/         # Web 层:Controller,可执行入口

# 根目录一条命令构建全部
mvn clean package -DskipTests
```

## 常见问题与坑

- **`dependencyManagement` 里写依赖但不生效**:它只管版本,子模块还得在 `dependencies` 里显式声明
- **依赖冲突难定位**:`mvn dependency:tree` 是利器,配合 `-Dverbose` 看省略的冲突版本
- **父 pom 忘写 `<packaging>pom</packaging>`**:默认是 jar,聚合/继承会出错
- **子模块版本不统一**:子模块 `<parent>` 里的 version 要和父一致,否则构建报错
- **循环依赖**:模块 A 依赖 B、B 又依赖 A,重构分层解决(依赖方向单向)
- **用 `<dependencyManagement>` 的 `<scope>import</scope>` 忘记 `<type>pom</type>`**:BOM 导入必须两个都写

## 参考资料

- [Maven POM 参考](https://maven.apache.org/pom.html)
- [Maven 多模块指南](https://maven.apache.org/guides/mini/guide-multiple-modules.html)
- [Spring Boot 依赖版本管理](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html)
