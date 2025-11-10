# 基于 GitHub 的 AI 开发工作流

本文档定义了利用 AI 辅助的完整软件开发生命周期工作流，涵盖从需求管理到运营的各个阶段。

## 概述

AI 开发工作流整合了人工智能工具与 GitHub 平台，实现高效、自动化的开发流程：
- 使用 AI 生成和管理需求文档（PRD）
- 建立需求与代码的可追溯映射关系
- 规范设计文档（API、数据库、云架构）
- 自动化开发计划管理
- AI 辅助代码编写与审查
- 自动化测试与部署
- 运营数据反馈循环

## 工作流阶段

### 1. 需求管理
详见：[需求管理规范](./requirements-management.md)
- 使用 AI 生成结构化 PRD 文档
- 自动分配需求编号（REQ-YYYY-NNN 格式）
- 需求版本控制与变更追踪
- 需求优先级与依赖管理

### 2. 需求-代码映射
详见：[需求代码映射](./requirement-code-mapping.md)
- 建立需求 ID 与代码变更的关联
- 通过 PR、Commit 消息追踪实现情况
- 自动生成需求实现报告
- 需求覆盖率统计

### 3. 设计资料管理
详见：[设计文档规范](./design-documentation.md)
- API 设计文档（OpenAPI/Swagger）
- 数据库设计（ER 图、Schema）
- 云架构设计（架构图、资源清单）
- 设计审查与版本管理

### 4. 开发计划
详见：[开发计划管理](./development-planning.md)
- GitHub Projects 看板管理
- Issue 模板与自动化
- Sprint 计划与迭代管理
- 进度追踪与报告

### 5. 代码开发
详见：[代码开发流程](./code-development.md)
- AI 辅助代码生成
- 代码审查（人工 + AI）
- 分支策略（参考 `../github/git-flow.md`）
- 代码质量门禁

### 6. 测试
详见：[测试流程](./testing-workflow.md)
- 单元测试自动化
- 集成测试流程
- AI 辅助测试用例生成
- 测试覆盖率要求

### 7. 部署
详见：[部署流程](./deployment-workflow.md)
- CI/CD 管道配置
- 环境管理（开发、测试、生产）
- 自动化部署流程
- 回滚策略

### 8. 运营
详见：[运营流程](./operations-workflow.md)
- 监控与告警
- 日志分析
- 性能优化
- 用户反馈收集

## 工具集成

### GitHub 原生工具
- **Issues**：需求与缺陷追踪
- **Projects**：看板与计划管理
- **Pull Requests**：代码审查与合并
- **Actions**：CI/CD 自动化
- **Discussions**：技术讨论与知识分享

### AI 工具
- **GitHub Copilot**：代码智能提示与生成
- **GPT/Claude**：PRD 生成、代码审查、文档编写
- **AI Code Review**：自动代码审查
- **AI Test Generation**：测试用例自动生成

### 第三方集成
- **项目管理**：JIRA、Linear、Notion
- **监控**：Datadog、New Relic、Sentry
- **通信**：Slack、Teams、飞书
- **文档**：Confluence、语雀

## 最佳实践

1. **需求先行**：所有开发工作必须关联明确的需求编号
2. **文档同步**：代码变更同步更新相关设计文档
3. **自动化优先**：能自动化的流程尽量自动化
4. **AI 辅助，人工决策**：AI 提供建议，关键决策由人做出
5. **持续反馈**：建立从运营到需求的反馈闭环
6. **质量门禁**：设置明确的质量标准，不达标不能合并/部署

## 快速开始

1. 阅读需求管理规范，了解如何使用 AI 生成 PRD
2. 根据 PRD 创建 GitHub Issue，关联需求编号
3. 在 GitHub Projects 中规划 Sprint
4. 按照 Git Flow 创建功能分支开发
5. 使用 GitHub Copilot 辅助编码
6. 提交 PR，触发自动化测试与 AI 代码审查
7. 代码审查通过后合并，自动触发部署
8. 监控运营数据，收集反馈，进入下一轮迭代

## 相关文档

- [Git Flow 规范](../github/git-flow.md)
- [Git Release 规范](../github/git-release.md)
- [需求管理规范](./requirements-management.md)
- [需求代码映射](./requirement-code-mapping.md)
- [设计文档规范](./design-documentation.md)
- [开发计划管理](./development-planning.md)
- [代码开发流程](./code-development.md)
- [测试流程](./testing-workflow.md)
- [部署流程](./deployment-workflow.md)
- [运营流程](./operations-workflow.md)
