# magic-api 技术栈文档

本文档记录了 magic-api 项目的技术栈、依赖关系和构建配置。

---

## 1. 项目基本信息

| 属性 | 值 |
|------|-----|
| 项目名称 | magic-api-parent |
| 组织 | org.ssssssss |
| 版本 | 2.2.2 |
| JDK 版本 | 1.8 |
| 构建工具 | Maven |
| 项目类型 | Spring Boot 多模块项目 |
| 许可证 | MIT License |

---

## 2. 核心框架

### 2.1 Spring Boot

| 依赖 | 版本 | 说明 |
|------|------|------|
| spring-boot-dependencies | 2.4.5 | Spring Boot 父依赖（BOM） |
| spring-boot-starter-web | 2.4.5 | Web 应用核心依赖 |
| spring-boot-starter-websocket | 2.4.5 | WebSocket 支持 |
| spring-boot-starter-jdbc | 2.4.5 | JDBC 支持 |
| spring-boot-configuration-processor | 2.4.5 | 配置元数据处理器 |

### 2.2 脚本引擎

| 依赖 | 版本 | 说明 |
|------|------|------|
| magic-script | 1.9.0 | 自研脚本引擎（核心依赖） |

---

## 3. 模块结构

```
magic-api-parent
├── magic-api                          # 核心模块
├── magic-editor                       # 编辑器前端资源
├── magic-api-servlet                  # Servlet 适配层
│   ├── magic-api-servlet-jakarta     # Jakarta Servlet 适配
│   └── magic-api-servlet-javaee      # JavaEE Servlet 适配
├── magic-api-spring-boot-starter      # Spring Boot 自动配置
└── magic-api-plugins                  # 插件模块
    ├── magic-api-plugin-task         # 任务调度插件
    ├── magic-api-plugin-component    # 组件插件
    ├── magic-api-plugin-swagger      # Swagger 文档插件
    ├── magic-api-plugin-springdoc    # SpringDoc 文档插件
    ├── magic-api-plugin-redis        # Redis 插件
    ├── magic-api-plugin-mongo        # MongoDB 插件
    ├── magic-api-plugin-elasticsearch # Elasticsearch 插件
    ├── magic-api-plugin-cluster      # 集群插件
    ├── magic-api-plugin-git          # Git 插件
    └── magic-api-plugin-nebula       # Nebula Graph 插件
```

---

## 4. 依赖关系图

```
                                    ┌─────────────────────────────────┐
                                    │   magic-api-spring-boot-starter │
                                    │   (自动配置 + 启动器)            │
                                    └───────────────┬─────────────────┘
                                                    │
                    ┌───────────────────────────────┼───────────────────────────────┐
                    │                               │                               │
                    ▼                               ▼                               ▼
        ┌───────────────────┐           ┌───────────────────┐           ┌───────────────────┐
        │   magic-api       │           │   magic-editor    │           │ magic-api-servlet │
        │   (核心模块)       │◄──────────│   (编辑器资源)     │           │   (Servlet 适配)   │
        └─────────┬─────────┘           └───────────────────┘           └───────────────────┘
                  │
                  │ 依赖
                  ▼
        ┌───────────────────┐
        │   magic-script    │
        │   (脚本引擎)       │
        └───────────────────┘

        ┌───────────────────────────────────────────────────────────────────────────┐
        │                         magic-api-plugins (插件模块)                       │
        │  task | component | swagger | springdoc | redis | mongo | es | cluster... │
        └───────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ 依赖
                                    ▼
                          ┌───────────────────┐
                          │    magic-api      │
                          └───────────────────┘
```

---

## 5. 核心依赖清单

### 5.1 核心依赖（必选）

| 依赖 | 版本 | 作用域 | 说明 |
|------|------|--------|------|
| org.ssssssss:magic-api | 2.2.2 | compile | 核心模块 |
| org.ssssssss:magic-script | 1.9.0 | compile | 脚本引擎 |
| org.springframework.boot:spring-boot-starter-web | 2.4.5 | compile | Web 支持 |
| org.springframework.boot:spring-boot-starter-websocket | 2.4.5 | compile | WebSocket 支持 |
| org.springframework:spring-jdbc | 2.4.5 | compile | JDBC 支持 |

### 5.2 工具库依赖

| 依赖 | 版本 | 作用域 | 说明 |
|------|------|--------|------|
| org.apache.commons:commons-text | 1.10.0 | compile | 文本处理工具 |
| commons-beanutils:commons-beanutils | 1.9.4 | compile | Bean 工具 |
| commons-io:commons-io | 2.7 | compile | IO 工具 |
| org.apache.commons:commons-compress | 1.21 | compile | 压缩工具 |
| com.alibaba:fastjson | 1.2.83 | provided | JSON 处理（可选） |

### 5.3 可选依赖（provided）

| 依赖 | 版本 | 说明 |
|------|------|------|
| spring-boot-starter-log4j | 1.3.8.RELEASE | Log4j 日志（可选） |
| spring-boot-starter-log4j2 | - | Log4j2 日志（可选） |
| spring-boot-starter-jdbc | 2.4.5 | JDBC（按需引入） |
| fastjson | 1.2.83 | JSON 库（按需引入） |

### 5.4 Jakarta/JavaEE 适配

| 依赖 | 版本 | 说明 |
|------|------|------|
| jakarta.servlet:jakarta.servlet-api | 6.0.0 | Jakarta Servlet 6.0 适配 |
| magic-api-servlet-javaee | 2.2.2 | JavaEE Servlet 适配 |
| magic-api-servlet-jakarta | 2.2.2 | Jakarta Servlet 适配 |

---

## 6. 构建配置

### 6.1 Maven 配置

```xml
<!-- 编译配置 -->
<maven.compiler.source>1.8</maven.compiler.source>
<maven.compiler.target>1.8</maven.compiler.target>
<maven.compiler.parameters>true</maven.compiler.parameters>
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
```

### 6.2 发布配置（Release Profile）

发布到 Maven Central 需要以下插件：

| 插件 | 版本 | 作用 |
|------|------|------|
| maven-source-plugin | 2.2.1 | 生成源码包 |
| maven-javadoc-plugin | 3.0.0 | 生成 Javadoc |
| maven-gpg-plugin | 1.5 | GPG 签名 |
| central-publishing-maven-plugin | 0.7.0 | 发布到 Maven Central |

激活方式：
```bash
mvn clean deploy -Prelease
```

---

## 7. 技术栈总结

### 7.1 后端技术栈

| 分类 | 技术 |
|------|------|
| 语言 | Java 8 |
| 框架 | Spring Boot 2.4.5 |
| Web | Spring MVC + WebSocket |
| 脚本引擎 | magic-script 1.9.0（自研） |
| 数据库访问 | Spring JDBC |
| JSON | FastJSON（可选） |
| 工具库 | Apache Commons 系列 |

### 7.2 前端技术栈（magic-editor）

| 分类 | 技术 |
|------|------|
| 编辑器 | Monaco Editor（推测） |
| 构建 | 独立模块，通过 Maven 打包 |

### 7.3 可选扩展

| 分类 | 插件 |
|------|------|
| 文档 | Swagger、SpringDoc |
| 缓存 | Redis |
| 数据库 | MongoDB、Elasticsearch、Nebula Graph |
| 部署 | Git、Cluster |
| 调度 | Task |

---

## 8. 依赖版本管理

项目使用 `dependencyManagement` 统一管理版本：

```xml
<dependencyManagement>
    <dependencies>
        <!-- Spring Boot BOM -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.4.5</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        
        <!-- 内部模块版本 -->
        <dependency>
            <groupId>org.ssssssss</groupId>
            <artifactId>magic-api</artifactId>
            <version>2.2.2</version>
        </dependency>
        
        <!-- 工具库版本 -->
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.7</version>
        </dependency>
        <!-- ... 其他依赖 ... -->
    </dependencies>
</dependencyManagement>
```

---

## 9. 构建命令

```bash
# 编译整个项目
mvn clean compile

# 打包（跳过测试）
mvn clean package -DskipTests

# 安装到本地仓库
mvn clean install

# 发布到 Maven Central（需要 GPG 密钥）
mvn clean deploy -Prelease
```

---

## 10. 系统要求

| 要求 | 版本 |
|------|------|
| JDK | 1.8+ |
| Maven | 3.6+ |
| Spring Boot | 2.4.5+ |
| Node.js | （仅前端编辑器的可选构建） |

---

*文档生成时间：2026-03-14*
