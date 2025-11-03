# Git Flow 规范

使用 main + develop + feature/bugfix/hotfix/release 的分支模型，面向持续集成与稳定发布。

分支命名规则
- main：受保护，始终指向生产可发布版本。
- develop：集成开发分支。
- feature/<id>-<short-desc>：从 develop 创建，用于新功能开发。
  - 示例：feature/ABC-123-user-login、feature/auth-login
  - 规则：小写、kebab-case；建议包含需求/缺陷ID（如 JIRA/Issue 编号）。
- bugfix/<id>-<short-desc>：从 develop 创建，用于修普通缺陷。
  - 示例：bugfix/ABC-456-fix-timezone
- hotfix/<id|version>-<short-desc>：从 main 创建，用于紧急线上修复。
  - 示例：hotfix/1.2.3-null-pointer
- release/<version>[-rc.n]：从 develop 创建，用于发布前稳定与验证。
  - 示例：release/1.3.0-rc.1、release/1.3.0

Pull Request 命名规则
- 标题格式：type(scope): summary [ID]
  - type：feat | fix | docs | chore | refactor | test | perf | ci | build | revert
  - scope：可选，模块/目录名（如 auth、api）
  - ID：可选，需求/缺陷编号（如 ABC-123）
  - 示例：
    - feat(auth): support SSO login [ABC-123]
    - fix(api): handle 204 response correctly [BUG-456]
- 目标分支与合并策略：
  - feature/、bugfix/ -> develop（建议 squash merge）
  - release/ -> main（合并后打 tag），并回合到 develop
  - hotfix/ -> main（合并后打 tag），并回合到 develop
- PR 内容建议（非强制）：
  - 变更说明、测试说明、风险与回滚方案、关联 issue/工单链接
