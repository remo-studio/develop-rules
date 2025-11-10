# 设计文档规范

本文档定义如何管理和维护技术设计文档，包括 API 设计、数据库设计和云架构设计。

## 文档目录结构

```
design/
├── api/                    # API 设计文档
│   ├── openapi.yaml       # OpenAPI 规范
│   ├── REQ-YYYY-NNN/      # 按需求组织的 API 设计
│   └── services/          # 按服务组织的 API 设计
├── database/              # 数据库设计文档
│   ├── schema/            # 数据库 Schema
│   ├── migrations/        # 数据库迁移脚本
│   ├── er-diagrams/       # ER 图
│   └── REQ-YYYY-NNN/      # 按需求的数据库设计
├── cloud/                 # 云架构设计
│   ├── architecture/      # 架构图
│   ├── infrastructure/    # 基础设施配置
│   ├── deployment/        # 部署配置
│   └── REQ-YYYY-NNN/      # 按需求的架构设计
└── diagrams/              # 通用图表资源
    ├── sequence/          # 时序图
    ├── flowchart/         # 流程图
    └── component/         # 组件图
```

## API 设计规范

### OpenAPI 规范
所有 REST API 必须使用 OpenAPI 3.0+ 规范定义。

#### 基本模板
```yaml
openapi: 3.0.3
info:
  title: <服务名称> API
  version: 1.0.0
  description: <服务描述>
  contact:
    name: <团队名称>
    email: team@example.com

servers:
  - url: https://api.example.com/v1
    description: 生产环境
  - url: https://api-staging.example.com/v1
    description: 测试环境

tags:
  - name: users
    description: 用户管理
  - name: auth
    description: 认证授权

paths:
  /users:
    get:
      summary: 获取用户列表
      tags: [users]
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'

components:
  schemas:
    User:
      type: object
      required: [id, username, email]
      properties:
        id:
          type: string
          format: uuid
        username:
          type: string
          minLength: 3
          maxLength: 50
        email:
          type: string
          format: email
    
    UserList:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/User'
        pagination:
          $ref: '#/components/schemas/Pagination'
  
  responses:
    BadRequest:
      description: 请求参数错误
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    
    Unauthorized:
      description: 未授权
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
  
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - bearerAuth: []
```

### API 设计原则
1. **RESTful**：遵循 REST 架构风格
2. **版本控制**：URL 中包含版本号（如 `/v1/`）
3. **统一响应格式**：成功和错误响应格式一致
4. **分页**：列表接口必须支持分页
5. **过滤与排序**：提供灵活的查询能力
6. **HATEOAS**（可选）：响应中包含相关资源链接
7. **幂等性**：PUT/DELETE 操作应是幂等的

### API 文档模板
为每个需求创建 API 设计文档：`design/api/REQ-YYYY-NNN-<desc>.md`

```markdown
# API 设计：<功能名称>

**需求编号**：REQ-YYYY-NNN  
**创建日期**：YYYY-MM-DD  
**负责人**：@username  
**版本**：1.0

## 概述
简要描述 API 的用途和范围。

## 端点列表

### 1. 创建用户
**端点**：`POST /v1/users`  
**描述**：创建新用户

**请求头**：
- `Content-Type: application/json`
- `Authorization: Bearer <token>`

**请求体**：
```json
{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "SecurePassword123!"
}
```

**响应**（201 Created）：
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "john_doe",
  "email": "john@example.com",
  "createdAt": "2025-01-10T10:00:00Z"
}
```

**错误响应**（400 Bad Request）：
```json
{
  "error": {
    "code": "INVALID_INPUT",
    "message": "Invalid email format",
    "details": {
      "field": "email",
      "value": "invalid-email"
    }
  }
}
```

## 数据模型
引用相关的 Schema 定义。

## 安全性
- 认证方式：JWT Bearer Token
- 授权规则：<描述权限要求>
- 限流策略：<描述限流规则>

## 性能要求
- 响应时间：P95 < 200ms
- 吞吐量：支持 1000 QPS
- 并发：支持 100 并发连接

## 变更历史
| 日期 | 版本 | 变更内容 | 修改人 |
|------|------|----------|--------|
| YYYY-MM-DD | 1.0 | 初始版本 | @username |
```

### API 文档生成工具
- **Swagger UI**：从 OpenAPI 规范生成交互式文档
- **Redoc**：生成美观的静态文档
- **Postman**：导入 OpenAPI 规范生成测试集合

## 数据库设计规范

### Schema 定义
使用 SQL DDL 或 ORM Schema 定义数据库结构。

#### 示例：PostgreSQL
```sql
-- design/database/schema/users.sql

-- 用户表
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP,
  
  CONSTRAINT chk_username_length CHECK (char_length(username) >= 3),
  CONSTRAINT chk_email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

-- 索引
CREATE INDEX idx_users_email ON users(email) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_created_at ON users(created_at);

-- 注释
COMMENT ON TABLE users IS '用户表';
COMMENT ON COLUMN users.id IS '用户唯一标识';
COMMENT ON COLUMN users.username IS '用户名，3-50个字符';
COMMENT ON COLUMN users.email IS '电子邮件，用于登录和通知';
```

### 数据库设计文档模板
为每个需求创建数据库设计文档：`design/database/REQ-YYYY-NNN-<desc>.md`

```markdown
# 数据库设计：<功能名称>

**需求编号**：REQ-YYYY-NNN  
**创建日期**：YYYY-MM-DD  
**负责人**：@username  
**数据库**：PostgreSQL 14

## 概述
描述数据库设计的目的和范围。

## 表设计

### 表：users
**用途**：存储用户基本信息

| 字段名 | 类型 | 约束 | 默认值 | 说明 |
|--------|------|------|--------|------|
| id | UUID | PK | gen_random_uuid() | 用户ID |
| username | VARCHAR(50) | UNIQUE, NOT NULL | - | 用户名 |
| email | VARCHAR(255) | UNIQUE, NOT NULL | - | 电子邮件 |
| password_hash | VARCHAR(255) | NOT NULL | - | 密码哈希 |
| created_at | TIMESTAMP | NOT NULL | CURRENT_TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | NOT NULL | CURRENT_TIMESTAMP | 更新时间 |
| deleted_at | TIMESTAMP | NULL | NULL | 软删除时间 |

**索引**：
- `idx_users_email`：email 字段，WHERE deleted_at IS NULL
- `idx_users_created_at`：created_at 字段

**约束**：
- `chk_username_length`：username 长度 >= 3
- `chk_email_format`：email 格式验证

## ER 图
![ER Diagram](../diagrams/REQ-2025-001-er-diagram.png)

## 关系说明
描述表之间的关系（一对一、一对多、多对多）。

## 数据迁移
- 迁移脚本：`migrations/2025-01-10_create_users_table.sql`
- 回滚脚本：`migrations/2025-01-10_create_users_table_rollback.sql`

## 性能考虑
- 预估数据量：10M 行
- 查询模式：主要按 email 查询
- 优化策略：在 email 字段建立索引

## 数据安全
- 敏感字段：password_hash
- 加密方式：bcrypt（cost=12）
- 备份策略：每日全量备份，每小时增量备份

## 变更历史
| 日期 | 版本 | 变更内容 | 修改人 |
|------|------|----------|--------|
| YYYY-MM-DD | 1.0 | 初始版本 | @username |
```

### 数据库设计工具
- **dbdiagram.io**：在线 ER 图工具
- **draw.io**：通用图表工具
- **DBeaver**：数据库管理和 ER 图生成
- **Liquibase/Flyway**：数据库版本管理

## 云架构设计规范

### 架构图
使用标准的云服务图标绘制架构图。

#### 架构图模板
```
┌─────────────────────────────────────────────────────────┐
│                      用户/客户端                         │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                   CDN / API Gateway                      │
└────────────────────┬────────────────────────────────────┘
                     │
         ┌───────────┼───────────┐
         ▼           ▼           ▼
┌────────────┐ ┌────────────┐ ┌────────────┐
│ Web Server │ │ API Server │ │ Auth Server│
│  (Nginx)   │ │  (Node.js) │ │  (OAuth)   │
└─────┬──────┘ └─────┬──────┘ └─────┬──────┘
      │              │              │
      └──────────────┼──────────────┘
                     ▼
┌─────────────────────────────────────────────────────────┐
│                    Database Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  PostgreSQL  │  │    Redis     │  │  S3 Storage  │  │
│  │  (Primary)   │  │   (Cache)    │  │   (Files)    │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### 云架构设计文档模板
为每个需求创建架构设计文档：`design/cloud/REQ-YYYY-NNN-<desc>.md`

```markdown
# 云架构设计：<功能名称>

**需求编号**：REQ-YYYY-NNN  
**创建日期**：YYYY-MM-DD  
**负责人**：@username  
**云平台**：AWS / Azure / GCP

## 概述
描述系统的整体架构和技术栈。

## 系统架构图
![Architecture Diagram](../diagrams/REQ-2025-001-architecture.png)

## 组件说明

### 前端层
- **技术栈**：React 18 + TypeScript
- **部署方式**：CDN (CloudFront)
- **域名**：app.example.com

### API 层
- **技术栈**：Node.js 20 + Express
- **部署方式**：ECS Fargate
- **实例规格**：2 vCPU, 4GB RAM
- **自动扩展**：2-10 实例
- **负载均衡**：ALB

### 数据层
- **主数据库**：RDS PostgreSQL 14
  - 实例类型：db.t3.medium
  - 存储：100GB SSD
  - 多可用区部署：是
  - 自动备份：每日
- **缓存**：ElastiCache Redis 7
  - 节点类型：cache.t3.micro
  - 副本数：2
- **对象存储**：S3
  - 存储类：Standard
  - 生命周期策略：30 天后转 IA

## 网络设计
- **VPC**：10.0.0.0/16
- **子网**：
  - Public Subnet A: 10.0.1.0/24
  - Public Subnet B: 10.0.2.0/24
  - Private Subnet A: 10.0.10.0/24
  - Private Subnet B: 10.0.20.0/24
- **安全组**：
  - Web SG: 允许 80/443
  - App SG: 允许 8080 (仅来自 Web SG)
  - DB SG: 允许 5432 (仅来自 App SG)

## 基础设施即代码
```terraform
# design/cloud/infrastructure/main.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

# VPC
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = "main-vpc"
    Requirement = "REQ-2025-001"
  }
}

# ... 其他资源定义
```

## 部署流程
1. 构建 Docker 镜像
2. 推送到 ECR
3. 更新 ECS 任务定义
4. 滚动更新 ECS 服务
5. 健康检查
6. 流量切换

## 监控与告警
- **监控工具**：CloudWatch / Datadog
- **关键指标**：
  - API 响应时间（P95）
  - 错误率
  - CPU/内存使用率
  - 数据库连接数
- **告警规则**：
  - API P95 > 500ms
  - 错误率 > 1%
  - CPU > 80%

## 成本估算
| 服务 | 规格 | 数量 | 月成本（USD） |
|------|------|------|---------------|
| ECS Fargate | 2vCPU, 4GB | 2-10 | $150-750 |
| RDS PostgreSQL | db.t3.medium | 1 | $80 |
| ElastiCache | cache.t3.micro | 2 | $30 |
| S3 | Standard | 100GB | $2.5 |
| CloudFront | - | - | $50 |
| **总计** | | | **$312.5-912.5** |

## 安全考虑
- TLS 1.3 加密传输
- IAM 最小权限原则
- 数据库访问限制在私有子网
- 定期安全审计
- 秘密管理：AWS Secrets Manager

## 灾难恢复
- **RTO**：2 小时
- **RPO**：15 分钟
- **备份策略**：
  - 数据库：每日全量 + 每小时增量
  - 代码：Git 仓库
  - 配置：Terraform 状态文件

## 变更历史
| 日期 | 版本 | 变更内容 | 修改人 |
|------|------|----------|--------|
| YYYY-MM-DD | 1.0 | 初始版本 | @username |
```

### IaC 工具
- **Terraform**：多云基础设施管理
- **CloudFormation**：AWS 原生
- **Pulumi**：使用编程语言定义基础设施
- **Ansible**：配置管理

## 设计审查流程

### 审查清单
- [ ] API 设计符合 RESTful 规范
- [ ] 数据库设计满足范式要求
- [ ] 架构满足性能要求
- [ ] 安全性考虑充分
- [ ] 可扩展性设计合理
- [ ] 成本在预算范围内
- [ ] 灾难恢复方案完整
- [ ] 文档完整清晰

### 审查会议
1. **参与人员**：架构师、开发负责人、运维负责人、产品经理
2. **会议时长**：1-2 小时
3. **输出**：审查意见和修改建议
4. **批准标准**：所有关键人员签字同意

### 文档版本管理
所有设计文档纳入 Git 版本管理：
```bash
git add design/
git commit -m "docs(design): add design for REQ-2025-001"
git push
```

## AI 辅助设计

### 使用 AI 生成设计文档
```
Prompt 示例：

请为以下需求生成 API 设计文档：

需求：用户认证功能（REQ-2025-001）
功能：支持用户注册、登录、登出、密码重置

请包括：
1. RESTful API 端点定义
2. 请求/响应格式
3. 错误处理
4. 认证方式（JWT）
5. OpenAPI 规范

格式：Markdown
```

### AI 辅助架构设计
- **ChatGPT/Claude**：生成架构建议
- **GitHub Copilot**：补全配置文件
- **AI 画图工具**：生成架构图草稿

## 最佳实践

1. **设计先行**：编码前完成设计文档
2. **版本控制**：所有设计文档纳入 Git
3. **及时更新**：实现与设计不符时及时更新文档
4. **关联需求**：设计文档必须关联需求编号
5. **可视化**：使用图表增强理解
6. **标准化**：遵循团队统一的设计规范
7. **审查机制**：重要设计必须经过审查
8. **文档模板**：使用模板提高一致性

## 相关文档

- [需求管理规范](./requirements-management.md)
- [开发计划管理](./development-planning.md)
- [代码开发流程](./code-development.md)
- [AI 开发工作流](./ai-development-workflow.md)
