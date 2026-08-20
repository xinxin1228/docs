# Spring Security

## 概述

> Spring Security 提供认证(Authentication,验证身份)与授权(Authorization,判断权限)。本笔记覆盖 Spring Security 6 的过滤器链配置、JWT 无状态认证、密码加密与方法级鉴权。

## 核心知识点

### 认证 vs 授权

- **认证 Authentication**:验证身份(登录、token)
- **授权 Authorization**:验证权限(角色、URL、方法)

### 核心概念

- **SecurityFilterChain**:安全过滤器链,拦截请求做认证/授权
- **UserDetailsService**:按用户名加载用户信息的接口(对接实际的用户表)
- **UserDetails**:用户信息抽象(用户名、密码、权限)
- **PasswordEncoder**:密码加密(BCrypt 是默认推荐)
- **AuthenticationManager**:认证管理器

### 基本配置(Spring Security 6)

Spring Security 6 弃用了 `WebSecurityConfigurerAdapter`,改用声明式 Bean:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())        // 前后端分离 + JWT 通常关掉 CSRF
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()   // 登录注册放行
                .requestMatchers("/actuator/health").permitAll()
                .anyRequest().authenticated())                  // 其余都要认证
            .sessionManagement(s -> s.sessionCreationPolicy(
                SessionCreationPolicy.STATELESS));   // 无状态,用 JWT
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();   // BCrypt 加密
    }
}
```

### 密码加密

```java
// 注册时:存加密后的密文,绝不存明文
String encoded = passwordEncoder.encode("123456");
// $2a$10$xxx...(每次结果不同,含随机盐)

// 登录时:用 matches 校验
boolean ok = passwordEncoder.matches("123456", encoded);
```

BCrypt 自带盐、不可逆、可调节强度(cost),是最稳妥的选择。

### JWT 无状态认证

JWT = Header + Payload + Signature,服务端不存会话,靠签名验签:

```java
// 生成 token
public String generateToken(UserDetails user) {
    return Jwts.builder()
        .setSubject(user.getUsername())
        .claim("roles", user.getAuthorities())
        .setIssuedAt(new Date())
        .setExpiration(new Date(System.currentTimeMillis() + 86400000))  // 1 天
        .signWith(secretKey, SignatureAlgorithm.HS256)
        .compact();
}
```

自定义过滤器,从 `Authorization` 头解析 JWT 并写入安全上下文:

```java
public class JwtAuthFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse resp,
                                    FilterChain chain) throws ServletException, IOException {
        String header = req.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);
            try {
                String username = jwtUtil.parseUsername(token);
                UserDetails user = userDetailsService.loadUserByUsername(username);
                // 校验通过则设置认证信息
                UsernamePasswordAuthenticationToken auth =
                    new UsernamePasswordAuthenticationToken(user, null, user.getAuthorities());
                SecurityContextHolder.getContext().setAuthentication(auth);
            } catch (Exception e) {
                SecurityContextHolder.clearContext();
            }
        }
        chain.doFilter(req, resp);
    }
}
```

在 `filterChain` 里挂载:`http.addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);`

### 方法级鉴权

```java
@EnableMethodSecurity   // 开启方法级安全
public class SecurityConfig { ... }

@RestController
public class UserController {
    @PreAuthorize("hasRole('ADMIN')")          // 仅 ADMIN
    @DeleteMapping("/{id}")
    public void delete(@PathVariable Long id) { ... }

    @PreAuthorize("hasAnyAuthority('user:read')")
    @GetMapping("/{id}")
    public User get(@PathVariable Long id) { ... }
}
```

在业务方法上获取当前用户:`SecurityContextHolder.getContext().getAuthentication()` 或 `@AuthenticationPrincipal UserDetails user`。

## 示例

```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {
    private final AuthenticationManager authManager;
    private final JwtUtil jwtUtil;
    private final UserRepository userRepository;

    @PostMapping("/login")
    public Result<String> login(@RequestBody LoginRequest req) {
        authManager.authenticate(
            new UsernamePasswordAuthenticationToken(req.username(), req.password()));
        User user = userRepository.findByUsername(req.username())
                .orElseThrow(() -> new BizException(401, "用户不存在"));
        return Result.ok(jwtUtil.generateToken(user));
    }
}
```

## 常见问题与坑

- **`WebSecurityConfigurerAdapter` 已废弃**:网上老教程还在用,Spring Security 6 改用 `SecurityFilterChain` Bean
- **CSRF 全开导致 POST 被拒**:前后端分离 + JWT 场景按需关闭 `csrf`
- **密码存明文**:必须 BCrypt 加密存储
- **JWT 不设过期时间**:token 永不过期是安全隐患,设合理 `expiration`
- **JWT 放敏感信息**:payload 只是 base64 编码(非加密),别放密码;配合 HTTPS 传输
- **`SecurityContextHolder` 泄漏**:线程池复用时要 `clearContext()`,避免串号
- **过滤器顺序**:`addFilterBefore` 的位置不对会导致认证不生效

## 参考资料

- [Spring Security 参考文档](https://docs.spring.io/spring-security/reference/)
- [JWT 官网](https://jwt.io/)
- [jjwt 库](https://github.com/jwtk/jjwt)
