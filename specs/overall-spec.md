# magic-api 整体功能规格说明书

> 版本：2.2.2  
> 生成时间：2026-03-14  
> 文档类型：功能规格

---

## 1. 文档概述

### 1.1 文档目的

本文档描述 magic-api 项目的整体功能规格，包括用户需求、功能模块划分、功能详细描述，为开发、测试、验收提供依据。

### 1.2 适用范围

- 项目开发人员：了解功能需求，进行开发实现
- 测试人员：根据功能规格编写测试用例
- 产品经理：确认功能完整性
- 用户：了解系统能力

### 1.3 术语定义

| 术语 | 定义 |
|------|------|
| magic-api | 基于 Java 的接口快速开发框架 |
| magic-script | magic-api 使用的动态脚本引擎 |
| 接口 | 通过 magic-api 创建的 HTTP API |
| 函数 | 可复用的脚本逻辑单元 |
| 数据源 | 数据库连接配置 |
| 模块 | magic-script 的功能扩展单元 |

---

## 2. 用户需求

### 2.1 用户画像

#### 2.1.1 后端开发人员

**特征：**
- 熟悉 Java 开发
- 需要快速开发 API 接口
- 希望减少样板代码

**需求：**
- 快速创建 CRUD 接口
- 支持复杂业务逻辑
- 便于调试和测试
- 易于维护和扩展

#### 2.1.2 前端开发人员

**特征：**
- 熟悉 JavaScript/TypeScript
- 需要后端接口支持
- 希望接口文档清晰

**需求：**
- 快速获得可用接口
- 接口文档自动生成
- 支持 Mock 数据
- 接口变更及时通知

#### 2.1.3 测试人员

**特征：**
- 需要测试 API 功能
- 关注接口稳定性
- 需要测试数据

**需求：**
- 接口易于测试
- 支持测试数据准备
- 便于问题定位
- 接口行为可预测

#### 2.1.4 运维人员

**特征：**
- 关注系统稳定性
- 需要监控和日志
- 负责部署和备份

**需求：**
- 部署简单
- 支持集群
- 有备份还原机制
- 有完善的日志

### 2.2 用户场景

#### 场景一：快速创建 CRUD 接口

**用户故事：**
> 作为后端开发人员，我希望通过简单的配置就能创建用户的增删改查接口，这样我就不需要编写大量的样板代码。

**功能需求：**
- 可视化创建接口
- 自动映射 HTTP 方法
- 支持参数验证
- 支持分页查询

#### 场景二：实现复杂业务逻辑

**用户故事：**
> 作为后端开发人员，我希望在接口中编写复杂的业务逻辑，包括条件判断、循环、事务等，这样我就能处理各种业务场景。

**功能需求：**
- 支持脚本编程
- 支持条件分支
- 支持循环
- 支持事务管理
- 支持异常处理

#### 场景三：接口调试

**用户故事：**
> 作为开发人员，我希望在 Web 界面直接调试接口，查看执行过程和结果，这样我就能快速定位问题。

**功能需求：**
- 在线调试
- 断点设置
- 变量查看
- 执行日志
- 错误提示

#### 场景四：多数据源操作

**用户故事：**
> 作为开发人员，我希望在接口中操作多个数据库，这样我就能实现跨库查询和数据同步。

**功能需求：**
- 配置多个数据源
- 运行时切换数据源
- 支持跨库事务
- 支持不同数据库类型

#### 场景五：接口版本管理

**用户故事：**
> 作为开发人员，我希望管理接口的历史版本，在出错时能够快速回滚，这样我就能降低变更风险。

**功能需求：**
- 自动保存历史版本
- 版本对比
- 版本还原
- 版本备份

#### 场景六：团队协作

**用户故事：**
> 作为团队负责人，我希望多个开发人员可以协作开发接口，同时避免冲突，这样我们就能提高开发效率。

**功能需求：**
- 接口锁定
- 冲突检测
- 操作日志
- 权限管理

---

## 3. 功能模块划分

### 3.1 功能架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                        magic-api 功能架构                        │
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│   核心功能模块    │ │   扩展功能模块    │ │   支撑功能模块    │
│                  │ │                  │ │                  │
│ • 接口管理       │ │ • 多数据源       │ │ • 用户认证       │
│ • 函数管理       │ │ • 插件系统       │ │ • 权限控制       │
│ • 脚本执行       │ │ • 集群支持       │ │ • 日志记录       │
│ • 请求处理       │ │ • 文档生成       │ │ • 监控告警       │
│ • 响应处理       │ │ • 定时任务       │ │ • 备份还原       │
└──────────────────┘ └──────────────────┘ └──────────────────┘
```

### 3.2 功能模块清单

| 模块编号 | 模块名称 | 模块描述 | 优先级 |
|---------|---------|---------|--------|
| M01 | 接口管理 | 创建、编辑、删除、查询 API 接口 | P0 |
| M02 | 函数管理 | 创建、管理可复用的脚本函数 | P1 |
| M03 | 脚本执行 | magic-script 脚本解析和执行 | P0 |
| M04 | 请求处理 | HTTP 请求接收、参数解析、路由匹配 | P0 |
| M05 | 响应处理 | 响应格式化、异常处理、拦截器 | P0 |
| M06 | 数据源管理 | 多数据源配置、切换、管理 | P1 |
| M07 | 数据库模块 | SQL 执行、方言适配、事务管理 | P0 |
| M08 | HTTP 模块 | HTTP 客户端调用 | P1 |
| M09 | 备份管理 | 自动备份、手动备份、还原 | P1 |
| M10 | 用户认证 | 登录、登出、会话管理 | P0 |
| M11 | 权限控制 | 接口访问权限、操作权限 | P1 |
| M12 | 日志记录 | 请求日志、错误日志、操作日志 | P1 |
| M13 | 插件系统 | 插件加载、注册、扩展 | P2 |
| M14 | 集群支持 | 接口同步、会话共享 | P2 |
| M15 | 文档生成 | API 文档自动生成 | P2 |
| M16 | 定时任务 | 定时任务配置和执行 | P2 |
| M17 | Web 编辑器 | 可视化界面、代码编辑、调试 | P0 |
| M18 | 监控告警 | 性能监控、错误告警 | P2 |

---

## 4. 功能详细描述

### 4.1 核心功能模块

#### M01: 接口管理

**功能描述：**
提供 API 接口的全生命周期管理，包括创建、编辑、删除、查询、导入、导出。

**子功能：**

| 子功能 | 描述 | 输入 | 输出 |
|-------|------|------|------|
| F01-01 创建接口 | 创建新的 API 接口 | 路径、方法、脚本 | 接口 ID |
| F01-02 编辑接口 | 修改已有接口 | 接口 ID、新内容 | 更新结果 |
| F01-03 删除接口 | 删除接口 | 接口 ID | 删除结果 |
| F01-04 查询接口 | 查询接口详情 | 接口 ID | 接口信息 |
| F01-05 接口列表 | 获取接口树形列表 | 文件夹 ID | 接口树 |
| F01-06 移动接口 | 移动接口到指定文件夹 | 接口 ID、目标文件夹 | 移动结果 |
| F01-07 复制接口 | 复制接口 | 接口 ID | 新接口 ID |
| F01-08 锁定接口 | 锁定接口防止误修改 | 接口 ID | 锁定状态 |
| F01-09 解锁接口 | 解锁接口 | 接口 ID、密码 | 解锁状态 |
| F01-10 导入接口 | 从文件导入接口 | ZIP 文件 | 导入结果 |
| F01-11 导出接口 | 导出接口到文件 | 接口列表 | ZIP 文件 |

**接口配置项：**

| 配置项 | 类型 | 必填 | 默认值 | 说明 |
|-------|------|------|--------|------|
| path | String | 是 | - | 接口路径 |
| method | String | 是 | GET | HTTP 方法 |
| script | String | 是 | - | 脚本内容 |
| description | String | 否 | - | 接口描述 |
| parameters | Array | 否 | [] | 参数定义 |
| headers | Array | 否 | [] | 请求头定义 |
| requestBody | Object | 否 | - | 请求体定义 |
| options | Object | 否 | {} | 高级选项 |

**高级选项：**

| 选项 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| require-login | Boolean | false | 是否需要登录 |
| disabled-unknown-parameter | Boolean | false | 是否禁用未知参数 |
| wrap-request-parameters | String | - | 请求参数包装名 |
| default-data-source | String | - | 默认数据源 |
| disabled-validate-request-body | Boolean | false | 是否禁用请求体验证 |

---

#### M02: 函数管理

**功能描述：**
提供可复用脚本函数的管理，函数可以在多个接口中调用。

**子功能：**

| 子功能 | 描述 | 输入 | 输出 |
|-------|------|------|------|
| F02-01 创建函数 | 创建新函数 | 函数名、参数、脚本 | 函数 ID |
| F02-02 编辑函数 | 修改函数 | 函数 ID、新内容 | 更新结果 |
| F02-03 删除函数 | 删除函数 | 函数 ID | 删除结果 |
| F02-04 调用函数 | 在接口中调用函数 | 函数路径、参数 | 函数返回值 |

**函数定义：**

```javascript
// 函数定义示例
function pageQuery(sql, params, page, pageSize) {
    var total = db.selectOne("SELECT COUNT(*) FROM (" + sql + ") t", params).total;
    var list = db.selectList(sql + " LIMIT ?, ?", params, [(page - 1) * pageSize, pageSize]);
    return { list: list, total: total, page: page, pageSize: pageSize };
}

// 函数调用
var result = pageQuery("SELECT * FROM user WHERE status = ?", [1], 1, 10);
```

---

#### M03: 脚本执行

**功能描述：**
解析和执行 magic-script 脚本，提供脚本上下文管理。

**功能特性：**

- 词法分析和语法分析
- AST 生成和优化
- 字节码编译（可选缓存）
- 变量作用域管理
- 异常捕获和处理
- 断点调试支持

**脚本上下文变量：**

| 变量名 | 类型 | 说明 |
|-------|------|------|
| parameters | Map | 请求参数（URL 参数 + 表单参数） |
| pathVariables | Map | 路径变量 |
| header | Map | 请求头 |
| cookie | CookieContext | Cookie 上下文 |
| session | SessionContext | Session 上下文 |
| requestBody | Object | 请求体 |
| apiInfo | ApiInfo | 接口信息 |
| response | ResponseModule | 响应操作模块 |
| db | SQLModule | 数据库操作模块 |
| http | HTTPModule | HTTP 调用模块 |

---

#### M04: 请求处理

**功能描述：**
接收 HTTP 请求，进行路由匹配、参数解析、验证。

**处理流程：**

1. 接收 HTTP 请求
2. 路由匹配（方法 + 路径）
3. 读取请求参数
4. 读取请求头
5. 读取请求体
6. 参数验证
7. 执行前置拦截器
8. 执行脚本
9. 执行后置拦截器
10. 构建响应

**参数验证规则：**

| 规则 | 说明 | 示例 |
|------|------|------|
| required | 是否必填 | required: true |
| dataType | 数据类型 | dataType: "Integer" |
| defaultValue | 默认值 | defaultValue: "10" |
| expression | 验证表达式 | expression: "$.length > 0" |
| pattern | 正则表达式 | pattern: "^[0-9]+$" |
| error | 错误提示 | error: "ID 必须为正整数" |

---

#### M05: 响应处理

**功能描述：**
处理脚本执行结果，构建统一的响应格式。

**响应格式配置：**

```yaml
magic-api:
  response: '{"code":200,"message":"success","data":$}'
```

**变量说明：**
- `$` 表示脚本返回值

**内置响应码：**

| 码值 | 说明 |
|------|------|
| 200 | 成功 |
| 400 | 参数错误 |
| 401 | 未登录 |
| 403 | 无权限 |
| 404 | 接口不存在 |
| 500 | 系统异常 |

---

### 4.2 扩展功能模块

#### M06: 数据源管理

**功能描述：**
支持配置和管理多个数据库连接，支持运行时动态切换。

**支持的数据源类型：**

| 类型 | 驱动 | 说明 |
|------|------|------|
| MySQL | com.mysql.cj.jdbc.Driver | MySQL 数据库 |
| Oracle | oracle.jdbc.OracleDriver | Oracle 数据库 |
| PostgreSQL | org.postgresql.Driver | PostgreSQL 数据库 |
| SQL Server | com.microsoft.sqlserver.jdbc.SQLServerDriver | SQL Server 数据库 |
| DB2 | com.ibm.db2.jcc.DB2Driver | DB2 数据库 |
| Clickhouse | com.clickhouse.jdbc.ClickHouseDriver | Clickhouse 数据库 |
| DM | dm.jdbc.driver.DmDriver | 达梦数据库 |
| Kingbase | com.kingbase8.Driver | 人大金仓数据库 |

**数据源配置：**

```yaml
spring:
  datasource:
    dynamic:
      datasource:
        master:
          url: jdbc:mysql://localhost:3306/db_master
          username: root
          password: 123456
          driver-class-name: com.mysql.cj.jdbc.Driver
        slave:
          url: jdbc:mysql://localhost:3306/db_slave
          username: root
          password: 123456
          driver-class-name: com.mysql.cj.jdbc.Driver
```

**数据源切换：**

```javascript
// 切换到从库
db.useDataSource("slave");
var list = db.selectList("SELECT * FROM user");

// 切换回主库
db.useDataSource("master");
```

---

#### M07: 数据库模块

**功能描述：**
提供数据库操作功能，包括 SQL 执行、事务管理、分页查询等。

**API 列表：**

| 方法 | 描述 | 参数 | 返回值 |
|------|------|------|--------|
| selectOne | 查询单条记录 | sql, params | Map |
| selectList | 查询多条记录 | sql, params | List<Map> |
| selectPage | 分页查询 | sql, params, page, pageSize | PageResult |
| update | 执行更新 | sql, params | int (影响行数) |
| insert | 插入记录 | sql, params | long (主键 ID) |
| delete | 删除记录 | sql, params | int (影响行数) |
| transaction | 事务执行 | callback | callback 返回值 |
| useDataSource | 切换数据源 | dataSourceName | void |

**使用示例：**

```javascript
// 查询单条
var user = db.selectOne("SELECT * FROM user WHERE id = ?", [id]);

// 查询列表
var list = db.selectList("SELECT * FROM user WHERE status = ?", [1]);

// 分页查询
var page = db.selectPage("SELECT * FROM user", null, 1, 10);

// 更新
var count = db.update("UPDATE user SET name = ? WHERE id = ?", [name, id]);

// 插入
var id = db.insert("INSERT INTO user (name, email) VALUES (?, ?)", [name, email]);

// 删除
var count = db.delete("DELETE FROM user WHERE id = ?", [id]);

// 事务
db.transaction(() => {
    db.insert("INSERT INTO ...");
    db.update("UPDATE ...");
});
```

---

#### M08: HTTP 模块

**功能描述：**
提供 HTTP 客户端功能，支持调用外部 HTTP 接口。

**API 列表：**

| 方法 | 描述 | 参数 | 返回值 |
|------|------|------|--------|
| get | GET 请求 | url, params, headers | Response |
| post | POST 请求 | url, body, headers | Response |
| put | PUT 请求 | url, body, headers | Response |
| delete | DELETE 请求 | url, headers | Response |
| patch | PATCH 请求 | url, body, headers | Response |

**使用示例：**

```javascript
// GET 请求
var response = http.get("https://api.example.com/users", { page: 1 });

// POST 请求
var response = http.post("https://api.example.com/users", { name: "张三" });

// 带请求头
var response = http.get("https://api.example.com/users", null, { 
    "Authorization": "Bearer " + token 
});

// 获取响应内容
var data = response.data;
var status = response.status;
var headers = response.headers;
```

---

#### M09: 备份管理

**功能描述：**
提供接口备份和还原功能，支持自动备份和手动备份。

**备份类型：**

| 类型 | 触发条件 | 说明 |
|------|---------|------|
| 自动备份 | 接口保存时 | 自动保存历史版本 |
| 手动备份 | 用户主动触发 | 全量备份当前所有接口 |
| 定时清理 | 每天执行 | 清理过期备份 |

**备份配置：**

```yaml
magic-api:
  backup:
    enable: true                     # 是否启用备份
    datasource: master               # 备份数据源
    table-name: magic_api_backup     # 备份表名
    max-history: 30                  # 保留天数
```

**备份操作：**

| 操作 | 描述 | API 路径 |
|------|------|---------|
| 查看备份列表 | 获取所有备份记录 | GET /backups |
| 查看备份详情 | 获取指定备份内容 | GET /backup/{id} |
| 还原备份 | 还原到指定版本 | POST /backup/rollback |
| 全量备份 | 执行全量备份 | POST /backup/full |

---

### 4.3 支撑功能模块

#### M10: 用户认证

**功能描述：**
提供用户登录、登出、会话管理功能。

**认证流程：**

1. 用户输入用户名密码
2. 后端验证凭据
3. 生成会话 Token
4. 前端存储 Token
5. 后续请求携带 Token
6. 后端验证 Token 有效性

**认证配置：**

```yaml
magic-api:
  security:
    username: admin      # 登录用户名
    password: admin123   # 登录密码
```

**自定义认证：**

```java
@Configuration
public class CustomAuthConfig {
    @Bean
    public AuthorizationInterceptor authorizationInterceptor() {
        return new CustomAuthorizationInterceptor();
    }
}
```

---

#### M11: 权限控制

**功能描述：**
提供接口访问权限和操作权限控制。

**权限类型：**

| 权限类型 | 说明 | 配置方式 |
|---------|------|---------|
| 登录验证 | 需要登录才能访问 | 接口选项：require-login = true |
| 角色权限 | 特定角色才能访问 | 自定义拦截器 |
| 操作权限 | 编辑、删除等操作权限 | 接口锁定 |

**拦截器示例：**

```java
public class RoleInterceptor implements RequestInterceptor {
    @Override
    public Object preHandle(RequestEntity requestEntity) {
        User user = (User) requestEntity.getSession().get("user");
        if (user == null || !"ADMIN".equals(user.getRole())) {
            return JsonBean.error(403, "无权限");
        }
        return null;
    }
}
```

---

#### M12: 日志记录

**功能描述：**
记录请求日志、错误日志、操作日志。

**日志类型：**

| 日志类型 | 内容 | 用途 |
|---------|------|------|
| 请求日志 | 请求方法、路径、参数、响应时间 | 性能分析 |
| 错误日志 | 异常堆栈、错误信息 | 问题排查 |
| 操作日志 | 用户操作记录 | 审计追踪 |
| SQL 日志 | 执行的 SQL 语句 | SQL 优化 |

**日志配置：**

```yaml
logging:
  level:
    org.ssssssss.magicapi: DEBUG
  
magic-api:
  show-sql: true       # 是否打印 SQL
```

---

#### M17: Web 编辑器

**功能描述：**
提供可视化的接口开发界面，包括代码编辑、调试、管理功能。

**功能清单：**

| 功能 | 描述 |
|------|------|
| 代码编辑 | Monaco Editor，支持语法高亮、自动提示 |
| 接口管理 | 树形结构展示，支持增删改查 |
| 在线调试 | 断点、单步执行、变量查看 |
| 参数配置 | 可视化配置接口参数、请求头、请求体 |
| 历史版本 | 查看和还原历史版本 |
| 全局搜索 | 搜索接口和脚本内容 |
| TODO 列表 | 查看代码中的 TODO/FIXME 注释 |
| 数据源管理 | 配置和管理数据源 |
| 函数管理 | 管理可复用函数 |

**访问地址：**
```
http://localhost:9999/magic/web
```

---

## 5. 功能优先级

### P0 - 核心功能（必须实现）

- M01: 接口管理
- M03: 脚本执行
- M04: 请求处理
- M05: 响应处理
- M07: 数据库模块
- M10: 用户认证
- M17: Web 编辑器

### P1 - 重要功能（应该实现）

- M02: 函数管理
- M06: 数据源管理
- M08: HTTP 模块
- M09: 备份管理
- M11: 权限控制
- M12: 日志记录

### P2 - 扩展功能（可以实现）

- M13: 插件系统
- M14: 集群支持
- M15: 文档生成
- M16: 定时任务
- M18: 监控告警

---

## 6. 功能验收标准

### 6.1 接口管理验收

- [ ] 可以创建、编辑、删除接口
- [ ] 接口支持所有 HTTP 方法
- [ ] 接口路径支持参数（如 `/user/:id`）
- [ ] 接口可以组织到文件夹中
- [ ] 接口支持锁定和解锁
- [ ] 接口支持导入和导出

### 6.2 脚本执行验收

- [ ] 脚本可以正确解析和执行
- [ ] 支持变量定义和使用
- [ ] 支持条件判断和循环
- [ ] 支持函数调用
- [ ] 支持异常处理
- [ ] 支持调试（断点、单步执行）

### 6.3 数据库操作验收

- [ ] 支持查询单条和多条记录
- [ ] 支持分页查询
- [ ] 支持增删改操作
- [ ] 支持事务
- [ ] 支持多数据源切换
- [ ] 支持不同数据库类型

### 6.4 用户认证验收

- [ ] 支持用户名密码登录
- [ ] 支持登出
- [ ] 支持会话管理
- [ ] 支持自定义认证

### 6.5 备份管理验收

- [ ] 支持自动备份
- [ ] 支持手动备份
- [ ] 支持备份还原
- [ ] 支持备份清理

---

## 7. 附录

### 7.1 参考资料

- [magic-api 官方文档](https://ssssssss.org/magic-api)
- [magic-script 文档](https://ssssssss.org/magic-script)
- [Spring Boot 文档](https://spring.io/projects/spring-boot)

### 7.2 修订历史

| 版本 | 日期 | 修订内容 | 修订人 |
|------|------|---------|--------|
| 1.0 | 2026-03-14 | 初始版本 | - |
