# PRD: 用户认证功能

**需求编号**：REQ-2025-001  
**创建日期**：2025-01-10  
**负责人**：@product-team  
**优先级**：P0  
**状态**：Approved

## 1. 背景与目标

### 1.1 业务背景
当前系统缺少用户认证机制，无法区分用户身份和权限。为了支持个性化服务和数据安全，需要实现完整的用户认证功能。

### 1.2 目标
- 支持用户注册和登录
- 支持多种登录方式（邮箱密码、OAuth）
- 提供安全的会话管理
- 实现密码找回功能

**预期收益**：
- 用户留存率提升 20%
- 减少匿名滥用行为
- 支持后续个性化功能开发

## 2. 需求描述

### 2.1 功能需求

#### 用户注册
- 支持邮箱 + 密码注册
- 邮箱验证流程
- 密码强度要求：至少 8 位，包含大小写字母和数字
- 重复注册检测

#### 用户登录
- 邮箱 + 密码登录
- OAuth 第三方登录（Google、GitHub）
- 记住登录状态（7 天）
- 登录失败次数限制（5 次锁定 15 分钟）

#### 密码管理
- 密码找回（通过邮箱）
- 密码修改
- 密码强度提示

#### 会话管理
- JWT Token 认证
- Token 刷新机制
- 登出功能
- 多设备登录支持

### 2.2 非功能需求

#### 性能要求
- 登录响应时间：P95 < 500ms
- 注册响应时间：P95 < 1s
- 支持并发用户：10,000 QPS

#### 安全要求
- 密码使用 bcrypt 加密存储（cost=12）
- JWT Token 使用 RS256 签名
- HTTPS 强制传输
- 防止暴力破解（登录限流）
- XSS 和 CSRF 防护

#### 可用性要求
- 系统可用性：99.9%
- 错误提示友好清晰
- 支持国际化（中文、英文）

#### 兼容性要求
- 支持主流浏览器（Chrome、Firefox、Safari、Edge）
- 移动端响应式设计
- API 兼容 REST 规范

### 2.3 约束条件
- 开发周期：2 周
- 必须使用现有技术栈（Node.js + PostgreSQL）
- 遵循 GDPR 数据隐私要求

## 3. 用户故事

### 故事 1：用户注册
**作为** 新用户，**我希望** 能够快速注册账号，**以便** 使用系统的个性化功能。

**验收标准**：
- [x] 用户可以通过邮箱和密码注册
- [x] 注册后收到验证邮件
- [x] 点击验证链接后账号激活
- [x] 密码不符合要求时显示明确提示
- [x] 重复邮箱注册时显示友好错误

### 故事 2：用户登录
**作为** 已注册用户，**我希望** 能够安全便捷地登录，**以便** 访问我的个人数据。

**验收标准**：
- [x] 用户可以使用邮箱和密码登录
- [x] 用户可以选择"记住我"保持 7 天登录
- [x] 密码错误时显示明确提示
- [x] 登录失败 5 次后账号锁定 15 分钟
- [x] 用户可以使用 Google 账号登录
- [x] 用户可以使用 GitHub 账号登录

### 故事 3：密码找回
**作为** 忘记密码的用户，**我希望** 能够通过邮箱找回密码，**以便** 重新登录系统。

**验收标准**：
- [x] 用户可以请求密码重置邮件
- [x] 重置链接 1 小时内有效
- [x] 用户可以设置新密码
- [x] 新密码符合强度要求
- [x] 重置成功后旧密码失效

## 4. 技术方案（概要）

### 4.1 系统架构
```
┌─────────┐     ┌──────────┐     ┌──────────┐
│  Client │────▶│   API    │────▶│ Database │
│         │◀────│  Server  │◀────│(Postgres)│
└─────────┘     └──────────┘     └──────────┘
                      │
                      ▼
                ┌──────────┐
                │  Redis   │
                │ (Cache)  │
                └──────────┘
```

### 4.2 接口设计

#### 注册接口
```
POST /api/v1/auth/register
Content-Type: application/json

{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "SecurePass123!"
}

Response (201):
{
  "id": "uuid",
  "username": "john_doe",
  "email": "john@example.com",
  "message": "Please check your email to verify your account"
}
```

#### 登录接口
```
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "john@example.com",
  "password": "SecurePass123!",
  "rememberMe": true
}

Response (200):
{
  "token": "eyJhbGciOiJSUzI1NiIs...",
  "expiresIn": "7d",
  "user": {
    "id": "uuid",
    "username": "john_doe",
    "email": "john@example.com"
  }
}
```

#### OAuth 登录
```
GET /api/v1/auth/oauth/google
-> 重定向到 Google OAuth

Callback:
GET /api/v1/auth/oauth/google/callback?code=xxx

Response (302):
-> 重定向到前端，携带 token
```

### 4.3 数据模型

#### users 表
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255),
  email_verified BOOLEAN DEFAULT FALSE,
  oauth_provider VARCHAR(50),
  oauth_id VARCHAR(255),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_login_at TIMESTAMP,
  login_attempts INT DEFAULT 0,
  locked_until TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_oauth ON users(oauth_provider, oauth_id);
```

#### verification_tokens 表
```sql
CREATE TABLE verification_tokens (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  token VARCHAR(255) UNIQUE NOT NULL,
  type VARCHAR(50) NOT NULL, -- 'email_verify', 'password_reset'
  expires_at TIMESTAMP NOT NULL,
  used_at TIMESTAMP
);

CREATE INDEX idx_tokens_user ON verification_tokens(user_id);
CREATE INDEX idx_tokens_token ON verification_tokens(token);
```

## 5. 依赖关系

### 5.1 依赖需求
- 无外部需求依赖

### 5.2 被依赖需求
- REQ-2025-002：用户权限管理（依赖本需求的用户系统）
- REQ-2025-003：用户个人资料（依赖本需求的用户系统）

### 5.3 技术依赖
- Node.js 20+
- PostgreSQL 14+
- Redis 7+
- bcrypt 库（密码加密）
- jsonwebtoken 库（JWT 生成）
- Passport.js（OAuth 集成）

## 6. 时间计划

| 阶段 | 开始时间 | 结束时间 | 负责人 | 状态 |
|------|----------|----------|--------|------|
| 需求评审 | 2025-01-10 | 2025-01-10 | @product-team | ✅ 完成 |
| API 设计 | 2025-01-11 | 2025-01-12 | @backend-team | ✅ 完成 |
| 数据库设计 | 2025-01-11 | 2025-01-12 | @backend-team | ✅ 完成 |
| 后端开发 | 2025-01-13 | 2025-01-17 | @backend-team | 🔄 进行中 |
| 前端开发 | 2025-01-13 | 2025-01-17 | @frontend-team | 🔄 进行中 |
| 测试 | 2025-01-18 | 2025-01-19 | @qa-team | ⏳ 待开始 |
| 部署 | 2025-01-20 | 2025-01-20 | @devops-team | ⏳ 待开始 |
| 上线 | 2025-01-21 | 2025-01-21 | @all-teams | ⏳ 待开始 |

## 7. 风险评估

| 风险 | 概率 | 影响 | 应对措施 |
|------|------|------|----------|
| OAuth 第三方服务不稳定 | 中 | 中 | 添加重试机制和降级方案（禁用 OAuth） |
| 密码重置邮件被拦截 | 低 | 高 | 配置 SPF/DKIM，使用可靠邮件服务商 |
| 性能达不到要求 | 低 | 中 | 提前进行负载测试，优化数据库查询 |
| 安全漏洞 | 低 | 高 | 进行安全审计，使用成熟的安全库 |
| 开发进度延迟 | 中 | 中 | 每日站会跟踪进度，及时调整资源 |

## 8. 附录

### 8.1 相关链接
- Issue: #42 - 实现用户认证功能
- 设计文档: [design/api/REQ-2025-001-auth-api.md](../design/api/REQ-2025-001-auth-api.md)
- 数据库设计: [design/database/REQ-2025-001-users-schema.md](../design/database/REQ-2025-001-users-schema.md)
- 原型: [Figma 链接](https://figma.com/...)

### 8.2 参考资料
- OAuth 2.0 RFC: https://tools.ietf.org/html/rfc6749
- JWT Best Practices: https://tools.ietf.org/html/rfc8725
- OWASP Authentication Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html

### 8.3 变更记录

| 日期 | 版本 | 变更内容 | 修改人 |
|------|------|----------|--------|
| 2025-01-10 | 1.0 | 初始版本 | @product-team |
| 2025-01-12 | 1.1 | 添加 OAuth 登录支持 | @product-team |
| 2025-01-15 | 1.2 | 更新时间计划 | @product-team |

---

**审批记录**：
- 产品负责人：✅ 已批准 (@product-lead, 2025-01-10)
- 技术负责人：✅ 已批准 (@tech-lead, 2025-01-10)
- 安全负责人：✅ 已批准 (@security-lead, 2025-01-10)
