# magic-api 项目宪法

> 版本：2.2.2  
> 生成时间：2026-03-14  
> 文档类型：项目宪法

---

## 1. 项目宪法原则

### 1.1 核心使命

**让 API 开发更简单** —— 通过可视化界面和动态脚本引擎，消除繁琐的样板代码，让开发者专注于业务逻辑本身。

### 1.2 设计哲学

#### 原则一：约定优于配置
- 默认配置满足 80% 的使用场景
- 必要时才进行自定义配置
- 配置项命名清晰、直观

#### 原则二：零样板代码
- 不强制要求定义 Controller、Service、Dao
- 不强制要求创建 Mapper、XML、VO
- 接口即代码，代码即接口

#### 原则三：即时反馈
- 接口修改后立即可见
- 错误信息实时提示
- 调试过程可视化

#### 原则四：渐进式复杂度
- 简单场景：写 SQL 即可
- 中等场景：使用脚本逻辑
- 复杂场景：调用 Java 函数或自定义模块

#### 原则五：安全优先
- 默认开启登录认证
- 敏感操作需要确认
- 支持接口锁定
- 备份还原机制

---

## 2. 开发规范和约束

### 2.1 代码规范

#### 2.1.1 Java 代码规范

**命名规范：**
```java
// 类名：大驼峰
public class MagicAPIService { }

// 方法名：小驼峰
public <T> T execute(String method, String path, Map<String, Object> context) { }

// 常量：全大写，下划线分隔
public static final String HEADER_REQUEST_CLIENT_ID = "X-Client-Id";

// 变量名：小驼峰，语义清晰
String scriptName = "user/getUserById";
Map<String, Object> context = new HashMap<>();
```

**注释规范：**
```java
/**
 * 执行 MagicAPI 中的接口
 *
 * @param method  请求方法
 * @param path    请求路径
 * @param context 变量信息
 * @return 执行结果
 */
<T> T execute(String method, String path, Map<String, Object> context);
```

**异常处理：**
```java
// 业务异常使用 MagicAPIException
if (info == null) {
    throw new MagicAPIException(String.format("找不到对应接口 [%s:%s]", method, path));
}

// 记录异常日志
logger.error("接口{}请求出错", requestEntity.getRequest().getRequestURI(), root);
```

#### 2.1.2 magic-script 脚本规范

**脚本结构：**
```javascript
// 1. 参数验证（可选）
if (!id) {
    return { code: 400, message: 'id 不能为空' };
}

// 2. 业务逻辑
var user = db.selectOne("SELECT * FROM user WHERE id = ?", [id]);

// 3. 返回结果
return user;
```

**命名规范：**
```javascript
// 变量：小驼峰
var userInfo = db.selectOne(...);
var orderList = db.selectList(...);

// 函数：小驼峰
function getUserById(id) {
    return db.selectOne(...);
}

// 常量：全大写
var DEFAULT_PAGE_SIZE = 10;
```

**SQL 编写规范：**
```javascript
// ✅ 推荐：使用参数化查询
var user = db.selectOne("SELECT * FROM user WHERE id = ?", [id]);

// ❌ 不推荐：字符串拼接（SQL 注入风险）
var user = db.selectOne("SELECT * FROM user WHERE id = " + id);

// ✅ 推荐：使用 db.update 返回影响行数
var count = db.update("UPDATE user SET name = ? WHERE id = ?", [name, id]);

// ✅ 推荐：使用事务
db.transaction(() => {
    db.update("INSERT INTO ...");
    db.update("UPDATE ...");
});
```

### 2.2 接口设计规范

#### 2.2.1 RESTful 风格

```
GET    /user/:id          # 获取单个用户
GET    /user              # 获取用户列表
POST   /user              # 创建用户
PUT    /user/:id          # 更新用户（全量）
PATCH  /user/:id          # 更新用户（部分）
DELETE /user/:id          # 删除用户
```

#### 2.2.2 路径命名

```javascript
// ✅ 推荐：使用名词复数，小写，连字符分隔
/user-list
/user-profile
/order-items

// ❌ 不推荐：使用动词、驼峰、下划线
/getUser
/userList
/user_profile
```

#### 2.2.3 响应格式

**成功响应：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "name": "张三"
  }
}
```

**错误响应：**
```json
{
  "code": 400,
  "message": "参数错误：id 不能为空",
  "data": null
}
```

**分页响应：**
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

### 2.3 数据库设计规范

#### 2.3.1 表命名

```sql
-- ✅ 推荐：小写，下划线分隔，业务前缀
t_user
t_user_profile
t_order_item

-- ❌ 不推荐：大写、驼峰、无意义前缀
User
tUser
tmp_table_123
```

#### 2.3.2 字段命名

```sql
-- ✅ 推荐：小写，下划线分隔，语义清晰
CREATE TABLE t_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '主键 ID',
    user_name VARCHAR(50) NOT NULL COMMENT '用户名',
    email VARCHAR(100) COMMENT '邮箱',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间'
);
```

#### 2.3.3 索引规范

```sql
-- 主键索引：自增 ID 或雪花 ID
PRIMARY KEY (id)

-- 唯一索引：业务唯一键
UNIQUE INDEX uk_email (email)

-- 普通索引：查询条件字段
INDEX idx_user_name (user_name)

-- 联合索引：最左前缀原则
INDEX idx_status_created (status, created_at)
```

### 2.4 配置规范

#### 2.4.1 application.yml 配置

```yaml
magic-api:
  web: /magic/web                    # Web 界面路径
  prefix: /api                       # 接口路径前缀
  banner: true                       # 是否打印 banner
  
  # 安全配置
  security:
    username: admin                  # 登录用户名
    password: admin123               # 登录密码
  
  # 资源存储配置
  resource:
    type: file                       # 存储类型：file | database
    location: /data/magic-api        # 文件存储路径
    # type: database                 # 使用数据库存储
    # datasource: master             # 数据源名称
    # table-name: magic_api_resource # 表名
  
  # 备份配置（强烈建议开启）
  backup:
    enable: true
    datasource: master
    table-name: magic_api_backup
    max-history: 30                  # 保留最近 30 天备份
  
  # SQL 配置
  show-sql: true                     # 是否打印 SQL
  sql-column-case: default           # SQL 列名转换：default | camel | snake
  
  # 响应配置
  response-code:
    success: 200                     # 成功码
    invalid: 400                     # 参数错误码
    exception: 500                   # 异常码
```

### 2.5 版本控制规范

#### 2.5.1 接口版本管理

```javascript
// 方式一：路径版本
/api/v1/user
/api/v2/user

// 方式二：请求头版本
X-API-Version: 1.0

// 方式三：查询参数版本
/api/user?version=1.0
```

#### 2.5.2 Git 提交规范

```bash
# 功能新增
feat: 添加用户管理接口

# Bug 修复
fix: 修复用户查询空指针异常

# 文档更新
docs: 更新 API 使用文档

# 代码重构
refactor: 重构数据库模块

# 配置修改
chore: 更新依赖版本
```

---

## 3. 质量要求

### 3.1 代码质量

#### 3.1.1 可维护性

- **单一职责**：每个接口只做一件事
- **代码复用**：公共逻辑提取为函数
- **命名清晰**：变量、函数名语义明确
- **注释充分**：复杂逻辑必须有注释

```javascript
// ✅ 推荐：提取公共函数
function pageQuery(sql, params, page, pageSize) {
    var total = db.selectOne("SELECT COUNT(*) FROM (" + sql + ") t", params).total;
    var list = db.selectList(sql + " LIMIT ?, ?", params, [(page - 1) * pageSize, pageSize]);
    return { list: list, total: total, page: page, pageSize: pageSize };
}

// 使用
return pageQuery("SELECT * FROM user WHERE status = ?", [1], page, pageSize);
```

#### 3.1.2 可读性

- 避免嵌套过深（不超过 3 层）
- 避免单行过长（不超过 120 字符）
- 适当空行分隔逻辑块
- 使用有意义的变量名

```javascript
// ✅ 推荐
if (user == null) {
    return null;
}

if (user.getStatus() != 1) {
    return null;
}

return user;

// ❌ 不推荐
return user != null && user.getStatus() == 1 ? user : null;
```

#### 3.1.3 健壮性

- 参数验证：必填项、类型、范围
- 异常处理：捕获并记录异常
- 空值处理：避免空指针
- 事务管理：保证数据一致性

```javascript
// ✅ 推荐：完整的参数验证和异常处理
if (!userId) {
    return { code: 400, message: 'userId 不能为空' };
}

try {
    var user = db.selectOne("SELECT * FROM user WHERE id = ?", [userId]);
    if (!user) {
        return { code: 404, message: '用户不存在' };
    }
    return user;
} catch (e) {
    log.error("查询用户失败", e);
    return { code: 500, message: '系统异常' };
}
```

### 3.2 性能要求

#### 3.2.1 响应时间

| 接口类型 | 目标响应时间 | 可接受范围 |
|---------|------------|-----------|
| 简单查询 | < 50ms | < 100ms |
| 复杂查询 | < 200ms | < 500ms |
| 批量操作 | < 500ms | < 1000ms |
| 外部调用 | < 1000ms | < 3000ms |

#### 3.2.2 数据库优化

```javascript
// ✅ 推荐：使用索引字段查询
var user = db.selectOne("SELECT * FROM user WHERE email = ?", [email]);
// email 字段有索引

// ❌ 不推荐：全表扫描
var user = db.selectOne("SELECT * FROM user WHERE name LIKE ?", ["%张三%"]);

// ✅ 推荐：只查询需要的字段
var userInfo = db.selectOne("SELECT id, name, email FROM user WHERE id = ?", [id]);

// ❌ 不推荐：SELECT *
var user = db.selectOne("SELECT * FROM user WHERE id = ?", [id]);

// ✅ 推荐：分页查询
var list = db.selectList("SELECT * FROM user LIMIT ?, ?", [offset, limit]);

// ❌ 不推荐：一次性查询大量数据
var list = db.selectList("SELECT * FROM user");
```

#### 3.2.3 缓存使用

```javascript
// ✅ 推荐：热点数据缓存
var cacheKey = "user:" + userId;
var user = cache.get(cacheKey);
if (!user) {
    user = db.selectOne("SELECT * FROM user WHERE id = ?", [userId]);
    if (user) {
        cache.set(cacheKey, user, 3600); // 缓存 1 小时
    }
}
return user;
```

### 3.3 安全要求

#### 3.3.1 认证授权

```javascript
// ✅ 推荐：接口级别权限控制
// 在接口选项中配置：require-login = true

// 或在脚本中验证
var currentUser = session.get("user");
if (!currentUser) {
    return { code: 401, message: '未登录' };
}

// 角色验证
if (currentUser.role != 'ADMIN') {
    return { code: 403, message: '无权限' };
}
```

#### 3.3.2 SQL 注入防护

```javascript
// ✅ 推荐：参数化查询
var user = db.selectOne("SELECT * FROM user WHERE name = ?", [name]);

// ❌ 禁止：字符串拼接
var user = db.selectOne("SELECT * FROM user WHERE name = '" + name + "'");
```

#### 3.3.3 XSS 防护

```javascript
// ✅ 推荐：输出编码
response.setHeader("Content-Type", "application/json");
response.setHeader("X-Content-Type-Options", "nosniff");

// ✅ 推荐：过滤用户输入
function sanitize(input) {
    return input.replace(/[<>&"']/g, function(c) {
        return {'<':'&lt;', '>':'&gt;', '&':'&amp;', '"':'&quot;', "'":"&#39;"}[c];
    });
}
```

### 3.4 测试要求

#### 3.4.1 接口测试

- 使用内置调试功能测试接口
- 覆盖正常场景和异常场景
- 验证参数边界条件
- 保存测试用例

#### 3.4.2 性能测试

- 关键接口进行压力测试
- 监控慢查询日志
- 定期性能回归测试

---

## 4. 文档规范

### 4.1 接口文档

每个接口应包含：

```markdown
## 接口名称

**路径：** `GET /user/:id`

**描述：** 根据 ID 获取用户信息

**请求参数：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | Long | 是 | 用户 ID |

**响应示例：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "name": "张三",
    "email": "zhangsan@example.com"
  }
}
```
```

### 4.2 变更日志

每次接口变更应记录：

```markdown
## 2026-03-14

- feat: 添加用户批量导入接口
- fix: 修复用户查询分页错误
- docs: 更新用户管理接口文档
```

---

## 5. 运维规范

### 5.1 部署规范

- 生产环境必须配置备份
- 生产环境必须开启认证
- 使用数据库存储资源（集群环境）
- 配置日志保留策略

### 5.2 监控规范

- 监控接口响应时间
- 监控错误率
- 监控慢查询
- 监控备份状态

### 5.3 备份规范

- 每天自动备份
- 保留最近 30 天备份
- 定期测试还原流程
- 备份数据异地存储

---

## 6. 违规处理

| 违规类型 | 处理方式 |
|---------|---------|
| 未经验证上线 | 立即回滚，通报批评 |
| SQL 注入漏洞 | 立即修复，安全审计 |
| 未备份导致数据丢失 | 责任人承担后果 |
| 性能问题影响生产 | 优化整改，性能测试 |

---

## 7. 附则

- 本宪法由项目维护团队解释和修订
- 所有贡献者应遵守本宪法
- 违反宪法的代码将不被接受
- 宪法根据项目发展持续演进

---

**记住：好的规范不是束缚，而是保护。遵守规范，让开发更高效，让系统更可靠。**
