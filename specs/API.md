# magic-api REST API 清单

本文档记录了 magic-api 项目的所有 REST API 端点，按功能模块分组。

## API 概览

| 模块 | 端点数量 | 说明 |
|------|---------|------|
| 核心工作台 (Workbench) | 16 | 编辑器配置、登录认证、类扫描、搜索等 |
| 资源管理 (Resource) | 9 | API 文件/文件夹的增删改查、移动、锁定 |
| 备份管理 (Backup) | 6 | 备份列表、还原、全量备份 |
| 数据源管理 (DataSource) | 1 | 数据源连接测试 |
| 任务管理 (Task) | 1 | 任务执行 |
| **总计** | **33** | |

---

## 1. 核心工作台 API (MagicWorkbenchController)

前端编辑器的核心功能，包括配置、认证、类扫描等。

| 路径 | Method | 描述 | 入口文件 |
|------|--------|------|---------|
| `/` | GET | 重定向到编辑器首页 | MagicWorkbenchController.java |
| `/index.html` | GET | 重定向到编辑器首页 | MagicWorkbenchController.java |
| `/config.json` | GET | 读取编辑器配置信息 | MagicWorkbenchController.java |
| `/classes.txt` | GET | 获取压缩后的所有类信息（文本） | MagicWorkbenchController.java |
| `/classes` | POST | 获取所有类、扩展、函数信息 | MagicWorkbenchController.java |
| `/class` | POST | 获取单个类的详细信息 | MagicWorkbenchController.java |
| `/login` | POST | 用户登录 | MagicWorkbenchController.java |
| `/user` | POST | 获取当前用户信息 | MagicWorkbenchController.java |
| `/logout` | POST | 用户登出 | MagicWorkbenchController.java |
| `/plugins` | GET | 获取插件列表 | MagicWorkbenchController.java |
| `/options` | GET/POST | 获取配置选项列表 | MagicWorkbenchController.java |
| `/reload` | GET | 重新加载资源 | MagicWorkbenchController.java |
| `/search` | POST | 搜索 API 脚本内容 | MagicWorkbenchController.java |
| `/todo` | GET | 获取所有 TODO/FIXME 注释 | MagicWorkbenchController.java |
| `/config-js` | GET | 获取编辑器 JavaScript 配置 | MagicWorkbenchController.java |
| `/download` | GET/POST | 下载 API 资源（ZIP） | MagicWorkbenchController.java |
| `/upload` | GET/POST | 上传 API 资源 | MagicWorkbenchController.java |
| `/push` | GET/POST | 推送 API 资源到目标环境 | MagicWorkbenchController.java |
| `/receivePush` | POST | 接收推送的 API 资源 | MagicWorkbenchController.java |

---

## 2. 资源管理 API (MagicResourceController)

管理 API 文件、文件夹的增删改查、移动、锁定等操作。

| 路径 | Method | 描述 | 入口文件 |
|------|--------|------|---------|
| `/resource/folder/save` | POST | 保存文件夹 | MagicResourceController.java |
| `/resource/folder/copy` | POST | 复制文件夹 | MagicResourceController.java |
| `/resource/delete` | POST | 删除文件或文件夹 | MagicResourceController.java |
| `/resource/file/{folder}/save` | POST | 保存文件到指定文件夹 | MagicResourceController.java |
| `/resource/file/{id}` | GET | 获取文件详情 | MagicResourceController.java |
| `/resource/move` | POST | 移动文件或文件夹 | MagicResourceController.java |
| `/resource/lock` | POST | 锁定文件 | MagicResourceController.java |
| `/resource/unlock` | POST | 解锁文件 | MagicResourceController.java |
| `/resource` | POST | 获取资源树结构 | MagicResourceController.java |

---

## 3. 备份管理 API (MagicBackupController)

管理 API 资源的备份和还原功能。

| 路径 | Method | 描述 | 入口文件 |
|------|--------|------|---------|
| `/backups` | GET | 获取备份列表 | MagicBackupController.java |
| `/backup/{id}` | GET | 获取指定备份的详情 | MagicBackupController.java |
| `/backup/rollback` | POST | 还原备份 | MagicBackupController.java |
| `/backup` | GET | 获取备份的脚本内容 | MagicBackupController.java |
| `/backup/full` | POST | 执行全量备份 | MagicBackupController.java |

---

## 4. 数据源管理 API (MagicDataSourceController)

管理数据库连接配置。

| 路径 | Method | 描述 | 入口文件 |
|------|--------|------|---------|
| `/datasource/jdbc/test` | GET/POST | 测试 JDBC 数据源连接 | MagicDataSourceController.java |

---

## 5. 任务管理 API (MagicTaskController)

管理定时任务的执行。

| 路径 | Method | 描述 | 入口文件 |
|------|--------|------|---------|
| `/task/execute` | POST | 手动执行任务 | MagicTaskController.java |

---

## API 路径前缀说明

所有 API 端点默认注册在 `magic-api` 上下文路径下，完整访问路径为：

```
{server.context-path}/magic-api{api-path}
```

例如：
- 登录：`POST /magic-api/login`
- 获取资源树：`POST /magic-api/resource`
- 测试数据源：`GET /magic-api/datasource/jdbc/test`

---

## 认证说明

- 大部分 API 需要登录后访问
- 登录成功后，前端通过 `MAGIC_TOKEN_HEADER` 传递认证 Token
- 部分公开 API（如 `/config.json`、`/classes.txt`）无需登录

---

## 响应格式

所有 API 统一返回 `JsonBean<T>` 格式：

```json
{
  "code": 200,
  "message": "success",
  "data": {}
}
```

错误响应：
```json
{
  "code": 500,
  "message": "错误信息",
  "data": null
}
```
