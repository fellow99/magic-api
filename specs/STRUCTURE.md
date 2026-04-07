# magic-api 工程结构文档

> 生成时间：2026-03-14  
> 项目版本：2.2.2

---

## 项目概述

magic-api 是一个基于 Java 的接口快速开发框架，通过 UI 界面编写接口，自动映射为 HTTP 接口，无需定义 Controller、Service、Dao、Mapper、XML、VO 等 Java 对象即可完成常见的 HTTP API 接口开发。

---

## 项目根目录结构

```
magic-api/
├── .git/                          # Git 版本控制
├── .gitee/                        # Gitee 平台配置
├── .github/                       # GitHub 配置（CI/CD、Issue 模板等）
├── .specify/                      # 规范配置目录
│   ├── memory/                    # 规范记忆
│   ├── scripts/                   # 脚本文件
│   │   └── bash/                  # Bash 脚本
│   └── templates/                 # 模板文件
├── db/                            # 数据库相关脚本
├── magic-api/                     # 核心模块
├── magic-api-plugins/             # 插件模块集合
├── magic-api-servlet/             # Servlet 适配模块
├── magic-api-spring-boot-starter/ # Spring Boot 启动器
├── magic-editor/                  # Web 编辑器模块
├── specs/                         # 规范文档（最终版）
├── specs-plan-b/                  # 规范计划 B（备选方案）
├── specs-plan-c/                  # 规范计划 C（备选方案）
├── pom.xml                        # 父 POM 配置文件
├── README.md                      # 项目说明文档
└── LICENSE                        # 开源许可证
```

---

## 主要模块详解

### 1. magic-api（核心模块）

**路径：** `magic-api/`

核心功能模块，包含所有核心业务逻辑。

```
magic-api/
├── pom.xml
└── src/main/java/org/ssssssss/magicapi/
    ├── backup/                    # 备份功能
    │   ├── model/                 # 备份数据模型
    │   ├── service/               # 备份服务
    │   │   ├── MagicBackupService.java
    │   │   └── MagicDatabaseBackupService.java
    │   └── web/                   # 备份控制器
    │       └── MagicBackupController.java
    ├── core/                      # 核心功能
    │   ├── annotation/            # 注解定义
    │   │   ├── MagicModule.java
    │   │   ├── Message.java
    │   │   └── Valid.java
    │   ├── config/                # 配置类
    │   │   ├── MagicAPIProperties.java
    │   │   ├── MagicConfiguration.java
    │   │   ├── MagicPluginConfiguration.java
    │   │   ├── MagicCorsFilter.java
    │   │   └── WebSocketSessionManager.java
    │   ├── context/               # 上下文管理
    │   │   ├── MagicConsoleSession.java
    │   │   ├── MagicUser.java
    │   │   ├── RequestContext.java
    │   │   └── SessionContext.java
    │   ├── event/                 # 事件系统
    │   │   ├── MagicEvent.java
    │   │   ├── FileEvent.java
    │   │   └── NotifyEvent.java
    │   ├── exception/             # 异常定义
    │   │   ├── MagicAPIException.java
    │   │   └── MagicLoginException.java
    │   ├── handler/               # 请求处理器
    │   │   ├── MagicDebugHandler.java
    │   │   └── MagicWorkbenchHandler.java
    │   ├── interceptor/           # 拦截器
    │   │   ├── AuthorizationInterceptor.java
    │   │   └── RequestInterceptor.java
    │   ├── logging/               # 日志
    │   ├── model/                 # 数据模型
    │   ├── resource/              # 资源管理
    │   ├── service/               # 核心服务
    │   │   └── impl/              # 服务实现
    │   ├── servlet/               # Servlet 适配
    │   └── web/                   # Web 相关
    ├── datasource/                # 数据源管理
    │   ├── model/                 # 数据源模型
    │   ├── service/               # 数据源服务
    │   └── web/                   # 数据源控制器
    ├── function/                  # 函数功能
    │   ├── model/                 # 函数模型
    │   └── service/               # 函数服务
    ├── jsr223/                    # JSR223 脚本引擎支持
    ├── modules/                   # 功能模块
    │   ├── db/                    # 数据库模块
    │   │   ├── cache/             # SQL 缓存
    │   │   ├── dialect/           # 数据库方言
    │   │   ├── inteceptor/        # 拦截器
    │   │   ├── model/             # 数据模型
    │   │   ├── mybatis/           # MyBatis 集成
    │   │   ├── provider/          # 提供者
    │   │   └── table/             # 表操作
    │   ├── http/                  # HTTP 模块
    │   ├── servlet/               # Servlet 模块
    │   └── spring/                # Spring 集成模块
    └── utils/                     # 工具类
```

**关键文件：**
- `MagicAPIProperties.java` - 核心配置属性
- `MagicConfiguration.java` - 核心配置类
- `MagicBackupService.java` - 备份服务接口

---

### 2. magic-editor（Web 编辑器）

**路径：** `magic-editor/`

提供 Web 界面的在线编辑器，用于编写和管理 API 接口。

```
magic-editor/
├── pom.xml
└── src/main/resources/magic-editor/
    ├── index.html                 # 编辑器入口页面
    └── assets/                    # 前端资源文件
        ├── app.60f63c60.js        # 应用主文件
        ├── vendor.295b3547.js     # 第三方库
        ├── vue.6f28a6f0.js        # Vue.js 框架
        ├── axios.23e7b955.js      # HTTP 客户端
        ├── editor.worker.3a89906b.js  # 编辑器 worker
        ├── html.worker.5cedee30.js    # HTML worker
        └── json.worker.69e5d80a.js    # JSON worker
```

**技术栈：**
- Vue.js（前端框架）
- Monaco Editor（代码编辑器）
- Axios（HTTP 请求）

**访问入口：** `http://localhost:9999/magic/web`

---

### 3. magic-api-plugins（插件模块集合）

**路径：** `magic-api-plugins/`

包含多个可选插件模块，扩展核心功能。

```
magic-api-plugins/
├── pom.xml                        # 插件父 POM
├── magic-api-plugin-cluster/      # 集群插件
│   ├── src/main/java/org/ssssssss/magicapi/cluster/
│   └── src/main/resources/META-INF/spring/
├── magic-api-plugin-component/    # 组件插件
│   ├── src/console/               # 前端组件
│   │   └── src/
│   │       ├── components/        # Vue 组件
│   │       ├── i18n/              # 国际化
│   │       ├── icons/             # 图标
│   │       └── service/           # 服务
│   └── src/main/java/             # Java 后端
├── magic-api-plugin-elasticsearch/ # Elasticsearch 插件
├── magic-api-plugin-git/          # Git 集成插件
├── magic-api-plugin-mongo/        # MongoDB 插件
├── magic-api-plugin-nebula/       # Nebula Graph 插件
├── magic-api-plugin-redis/        # Redis 插件
├── magic-api-plugin-springdoc/    # SpringDoc 文档插件
├── magic-api-plugin-swagger/      # Swagger 文档插件
└── magic-api-plugin-task/         # 定时任务插件
    ├── src/console/               # 任务管理前端
    └── src/main/java/             # 任务后端
        └── org/ssssssss/magicapi/task/
            ├── model/             # 任务模型
            ├── service/           # 任务服务
            ├── starter/           # 任务启动器
            └── web/               # 任务控制器
```

**插件列表：**

| 插件名称 | 功能描述 |
|---------|---------|
| magic-api-plugin-cluster | 集群部署支持，接口自动同步 |
| magic-api-plugin-component | 自定义组件支持 |
| magic-api-plugin-elasticsearch | Elasticsearch 数据库支持 |
| magic-api-plugin-git | Git 版本控制集成 |
| magic-api-plugin-mongo | MongoDB 非关系型数据库支持 |
| magic-api-plugin-nebula | Nebula Graph 图数据库支持 |
| magic-api-plugin-redis | Redis 缓存支持 |
| magic-api-plugin-springdoc | SpringDoc API 文档生成 |
| magic-api-plugin-swagger | Swagger API 文档生成 |
| magic-api-plugin-task | 定时任务管理 |

---

### 4. magic-api-servlet（Servlet 适配模块）

**路径：** `magic-api-servlet/`

提供不同 Servlet 规范的适配实现。

```
magic-api-servlet/
├── pom.xml
├── magic-api-servlet-jakarta/     # Jakarta Servlet 6.0+ 适配
│   └── src/main/java/org/ssssssss/magicapi/servlet/
└── magic-api-servlet-javaee/      # JavaEE Servlet 适配
    └── src/main/java/org/ssssssss/magicapi/servlet/
```

**说明：**
- `magic-api-servlet-jakarta` - 适配 Jakarta EE 9+ (Servlet 6.0+)
- `magic-api-servlet-javaee` - 适配传统 JavaEE (Servlet 3.0/4.0)

---

### 5. magic-api-spring-boot-starter（Spring Boot 启动器）

**路径：** `magic-api-spring-boot-starter/`

Spring Boot 自动配置模块，简化集成流程。

```
magic-api-spring-boot-starter/
├── pom.xml
└── src/
    ├── main/java/org/ssssssss/magicapi/spring/boot/starter/
    │   ├── ApplicationUriPrinter.java           # 应用 URI 打印
    │   ├── MagicAPIAutoConfiguration.java       # 自动配置主类
    │   ├── MagicDynamicRegistryConfiguration.java # 动态注册配置
    │   ├── MagicJsonAutoConfiguration.java      # JSON 自动配置
    │   ├── MagicModuleConfiguration.java        # 模块配置
    │   └── MagicServletConfiguration.java       # Servlet 配置
    └── main/resources/META-INF/spring/
        ├── org.springframework.boot.autoconfigure.AutoConfiguration.imports
        └── spring.factories                   # Spring 工厂配置
```

**关键配置类：**
- `MagicAPIAutoConfiguration.java` - 核心自动配置
- `MagicServletConfiguration.java` - Servlet 注册配置
- `MagicJsonAutoConfiguration.java` - JSON 序列化配置

---

## 前端页面和路由清单

### magic-editor 前端资源

| 文件 | 说明 |
|-----|------|
| `index.html` | 编辑器入口页面 |
| `assets/app.*.js` | 应用主逻辑 |
| `assets/vendor.*.js` | 第三方依赖库 |
| `assets/vue.*.js` | Vue.js 框架 |
| `assets/axios.*.js` | HTTP 请求库 |
| `assets/editor.worker.*.js` | Monaco Editor Web Worker |
| `assets/html.worker.*.js` | HTML 语法高亮 Worker |
| `assets/json.worker.*.js` | JSON 语法高亮 Worker |

### magic-api-plugin-component 前端组件

```
src/console/src/
├── components/    # Vue 组件
├── i18n/          # 国际化文件
├── icons/         # 图标资源
└── service/       # 前端服务
```

### magic-api-plugin-task 前端组件

```
src/console/src/
├── components/    # 任务管理组件
├── i18n/          # 国际化文件
├── icons/         # 图标资源
└── service/       # 任务服务
```

---

## 关键配置文件

### 根 POM (pom.xml)

**位置：** `/pom.xml`

**关键属性：**
- `spring-boot.version`: 2.4.5
- `magic-script.version`: 1.9.0
- `java.version`: 1.8

**子模块：**
1. magic-api
2. magic-editor
3. magic-api-servlet
4. magic-api-plugins
5. magic-api-spring-boot-starter

### Spring Boot 配置示例

```properties
server.port=9999
magic-api.web=/magic/web
magic-api.resource.location=/data/magic-api
```

### Spring Boot 自动配置注册

**位置：** `magic-api-spring-boot-starter/src/main/resources/META-INF/spring.factories`

**位置：** `magic-api-spring-boot-starter/src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

---

## 模块依赖关系

```
magic-api-spring-boot-starter
    ├── magic-api (核心)
    ├── magic-editor (Web 界面)
    └── magic-api-servlet (Servlet 适配)

magic-api-plugins (各插件)
    └── magic-api (核心依赖)
```

---

## 目录命名规范

- **Java 包路径：** `org/ssssssss/magicapi/{module}/`
- **配置资源：** `src/main/resources/META-INF/spring/`
- **前端资源：** `src/main/resources/magic-editor/`
- **插件前端：** `src/console/src/`

---

## 总结

magic-api 工程采用多模块 Maven 项目结构，主要包含：

1. **核心模块** (`magic-api`) - 提供 API 开发核心功能
2. **编辑器模块** (`magic-editor`) - 提供 Web 界面编辑器
3. **插件模块** (`magic-api-plugins`) - 10 个可选功能插件
4. **Servlet 适配** (`magic-api-servlet`) - 兼容不同 Servlet 规范
5. **启动器模块** (`magic-api-spring-boot-starter`) - Spring Boot 集成

项目支持多种数据库（MySQL、Oracle、PostgreSQL、MongoDB、Redis 等），提供完整的在线编辑、调试、版本管理功能，适合快速开发 HTTP API 接口。
