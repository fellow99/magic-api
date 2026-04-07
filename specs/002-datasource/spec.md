# 002-datasource 数据源管理模块规格说明书

**模块名称**: 数据源管理 (Datasource Management)  
**模块路径**: magic-api/magic-api/src/main/java/org/ssssssss/magicapi/datasource  
**创建时间**: 2026-03-14  
**状态**: 已完成  
**版本**: 2.2.2

---

## 1. 模块概述

### 1.1 模块定位

数据源管理模块提供多数据源配置和管理功能，支持运行时动态切换数据源，实现在同一应用中访问多个数据库。

### 1.2 核心价值

- **多数据源支持**: 同时配置和管理多个数据库连接
- **动态切换**: 运行时动态修改和切换数据源
- **在线配置**: 通过 Web 界面在线配置数据源
- **安全存储**: 支持数据库密码加密存储

### 1.3 模块范围

**包含**:
- 数据源配置模型
- 动态数据源管理
- 数据源加密服务
- 数据源资源存储
- 数据源管理接口

**不包含**:
- 具体数据库操作 (由 modules-db 负责)
- 数据库连接池管理 (由底层驱动负责)

---

## 2. 功能需求

### 2.1 数据源配置 (FR-001)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-001-1 | DataSourceInfo | P0 | 数据源配置信息模型 |
| FR-001-2 | 多数据源配置 | P0 | 支持配置多个数据源 |
| FR-001-3 | 默认数据源 | P0 | 指定默认数据源 |

**数据源属性**:
- id: 数据源唯一标识
- name: 数据源名称
- driverClassName: 驱动类名
- url: JDBC URL
- username: 用户名
- password: 密码 (加密存储)
- pool: 连接池配置

---

### 2.2 动态数据源 (FR-002)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-002-1 | MagicDynamicDataSource | P0 | 动态数据源管理类 |
| FR-002-2 | 数据源注册 | P0 | 动态注册新数据源 |
| FR-002-3 | 数据源移除 | P1 | 动态移除数据源 |
| FR-002-4 | 数据源切换 | P0 | 运行时切换数据源 |

---

### 2.3 数据源加密 (FR-003)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-003-1 | DataSourceEncryptProvider | P1 | 数据源加密提供者接口 |
| FR-003-2 | 密码加密 | P1 | 数据库密码加密存储 |
| FR-003-3 | 密码解密 | P1 | 使用时解密密码 |

---

### 2.4 数据源管理接口 (FR-004)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-004-1 | MagicDataSourceController | P0 | 数据源管理 REST API |
| FR-004-2 | 数据源列表 | P0 | 获取所有数据源配置 |
| FR-004-3 | 数据源详情 | P0 | 获取单个数据源配置 |
| FR-004-4 | 数据源保存 | P0 | 保存/更新数据源配置 |
| FR-004-5 | 数据源删除 | P1 | 删除数据源配置 |
| FR-004-6 | 数据源测试 | P1 | 测试数据源连接 |

---

## 3. 数据模型

### 3.1 DataSourceInfo

```java
public class DataSourceInfo {
    private String id;              // 数据源 ID
    private String name;            // 数据源名称
    private String driverClassName; // 驱动类名
    private String url;             // JDBC URL
    private String username;        // 用户名
    private String password;        // 密码 (加密)
    private Map<String, Object> properties; // 其他属性
    private Long createTime;        // 创建时间
    private Long updateTime;        // 更新时间
}
```

### 3.2 MagicDynamicDataSource

```java
public class MagicDynamicDataSource extends AbstractRoutingDataSource {
    private Map<Object, Object> targetDataSources;  // 目标数据源 Map
    private Object defaultTargetDataSource;         // 默认数据源
    
    public void addDataSource(String key, DataSource dataSource);
    public void removeDataSource(String key);
    public void setDefaultDataSource(DataSource dataSource);
}
```

---

## 4. 接口清单

### 4.1 REST API

| 路径 | Method | 说明 |
|------|--------|------|
| /magic/api/datasource/list | GET | 获取数据源列表 |
| /magic/api/datasource/info/{id} | GET | 获取数据源详情 |
| /magic/api/datasource/save | POST | 保存数据源 |
| /magic/api/datasource/delete/{id} | POST | 删除数据源 |
| /magic/api/datasource/test | POST | 测试数据源连接 |

---

## 5. 非功能需求

### 5.1 性能要求

- 数据源切换延迟 < 10ms
- 支持 50+ 个数据源配置

### 5.2 安全性要求

- 数据库密码必须加密存储
- 支持自定义加密算法

### 5.3 可用性要求

- 支持运行时动态添加/删除数据源
- 数据源配置持久化存储

---

## 6. 模块依赖

| 依赖模块 | 说明 |
|----------|------|
| 001-core | 核心框架 |
| magic-api-plugins/redis | Redis 数据源 (可选) |
| magic-api-plugins/mongo | MongoDB 数据源 (可选) |

---

## 7. 配置项

```properties
# 默认数据源
spring.datasource.url=jdbc:mysql://localhost:3306/db
spring.datasource.username=root
spring.datasource.password=encrypted_password

# 数据源加密密钥
magic-api.datasource.encrypt.key=your-encrypt-key
```

---

## 8. 验收标准

- [ ] 支持配置多个数据源
- [ ] 支持运行时动态切换数据源
- [ ] 支持数据库密码加密存储
- [ ] 提供完整的数据源管理 API

---

*文档生成时间：2026-03-14*
