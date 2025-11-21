# GitHub Issues 信息收集使用指南

**项目**: JUREN 跨平台数据整合架构优化
**任务编号**: task-0001
**版本**: v1.0
**更新日期**: 2025-11-20

---

## 📖 目录

1. [概述](#概述)
2. [快速开始](#快速开始)
3. [Issue 结构说明](#issue-结构说明)
4. [如何填写信息](#如何填写信息)
5. [任务清单使用](#任务清单使用)
6. [协作最佳实践](#协作最佳实践)
7. [导出信息汇总](#导出信息汇总)
8. [常见问题](#常见问题)

---

## 📝 概述

本项目使用 **GitHub Issues** 作为信息收集和协作平台,具有以下优势:

✅ **多人协作** - 每个人可以独立填写负责的部分
✅ **进度可视化** - 实时查看收集进度
✅ **历史追溯** - 所有讨论和修改都有完整记录
✅ **通知机制** - 自动通知相关人员
✅ **集中管理** - 所有信息集中在 GitHub 仓库

### 仓库信息

- **GitHub 仓库**: [`juren-gu/juren-documents`](https://github.com/juren-gu/juren-documents)
- **Milestone**: `task-0001-信息收集阶段`
- **Issues 数量**: 7 个 (分 7 大类别)

---

## 🚀 快速开始

### 步骤 1: 访问 Issues 列表

打开浏览器访问:
```
https://github.com/juren-gu/juren-documents/issues?q=is%3Aissue+label%3Atask-0001
```

或者在仓库页面:
1. 点击 `Issues` 标签
2. 筛选标签 `task-0001`

### 步骤 2: 选择你负责的 Issue

找到分配给你的 Issue,或者你感兴趣的收集项:

| Issue # | 标题 | 优先级 | 任务数 |
|---------|------|--------|--------|
| #1 | 平台与环境信息收集 | P0 | 4 项 |
| #2 | 数据库架构信息收集 | P0 | 6 项 |
| #3 | 同步机制现状收集 | P0 | 6 项 |
| #4 | 性能与容量指标收集 | P1 | 5 项 |
| #5 | 业务约束与要求收集 | P1 | 4 项 |
| #6 | 技术栈与依赖收集 | P1 | 5 项 |
| #7 | 安全与合规信息收集 | P0 | 4 项 |

### 步骤 3: 阅读填写指南

每个 Issue 都包含详细的填写指南和示例,请仔细阅读。

### 步骤 4: 在评论中填写信息

使用评论回复的方式提交你收集到的信息。

---

## 🏗️ Issue 结构说明

每个 Issue 包含以下部分:

### 1. 基本信息
```markdown
## 📋 信息收集项: [类别名称]

**优先级**: P0 (必填) / P1 (重要)
**截止日期**: _待设定_
**负责人**: @username
```

### 2. 任务清单 (Task List)
```markdown
### ✅ 任务清单 (Task List)

- [ ] **子任务 1** - 说明 (负责人: @user1)
- [ ] **子任务 2** - 说明 (负责人: @user2)
- [ ] **子任务 3** - 说明 (负责人: @user3)
```

GitHub 会自动显示完成进度,例如: `2/6 完成 (33%)`

### 3. 填写指南
详细说明每个子任务需要收集的信息。

### 4. 收集结果 (评论区)
在评论中填写实际收集到的信息。

### 5. 相关文档链接
提供参考文档链接。

---

## ✍️ 如何填写信息

### 方法 1: 使用 Markdown 表格 (推荐)

```markdown
### 1.1 平台清单 - @your-username

**平台总数**: 3 个

| 平台名称 | 云服务商 | 区域/Region | 用途说明 | 网络类型 |
|---------|---------|------------|---------|---------|
| Production-AWS | AWS | ap-northeast-1 | 生产环境 | VPC |
| Analytics-GCP | GCP | asia-northeast1 | 数据分析 | VPC |
| Legacy-IDC | 自建IDC | - | 遗留系统 | 物理网络 |

**备注**: [补充说明]
```

### 方法 2: 使用列表格式

```markdown
### 1.2 服务部署清单 - @your-username

**Production-AWS**:
- 服务名称: user-service
- 服务类型: API
- 编程语言: PHP 7.4
- 框架版本: Laravel 8.x

**Analytics-GCP**:
- 服务名称: analytics-worker
- 服务类型: Worker
- 编程语言: Python 3.9
```

### 方法 3: 使用代码块

适合展示配置、脚本、日志等:

````markdown
### 3.1 同步脚本示例 - @your-username

```bash
#!/bin/bash
# sync_orders.sh
# 从 DB-1 同步订单数据到 DB-2
...
```
````

### 方法 4: 上传附件

如果有图表、截图、文档:

1. 直接拖拽图片到评论框
2. GitHub 会自动上传并生成链接
3. 或者使用 `[文件名](链接)` 格式

---

## ✅ 任务清单使用

### 如何勾选任务

**方法 1: 通过 Web 界面**
1. 打开 Issue
2. 找到任务清单
3. 直接点击 `[ ]` checkbox 即可勾选

**方法 2: 通过编辑 Issue**
1. 点击 Issue 标题右侧的 `Edit` 按钮
2. 将 `- [ ]` 改为 `- [x]`
3. 保存更改

### 任务完成流程

1. **开始任务**: 在评论中说明你开始填写某个子任务
   ```
   @team 我开始收集"平台清单"信息
   ```

2. **填写信息**: 在评论中填写收集到的信息

3. **勾选任务**: 完成后勾选任务清单中的对应项

4. **请求审核**: 所有子任务完成后,@提及审核人
   ```
   @reviewer 所有信息已填写完成,请审核
   ```

---

## 🤝 协作最佳实践

### 1. 使用 @ 提及

- 提问: `@username 请问 Platform-A 的区域是哪里?`
- 通知: `@team 平台清单已更新`
- 请求审核: `@reviewer 请审核数据库信息`

### 2. 使用标签和状态

Issue 会自动添加状态标签:
- `待填写` - 尚未开始
- `填写中` - 正在填写
- `待审核` - 等待审核
- `已完成` - 已完成

### 3. 讨论与澄清

如果对收集项有疑问:
1. 在 Issue 评论中提问
2. @ 相关人员
3. 讨论达成一致后再填写

### 4. 及时更新

- 发现新信息时及时补充
- 信息有变化时及时更新
- 在原评论中使用 `编辑` 功能修改

### 5. 敏感信息处理

⚠️ **不要在 Issue 中直接贴出**:
- 密码、密钥
- 数据库连接串
- API 密钥
- 个人身份信息

建议做法:
- 使用占位符: `密码: ******** (已私下发送)`
- 引用文档: `详见内部 Confluence 文档 [链接]`
- 加密分享: 使用 1Password/LastPass 等工具分享

---

## 📊 导出信息汇总

### 方法 1: 使用导出脚本

在本地运行导出脚本:

```bash
cd /path/to/workspace-juren/agents/scripts
./export_issues_to_md.sh
```

会生成汇总文档:
- `agents/task-0001/collected-data.md` - 完整汇总
- `agents/task-0001/collected-data-summary.md` - 简要汇总

### 方法 2: 手动导出

1. 打开每个 Issue
2. 复制评论中的信息
3. 粘贴到本地 Markdown 文档

### 方法 3: 使用 GitHub API

```bash
gh api repos/juren-gu/juren-documents/issues \
  --jq '.[] | select(.labels[].name == "task-0001") | {number, title, comments}'
```

---

## ❓ 常见问题

### Q1: 我不知道某些信息,怎么办?

**A**:
1. 在 Issue 评论中说明: `❓ 此信息暂时无法获取,原因是...`
2. @ 可能知道的人询问
3. 标注为 `待确认` 并继续填写其他项

### Q2: 信息填写错误了怎么办?

**A**:
1. 点击你的评论右上角的 `···` 菜单
2. 选择 `Edit` 编辑评论
3. 修改后保存
4. GitHub 会显示编辑历史

### Q3: 如何查看整体进度?

**A**:
1. 访问 Milestone 页面:
   ```
   https://github.com/juren-gu/juren-documents/milestone/[NUMBER]
   ```
2. 查看进度条和完成百分比
3. 或者使用 GitHub Project 看板(如已创建)

### Q4: 可以同时填写多个 Issue 吗?

**A**:
可以!GitHub Issues 支持并行协作,互不影响。

### Q5: 填写完成后还能修改吗?

**A**:
可以!即使 Issue 关闭后,仍然可以:
1. 添加新评论
2. 编辑已有评论
3. 重新打开 Issue

### Q6: 如何知道我负责哪些 Issue?

**A**:
1. 查看分配给你的 Issue:
   ```
   https://github.com/juren-gu/juren-documents/issues/assigned/@me
   ```
2. 或者查看筛选: `label:task-0001 assignee:@me`

### Q7: Issue 中的信息会公开吗?

**A**:
- 公开仓库: 所有人可见
- 私有仓库: 仅团队成员可见
- `juren-documents` 是私有仓库,仅授权人员可见

---

## 📞 获取帮助

遇到问题?

1. **查看本指南** - 先查看上述说明
2. **查看 Issue 模板** - 每个 Issue 都有详细说明
3. **在 Issue 中提问** - @ 项目负责人
4. **Slack/钉钉** - 在团队群组讨论

---

## 🔗 相关资源

- [GitHub Issues 官方文档](https://docs.github.com/en/issues)
- [Markdown 语法指南](https://guides.github.com/features/mastering-markdown/)
- [项目需求文档](./需求.md)
- [信息收集清单模板](./信息收集清单0001.md)

---

**最后更新**: 2025-11-20
**维护人**: Project Manager Agent (Warp)
**问题反馈**: 在 juren-documents 仓库创建 Issue
