# 002-datasource 数据源管理模块技术方案

**模块名称**: 数据源管理 (Datasource Management)  
**创建时间**: 2026-03-14  
**状态**: 已完成  
**版本**: 2.2.2

---

## 1. 架构设计

### 1.1 组件架构

```
┌─────────────────────────────────────────────────────┐
│            MagicDataSourceController                 │
│              (REST API Layer)                        │
└─────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────┐
│          MagicDynamicDataSource                      │
│         (Dynamic DataSource Manager)                 │
└─────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  DataSource  │  │  DataSource  │  │  DataSource  │
│   (MySQL)    │  │   (Redis)    │  │   (Mongo)    │
└──────────────┘  └──────────────┘  └──────────────┘
```

---

## 2. 核心实现

### 2.1 动态数据源管理

```java
public class MagicDynamicDataSource extends AbstractRoutingDataSource {
    
    private final Map<Object, Object> targetDataSources = new ConcurrentHashMap<>();
    private DataSource defaultDataSource;
    
    @Override
    protected Object determineCurrentLookupKey() {
        return DataSourceContext.getCurrent();
    }
    
    public void addDataSource(String key, DataSource dataSource) {
        targetDataSources.put(key, dataSource);
        super.setTargetDataSources(targetDataSources);
        super.afterPropertiesSet();
    }
    
    public void removeDataSource(String key) {
        targetDataSources.remove(key);
        super.setTargetDataSources(targetDataSources);
        super.afterPropertiesSet();
    }
    
    public void setDefaultDataSource(DataSource dataSource) {
        this.defaultDataSource = dataSource;
        super.setDefaultTargetDataSource(dataSource);
    }
}
```

### 2.2 数据源上下文

```java
public class DataSourceContext {
    private static final ThreadLocal<String> CONTEXT = new ThreadLocal<>();
    
    public static void set(String dataSourceKey) {
        CONTEXT.set(dataSourceKey);
    }
    
    public static String getCurrent() {
        return CONTEXT.get();
    }
    
    public static void clear() {
        CONTEXT.remove();
    }
}
```

### 2.3 数据源加密

```java
public interface DataSourceEncryptProvider {
    String encrypt(String password);
    String decrypt(String encryptedPassword);
}

@Component
public class DefaultDataSourceEncryptProvider implements DataSourceEncryptProvider {
    
    @Value("${magic-api.datasource.encrypt.key:}")
    private String encryptKey;
    
    @Override
    public String encrypt(String password) {
        if (encryptKey == null || encryptKey.isEmpty()) {
            return password;
        }
        // AES 加密实现
        return AESUtil.encrypt(password, encryptKey);
    }
    
    @Override
    public String decrypt(String encryptedPassword) {
        if (encryptKey == null || encryptKey.isEmpty()) {
            return encryptedPassword;
        }
        return AESUtil.decrypt(encryptedPassword, encryptKey);
    }
}
```

---

## 3. REST API 实现

### 3.1 数据源控制器

```java
@RestController
@RequestMapping("/magic/api/datasource")
public class MagicDataSourceController {
    
    @Autowired
    private MagicDynamicDataSource dynamicDataSource;
    
    @Autowired
    private DataSourceEncryptProvider encryptProvider;
    
    @GetMapping("/list")
    public List<DataSourceInfo> list() {
        return dataSourceService.list();
    }
    
    @GetMapping("/info/{id}")
    public DataSourceInfo info(@PathVariable String id) {
        return dataSourceService.getById(id);
    }
    
    @PostMapping("/save")
    public Result save(@RequestBody DataSourceInfo info) {
        // 加密密码
        info.setPassword(encryptProvider.encrypt(info.getPassword()));
        dataSourceService.save(info);
        return Result.success();
    }
    
    @PostMapping("/delete/{id}")
    public Result delete(@PathVariable String id) {
        dataSourceService.delete(id);
        dynamicDataSource.removeDataSource(id);
        return Result.success();
    }
    
    @PostMapping("/test")
    public Result test(@RequestBody DataSourceInfo info) {
        try {
            // 测试连接
            DataSource dataSource = createDataSource(info);
            Connection conn = dataSource.getConnection();
            conn.close();
            return Result.success("连接成功");
        } catch (Exception e) {
            return Result.error("连接失败：" + e.getMessage());
        }
    }
}
```

---

## 4. 配置管理

```java
@Configuration
@ConfigurationProperties(prefix = "magic-api.datasource")
public class DataSourceProperties {
    
    private String encryptKey;
    private List<DataSourceInfo> datasources = new ArrayList<>();
    
    // Getters and Setters
}
```

---

## 5. 数据源切换

### 5.1 注解方式

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface DataSource {
    String value();
}

@Aspect
@Component
public class DataSourceAspect {
    
    @Around("@annotation(dataSource)")
    public Object around(ProceedingJoinPoint pjp, DataSource dataSource) throws Throwable {
        DataSourceContext.set(dataSource.value());
        try {
            return pjp.proceed();
        } finally {
            DataSourceContext.clear();
        }
    }
}
```

### 5.2 代码方式

```java
public void switchDataSource(String dataSourceKey) {
    DataSourceContext.set(dataSourceKey);
}

public void clearDataSource() {
    DataSourceContext.clear();
}
```

---

## 6. 验收标准

- [ ] 支持配置多个数据源
- [ ] 支持运行时动态切换
- [ ] 支持密码加密存储
- [ ] 提供完整的管理 API

---

*文档生成时间：2026-03-14*
