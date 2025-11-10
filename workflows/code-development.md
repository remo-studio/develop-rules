# 代码开发流程规范

本文档定义 AI 辅助的代码开发流程，包括代码编写、审查和质量控制。

## 开发流程概览

```
需求确认 → 设计审查 → 分支创建 → AI 辅助编码 → 自测 → 提交 PR → 
代码审查(人工+AI) → 自动化测试 → 合并 → 部署
```

## 分支策略

遵循 [Git Flow 规范](../github/git-flow.md)：

### 分支命名
```bash
# 新功能
feature/<req-id>-<short-desc>
# 示例
feature/req-2025-001-oauth-login

# 缺陷修复
bugfix/<bug-id>-<short-desc>
# 示例
bugfix/bug-123-login-timeout

# 紧急修复
hotfix/<version>-<short-desc>
# 示例
hotfix/1.2.1-security-patch
```

### 分支操作
```bash
# 1. 从 develop 创建功能分支
git checkout develop
git pull origin develop
git checkout -b feature/req-2025-001-oauth-login

# 2. 开发过程中定期同步 develop
git checkout develop
git pull origin develop
git checkout feature/req-2025-001-oauth-login
git merge develop

# 3. 开发完成后推送分支
git push -u origin feature/req-2025-001-oauth-login

# 4. 创建 Pull Request
# 在 GitHub 网页上操作
```

## AI 辅助代码编写

### GitHub Copilot 使用指南

#### 1. 智能补全
在编辑器中开始输入，Copilot 会自动提供建议：

```javascript
// 输入函数签名，Copilot 自动生成实现
async function authenticateUser(username, password) {
  // Copilot 会建议：
  // const user = await User.findOne({ username });
  // if (!user) throw new Error('User not found');
  // const isValid = await bcrypt.compare(password, user.passwordHash);
  // if (!isValid) throw new Error('Invalid password');
  // return generateToken(user);
}
```

#### 2. 注释驱动开发
写详细注释，让 Copilot 生成代码：

```python
# 函数：验证用户邮箱格式
# 参数：email (str) - 用户邮箱地址
# 返回：bool - 邮箱格式是否有效
# 规则：必须包含 @ 符号，域名至少两级
def validate_email(email: str) -> bool:
    # Copilot 会生成完整的正则验证逻辑
    pass
```

#### 3. 测试用例生成
```typescript
// 为已有函数生成测试用例
describe('authenticateUser', () => {
  // Copilot 会建议多个测试用例：
  // - 成功登录
  // - 用户不存在
  // - 密码错误
  // - 空用户名
  // - 空密码
});
```

#### 4. 重构建议
```java
// 选中代码，按 Ctrl+Enter 请求 Copilot 提供重构建议
public void processOrder(Order order) {
    // 原有代码...
    // Copilot 会建议提取方法、优化逻辑等
}
```

### Copilot Chat 使用场景

#### 1. 代码解释
```
# 提问
@workspace /explain 这个函数的作用是什么？

# Copilot 会解释选中的代码块
```

#### 2. 代码优化
```
# 提问
@workspace /optimize 如何提高这段代码的性能？

# Copilot 会提供优化建议
```

#### 3. 查找 Bug
```
# 提问
@workspace /fix 这段代码有什么问题？

# Copilot 会分析潜在问题并提供修复建议
```

#### 4. 生成文档
```
# 提问
@workspace /doc 为这个函数生成 JSDoc 注释

# Copilot 会生成完整的文档注释
```

### ChatGPT/Claude 辅助开发

#### 提示词模板

**代码生成**：
```
请用 [语言] 实现以下功能：

需求：[详细描述]
输入：[输入参数说明]
输出：[输出结果说明]
约束：[性能、安全等约束]

请包含：
1. 完整的函数实现
2. 错误处理
3. JSDoc/注释
4. 使用示例
```

**代码审查**：
```
请审查以下代码，重点关注：
1. 潜在的 bug
2. 性能问题
3. 安全漏洞
4. 代码风格

代码：
[粘贴代码]
```

**重构建议**：
```
请为以下代码提供重构建议：

代码：
[粘贴代码]

目标：
- 提高可读性
- 减少重复
- 提高性能
- 符合设计模式
```

## 代码规范

### 通用规范

1. **命名规范**
   - 变量名：驼峰命名（camelCase）
   - 常量：大写下划线（UPPER_SNAKE_CASE）
   - 类名：帕斯卡命名（PascalCase）
   - 文件名：小写连字符（kebab-case.js）

2. **注释规范**
   - 所有公共 API 必须有文档注释
   - 复杂逻辑必须有行内注释说明
   - 注释应解释"为什么"而非"是什么"

3. **函数规范**
   - 单一职责：一个函数只做一件事
   - 参数数量：不超过 3 个（可用对象封装）
   - 函数长度：不超过 50 行

4. **错误处理**
   - 所有异步操作必须处理错误
   - 使用自定义错误类型
   - 错误信息应包含上下文

### 语言特定规范

#### JavaScript/TypeScript
```typescript
// ✅ 好的示例
interface User {
  id: string;
  username: string;
  email: string;
}

async function getUserById(id: string): Promise<User> {
  try {
    const user = await db.users.findOne({ id });
    if (!user) {
      throw new NotFoundError(`User not found: ${id}`);
    }
    return user;
  } catch (error) {
    logger.error('Failed to get user', { id, error });
    throw error;
  }
}

// ❌ 不好的示例
async function getUser(id) {  // 缺少类型
  let user = await db.users.findOne({ id });  // 不要使用 let
  if (!user) return null;  // 不要返回 null
  return user;
}
```

#### Python
```python
# ✅ 好的示例
from typing import Optional
import logging

logger = logging.getLogger(__name__)

class UserNotFoundError(Exception):
    """用户不存在错误"""
    pass

def get_user_by_id(user_id: str) -> dict:
    """
    根据 ID 获取用户信息
    
    Args:
        user_id: 用户唯一标识
        
    Returns:
        用户信息字典
        
    Raises:
        UserNotFoundError: 用户不存在
    """
    try:
        user = db.users.find_one({'id': user_id})
        if not user:
            raise UserNotFoundError(f'User not found: {user_id}')
        return user
    except Exception as e:
        logger.error(f'Failed to get user: {user_id}', exc_info=True)
        raise

# ❌ 不好的示例
def getUser(id):  # 命名不符合规范，缺少类型注解
    user = db.users.find_one({'id': id})
    return user  # 未处理用户不存在的情况
```

## 代码审查

### 审查清单

#### 功能性
- [ ] 代码实现了需求的所有验收标准
- [ ] 边界条件和异常情况处理正确
- [ ] 没有明显的逻辑错误

#### 代码质量
- [ ] 代码清晰易读，命名恰当
- [ ] 没有重复代码（DRY 原则）
- [ ] 函数和类职责单一（SOLID 原则）
- [ ] 适当的抽象层次

#### 性能
- [ ] 没有明显的性能问题
- [ ] 数据库查询优化（使用索引、避免 N+1）
- [ ] 大数据集处理使用分页或流式处理

#### 安全性
- [ ] 输入验证和清理
- [ ] SQL 注入防护（使用参数化查询）
- [ ] XSS 防护（输出编码）
- [ ] 敏感数据不打印到日志
- [ ] 认证和授权正确实现

#### 测试
- [ ] 单元测试覆盖核心逻辑
- [ ] 测试用例充分（正常、异常、边界）
- [ ] 集成测试覆盖关键流程

#### 文档
- [ ] 公共 API 有文档注释
- [ ] 复杂逻辑有注释说明
- [ ] README 更新（如有必要）

### 人工审查流程

1. **自审**：提交 PR 前自己先审查一遍
2. **提交 PR**：填写完整的 PR 描述
3. **自动检查**：等待 CI/CD 通过
4. **请求审查**：指派 2 名审查者
5. **审查反馈**：审查者提出意见
6. **修改代码**：根据反馈修改
7. **批准合并**：至少 1 名审查者批准后可合并

### AI 辅助代码审查

#### GitHub Actions 集成
创建 `.github/workflows/ai-code-review.yml`：

```yaml
name: AI Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  ai-review:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: AI Code Review
        uses: coderabbitai/ai-pr-reviewer@latest
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          openai_api_key: ${{ secrets.OPENAI_API_KEY }}
          review_simple_changes: false
          review_comment_lgtm: false
```

#### 使用 AI 审查服务
- **CodeRabbit**：自动代码审查和建议
- **Codacy**：代码质量和安全性分析
- **SonarCloud**：代码质量和技术债务追踪

## 代码质量门禁

### 自动化检查

#### 1. Linting
```yaml
# .github/workflows/lint.yml
name: Lint

on: [pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run lint
```

#### 2. 测试覆盖率
```yaml
# .github/workflows/test.yml
name: Test

on: [pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm test -- --coverage
      - name: Check coverage
        run: |
          if [ $(npm run coverage:check) -lt 80 ]; then
            echo "Coverage below 80%"
            exit 1
          fi
```

#### 3. 安全扫描
```yaml
# .github/workflows/security.yml
name: Security Scan

on: [pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

### 质量标准
- **代码覆盖率**：≥ 80%
- **复杂度**：圈复杂度 ≤ 10
- **重复率**：< 3%
- **Lint 错误**：0 个
- **安全漏洞**：0 个高危

## 调试技巧

### 1. 日志调试
```javascript
// 使用结构化日志
logger.info('User authentication', {
  username,
  timestamp: new Date(),
  ip: req.ip,
});

// 不要使用 console.log
// console.log('user:', username);  // ❌
```

### 2. 断点调试
```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Program",
      "program": "${workspaceFolder}/src/index.js",
      "skipFiles": ["<node_internals>/**"]
    }
  ]
}
```

### 3. 单元测试调试
```bash
# 只运行特定测试
npm test -- --grep "authenticateUser"

# 调试模式运行测试
node --inspect-brk node_modules/.bin/jest --runInBand
```

### 4. AI 辅助调试
```
# 向 Copilot Chat 提问
@workspace /explain 为什么这段代码会抛出 NullPointerException？

[粘贴代码和错误栈]
```

## 最佳实践

1. **TDD（测试驱动开发）**
   - 先写测试，再写实现
   - 红-绿-重构循环
   
2. **小步提交**
   - 频繁提交，保持每次提交原子性
   - Commit 消息清晰描述变更
   
3. **持续集成**
   - 每天至少合并一次代码
   - 及时解决冲突
   
4. **代码重构**
   - 定期重构，清理技术债
   - 重构时保持测试通过
   
5. **文档同步**
   - 代码变更时同步更新文档
   - API 变更更新 OpenAPI 规范
   
6. **AI 辅助，人工决策**
   - 使用 AI 提高效率
   - 关键决策仍需人工判断
   
7. **安全第一**
   - 所有输入必须验证
   - 敏感数据加密存储
   - 定期安全审计

## 常见问题

### Q1: AI 生成的代码可以直接使用吗？
A: 不可以。必须仔细审查 AI 生成的代码，确保其正确性、安全性和性能。

### Q2: 如何处理 AI 生成的不完善代码？
A: 将 AI 代码作为起点，根据实际需求调整和完善。AI 是辅助工具，不是替代品。

### Q3: 代码审查发现问题怎么办？
A: 根据审查意见修改代码，与审查者讨论，达成一致后再合并。

### Q4: 如何提高代码审查效率？
A: 保持 PR 小而专注，提供清晰的描述和上下文，使用 AI 辅助预审。

### Q5: 遇到复杂 Bug 无法解决怎么办？
A: 使用 AI 辅助分析，向团队求助，必要时进行结对编程。

## 相关文档

- [Git Flow 规范](../github/git-flow.md)
- [测试流程](./testing-workflow.md)
- [需求代码映射](./requirement-code-mapping.md)
- [AI 开发工作流](./ai-development-workflow.md)
