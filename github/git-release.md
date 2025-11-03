# Git Release 规范

版本与 Tag 命名
- 采用语义化版本（SemVer）：vMAJOR.MINOR.PATCH
  - 正式版：v1.3.0、v1.3.1
  - 预发布：v1.3.0-rc.1、v1.3.0-beta.2
  - 可选构建元数据：v1.3.0+build.20251103
- Tag 要求：使用注解 tag（annotated）并包含简要说明
  - 示例：git tag -a v1.3.0 -m "Release v1.3.0: 简述"
  - Tag 必须指向 main 上的发布 commit

Release 分支命名
- release/<version>[-rc.n]
  - 示例：release/1.3.0-rc.1、release/1.3.0

发布流程（建议）
1. 从 develop 创建 release/x.y.z，冻结功能，仅接受缺陷修复与文档更新。
2. 验证通过后，将 release 合并到 main；在 main 上打注解 Tag：vX.Y.Z。
3. 将 main 的发布变更回合到 develop，保持分支一致性。
4. 预发布阶段可在 release 分支上按需打 vX.Y.Z-rc.N 标签。
5. 紧急修复：从 main 创建 hotfix 分支，修复后合并回 main 并打 vX.Y.Z+1 标签，同时回合到 develop。

Release 页面命名与内容（GitHub Releases）
- Release 名称：vX.Y.Z（YYYY-MM-DD）
- 对应 Tag：vX.Y.Z 或 vX.Y.Z-rc.N
- 正文分区建议：Added / Changed / Fixed / Removed / Security / Breaking Changes
- 附件：必要时附上构建产物（如二进制/归档包）

命名规则汇总
- Tag：vX.Y.Z、vX.Y.Z-rc.N
- Release 分支：release/X.Y.Z、release/X.Y.Z-rc.N
- 发布相关 PR 标题：release: vX.Y.Z
