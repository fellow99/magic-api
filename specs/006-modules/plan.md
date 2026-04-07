# 006-modules 模块技术方案

**模块名称**: 模块集合 (Modules Collection)  
**创建时间**: 2026-03-14  
**状态**: 已完成  
**版本**: 2.2.2

---

## 1. 架构设计

```
┌─────────────────────────────────────────────────────────┐
│                    MagicAPI Modules                      │
├─────────────────┬─────────────────┬─────────────────────┤
│   DBModule      │   HTTPModule    │   ServletException  │
│   (数据库)       │   (HTTP 客户端)   │   (Servlet API)     │
├─────────────────┴─────────────────┴─────────────────────┤
│                   SpringModule                           │
│                   (Spring 集成)                           │
└─────────────────────────────────────────────────────────┘
```

---

## 2. 数据库模块实现

### 2.1 SQL 执行器

```java
@Component
public class SQLExecutor {
    
    @Autowired
    private DataSource dataSource;
    
    @Autowired
    private DialectAdapter dialectAdapter;
    
    public List<Map<String, Object>> select(String sql, Object... params) {
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {
            
            setParams(ps, params);
            ResultSet rs = ps.executeQuery();
            return mapResultSet(rs);
        } catch (SQLException e) {
            throw new DatabaseException(e);
        }
    }
    
    public int update(String sql, Object... params) {
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {
            
            setParams(ps, params);
            return ps.executeUpdate();
        } catch (SQLException e) {
            throw new DatabaseException(e);
        }
    }
    
    public PageResult page(String sql, int page, int size, Object... params) {
        String countSql = dialectAdapter.getCountSql(sql);
        String pageSql = dialectAdapter.getPageSql(sql, page, size);
        
        long total = count(countSql, params);
        List<Map<String, Object>> data = select(pageSql, params);
        
        return new PageResult(total, page, size, data);
    }
}
```

### 2.2 方言适配器

```java
public interface DialectAdapter {
    String getCountSql(String sql);
    String getPageSql(String sql, int page, int size);
}

@Component("mysqlDialect")
public class MySQLDialectAdapter implements DialectAdapter {
    
    @Override
    public String getCountSql(String sql) {
        return "select count(1) from (" + sql + ") tmp";
    }
    
    @Override
    public String getPageSql(String sql, int page, int size) {
        int offset = (page - 1) * size;
        return sql + " limit " + offset + ", " + size;
    }
}

@Component("oracleDialect")
public class OracleDialectAdapter implements DialectAdapter {
    
    @Override
    public String getCountSql(String sql) {
        return "select count(1) from (" + sql + ")";
    }
    
    @Override
    public String getPageSql(String sql, int page, int size) {
        int start = (page - 1) * size + 1;
        int end = page * size;
        return "select * from (select tmp.*, rownum rn from (" + sql + ") tmp where rownum <= " + end + ") where rn >= " + start;
    }
}
```

### 2.3 事务管理

```java
@Component
public class TransactionManager {
    
    @Autowired
    private PlatformTransactionManager transactionManager;
    
    public <T> T execute(TransactionCallback<T> callback) {
        TransactionStatus status = transactionManager.getTransaction(
            new DefaultTransactionDefinition()
        );
        try {
            T result = callback.doInTransaction();
            transactionManager.commit(status);
            return result;
        } catch (Exception e) {
            transactionManager.rollback(status);
            throw new TransactionException(e);
        }
    }
}
```

### 2.4 数据库脚本模块

```java
@MagicModule("db")
public class DBModule {
    
    @Autowired
    private SQLExecutor sqlExecutor;
    
    @Autowired
    private TransactionManager transactionManager;
    
    public List<Map<String, Object>> select(String sql, Object... params) {
        return sqlExecutor.select(sql, params);
    }
    
    public int update(String sql, Object... params) {
        return sqlExecutor.update(sql, params);
    }
    
    public PageResult page(String sql, int page, int size, Object... params) {
        return sqlExecutor.page(sql, page, size, params);
    }
    
    public <T> T transaction(TransactionCallback<T> callback) {
        return transactionManager.execute(callback);
    }
}
```

---

## 3. HTTP 模块实现

```java
@MagicModule("http")
public class HTTPModule {
    
    private final RestTemplate restTemplate = new RestTemplate();
    
    public Object get(String url) {
        return get(url, null);
    }
    
    public Object get(String url, Map<String, Object> options) {
        HttpHeaders headers = extractHeaders(options);
        int timeout = extractTimeout(options);
        
        restTemplate.setRequestFactory(
            new HttpComponentsClientHttpRequestFactory() {{
                setConnectTimeout(timeout);
                setReadTimeout(timeout);
            }}
        );
        
        ResponseEntity<String> response = restTemplate.exchange(
            url, HttpMethod.GET, new HttpEntity<>(headers), String.class
        );
        
        return parseResponse(response);
    }
    
    public Object post(String url, Object body) {
        return post(url, body, null);
    }
    
    public Object post(String url, Object body, Map<String, Object> options) {
        HttpHeaders headers = extractHeaders(options);
        headers.setContentType(MediaType.APPLICATION_JSON);
        
        HttpEntity<Object> entity = new HttpEntity<>(body, headers);
        ResponseEntity<String> response = restTemplate.postForEntity(url, entity, String.class);
        
        return parseResponse(response);
    }
    
    private HttpHeaders extractHeaders(Map<String, Object> options) {
        HttpHeaders headers = new HttpHeaders();
        if (options != null && options.containsKey("headers")) {
            Map<String, String> headerMap = (Map) options.get("headers");
            headerMap.forEach(headers::set);
        }
        return headers;
    }
    
    private int extractTimeout(Map<String, Object> options) {
        if (options != null && options.containsKey("timeout")) {
            return (Integer) options.get("timeout");
        }
        return 30000;
    }
    
    private Object parseResponse(ResponseEntity<String> response) {
        if (response.getStatusCode().is2xxSuccessful()) {
            return JSON.parseObject(response.getBody());
        }
        throw new HTTPException(response.getStatusCodeValue(), response.getBody());
    }
}
```

---

## 4. Servlet 模块实现

```java
@MagicModule("servlet")
public class ServletModule {
    
    public HttpServletRequest getRequest() {
        return RequestContext.get().getRequest();
    }
    
    public HttpServletResponse getResponse() {
        return RequestContext.get().getResponse();
    }
    
    public HttpSession getSession() {
        return getRequest().getSession();
    }
    
    public void download(File file, String filename) throws IOException {
        HttpServletResponse response = getResponse();
        response.setContentType("application/octet-stream");
        response.setHeader("Content-Disposition", 
            "attachment; filename=\"" + filename + "\"");
        
        try (InputStream is = new FileInputStream(file);
             OutputStream os = response.getOutputStream()) {
            IOUtils.copy(is, os);
        }
    }
}
```

---

## 5. Spring 模块实现

```java
@MagicModule("spring")
public class SpringModule implements ApplicationContextAware {
    
    private static ApplicationContext context;
    
    @Override
    public void setApplicationContext(ApplicationContext ctx) {
        context = ctx;
    }
    
    public Object getBean(String name) {
        return context.getBean(name);
    }
    
    public <T> T getBean(Class<T> clazz) {
        return context.getBean(clazz);
    }
    
    public void publishEvent(ApplicationEvent event) {
        context.publishEvent(event);
    }
    
    public Environment getEnv() {
        return context.getEnvironment();
    }
}
```

---

## 6. 验收标准

- [ ] 支持 SQL 查询和更新
- [ ] 支持分页查询
- [ ] 支持事务管理
- [ ] 支持多种数据库方言
- [ ] 支持 HTTP 请求方法
- [ ] 支持请求配置
- [ ] 支持响应处理
- [ ] 支持 Servlet API 访问
- [ ] 支持文件下载
- [ ] 支持 Spring Bean 访问
- [ ] 支持事件发布

---

*文档生成时间：2026-03-14*
