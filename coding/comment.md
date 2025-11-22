# 代码注释规范

代码注释是提高代码可读性、可维护性的重要手段。良好的注释能够帮助开发者快速理解代码意图，降低维护成本。

---

## 注释原则

### 1. 注释应该说明"为什么"，而不是"是什么"
- ✅ 好的注释：解释代码背后的业务逻辑、设计决策、特殊处理原因
- ❌ 坏的注释：重复代码本身已经表达清楚的内容

```javascript
// ❌ 坏的注释
// 将用户年龄设置为18
user.age = 18;

// ✅ 好的注释
// 根据法律要求，未成年用户默认设置为18岁以保护隐私
user.age = 18;
```

### 2. 代码即文档
- 优先通过清晰的命名、合理的结构来表达意图
- 复杂逻辑才需要注释说明

```python
# ❌ 不推荐：通过注释说明不清晰的变量
# 用户数据
d = get_data()

# ✅ 推荐：使用清晰的命名
user_data = get_user_data()
```

### 3. 保持注释与代码同步
- 修改代码时必须同步更新相关注释
- 过时的注释比没有注释更糟糕

### 4. 使用母语或英语，保持一致
- 团队统一使用中文或英文注释
- 避免中英文混用（专业术语除外）

---

## 文件头注释

每个源代码文件应包含文件头注释，说明文件用途、作者、版本等信息。

### JavaScript/TypeScript

```javascript
/**
 * 用户认证服务
 * 
 * 提供用户登录、登出、Token 刷新等功能
 * 
 * @module services/auth
 * @author 张三 <zhangsan@example.com>
 * @created 2025-01-01
 * @modified 2025-01-15
 */
```

### Python

```python
"""
用户认证服务

提供用户登录、登出、Token 刷新等功能

Author: 张三 <zhangsan@example.com>
Created: 2025-01-01
Modified: 2025-01-15
"""
```

### Java

```java
/**
 * 用户认证服务
 * 
 * <p>提供用户登录、登出、Token 刷新等功能</p>
 * 
 * @author 张三 <zhangsan@example.com>
 * @version 1.0.0
 * @since 2025-01-01
 */
```

---

## 函数/方法注释

函数注释应说明功能、参数、返回值、异常等信息。

### JavaScript/TypeScript (JSDoc)

```javascript
/**
 * 用户登录
 * 
 * 验证用户凭证并生成访问令牌
 * 
 * @param {string} username - 用户名
 * @param {string} password - 密码
 * @returns {Promise<{token: string, expiresIn: number}>} 包含 token 和过期时间的对象
 * @throws {AuthError} 当认证失败时抛出
 * 
 * @example
 * const result = await login('user@example.com', 'password123');
 * console.log(result.token);
 */
async function login(username, password) {
  // 实现代码
}
```

### Python (Docstring)

```python
def login(username: str, password: str) -> dict:
    """
    用户登录
    
    验证用户凭证并生成访问令牌
    
    Args:
        username (str): 用户名
        password (str): 密码
    
    Returns:
        dict: 包含 token 和过期时间的字典
            - token (str): 访问令牌
            - expires_in (int): 过期时间（秒）
    
    Raises:
        AuthError: 当认证失败时抛出
    
    Examples:
        >>> result = login('user@example.com', 'password123')
        >>> print(result['token'])
    """
    # 实现代码
```

### Java (JavaDoc)

```java
/**
 * 用户登录
 * 
 * <p>验证用户凭证并生成访问令牌</p>
 * 
 * @param username 用户名
 * @param password 密码
 * @return 包含 token 和过期时间的 LoginResult 对象
 * @throws AuthException 当认证失败时抛出
 * 
 * @see LoginResult
 * @since 1.0.0
 */
public LoginResult login(String username, String password) throws AuthException {
    // 实现代码
}
```

### Go

```go
// Login 用户登录
//
// 验证用户凭证并生成访问令牌
//
// 参数:
//   - username: 用户名
//   - password: 密码
//
// 返回:
//   - *LoginResult: 包含 token 和过期时间的结构体
//   - error: 认证失败时返回错误
//
// 示例:
//   result, err := Login("user@example.com", "password123")
//   if err != nil {
//       log.Fatal(err)
//   }
func Login(username, password string) (*LoginResult, error) {
    // 实现代码
}
```

---

## 类/接口注释

### JavaScript/TypeScript

```typescript
/**
 * 用户服务类
 * 
 * 提供用户管理相关的所有操作，包括 CRUD、认证、权限管理等
 * 
 * @class UserService
 * @implements {IUserService}
 */
class UserService implements IUserService {
  /**
   * 创建用户服务实例
   * 
   * @param {Database} db - 数据库连接实例
   * @param {Logger} logger - 日志记录器
   */
  constructor(db, logger) {
    this.db = db;
    this.logger = logger;
  }
}
```

### Python

```python
class UserService:
    """
    用户服务类
    
    提供用户管理相关的所有操作，包括 CRUD、认证、权限管理等
    
    Attributes:
        db (Database): 数据库连接实例
        logger (Logger): 日志记录器
    """
    
    def __init__(self, db: Database, logger: Logger):
        """
        初始化用户服务
        
        Args:
            db (Database): 数据库连接实例
            logger (Logger): 日志记录器
        """
        self.db = db
        self.logger = logger
```

### Java

```java
/**
 * 用户服务类
 * 
 * <p>提供用户管理相关的所有操作，包括 CRUD、认证、权限管理等</p>
 * 
 * @see IUserService
 * @since 1.0.0
 */
public class UserService implements IUserService {
    /**
     * 数据库连接实例
     */
    private final Database db;
    
    /**
     * 日志记录器
     */
    private final Logger logger;
    
    /**
     * 创建用户服务实例
     * 
     * @param db 数据库连接实例
     * @param logger 日志记录器
     */
    public UserService(Database db, Logger logger) {
        this.db = db;
        this.logger = logger;
    }
}
```

---

## 行内注释

### 单行注释

```javascript
// 使用单行注释说明单个语句或简单逻辑

// 缓存用户数据以提高性能
const cachedUser = cache.get(userId);

// 验证邮箱格式
if (!emailRegex.test(email)) {
  throw new ValidationError('Invalid email format');
}
```

### 多行注释

```javascript
/*
 * 复杂算法的详细说明
 * 这里使用改进的快速排序算法，针对小数组（长度 < 10）使用插入排序
 * 以减少递归开销，提高整体性能
 */
function hybridSort(arr) {
  if (arr.length < 10) {
    return insertionSort(arr);
  }
  return quickSort(arr);
}
```

### 代码块注释

```python
# === 数据验证 ===
# 在处理用户输入前，进行完整的数据验证
# 包括类型检查、范围检查、格式检查等
if not isinstance(data, dict):
    raise TypeError("Data must be a dictionary")

if 'email' not in data:
    raise ValueError("Email is required")

# === 业务逻辑处理 ===
# 根据用户类型执行不同的处理流程
if user.is_premium:
    process_premium_user(user)
else:
    process_regular_user(user)
```

---

## 特殊标记注释

使用标准标记来标识代码中的特殊情况。

### TODO - 待完成的工作

```javascript
// TODO(zhangsan): 实现缓存失效机制
// TODO: 添加输入验证 [ISSUE-123]
// TODO(2025-02-01): 在下个版本中移除此临时方案
```

### FIXME - 需要修复的问题

```python
# FIXME: 并发场景下可能出现竞态条件
# FIXME(lisi): 性能瓶颈，需要优化查询语句 [BUG-456]
```

### HACK - 临时解决方案

```java
// HACK: 绕过第三方库的 Bug，等待官方修复
// 参考: https://github.com/library/issues/789
```

### NOTE/INFO - 重要说明

```javascript
// NOTE: 此方法会修改原数组，调用时需注意
// INFO: 返回值为 null 表示未找到匹配项
```

### DEPRECATED - 已废弃

```typescript
/**
 * @deprecated 使用 newLogin() 替代，将在 v2.0 中移除
 */
function oldLogin() {
  // ...
}
```

### OPTIMIZE - 性能优化点

```python
# OPTIMIZE: 考虑使用批量查询减少数据库请求次数
```

### SECURITY - 安全相关

```java
// SECURITY: 敏感数据，必须加密存储
// SECURITY: 需要验证用户权限，防止越权访问
```

---

## 注释最佳实践

### 1. 解释复杂的业务逻辑

```javascript
/**
 * 计算订单折扣
 * 
 * 折扣规则（按优先级应用）：
 * 1. VIP 用户享受 15% 折扣
 * 2. 订单金额 > 1000 元享受 10% 折扣
 * 3. 首次购买享受 5% 折扣
 * 4. 优惠券折扣（如果有）
 * 
 * 注意：折扣不叠加，仅应用优惠力度最大的一项
 */
function calculateDiscount(order, user) {
  // 实现代码
}
```

### 2. 说明特殊处理的原因

```python
# 由于第三方支付接口的限制，金额必须转换为整数（分为单位）
# 直接使用浮点数可能导致精度问题，参考文档：
# https://pay.example.com/docs/api#amount-format
amount_in_cents = int(amount * 100)
```

### 3. 标注假设和约束

```java
/**
 * 批量导入用户数据
 * 
 * 假设：
 * - 输入数据已经过验证
 * - 用户邮箱唯一
 * - 最大批量大小为 1000 条
 * 
 * 约束：
 * - 单次操作不超过 30 秒
 * - 失败时回滚整个批次
 */
public void batchImportUsers(List<User> users) {
    // 实现代码
}
```

### 4. 引用外部资源

```javascript
// 实现 OAuth 2.0 授权码流程
// 规范参考：https://tools.ietf.org/html/rfc6749#section-4.1
// 
// 流程说明：
// 1. 重定向到授权服务器
// 2. 用户授权后获取授权码
// 3. 使用授权码换取访问令牌
```

### 5. 警告潜在问题

```python
def delete_user(user_id: int) -> None:
    """
    删除用户
    
    ⚠️ 警告：此操作不可逆！
    删除用户会同时删除：
    - 用户的所有订单记录
    - 用户的收藏和历史记录
    - 用户的积分和优惠券
    
    建议：在生产环境中使用软删除（标记为已删除）
    """
    # 实现代码
```

---

## 注释反模式（应避免）

### ❌ 1. 无用的噪音注释

```javascript
// 不好的例子
let i = 0; // 设置 i 为 0
i++; // i 加 1
return i; // 返回 i
```

### ❌ 2. 注释掉的代码

```python
# 不好的例子 - 应该使用版本控制而不是注释旧代码
def process_data(data):
    # old_method(data)  # 旧的处理方式
    # another_old_method(data)
    new_method(data)  # 新的处理方式
```

**建议**：删除注释的代码，依赖 Git 历史记录。

### ❌ 3. 日志式注释

```java
// 不好的例子
// 2025-01-01: 添加验证逻辑 - 张三
// 2025-01-05: 修复空指针问题 - 李四
// 2025-01-10: 优化性能 - 王五
public void validate(String input) {
    // 实现代码
}
```

**建议**：使用 Git commit 记录变更历史。

### ❌ 4. 过度注释

```javascript
// 不好的例子
// 创建一个新的用户对象
const user = new User();
// 设置用户名
user.name = 'John';
// 设置用户邮箱
user.email = 'john@example.com';
// 保存用户到数据库
await user.save();
```

### ❌ 5. 误导性注释

```python
# 不好的例子 - 注释与代码不符
# 计算用户平均年龄
total_age = sum(user.age for user in users)  # 实际上是计算总和
```

---

## 文档注释规范

### API 文档生成

使用标准文档格式，支持自动生成 API 文档：

- **JavaScript/TypeScript**: JSDoc → 生成 HTML 文档
- **Python**: Docstring → Sphinx/pdoc
- **Java**: JavaDoc → HTML 文档
- **Go**: godoc → 在线文档

### 示例：完整的函数文档

```typescript
/**
 * 分页查询用户列表
 * 
 * 根据查询条件返回用户列表，支持分页、排序和过滤
 * 
 * @param {Object} options - 查询选项
 * @param {number} [options.page=1] - 页码（从 1 开始）
 * @param {number} [options.limit=10] - 每页数量（最大 100）
 * @param {string} [options.sort='created_at'] - 排序字段
 * @param {'asc'|'desc'} [options.order='desc'] - 排序方向
 * @param {Object} [options.filters] - 过滤条件
 * @param {string} [options.filters.status] - 用户状态
 * @param {string} [options.filters.role] - 用户角色
 * 
 * @returns {Promise<{
 *   data: User[],
 *   meta: {
 *     page: number,
 *     limit: number,
 *     total: number,
 *     totalPages: number
 *   }
 * }>} 包含用户列表和分页信息的对象
 * 
 * @throws {ValidationError} 当参数验证失败时
 * @throws {DatabaseError} 当数据库查询失败时
 * 
 * @example
 * // 基本用法
 * const result = await getUserList({ page: 1, limit: 20 });
 * 
 * @example
 * // 带过滤条件
 * const result = await getUserList({
 *   page: 1,
 *   limit: 20,
 *   filters: { status: 'active', role: 'admin' }
 * });
 * 
 * @see {@link User} 用户模型定义
 * @see {@link https://docs.example.com/api/users|API 文档}
 * 
 * @since 1.0.0
 * @version 1.2.0
 */
async function getUserList(options) {
  // 实现代码
}
```

---

## 多语言注释示例对比

### JavaScript/TypeScript

```typescript
/**
 * 字符串工具类
 */
class StringUtils {
  /**
   * 转换为驼峰命名
   * @param str 输入字符串
   * @returns 驼峰格式字符串
   */
  static toCamelCase(str: string): string {
    // 分割字符串并转换
    return str.split('-').map((word, index) => {
      // 第一个单词保持小写，其余首字母大写
      return index === 0 ? word : word.charAt(0).toUpperCase() + word.slice(1);
    }).join('');
  }
}
```

### Python

```python
class StringUtils:
    """字符串工具类"""
    
    @staticmethod
    def to_camel_case(s: str) -> str:
        """
        转换为驼峰命名
        
        Args:
            s: 输入字符串
            
        Returns:
            驼峰格式字符串
        """
        # 分割字符串并转换
        words = s.split('-')
        # 第一个单词保持小写，其余首字母大写
        return words[0] + ''.join(word.capitalize() for word in words[1:])
```

### Java

```java
/**
 * 字符串工具类
 */
public class StringUtils {
    /**
     * 转换为驼峰命名
     * 
     * @param str 输入字符串
     * @return 驼峰格式字符串
     */
    public static String toCamelCase(String str) {
        // 分割字符串
        String[] words = str.split("-");
        StringBuilder result = new StringBuilder(words[0]);
        
        // 处理后续单词（首字母大写）
        for (int i = 1; i < words.length; i++) {
            result.append(Character.toUpperCase(words[i].charAt(0)))
                  .append(words[i].substring(1));
        }
        
        return result.toString();
    }
}
```

### Go

```go
// StringUtils 字符串工具类
type StringUtils struct{}

// ToCamelCase 转换为驼峰命名
//
// 参数:
//   - str: 输入字符串
//
// 返回:
//   - string: 驼峰格式字符串
func (u *StringUtils) ToCamelCase(str string) string {
    // 分割字符串
    words := strings.Split(str, "-")
    
    // 第一个单词保持小写，其余首字母大写
    for i := 1; i < len(words); i++ {
        words[i] = strings.Title(words[i])
    }
    
    return strings.Join(words, "")
}
```

---

## 注释检查清单

在代码审查时，使用以下清单检查注释质量：

- [ ] 注释是否解释了"为什么"而非"是什么"
- [ ] 注释是否与代码保持同步
- [ ] 复杂逻辑是否有充分的说明
- [ ] 公共 API 是否有完整的文档注释
- [ ] 特殊处理是否说明了原因
- [ ] 是否移除了注释掉的代码
- [ ] 是否移除了过时的注释
- [ ] TODO/FIXME 是否包含责任人和期限
- [ ] 是否避免了无意义的噪音注释
- [ ] 是否使用了团队统一的语言（中文/英文）

---

## 工具推荐

### 文档生成工具
- **JavaScript/TypeScript**: JSDoc, TypeDoc, Docusaurus
- **Python**: Sphinx, pdoc, MkDocs
- **Java**: JavaDoc, Dokka
- **Go**: godoc, pkgsite
- **C#**: DocFX, Sandcastle

### 注释检查工具
- **ESLint** (JavaScript): 检查 JSDoc 格式
- **Pylint** (Python): 检查 docstring 完整性
- **Checkstyle** (Java): 检查 JavaDoc 规范
- **golangci-lint** (Go): 检查注释规范

### IDE 插件
- **Better Comments**: 高亮 TODO/FIXME 等标记
- **Document This**: 自动生成文档注释模板
- **Comment Anchors**: 管理和跳转特殊标记注释

---

## 总结

良好的代码注释应该：
1. **有价值**：提供代码本身无法表达的信息
2. **准确**：与代码保持同步，避免误导
3. **简洁**：言简意赅，避免冗余
4. **一致**：遵循团队和项目的统一规范
5. **及时**：在编写代码时同步添加注释

**记住**：最好的注释是不需要注释的代码——通过清晰的命名、合理的结构、良好的设计来减少注释需求。注释是补充，而非替代。