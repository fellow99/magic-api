# 001-core 核心框架模块规格说明书

**模块名称**: 核心框架 (Core Framework)  
**模块路径**: magic-api/magic-api/src/main/java/org/ssssssss/magicapi/core  
**创建时间**: 2026-03-14  
**状态**: 已完成  
**版本**: 2.2.2

---

## 1. 模块概述

### 1.1 模块定位

核心框架模块是 magic-api 的基础引擎，提供接口开发、请求处理、配置管理、上下文维护、事件系统、异常处理、拦截器机制等核心功能。该模块是整个框架的运行基石，所有其他模块和插件都依赖于此。

### 1.2 核心价值

- **动态接口引擎**: 支持运行时动态注册、更新、删除 HTTP 接口
- **请求处理核心**: 统一处理 HTTP 请求，执行脚本，返回响应
- **配置管理中心**: 提供统一的配置管理和属性绑定
- **上下文管理**: 维护请求上下文、会话上下文、用户信息
- **扩展机制**: 提供拦截器、结果提供者等扩展点

### 1.3 模块范围

**包含**:
- 注解系统定义
- 配置属性管理
- 上下文管理
- 事件系统
- 异常处理
- 拦截器机制
- 日志管理
- 数据模型定义
- 资源存储抽象
- 核心服务接口
- Web 控制器

**不包含**:
- 具体数据库操作 (由 modules-db 负责)
- HTTP 客户端功能 (由 modules-http 负责)
- Servlet 适配 (由 magic-api-servlet 负责)
- Spring 集成 (由 modules-spring 负责)
- 插件功能 (由各 plugin 模块负责)

---

## 2. 功能需求

### 2.1 注解系统 (FR-001)

**功能描述**: 提供用于脚本模块定义的注解。

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-001-1 | @MagicModule | P0 | 定义脚本模块，支持模块导入 |
| FR-001-2 | @Message | P2 | 定义消息提示注解 |
| FR-001-3 | @Valid | P1 | 参数验证注解 |

**代码依据**:
- `annotation/MagicModule.java`
- `annotation/Message.java`
- `annotation/Valid.java`

---

### 2.2 配置管理 (FR-002)

**功能描述**: 提供核心配置属性管理和自动配置功能。

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-002-1 | MagicAPIProperties | P0 | 核心配置属性类，支持所有配置项绑定 |
| FR-002-2 | MagicConfiguration | P0 | 自动配置类，初始化核心 Bean |
| FR-002-3 | MagicPluginConfiguration | P1 | 插件配置管理 |
| FR-002-4 | 配置项支持 | P0 | 备份、缓存、CRUD、调试、安全、分页、资源等配置 |

**配置项分类**:
- **基础配置**: web 路径、prefix、secret-key
- **安全配置**: 启用状态、token 超时、跨域设置
- **缓存配置**: 启用状态、过期时间
- **备份配置**: 启用状态、保留版本数
- **调试配置**: 启用状态、日志级别
- **分页配置**: 默认页大小、最大页数

**代码依据**:
- `config/MagicAPIProperties.java`
- `config/MagicConfiguration.java`
- `config/MagicPluginConfiguration.java`
- `config/` 目录下所有配置类

---

### 2.3 上下文管理 (FR-003)

**功能描述**: 提供请求上下文、会话上下文、用户信息的管理。

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-003-1 | RequestContext | P0 | 请求上下文，使用 ThreadLocal 存储请求信息 |
| FR-003-2 | SessionContext | P0 | 会话上下文接口，管理用户会话 |
| FR-003-3 | MagicUser | P0 | 用户信息模型 |
| FR-003-4 | MagicConsoleSession | P1 | 控制台会话实现 |
| FR-003-5 | CookieContext | P1 | Cookie 处理工具 |
| FR-003-6 | RequestEntity | P1 | 请求实体封装 |

**代码依据**:
- `context/RequestContext.java`
- `context/SessionContext.java`
- `context/MagicUser.java`
- `context/MagicConsoleSession.java`
- `context/CookieContext.java`
- `context/RequestEntity.java`

---

### 2.4 事件系统 (FR-004)

**功能描述**: 提供基于 Spring 事件机制的事件驱动架构。

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-004-1 | MagicEvent | P0 | 基础事件类 |
| FR-004-2 | FileEvent | P1 | 文件变更事件 |
| FR-004-3 | GroupEvent | P1 | 分组变更事件 |
| FR-004-4 | EventAction | P1 | 事件动作枚举 (CREATE, UPDATE, DELETE) |

**事件类型**:
- **文件事件**: 接口脚本创建、更新、删除时触发
- **分组事件**: 接口分组变更时触发
- **通知事件**: 系统通知时触发

**代码依据**:
- `event/MagicEvent.java`
- `event/FileEvent.java`
- `event/GroupEvent.java`
- `event/EventAction.java`

---

### 2.5 异常处理 (FR-005)

**功能描述**: 提供统一的异常处理和自定义异常体系。

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-005-1 | MagicAPIException | P0 | 基础异常类 |
| FR-005-2 | MagicLoginException | P1 | 登录异常 |
| FR-005-3 | MagicResourceNotFoundException | P1 | 资源未找到异常 |
| FR-005-4 | InvalidArgumentException | P1 | 参数无效异常 |
| FR-005-5 | ValidateException | P1 | 验证异常 |
| FR-005-6 | MagicExceptionHandler | P0 | 统一异常处理器 |

**代码依据**:
- `exception/MagicAPIException.java`
- `exception/MagicLoginException.java`
- `exception/MagicResourceNotFoundException.java`
- `exception/InvalidArgumentException.java`
- `exception/ValidateException.java`
- `handler/MagicExceptionHandler.java`

---

### 2.6 拦截器机制 (FR-006)

**功能描述**: 提供请求拦截器接口和默认实现。

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-006-1 | RequestInterceptor | P0 | 请求拦截器接口 |
| FR-006-2 | AuthorizationInterceptor | P0 | 认证拦截器接口 |
| FR-006-3 | DefaultAuthorizationInterceptor | P1 | 默认认证实现 |
| FR-006-4 | ResultProvider | P0 | 结果提供者接口 |
| FR-006-5 | DefaultResultProvider | P1 | 默认结果提供者 |

**拦截器生命周期**:
1. `preHandle`: 请求处理前执行，返回 false 则中断请求
2. `postHandle`: 请求处理后执行，可修改响应
3. `afterCompletion`: 完成后执行，用于资源清理

**代码依据**:
- `interceptor/RequestInterceptor.java`
- `interceptor/AuthorizationInterceptor.java`
- `interceptor/DefaultAuthorizationInterceptor.java`
- `interceptor/ResultProvider.java`
- `interceptor/DefaultResultProvider.java`

---

### 2.7 日志系统 (FR-007)

**功能描述**: 提供日志管理器和日志上下文。

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-007-1 | LoggerManager | P0 | 日志管理器 |
| FR-007-2 | MagicLoggerContext | P1 | 日志上下文接口 |
| FR-007-3 | 多日志框架支持 | P1 | 支持 Log4j、Log4j2、Logback |

**代码依据**:
- `logging/LoggerManager.java`
- `logging/MagicLoggerContext.java`

---

### 2.8 数据模型 (FR-008)

**功能描述**: 定义核心数据模型和实体类。

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-008-1 | ApiInfo | P0 | 接口信息模型 (路径、方法、脚本、参数等) |
| FR-008-2 | MagicEntity | P0 | 基础实体类 |
| FR-008-3 | PathMagicEntity | P1 | 路径实体 |
| FR-008-4 | Group | P1 | 分组模型 |
| FR-008-5 | Parameter | P1 | 参数模型 |
| FR-008-6 | Header | P1 | 请求头模型 |
| FR-008-7 | Path | P1 | 路径模型 |
| FR-008-8 | TreeNode | P1 | 树节点模型 |

**代码依据**:
- `model/ApiInfo.java`
- `model/MagicEntity.java`
- `model/PathMagicEntity.java`
- `model/Group.java`
- `model/Parameter.java`
- `model/Header.java`
- `model/Path.java`
- `model/TreeNode.java`

---

### 2.9 资源管理 (FR-009)

**功能描述**: 提供资源存储的抽象接口和多种实现。

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-009-1 | Resource | P0 | 资源接口 |
| FR-009-2 | ResourceAdapter | P1 | 资源适配器 |
| FR-009-3 | FileResource | P0 | 文件资源实现 |
| FR-009-4 | DatabaseResource | P1 | 数据库资源实现 |
| FR-009-5 | JarResource | P2 | JAR 资源实现 |
| FR-009-6 | KeyValueResource | P2 | KV 资源实现 |
| FR-009-7 | ZipResource | P2 | ZIP 资源实现 |

**代码依据**:
- `resource/Resource.java`
- `resource/ResourceAdapter.java`
- `resource/FileResource.java`
- `resource/DatabaseResource.java`
- `resource/JarResource.java`
- `resource/KeyValueResource.java`
- `resource/ZipResource.java`

---

### 2.10 服务层 (FR-010)

**功能描述**: 提供核心服务接口和实现。

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-010-1 | MagicAPIService | P0 | 核心服务接口 |
| FR-010-2 | MagicResourceService | P1 | 资源服务 |
| FR-010-3 | MagicNotifyService | P1 | 通知服务接口 |
| FR-010-4 | MagicDynamicRegistry | P0 | 动态注册器接口 |

**代码依据**:
- `service/MagicAPIService.java`
- `service/MagicResourceService.java`
- `service/MagicNotifyService.java`
- `service/MagicDynamicRegistry.java`

---

### 2.11 Web 控制器 (FR-011)

**功能描述**: 提供核心 Web 控制器和注册器。

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-011-1 | MagicController | P0 | 核心控制器，处理所有接口请求 |
| FR-011-2 | MagicControllerRegister | P1 | 控制器注册器 |

**代码依据**:
- `web/MagicController.java`
- `web/MagicControllerRegister.java`

---

### 2.12 工具类 (FR-012)

**功能描述**: 提供通用工具类。

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-012-1 | 集合工具 | P2 | 集合操作工具 |
| FR-012-2 | 字符串工具 | P2 | 字符串处理工具 |
| FR-012-3 | 反射工具 | P2 | 反射操作工具 |
| FR-012-4 | IO 工具 | P2 | IO 操作工具 |

**代码依据**:
- `utils/` 目录下所有工具类

---

## 3. 非功能需求

### 3.1 性能要求

| 指标 | 目标值 | 说明 |
|------|--------|------|
| 接口注册延迟 | < 100ms | 动态注册接口的延迟 |
| 请求处理延迟 | < 50ms | 核心框架处理开销 |
| 并发支持 | 1000+ QPS | 单节点并发处理能力 |

### 3.2 可用性要求

- **热更新**: 支持接口修改即时生效，无需重启应用
- **集群支持**: 支持集群部署，接口变更自动同步
- **故障恢复**: 支持异常自动捕获和友好提示

### 3.3 安全性要求

- **接口权限**: 支持接口级权限控制
- **认证机制**: 支持自定义认证拦截器
- **配置加密**: 敏感配置项支持加密存储

### 3.4 可扩展性要求

- **拦截器扩展**: 支持自定义拦截器
- **结果提供者扩展**: 支持自定义响应格式
- **资源存储扩展**: 支持自定义资源存储方式
- **自定义模块**: 支持通过注解定义自定义模块

---

## 4. 关键类说明

### 4.1 ApiInfo

接口信息模型，是 magic-api 的核心数据结构。

**属性**:
- `id`: 接口唯一标识
- `path`: 接口路径
- `method`: HTTP 方法 (GET/POST/PUT/DELETE 等)
- `script`: 接口脚本内容
- `parameters`: 参数定义列表
- `headers`: 请求头定义列表
- `group`: 所属分组
- `description`: 接口描述
- `createTime`: 创建时间
- `updateTime`: 更新时间

**用途**: 存储接口的完整定义，用于动态注册和请求处理。

---

### 4.2 MagicDynamicRegistry

动态注册器接口，负责接口的生命周期管理。

**方法**:
- `register(ApiInfo apiInfo)`: 注册新接口
- `unregister(String path)`: 注销接口
- `update(ApiInfo apiInfo)`: 更新接口
- `list()`: 列出所有接口

**实现类**: `RequestMagicDynamicRegistry`

---

### 4.3 RequestInterceptor

请求拦截器接口，提供请求处理的扩展点。

**方法**:
- `boolean preHandle(RequestContext context)`: 请求前处理
- `void postHandle(RequestContext context, Object result)`: 请求后处理
- `void afterCompletion(RequestContext context, Exception ex)`: 完成后处理

---

### 4.4 RequestContext

请求上下文，使用 ThreadLocal 存储当前请求的信息。

**属性**:
- `request`: HttpServletRequest
- `response`: HttpServletResponse
- `apiInfo`: 当前接口信息
- `user`: 当前用户信息
- `variables`: 请求变量 Map

---

## 5. 模块依赖

### 5.1 内部依赖

| 依赖模块 | 说明 |
|----------|------|
| magic-script | 脚本引擎，用于执行接口脚本 |
| magic-api-servlet | Servlet 适配层 (可选) |

### 5.2 外部依赖

| 依赖 | 版本 | 用途 |
|------|------|------|
| Spring Boot | 2.4.5+ | 基础框架 |
| FastJSON | 1.2.x | JSON 处理 |
| SLF4J | 1.7.x | 日志门面 |

---

## 6. 配置项

### 6.1 基础配置

```properties
# Web 页面入口
magic-api.web=/magic/web

# 接口路径前缀
magic-api.prefix=

# 密钥 (集群部署时必须相同)
magic-api.secret-key=
```

### 6.2 安全配置

```properties
# 启用安全
magic-api.security.enabled=true

# Token 超时时间
magic-api.security.token-timeout=30m

# 跨域配置
magic-api.security.cors.enabled=true
```

### 6.3 缓存配置

```properties
# 启用缓存
magic-api.cache.enabled=true

# 缓存过期时间 (秒)
magic-api.cache.expire=3600
```

### 6.4 备份配置

```properties
# 启用备份
magic-api.backup.enabled=true

# 保留版本数
magic-api.backup.keep-versions=10
```

---

## 7. 接口清单

核心模块本身不直接提供 REST API，但提供以下内部接口：

| 接口 | 说明 |
|------|------|
| MagicDynamicRegistry | 接口动态注册接口 |
| RequestInterceptor | 请求拦截器接口 |
| ResultProvider | 结果提供者接口 |
| MagicResourceStorage | 资源存储接口 |
| MagicNotifyService | 通知服务接口 |

---

## 8. 验收标准

### 8.1 功能验收

- [ ] 支持动态注册、更新、删除 HTTP 接口
- [ ] 支持接口脚本即时生效，无需重启
- [ ] 支持自定义拦截器
- [ ] 支持自定义结果提供者
- [ ] 支持多种资源存储方式
- [ ] 支持事件驱动架构

### 8.2 性能验收

- [ ] 接口注册延迟 < 100ms
- [ ] 核心框架处理开销 < 50ms
- [ ] 单节点支持 1000+ QPS

### 8.3 质量验收

- [ ] 代码符合 Java 编码规范
- [ ] 关键类有完整的 Javadoc 注释
- [ ] 单元测试覆盖率 > 80%

---

## 9. 参考资料

- [magic-api 官方文档](https://ssssssss.org)
- [magic-script 文档](https://gitee.com/ssssssss-team/magic-script)
- [Spring Boot 文档](https://spring.io/projects/spring-boot)

---

*文档生成时间：2026-03-14*  
*最后更新：2026-03-14*
