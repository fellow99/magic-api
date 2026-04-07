# 006-modules 模块规格说明书

**模块名称**: 模块集合 (Modules Collection)  
**模块路径**: magic-api/magic-api/src/main/java/org/ssssssss/magicapi/modules  
**创建时间**: 2026-03-14  
**状态**: 已完成  
**版本**: 2.2.2

---

## 1. 模块概述

### 1.1 模块定位

模块集合提供数据库、HTTP、Servlet、Spring 等核心功能模块，支持在脚本中访问各种系统资源和服务。

### 1.2 核心价值

- **数据库模块**: SQL 执行、查询、事务、多数据库方言支持
- **HTTP 模块**: HTTP 客户端功能，支持发起 HTTP 请求
- **Servlet 模块**: Servlet API 访问，支持 request/response/session 操作
- **Spring 模块**: Spring 框架集成，支持 Bean 访问和事件发布

---

## 2. 功能需求

### 2.1 数据库模块 (FR-DB)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-DB-001 | 查询执行 | P0 | SELECT 查询 |
| FR-DB-002 | 更新执行 | P0 | INSERT/UPDATE/DELETE |
| FR-DB-003 | 参数绑定 | P0 | 占位符参数绑定 |
| FR-DB-004 | 结果映射 | P0 | 结果集映射为 Map/List |
| FR-DB-005 | 分页查询 | P0 | 支持 page/size 参数 |
| FR-DB-006 | 自动计数 | P0 | 自动生成 COUNT 查询 |
| FR-DB-007 | 事务开始 | P0 | 开启事务 |
| FR-DB-008 | 事务提交 | P0 | 提交事务 |
| FR-DB-009 | 事务回滚 | P0 | 回滚事务 |
| FR-DB-010 | MySQL 方言 | P0 | MySQL 分页语法 |
| FR-DB-011 | Oracle 方言 | P1 | Oracle 分页语法 |
| FR-DB-012 | PostgreSQL 方言 | P1 | PostgreSQL 分页语法 |

### 2.2 HTTP 模块 (FR-HTTP)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-HTTP-001 | GET 请求 | P0 | 发起 GET 请求 |
| FR-HTTP-002 | POST 请求 | P0 | 发起 POST 请求 |
| FR-HTTP-003 | PUT 请求 | P1 | 发起 PUT 请求 |
| FR-HTTP-004 | DELETE 请求 | P1 | 发起 DELETE 请求 |
| FR-HTTP-005 | 请求头 | P0 | 设置请求头 |
| FR-HTTP-006 | 请求体 | P0 | 设置请求体 (JSON/Form) |
| FR-HTTP-007 | 查询参数 | P0 | 设置 URL 参数 |
| FR-HTTP-008 | 超时设置 | P1 | 设置超时时间 |
| FR-HTTP-009 | 响应解析 | P0 | 解析 JSON/XML 响应 |
| FR-HTTP-010 | 状态码检查 | P0 | 检查 HTTP 状态码 |
| FR-HTTP-011 | 错误处理 | P1 | 处理请求错误 |

### 2.3 Servlet 模块 (FR-SERVLET)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-SERVLET-001 | 访问 request 对象 | P0 | 获取 HttpServletRequest |
| FR-SERVLET-002 | 访问 response 对象 | P0 | 获取 HttpServletResponse |
| FR-SERVLET-003 | 访问 session 对象 | P0 | 获取 HttpSession |
| FR-SERVLET-004 | 文件下载支持 | P1 | 支持文件下载 |

### 2.4 Spring 模块 (FR-SPRING)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-SPRING-001 | Spring Bean 访问 | P0 | 获取 Spring Bean |
| FR-SPRING-002 | 应用上下文访问 | P0 | 获取 ApplicationContext |
| FR-SPRING-003 | 事件发布支持 | P1 | 发布应用事件 |
| FR-SPRING-004 | 环境配置访问 | P1 | 获取 Environment 配置 |

---

## 3. 接口清单

### 3.1 数据库模块脚本 API

```javascript
// 查询
db.select("select * from user where id = ?", userId)

// 更新
db.update("insert into user(name) values(?)", name)

// 分页
db.page("select * from user", page, size)

// 事务
db.transaction(() => {
    db.update("insert ...")
    db.update("update ...")
})
```

### 3.2 HTTP 模块脚本 API

```javascript
// GET 请求
http.get("https://api.example.com/users")

// POST 请求
http.post("https://api.example.com/users", {name: "test"})

// 带请求头
http.get("https://api.example.com/users", {
    headers: {"Authorization": "Bearer token"}
})

// 带超时
http.get("https://api.example.com/users", {
    timeout: 5000
})
```

### 3.3 Servlet 模块脚本 API

```javascript
// 获取请求参数
servlet.request.getParameter("name")
servlet.request.getHeader("Authorization")

// 设置响应
servlet.response.setStatus(200)
servlet.response.setHeader("Content-Type", "application/json")

// Session 操作
servlet.session.getAttribute("user")
servlet.session.setAttribute("user", user)

// 文件下载
servlet.download(file, "filename.pdf")
```

### 3.4 Spring 模块脚本 API

```javascript
// 获取 Spring Bean
spring.getBean("userService")
spring.bean("dataSource")

// 发布事件
spring.publishEvent(event)

// 获取环境配置
spring.env.getProperty("app.name")
```

---

## 4. 非功能需求

- 数据库查询延迟 < 50ms (简单查询)
- 支持连接池管理
- 支持 SQL 日志
- HTTP 请求超时默认 30 秒
- 支持 HTTPS
- 支持多种数据库方言

---

## 5. 配置项

```properties
# 数据库模块配置
magic-api.modules.db.enabled=true
magic-api.modules.db.show-sql=false
magic-api.modules.db.dialect=mysql

# HTTP 模块配置
magic-api.modules.http.enabled=true
magic-api.modules.http.timeout=30000

# Servlet 模块配置
magic-api.modules.servlet.enabled=true

# Spring 模块配置
magic-api.modules.spring.enabled=true
```

---

## 6. 验收标准

### 6.1 数据库模块
- [ ] 支持 SQL 查询和更新
- [ ] 支持分页查询
- [ ] 支持事务管理
- [ ] 支持多种数据库方言

### 6.2 HTTP 模块
- [ ] 支持 HTTP 请求方法
- [ ] 支持请求配置
- [ ] 支持响应处理

### 6.3 Servlet 模块
- [ ] 支持 Servlet API 访问
- [ ] 支持文件下载

### 6.4 Spring 模块
- [ ] 支持 Spring Bean 访问
- [ ] 支持事件发布

---

*文档生成时间：2026-03-14*
