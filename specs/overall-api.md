# magic-api 对外接口模型

> 版本：2.2.2  
> 生成时间：2026-03-14  
> 文档类型：接口模型

---

## 1. 文档概述

### 1.1 文档目的

本文档描述 magic-api 项目的对外接口模型，包括 API 分类、接口规范、请求响应格式等，为接口使用和集成提供依据。

### 1.2 接口分类

magic-api 提供两类接口：

1. **管理接口**：用于管理 magic-api 自身的资源（接口、函数、数据源等）
2. **业务接口**：用户通过 magic-api 创建的业务 API

---

## 2. 接口规范

### 2.1 基础路径

所有管理接口的基础路径为：

```
{server.context-path}/magic-api
```

**示例：**
- 默认：`http://localhost:9999/magic-api`
- 自定义 context-path：`http://localhost:9999/app/magic-api`

### 2.2 请求格式

**Content-Type：**
- `application/json` - JSON 数据
- `application/x-www-form-urlencoded` - 表单数据
- `multipart/form-data` - 文件上传

**字符编码：** `UTF-8`

### 2.3 响应格式

**统一响应结构：**

```json
{
  "code": 200,
  "message": "success",
  "data": {}
}
```

**字段说明：**

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 响应码 |
| message | String | 响应消息 |
| data | Object | 响应数据 |

**响应码定义：**

| 码值 | 说明 |
|------|------|
| 200 | 成功 |
| 400 | 参数错误 |
| 401 | 未登录 |
| 403 | 无权限 |
| 404 | 资源不存在 |
| 500 | 系统异常 |

### 2.4 认证方式

**Token 认证：**

1. 调用 `/login` 接口获取 Token
2. 在请求头中携带 Token

```http
Authorization: Bearer {token}
```

或

```http
MAGIC_TOKEN: {token}
```

**Session 认证：**

1. 调用 `/login` 接口登录
2. Cookie 自动保存 Session ID
3. 后续请求自动携带 Cookie

---

## 3. 管理接口清单

### 3.1 接口概览

| 模块 | 接口数量 | 说明 |
|------|---------|------|
| 核心工作台 | 18 | 编辑器配置、登录认证、类扫描、搜索等 |
| 资源管理 | 9 | API 文件/文件夹的增删改查、移动、锁定 |
| 备份管理 | 5 | 备份列表、还原、全量备份 |
| 数据源管理 | 1 | 数据源连接测试 |
| 任务管理 | 1 | 任务执行 |
| **总计** | **34** | |

---

## 4. 核心工作台接口

### 4.1 获取编辑器配置

**接口：** `GET /config.json`

**描述：** 获取编辑器配置信息

**请求参数：** 无

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "version": "2.2.2",
    "web": "/magic/web",
    "prefix": "/api",
    "security": {
      "username": null,
      "password": null
    },
    "responseCode": {
      "success": 200,
      "invalid": 400,
      "exception": 500
    }
  }
}
```

**认证：** 不需要

---

### 4.2 用户登录

**接口：** `POST /login`

**描述：** 用户登录

**请求参数：**

```json
{
  "username": "admin",
  "password": "admin123"
}
```

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "username": "admin",
      "role": "ADMIN"
    }
  }
}
```

**认证：** 不需要

---

### 4.3 获取当前用户信息

**接口：** `POST /user`

**描述：** 获取当前登录用户信息

**请求参数：** 无

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "username": "admin",
    "role": "ADMIN",
    "permissions": ["*"]
  }
}
```

**认证：** 需要

---

### 4.4 用户登出

**接口：** `POST /logout`

**描述：** 用户登出

**请求参数：** 无

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": null
}
```

**认证：** 需要

---

### 4.5 获取所有类信息

**接口：** `POST /classes`

**描述：** 获取所有 Java 类、扩展、函数信息（用于代码提示）

**请求参数：** 无

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "classes": [
      {
        "name": "java.lang.String",
        "methods": [...]
      }
    ],
    "extensions": [...],
    "functions": [...]
  }
}
```

**认证：** 需要

---

### 4.6 获取单个类信息

**接口：** `POST /class`

**描述：** 获取单个 Java 类的详细信息

**请求参数：**

```json
{
  "className": "java.lang.String"
}
```

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "name": "java.lang.String",
    "methods": [
      {
        "name": "length",
        "returnType": "int",
        "parameters": []
      },
      {
        "name": "substring",
        "returnType": "String",
        "parameters": [
          {"name": "beginIndex", "type": "int"},
          {"name": "endIndex", "type": "int"}
        ]
      }
    ]
  }
}
```

**认证：** 需要

---

### 4.7 获取资源树

**接口：** `POST /resource`

**描述：** 获取接口和函数的树形结构

**请求参数：** 无

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "id": "group_root",
      "name": "根分组",
      "type": "GROUP",
      "children": [
        {
          "id": "group_user",
          "name": "用户管理",
          "type": "GROUP",
          "children": [
            {
              "id": "api_123",
              "name": "getUserById",
              "type": "API",
              "path": "/user/:id",
              "method": "GET"
            }
          ]
        }
      ]
    }
  ]
}
```

**认证：** 需要

---

### 4.8 搜索接口

**接口：** `POST /search`

**描述：** 搜索接口和脚本内容

**请求参数：**

```json
{
  "keyword": "user"
}
```

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "id": "api_123",
      "name": "getUserById",
      "type": "API",
      "path": "/user/:id",
      "script": "return db.selectOne(...)",
      "matchType": "SCRIPT"
    }
  ]
}
```

**认证：** 需要

---

### 4.9 获取配置选项

**接口：** `GET /options`

**描述：** 获取配置选项列表（数据源、模块等）

**请求参数：** 无

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "dataSources": ["master", "slave"],
    "modules": ["db", "http", "response", "log"],
    "functions": ["pageQuery", "formatDate"]
  }
}
```

**认证：** 需要

---

### 4.10 重新加载资源

**接口：** `GET /reload`

**描述：** 重新加载资源（从文件系统或数据库）

**请求参数：** 无

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "count": 100
  }
}
```

**认证：** 需要

---

### 4.11 获取 TODO 列表

**接口：** `GET /todo`

**描述：** 获取所有接口中的 TODO/FIXME 注释

**请求参数：** 无

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "apiId": "api_123",
      "apiName": "getUserById",
      "line": 10,
      "content": "TODO: 添加缓存"
    }
  ]
}
```

**认证：** 需要

---

### 4.12 下载资源

**接口：** `GET /download`

**描述：** 下载接口资源（ZIP 格式）

**请求参数：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| ids | String | 否 | 资源 ID 列表，逗号分隔 |
| all | Boolean | 否 | 是否下载全部 |

**响应：** ZIP 文件

**认证：** 需要

---

### 4.13 上传资源

**接口：** `POST /upload`

**描述：** 上传接口资源（ZIP 格式）

**请求参数：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| file | File | 是 | ZIP 文件 |
| mode | String | 否 | 上传模式：full（全量）| incremental（增量） |

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "count": 10
  }
}
```

**认证：** 需要

---

### 4.14 推送资源

**接口：** `POST /push`

**描述：** 推送资源到目标环境

**请求参数：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| target | String | 是 | 目标地址 |
| secretKey | String | 是 | 秘钥 |
| mode | String | 否 | 推送模式 |
| ids | String | 否 | 资源 ID 列表 |

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "count": 10
  }
}
```

**认证：** 需要

---

### 4.15 接收推送

**接口：** `POST /receivePush`

**描述：** 接收推送的资源

**请求参数：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| file | File | 是 | ZIP 文件 |
| timestamp | Long | 是 | 时间戳 |
| sign | String | 是 | 签名 |
| mode | String | 否 | 推送模式 |

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "count": 10
  }
}
```

**认证：** 签名验证

---

### 4.16 获取插件列表

**接口：** `GET /plugins`

**描述：** 获取已安装的插件列表

**请求参数：** 无

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "name": "magic-api-plugin-redis",
      "version": "2.2.2",
      "description": "Redis 插件"
    },
    {
      "name": "magic-api-plugin-task",
      "version": "2.2.2",
      "description": "定时任务插件"
    }
  ]
}
```

**认证：** 需要

---

### 4.17 获取编辑器 JavaScript 配置

**接口：** `GET /config-js`

**描述：** 获取编辑器的 JavaScript 配置

**请求参数：** 无

**响应：** JavaScript 代码

**认证：** 需要

---

### 4.18 重定向到首页

**接口：** `GET /` 或 `GET /index.html`

**描述：** 重定向到编辑器首页

**请求参数：** 无

**响应：** 302 重定向

**认证：** 不需要

---

## 5. 资源管理接口

### 5.1 保存文件夹

**接口：** `POST /resource/folder/save`

**描述：** 创建或更新文件夹

**请求参数：**

```json
{
  "id": "group_user",
  "name": "用户管理",
  "parentId": "group_root",
  "type": "GROUP"
}
```

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": "group_user"
  }
}
```

**认证：** 需要

---

### 5.2 复制文件夹

**接口：** `POST /resource/folder/copy`

**描述：** 复制文件夹及其内容

**请求参数：**

```json
{
  "id": "group_user",
  "targetParentId": "group_root",
  "newName": "用户管理（副本）"
}
```

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": "group_user_copy"
  }
}
```

**认证：** 需要

---

### 5.3 删除资源

**接口：** `POST /resource/delete`

**描述：** 删除文件或文件夹

**请求参数：**

```json
{
  "id": "api_123"
}
```

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": null
}
```

**认证：** 需要

---

### 5.4 保存文件

**接口：** `POST /resource/file/{folderId}/save`

**描述：** 保存文件到指定文件夹

**路径参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| folderId | String | 文件夹 ID |

**请求参数：**

```json
{
  "id": "api_123",
  "name": "getUserById",
  "path": "/user/:id",
  "method": "GET",
  "script": "return db.selectOne(...);",
  "description": "根据 ID 获取用户信息",
  "parameters": [...],
  "options": {...}
}
```

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": "api_123"
  }
}
```

**认证：** 需要

---

### 5.5 获取文件详情

**接口：** `GET /resource/file/{id}`

**描述：** 获取文件详情

**路径参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| id | String | 文件 ID |

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": "api_123",
    "name": "getUserById",
    "path": "/user/:id",
    "method": "GET",
    "script": "return db.selectOne(...);",
    "description": "根据 ID 获取用户信息",
    "parameters": [...],
    "options": {...}
  }
}
```

**认证：** 需要

---

### 5.6 移动资源

**接口：** `POST /resource/move`

**描述：** 移动文件或文件夹

**请求参数：**

```json
{
  "id": "api_123",
  "targetParentId": "group_order"
}
```

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": null
}
```

**认证：** 需要

---

### 5.7 锁定文件

**接口：** `POST /resource/lock`

**描述：** 锁定文件防止误修改

**请求参数：**

```json
{
  "id": "api_123",
  "password": "123456"
}
```

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": null
}
```

**认证：** 需要

---

### 5.8 解锁文件

**接口：** `POST /resource/unlock`

**描述：** 解锁文件

**请求参数：**

```json
{
  "id": "api_123",
  "password": "123456"
}
```

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": null
}
```

**认证：** 需要

---

## 6. 备份管理接口

### 6.1 获取备份列表

**接口：** `GET /backups`

**描述：** 获取备份列表

**请求参数：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| resourceId | String | 否 | 资源 ID |
| page | Integer | 否 | 页码 |
| pageSize | Integer | 否 | 每页大小 |

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "list": [
      {
        "id": "backup_123",
        "resourceId": "api_123",
        "resourceName": "getUserById",
        "backupTime": 1710403200000,
        "operator": "admin"
      }
    ],
    "total": 10
  }
}
```

**认证：** 需要

---

### 6.2 获取备份详情

**接口：** `GET /backup/{id}`

**描述：** 获取指定备份的详情

**路径参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| id | String | 备份 ID |

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": "backup_123",
    "resourceId": "api_123",
    "content": "{...}",
    "backupTime": 1710403200000,
    "operator": "admin"
  }
}
```

**认证：** 需要

---

### 6.3 还原备份

**接口：** `POST /backup/rollback`

**描述：** 还原备份

**请求参数：**

```json
{
  "backupId": "backup_123"
}
```

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": null
}
```

**认证：** 需要

---

### 6.4 获取备份脚本内容

**接口：** `GET /backup`

**描述：** 获取备份的脚本内容

**请求参数：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| backupId | String | 是 | 备份 ID |

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": "return db.selectOne(...);"
}
```

**认证：** 需要

---

### 6.5 全量备份

**接口：** `POST /backup/full`

**描述：** 执行全量备份

**请求参数：** 无

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "count": 100
  }
}
```

**认证：** 需要

---

## 7. 数据源管理接口

### 7.1 测试数据源连接

**接口：** `GET/POST /datasource/jdbc/test`

**描述：** 测试 JDBC 数据源连接

**请求参数：**

```json
{
  "url": "jdbc:mysql://localhost:3306/test",
  "username": "root",
  "password": "123456",
  "driverClassName": "com.mysql.cj.jdbc.Driver"
}
```

**响应示例：**

```json
{
  "code": 200,
  "message": "连接成功",
  "data": null
}
```

**认证：** 需要

---

## 8. 任务管理接口

### 8.1 执行任务

**接口：** `POST /task/execute`

**描述：** 手动执行定时任务（需要 task 插件）

**请求参数：**

```json
{
  "taskId": "task_123"
}
```

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "result": "执行成功"
  }
}
```

**认证：** 需要

---

## 9. WebSocket 接口

### 9.1 WebSocket 连接

**端点：** `ws://{host}:{port}/magic/web/console`

**描述：** 建立 WebSocket 连接，用于实时调试和协作

**连接参数：** 无

**消息格式：**

```json
{
  "type": "DEBUG",
  "clientId": "client_123",
  "scriptId": "api_123",
  "content": {...}
}
```

**消息类型：**

| 类型 | 说明 |
|------|------|
| DEBUG | 调试命令 |
| LOG | 日志消息 |
| BREAKPOINT | 断点信息 |
| EVENT | 事件通知 |

---

## 10. 业务接口示例

### 10.1 用户管理接口

**创建用户：**

```http
POST /api/user
Content-Type: application/json

{
  "userName": "张三",
  "email": "zhangsan@example.com",
  "age": 25
}
```

**响应：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "userName": "张三",
    "email": "zhangsan@example.com"
  }
}
```

**获取用户列表：**

```http
GET /api/user?page=1&pageSize=10
```

**响应：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "list": [...],
    "total": 100,
    "page": 1,
    "pageSize": 10
  }
}
```

**获取用户详情：**

```http
GET /api/user/1
```

**响应：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "userName": "张三",
    "email": "zhangsan@example.com"
  }
}
```

**更新用户：**

```http
PUT /api/user/1
Content-Type: application/json

{
  "userName": "李四",
  "email": "lisi@example.com"
}
```

**删除用户：**

```http
DELETE /api/user/1
```

---

## 11. 错误处理

### 11.1 错误响应格式

```json
{
  "code": 400,
  "message": "参数错误：id 不能为空",
  "data": null
}
```

### 11.2 常见错误码

| 错误码 | 说明 | 处理建议 |
|--------|------|---------|
| 400 | 参数错误 | 检查请求参数 |
| 401 | 未登录 | 先调用登录接口 |
| 403 | 无权限 | 检查用户权限 |
| 404 | 资源不存在 | 检查资源 ID |
| 500 | 系统异常 | 查看日志排查原因 |

---

## 12. 总结

本文档描述了 magic-api 的对外接口模型，包括：

1. **接口规范**：基础路径、请求格式、响应格式、认证方式
2. **管理接口**：核心工作台、资源管理、备份管理、数据源管理、任务管理
3. **WebSocket 接口**：实时调试和协作
4. **业务接口示例**：用户管理接口示例
5. **错误处理**：错误响应格式和常见错误码

这些接口是 magic-api 管理和使用的基础，理解这些接口有助于进行系统集成和二次开发。
