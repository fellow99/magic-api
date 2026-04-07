# 007-plugins 插件规格说明书

**模块名称**: 插件集合 (Plugins Collection)  
**模块路径**: magic-api-plugins  
**创建时间**: 2026-03-14  
**状态**: 已完成  
**版本**: 2.2.2

---

## 1. 模块概述

### 1.1 模块定位

插件集合提供多种扩展功能插件，包括集群、Redis、MongoDB、Elasticsearch、Nebula 图数据库、Swagger、SpringDoc、Git、定时任务、组件扩展等。

### 1.2 核心价值

- **集群插件**: 集群部署支持，接口变更自动同步
- **Redis 插件**: Redis 数据源支持，缓存操作
- **MongoDB 插件**: MongoDB 文档数据库支持
- **Elasticsearch 插件**: 全文检索和数据分析
- **Nebula 插件**: 图数据库查询和操作
- **Swagger 插件**: Swagger API 文档生成
- **SpringDoc 插件**: OpenAPI 3.0 文档生成
- **Git 插件**: Git 版本控制集成
- **定时任务插件**: Cron 表达式定时任务调度
- **组件插件**: 自定义组件扩展

---

## 2. 功能需求

### 2.1 集群插件 (FR-CLUSTER)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-CLUSTER-001 | 节点注册与发现 | P0 | 集群节点管理 |
| FR-CLUSTER-002 | 接口变更通知 | P0 | 接口变更自动同步 |
| FR-CLUSTER-003 | 配置同步 | P1 | 配置跨节点同步 |
| FR-CLUSTER-004 | WebSocket 通信 | P0 | 节点间通信 |

### 2.2 Redis 插件 (FR-REDIS)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-REDIS-001 | String 操作 | P0 | set/get/del |
| FR-REDIS-002 | Hash 操作 | P1 | hset/hget |
| FR-REDIS-003 | List 操作 | P2 | lpush/rpop |
| FR-REDIS-004 | Set 操作 | P2 | sadd/smembers |
| FR-REDIS-005 | Key 过期管理 | P1 | expire 设置 |

### 2.3 MongoDB 插件 (FR-MONGO)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-MONGO-001 | 文档查询 | P0 | find 操作 |
| FR-MONGO-002 | 文档插入 | P0 | insert 操作 |
| FR-MONGO-003 | 文档更新 | P0 | update 操作 |
| FR-MONGO-004 | 文档删除 | P0 | delete 操作 |
| FR-MONGO-005 | 聚合查询 | P1 | aggregate 操作 |

### 2.4 Elasticsearch 插件 (FR-ES)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-ES-001 | 文档索引 | P0 | index 操作 |
| FR-ES-002 | 全文搜索 | P0 | search 操作 |
| FR-ES-003 | 聚合查询 | P1 | aggregation |
| FR-ES-004 | 索引管理 | P1 | 创建/删除索引 |

### 2.5 Nebula 插件 (FR-NEBULA)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-NEBULA-001 | 顶点操作 | P0 | 插入/查询顶点 |
| FR-NEBULA-002 | 边操作 | P0 | 插入/查询边 |
| FR-NEBULA-003 | 图查询 | P0 | GO 查询 |
| FR-NEBULA-004 | nGQL 执行 | P0 | 原生 nGQL 执行 |

### 2.6 Swagger 插件 (FR-SWAGGER)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-SWAGGER-001 | 接口文档生成 | P0 | 自动生成 Swagger 文档 |
| FR-SWAGGER-002 | Swagger UI 集成 | P0 | Swagger UI 访问 |
| FR-SWAGGER-003 | OpenAPI 规范支持 | P1 | OpenAPI 2.0 规范 |

### 2.7 SpringDoc 插件 (FR-SPRINGDOC)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-SPRINGDOC-001 | OpenAPI 3.0 文档生成 | P0 | OpenAPI 3.0 规范 |
| FR-SPRINGDOC-002 | SpringDoc 集成 | P0 | SpringDoc 集成 |

### 2.8 Git 插件 (FR-GIT)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-GIT-001 | Git 仓库初始化 | P0 | 初始化 Git 仓库 |
| FR-GIT-002 | 代码提交 | P0 | commit 操作 |
| FR-GIT-003 | 版本历史 | P1 | log 查询 |
| FR-GIT-004 | 分支管理 | P1 | 分支操作 |

### 2.9 定时任务插件 (FR-TASK)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-TASK-001 | 任务创建 | P0 | 创建定时任务 |
| FR-TASK-002 | Cron 表达式配置 | P0 | Cron 表达式支持 |
| FR-TASK-003 | 任务执行 | P0 | 定时执行任务 |
| FR-TASK-004 | 任务管理 | P1 | 取消/暂停任务 |

### 2.10 组件插件 (FR-COMPONENT)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-COMPONENT-001 | 组件注册 | P0 | 注册自定义组件 |
| FR-COMPONENT-002 | 组件加载 | P0 | 加载组件 |
| FR-COMPONENT-003 | 组件管理 | P1 | 组件生命周期管理 |

---

## 3. 接口清单

### 3.1 集群插件 API

| 路径 | Method | 说明 |
|------|--------|------|
| /magic/api/cluster/nodes | GET | 获取集群节点列表 |
| /magic/api/cluster/notify | POST | 通知节点变更 |

### 3.2 Redis 插件脚本 API

```javascript
// String 操作
redis.set("key", "value")
redis.get("key")
redis.del("key")

// Hash 操作
redis.hset("hash", "field", "value")
redis.hget("hash", "field")

// Key 过期
redis.expire("key", 3600)
```

### 3.3 MongoDB 插件脚本 API

```javascript
// 查询
mongo.find("collection", {name: "test"})

// 插入
mongo.insert("collection", {name: "test"})

// 更新
mongo.update("collection", {id: 1}, {name: "new"})

// 删除
mongo.delete("collection", {id: 1})
```

### 3.4 Elasticsearch 插件脚本 API

```javascript
// 搜索
es.search("index", {query: {match: {title: "test"}}})

// 索引
es.index("index", {id: 1, title: "test"})

// 删除
es.delete("index", 1)
```

### 3.5 Nebula 插件脚本 API

```javascript
// 查询
nebula.query("GO FROM 1 OVER follow")

// 插入顶点
nebula.insertVertex("person", {id: 1, name: "test"})

// 插入边
nebula.insertEdge("follow", {src: 1, dst: 2})
```

### 3.6 Git 插件脚本 API

```javascript
// 提交
git.commit("update interface")

// 查看历史
git.log()

// 推送
git.push()
```

### 3.7 定时任务插件脚本 API

```javascript
// 创建任务
task.create("myTask", "0 0 * * * ?", () => {
    // 任务逻辑
})

// 删除任务
task.cancel("myTask")
```

---

## 4. 配置项

```properties
# 集群插件配置
magic-api.plugin.cluster.enabled=true
magic-api.plugin.cluster.nodes=node1:port,node2:port

# Redis 插件配置
magic-api.plugin.redis.enabled=true
magic-api.plugin.redis.host=localhost
magic-api.plugin.redis.port=6379

# MongoDB 插件配置
magic-api.plugin.mongo.enabled=true
magic-api.plugin.mongo.uri=mongodb://localhost:27017/db

# Elasticsearch 插件配置
magic-api.plugin.es.enabled=true
magic-api.plugin.es.hosts=localhost:9200

# Nebula 插件配置
magic-api.plugin.nebula.enabled=true
magic-api.plugin.nebula.hosts=localhost:9669
magic-api.plugin.nebula.space=test

# Swagger 插件配置
magic-api.plugin.swagger.enabled=true
magic-api.plugin.swagger.title=magic-api 文档
magic-api.plugin.swagger.version=1.0.0

# SpringDoc 插件配置
magic-api.plugin.springdoc.enabled=true
magic-api.plugin.springdoc.api-docs-path=/v3/api-docs

# Git 插件配置
magic-api.plugin.git.enabled=true
magic-api.plugin.git.repository=/data/magic-api/git

# 定时任务插件配置
magic-api.plugin.task.enabled=true

# 组件插件配置
magic-api.plugin.component.enabled=true
magic-api.plugin.component.scan-packages=
```

---

## 5. 验收标准

### 5.1 集群插件
- [ ] 支持集群节点管理
- [ ] 支持接口变更同步
- [ ] 支持配置同步

### 5.2 Redis 插件
- [ ] 支持 Redis 基本操作
- [ ] 支持 Key 过期管理

### 5.3 MongoDB 插件
- [ ] 支持 MongoDB 基本操作
- [ ] 支持聚合查询

### 5.4 Elasticsearch 插件
- [ ] 支持 ES 基本操作
- [ ] 支持全文搜索

### 5.5 Nebula 插件
- [ ] 支持 Nebula 基本操作
- [ ] 支持 nGQL 查询

### 5.6 Swagger 插件
- [ ] 支持 Swagger 文档生成
- [ ] 支持 Swagger UI 访问

### 5.7 SpringDoc 插件
- [ ] 支持 OpenAPI 3.0 文档生成

### 5.8 Git 插件
- [ ] 支持 Git 版本管理
- [ ] 支持代码提交

### 5.9 定时任务插件
- [ ] 支持定时任务创建
- [ ] 支持 Cron 表达式
- [ ] 支持任务管理

### 5.10 组件插件
- [ ] 支持组件扩展

---

*文档生成时间：2026-03-14*
