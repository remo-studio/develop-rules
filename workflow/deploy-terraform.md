---
description: 如何部署 Good7ob 后端到 AWS Lambda + API Gateway
---

# Good7ob Backend 部署到 AWS

本 workflow 描述如何使用 Terraform 将 Good7ob 后端应用部署到 AWS Lambda + API Gateway。

## 前置条件

1. **AWS 凭证已配置**
   ```bash
   aws configure
   # 输入 Access Key ID, Secret Access Key, Region
   ```

2. **工具已安装**
   - Terraform >= 1.5.0
   - AWS CLI >= 2.0
   - Maven >= 3.8
   - Java 21

3. **代码已编译**
   ```bash
   cd backend
   mvn clean package -Dmaven.test.skip=true
   ```

## 部署步骤

### 1. 配置 Terraform 变量

```bash
cd terraform
cp terraform.tfvars.example terraform.tfvars
```

编辑 `terraform.tfvars`，至少配置以下内容：

```hcl
project_name = "good7ob"
environment  = "dev"
aws_region   = "ap-northeast-1"

# Lambda 配置
lambda_memory_mb = 1024  # 推荐 1GB+
lambda_timeout   = 30

# API Gateway CORS
api_cors_origins = [
  "https://your-frontend-domain.com",
  "http://localhost:5173"
]

# 数据库配置
rds_instance_class = "db.t3.medium"
rds_database_name  = "good7ob"
rds_username       = "admin"
```

### 2. 初始化 Terraform

```bash
terraform init
```

### 3. 预览部署计划

```bash
terraform plan
```

检查将要创建的资源。

### 4. 执行部署

// turbo
```bash
terraform apply
```

输入 `yes` 确认。等待 10-15 分钟。

### 5. 部署应用代码

```bash
cd ..
chmod +x scripts/deploy-lambda.sh
```

// turbo
```bash
./scripts/deploy-lambda.sh
```

### 6. 获取部署信息

```bash
cd terraform
terraform output
```

记录以下信息：
- `api_gateway_endpoint`: API 入口地址
- `lambda_function_name`: Lambda 函数名

### 7. 配置数据库

获取数据库密码：
```bash
aws secretsmanager get-secret-value \
  --secret-id good7ob-db-credentials \
  --region ap-northeast-1 \
  --query SecretString \
  --output text | jq -r .password
```

连接数据库并运行迁移脚本：
```bash
# 获取 RDS 端点
RDS_ENDPOINT=$(aws rds describe-db-instances \
  --db-instance-identifier good7ob-mysql \
  --query 'DBInstances[0].Endpoint.Address' \
  --output text \
  --region ap-northeast-1)

# 连接数据库
mysql -h $RDS_ENDPOINT -u admin -p good7ob < database/schema.sql
```

### 8. 测试 API

```bash
API_ENDPOINT=$(terraform -chdir=terraform output -raw api_gateway_endpoint)

# 测试健康检查
curl $API_ENDPOINT/health

# 测试用户 API
curl $API_ENDPOINT/user/info -H "userId: 1"
```

## 更新部署

### 更新 Lambda 代码

```bash
./scripts/deploy-lambda.sh
```

### 更新基础设施

```bash
cd terraform
terraform apply
```

### 更新环境变量

编辑 `terraform/api_lambda.tf` 中的 `environment.variables`，然后：

```bash
terraform apply
```

## 监控和日志

### 查看 Lambda 日志

```bash
aws logs tail /aws/lambda/good7ob-api --follow --region ap-northeast-1
```

### 查看 API Gateway 日志

```bash
aws logs tail /aws/apigateway/good7ob-http-api --follow --region ap-northeast-1
```

### CloudWatch Metrics

访问 AWS Console > CloudWatch > Metrics 查看：
- Lambda 调用次数、错误率、持续时间
- API Gateway 请求数、4xx/5xx 错误
- RDS CPU、内存、连接数
- Redis CPU、内存使用率

## 故障排查

### Lambda 超时

**症状**: API 返回 504 Gateway Timeout

**解决方案**:
1. 增加 `lambda_timeout` (最大 900 秒)
2. 增加 `lambda_memory_mb` (推荐 1024MB+)
3. 检查数据库查询性能

### 数据库连接失败

**症状**: Lambda 日志显示数据库连接超时

**解决方案**:
1. 检查安全组配置
2. 确认 Lambda 在正确的 VPC 子网
3. 验证数据库凭证

### 冷启动慢

**症状**: 首次请求响应时间长

**解决方案**:
1. 增加 Lambda 内存 (1024MB+)
2. 启用预留并发: `lambda_reserved_concurrency = 5`
3. 考虑使用 SnapStart (Java 11+)

## 清理资源

**警告**: 这将删除所有资源，包括数据库！

```bash
cd terraform
terraform destroy
```

## 成本优化

### 开发环境
- 禁用 NAT Gateway: `enable_nat_gateway = false`
- 使用小规格: `rds_instance_class = "db.t3.micro"`
- 减少 Lambda 内存: `lambda_memory_mb = 512`

### 生产环境
- 启用 Multi-AZ: `rds_multi_az = true`
- 启用预留并发: `lambda_reserved_concurrency = 10`
- 启用 CloudWatch Alarms

## 相关文档

- [完整部署指南](../terraform/DEPLOYMENT_GUIDE.md)
- [快速开始](../terraform/QUICK_START.md)
- [Terraform 配置示例](../terraform/terraform.tfvars.example)
