# 开发规范

该仓库用于定义与统一团队的开发标准，涵盖两大类内容：
- 代码规范（语言与工具链的风格与约束）
- GitHub 使用规范（分支、PR、发布/Tag/Release 等）

文档导航
- 分支与 PR 规范：`github/git-flow.md`
- 发布与 Tag/Release 规范：`github/git-release.md`

如何使用
- 在团队项目的 README 中链接到本仓库相应章节，以对齐统一流程。
- 新建功能/修复分支时遵循分支命名约定；提交 PR 按约定命名并选择正确目标分支。
- 发布版本时按语义化版本为主线分支打注解 Tag，并创建对应 Release 页面。

摘要
- 分支命名：`feature/<id>-<desc>`、`bugfix/<id>-<desc>`、`hotfix/<id|version>-<desc>`、`release/<version>`；主线：`main`，集成：`develop`。
- PR 标题：`type(scope): summary [ID]`（示例：`feat(auth): support SSO login [ABC-123]`）。
- 版本与标签：语义化版本 `vMAJOR.MINOR.PATCH`（示例：`v1.3.0`、`v1.3.0-rc.1`），发布分支 `release/X.Y.Z`。

贡献
- 通过 Pull Request 更新规范文档，建议分支名：`feature/docs-<desc>`；PR 标题：`docs: <summary>`。
