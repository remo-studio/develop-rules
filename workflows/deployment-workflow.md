# 部署流程规范

本文档定义自动化部署流程、环境管理和发布策略。

## 环境管理

### 环境类型

#### 1. 开发环境 (Development)
**用途**：开发人员日常开发和调试

**特点**：
- 部署频率：每次 commit 自动部署
- 数据：测试数据，可随时重置
- 配置：调试模式，详细日志
- 访问：开发团队

**域名示例**：`dev.example.com`

#### 2. 测试环境 (Staging/QA)
**用途**：QA 测试和预发布验证

**特点**：
- 部署频率：develop 分支合并后自动部署
- 数据：模拟生产数据
- 配置：接近生产环境
- 访问：开发团队 + QA 团队

**域名示例**：`staging.example.com`

#### 3. 预生产环境 (Pre-production)
**用途**：最终验证，与生产环境完全一致

**特点**：
- 部署频率：release 分支创建后部署
- 数据：生产数据副本（脱敏）
- 配置：与生产完全一致
- 访问：核心团队

**域名示例**：`pre-prod.example.com`

#### 4. 生产环境 (Production)
**用途**：对外提供服务的正式环境

**特点**：
- 部署频率：手动触发或定时发布
- 数据：真实用户数据
- 配置：生产配置，优化性能
- 访问：所有用户

**域名示例**：`app.example.com`, `www.example.com`

### 环境配置管理

#### 配置文件结构
```
config/
├── default.js          # 默认配置
├── development.js      # 开发环境配置
├── staging.js          # 测试环境配置
├── pre-production.js   # 预生产环境配置
└── production.js       # 生产环境配置
```

#### 配置示例
```javascript
// config/production.js
module.exports = {
  server: {
    port: process.env.PORT || 3000,
    host: '0.0.0.0',
  },
  database: {
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
    name: process.env.DB_NAME,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    ssl: true,
    poolSize: 20,
  },
  redis: {
    host: process.env.REDIS_HOST,
    port: process.env.REDIS_PORT,
    password: process.env.REDIS_PASSWORD,
  },
  auth: {
    jwtSecret: process.env.JWT_SECRET,
    jwtExpiry: '7d',
  },
  logging: {
    level: 'info',
    format: 'json',
  },
};
```

#### 环境变量管理
使用 `.env` 文件（不提交到 Git）：

```bash
# .env.example (提交到 Git 作为模板)
# Server
PORT=3000
NODE_ENV=production

# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
DB_USER=myapp_user
DB_PASSWORD=<your-password>

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=<your-password>

# Authentication
JWT_SECRET=<your-secret>

# External Services
AWS_ACCESS_KEY_ID=<your-key>
AWS_SECRET_ACCESS_KEY=<your-secret>
AWS_REGION=us-east-1

# Monitoring
SENTRY_DSN=<your-dsn>
```

## CI/CD 管道

### GitHub Actions 工作流

#### 持续集成 (CI)
```yaml
# .github/workflows/ci.yml
name: Continuous Integration

on:
  push:
    branches: [develop, main]
  pull_request:
    branches: [develop, main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Lint
        run: npm run lint
      
      - name: Type check
        run: npm run type-check
      
      - name: Run tests
        run: npm test -- --coverage
      
      - name: Build
        run: npm run build
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-artifacts
          path: dist/
```

#### 持续部署 (CD)
```yaml
# .github/workflows/cd.yml
name: Continuous Deployment

on:
  push:
    branches:
      - develop  # 自动部署到测试环境
      - main     # 手动部署到生产环境

jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: staging
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
        env:
          NODE_ENV: staging
      
      - name: Deploy to Staging
        run: |
          echo "Deploying to staging environment..."
          # 部署命令（根据实际情况调整）
          npm run deploy:staging
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      
      - name: Run smoke tests
        run: npm run test:smoke
        env:
          API_URL: https://staging.example.com

  deploy-production:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: 
      name: production
      url: https://app.example.com
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
        env:
          NODE_ENV: production
      
      - name: Deploy to Production
        run: |
          echo "Deploying to production environment..."
          npm run deploy:production
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      
      - name: Run smoke tests
        run: npm run test:smoke
        env:
          API_URL: https://app.example.com
      
      - name: Notify deployment
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: '🚀 Production deployment completed!'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

## 部署策略

### 1. 蓝绿部署 (Blue-Green Deployment)

**原理**：维护两套环境（蓝/绿），切换流量

```
┌─────────┐     ┌─────────┐
│  Blue   │     │  Green  │
│(Current)│────▶│  (New)  │
└─────────┘     └─────────┘
     │               │
     └───── LB ──────┘
            │
         Users
```

**流程**：
1. 绿环境部署新版本
2. 运行测试验证
3. 将负载均衡器流量切换到绿环境
4. 监控绿环境运行状态
5. 如有问题，立即切回蓝环境

**优点**：
- 零停机时间
- 快速回滚
- 充分测试

**缺点**：
- 需要双倍资源
- 数据库迁移复杂

### 2. 滚动部署 (Rolling Deployment)

**原理**：逐步替换实例

```
Instance 1: Old → New ✓
Instance 2: Old → New ✓
Instance 3: Old → New ✓
Instance 4: Old → New ✓
```

**流程**：
1. 从负载均衡器移除实例 1
2. 更新实例 1 到新版本
3. 健康检查通过后加回负载均衡器
4. 重复步骤 1-3 直到所有实例更新

**优点**：
- 不需要额外资源
- 渐进式部署，风险可控

**缺点**：
- 部署时间较长
- 回滚复杂

### 3. 金丝雀部署 (Canary Deployment)

**原理**：先部署到小部分实例，逐步扩大

```
Wave 1: 10% traffic → New version
        (Monitor)
Wave 2: 50% traffic → New version
        (Monitor)
Wave 3: 100% traffic → New version
```

**流程**：
1. 部署新版本到 10% 的实例
2. 引导 10% 流量到新版本
3. 监控指标（错误率、响应时间等）
4. 如正常，逐步扩大到 50%、100%
5. 如异常，立即回滚

**优点**：
- 渐进式验证
- 影响范围可控
- 实时用户反馈

**缺点**：
- 部署时间长
- 需要复杂的流量管理

### 4. Feature Toggle

**原理**：通过功能开关控制新功能

```javascript
// 功能开关配置
const featureFlags = {
  newAuthFlow: {
    enabled: process.env.ENABLE_NEW_AUTH === 'true',
    rolloutPercentage: 20, // 20% 用户启用
  },
};

// 代码中使用
if (isFeatureEnabled('newAuthFlow', userId)) {
  // 新的认证流程
  return newAuthFlow(req);
} else {
  // 原有认证流程
  return oldAuthFlow(req);
}
```

**优点**：
- 代码和部署解耦
- 可快速启用/禁用功能
- A/B 测试友好

**缺点**：
- 代码复杂度增加
- 需要清理旧代码

## Docker 容器化部署

### Dockerfile
```dockerfile
# 多阶段构建
FROM node:20-alpine AS builder

WORKDIR /app

# 复制依赖文件
COPY package*.json ./

# 安装依赖
RUN npm ci --only=production

# 复制源代码
COPY . .

# 构建
RUN npm run build

# 生产镜像
FROM node:20-alpine

WORKDIR /app

# 复制构建产物
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package*.json ./

# 创建非 root 用户
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

USER nodejs

EXPOSE 3000

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => r.statusCode === 200 ? process.exit(0) : process.exit(1))"

CMD ["node", "dist/index.js"]
```

### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=postgres
      - REDIS_HOST=redis
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped
    networks:
      - app-network

  postgres:
    image: postgres:14-alpine
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=myapp_user
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myapp_user"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis-data:/data
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - app
    networks:
      - app-network

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

## Kubernetes 部署

### Deployment 配置
```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        version: v1.0.0
    spec:
      containers:
      - name: myapp
        image: myregistry.com/myapp:v1.0.0
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: myapp-config
              key: db-host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: db-password
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Service 配置
```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 3000
    protocol: TCP
```

### Ingress 配置
```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - app.example.com
    secretName: myapp-tls
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
```

## 数据库迁移

### 迁移工具：Flyway/Liquibase/TypeORM

#### TypeORM 迁移示例
```typescript
// migrations/1704067200000-CreateUsersTable.ts
import { MigrationInterface, QueryRunner, Table } from 'typeorm';

export class CreateUsersTable1704067200000 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(
      new Table({
        name: 'users',
        columns: [
          {
            name: 'id',
            type: 'uuid',
            isPrimary: true,
            default: 'gen_random_uuid()',
          },
          {
            name: 'username',
            type: 'varchar',
            length: '50',
            isUnique: true,
            isNullable: false,
          },
          {
            name: 'email',
            type: 'varchar',
            length: '255',
            isUnique: true,
            isNullable: false,
          },
          {
            name: 'password_hash',
            type: 'varchar',
            length: '255',
            isNullable: false,
          },
          {
            name: 'created_at',
            type: 'timestamp',
            default: 'CURRENT_TIMESTAMP',
          },
          {
            name: 'updated_at',
            type: 'timestamp',
            default: 'CURRENT_TIMESTAMP',
          },
        ],
        indices: [
          {
            name: 'idx_users_email',
            columnNames: ['email'],
          },
        ],
      }),
      true
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('users');
  }
}
```

### 迁移最佳实践
1. **向后兼容**：新版本兼容旧版本数据
2. **可回滚**：每个迁移都有 down 方法
3. **小步迁移**：每次迁移只做一件事
4. **测试迁移**：在测试环境充分测试
5. **备份数据**：生产环境迁移前备份

## 回滚策略

### 快速回滚
```bash
# Kubernetes 回滚到上一个版本
kubectl rollout undo deployment/myapp

# Kubernetes 回滚到特定版本
kubectl rollout undo deployment/myapp --to-revision=3

# 查看回滚历史
kubectl rollout history deployment/myapp
```

### 数据库回滚
```bash
# TypeORM 回滚最后一次迁移
npm run typeorm migration:revert

# 回滚到特定版本
npm run typeorm migration:revert -- -t 1704067200000
```

### 回滚清单
- [ ] 确认问题严重程度
- [ ] 通知相关人员
- [ ] 执行回滚操作
- [ ] 验证回滚成功
- [ ] 监控系统指标
- [ ] 更新状态页面
- [ ] 记录回滚原因
- [ ] 计划修复和重新部署

## 健康检查

### 健康检查端点
```typescript
// src/health.ts
import express from 'express';

const router = express.Router();

// 存活检查（Liveness）
router.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok' });
});

// 就绪检查（Readiness）
router.get('/ready', async (req, res) => {
  try {
    // 检查数据库连接
    await db.ping();
    
    // 检查 Redis 连接
    await redis.ping();
    
    res.status(200).json({
      status: 'ready',
      checks: {
        database: 'ok',
        redis: 'ok',
      },
    });
  } catch (error) {
    res.status(503).json({
      status: 'not ready',
      error: error.message,
    });
  }
});

export default router;
```

## 最佳实践

1. **自动化优先**：尽可能自动化部署流程
2. **环境一致性**：所有环境配置应一致
3. **版本控制**：所有配置纳入版本控制
4. **渐进式部署**：使用金丝雀或蓝绿部署
5. **监控告警**：部署后密切监控
6. **快速回滚**：准备好回滚方案
7. **文档完善**：记录部署流程和配置
8. **安全第一**：秘密不提交到代码库

## 常见问题

### Q1: 部署失败如何处理？
A: 立即回滚到上一个稳定版本，分析失败原因，修复后重新部署。

### Q2: 如何实现零停机部署？
A: 使用蓝绿部署或滚动部署，配合健康检查和优雅关闭。

### Q3: 数据库迁移失败怎么办？
A: 如果有备份，恢复备份；如果迁移有 down 方法，执行回滚。

### Q4: 如何保证配置安全？
A: 使用密钥管理服务（AWS Secrets Manager、HashiCorp Vault），不在代码中硬编码。

### Q5: 多区域部署如何实现？
A: 使用 CDN 和全局负载均衡，在多个区域部署应用实例。

## 相关文档

- [测试流程](./testing-workflow.md)
- [运营流程](./operations-workflow.md)
- [Git Release 规范](../github/git-release.md)
- [AI 开发工作流](./ai-development-workflow.md)
