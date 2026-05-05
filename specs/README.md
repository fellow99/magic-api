# 规格文档索引

**项目名称：** magic-api  
**版本：** 2.2.2  
**技术栈：** Java 1.8+ + Spring Boot 2.x/3.x + magic-script + JSR-223  
**文档生成时间：** 2026-03-14  
**最后更新：** 2026-03-17

---

## 一、文档总览

| 层级 | 分类 | 文档数量 | 说明 |
|------|------|---------|------|
| 整体 | 项目级顶层文档 | 11 | 架构、技术、宪法等全局文档 |
| 整体 | 整体规格文档 | 5 | overall-* 系列文档 |
| 模块 | 核心模块 | 14 | 001~007 共 7 个功能模块 |
| 模块 | Servlet 适配模块 | 2 | 008 Servlet 适配 |
| 模块 | Spring Boot Starter | 2 | 009 Starter 自动配置 |
| 模块 | 编辑器模块 | 2 | 010 Web 编辑器 |
| **合计** | **31 文档 / 10 模块** | | |

---

## 二、项目级顶层文档

全局性的架构、技术、宪法等文档，定义项目基线和开发准则。

| 文档 | 路径 | 说明 |
|------|------|------|
| **项目介绍** | [README.md](./README.md) | 项目介绍和文档索引 |
| **检查清单** | [SPECS_CHECKLIST.md](./SPECS_CHECKLIST.md) | 规格文档完成度追踪，31 份文档的完成状态 |
| **项目结构** | [STRUCTURE.md](./STRUCTURE.md) | 源码目录结构、路由清单、组件清单、API 清单 |
| **API 清单** | [API.md](./API.md) | 全量 API 接口清单、请求参数与响应格式 |
| **技术选型** | [TECH.md](./TECH.md) | 核心技术栈选型理由、版本、依赖说明 |
| **架构设计** | [ARCHITECTURE.md](./ARCHITECTURE.md) | 系统整体架构设计：分层架构、模块划分、数据流、部署架构 |
| **宪法原则** | [constitution.md](./constitution.md) | 项目开发原则、编码规范、治理规则 |

### 整体规格文档

描述跨模块的全局规格、方案和数据模型。

| 文档 | 路径 | 说明 |
|------|------|------|
| **整体规格** | [overall-spec.md](./overall-spec.md) | 系统级功能规格：核心特性、用户故事、非功能需求 |
| **整体方案** | [overall-plan.md](./overall-plan.md) | 系统级技术方案：选型理由、架构决策、关键技术实现 |
| **数据模型** | [overall-data-model.md](./overall-data-model.md) | 全局数据实体定义：核心类型、枚举、实体关系 |
| **接口模型** | [overall-api.md](./overall-api.md) | 全局 API 规范：请求/响应格式、认证机制、错误码 |

---

## 三、核心模块 (001 ~ 007)

### 001 — 核心框架 (Core)

> magic-api 核心框架：注解、配置、上下文、事件、异常、处理器、拦截器等基础设施。

| 文档 | 链接 | 说明 |
|------|------|------|
| 功能规格 | [001-core/spec.md](./001-core/spec.md) | 核心框架功能规格 |
| 技术方案 | [001-core/plan.md](./001-core/plan.md) | 核心框架技术实现方案 |

---

### 002 — 数据源管理 (Datasource)

> 多数据源配置与管理：支持在线配置数据源、动态切换数据源、数据源连接池管理。

| 文档 | 链接 | 说明 |
|------|------|------|
| 功能规格 | [002-datasource/spec.md](./002-datasource/spec.md) | 数据源管理功能规格 |
| 技术方案 | [002-datasource/plan.md](./002-datasource/plan.md) | 数据源管理技术实现方案 |

---

### 003 — 备份恢复 (Backup)

> 历史版本管理：接口脚本备份、版本对比、版本恢复、版本历史记录。

| 文档 | 链接 | 说明 |
|------|------|------|
| 功能规格 | [003-backup/spec.md](./003-backup/spec.md) | 备份恢复功能规格 |
| 技术方案 | [003-backup/plan.md](./003-backup/plan.md) | 备份恢复技术实现方案 |

---

### 004 — 函数管理 (Function)

> 自定义函数管理：支持在线编写自定义函数、函数调用、函数库管理。

| 文档 | 链接 | 说明 |
|------|------|------|
| 功能规格 | [004-function/spec.md](./004-function/spec.md) | 函数管理功能规格 |
| 技术方案 | [004-function/plan.md](./004-function/plan.md) | 函数管理技术实现方案 |

---

### 005 — 脚本引擎 (JSR223)

> JSR-223 脚本引擎支持：基于 magic-script 的动态编译、实时发布、脚本执行。

| 文档 | 链接 | 说明 |
|------|------|------|
| 功能规格 | [005-jsr223/spec.md](./005-jsr223/spec.md) | 脚本引擎功能规格 |
| 技术方案 | [005-jsr223/plan.md](./005-jsr223/plan.md) | 脚本引擎技术实现方案 |

---

### 006 — 模块集合 (Modules)

> 内置模块集合：数据库模块、HTTP 模块、Servlet 模块、Spring 模块，提供丰富的内置功能。

| 文档 | 链接 | 说明 |
|------|------|------|
| 功能规格 | [006-modules/spec.md](./006-modules/spec.md) | 模块集合功能规格 |
| 技术方案 | [006-modules/plan.md](./006-modules/plan.md) | 模块集合技术实现方案 |

---

### 007 — 插件集合 (Plugins)

> 插件扩展集合：集群插件、Redis 插件、MongoDB 插件、ES 插件、Nebula 插件、Swagger 插件、SpringDoc 插件、Git 插件、定时任务插件、组件插件。

| 文档 | 链接 | 说明 |
|------|------|------|
| 功能规格 | [007-plugins/spec.md](./007-plugins/spec.md) | 插件集合功能规格 |
| 技术方案 | [007-plugins/plan.md](./007-plugins/plan.md) | 插件集合技术实现方案 |

---

## 四、Servlet 适配模块 (008)

### 008 — Servlet 适配 (Servlet)

> JavaEE/Jakarta Servlet 规范适配：支持两种 Servlet 规范的接口映射与请求处理。

| 文档 | 链接 | 说明 |
|------|------|------|
| 功能规格 | [008-servlet/spec.md](./008-servlet/spec.md) | Servlet 适配功能规格 |
| 技术方案 | [008-servlet/plan.md](./008-servlet/plan.md) | Servlet 适配技术实现方案 |

---

## 五、Spring Boot Starter 模块 (009)

### 009 — Spring Boot Starter (Starter)

> Spring Boot 自动配置：快速集成 magic-api，开箱即用的 Starter 配置。

| 文档 | 链接 | 说明 |
|------|------|------|
| 功能规格 | [009-starter/spec.md](./009-starter/spec.md) | Spring Boot Starter 功能规格 |
| 技术方案 | [009-starter/plan.md](./009-starter/plan.md) | Spring Boot Starter 技术实现方案 |

---

## 六、编辑器模块 (010)

### 010 — Web 编辑器 (Editor)

> 前端 Web 编辑界面：在线接口编辑、代码提示、调试、可视化操作界面。

| 文档 | 链接 | 说明 |
|------|------|------|
| 功能规格 | [010-editor/spec.md](./010-editor/spec.md) | Web 编辑器功能规格 |
| 技术方案 | [010-editor/plan.md](./010-editor/plan.md) | Web 编辑器技术实现方案 |

---

## 七、模块编号一览

| 编号 | 模块名 | 英文名 | 分类 |
|------|--------|--------|------|
| 001 | 核心框架 | Core | 核心模块 |
| 002 | 数据源管理 | Datasource | 核心模块 |
| 003 | 备份恢复 | Backup | 核心模块 |
| 004 | 函数管理 | Function | 核心模块 |
| 005 | 脚本引擎 | JSR223 | 核心模块 |
| 006 | 模块集合 | Modules | 核心模块 |
| 007 | 插件集合 | Plugins | 核心模块 |
| 008 | Servlet 适配 | Servlet | Servlet 适配模块 |
| 009 | Spring Boot Starter | Starter | Spring Boot Starter |
| 010 | Web 编辑器 | Editor | 编辑器模块 |

---

## 八、模块文档结构规范

每个模块目录 `NNN-name/` 下包含以下 2 份标准文档：

| 文件 | 命名 | 说明 |
|------|------|------|
| 功能规格 | `spec.md` | 定义模块的功能需求、用户故事、验收标准 |
| 技术方案 | `plan.md` | 模块的技术实现方案、架构决策、组件设计 |

> 注：magic-api 模块文档采用精简结构，聚焦于功能规格与技术方案两份核心文档。

---

## 九、快速导航

| 目标读者 | 推荐阅读顺序 |
|---------|-------------|
| **新加入开发者** | constitution.md → STRUCTURE.md → overall-spec.md → 具体模块 spec.md |
| **架构师 / Tech Lead** | ARCHITECTURE.md → TECH.md → overall-plan.md → overall-api.md |
| **Java 后端开发** | STRUCTURE.md → 对应模块的 spec.md + plan.md + overall-api.md |
| **前端开发** | STRUCTURE.md → 010-editor/ 相关文档 + overall-api.md |
| **测试 / QA** | overall-test-cases.md → SPECS_CHECKLIST.md → 各模块 test-cases.md |
| **产品经理** | overall-spec.md → 对应模块 spec.md |

---

## 十、项目统计

| 指标 | 数值 |
|------|------|
| 项目版本 | 2.2.2 |
| JDK 要求 | 1.8+ |
| REST API 端点 | 33 个 |
| 核心模块 | 7 个 (001-007) |
| Servlet 模块 | 1 个 (008-servlet) |
| Starter 模块 | 1 个 (009-starter) |
| 编辑器模块 | 1 个 (010-editor) |
| **总模块数** | **10 个** |
| 规范文档总数 | 31 份 (11 整体 + 20 模块) |
| 已生成文档 | 31 份 |
| 文档完成率 | 100% |

---

## 十一、模块合并说明

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

## 十二、相关链接

- **项目仓库**: [Gitee](https://gitee.com/ssssssss-team/magic-api) | [GitHub](https://github.com/ssssssss-team/magic-api)
- **官方文档**: [https://ssssssss.org](https://ssssssss.org)
- **在线演示**: [https://magic-api.ssssssss.org.cn](https://magic-api.ssssssss.org.cn)
- **示例项目**: [magic-api-example](https://gitee.com/ssssssss-team/magic-api-example)

---

**文档维护者：** magic-api 开发团队  
**文档仓库：** `magic-api/specs/`

---

*最后更新：2026-03-17 (Asia/Shanghai)*
