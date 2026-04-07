# 003-backup 备份恢复模块规格说明书

**模块名称**: 备份恢复 (Backup & Recovery)  
**模块路径**: magic-api/magic-api/src/main/java/org/ssssssss/magicapi/backup  
**创建时间**: 2026-03-14  
**状态**: 已完成  
**版本**: 2.2.2

---

## 1. 模块概述

### 1.1 模块定位

备份恢复模块提供接口脚本的历史版本管理、备份和恢复功能，支持版本对比和历史版本恢复。

### 1.2 核心价值

- **版本管理**: 自动保存接口脚本的历史版本
- **版本对比**: 支持不同版本之间的差异对比
- **版本恢复**: 支持恢复到任意历史版本
- **备份导出**: 支持接口备份和导出

---

## 2. 功能需求

### 2.1 历史版本管理 (FR-001)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-001-1 | 自动保存版本 | P0 | 接口修改时自动保存历史版本 |
| FR-001-2 | 版本列表 | P0 | 获取接口的所有历史版本 |
| FR-001-3 | 版本详情 | P0 | 查看特定版本的内容 |
| FR-001-4 | 版本删除 | P2 | 删除指定的历史版本 |

### 2.2 版本对比 (FR-002)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-002-1 | 版本差异对比 | P0 | 对比两个版本的差异 |
| FR-002-2 | 可视化对比 | P1 | Web 界面展示差异 |

### 2.3 版本恢复 (FR-003)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-003-1 | 恢复历史版本 | P0 | 将接口恢复到指定版本 |
| FR-003-2 | 恢复确认 | P1 | 恢复前确认提示 |

### 2.4 备份导出 (FR-004)

| ID | 功能点 | 优先级 | 说明 |
|----|--------|--------|------|
| FR-004-1 | 全量备份 | P1 | 备份所有接口 |
| FR-004-2 | 单接口备份 | P1 | 备份单个接口 |
| FR-004-3 | 导出为文件 | P1 | 导出为 JSON/ZIP 文件 |
| FR-004-4 | 导入恢复 | P1 | 从备份文件恢复 |

---

## 3. 数据模型

### 3.1 BackupInfo

```java
public class BackupInfo {
    private String id;              // 备份 ID
    private String apiId;           // 接口 ID
    private String content;         // 脚本内容
    private String version;         // 版本号
    private Long backupTime;        // 备份时间
    private String operator;        // 操作人
    private String comment;         // 备注
}
```

### 3.2 BackupConfig

```java
public class BackupConfig {
    private boolean enabled = true;     // 是否启用备份
    private int keepVersions = 10;      // 保留版本数
    private String storageType = "file"; // 存储类型
    private String storagePath;         // 存储路径
}
```

---

## 4. 接口清单

| 路径 | Method | 说明 |
|------|--------|------|
| /magic/api/backup/list | GET | 获取备份列表 |
| /magic/api/backup/detail/{id} | GET | 获取备份详情 |
| /magic/api/backup/restore/{id} | POST | 恢复备份 |
| /magic/api/backup/delete/{id} | POST | 删除备份 |
| /magic/api/backup/export | POST | 导出备份 |
| /magic/api/backup/import | POST | 导入备份 |
| /magic/api/backup/compare | POST | 对比版本 |

---

## 5. 非功能需求

### 5.1 性能要求

- 版本保存延迟 < 50ms
- 支持 100+ 版本/接口

### 5.2 存储要求

- 默认保留 10 个历史版本
- 支持自定义保留数量

### 5.3 可靠性要求

- 备份数据持久化存储
- 支持备份数据完整性校验

---

## 6. 配置项

```properties
# 启用备份
magic-api.backup.enabled=true

# 保留版本数
magic-api.backup.keep-versions=10

# 存储类型 (file/database)
magic-api.backup.storage-type=file

# 存储路径
magic-api.backup.storage-path=/data/magic-api/backup
```

---

## 7. 验收标准

- [ ] 自动保存接口历史版本
- [ ] 支持版本列表查看
- [ ] 支持版本差异对比
- [ ] 支持恢复到历史版本
- [ ] 支持备份导出和导入

---

*文档生成时间：2026-03-14*
