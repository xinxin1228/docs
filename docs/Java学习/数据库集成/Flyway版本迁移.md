# Flyway 版本迁移

## 概述

> Flyway 是数据库版本迁移工具,把数据库 schema 的变更像代码一样用「版本化 SQL 脚本」管理,团队协作、环境部署时自动同步表结构。它解决了「每个人的本地库、测试库、生产库结构不一致」的老大难问题。

## 核心知识点

### 为什么需要 Flyway

- 表结构变更可追踪、可回滚、可复现
- 新环境/新人拉代码后,一条命令自动建好库结构
- 变更历史有 `flyway_schema_history` 表记录,谁在哪个版本改了什么一目了然

### 脚本命名规范

放在 `src/main/resources/db/migration/` 下,命名规则:

```
V{版本号}__{描述}.sql       # 版本迁移(一次性,不可修改已执行过的)
R__{描述}.sql               # 可重复迁移(每次内容变化都重新执行)
U{版本号}__{描述}.sql       # 撤销迁移(需要付费版,慎用)
```

```text
db/migration/
├── V1__init_schema.sql
├── V2__add_user_table.sql
├── V3__add_email_column.sql
└── R__refresh_view.sql
```

版本号规则:`V1`、`V1.1`、`V1_1`、`V20240101` 都合法,团队统一一种即可。

### 命令

```bash
mvn flyway:migrate      # 执行迁移(核心)
mvn flyway:info         # 查看已应用/待应用的迁移
mvn flyway:baseline     # 对已有库打基线(标记当前版本为起点)
mvn flyway:validate     # 校验脚本与已执行记录是否一致(检查篡改)
mvn flyway:clean        # 清空所有对象(危险,生产禁用)
mvn flyway:repair       # 修复 checksum 不一致
```

### 与 Spring Boot 集成

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
<!-- 若用 MySQL,还需驱动 -->
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-mysql</artifactId>
</dependency>
```

```yaml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true        # 已有表的老库设为 true,打基线
    baseline-version: 1
    # url/user/password 默认复用 datasource
```

Spring Boot 启动时会**自动执行 Flyway 迁移**(应用启动即同步表结构),这是最常用的方式。

### 关键原则

1. **已应用的迁移脚本绝不修改**:改了会导致 checksum 不一致、迁移失败。要改就写新的 `V{n+1}` 迁移
2. **一次变更一个脚本**:每个迁移只做一件事,粒度小、好回滚
3. **脚本幂等与可重复**:`R__` 脚本要写幂等(如 `CREATE OR REPLACE`)

## 示例

```sql
-- V1__init_schema.sql
CREATE TABLE t_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100),
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

```sql
-- V2__add_age_column.sql(后续需求加字段,写新脚本,不改 V1)
ALTER TABLE t_user ADD COLUMN age INT DEFAULT NULL;
```

## 常见问题与坑

- **改历史迁移脚本导致 checksum 失败**:报 `Validate failed: Migration checksum mismatch`,用 `flyway:repair` 或写新脚本
- **首次对接已有库报错**:库里有表但无迁移记录,设 `baseline-on-migrate: true`
- **生产禁用 `clean`**:`flyway:clean` 会清空所有表,配置里关掉或权限隔离
- **迁移顺序混乱**:版本号要严格递增,`V2` 之后不能再插 `V1.5`(会乱序/部分工具不支持)
- **数据库驱动缺失**:MySQL 8 需要 `flyway-mysql` 模块,否则报数据库不支持
- **Spring Boot 版本与 Flyway 版本兼容性**:Boot 会管理 Flyway 版本,别手动引入冲突版本

## 参考资料

- [Flyway 官方文档](https://documentation.red-gate.com/flyway/)
- [Spring Boot Flyway 集成](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.data-initialization.migration-tool.flyway)
- [MySQL 笔记](/docs/数据库/MySQL.md)
