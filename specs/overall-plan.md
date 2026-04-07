# magic-api 整体技术方案

> 版本：2.2.2  
> 生成时间：2026-03-14  
> 文档类型：技术方案

---

## 1. 文档概述

### 1.1 文档目的

本文档描述 magic-api 项目的整体技术方案，包括架构设计、技术选型、实现策略、部署方案等，为开发实施提供指导。

### 1.2 适用范围

- 架构师：了解整体架构设计
- 开发人员：了解技术实现细节
- 运维人员：了解部署和运维方案

---

## 2. 架构设计

### 2.1 系统架构

```
┌─────────────────────────────────────────────────────────────────┐
│                         客户端层                                 │
│  Web 浏览器  │  HTTP Client  │  Mobile App  │  第三方系统        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼ HTTP/WebSocket
┌─────────────────────────────────────────────────────────────────┐
│                         接入层                                   │
│              Spring Boot Web 容器                                │
│  ┌─────────────────────┐  ┌─────────────────────────────────┐   │
│  │  RequestHandler     │  │  MagicWebSocketDispatcher       │   │
│  │  (HTTP 请求入口)     │  │  (WebSocket 消息分发)            │   │
│  └─────────────────────┘  └─────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                         核心层                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  MagicConfiguration (配置管理)                           │   │
│  └─────────────────────────────────────────────────────────┘   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │ RequestMagic │  │  Function    │  │   MagicResource      │  │
│  │ DynamicReg-  │  │ DynamicReg-  │  │   Service            │  │
│  │ istry        │  │ istry        │  │   (资源管理)          │  │
│  │ (请求注册)   │  │ (函数注册)   │  │                      │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  RequestInterceptor Chain (拦截器链)                     │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        执行层                                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              MagicScript Engine                          │   │
│  │         (magic-script v1.9.0 脚本引擎)                    │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        模块层                                    │
│  DB Module  │  HTTP Module  │  Servlet Module  │  Spring Module │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        数据层                                    │
│  MySQL  │  Oracle  │  PostgreSQL  │  Redis  │  MongoDB  │  ...  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 技术架构分层

| 层级 | 职责 | 关键技术 |
|------|------|---------|
| 客户端层 | 用户交互、API 调用 | 浏览器、HTTP Client、Mobile SDK |
| 接入层 | 请求接入、协议转换 | Spring Boot、WebSocket |
| 核心层 | 业务核心逻辑 | Spring、magic-script |
| 执行层 | 脚本解析执行 | magic-script Engine |
| 模块层 | 功能扩展 | Java 模块系统 |
| 数据层 | 数据存储 | MySQL、Redis、MongoDB 等 |

### 2.3 逻辑架构

```
┌─────────────────────────────────────────────────────────────────┐
│                      用户界面 (UI Layer)                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              magic-editor (Web 编辑器)                    │   │
│  │  • 代码编辑  • 接口管理  • 在线调试  • 数据源配置        │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     API 网关层 (Gateway Layer)                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              RequestHandler                              │   │
│  │  • 路由匹配  • 参数解析  • 验证  • 拦截器                │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    业务逻辑层 (Business Layer)                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              MagicScript Engine                          │   │
│  │  • 脚本解析  • 变量管理  • 模块调用  • 异常处理          │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    数据访问层 (DAL Layer)                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              SQLModule / HTTPModule                      │   │
│  │  • SQL 执行  • HTTP 调用  • 事务管理  • 缓存              │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. 技术选型

### 3.1 核心技术栈

| 技术 | 选型 | 版本 | 用途 | 选型理由 |
|------|------|------|------|---------|
| 编程语言 | Java | 1.8+ | 后端开发 | 成熟稳定，生态丰富 |
| 核心框架 | Spring Boot | 2.4.5 | 应用框架 | 自动配置，开箱即用 |
| 脚本引擎 | magic-script | 1.9.0 | 动态脚本 | 自研，深度集成 |
| JSON 处理 | FastJSON | 1.2.83 | JSON 序列化 | 性能优异，API 简洁 |
| 构建工具 | Maven | 3.6+ | 项目构建 | 标准构建工具 |
| Web 容器 | Tomcat | 内嵌 | HTTP 服务 | Spring Boot 默认 |

### 3.2 前端技术栈

| 技术 | 选型 | 用途 |
|------|------|------|
| 框架 | Vue.js | 前端框架 |
| 编辑器 | Monaco Editor | 代码编辑器 |
| HTTP | Axios | HTTP 客户端 |
| 构建 | Webpack | 打包工具 |

### 3.3 数据库支持

| 类型 | 数据库 | 支持方式 |
|------|--------|---------|
| 关系型 | MySQL, Oracle, PostgreSQL, SQL Server, DB2 | 内置支持 |
| 关系型 | Clickhouse, 达梦，人大金仓 | 内置支持 |
| NoSQL | Redis, MongoDB, Elasticsearch | 插件支持 |
| 图数据库 | Nebula Graph | 插件支持 |

### 3.4 中间件支持

| 中间件 | 用途 | 支持方式 |
|--------|------|---------|
| Redis | 缓存、分布式锁 | 插件支持 |
| Elasticsearch | 搜索 | 插件支持 |
| RabbitMQ/Kafka | 消息队列 | 自定义扩展 |

---

## 4. 核心实现策略

### 4.1 动态路由注册

**目标：** 实现运行时动态注册 HTTP 接口

**实现方案：**

```java
@Configuration
public class MagicServletConfiguration {
    
    @Autowired
    private RequestMappingHandlerMapping requestMappingHandlerMapping;
    
    /**
     * 注册接口映射
     */
    public void registerMapping(String path, String method, Object handler, Method methodObj) {
        RequestMappingInfo info = RequestMappingInfo
            .paths(path)
            .methods(RequestMethod.valueOf(method))
            .build();
        requestMappingHandlerMapping.registerMapping(info, handler, methodObj);
    }
    
    /**
     * 删除接口映射
     */
    public void removeMapping(String path, String method) {
        RequestMappingInfo info = RequestMappingInfo
            .paths(path)
            .methods(RequestMethod.valueOf(method))
            .build();
        requestMappingHandlerMapping.unregisterMapping(info);
    }
}
```

**关键点：**
- 利用 Spring MVC 的 `RequestMappingHandlerMapping` 扩展点
- 支持运行时动态注册和删除
- 与 Spring MVC 完整兼容（拦截器、消息转换器等）

### 4.2 脚本执行引擎

**目标：** 实现 magic-script 脚本的解析和执行

**实现方案：**

```java
public class ScriptManager {
    
    /**
     * 执行脚本
     */
    public static Object executeScript(String script, MagicScriptContext context) {
        // 1. 获取或创建脚本实例（带缓存）
        MagicScriptInstance instance = MagicScript.getScriptInstance(script);
        
        // 2. 执行脚本
        return instance.execute(context);
    }
    
    /**
     * 执行表达式
     */
    public static Object executeExpression(String expression, MagicScriptContext context) {
        return MagicScript.executeExpression(expression, context);
    }
}
```

**脚本编译缓存：**

```java
public class MagicScript {
    
    // 编译缓存：LRU Cache
    private static final Cache<String, MagicScriptInstance> CACHE = 
        new LruCache<>(compileCacheSize);
    
    public static MagicScriptInstance getScriptInstance(String script) {
        MagicScriptInstance instance = CACHE.get(script);
        if (instance == null) {
            instance = compile(script);
            CACHE.put(script, instance);
        }
        return instance;
    }
}
```

### 4.3 多数据源管理

**目标：** 支持运行时动态切换数据源

**实现方案：**

```java
public class MagicDynamicDataSource extends AbstractRoutingDataSource {
    
    // ThreadLocal 存储当前数据源
    private static final ThreadLocal<String> CONTEXT_HOLDER = new ThreadLocal<>();
    
    @Override
    protected Object determineCurrentLookupKey() {
        return CONTEXT_HOLDER.get();
    }
    
    public void setDataSource(String dataSourceName) {
        CONTEXT_HOLDER.set(dataSourceName);
    }
    
    public void clearDataSource() {
        CONTEXT_HOLDER.remove();
    }
}
```

**使用方式：**

```javascript
// 切换数据源
db.useDataSource("slave");
var list = db.selectList("SELECT * FROM user");

// 自动恢复
db.useDataSource("master");
```

### 4.4 事务管理

**目标：** 支持脚本级别的事务控制

**实现方案：**

```java
public class Transaction {
    
    @Autowired
    private PlatformTransactionManager transactionManager;
    
    public <T> T execute(TransactionCallback<T> callback) {
        DefaultTransactionDefinition def = new DefaultTransactionDefinition();
        def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
        
        TransactionStatus status = transactionManager.getTransaction(def);
        try {
            T result = callback.doInTransaction();
            transactionManager.commit(status);
            return result;
        } catch (Exception ex) {
            transactionManager.rollback(status);
            throw ex;
        }
    }
}
```

**脚本使用：**

```javascript
db.transaction(() => {
    db.insert("INSERT INTO user (name) VALUES (?)", ["张三"]);
    db.insert("INSERT INTO order (user_id, amount) VALUES (?, ?)", [1, 100]);
});
```

### 4.5 拦截器链

**目标：** 支持请求处理的前后拦截

**实现方案：**

```java
public interface RequestInterceptor {
    
    /**
     * 前置拦截
     * @return null 继续执行，非 null 直接返回
     */
    Object preHandle(RequestEntity requestEntity) throws Exception;
    
    /**
     * 后置拦截
     * @return null 继续执行，非 null 作为返回值
     */
    Object postHandle(RequestEntity requestEntity, Object returnValue) throws Exception;
    
    /**
     * 完成后拦截
     */
    void afterCompletion(RequestEntity requestEntity, Object returnValue, Throwable ex);
}
```

**拦截器执行流程：**

```java
// 前置拦截
for (RequestInterceptor interceptor : interceptors) {
    Object result = interceptor.preHandle(requestEntity);
    if (result != null) {
        return result; // 拦截，直接返回
    }
}

// 执行脚本
Object value = executeScript(...);

// 后置拦截
for (RequestInterceptor interceptor : interceptors) {
    Object result = interceptor.postHandle(requestEntity, value);
    if (result != null) {
        value = result; // 修改返回值
    }
}

// 完成后
for (RequestInterceptor interceptor : interceptors) {
    interceptor.afterCompletion(requestEntity, value, null);
}
```

### 4.6 WebSocket 实时通信

**目标：** 支持前端编辑器的实时调试和协作

**实现方案：**

```java
@ServerEndpoint("/magic/web/console")
public class MagicWebSocketDispatcher extends WebSocketHandler {
    
    // 会话管理
    private static final Map<String, WebSocketSession> sessions = new ConcurrentHashMap<>();
    
    @Override
    public void afterConnectionEstablished(WebSocketSession session) {
        String clientId = generateClientId();
        sessions.put(clientId, session);
    }
    
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) {
        // 解析消息
        MagicNotify notify = parseMessage(message.getPayload());
        
        // 分发处理
        handlers.get(notify.getType()).handle(notify);
    }
    
    // 发送消息到指定客户端
    public static void sendByClientId(String clientId, String content) {
        WebSocketSession session = sessions.get(clientId);
        if (session != null && session.isOpen()) {
            session.sendMessage(new TextMessage(content));
        }
    }
}
```

### 4.7 资源存储

**目标：** 支持文件和数据库两种存储方式

**实现方案：**

```java
public interface Resource {
    InputStream get(String id) throws IOException;
    void save(String id, InputStream inputStream) throws IOException;
    void delete(String id) throws IOException;
    List<String> list(String path) throws IOException;
}

// 文件存储实现
public class FileResource implements Resource {
    private final String basePath;
    
    @Override
    public InputStream get(String id) throws IOException {
        return new FileInputStream(new File(basePath, id + ".sql"));
    }
    
    @Override
    public void save(String id, InputStream inputStream) throws IOException {
        File file = new File(basePath, id + ".sql");
        FileUtils.copyInputStreamToFile(inputStream, file);
    }
}

// 数据库存储实现
public class DatabaseResource implements Resource {
    private final JdbcTemplate jdbcTemplate;
    private final String tableName;
    
    @Override
    public InputStream get(String id) throws IOException {
        byte[] content = jdbcTemplate.queryForObject(
            "SELECT content FROM " + tableName + " WHERE id = ?", 
            byte[].class, id
        );
        return new ByteArrayInputStream(content);
    }
}
```

### 4.8 备份还原

**目标：** 支持接口备份和还原

**实现方案：**

```java
public interface MagicBackupService {
    
    /**
     * 创建备份
     */
    void backup(String resourceId, String content);
    
    /**
     * 获取备份列表
     */
    List<BackupRecord> listBackups(String resourceId);
    
    /**
     * 还原备份
     */
    void rollback(String backupId);
    
    /**
     * 清理过期备份
     */
    long removeBackupByTimestamp(long timestamp);
}
```

**备份表结构：**

```sql
CREATE TABLE magic_api_backup (
    id VARCHAR(64) PRIMARY KEY,
    resource_id VARCHAR(64) NOT NULL,
    content TEXT NOT NULL,
    backup_time DATETIME NOT NULL,
    operator VARCHAR(64),
    remark VARCHAR(255)
);

CREATE INDEX idx_resource_id ON magic_api_backup(resource_id);
CREATE INDEX idx_backup_time ON magic_api_backup(backup_time);
```

---

## 5. 模块设计

### 5.1 核心模块

#### 5.1.1 magic-api (核心模块)

**职责：** 提供核心业务逻辑

**包结构：**
```
org.ssssssss.magicapi/
├── core/                    # 核心功能
│   ├── annotation/          # 注解
│   ├── config/              # 配置
│   ├── context/             # 上下文
│   ├── event/               # 事件
│   ├── exception/           # 异常
│   ├── handler/             # 处理器
│   ├── interceptor/         # 拦截器
│   ├── logging/             # 日志
│   ├── model/               # 模型
│   ├── resource/            # 资源
│   ├── service/             # 服务
│   ├── servlet/             # Servlet 适配
│   └── web/                 # Web 控制器
├── datasource/              # 数据源
├── function/                # 函数
├── modules/                 # 模块
│   ├── db/                  # 数据库
│   ├── http/                # HTTP
│   ├── servlet/             # Servlet
│   └── spring/              # Spring
├── backup/                  # 备份
└── utils/                   # 工具
```

#### 5.1.2 magic-editor (编辑器模块)

**职责：** 提供 Web 可视化编辑器

**资源结构：**
```
magic-editor/
└── src/main/resources/magic-editor/
    ├── index.html           # 入口页面
    └── assets/              # 前端资源
        ├── app.*.js         # 应用代码
        ├── vendor.*.js      # 第三方库
        ├── vue.*.js         # Vue.js
        └── *.worker.js      # Web Workers
```

### 5.2 插件模块

#### 5.2.1 插件结构

```
magic-api-plugin-xxx/
├── src/main/java/           # Java 代码
│   └── org/ssssssss/magicapi/xxx/
│       ├── config/          # 配置
│       ├── service/         # 服务
│       └── web/             # 控制器
├── src/console/             # 前端代码（可选）
│   └── src/
│       ├── components/      # Vue 组件
│       └── service/         # 前端服务
└── src/main/resources/
    └── META-INF/spring/
        └── org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

#### 5.2.2 插件注册

```java
@Configuration
public class XxxPluginConfiguration implements MagicPluginConfiguration {
    
    @Override
    public Plugin plugin() {
        return new Plugin("xxx", "1.0.0", "插件描述");
    }
    
    @Override
    public ControllerRegister controllerRegister() {
        return (mapping, configuration) -> {
            // 注册 Controller
            mapping.registerController(new XxxController(configuration));
        };
    }
    
    @Bean
    public XxxService xxxService() {
        return new XxxService();
    }
}
```

---

## 6. 部署方案

### 6.1 单机部署

**适用场景：** 开发、测试、小规模生产

**部署架构：**
```
┌─────────────────────────────────────┐
│         应用服务器                   │
│  ┌─────────────────────────────┐   │
│  │  Spring Boot Application    │   │
│  │  + magic-api                │   │
│  │  + 内嵌 Tomcat              │   │
│  └─────────────────────────────┘   │
│                                     │
│  ┌─────────────────────────────┐   │
│  │  MySQL (本地或远程)          │   │
│  └─────────────────────────────┘   │
└─────────────────────────────────────┘
```

**启动命令：**
```bash
java -jar magic-api-demo.jar \
    --server.port=9999 \
    --magic-api.security.username=admin \
    --magic-api.security.password=admin123
```

### 6.2 集群部署

**适用场景：** 大规模生产环境

**部署架构：**
```
                    ┌─────────────┐
                    │   Nginx     │
                    │  (负载均衡)  │
                    └──────┬──────┘
                           │
            ┌──────────────┼──────────────┐
            │              │              │
    ┌───────▼───────┐ ┌────▼─────┐ ┌─────▼──────┐
    │  Node 1       │ │  Node 2  │ │  Node 3    │
    │  magic-api    │ │ magic-api│ │ magic-api  │
    └───────┬───────┘ └────┬─────┘ └─────┬──────┘
            │              │              │
            └──────────────┼──────────────┘
                           │
                    ┌──────▼──────┐
                    │    Redis    │
                    │  (会话共享)  │
                    └─────────────┘
                           │
                    ┌──────▼──────┐
                    │    MySQL    │
                    │  (主从复制)  │
                    └─────────────┘
```

**集群配置：**
```yaml
magic-api:
  # 实例 ID，每台机器必须不同
  instance-id: node-1
  
  # 资源存储到数据库
  resource:
    type: database
    datasource: master
    table-name: magic_api_resource
  
  # 备份存储到数据库
  backup:
    enable: true
    datasource: master
    table-name: magic_api_backup
```

**集群插件：**
```xml
<dependency>
    <groupId>org.ssssssss</groupId>
    <artifactId>magic-api-plugin-cluster</artifactId>
    <version>${magic-api.version}</version>
</dependency>
```

### 6.3 Docker 部署

**Dockerfile：**
```dockerfile
FROM openjdk:8-jre-alpine

WORKDIR /app

COPY magic-api-demo.jar app.jar

EXPOSE 9999

ENV JAVA_OPTS="-Xms512m -Xmx1g"

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

**docker-compose.yml：**
```yaml
version: '3'
services:
  magic-api:
    build: .
    ports:
      - "9999:9999"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/magic_api
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=123456
      - MAGIC_API_SECURITY_USERNAME=admin
      - MAGIC_API_SECURITY_PASSWORD=admin123
    depends_on:
      - mysql
  
  mysql:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=123456
      - MYSQL_DATABASE=magic_api
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

---

## 7. 性能优化

### 7.1 脚本编译缓存

```yaml
magic-api:
  compile-cache-size: 500  # 编译缓存容量
```

### 7.2 SQL 缓存

```javascript
// 使用缓存
var user = db.selectOne("SELECT * FROM user WHERE id = ?", [id], 60); // 缓存 60 秒
```

### 7.3 线程池优化

```yaml
magic-api:
  thread-pool-executor-size: 16  # 默认 CPU 核心数 * 2
```

### 7.4 响应压缩

```yaml
server:
  compression:
    enabled: true
    min-response-size: 1024
    mime-types: application/json,text/html,text/plain
```

---

## 8. 安全策略

### 8.1 认证安全

- 强制登录认证
- 密码加密存储
- Session 超时控制
- 支持自定义认证

### 8.2 接口安全

- 接口锁定防止误修改
- 操作日志记录
- 推送签名验证
- CORS 配置

### 8.3 数据安全

- SQL 参数化查询（防注入）
- 数据源密码加密
- 敏感数据脱敏
- 备份数据加密

---

## 9. 监控方案

### 9.1 应用监控

- Spring Boot Actuator
- 自定义健康检查
- 指标暴露（Prometheus）

### 9.2 日志监控

- 请求日志
- 错误日志
- SQL 日志
- 操作日志

### 9.3 性能监控

- 接口响应时间
- 慢查询监控
- JVM 监控
- 数据库连接池监控

---

## 10. 总结

本技术方案描述了 magic-api 的整体架构设计、技术选型、核心实现策略、部署方案等。通过动态路由注册、脚本执行引擎、多数据源管理等核心技术，实现了零代码模板的 API 快速开发能力。方案支持单机和集群部署，提供了完善的性能优化和安全策略。
