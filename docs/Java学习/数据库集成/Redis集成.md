# Redis 集成

## 概述

> Redis 是内存 KV 存储,用于缓存、会话、分布式锁、计数器、排行榜等。本笔记覆盖 Spring Data Redis 的 `RedisTemplate`、Spring Cache 注解缓存,以及缓存三大经典问题(穿透/击穿/雪崩)。

## 核心知识点

### 数据类型与典型场景

| 类型 | 命令 | 场景 |
|------|------|------|
| String | `SET`/`GET`/`INCR` | 缓存、计数器、分布式锁 |
| Hash | `HSET`/`HGET` | 对象缓存、购物车 |
| List | `LPUSH`/`RPOP` | 消息队列、最新列表 |
| Set | `SADD`/`SINTER` | 去重、共同好友 |
| ZSet | `ZADD`/`ZRANGE` | 排行榜、延时队列 |

### 与 Spring Boot 集成

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      password: ${REDIS_PASSWORD}
      timeout: 3000ms
```

Spring Boot 默认客户端是 **Lettuce**(线程安全、异步、连接复用),无需额外配置。

### RedisTemplate

```java
@Bean
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
    RedisTemplate<String, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(factory);
    // key 用 String,value 用 JSON 序列化(默认 JDK 序列化可读性差)
    template.setKeySerializer(new StringRedisSerializer());
    template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
    return template;
}
```

```java
// 简单场景直接用 StringRedisTemplate
stringRedisTemplate.opsForValue().set("user:1", json, Duration.ofMinutes(10));
String json = stringRedisTemplate.opsForValue().get("user:1");

// Hash 操作
stringRedisTemplate.opsForHash().put("cart:1", "sku_001", "2");
```

### Spring Cache 注解缓存

开启缓存 `@EnableCaching`,方法上用注解,自动缓存返回结果:

```java
@Cacheable(cacheNames = "user", key = "#id")          // 有缓存直接用,无则查库并缓存
public User getUser(Long id) { return userMapper.selectById(id); }

@CacheEvict(cacheNames = "user", key = "#id")          // 更新/删除时清缓存
public void deleteUser(Long id) { userMapper.deleteById(id); }

@CachePut(cacheNames = "user", key = "#user.id")       // 更新并刷新缓存
public User updateUser(User user) { return userMapper.updateById(user); }
```

### 缓存三大问题

| 问题 | 现象 | 解决 |
|------|------|------|
| **穿透** | 查不存在的 key,每次都打库 | 缓存空值(短 TTL)、布隆过滤器 |
| **击穿** | 热点 key 过期瞬间大量请求打库 | 热点 key 不过期、互斥锁、逻辑过期 |
| **雪崩** | 大量 key 同时过期/Redis 宕机 | TTL 加随机值、多级缓存、限流降级 |

```java
// 防穿透:缓存空值
public User getUser(Long id) {
    String key = "user:" + id;
    String v = redis.get(key);
    if (v != null) return deserialize(v);
    if (redis.exists(key + ":null")) return null;   // 命中空值缓存
    User u = userMapper.selectById(id);
    if (u == null) redis.set(key + ":null", "1", Duration.ofMinutes(1));  // 空值缓存
    else redis.set(key, serialize(u), Duration.ofMinutes(30) + random);
    return u;
}
```

### 分布式锁

用 Redis 实现跨实例互斥,`SET key value NX PX ttl` 原子加锁:

```java
// 加锁:NX 不存在才设置,PX 毫秒过期
Boolean ok = redis.opsForValue()
    .setIfAbsent("lock:order:" + id, uuid, Duration.ofSeconds(30));
if (Boolean.TRUE.equals(ok)) {
    try {
        // 执行业务
    } finally {
        // 释放:用 Lua 脚本保证「校验 value 是自己 + 删除」原子
        redis.execute(unlockScript, keys, uuid);
    }
}
```

生产更推荐直接用 **Redisson**,封装了可重入锁、看门狗续期、公平锁,避免手写锁的坑。

## 示例

```java
@RestController
@RequestMapping("/api/rank")
public class RankController {
    private final StringRedisTemplate redis;

    @PostMapping("/{key}")
    public void addScore(@PathVariable String key, @RequestParam String member,
                         @RequestParam double score) {
        redis.opsForZSet().incrementScore(key, member, score);   // ZSet 加分
    }

    @GetMapping("/{key}")
    public List<Object> top10(@PathVariable String key) {
        // 取排行榜前 10(分数从高到低)
        return new ArrayList<>(redis.opsForZSet()
            .reverseRange(key, 0, 9));
    }
}
```

## 常见问题与坑

- **默认 JDK 序列化**:`RedisTemplate` 默认序列化可读性差、跨语言难,改 JSON 序列化
- **缓存与数据库一致性**:先更新 DB 再删缓存(而非先删缓存),用 `@CacheEvict` 保证最终一致
- **大 key / 热 key**:单个 key 过大(几 MB)拖慢 Redis,拆 key 或压缩;热 key 用本地缓存兜底
- **缓存不过期也不更新**:脏数据常驻,TTL 要设,更新要 evict
- **分布式锁的误删**:释放时校验 value 是自己,否则可能删了别人的锁(用 Redisson 规避)
- **Redis 挂了拖垮服务**:加连接超时、熔断降级,别让缓存依赖变成单点

## 参考资料

- [Spring Data Redis 文档](https://docs.spring.io/spring-data/redis/reference/)
- [Redisson](https://github.com/redisson/redisson)
- [Redis 笔记](/docs/数据库/Redis.md)
