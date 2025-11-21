## 📋 信息收集项: 技术栈与依赖

**优先级**: P1 (重要)
**截止日期**: _待设定_
**负责人**: @_待分配_

---

### ✅ 任务清单 (Task List)

- [ ] **6.1 当前技术栈** - 列出后端语言与框架 (负责人: @___)
- [ ] **6.2 依赖服务** - 列出使用的中间件 (负责人: @___)
- [ ] **6.3 运维工具** - 了解 DevOps 工具链 (负责人: @___)
- [ ] **6.4 监控与日志** - 收集监控和日志系统 (负责人: @___)
- [ ] **6.5 语言迁移考虑** - PHP → Java 迁移规划 (负责人: @___)

---

### 💬 收集结果

```markdown
### 6.1 当前技术栈 - @username

| 服务名称 | 语言 | 框架 | 版本 | 备注 |
|---------|------|------|------|------|
| user-service | PHP | Laravel | 8.x | 主要用户服务 |
| order-service | PHP | Laravel | 8.x | 订单处理 |
| payment-service | PHP | Laravel | 8.x | 支付服务 |
| inventory-service | PHP | ThinkPHP | 6.0 | 库存管理(遗留系统) |
| analytics-worker | Python | - | 3.9 | 数据分析任务 |
| data-sync-service | PHP/Python | - | 混合 | 数据同步脚本 |
| notification-service | Node.js | Express | 4.x | 通知服务 |
```

```markdown
### 6.2 依赖服务 - @username

**使用的中间件**:

| 类型 | 产品 | 版本 | 用途 |
|------|------|------|------|
| 消息队列 | RabbitMQ | 3.11 | 异步任务处理 |
| 消息队列 | AWS SQS | - | 部分异步任务 |
| 缓存 | Redis | 7.0 | Session + 数据缓存 |
| 任务调度 | Cron | - | 定时任务 |
| 任务调度 | Celery | 5.x | Python 异步任务 |
```

```markdown
### 6.3 运维工具 - @username

| 工具类型 | 产品 | 说明 |
|---------|------|------|
| CI/CD 平台 | GitLab CI + AWS CodeDeploy | 自动化部署 |
| 容器化 | ⚠️ 部分容器化 | 新服务使用 Docker,遗留系统未容器化 |
| 容器编排 | AWS ECS | 部分服务 |
| IaC 工具 | ❌ 未使用 | 基础设施手动配置 |
```

```markdown
### 6.4 监控与日志 - @username

| 工具类型 | 产品 | 说明 |
|---------|------|------|
| APM 工具 | ❌ 未部署 | - |
| 日志收集 | CloudWatch Logs | AWS 侧服务 |
| 日志收集 | ELK Stack | IDC 侧服务(自建) |
| 监控告警 | CloudWatch | 基础设施监控 |
| 监控告警 | Prometheus + Grafana | 应用监控(部分) |
| 分布式追踪 | ❌ 未部署 | - |
```

```markdown
### 6.5 语言迁移考虑 - @username

**是否计划 PHP → Java 迁移**: ⚠️ 考虑中

**如果是,迁移优先级**: 🟡 中 - 数据库改造完成后进行

**PHP 服务清单**:

| 服务名称 | 代码量(行) | 复杂度 | 是否有单元测试 | 建议迁移顺序 |
|---------|-----------|--------|---------------|-------------|
| user-service | ~15000 | 中 | ⚠️ 部分 | 3 |
| order-service | ~20000 | 高 | ⚠️ 部分 | 2 |
| payment-service | ~12000 | 高 | ✅ 是 | 4 |
| inventory-service | ~25000 | 高 | ❌ 否 | 1 (最优先) |
| data-sync-service | ~8000 | 中 | ❌ 否 | 与数据库改造同步 |

**迁移考虑**:
- inventory-service (库存服务) 性能最差,建议优先迁移
- order-service (订单服务) 业务复杂度高,需要充分测试
- 建议采用"绞杀者模式"(Strangler Fig Pattern)逐步迁移
```

---

### 🔗 相关文档

- [信息收集清单完整模板](../../agents/task-0001/信息收集清单0001.md#6%EF%B8%8F%E2%83%A3-%E6%8A%80%E6%9C%AF%E6%A0%88%E4%B8%8E%E4%BE%9D%E8%B5%96--tech-stack--dependencies)
