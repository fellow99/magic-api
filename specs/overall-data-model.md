# magic-api 核心数据模型

> 版本：2.2.2  
> 生成时间：2026-03-14  
> 文档类型：数据模型

---

## 1. 文档概述

### 1.1 文档目的

本文档描述 magic-api 项目的核心数据模型，包括实体定义、关系图、数据结构等，为开发实现和数据库设计提供依据。

### 1.2 数据模型概览

```
┌─────────────────────────────────────────────────────────────────┐
│                      magic-api 数据模型                          │
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│   核心实体模型    │ │   存储数据模型    │ │   运行时模型     │
│                  │ │                  │ │                  │
│ • ApiInfo        │ │ • Resource       │ │ • RequestEntity  │
│ • FunctionInfo   │ │ • Backup         │ │ • MagicContext   │
│ • GroupInfo      │ │ • DataSource     │ │ • ScriptContext  │
│ • ParameterInfo  │ │ • User           │ │ • SessionContext │
│ • HeaderInfo     │ │                  │ │                  │
└──────────────────┘ └──────────────────┘  └──────────────────┘
```

---

## 2. 核心实体模型

### 2.1 ApiInfo（接口信息）

**描述：** 表示一个 API 接口的完整定义

**类图：**
```java
public class ApiInfo extends PathMagicEntity {
    private String id;                    // 接口 ID
    private String name;                  // 接口名称
    private String path;                  // 接口路径
    private String method;                // HTTP 方法
    private String script;                // 脚本内容
    private String description;           // 描述
    private String groupId;               // 所属分组 ID
    private List<ParameterInfo> parameters;  // 参数列表
    private List<ParameterInfo> headers;     // 请求头列表
    private BodyDefinition requestBody;   // 请求体定义
    private Map<String, Object> options;  // 高级选项
    private Long createTime;              // 创建时间
    private Long updateTime;              // 更新时间
    private String createBy;              // 创建人
    private String updateBy;              // 更新人
    private Boolean locked;               // 是否锁定
}
```

**字段说明：**

| 字段 | 类型 | 必填 | 说明 | 示例 |
|------|------|------|------|------|
| id | String | 是 | 接口唯一标识 | "api_123456" |
| name | String | 是 | 接口名称 | "getUserById" |
| path | String | 是 | 接口路径 | "/user/:id" |
| method | String | 是 | HTTP 方法 | "GET", "POST" |
| script | String | 是 | 脚本内容 | "return db.selectOne(...)" |
| description | String | 否 | 接口描述 | "根据 ID 获取用户信息" |
| groupId | String | 是 | 所属分组 ID | "group_user" |
| parameters | List | 否 | 参数定义列表 | 见 ParameterInfo |
| headers | List | 否 | 请求头定义列表 | 见 ParameterInfo |
| requestBody | BodyDefinition | 否 | 请求体定义 | 见 BodyDefinition |
| options | Map | 否 | 高级选项 | {"require-login": true} |
| createTime | Long | 是 | 创建时间戳 | 1710403200000 |
| updateTime | Long | 是 | 更新时间戳 | 1710403200000 |
| createBy | String | 否 | 创建人 | "admin" |
| updateBy | String | 否 | 更新人 | "admin" |
| locked | Boolean | 否 | 是否锁定 | false |

**高级选项 (options)：**

| 选项 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| require-login | Boolean | false | 是否需要登录 |
| disabled-unknown-parameter | Boolean | false | 是否禁用未知参数 |
| wrap-request-parameters | String | - | 请求参数包装名 |
| default-data-source | String | - | 默认数据源 |
| disabled-validate-request-body | Boolean | false | 是否禁用请求体验证 |

---

### 2.2 FunctionInfo（函数信息）

**描述：** 表示一个可复用的脚本函数

**类图：**
```java
public class FunctionInfo extends MagicEntity {
    private String id;                    // 函数 ID
    private String name;                  // 函数名称
    private String path;                  // 函数路径
    private List<String> parameters;      // 参数名列表
    private String script;                // 脚本内容
    private String description;           // 描述
    private String groupId;               // 所属分组 ID
    private Long createTime;              // 创建时间
    private Long updateTime;              // 更新时间
    private String createBy;              // 创建人
    private String updateBy;              // 更新人
}
```

**字段说明：**

| 字段 | 类型 | 必填 | 说明 | 示例 |
|------|------|------|------|------|
| id | String | 是 | 函数唯一标识 | "func_123456" |
| name | String | 是 | 函数名称 | "pageQuery" |
| path | String | 是 | 函数路径 | "common/pageQuery" |
| parameters | List | 否 | 参数名列表 | ["sql", "params", "page", "pageSize"] |
| script | String | 是 | 脚本内容 | "return db.selectPage(...)" |
| description | String | 否 | 函数描述 | "分页查询通用函数" |
| groupId | String | 是 | 所属分组 ID | "group_common" |
| createTime | Long | 是 | 创建时间戳 | 1710403200000 |
| updateTime | Long | 是 | 更新时间戳 | 1710403200000 |
| createBy | String | 否 | 创建人 | "admin" |
| updateBy | String | 否 | 更新人 | "admin" |

---

### 2.3 GroupInfo（分组信息）

**描述：** 表示接口或函数的分组（文件夹）

**类图：**
```java
public class GroupInfo extends MagicEntity {
    private String id;                    // 分组 ID
    private String name;                  // 分组名称
    private String parentId;              // 父分组 ID
    private String path;                  // 分组路径
    private String type;                  // 类型：API | FUNCTION
    private List<String> children;        // 子分组 ID 列表
    private List<String> files;           // 文件 ID 列表
    private Long createTime;              // 创建时间
    private Long updateTime;              // 更新时间
}
```

**字段说明：**

| 字段 | 类型 | 必填 | 说明 | 示例 |
|------|------|------|------|------|
| id | String | 是 | 分组唯一标识 | "group_user" |
| name | String | 是 | 分组名称 | "用户管理" |
| parentId | String | 否 | 父分组 ID（根分组为 null） | "group_root" |
| path | String | 是 | 分组路径 | "/user" |
| type | String | 是 | 分组类型 | "API", "FUNCTION" |
| children | List | 否 | 子分组 ID 列表 | ["group_user_1", "group_user_2"] |
| files | List | 否 | 文件（接口/函数）ID 列表 | ["api_1", "api_2"] |
| createTime | Long | 是 | 创建时间戳 | 1710403200000 |
| updateTime | Long | 是 | 更新时间戳 | 1710403200000 |

---

### 2.4 ParameterInfo（参数信息）

**描述：** 表示接口参数、请求头或请求体字段的定义

**类图：**
```java
public class ParameterInfo extends BaseDefinition {
    private String name;                  // 参数名
    private String alias;                 // 参数别名
    private DataType dataType;            // 数据类型
    private Boolean required;             // 是否必填
    private Object defaultValue;          // 默认值
    private String validateType;          // 验证类型
    private String expression;            // 验证表达式
    private String pattern;               // 正则表达式
    private String error;                 // 错误提示
    private String description;           // 描述
    private List<ParameterInfo> children; // 子参数（对象类型）
}
```

**字段说明：**

| 字段 | 类型 | 必填 | 说明 | 示例 |
|------|------|------|------|------|
| name | String | 是 | 参数名 | "id" |
| alias | String | 否 | 参数别名 | "userId" |
| dataType | DataType | 是 | 数据类型 | "Integer", "String" |
| required | Boolean | 是 | 是否必填 | true |
| defaultValue | Object | 否 | 默认值 | "10" |
| validateType | String | 否 | 验证类型 | "pattern", "expression" |
| expression | String | 否 | 验证表达式 | "$.length > 0" |
| pattern | String | 否 | 正则表达式 | "^[0-9]+$" |
| error | String | 否 | 错误提示 | "ID 必须为正整数" |
| description | String | 否 | 参数描述 | "用户 ID" |
| children | List | 否 | 子参数（对象类型时使用） | 见 ParameterInfo |

**数据类型 (DataType)：**

| 类型 | 说明 | 示例 |
|------|------|------|
| String | 字符串 | "hello" |
| Integer | 整数 | 123 |
| Long | 长整数 | 1234567890 |
| Double | 浮点数 | 3.14 |
| Boolean | 布尔值 | true |
| Date | 日期 | "2024-01-01" |
| DateTime | 日期时间 | "2024-01-01 12:00:00" |
| Array | 数组 | [1, 2, 3] |
| Object | 对象 | {"name": "张三"} |
| MultipartFile | 单文件上传 | - |
| MultipartFiles | 多文件上传 | - |
| Any | 任意类型 | - |

---

### 2.5 BodyDefinition（请求体定义）

**描述：** 表示请求体的结构定义

**类图：**
```java
public class BodyDefinition extends BaseDefinition {
    private String name;                  // 名称
    private DataType dataType;            // 数据类型
    private List<ParameterInfo> children; // 子字段
}
```

**示例：**

```json
{
  "name": "root",
  "dataType": "Object",
  "children": [
    {
      "name": "userName",
      "dataType": "String",
      "required": true,
      "description": "用户名"
    },
    {
      "name": "email",
      "dataType": "String",
      "required": false,
      "pattern": "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$",
      "description": "邮箱"
    },
    {
      "name": "age",
      "dataType": "Integer",
      "required": false,
      "defaultValue": 18,
      "description": "年龄"
    }
  ]
}
```

---

### 2.6 MagicEntity（实体基类）

**描述：** 所有实体的基类

**类图：**
```java
public abstract class MagicEntity {
    private String id;                    // 实体 ID
    private String name;                  // 名称
    private Long createTime;              // 创建时间
    private Long updateTime;              // 更新时间
    private String createBy;              // 创建人
    private String updateBy;              // 更新人
}
```

---

## 3. 存储数据模型

### 3.1 资源存储表 (magic_api_resource)

**描述：** 存储接口、函数、分组的资源数据（数据库存储模式）

**表结构：**

```sql
CREATE TABLE magic_api_resource (
    id VARCHAR(64) PRIMARY KEY COMMENT '资源 ID',
    name VARCHAR(255) NOT NULL COMMENT '资源名称',
    type VARCHAR(20) NOT NULL COMMENT '资源类型：API | FUNCTION | GROUP',
    content TEXT COMMENT '资源内容（JSON 格式）',
    parent_id VARCHAR(64) COMMENT '父资源 ID',
    path VARCHAR(512) COMMENT '资源路径',
    create_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    update_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    create_by VARCHAR(64) COMMENT '创建人',
    update_by VARCHAR(64) COMMENT '更新人',
    locked TINYINT(1) DEFAULT 0 COMMENT '是否锁定',
    INDEX idx_type (type),
    INDEX idx_parent_id (parent_id),
    INDEX idx_path (path(255))
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='magic-api 资源表';
```

**content 字段格式（API 类型）：**

```json
{
  "id": "api_123456",
  "name": "getUserById",
  "path": "/user/:id",
  "method": "GET",
  "script": "return db.selectOne(\"SELECT * FROM user WHERE id = ?\", [id]);",
  "description": "根据 ID 获取用户信息",
  "groupId": "group_user",
  "parameters": [...],
  "headers": [...],
  "requestBody": {...},
  "options": {...}
}
```

---

### 3.2 备份表 (magic_api_backup)

**描述：** 存储接口备份数据

**表结构：**

```sql
CREATE TABLE magic_api_backup (
    id VARCHAR(64) PRIMARY KEY COMMENT '备份 ID',
    resource_id VARCHAR(64) NOT NULL COMMENT '资源 ID',
    resource_name VARCHAR(255) NOT NULL COMMENT '资源名称',
    resource_type VARCHAR(20) NOT NULL COMMENT '资源类型：API | FUNCTION',
    content TEXT NOT NULL COMMENT '备份内容',
    backup_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '备份时间',
    operator VARCHAR(64) COMMENT '操作人',
    remark VARCHAR(255) COMMENT '备注',
    INDEX idx_resource_id (resource_id),
    INDEX idx_backup_time (backup_time)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='magic-api 备份表';
```

---

### 3.3 数据源配置表 (magic_api_datasource)

**描述：** 存储数据源配置信息（可选，通常使用 application.yml 配置）

**表结构：**

```sql
CREATE TABLE magic_api_datasource (
    id VARCHAR(64) PRIMARY KEY COMMENT '数据源 ID',
    name VARCHAR(64) NOT NULL COMMENT '数据源名称',
    url VARCHAR(512) NOT NULL COMMENT 'JDBC URL',
    username VARCHAR(128) NOT NULL COMMENT '用户名',
    password VARCHAR(256) NOT NULL COMMENT '密码（加密）',
    driver_class_name VARCHAR(256) NOT NULL COMMENT '驱动类名',
    pool_size INT DEFAULT 10 COMMENT '连接池大小',
    create_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    update_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    UNIQUE KEY uk_name (name)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='magic-api 数据源配置表';
```

---

### 3.4 用户表 (magic_api_user)

**描述：** 存储系统用户信息（可选，通常使用自定义认证）

**表结构：**

```sql
CREATE TABLE magic_api_user (
    id VARCHAR(64) PRIMARY KEY COMMENT '用户 ID',
    username VARCHAR(64) NOT NULL COMMENT '用户名',
    password VARCHAR(256) NOT NULL COMMENT '密码（加密）',
    nickname VARCHAR(64) COMMENT '昵称',
    email VARCHAR(128) COMMENT '邮箱',
    role VARCHAR(32) DEFAULT 'USER' COMMENT '角色',
    status TINYINT(1) DEFAULT 1 COMMENT '状态：0-禁用 1-启用',
    create_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    update_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    UNIQUE KEY uk_username (username)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='magic-api 用户表';
```

---

### 3.5 操作日志表 (magic_api_operation_log)

**描述：** 存储用户操作日志（可选）

**表结构：**

```sql
CREATE TABLE magic_api_operation_log (
    id VARCHAR(64) PRIMARY KEY COMMENT '日志 ID',
    operator VARCHAR(64) NOT NULL COMMENT '操作人',
    operation VARCHAR(32) NOT NULL COMMENT '操作类型：CREATE | UPDATE | DELETE | LOGIN | LOGOUT',
    resource_type VARCHAR(20) COMMENT '资源类型：API | FUNCTION | GROUP',
    resource_id VARCHAR(64) COMMENT '资源 ID',
    resource_name VARCHAR(255) COMMENT '资源名称',
    content TEXT COMMENT '操作内容',
    ip VARCHAR(64) COMMENT '操作 IP',
    create_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '操作时间',
    INDEX idx_operator (operator),
    INDEX idx_operation (operation),
    INDEX idx_create_time (create_time)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='magic-api 操作日志表';
```

---

## 4. 运行时模型

### 4.1 RequestEntity（请求实体）

**描述：** 封装 HTTP 请求的运行时信息

**类图：**
```java
public class RequestEntity {
    private ApiInfo apiInfo;                      // 接口信息
    private MagicHttpServletRequest request;      // 请求对象
    private MagicHttpServletResponse response;    // 响应对象
    private Map<String, Object> parameters;       // 请求参数
    private Map<String, Object> pathVariables;    // 路径变量
    private Map<String, Object> headers;          // 请求头
    private Object requestBody;                   // 请求体
    private MagicScriptContext magicScriptContext;// 脚本上下文
    private DebugRequest debugRequest;            // 调试请求
    private Boolean requestedFromTest;            // 是否来自测试
}
```

---

### 4.2 MagicScriptContext（脚本上下文）

**描述：** 脚本执行时的变量上下文

**类图：**
```java
public class MagicScriptContext {
    private String scriptName;                    // 脚本名称
    private Map<String, Object> variables;        // 变量集合
    
    // 内置变量
    public Map<String, Object> getParameters();   // 请求参数
    public Map<String, Object> getPathVariables();// 路径变量
    public Map<String, Object> getHeader();       // 请求头
    public CookieContext getCookie();             // Cookie 上下文
    public SessionContext getSession();           // Session 上下文
    public Object getRequestBody();               // 请求体
    public ApiInfo getApiInfo();                 // 接口信息
    public ResponseModule getResponse();          // 响应模块
    public SQLModule getDb();                     // 数据库模块
    public HTTPModule getHttp();                  // HTTP 模块
}
```

**变量作用域：**

```
┌─────────────────────────────────────────┐
│           全局作用域                     │
│  (所有脚本可访问的内置模块和函数)         │
│  - db, http, response, log, ...         │
└─────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────┐
│           脚本作用域                     │
│  (当前脚本定义的变量)                    │
│  - var a = 1;                           │
│  - function foo() {}                    │
└─────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────┐
│           函数作用域                     │
│  (函数内部定义的变量)                    │
│  - function bar() {                     │
│      var local = 2;                     │
│    }                                    │
└─────────────────────────────────────────┘
```

---

### 4.3 SessionContext（会话上下文）

**描述：** 封装 Session 操作

**类图：**
```java
public class SessionContext {
    private HttpSession session;
    
    public Object get(String key);
    public void set(String key, Object value);
    public void remove(String key);
    public void invalidate();
    public long getCreationTime();
    public long getLastAccessedTime();
    public String getId();
}
```

---

### 4.4 CookieContext（Cookie 上下文）

**描述：** 封装 Cookie 操作

**类图：**
```java
public class CookieContext {
    private MagicHttpServletRequest request;
    
    public String get(String name);
    public Cookie[] getAll();
    public void add(Cookie cookie);
    public void remove(String name);
}
```

---

### 4.5 DebugRequest（调试请求）

**描述：** 封装调试相关信息

**类图：**
```java
public class DebugRequest {
    private String requestedClientId;         // 客户端 ID
    private String requestedScriptId;         // 脚本 ID
    private List<Integer> breakpoints;        // 断点列表
    private Integer timeout;                  // 超时时间
    
    public MagicScriptDebugContext createMagicScriptContext(int timeout);
}
```

---

## 5. 实体关系图

### 5.1 核心实体关系

```
┌──────────────┐       ┌──────────────┐
│   GroupInfo  │       │   GroupInfo  │
│   (分组)      │       │   (分组)      │
└──────┬───────┘       └──────┬───────┘
       │ 1:N                  │ 1:N
       ▼                      ▼
┌──────────────┐       ┌──────────────┐
│   ApiInfo    │       │ FunctionInfo │
│   (接口)      │       │   (函数)      │
└──────────────┘       └──────────────┘
       │                        │
       │ N:M                    │ N:M
       ▼                        ▼
┌──────────────┐       ┌──────────────┐
│ ParameterInfo│       │ ParameterInfo│
│   (参数)      │       │   (参数)      │
└──────────────┘       └──────────────┘
```

### 5.2 存储关系

```
┌──────────────┐
│   Resource   │
│   (资源表)    │
└──────┬───────┘
       │ 1:N
       ▼
┌──────────────┐
│   Backup     │
│   (备份表)    │
└──────────────┘
```

### 5.3 运行时关系

```
┌──────────────┐
│ RequestEntity│
│  (请求实体)   │
└──────┬───────┘
       │ 1:1
       ▼
┌──────────────┐
│ ApiInfo      │
│  (接口信息)   │
└──────────────┘

┌──────────────┐
│ RequestEntity│
│  (请求实体)   │
└──────┬───────┘
       │ 1:1
       ▼
┌──────────────┐
│MagicScriptCtx│
│ (脚本上下文)  │
└──────┬───────┘
       │ 1:1
       ▼
┌──────────────┐
│ SessionContext│
│ (会话上下文)  │
└──────────────┘
```

---

## 6. 数据字典

### 6.1 资源类型 (resource_type)

| 值 | 说明 |
|----|------|
| API | 接口 |
| FUNCTION | 函数 |
| GROUP | 分组 |

### 6.2 HTTP 方法 (method)

| 值 | 说明 |
|----|------|
| GET | 获取资源 |
| POST | 创建资源 |
| PUT | 更新资源（全量） |
| PATCH | 更新资源（部分） |
| DELETE | 删除资源 |
| HEAD | 获取资源头 |
| OPTIONS | 获取支持的方法 |

### 6.3 操作类型 (operation)

| 值 | 说明 |
|----|------|
| CREATE | 创建 |
| UPDATE | 更新 |
| DELETE | 删除 |
| LOGIN | 登录 |
| LOGOUT | 登出 |
| IMPORT | 导入 |
| EXPORT | 导出 |
| BACKUP | 备份 |
| ROLLBACK | 还原 |

### 6.4 角色类型 (role)

| 值 | 说明 |
|----|------|
| ADMIN | 管理员 |
| USER | 普通用户 |
| GUEST | 访客 |

---

## 7. 总结

本文档描述了 magic-api 的核心数据模型，包括：

1. **核心实体模型**：ApiInfo、FunctionInfo、GroupInfo、ParameterInfo 等
2. **存储数据模型**：资源表、备份表、数据源表、用户表、日志表
3. **运行时模型**：RequestEntity、MagicScriptContext、SessionContext 等
4. **实体关系**：各实体之间的关系图

这些数据模型是 magic-api 实现的基础，理解这些模型有助于深入理解系统架构和进行二次开发。
