# 测试流程规范

本文档定义完整的测试策略和流程，包括 AI 辅助的测试用例生成和自动化测试。

## 测试金字塔

```
         /\
        /  \  E2E Tests (10%)
       /____\
      /      \
     / Integ  \ Integration Tests (20%)
    /  -ration\
   /__________\
  /            \
 /    Unit      \ Unit Tests (70%)
/________________\
```

### 测试类型分布
- **单元测试（70%）**：测试单个函数/类
- **集成测试（20%）**：测试模块间交互
- **端到端测试（10%）**：测试完整用户流程

## 单元测试

### 测试原则
1. **FIRST 原则**
   - **Fast**：快速执行
   - **Independent**：测试间独立
   - **Repeatable**：可重复执行
   - **Self-Validating**：自动验证
   - **Timely**：及时编写

2. **AAA 模式**
   - **Arrange**：准备测试数据
   - **Act**：执行被测代码
   - **Assert**：验证结果

### JavaScript/TypeScript 示例

#### Jest 测试
```typescript
// src/auth/authenticate.test.ts
import { authenticateUser } from './authenticate';
import { UserRepository } from './user-repository';
import { TokenService } from './token-service';

// Mock 依赖
jest.mock('./user-repository');
jest.mock('./token-service');

describe('authenticateUser', () => {
  let userRepo: jest.Mocked<UserRepository>;
  let tokenService: jest.Mocked<TokenService>;

  beforeEach(() => {
    // Arrange: 准备测试环境
    userRepo = new UserRepository() as jest.Mocked<UserRepository>;
    tokenService = new TokenService() as jest.Mocked<TokenService>;
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  it('should return token when credentials are valid', async () => {
    // Arrange
    const username = 'john_doe';
    const password = 'SecurePass123!';
    const mockUser = {
      id: '123',
      username,
      passwordHash: '$2b$10$...',
    };
    const mockToken = 'jwt.token.here';

    userRepo.findByUsername.mockResolvedValue(mockUser);
    tokenService.generate.mockReturnValue(mockToken);

    // Act
    const result = await authenticateUser(username, password);

    // Assert
    expect(result).toEqual({ token: mockToken });
    expect(userRepo.findByUsername).toHaveBeenCalledWith(username);
    expect(tokenService.generate).toHaveBeenCalledWith(mockUser);
  });

  it('should throw error when user not found', async () => {
    // Arrange
    userRepo.findByUsername.mockResolvedValue(null);

    // Act & Assert
    await expect(
      authenticateUser('nonexistent', 'password')
    ).rejects.toThrow('User not found');
  });

  it('should throw error when password is invalid', async () => {
    // Arrange
    const mockUser = {
      id: '123',
      username: 'john_doe',
      passwordHash: '$2b$10$...',
    };
    userRepo.findByUsername.mockResolvedValue(mockUser);

    // Act & Assert
    await expect(
      authenticateUser('john_doe', 'WrongPassword')
    ).rejects.toThrow('Invalid password');
  });

  describe('edge cases', () => {
    it('should handle empty username', async () => {
      await expect(
        authenticateUser('', 'password')
      ).rejects.toThrow('Username is required');
    });

    it('should handle empty password', async () => {
      await expect(
        authenticateUser('username', '')
      ).rejects.toThrow('Password is required');
    });
  });
});
```

### Python 示例

#### pytest 测试
```python
# tests/test_authenticate.py
import pytest
from unittest.mock import Mock, patch
from auth.authenticate import authenticate_user
from auth.exceptions import UserNotFoundError, InvalidPasswordError

class TestAuthenticateUser:
    """测试用户认证功能"""

    @pytest.fixture
    def mock_user_repo(self):
        """模拟用户仓库"""
        return Mock()

    @pytest.fixture
    def mock_token_service(self):
        """模拟 Token 服务"""
        return Mock()

    def test_authenticate_with_valid_credentials(
        self, mock_user_repo, mock_token_service
    ):
        """测试有效凭据认证"""
        # Arrange
        username = 'john_doe'
        password = 'SecurePass123!'
        mock_user = {
            'id': '123',
            'username': username,
            'password_hash': '$2b$10$...'
        }
        mock_token = 'jwt.token.here'
        
        mock_user_repo.find_by_username.return_value = mock_user
        mock_token_service.generate.return_value = mock_token

        # Act
        result = authenticate_user(
            username, password, mock_user_repo, mock_token_service
        )

        # Assert
        assert result['token'] == mock_token
        mock_user_repo.find_by_username.assert_called_once_with(username)
        mock_token_service.generate.assert_called_once_with(mock_user)

    def test_authenticate_with_nonexistent_user(self, mock_user_repo):
        """测试用户不存在"""
        # Arrange
        mock_user_repo.find_by_username.return_value = None

        # Act & Assert
        with pytest.raises(UserNotFoundError):
            authenticate_user('nonexistent', 'password', mock_user_repo, None)

    def test_authenticate_with_invalid_password(self, mock_user_repo):
        """测试密码错误"""
        # Arrange
        mock_user = {
            'id': '123',
            'username': 'john_doe',
            'password_hash': '$2b$10$...'
        }
        mock_user_repo.find_by_username.return_value = mock_user

        # Act & Assert
        with pytest.raises(InvalidPasswordError):
            authenticate_user('john_doe', 'WrongPassword', mock_user_repo, None)

    @pytest.mark.parametrize('username,password,expected_error', [
        ('', 'password', 'Username is required'),
        ('username', '', 'Password is required'),
        (None, 'password', 'Username is required'),
        ('username', None, 'Password is required'),
    ])
    def test_authenticate_with_invalid_input(
        self, username, password, expected_error
    ):
        """测试无效输入"""
        with pytest.raises(ValueError, match=expected_error):
            authenticate_user(username, password, None, None)
```

## AI 辅助测试用例生成

### 使用 GitHub Copilot

#### 1. 注释驱动生成
```typescript
// 测试用户认证函数
// 场景：成功登录、用户不存在、密码错误、空参数、SQL注入
describe('authenticateUser', () => {
  // Copilot 会自动生成多个测试用例
});
```

#### 2. 从实现生成测试
```typescript
// 1. 选中函数实现
// 2. 打开 Copilot Chat
// 3. 输入：@workspace /tests 为这个函数生成完整的测试套件
```

### 使用 ChatGPT/Claude

#### 提示词模板
```
请为以下函数生成完整的测试用例：

函数代码：
[粘贴代码]

要求：
1. 使用 [测试框架] (如 Jest, pytest)
2. 包含正常场景、异常场景、边界条件
3. 使用 AAA 模式
4. Mock 外部依赖
5. 测试覆盖率 > 90%

请生成：
1. 测试代码
2. 测试用例说明
3. Mock 数据准备
```

### AI 生成测试的审查要点
- ✅ 测试场景是否完整
- ✅ 边界条件是否考虑
- ✅ Mock 是否正确
- ✅ 断言是否充分
- ✅ 测试是否独立

## 集成测试

### API 集成测试

#### Supertest 示例 (Node.js)
```typescript
// tests/integration/auth.test.ts
import request from 'supertest';
import { app } from '../../src/app';
import { setupTestDatabase, teardownTestDatabase } from '../helpers/database';

describe('Authentication API', () => {
  beforeAll(async () => {
    await setupTestDatabase();
  });

  afterAll(async () => {
    await teardownTestDatabase();
  });

  describe('POST /api/v1/auth/register', () => {
    it('should register a new user', async () => {
      const response = await request(app)
        .post('/api/v1/auth/register')
        .send({
          username: 'newuser',
          email: 'newuser@example.com',
          password: 'SecurePass123!',
        })
        .expect(201);

      expect(response.body).toMatchObject({
        id: expect.any(String),
        username: 'newuser',
        email: 'newuser@example.com',
      });
      expect(response.body.password).toBeUndefined();
    });

    it('should reject duplicate username', async () => {
      // First registration
      await request(app)
        .post('/api/v1/auth/register')
        .send({
          username: 'duplicate',
          email: 'user1@example.com',
          password: 'SecurePass123!',
        });

      // Duplicate registration
      const response = await request(app)
        .post('/api/v1/auth/register')
        .send({
          username: 'duplicate',
          email: 'user2@example.com',
          password: 'SecurePass123!',
        })
        .expect(400);

      expect(response.body.error.code).toBe('DUPLICATE_USERNAME');
    });
  });

  describe('POST /api/v1/auth/login', () => {
    beforeEach(async () => {
      // 创建测试用户
      await request(app)
        .post('/api/v1/auth/register')
        .send({
          username: 'testuser',
          email: 'test@example.com',
          password: 'TestPass123!',
        });
    });

    it('should login with valid credentials', async () => {
      const response = await request(app)
        .post('/api/v1/auth/login')
        .send({
          username: 'testuser',
          password: 'TestPass123!',
        })
        .expect(200);

      expect(response.body).toHaveProperty('token');
      expect(response.body.token).toMatch(/^[A-Za-z0-9-_]+\.[A-Za-z0-9-_]+\.[A-Za-z0-9-_]+$/);
    });

    it('should reject invalid password', async () => {
      await request(app)
        .post('/api/v1/auth/login')
        .send({
          username: 'testuser',
          password: 'WrongPassword',
        })
        .expect(401);
    });
  });
});
```

### 数据库集成测试

#### 测试数据库设置
```typescript
// tests/helpers/database.ts
import { Client } from 'pg';

let testClient: Client;

export async function setupTestDatabase() {
  testClient = new Client({
    connectionString: process.env.TEST_DATABASE_URL,
  });
  await testClient.connect();
  
  // 运行迁移
  await runMigrations(testClient);
  
  // 插入测试数据
  await seedTestData(testClient);
}

export async function teardownTestDatabase() {
  // 清理所有表
  await testClient.query('TRUNCATE TABLE users, sessions CASCADE');
  await testClient.end();
}

export async function clearTable(tableName: string) {
  await testClient.query(`TRUNCATE TABLE ${tableName} CASCADE`);
}
```

## 端到端测试

### Playwright 示例
```typescript
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('User Authentication', () => {
  test('user can register and login', async ({ page }) => {
    // 访问注册页面
    await page.goto('/register');

    // 填写注册表单
    await page.fill('[data-testid="username"]', 'e2euser');
    await page.fill('[data-testid="email"]', 'e2e@example.com');
    await page.fill('[data-testid="password"]', 'SecurePass123!');
    await page.fill('[data-testid="confirm-password"]', 'SecurePass123!');

    // 提交表单
    await page.click('[data-testid="submit"]');

    // 验证跳转到首页
    await expect(page).toHaveURL('/');
    await expect(page.locator('[data-testid="welcome"]')).toContainText('e2euser');

    // 登出
    await page.click('[data-testid="logout"]');
    await expect(page).toHaveURL('/login');

    // 重新登录
    await page.fill('[data-testid="username"]', 'e2euser');
    await page.fill('[data-testid="password"]', 'SecurePass123!');
    await page.click('[data-testid="login"]');

    // 验证登录成功
    await expect(page).toHaveURL('/');
    await expect(page.locator('[data-testid="welcome"]')).toContainText('e2euser');
  });

  test('login fails with invalid credentials', async ({ page }) => {
    await page.goto('/login');

    await page.fill('[data-testid="username"]', 'nonexistent');
    await page.fill('[data-testid="password"]', 'WrongPassword');
    await page.click('[data-testid="login"]');

    // 验证错误消息
    await expect(page.locator('[data-testid="error"]')).toContainText(
      'Invalid username or password'
    );
  });
});
```

## 测试覆盖率

### 配置覆盖率报告

#### Jest 配置
```javascript
// jest.config.js
module.exports = {
  collectCoverage: true,
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  coverageThresholds: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  collectCoverageFrom: [
    'src/**/*.{js,ts}',
    '!src/**/*.test.{js,ts}',
    '!src/**/*.spec.{js,ts}',
    '!src/**/index.{js,ts}',
  ],
};
```

#### pytest 配置
```ini
# pytest.ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*

addopts = 
    --cov=src
    --cov-report=html
    --cov-report=term-missing
    --cov-fail-under=80
```

### 查看覆盖率报告
```bash
# JavaScript/TypeScript
npm test -- --coverage
open coverage/index.html

# Python
pytest --cov=src --cov-report=html
open htmlcov/index.html
```

## 性能测试

### 负载测试 (k6)
```javascript
// performance/auth-load.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '1m', target: 10 },   // 预热
    { duration: '3m', target: 100 },  // 增压
    { duration: '5m', target: 100 },  // 稳定负载
    { duration: '1m', target: 0 },    // 降压
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% 请求 < 500ms
    http_req_failed: ['rate<0.01'],    // 错误率 < 1%
  },
};

export default function () {
  const payload = JSON.stringify({
    username: 'testuser',
    password: 'TestPass123!',
  });

  const params = {
    headers: {
      'Content-Type': 'application/json',
    },
  };

  const res = http.post('http://localhost:3000/api/v1/auth/login', payload, params);

  check(res, {
    'status is 200': (r) => r.status === 200,
    'has token': (r) => r.json('token') !== undefined,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });

  sleep(1);
}
```

运行负载测试：
```bash
k6 run performance/auth-load.js
```

## 自动化测试流程

### CI/CD 集成

#### GitHub Actions 工作流
```yaml
# .github/workflows/test.yml
name: Test

on:
  push:
    branches: [develop, main]
  pull_request:
    branches: [develop, main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm run test:unit -- --coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  integration-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test

  e2e-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      - name: Upload test results
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-results
          path: test-results/
```

## 测试数据管理

### 测试固件 (Fixtures)
```typescript
// tests/fixtures/users.ts
export const validUser = {
  username: 'testuser',
  email: 'test@example.com',
  password: 'SecurePass123!',
};

export const adminUser = {
  username: 'admin',
  email: 'admin@example.com',
  password: 'AdminPass123!',
  role: 'admin',
};

export const invalidUsers = [
  { username: '', email: 'test@example.com', password: 'pass' },
  { username: 'test', email: 'invalid-email', password: 'pass' },
  { username: 'test', email: 'test@example.com', password: '123' },
];
```

### 测试数据工厂
```typescript
// tests/factories/user.factory.ts
import { faker } from '@faker-js/faker';

export function createUser(overrides = {}) {
  return {
    id: faker.string.uuid(),
    username: faker.internet.userName(),
    email: faker.internet.email(),
    passwordHash: faker.string.alphanumeric(60),
    createdAt: faker.date.past(),
    ...overrides,
  };
}

export function createManyUsers(count: number, overrides = {}) {
  return Array.from({ length: count }, () => createUser(overrides));
}
```

## 最佳实践

1. **测试先行（TDD）**
   - 先写测试，再写实现
   - 保持红-绿-重构循环

2. **保持测试独立**
   - 测试间不共享状态
   - 每个测试后清理数据

3. **使用有意义的测试名称**
   - 描述测试场景，而非实现
   - 格式：`should [expected behavior] when [condition]`

4. **Mock 外部依赖**
   - 数据库、API、文件系统
   - 保持测试快速和稳定

5. **测试边界条件**
   - 空值、null、undefined
   - 最大值、最小值
   - 异常情况

6. **维护测试代码质量**
   - 测试代码和生产代码同样重要
   - 重构测试，消除重复

7. **CI 自动化**
   - PR 必须通过所有测试
   - 定期运行完整测试套件

8. **监控测试执行时间**
   - 单元测试应该很快（< 1s）
   - 慢测试应优化或移到集成测试

## 常见问题

### Q1: 测试覆盖率要达到 100% 吗？
A: 不必要。80% 以上即可，重点是覆盖核心逻辑和边界条件。

### Q2: 如何测试私有方法？
A: 不应该直接测试私有方法。通过测试公共接口间接测试私有方法。

### Q3: Mock 还是真实依赖？
A: 单元测试用 Mock，集成测试用真实依赖，E2E 测试用生产环境配置。

### Q4: AI 生成的测试可靠吗？
A: 需要人工审查。AI 可能遗漏边界条件或生成不必要的测试。

### Q5: 测试失败时如何调试？
A: 使用 `.only` 隔离失败测试，使用调试器单步执行，查看详细错误信息。

## 相关文档

- [代码开发流程](./code-development.md)
- [部署流程](./deployment-workflow.md)
- [需求代码映射](./requirement-code-mapping.md)
- [AI 开发工作流](./ai-development-workflow.md)
