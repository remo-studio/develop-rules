# PRD 文档目录

本目录存放产品需求文档（Product Requirement Document）。

## 目录结构

```
prd/
├── README.md                           # 本说明文件
├── REQ-2025-001-user-authentication.md # 示例 PRD
└── REQ-YYYY-NNN-<description>.md       # 按需求编号命名的 PRD 文件
```

## 命名规范

PRD 文件命名格式：`REQ-YYYY-NNN-<简短描述>.md`

- `REQ`：需求标识符
- `YYYY`：年份（4 位数）
- `NNN`：序号（3 位数，001 开始）
- `<简短描述>`：小写字母，单词用连字符分隔

### 示例
- `REQ-2025-001-user-authentication.md`
- `REQ-2025-002-user-profile.md`
- `REQ-2025-003-payment-integration.md`

## 创建 PRD

### 使用 AI 生成 PRD

参考 [需求管理规范](../workflows/requirements-management.md)，使用以下 Prompt：

```
请根据以下信息生成一份产品需求文档（PRD）：

背景：[业务背景]
功能：[功能描述]
用户：[目标用户]
目标：[预期目标]

请按照以下结构生成：
1. 背景与目标
2. 需求描述（功能需求 + 非功能需求）
3. 用户故事与验收标准
4. 技术方案概要
5. 依赖关系
6. 时间计划
7. 风险评估

格式要求：Markdown，清晰易读，包含表格和列表。
```

### 手动创建 PRD

1. 复制模板文件 `REQ-2025-001-user-authentication.md`
2. 根据实际需求修改内容
3. 分配新的需求编号
4. 填写完整信息

## PRD 状态

PRD 文档头部应标注当前状态：

- **Draft**：草稿，正在编写中
- **In Review**：审查中，等待反馈
- **Approved**：已批准，可以开始开发
- **In Progress**：开发中
- **Done**：已完成，已上线
- **Cancelled**：已取消

## 审查流程

1. 创建 PRD 草稿（状态：Draft）
2. 创建 PR 提交到仓库
3. 邀请相关人员审查（产品、技术、安全）
4. 根据反馈修改
5. 审查通过后合并，更新状态为 Approved
6. 开发开始时更新状态为 In Progress
7. 功能上线后更新状态为 Done

## 需求追踪

每个需求对应一个 GitHub Issue：

1. 创建 Issue，标题：`[REQ-YYYY-NNN] <需求标题>`
2. 在 Issue 描述中链接到 PRD 文档
3. 添加标签：`requirement`, `prd`
4. 关联到 GitHub Project

所有相关的 PR 和 Commit 都应包含需求编号：
```
feat(auth): implement user login [REQ-2025-001]
```

## 示例文档

- [REQ-2025-001-user-authentication.md](./REQ-2025-001-user-authentication.md) - 用户认证功能

## 相关文档

- [需求管理规范](../workflows/requirements-management.md)
- [需求代码映射](../workflows/requirement-code-mapping.md)
- [AI 开发工作流](../workflows/ai-development-workflow.md)
