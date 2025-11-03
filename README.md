# 开发规范

该仓库用于定义与统一团队的开发标准，涵盖三大类内容：
- 代码规范（语言与工具链的风格与约束）
- GitHub 使用规范（分支、PR、发布/Tag/Release 等）
- AI 开发工作流（基于 AI 的完整开发生命周期）

## 文档导航

### GitHub 使用规范
- 分支与 PR 规范：[`github/git-flow.md`](github/git-flow.md)
- 发布与 Tag/Release 规范：[`github/git-release.md`](github/git-release.md)

### AI 开发工作流
- **工作流总览**：[`workflows/ai-development-workflow.md`](workflows/ai-development-workflow.md)
- 需求管理：[`workflows/requirements-management.md`](workflows/requirements-management.md)
- 需求代码映射：[`workflows/requirement-code-mapping.md`](workflows/requirement-code-mapping.md)
- 设计文档规范：[`workflows/design-documentation.md`](workflows/design-documentation.md)
- 开发计划管理：[`workflows/development-planning.md`](workflows/development-planning.md)
- 代码开发流程：[`workflows/code-development.md`](workflows/code-development.md)
- 测试流程：[`workflows/testing-workflow.md`](workflows/testing-workflow.md)
- 部署流程：[`workflows/deployment-workflow.md`](workflows/deployment-workflow.md)
- 运营流程：[`workflows/operations-workflow.md`](workflows/operations-workflow.md)

## 如何使用

### 基本使用
- 在团队项目的 README 中链接到本仓库相应章节，以对齐统一流程。
- 新建功能/修复分支时遵循分支命名约定；提交 PR 按约定命名并选择正确目标分支。
- 发布版本时按语义化版本为主线分支打注解 Tag，并创建对应 Release 页面。

### AI 工作流快速入门
1. **需求阶段**：使用 AI（ChatGPT/Claude）生成 PRD 文档，分配需求编号（REQ-YYYY-NNN）
2. **设计阶段**：编写 API、数据库、云架构设计文档
3. **计划阶段**：在 GitHub Projects 中创建 Sprint 看板，分解任务
4. **开发阶段**：使用 GitHub Copilot 辅助编码，关联需求编号
5. **测试阶段**：使用 AI 生成测试用例，运行自动化测试
6. **部署阶段**：通过 CI/CD 管道自动部署
7. **运营阶段**：监控指标，收集用户反馈，持续改进

摘要
- 分支命名：`feature/<id>-<desc>`、`bugfix/<id>-<desc>`、`hotfix/<id|version>-<desc>`、`release/<version>`；主线：`main`，集成：`develop`。
- PR 标题：`type(scope): summary [ID]`（示例：`feat(auth): support SSO login [ABC-123]`）。
- 版本与标签：语义化版本 `vMAJOR.MINOR.PATCH`（示例：`v1.3.0`、`v1.3.0-rc.1`），发布分支 `release/X.Y.Z`。

贡献
- 通过 Pull Request 更新规范文档，建议分支名：`feature/docs-<desc>`；PR 标题：`docs: <summary>`。
