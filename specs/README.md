# magic-api 规范文档

> 项目版本：2.2.2  
> 文档生成时间：2026-03-14  
> 规范目录：specs

---

## 项目介绍

**magic-api** 是一个基于 Java 的接口快速开发框架，通过 UI 界面编写接口，自动映射为 HTTP 接口，无需定义 Controller、Service、Dao、Mapper、XML、VO 等 Java 对象即可完成常见的 HTTP API 接口开发。

### 核心特性

- 🚀 **快速开发** - 无需编写 Java 代码，在线编辑接口脚本
- 💾 **多数据源支持** - MySQL、MariaDB、Oracle、DB2、PostgreSQL、SQLServer、Redis、MongoDB 等
- 🔌 **插件化架构** - 支持集群、Swagger、Git、定时任务等插件扩展
- 📝 **实时发布** - 动态编译，无需重启，实时生效
- 🔒 **权限控制** - 支持接口权限配置、拦截器
- 📊 **在线调试** - 支持接口在线调试、历史版本对比与恢复
- 💡 **智能提示** - 代码自动提示、参数提示、错误提示

### 技术栈

- **JDK**: 1.8+
- **脚本引擎**: magic-script (基于 JSR-223)
- **Web 框架**: Spring Boot 2.x/3.x
- **前端**: 自研 Web 编辑器
- **数据库**: 支持所有 JDBC 规范数据库 + Redis + MongoDB

---

## 文档介绍

本规范文档目录 (specs) 包含 magic-api 项目的完整技术规格说明，用于：

1. **理解项目架构** - 了解整体设计和技术选型
2. **开发参考** - 为功能扩展和维护提供依据
3. **质量保证** - 确保实现符合规格要求
4. **知识传承** - 帮助新成员快速上手

### 文档结构

```
specs/
├── README.md                    # 本文档 - 项目介绍和文档索引
├── SPECS_CHECKLIST.md           # 规范文档完成情况检查清单
├── STRUCTURE.md                 # 工程目录结构
├── API.md                       # REST API 清单
├── TECH.md                      # 技术选型文档
├── ARCHITECTURE.md              # 整体架构设计
├── constitution.md              # 项目宪法原则
├── overall-spec.md              # 整体规格说明
├── overall-plan.md              # 整体技术方案
├── overall-data-model.md        # 数据模型设计
├── overall-api.md               # 对外接口模型
└── <模块目录>/                   # 各功能模块详细文档
    ├── spec.md                  # 模块规格
    └── plan.md                  # 实现方案
```

---

## 文档文件索引

### 📋 整体文档

| 文件名 | 说明 | 状态 | 大小 |
|--------|------|------|------|
| [README.md](./README.md) | 项目介绍和文档索引 | ✅ | - |
| [SPECS_CHECKLIST.md](./SPECS_CHECKLIST.md) | 规范文档检查清单 | ✅ | 3.5 KB |
| [STRUCTURE.md](./STRUCTURE.md) | 工程目录结构 | ✅ | 13.4 KB |
| [API.md](./API.md) | REST API 清单 (33 个端点) | ✅ | 5.2 KB |
| [TECH.md](./TECH.md) | 技术选型 | ✅ | 9.4 KB |
| [ARCHITECTURE.md](./ARCHITECTURE.md) | 整体架构设计 | ✅ | 24 KB |
| [constitution.md](./constitution.md) | 项目宪法原则 | ✅ | 13 KB |
| [overall-spec.md](./overall-spec.md) | 整体规格说明 | ✅ | 21 KB |
| [overall-plan.md](./overall-plan.md) | 整体技术方案 | ✅ | 30 KB |
| [overall-data-model.md](./overall-data-model.md) | 数据模型设计 | ✅ | 24 KB |
| [overall-api.md](./overall-api.md) | 对外接口模型 | ✅ | 18 KB |

**整体文档进度**: 11/11 完成 (100%)

---

### 🔧 功能模块文档

#### 核心模块 (magic-api)

| 编码 | 模块 | 说明 | 状态 |
|------|------|------|------|
| 001-core | 核心框架 | 注解、配置、上下文、事件、异常、处理器、拦截器 | ✅ |
| 002-datasource | 数据源管理 | 多数据源配置与管理 | ✅ |
| 003-backup | 备份恢复 | 历史版本、备份管理 | ✅ |
| 004-function | 函数管理 | 自定义函数管理 | ✅ |
| 005-jsr223 | 脚本引擎 | JSR-223 脚本支持 | ✅ |
| 006-modules | 模块集合 | 数据库/HTTP/Servlet/Spring 模块合并 | ✅ |
| 007-plugins | 插件集合 | 10 个插件合并 (集群/Redis/MongoDB/ES/Nebula/Swagger/SpringDoc/Git/Task/Component) | ✅ |

#### Servlet 适配模块 (magic-api-servlet)

| 编码 | 模块 | 说明 | 状态 |
|------|------|------|------|
| 008-servlet | Servlet 适配 | JavaEE/Jakarta Servlet 规范适配 | ✅ |

#### Spring Boot Starter (magic-api-spring-boot-starter)

| 编码 | 模块 | 说明 | 状态 |
|------|------|------|------|
| 009-starter | Spring Boot Starter | 自动配置、快速集成 | ✅ |

#### 编辑器模块 (magic-editor)

| 编码 | 模块 | 说明 | 状态 |
|------|------|------|------|
| 010-editor | Web 编辑器 | 前端 Web 编辑界面 | ✅ |

**功能模块文档进度**: 10/10 完成 (100%)

---

## 快速导航

### 📖 按用途查找

- **新成员入门** → 先阅读 [README.md](./README.md) 和 [constitution.md](./constitution.md)
- **了解架构** → 阅读 [ARCHITECTURE.md](./ARCHITECTURE.md) 和 [STRUCTURE.md](./STRUCTURE.md)
- **API 参考** → 查看 [API.md](./API.md) 和 [overall-api.md](./overall-api.md)
- **技术细节** → 阅读 [TECH.md](./TECH.md) 和 [overall-data-model.md](./overall-data-model.md)
- **实现方案** → 查看 [overall-plan.md](./overall-plan.md) 和各模块 plan.md

### 🔍 按模块查找

- **核心功能** → `001-core/` 至 `007-plugins/`
- **部署集成** → `008-servlet/`、`009-starter/`
- **前端界面** → `010-editor/`

---

## 项目统计

| 指标 | 数值 |
|------|------|
| 项目版本 | 2.2.2 |
| JDK 要求 | 1.8+ |
| REST API 端点 | 33 个 |
| 核心模块 | 5 个 (001-005) |
| 模块集合 | 1 个 (006-modules，含 4 个子模块) |
| 插件集合 | 1 个 (007-plugins，含 10 个子插件) |
| Servlet 模块 | 1 个 (008-servlet) |
| Starter 模块 | 1 个 (009-starter) |
| 编辑器模块 | 1 个 (010-editor) |
| **总模块数** | **10 个** |
| 规范文档总数 | 31 份 (11 整体 + 20 模块) |
| 已生成文档 | 31 份 |
| 文档完成率 | 100% |

---

## 模块合并说明

为优化文档结构，原有 23 个模块已合并为 10 个模块：

| 新模块 | 原模块 | 说明 |
|--------|--------|------|
| 001-core ~ 005-jsr223 | 保持不变 | 5 个核心模块 |
| 006-modules | 006-modules-db, 007-modules-http, 008-modules-servlet, 009-modules-spring | 合并为模块集合 |
| 007-plugins | 010-plugin-cluster ~ 019-plugin-component (10 个) | 合并为插件集合 |
| 008-servlet | 020-servlet-javaee, 021-servlet-jakarta | 合并为 Servlet 适配 |
| 009-starter | 022-spring-boot-starter | 重命名 |
| 010-editor | 023-editor | 重命名 |

---

## 相关链接

- **项目仓库**: [Gitee](https://gitee.com/ssssssss-team/magic-api) | [GitHub](https://github.com/ssssssss-team/magic-api)
- **官方文档**: [https://ssssssss.org](https://ssssssss.org)
- **在线演示**: [https://magic-api.ssssssss.org.cn](https://magic-api.ssssssss.org.cn)
- **示例项目**: [magic-api-example](https://gitee.com/ssssssss-team/magic-api-example)

---

## 更新日志

| 日期 | 更新内容 | 更新人 |
|------|----------|--------|
| 2026-03-14 15:30 | 目录重命名：specs-plan-a → specs | 自动生成 |
| 2026-03-14 15:21 | 模块合并优化：23 个模块合并为 10 个 | 自动生成 |
| 2026-03-14 14:50 | 阶段 4 完成：23 个功能模块规范文档全部生成 | 自动生成 |
| 2026-03-14 14:35 | 整体技术方案文档完成 (overall-plan/data-model/api) | 自动生成 |
| 2026-03-14 14:31 | 更新文档状态（overall-spec.md 已完成） | 自动生成 |
| 2026-03-14 14:30 | 更新文档状态（constitution.md 已完成） | 自动生成 |
| 2026-03-14 14:29 | 更新文档状态（ARCHITECTURE.md 已完成） | 自动生成 |
| 2026-03-14 14:28 | 更新文档状态（TECH.md 已完成） | 自动生成 |
| 2026-03-14 14:27 | 初始版本，创建 README.md 和 SPECS_CHECKLIST.md | 自动生成 |
| 2026-03-14 14:26 | STRUCTURE.md 生成完成 | 自动生成 |
| 2026-03-14 14:27 | API.md 生成完成 | 自动生成 |
| 2026-03-14 14:27 | TECH.md 生成完成 | 自动生成 |

---

*最后更新：2026-03-14 15:21 (Asia/Shanghai)*
