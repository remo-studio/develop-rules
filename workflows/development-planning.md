# 开发计划管理规范

本文档定义如何使用 GitHub Projects 和 Issues 进行开发计划管理和任务追踪。

## GitHub Projects 使用规范

### Project 类型

#### 1. 产品路线图 (Product Roadmap)
**用途**：长期产品规划，跨季度/年度视图

**视图配置**：
- **时间线视图**：按季度展示需求
- **表格视图**：需求列表
- **看板视图**：按状态分组

**字段定义**：
- Status: Backlog / Planned / In Progress / Done / Cancelled
- Priority: P0 (Critical) / P1 (High) / P2 (Medium) / P3 (Low)
- Quarter: Q1 2025 / Q2 2025 / ...
- Epic: 关联的史诗需求
- Owner: 负责人
- Requirement ID: REQ-YYYY-NNN

#### 2. Sprint 看板 (Sprint Board)
**用途**：迭代开发，1-2 周一个 Sprint

**视图配置**：
- **看板视图**：Todo / In Progress / In Review / Done
- **表格视图**：任务列表
- **日历视图**：按到期日期展示

**字段定义**：
- Status: Todo / In Progress / In Review / Done
- Priority: P0 / P1 / P2 / P3
- Sprint: Sprint 1 / Sprint 2 / ...
- Assignee: 指派人
- Story Points: 1 / 2 / 3 / 5 / 8 / 13
- Requirement ID: REQ-YYYY-NNN

#### 3. Bug 追踪 (Bug Tracking)
**用途**：缺陷管理和修复追踪

**视图配置**：
- **看板视图**：New / Confirmed / In Progress / Fixed / Closed
- **表格视图**：按严重程度排序
- **优先级视图**：按优先级分组

**字段定义**：
- Status: New / Confirmed / In Progress / Fixed / Verified / Closed
- Severity: Critical / High / Medium / Low
- Priority: P0 / P1 / P2 / P3
- Affected Version: v1.0.0 / v1.1.0 / ...
- Fixed Version: v1.0.1 / v1.1.0 / ...

## GitHub Issues 使用规范

### Issue 模板

#### 1. 功能需求模板 (Feature Request)
创建 `.github/ISSUE_TEMPLATE/feature_request.md`：

```markdown
---
name: 功能需求
about: 提出新功能或改进建议
title: '[REQ-YYYY-NNN] '
labels: 'requirement, feature'
assignees: ''
---

## 需求信息
- **需求编号**：REQ-YYYY-NNN
- **PRD 文档**：[链接到 PRD]
- **优先级**：P0 / P1 / P2 / P3
- **预计工作量**：<Story Points>

## 需求描述
简要描述功能需求和业务价值。

## 用户故事
作为 [角色]，我希望 [功能]，以便 [价值]。

## 验收标准
- [ ] 验收标准 1
- [ ] 验收标准 2
- [ ] 验收标准 3

## 技术方案（可选）
如有技术方案，简要说明。

## 依赖关系
- 依赖需求：#XX
- 阻塞需求：#YY

## 相关资源
- 设计稿：[链接]
- API 文档：[链接]
- 参考资料：[链接]
```

#### 2. 缺陷报告模板 (Bug Report)
创建 `.github/ISSUE_TEMPLATE/bug_report.md`：

```markdown
---
name: 缺陷报告
about: 报告软件缺陷或错误
title: '[BUG] '
labels: 'bug'
assignees: ''
---

## 缺陷信息
- **严重程度**：Critical / High / Medium / Low
- **优先级**：P0 / P1 / P2 / P3
- **影响版本**：v1.0.0
- **环境**：生产 / 测试 / 开发

## 缺陷描述
简要描述缺陷现象。

## 重现步骤
1. 第一步
2. 第二步
3. 第三步

## 预期行为
描述预期的正确行为。

## 实际行为
描述实际发生的错误行为。

## 截图或日志
如有，请附上截图或相关日志。

## 环境信息
- 操作系统：
- 浏览器：
- 版本：

## 可能的原因（可选）
如果有初步分析，可以说明。

## 相关需求
- 关联需求：REQ-YYYY-NNN
```

#### 3. 任务模板 (Task)
创建 `.github/ISSUE_TEMPLATE/task.md`：

```markdown
---
name: 任务
about: 创建开发任务或工作项
title: ''
labels: 'task'
assignees: ''
---

## 任务信息
- **类型**：开发 / 测试 / 文档 / 运维
- **需求编号**：REQ-YYYY-NNN
- **优先级**：P0 / P1 / P2 / P3
- **预计工作量**：<Story Points>

## 任务描述
详细描述任务内容和目标。

## 完成标准
- [ ] 标准 1
- [ ] 标准 2
- [ ] 标准 3

## 技术要点
列出关键技术点或注意事项。

## 依赖任务
- 依赖：#XX
- 阻塞：#YY

## 相关资源
- 设计文档：[链接]
- 参考代码：[链接]
```

### Issue 命名规范

#### 格式
```
[<类型>] <简短描述> [REQ-YYYY-NNN]
```

#### 类型标识
- `[REQ-YYYY-NNN]`：需求
- `[BUG]`：缺陷
- `[TASK]`：任务
- `[SPIKE]`：技术调研
- `[DOC]`：文档

#### 示例
- `[REQ-2025-001] 用户认证功能`
- `[BUG] 登录页面无法提交 [REQ-2025-001]`
- `[TASK] 实现 OAuth 登录接口 [REQ-2025-001]`

### Issue 标签体系

#### 类型标签
- `requirement`: 功能需求
- `bug`: 缺陷
- `task`: 开发任务
- `spike`: 技术调研
- `documentation`: 文档

#### 优先级标签
- `priority: p0`: 关键，立即处理
- `priority: p1`: 高优先级
- `priority: p2`: 中优先级
- `priority: p3`: 低优先级

#### 状态标签
- `status: backlog`: 待规划
- `status: todo`: 待开始
- `status: in-progress`: 进行中
- `status: in-review`: 审查中
- `status: blocked`: 已阻塞
- `status: done`: 已完成

#### 模块标签
- `module: auth`: 认证模块
- `module: api`: API 模块
- `module: ui`: UI 模块
- `module: database`: 数据库

## Sprint 规划流程

### 1. Sprint 准备
**时间**：Sprint 开始前 1-2 天

**活动**：
1. Product Owner 整理 Backlog
2. 评估需求优先级
3. 准备需求细节和验收标准
4. 确保高优先级需求已有设计

### 2. Sprint 计划会议
**时间**：2 小时（2 周 Sprint）

**参与者**：团队所有成员

**议程**：
1. **回顾上个 Sprint**（15 分钟）
   - 完成情况
   - 遇到的问题
   - 改进点
   
2. **确定 Sprint 目标**（15 分钟）
   - 本 Sprint 要达成什么
   - 关键需求或里程碑
   
3. **选择 Backlog 项**（60 分钟）
   - 从 Product Backlog 选择本 Sprint 要完成的需求
   - 估算工作量（Story Points）
   - 确保团队承诺的工作量合理
   
4. **任务分解**（30 分钟）
   - 将需求分解为具体任务
   - 分配任务给团队成员
   - 识别依赖和风险

**输出**：
- Sprint Backlog（本 Sprint 的任务列表）
- Sprint 目标声明
- 任务分配表

### 3. 每日站会 (Daily Standup)
**时间**：15 分钟，每天固定时间

**议程**（每人回答 3 个问题）：
1. 昨天完成了什么？
2. 今天计划做什么？
3. 有什么阻碍？

**注意事项**：
- 保持简短，不展开讨论
- 有阻碍立即提出，会后解决
- 在 GitHub Projects 上更新任务状态

### 4. Sprint 评审会议 (Sprint Review)
**时间**：1 小时

**参与者**：团队 + 利益相关者

**议程**：
1. 演示完成的功能（30 分钟）
2. 收集反馈（20 分钟）
3. 更新 Product Backlog（10 分钟）

### 5. Sprint 回顾会议 (Sprint Retrospective)
**时间**：1 小时

**参与者**：团队成员

**议程**：
1. 做得好的地方（What went well）
2. 需要改进的地方（What could be improved）
3. 行动计划（Action items）

**输出**：
- 改进行动清单
- 下个 Sprint 要尝试的新实践

## 工作量估算

### Story Points 定义
- **1 点**：非常简单，< 2 小时
- **2 点**：简单，半天
- **3 点**：中等，1 天
- **5 点**：较复杂，2-3 天
- **8 点**：复杂，1 周
- **13 点**：非常复杂，需要分解

### 估算方法：Planning Poker
1. Product Owner 介绍需求
2. 团队讨论和提问
3. 每人私下选择 Story Points
4. 同时亮牌
5. 如有差异，讨论后重新估算
6. 达成一致

### 团队速率 (Velocity)
- 记录每个 Sprint 完成的 Story Points
- 计算最近 3 个 Sprint 的平均值
- 作为下个 Sprint 的参考容量

## 进度追踪与报告

### 燃尽图 (Burndown Chart)
跟踪 Sprint 期间剩余工作量的变化。

**示例**：
```
Story Points
     ^
30   |●
     |  ●
20   |    ●
     |      ●
10   |        ●
     |          ●
0    |____________●____> Days
     1  2  3  4  5  6  7
```

### 进度报告
**频率**：每周

**内容**：
- Sprint 目标和当前进度
- 已完成任务列表
- 进行中任务状态
- 风险和阻碍
- 下周计划

**示例模板**：
```markdown
# Sprint 10 - 周进度报告

**日期**：2025-01-06 至 2025-01-12  
**Sprint 目标**：完成用户认证功能（REQ-2025-001）

## 完成情况
- ✅ OAuth 登录接口实现 (#42)
- ✅ JWT Token 生成和验证 (#43)
- ✅ 用户注册接口 (#44)

## 进行中
- 🔄 密码重置功能 (#45) - 80% 完成
- 🔄 登录页面 UI (#46) - 60% 完成

## 计划下周
- 完成密码重置功能
- 完成登录页面 UI
- 编写集成测试

## 风险与阻碍
- ⚠️ 第三方 OAuth 服务偶尔超时，需要添加重试逻辑

## 指标
- 完成 Story Points: 13 / 21
- 燃尽率: 62%
- 预计可按时完成: 是
```

## 自动化工作流

### GitHub Actions 自动化

#### 1. 自动标签
创建 `.github/workflows/auto-label.yml`：

```yaml
name: Auto Label Issues

on:
  issues:
    types: [opened, edited]

jobs:
  label:
    runs-on: ubuntu-latest
    steps:
      - name: Add requirement label
        if: contains(github.event.issue.title, '[REQ-')
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.addLabels({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              labels: ['requirement']
            });
      
      - name: Add bug label
        if: contains(github.event.issue.title, '[BUG]')
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.addLabels({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              labels: ['bug']
            });
```

#### 2. 自动关联 Project
```yaml
name: Add to Project

on:
  issues:
    types: [opened]
  pull_request:
    types: [opened]

jobs:
  add-to-project:
    runs-on: ubuntu-latest
    steps:
      - name: Add to Sprint Board
        uses: actions/add-to-project@v0.5.0
        with:
          project-url: https://github.com/orgs/<org>/projects/<number>
          github-token: ${{ secrets.ADD_TO_PROJECT_PAT }}
```

#### 3. Sprint 提醒
```yaml
name: Sprint Reminder

on:
  schedule:
    - cron: '0 9 * * 1'  # 每周一早上 9 点

jobs:
  remind:
    runs-on: ubuntu-latest
    steps:
      - name: Send Sprint Start Reminder
        uses: actions/github-script@v7
        with:
          script: |
            // 发送 Slack 通知或创建 Issue
            console.log('New Sprint starts today!');
```

## 最佳实践

1. **小步快跑**：保持 Sprint 短小（1-2 周）
2. **持续沟通**：每日站会不可缺席
3. **及时更新**：实时更新任务状态
4. **透明可见**：进度对所有人可见
5. **拥抱变化**：需求变更通过 Backlog 管理
6. **回顾改进**：每个 Sprint 后认真回顾
7. **合理承诺**：不过度承诺工作量
8. **关注价值**：优先完成高价值需求

## 常见问题

### Q1: Sprint 中可以增加需求吗？
A: 原则上不可以。如果有紧急需求，需要与团队协商，可能需要移除其他需求。

### Q2: 如何处理跨 Sprint 的大需求？
A: 将大需求分解为多个小需求，分布在多个 Sprint 中完成。

### Q3: Story Points 和工时的关系？
A: Story Points 不等于工时。1 点可能是 2 小时，也可能是 4 小时，取决于团队速率。

### Q4: Bug 修复需要估算 Story Points 吗？
A: P0/P1 的 Bug 直接处理，不占用 Sprint 容量。P2/P3 的 Bug 纳入 Backlog，需要估算。

### Q5: 如何提高团队速率？
A: 不应追求提高速率，而应追求稳定的速率。改进流程和技术债务可以自然提高速率。

## 相关文档

- [需求管理规范](./requirements-management.md)
- [需求代码映射](./requirement-code-mapping.md)
- [代码开发流程](./code-development.md)
- [AI 开发工作流](./ai-development-workflow.md)
