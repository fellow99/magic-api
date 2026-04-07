# 001-core 核心框架模块技术方案

**模块名称**: 核心框架 (Core Framework)  
**创建时间**: 2026-03-14  
**状态**: 已完成  
**版本**: 2.2.2

---

## 1. 架构设计

### 1.1 分层架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Web Controller Layer                      │
│            (MagicController, MagicControllerRegister)        │
├─────────────────────────────────────────────────────────────┤
│                    Service Layer                             │
│    (MagicAPIService, MagicResourceService,                   │
│     MagicNotifyService, MagicDynamicRegistry)                │
├─────────────────────────────────────────────────────────────┤
│                    Core Layer                                │
│    (RequestContext, SessionContext, Event System,            │
│     Interceptor Chain, Exception Handler)                    │
├─────────────────────────────────────────────────────────────┤
│                    Model Layer                               │
│    (ApiInfo, MagicEntity, Group, Parameter, Header, Path)    │
├─────────────────────────────────────────────────────────────┤
│                    Resource Layer                            │
│    (FileResource, DatabaseResource, JarResource,             │
│     KeyValueResource, ZipResource)                           │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 核心组件关系

```
┌──────────────────┐     ┌──────────────────┐
│  MagicController │────▶│MagicDynamicRegistry│
└──────────────────┘     └──────────────────┘
         │                        │
         ▼                        ▼
┌──────────────────┐     ┌──────────────────┐
│RequestInterceptor│────▶│  Script Engine   │
└──────────────────┘     └──────────────────┘
         │                        │
         ▼                        ▼
┌──────────────────┐     ┌──────────────────┐
│  RequestContext  │────▶│  ResultProvider  │
└──────────────────┘     └──────────────────┘
```

---

## 2. 核心组件设计

### 2.1 动态接口注册机制

**设计目标**: 支持运行时动态添加、更新、删除 HTTP 接口，无需重启应用。

**实现方案**:
```java
public interface MagicDynamicRegistry {
    /**
     * 注册新接口
     */
    void register(ApiInfo apiInfo);
    
    /**
     * 注销接口
     */
    void unregister(String path);
    
    /**
     * 更新接口
     */
    void update(ApiInfo apiInfo);
    
    /**
     * 列出所有接口
     */
    List<ApiInfo> list();
}
```

**实现类**: `RequestMagicDynamicRegistry`

**关键实现**:
- 使用 `ConcurrentHashMap` 存储接口定义，支持并发访问
- 接口变更时触发 `FileEvent` 事件，通知集群节点
- 支持接口版本管理，保留历史版本

---

### 2.2 请求处理流程

**流程说明**:
```
HTTP Request
    │
    ▼
┌─────────────────────┐
│  MagicController    │  ← 接收所有接口请求
└─────────────────────┘
    │
    ▼
┌─────────────────────┐
│ RequestInterceptor  │  ← 拦截器链处理
│  preHandle()        │
└─────────────────────┘
    │
    ▼
┌─────────────────────┐
│  RequestContext     │  ← 初始化请求上下文
│  (ThreadLocal)      │
└─────────────────────┘
    │
    ▼
┌─────────────────────┐
│  Script Engine      │  ← 执行接口脚本
│  (magic-script)     │
└─────────────────────┘
    │
    ▼
┌─────────────────────┐
│ RequestInterceptor  │  ← 后处理
│  postHandle()       │
└─────────────────────┘
    │
    ▼
┌─────────────────────┐
│  ResultProvider     │  ← 格式化响应
└─────────────────────┘
    │
    ▼
┌─────────────────────┐
│ RequestInterceptor  │  ← 完成处理
│  afterCompletion()  │
└─────────────────────┘
    │
    ▼
HTTP Response
```

---

### 2.3 上下文管理

**设计目标**: 在请求处理过程中维护请求信息、会话信息、用户信息。

**实现方案**:

#### RequestContext (请求上下文)
```java
public class RequestContext {
    private static final ThreadLocal<RequestContext> LOCAL = 
        new ThreadLocal<>();
    
    private HttpServletRequest request;
    private HttpServletResponse response;
    private ApiInfo apiInfo;
    private MagicUser user;
    private Map<String, Object> variables;
    
    // 静态方法获取当前请求上下文
    public static RequestContext get() { ... }
    public static void set(RequestContext context) { ... }
    public static void remove() { ... }
}
```

#### SessionContext (会话上下文)
```java
public interface SessionContext {
    /**
     * 获取会话属性
     */
    Object getAttribute(String name);
    
    /**
     * 设置会话属性
     */
    void setAttribute(String name, Object value);
    
    /**
     * 移除会话属性
     */
    void removeAttribute(String name);
    
    /**
     * 销毁会话
     */
    void invalidate();
}
```

---

### 2.4 事件驱动架构

**设计目标**: 使用事件机制解耦组件，支持接口变更通知集群。

**实现方案**:
```java
// 基础事件类
public class MagicEvent extends ApplicationEvent {
    private EventAction action;  // CREATE, UPDATE, DELETE
    private String type;          // 事件类型
    private Object data;          // 事件数据
    
    public MagicEvent(Object source, EventAction action, 
                      String type, Object data) {
        super(source);
        this.action = action;
        this.type = type;
        this.data = data;
    }
}

// 文件事件
public class FileEvent extends MagicEvent {
    private String fileId;
    private String content;
    
    public FileEvent(Object source, EventAction action,
                     String fileId, String content) {
        super(source, action, "file", null);
        this.fileId = fileId;
        this.content = content;
    }
}

// 分组事件
public class GroupEvent extends MagicEvent {
    private Group group;
    
    public GroupEvent(Object source, EventAction action,
                      Group group) {
        super(source, action, "group", null);
        this.group = group;
    }
}
```

**事件发布**:
```java
@Autowired
private ApplicationEventPublisher eventPublisher;

// 发布事件
eventPublisher.publishEvent(new FileEvent(this, 
    EventAction.UPDATE, fileId, content));
```

---

### 2.5 拦截器链

**设计目标**: 提供请求处理的扩展点，支持认证、日志、权限等功能。

**实现方案**:
```java
public interface RequestInterceptor {
    /**
     * 请求前处理
     * @return true 继续处理，false 中断请求
     */
    boolean preHandle(RequestContext context);
    
    /**
     * 请求后处理
     */
    void postHandle(RequestContext context, Object result);
    
    /**
     * 完成后处理
     */
    void afterCompletion(RequestContext context, Exception ex);
}
```

**拦截器链执行**:
```java
public class InterceptorChain {
    private List<RequestInterceptor> interceptors = new ArrayList<>();
    
    public boolean preHandle(RequestContext context) {
        for (RequestInterceptor interceptor : interceptors) {
            if (!interceptor.preHandle(context)) {
                return false;  // 中断请求
            }
        }
        return true;
    }
    
    public void postHandle(RequestContext context, Object result) {
        for (RequestInterceptor interceptor : interceptors) {
            interceptor.postHandle(context, result);
        }
    }
    
    public void afterCompletion(RequestContext context, Exception ex) {
        for (RequestInterceptor interceptor : interceptors) {
            interceptor.afterCompletion(context, ex);
        }
    }
}
```

---

### 2.6 异常处理

**设计目标**: 统一处理异常，提供友好的错误响应。

**实现方案**:
```java
@RestControllerAdvice
public class MagicExceptionHandler {
    
    @ExceptionHandler(MagicAPIException.class)
    @ResponseBody
    public Map<String, Object> handleMagicAPIException(
            MagicAPIException ex, RequestContext context) {
        Map<String, Object> result = new HashMap<>();
        result.put("code", ex.getCode());
        result.put("message", ex.getMessage());
        result.put("timestamp", System.currentTimeMillis());
        return result;
    }
    
    @ExceptionHandler(Exception.class)
    @ResponseBody
    public Map<String, Object> handleException(
            Exception ex, RequestContext context) {
        Map<String, Object> result = new HashMap<>();
        result.put("code", 500);
        result.put("message", "Internal Server Error");
        result.put("error", ex.getMessage());
        result.put("timestamp", System.currentTimeMillis());
        return result;
    }
}
```

---

### 2.7 资源存储抽象

**设计目标**: 提供统一的资源存储接口，支持多种存储方式。

**接口定义**:
```java
public interface MagicResourceStorage {
    /**
     * 列出所有接口
     */
    List<ApiInfo> list();
    
    /**
     * 获取接口
     */
    ApiInfo get(String id);
    
    /**
     * 保存接口
     */
    void save(ApiInfo apiInfo);
    
    /**
     * 删除接口
     */
    void delete(String id);
    
    /**
     * 是否存在
     */
    boolean exists(String id);
}
```

**实现类**:
- `FileResourceStorage`: 文件系统存储
- `DatabaseResourceStorage`: 数据库存储
- `JarResourceStorage`: JAR 包资源
- `KeyValueResourceStorage`: KV 存储 (Redis 等)
- `ZipResourceStorage`: ZIP 包资源

---

## 3. 关键实现

### 3.1 接口动态注册

```java
@Component
public class RequestMagicDynamicRegistry implements MagicDynamicRegistry {
    
    private final ConcurrentHashMap<String, ApiInfo> apiMap = 
        new ConcurrentHashMap<>();
    
    @Autowired
    private ApplicationEventPublisher eventPublisher;
    
    @Override
    public void register(ApiInfo apiInfo) {
        apiMap.put(apiInfo.getPath(), apiInfo);
        // 发布事件通知集群
        eventPublisher.publishEvent(new FileEvent(this, 
            EventAction.CREATE, apiInfo.getId(), apiInfo.getScript()));
    }
    
    @Override
    public void unregister(String path) {
        apiMap.remove(path);
        eventPublisher.publishEvent(new FileEvent(this, 
            EventAction.DELETE, path, null));
    }
    
    @Override
    public void update(ApiInfo apiInfo) {
        apiMap.put(apiInfo.getPath(), apiInfo);
        eventPublisher.publishEvent(new FileEvent(this, 
            EventAction.UPDATE, apiInfo.getId(), apiInfo.getScript()));
    }
    
    @Override
    public List<ApiInfo> list() {
        return new ArrayList<>(apiMap.values());
    }
}
```

---

### 3.2 请求处理核心

```java
@RestController
public class MagicController {
    
    @Autowired
    private MagicDynamicRegistry registry;
    
    @Autowired
    private InterceptorChain interceptorChain;
    
    @Autowired
    private ScriptEngine scriptEngine;
    
    @Autowired
    private ResultProvider resultProvider;
    
    @RequestMapping(value = "**", method = {
        RequestMethod.GET, RequestMethod.POST,
        RequestMethod.PUT, RequestMethod.DELETE
    })
    @ResponseBody
    public Object handle(HttpServletRequest request,
                         HttpServletResponse response,
                         HttpMethod method) {
        // 1. 创建请求上下文
        RequestContext context = new RequestContext(request, response);
        RequestContext.set(context);
        
        try {
            // 2. 查找接口
            String path = request.getRequestURI();
            ApiInfo apiInfo = findApiInfo(path, method);
            if (apiInfo == null) {
                throw new MagicResourceNotFoundException(path);
            }
            context.setApiInfo(apiInfo);
            
            // 3. 拦截器前处理
            if (!interceptorChain.preHandle(context)) {
                return null;  // 已中断
            }
            
            // 4. 执行脚本
            Object result = scriptEngine.execute(apiInfo.getScript(), context);
            
            // 5. 拦截器后处理
            interceptorChain.postHandle(context, result);
            
            // 6. 格式化响应
            return resultProvider.provide(result, context);
            
        } catch (Exception ex) {
            interceptorChain.afterCompletion(context, ex);
            throw ex;
        } finally {
            RequestContext.remove();
        }
    }
    
    private ApiInfo findApiInfo(String path, HttpMethod method) {
        List<ApiInfo> apis = registry.list();
        for (ApiInfo api : apis) {
            if (matchPath(path, api.getPath()) && 
                matchMethod(method, api.getMethod())) {
                return api;
            }
        }
        return null;
    }
}
```

---

### 3.3 配置属性绑定

```java
@ConfigurationProperties(prefix = "magic-api")
public class MagicAPIProperties {
    
    /**
     * Web 页面入口
     */
    private String web = "/magic/web";
    
    /**
     * 接口路径前缀
     */
    private String prefix = "";
    
    /**
     * 密钥
     */
    private String secretKey;
    
    /**
     * 安全配置
     */
    private Security security = new Security();
    
    /**
     * 缓存配置
     */
    private Cache cache = new Cache();
    
    /**
     * 备份配置
     */
    private Backup backup = new Backup();
    
    // Getters and Setters...
    
    public static class Security {
        private boolean enabled = true;
        private String tokenTimeout = "30m";
        private Cors cors = new Cors();
        // Getters and Setters...
    }
    
    public static class Cache {
        private boolean enabled = true;
        private long expire = 3600;
        // Getters and Setters...
    }
    
    public static class Backup {
        private boolean enabled = true;
        private int keepVersions = 10;
        // Getters and Setters...
    }
}
```

---

## 4. 性能优化

### 4.1 缓存策略

**接口定义缓存**:
- 使用 `ConcurrentHashMap` 缓存接口定义
- 支持接口变更时自动失效

**脚本编译缓存**:
- 脚本首次编译后缓存编译结果
- 脚本变更时重新编译

**会话缓存**:
- 使用 `ConcurrentHashMap` 存储会话
- 支持会话超时自动清理

### 4.2 并发控制

**读写锁**:
```java
private final ReadWriteLock lock = new ReentrantReadWriteLock();

public ApiInfo get(String path) {
    lock.readLock().lock();
    try {
        return apiMap.get(path);
    } finally {
        lock.readLock().unlock();
    }
}

public void update(ApiInfo apiInfo) {
    lock.writeLock().lock();
    try {
        apiMap.put(apiInfo.getPath(), apiInfo);
    } finally {
        lock.writeLock().unlock();
    }
}
```

### 4.3 懒加载

- 脚本按需编译，不预加载所有脚本
- 资源按需加载，支持延迟初始化

---

## 5. 安全设计

### 5.1 认证机制

**Session 认证**:
```java
public class DefaultAuthorizationInterceptor 
        implements AuthorizationInterceptor {
    
    @Override
    public boolean preHandle(RequestContext context) {
        // 检查 Session 中是否有用户信息
        MagicUser user = context.getSession().getUser();
        if (user == null) {
            // 未登录，返回错误
            context.getResponse().setStatus(401);
            return false;
        }
        return true;
    }
}
```

**Token 认证**:
```java
@Override
public boolean preHandle(RequestContext context) {
    String token = context.getHeader("Authorization");
    if (token == null || !tokenService.validate(token)) {
        context.getResponse().setStatus(401);
        return false;
    }
    return true;
}
```

### 5.2 权限控制

**接口级权限**:
```java
public class ApiInfo {
    private String path;
    private String method;
    private String script;
    private List<String> roles;  // 允许的角色
    private List<String> permissions;  // 允许的权限
    
    public boolean hasPermission(MagicUser user) {
        if (roles == null || roles.isEmpty()) {
            return true;  // 无限制
        }
        for (String role : roles) {
            if (user.hasRole(role)) {
                return true;
            }
        }
        return false;
    }
}
```

### 5.3 安全防护

**CORS 配置**:
```java
@Configuration
public class MagicCorsFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest request, 
                         ServletResponse response,
                         FilterChain chain) {
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        httpResponse.setHeader("Access-Control-Allow-Origin", "*");
        httpResponse.setHeader("Access-Control-Allow-Methods", 
            "GET, POST, PUT, DELETE, OPTIONS");
        httpResponse.setHeader("Access-Control-Allow-Headers", 
            "Content-Type, Authorization");
        chain.doFilter(request, response);
    }
}
```

---

## 6. 扩展点

### 6.1 拦截器扩展

**实现自定义拦截器**:
```java
@Component
public class CustomInterceptor implements RequestInterceptor {
    
    @Override
    public boolean preHandle(RequestContext context) {
        // 自定义前处理逻辑
        log.info("Request: " + context.getRequest().getRequestURI());
        return true;
    }
    
    @Override
    public void postHandle(RequestContext context, Object result) {
        // 自定义后处理逻辑
        log.info("Response: " + result);
    }
    
    @Override
    public void afterCompletion(RequestContext context, Exception ex) {
        // 自定义完成处理逻辑
        if (ex != null) {
            log.error("Error", ex);
        }
    }
}
```

### 6.2 结果提供者扩展

**实现自定义响应格式**:
```java
@Component
public class CustomResultProvider implements ResultProvider {
    
    @Override
    public Object provide(Object result, RequestContext context) {
        Map<String, Object> response = new HashMap<>();
        response.put("success", true);
        response.put("data", result);
        response.put("timestamp", System.currentTimeMillis());
        return response;
    }
}
```

### 6.3 资源存储扩展

**实现自定义存储**:
```java
@Component
public class RedisResourceStorage implements MagicResourceStorage {
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    @Override
    public List<ApiInfo> list() {
        Set<String> keys = redisTemplate.keys("magic-api:*");
        List<ApiInfo> result = new ArrayList<>();
        for (String key : keys) {
            String json = redisTemplate.opsForValue().get(key);
            result.add(JSON.parseObject(json, ApiInfo.class));
        }
        return result;
    }
    
    @Override
    public void save(ApiInfo apiInfo) {
        String key = "magic-api:" + apiInfo.getId();
        redisTemplate.opsForValue().set(key, 
            JSON.toJSONString(apiInfo));
    }
    
    // 其他方法实现...
}
```

---

## 7. 部署考虑

### 7.1 单机部署

**配置**:
```properties
magic-api.web=/magic/web
magic-api.secret-key=your-secret-key
magic-api.resource.location=/data/magic-api
```

### 7.2 集群部署

**配置**:
```properties
# 所有节点必须配置相同的 secret-key
magic-api.secret-key=cluster-secret-key

# 启用集群插件
magic-api.plugin.cluster.enabled=true

# 使用共享存储 (如 Redis)
magic-api.resource.type=redis
magic-api.resource.redis.host=redis-cluster-host
```

**注意事项**:
- 所有节点必须配置相同的 `secret-key`
- 使用共享存储或启用集群插件进行接口同步
- 确保事件通知机制正常工作

---

## 8. 监控与日志

### 8.1 日志级别

| 级别 | 说明 | 示例 |
|------|------|------|
| DEBUG | 调试信息 | 请求详情、脚本执行日志 |
| INFO | 运行信息 | 接口注册、用户登录 |
| WARN | 警告信息 | 配置警告、性能警告 |
| ERROR | 错误信息 | 异常堆栈、系统错误 |

### 8.2 监控指标

**建议监控的指标**:
- 接口调用次数 (按接口分组)
- 接口响应时间 (P50, P95, P99)
- 错误率 (按错误类型分组)
- 活跃会话数
- 缓存命中率

**监控实现**:
```java
@Component
public class MagicMetrics {
    
    private final MeterRegistry meterRegistry;
    
    public MagicMetrics(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        
        // 注册指标
        meterRegistry.counter("magic.api.requests.total");
        meterRegistry.timer("magic.api.requests.duration");
        meterRegistry.gauge("magic.api.sessions.active", 
            SessionContext.getActiveCount());
    }
    
    public void recordRequest(String path, long duration, 
                              boolean success) {
        meterRegistry.counter("magic.api.requests.total", 
            "path", path, "success", String.valueOf(success)).increment();
        meterRegistry.timer("magic.api.requests.duration")
            .record(duration, TimeUnit.MILLISECONDS);
    }
}
```

---

## 9. 测试策略

### 9.1 单元测试

**测试范围**:
- 核心类的方法逻辑
- 配置属性绑定
- 数据模型序列化

**示例**:
```java
@SpringBootTest
public class MagicDynamicRegistryTest {
    
    @Autowired
    private MagicDynamicRegistry registry;
    
    @Test
    public void testRegister() {
        ApiInfo apiInfo = new ApiInfo();
        apiInfo.setPath("/test");
        apiInfo.setMethod("GET");
        apiInfo.setScript("return 'hello'");
        
        registry.register(apiInfo);
        
        List<ApiInfo> apis = registry.list();
        assertEquals(1, apis.size());
        assertEquals("/test", apis.get(0).getPath());
    }
}
```

### 9.2 集成测试

**测试范围**:
- 接口动态注册和调用
- 拦截器链执行
- 异常处理

### 9.3 性能测试

**测试场景**:
- 并发注册接口
- 高并发请求处理
- 大数据量接口列表

---

## 10. 依赖管理

### 10.1 Maven 依赖

```xml
<dependencies>
    <!-- Spring Boot -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.4.5</version>
    </dependency>
    
    <!-- Magic Script -->
    <dependency>
        <groupId>org.ssssssss</groupId>
        <artifactId>magic-script</artifactId>
        <version>1.9.0</version>
    </dependency>
    
    <!-- FastJSON -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.83</version>
    </dependency>
    
    <!-- SLF4J -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.36</version>
    </dependency>
</dependencies>
```

---

## 11. 验收标准

### 11.1 功能验收

- [ ] 支持动态注册、更新、删除 HTTP 接口
- [ ] 支持接口脚本即时生效
- [ ] 支持自定义拦截器
- [ ] 支持自定义结果提供者
- [ ] 支持多种资源存储方式
- [ ] 支持事件驱动架构
- [ ] 支持集群部署

### 11.2 性能验收

- [ ] 接口注册延迟 < 100ms
- [ ] 核心框架处理开销 < 50ms
- [ ] 单节点支持 1000+ QPS
- [ ] 内存占用 < 200MB (空载)

### 11.3 质量验收

- [ ] 代码符合 Java 编码规范
- [ ] 关键类有完整的 Javadoc 注释
- [ ] 单元测试覆盖率 > 80%
- [ ] 通过 SonarQube 质量门禁

---

## 12. 参考资料

- [magic-api 官方文档](https://ssssssss.org)
- [Spring Boot 文档](https://spring.io/projects/spring-boot)
- [JSR-223 规范](https://www.jcp.org/en/jsr/detail?id=223)

---

*文档生成时间：2026-03-14*  
*最后更新：2026-03-14*
