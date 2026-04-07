# magic-api 整体架构文档

> 版本：2.2.2  
> 生成时间：2026-03-14  
> 文档类型：架构设计

---

## 1. 整体架构概述

magic-api 是一个基于 Java 的接口快速开发框架，采用**可视化 UI + 动态脚本引擎**的架构设计，通过 Web 界面编写接口逻辑，自动映射为 HTTP 接口，无需定义 Controller、Service、Dao、Mapper、XML、VO 等 Java 对象即可完成 HTTP API 接口开发。

### 1.1 核心设计理念

- **零代码模板**：无需编写 Controller、Service、Dao 等样板代码
- **可视化开发**：通过 Web 编辑器在线编写、调试、管理接口
- **热更新**：接口修改后即时生效，无需重启服务
- **插件化扩展**：通过插件机制支持多种数据库和功能扩展
- **动态脚本**：基于 magic-script 脚本引擎实现动态逻辑执行

---

## 2. 系统架构图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          客户端层 (Client Layer)                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐ │
│  │  Web 浏览器   │  │  HTTP Client │  │  Mobile App  │  │  第三方系统 │ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼ HTTP/WebSocket
┌─────────────────────────────────────────────────────────────────────────┐
│                        接入层 (Access Layer)                             │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │              Spring Boot Web 容器 (Tomcat/Jetty/Undertow)          │  │
│  │  ┌─────────────────────┐  ┌─────────────────────────────────┐    │  │
│  │  │  RequestHandler     │  │  MagicWebSocketDispatcher       │    │  │
│  │  │  (HTTP 请求入口)     │  │  (WebSocket 消息分发)            │    │  │
│  │  └─────────────────────┘  └─────────────────────────────────┘    │  │
│  └───────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        核心层 (Core Layer)                               │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                     MagicConfiguration                            │  │
│  │                  (核心配置与上下文管理)                             │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐ │
│  │ RequestMagic │  │  Function    │  │   Magic      │  │   Magic      │ │
│  │ DynamicReg-  │  │ DynamicReg-  │  │  Resource    │  │     API      │ │
│  │ istry        │  │ istry        │  │  Service     │  │   Service    │ │
│  │ (请求动态注册)│  │ (函数注册)   │  │ (资源管理)   │  │ (API 调用)   │ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └─────────────┘ │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                  RequestInterceptor Chain                         │  │
│  │                (前置/后置/完成 拦截器链)                            │  │
│  └───────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                       执行层 (Execution Layer)                           │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                    MagicScript Engine                             │  │
│  │                  (magic-script v1.9.0 脚本引擎)                     │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐   │  │
│  │  │  Script     │  │   Context   │  │    Module System        │   │  │
│  │  │  Parser     │  │  Management │  │  (db/http/servlet/...)  │   │  │
│  │  └─────────────┘  └─────────────┘  └─────────────────────────┘   │  │
│  └───────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        模块层 (Module Layer)                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐ │
│  │     DB       │  │    HTTP      │  │   Servlet    │  │    Spring   │ │
│  │   Module     │  │   Module     │  │   Module     │  │   Module    │ │
│  │ (数据库操作) │  │ (HTTP 调用)  │  │ (请求响应)   │  │ (Spring 集成)│ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └─────────────┘ │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                    Dialect Adapter                                │  │
│  │         (MySQL/Oracle/PostgreSQL/SQLServer/DB2/Clickhouse...)     │  │
│  └───────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                       数据层 (Data Layer)                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐ │
│  │   MySQL      │  │   Oracle     │  │  PostgreSQL  │  │   Redis     │ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └─────────────┘ │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐ │
│  │   MongoDB    │  │Elasticsearch │  │  Nebula Graph│  │   文件存储  │ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. 模块划分和职责

### 3.1 核心模块结构

```
magic-api/
├── magic-api                          # 核心模块
│   ├── core/                          # 核心功能
│   │   ├── annotation/                # 注解定义
│   │   ├── config/                    # 配置管理
│   │   ├── context/                   # 上下文管理
│   │   ├── event/                     # 事件系统
│   │   ├── exception/                 # 异常定义
│   │   ├── handler/                   # 请求处理器
│   │   ├── interceptor/               # 拦截器
│   │   ├── logging/                   # 日志系统
│   │   ├── model/                     # 数据模型
│   │   ├── resource/                  # 资源管理
│   │   ├── service/                   # 核心服务
│   │   ├── servlet/                   # Servlet 适配
│   │   └── web/                       # Web 控制器
│   ├── datasource/                    # 数据源管理
│   ├── function/                      # 函数功能
│   ├── jsr223/                        # JSR223 脚本引擎支持
│   ├── modules/                       # 功能模块
│   │   ├── db/                        # 数据库模块
│   │   ├── http/                      # HTTP 模块
│   │   ├── servlet/                   # Servlet 模块
│   │   └── spring/                    # Spring 集成模块
│   ├── backup/                        # 备份功能
│   └── utils/                         # 工具类
│
├── magic-editor                       # Web 编辑器模块
├── magic-api-servlet                  # Servlet 适配模块
│   ├── magic-api-servlet-javaee       # JavaEE Servlet 适配
│   └── magic-api-servlet-jakarta      # Jakarta EE Servlet 适配
├── magic-api-spring-boot-starter      # Spring Boot 启动器
└── magic-api-plugins                  # 插件模块集合
    ├── magic-api-plugin-cluster       # 集群插件
    ├── magic-api-plugin-component     # 组件插件
    ├── magic-api-plugin-elasticsearch # ES 插件
    ├── magic-api-plugin-git           # Git 集成插件
    ├── magic-api-plugin-mongo         # MongoDB 插件
    ├── magic-api-plugin-nebula        # Nebula Graph 插件
    ├── magic-api-plugin-redis         # Redis 插件
    ├── magic-api-plugin-springdoc     # SpringDoc 文档插件
    ├── magic-api-plugin-swagger       # Swagger 文档插件
    └── magic-api-plugin-task          # 定时任务插件
```

### 3.2 各模块职责

| 模块 | 职责描述 | 关键类 |
|------|---------|--------|
| **magic-api (core)** | 核心业务逻辑，包含请求处理、资源管理、脚本执行 | `RequestHandler`, `MagicConfiguration`, `MagicAPIService` |
| **magic-api (modules/db)** | 数据库操作支持，SQL 执行、方言适配、事务管理 | `SQLModule`, `DialectAdapter`, `Transaction` |
| **magic-api (modules/http)** | HTTP 客户端调用支持 | `HTTPModule` |
| **magic-api (datasource)** | 多数据源管理与动态切换 | `MagicDynamicDataSource`, `DataSourceService` |
| **magic-api (function)** | 自定义函数注册与调用 | `FunctionInfo`, `FunctionMagicDynamicRegistry` |
| **magic-api (backup)** | 接口备份与还原 | `MagicBackupService`, `MagicDatabaseBackupService` |
| **magic-editor** | Web 可视化编辑器，提供代码编写、调试界面 | Monaco Editor + Vue.js |
| **magic-api-servlet** | Servlet 规范适配，兼容 JavaEE 和 Jakarta EE | `MagicHttpServletRequest`, `MagicHttpServletResponse` |
| **magic-api-spring-boot-starter** | Spring Boot 自动配置，简化集成 | `MagicAPIAutoConfiguration` |
| **magic-api-plugins** | 可选功能扩展，按需引入 | 各插件配置类 |

---

## 4. 数据流和控制流

### 4.1 HTTP 请求处理流程

```
客户端请求
    │
    ▼
┌─────────────────────────────────┐
│  Spring MVC DispatcherServlet   │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│  RequestMappingHandlerMapping   │
│  (路由匹配：GET /api/user/:id)   │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│      RequestHandler.invoke()    │
│  1. 构建 RequestEntity          │
│  2. 读取请求参数/Body/Header    │
│  3. 参数验证                    │
│  4. 执行前置拦截器              │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│   ScriptManager.executeScript() │
│  1. 创建 MagicScriptContext     │
│  2. 注入请求上下文变量          │
│  3. 执行 magic-script 脚本       │
│  4. 调用模块函数 (db/http/...)  │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│      ResultProvider.build()     │
│  1. 执行后置拦截器              │
│  2. 构建统一响应格式            │
│  3. 执行 afterCompletion        │
└─────────────────────────────────┘
    │
    ▼
HTTP 响应返回客户端
```

### 4.2 脚本执行流程

```
magic-script 脚本
    │
    ▼
┌─────────────────────────────────┐
│     MagicScriptEngine           │
│  1. 词法分析 (Lexer)            │
│  2. 语法分析 (Parser)           │
│  3. 生成 AST                    │
│  4. 编译为字节码 (可选缓存)      │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│    MagicScriptContext           │
│  - 请求参数 (parameters)        │
│  - 路径变量 (pathVariables)     │
│  - 请求头 (header)              │
│  - Cookie (cookie)              │
│  - Session (session)            │
│  - 请求体 (requestBody)         │
│  - 内置模块 (db/http/response)  │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│      Module System              │
│  - db.select("SELECT * FROM...")│
│  - http.get("https://...")      │
│  - response.setHeader(...)      │
│  - 自定义函数调用               │
└─────────────────────────────────┘
    │
    ▼
返回执行结果
```

### 4.3 WebSocket 通信流程

```
前端编辑器                      后端 WebSocket
    │                              │
    │  1. 连接 WebSocket           │
    ├─────────────────────────────>│
    │                              │
    │  2. 发送调试请求             │
    ├─────────────────────────────>│
    │                              │
    │                              │ MagicDebugHandler
    │                              │ 处理调试命令
    │                              │
    │  3. 返回执行结果/断点信息    │
    │<─────────────────────────────┤
    │                              │
    │  4. 实时日志推送             │
    │<─────────────────────────────┤
    │                              │
```

---

## 5. 关键技术决策

### 5.1 脚本引擎选型：magic-script

**决策理由：**
- 自研脚本引擎，与 magic-api 深度集成
- 支持热更新，无需重启服务
- 提供类似 JavaScript 的语法，学习成本低
- 支持语法提示、错误提示、断点调试
- 可自定义扩展函数和模块

**替代方案考虑：**
- Groovy：功能强大但较重，编译速度慢
- JavaScript (Nashorn)：JDK 15+ 已移除
- Python (Jython)：性能较差，集成复杂

### 5.2 动态路由注册：RequestMappingHandlerMapping

**决策理由：**
- 利用 Spring MVC 原生扩展点
- 支持完整的 Spring MVC 功能（拦截器、消息转换器等）
- 运行时动态注册/删除映射
- 与 Spring Boot 无缝集成

**实现方式：**
```java
RequestMappingInfo info = RequestMappingInfo
    .paths(path)
    .methods(RequestMethod.valueOf(method))
    .build();
requestMappingHandlerMapping.registerMapping(info, handler, method);
```

### 5.3 多数据源支持：MagicDynamicDataSource

**决策理由：**
- 支持运行时动态切换数据源
- 基于 ThreadLocal 实现线程隔离
- 支持数据源连接池管理
- 与 Spring JDBC 兼容

**数据源类型：**
- 关系型数据库：MySQL、Oracle、PostgreSQL、SQL Server、DB2、Clickhouse、达梦、人大金仓
- NoSQL：Redis、MongoDB、Elasticsearch、Nebula Graph（通过插件）

### 5.4 资源存储策略：可插拔 Resource

**决策理由：**
- 支持文件存储和数据库存储两种模式
- 通过 Resource 接口抽象，便于扩展
- 默认文件存储，便于版本管理
- 数据库存储适合集群环境

**实现：**
```java
public interface Resource {
    InputStream get(String id) throws IOException;
    void save(String id, InputStream inputStream) throws IOException;
    void delete(String id) throws IOException;
    List<String> list(String path) throws IOException;
}
```

### 5.5 插件化架构：可选依赖

**决策理由：**
- 核心模块保持轻量
- 按需引入插件，避免依赖膨胀
- 通过 Spring Boot Starter 机制自动配置
- 插件可注册 Controller、Service、模块扩展

**插件注册机制：**
```java
@Configuration
public class MagicPluginConfiguration {
    public Plugin plugin();
    public ControllerRegister controllerRegister();
}
```

### 5.6 响应格式统一：ResultProvider

**决策理由：**
- 统一 API 响应格式
- 支持自定义响应结构
- 支持异常统一处理
- 可配置成功/失败码

**默认格式：**
```json
{
  "code": 200,
  "message": "success",
  "data": {}
}
```

---

## 6. 架构特性

### 6.1 高性能

- 脚本编译缓存（默认 500）
- SQL 执行结果缓存
- 异步调用支持（`async` 关键字）
- 线程池优化（CPU 核心数 * 2）

### 6.2 高可用

- 集群支持（通过 magic-api-plugin-cluster）
- 接口自动同步
- WebSocket 会话管理
- 备份还原机制

### 6.3 安全性

- 登录认证（可自定义 AuthorizationInterceptor）
- 接口锁定防止误修改
- 推送签名验证
- 数据源密码加密

### 6.4 可扩展性

- 自定义函数注册
- 自定义拦截器
- 自定义模块扩展
- 自定义响应格式

---

## 7. 技术约束

| 约束项 | 说明 |
|--------|------|
| JDK 版本 | 最低 Java 8，推荐 Java 11+ |
| Spring Boot | 最低 2.4.5，推荐 2.7.x |
| Servlet | 最低 3.1，推荐 4.0+ |
| 脚本引擎 | magic-script 1.9.0（固定版本） |
| JSON 处理 | FastJSON 1.2.83 |

---

## 8. 总结

magic-api 采用分层架构设计，核心层负责请求路由和脚本执行，模块层提供数据库、HTTP 等功能支持，数据层支持多种数据库。通过动态脚本引擎实现零代码模板开发，通过插件化架构实现功能扩展，通过 WebSocket 实现实时调试。整体架构简洁、灵活、易扩展。
