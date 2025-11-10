# 需求-代码映射规范

本文档定义如何建立和维护需求与代码变更之间的可追溯映射关系。

## 映射原则

### 核心原则
1. **所有代码变更必须关联需求**：无需求编号的代码不应合并
2. **双向可追溯**：从需求可以找到代码，从代码可以追溯到需求
3. **自动化优先**：通过工具自动建立和维护映射关系
4. **实时更新**：映射关系应随开发进度实时更新

### 例外情况
以下类型的变更可不强制关联需求编号：
- 代码格式化（`style:` 类型）
- 文档更新（`docs:` 类型，不涉及功能）
- 依赖升级（`chore(deps):` 类型）
- CI/CD 配置（`ci:` 类型）

## 映射方法

### 1. 通过 Commit 消息关联

#### Commit 消息格式
```
<type>(<scope>): <subject> [REQ-YYYY-NNN]

<body>

Refs: REQ-YYYY-NNN
```

#### 示例
```bash
git commit -m "feat(auth): implement OAuth login [REQ-2025-001]

Add OAuth 2.0 authentication flow for Google and GitHub providers.

Refs: REQ-2025-001"
```

#### 多个需求关联
如果一个变更涉及多个需求：
```
feat(user): update user profile page [REQ-2025-001, REQ-2025-003]

Refs: REQ-2025-001, REQ-2025-003
```

### 2. 通过 Pull Request 关联

#### PR 标题格式
```
<type>(<scope>): <subject> [REQ-YYYY-NNN]
```

#### PR 描述模板
```markdown
## 需求关联
- 需求编号：REQ-YYYY-NNN
- 需求标题：<标题>
- PRD 文档：[链接到 PRD]

## 变更说明
<描述本 PR 的主要变更>

## 测试说明
- [ ] 单元测试已通过
- [ ] 集成测试已通过
- [ ] 手动测试场景：<描述>

## 验收标准
本 PR 实现了以下验收标准：
- [x] 验收标准 1
- [x] 验收标准 2
- [ ] 验收标准 3（后续 PR）

## 相关 Issue
Closes #XXX
Related to #YYY
```

#### 示例
```markdown
## 需求关联
- 需求编号：REQ-2025-001
- 需求标题：用户认证功能
- PRD 文档：[prd/REQ-2025-001-user-authentication.md](../prd/REQ-2025-001-user-authentication.md)

## 变更说明
实现了 OAuth 2.0 登录流程，支持 Google 和 GitHub 两种登录方式。

## 测试说明
- [x] 单元测试已通过（覆盖率 95%）
- [x] 集成测试已通过
- [x] 手动测试场景：
  - 使用 Google 账号登录
  - 使用 GitHub 账号登录
  - 登录失败处理
  - 登录后跳转

## 验收标准
本 PR 实现了以下验收标准：
- [x] 用户可以使用 Google 账号登录
- [x] 用户可以使用 GitHub 账号登录
- [x] 登录失败时显示友好错误信息
- [ ] 支持微信登录（后续 PR）

## 相关 Issue
Closes #42
```

### 3. 通过 GitHub Issue 关联

#### Issue 标题格式
```
[REQ-YYYY-NNN] <需求标题>
```

#### Issue 描述
```markdown
## 需求信息
- **需求编号**：REQ-YYYY-NNN
- **PRD 文档**：[链接]
- **优先级**：P0 / P1 / P2 / P3
- **负责人**：@username

## 需求描述
<简要描述>

## 验收标准
- [ ] 标准 1
- [ ] 标准 2
- [ ] 标准 3

## 子任务
- [ ] #XX：任务 1
- [ ] #YY：任务 2

## 相关 PR
- #XX：实现核心功能
- #YY：添加测试

## 进度追踪
- 2025-01-10: 需求评审通过
- 2025-01-15: 设计完成
- 2025-01-20: 开发中
```

### 4. 通过分支命名关联

#### 分支命名格式
```
<type>/<req-id>-<short-desc>
```

#### 示例
```bash
feature/req-2025-001-oauth-login
bugfix/req-2025-003-user-profile
hotfix/req-2025-005-security-patch
```

## 自动化映射

### GitHub Actions 工作流

创建 `.github/workflows/requirement-mapping.yml`：

```yaml
name: Requirement Mapping Check

on:
  pull_request:
    types: [opened, edited, synchronize]

jobs:
  check-requirement:
    runs-on: ubuntu-latest
    steps:
      - name: Check REQ ID in PR
        uses: actions/github-script@v7
        with:
          script: |
            const prTitle = context.payload.pull_request.title;
            const prBody = context.payload.pull_request.body || '';
            
            // 检查 PR 标题或正文中是否包含 REQ-YYYY-NNN
            const reqPattern = /REQ-\d{4}-\d{3}/;
            
            // 排除不需要需求编号的类型
            const exemptTypes = ['docs:', 'chore(deps):', 'ci:', 'style:'];
            const isExempt = exemptTypes.some(type => prTitle.startsWith(type));
            
            if (!isExempt && !reqPattern.test(prTitle) && !reqPattern.test(prBody)) {
              core.setFailed('❌ PR must include requirement ID (REQ-YYYY-NNN) in title or description');
            } else {
              console.log('✅ Requirement ID found or PR is exempt');
            }
```

### Commit 消息检查

使用 Git Hooks（`pre-commit` 或 `commit-msg`）检查提交消息：

```bash
#!/bin/bash
# .git/hooks/commit-msg

commit_msg_file=$1
commit_msg=$(cat "$commit_msg_file")

# 提取提交类型
type=$(echo "$commit_msg" | grep -oP '^[a-z]+')

# 排除类型
exempt_types=("docs" "chore" "ci" "style")

# 检查是否需要需求编号
needs_req=true
for exempt in "${exempt_types[@]}"; do
  if [[ "$commit_msg" == "$exempt:"* ]] || [[ "$commit_msg" == "$exempt("* ]]; then
    needs_req=false
    break
  fi
done

# 如果需要需求编号，检查是否存在
if [ "$needs_req" = true ]; then
  if ! echo "$commit_msg" | grep -qP 'REQ-\d{4}-\d{3}'; then
    echo "❌ Commit message must include requirement ID (REQ-YYYY-NNN)"
    echo "Format: type(scope): subject [REQ-YYYY-NNN]"
    exit 1
  fi
fi

echo "✅ Commit message format is valid"
exit 0
```

## 映射报告生成

### 自动生成需求实现报告

创建脚本 `scripts/generate-requirement-report.sh`：

```bash
#!/bin/bash

# 生成需求映射报告
# 用法: ./scripts/generate-requirement-report.sh REQ-2025-001

REQ_ID=$1

if [ -z "$REQ_ID" ]; then
  echo "Usage: $0 REQ-YYYY-NNN"
  exit 1
fi

echo "# 需求实现报告: $REQ_ID"
echo ""
echo "生成时间: $(date '+%Y-%m-%d %H:%M:%S')"
echo ""

# 查找相关 commits
echo "## 相关 Commits"
echo ""
git log --all --grep="$REQ_ID" --pretty=format:"- %h - %s (%an, %ar)" --abbrev-commit
echo ""
echo ""

# 查找相关文件变更
echo "## 涉及文件"
echo ""
git log --all --grep="$REQ_ID" --name-only --pretty=format:"" | sort -u | grep -v '^$'
echo ""
echo ""

# 统计代码行数变更
echo "## 代码统计"
echo ""
git log --all --grep="$REQ_ID" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "添加: %s 行\n删除: %s 行\n净变化: %s 行\n", add, subs, loc }'
echo ""
```

### 使用方法
```bash
# 生成特定需求的实现报告
./scripts/generate-requirement-report.sh REQ-2025-001 > reports/REQ-2025-001-report.md

# 生成所有需求的报告
for req in $(git log --all --pretty=format:"%s" | grep -oP 'REQ-\d{4}-\d{3}' | sort -u); do
  ./scripts/generate-requirement-report.sh $req > reports/${req}-report.md
done
```

## 需求覆盖率追踪

### 覆盖率定义
- **已开始**：存在相关的分支或 PR
- **开发中**：存在未合并的 PR
- **已完成**：所有验收标准已满足，代码已合并
- **已上线**：代码已部署到生产环境

### 覆盖率统计
在 GitHub Projects 中创建自动化视图：
1. 按需求编号分组
2. 统计各状态的需求数量
3. 计算完成率

### 示例查询
```
# 查找所有未完成的需求
state:open label:requirement

# 查找特定需求的所有 PR
REQ-2025-001 is:pr

# 查找已完成但未上线的需求
label:requirement label:done -label:deployed
```

## 可视化与报告

### 需求追踪仪表板
使用 GitHub Projects 的视图功能：
1. **按需求状态分组**：Draft / In Progress / Done
2. **按优先级过滤**：P0 / P1 / P2 / P3
3. **按负责人分组**：查看每个人的任务
4. **时间线视图**：查看需求时间计划

### 自动化报告
设置 GitHub Actions 定期生成报告：
- 每日需求进度报告
- 每周需求完成统计
- 每月需求覆盖率分析

## 最佳实践

1. **一致性**：整个团队使用统一的格式和规范
2. **及时更新**：代码变更后立即关联需求
3. **粒度适中**：需求不宜过大或过小
4. **清晰明确**：需求编号和描述应清晰易懂
5. **自动化检查**：通过 CI/CD 强制执行规范
6. **定期审查**：定期检查映射关系的准确性
7. **文档同步**：保持代码、文档、需求的一致性

## 常见问题

### Q1: 一个 PR 涉及多个需求怎么办？
A: 建议拆分 PR，一个 PR 对应一个需求。如果确实需要，可以在标题和描述中列出所有需求编号。

### Q2: 需求变更后如何更新映射？
A: 在 PRD 中记录变更，相关 PR 的描述中说明变更原因，并在 commit 消息中引用需求编号。

### Q3: 如何处理技术债务和重构？
A: 技术债务也应创建需求编号（可使用 TECH-YYYY-NNN 格式），遵循同样的映射规则。

### Q4: Hotfix 如何关联需求？
A: Hotfix 应关联到原需求或创建新的紧急需求编号（HOT-YYYY-NNN）。

## 相关文档

- [需求管理规范](./requirements-management.md)
- [开发计划管理](./development-planning.md)
- [Git Flow 规范](../github/git-flow.md)
- [AI 开发工作流](./ai-development-workflow.md)
