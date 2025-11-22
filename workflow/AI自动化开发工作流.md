---
description: AI 辅助自动化开发工作流程
---

# AI 自动化开发工作流

本工作流描述了使用 AI 工具（如 Cursor、Claude、GitHub Copilot 等）进行自动化开发的完整流程，从需求分析到部署上线的全生命周期管理。

---

## 📋 目录

- [前置条件](#前置条件)
- [工作流程](#工作流程)
  - [阶段 1: 准备阶段](#阶段-1-准备阶段)
  - [阶段 2: 分析需求阶段](#阶段-2-分析需求阶段)
  - [阶段 3: 项目计划](#阶段-3-项目计划)
  - [阶段 4: 代码分析](#阶段-4-代码分析)
  - [阶段 5: 开发阶段](#阶段-5-开发阶段)
  - [阶段 6: 自动化测试](#阶段-6-自动化测试)
  - [阶段 7: 部署阶段](#阶段-7-部署阶段)
- [最佳实践](#最佳实践)
- [检查清单](#检查清单)
- [相关资源](#相关资源)

---

## 前置条件

### 1. 工具准备

- ✅ **AI 开发工具**: Cursor / Claude / GitHub Copilot / Codeium
- ✅ **版本控制**: Git >= 2.30
- ✅ **代码仓库**: GitHub / GitLab / Bitbucket 账号
- ✅ **项目管理**: GitHub Projects / Jira / Linear
- ✅ **文档工具**: Markdown 编辑器

### 2. 环境准备

- ✅ **工作空间**: 本地开发环境已配置
- ✅ **依赖管理**: 项目依赖工具（npm/pip/maven 等）
- ✅ **测试框架**: 单元测试、集成测试、E2E 测试工具
- ✅ **CI/CD**: GitHub Actions / GitLab CI / Jenkins

### 3. 模板准备

确保已熟悉 `develop-rules` 仓库中的模板：

- 📄 [PRD 模板](../prd/prd-template.md)
- 📄 [API 接口文档模板](../api/api-template.md)
- 📄 [数据库设计模板](../db/db-template.md)
- 📄 [测试指南](../testing/automation-testing-guide.md)
- 📄 [Git Flow 规范](../github/git-flow.md)

---

## 工作流程

### 阶段 1: 准备阶段

#### 1.1 整理项目资料

**目标**: 将所有相关资料集中到工作空间，为 AI 分析做准备。

**步骤**:

1. **创建工作空间目录**
   ```bash
   mkdir -p workspace-project-name
   cd workspace-project-name
   ```

2. **整理现有资料**
   ```
   workspace-project-name/
   ├── docs/              # 现有文档
   │   ├── requirements/  # 需求文档
   │   ├── design/        # 设计文档
   │   └── api/          # API 文档
   ├── code/              # 现有代码（如有）
   │   ├── backend/
   │   └── frontend/
   └── assets/            # 其他资源
   ```

3. **准备参考资料**
   - 业务需求文档
   - 现有系统架构图
   - 数据库设计文档
   - API 接口文档
   - 用户故事/用例
   - 设计稿/原型图

#### 1.2 准备 GitHub 仓库

**步骤**:

1. **创建仓库**
   ```bash
   # 在 GitHub 上创建新仓库
   # 或使用现有仓库
   git clone https://github.com/org/project-name.git
   cd project-name
   ```

2. **初始化仓库结构**
   ```bash
   # 创建标准目录结构
   mkdir -p docs/{prd,design,api,db}
   mkdir -p tests/{unit,integration,e2e}
   mkdir -p scripts
   
   # 添加 .gitignore
   # 添加 README.md
   ```

3. **配置仓库设置**
   - 设置分支保护规则
   - 配置 GitHub Actions
   - 设置代码审查要求

#### 1.3 明确工作流和模板

**步骤**:

1. **克隆 develop-rules 仓库**
   ```bash
   git clone https://github.com/org/develop-rules.git
   # 或使用符号链接
   ln -s /path/to/develop-rules ./templates
   ```

2. **熟悉模板结构**
   - 查看 [README.md](../../README.md) 了解所有可用模板
   - 根据项目类型选择合适的模板
   - 准备模板的本地副本

3. **配置 AI 工具**
   - 设置 Cursor/Claude 的工作目录
   - 配置代码库索引
   - 设置上下文窗口和提示词模板

**检查清单**:
- [ ] 所有资料已整理到工作空间
- [ ] GitHub 仓库已创建并初始化
- [ ] 熟悉 develop-rules 中的模板
- [ ] AI 工具已配置完成

---

### 阶段 2: 分析需求阶段

#### 2.1 收集需求信息

**目标**: 全面收集和理解项目需求。

**步骤**:

1. **整理现有需求文档**
   - 业务需求文档
   - 用户故事
   - 功能清单
   - 非功能性需求

2. **识别需求来源**
   - 产品经理提供的需求
   - 用户反馈和调研
   - 竞品分析
   - 技术债务和优化需求

3. **需求分类**
   - 功能需求
   - 非功能需求（性能、安全、可用性等）
   - 技术需求
   - 业务需求

#### 2.2 使用 AI 生成标准化文档

**目标**: 将非结构化需求转换为标准化的项目文档。

**步骤**:

1. **生成 PRD 文档**

   **使用模板**: [`prd/prd-template.md`](../prd/prd-template.md)

   **AI 提示词示例**:
   ```
   请根据以下需求信息，使用 develop-rules/prd/prd-template.md 模板，
   生成完整的产品需求文档：
   
   [粘贴需求信息]
   
   要求：
   1. 填写所有必填字段
   2. 保持模板结构完整
   3. 添加详细的功能描述
   4. 包含非功能性需求
   ```

   **输出**: `docs/prd/prd-v1.0.md`

2. **生成数据库设计文档**

   **使用模板**: [`db/db-template.md`](../db/db-template.md)

   **AI 提示词示例**:
   ```
   根据 PRD 文档，设计数据库表结构，使用 develop-rules/db/db-template.md 模板。
   
   要求：
   1. 分析业务实体和关系
   2. 设计表结构、字段、索引
   3. 包含 ER 图
   4. 添加性能优化建议
   ```

   **输出**: `docs/db/database-design.md`

3. **生成 API 接口文档**

   **使用模板**: [`api/api-template.md`](../api/api-template.md)

   **AI 提示词示例**:
   ```
   根据 PRD 和数据库设计，设计 RESTful API 接口，
   使用 develop-rules/api/api-template.md 模板。
   
   要求：
   1. 设计完整的 CRUD 接口
   2. 定义请求/响应格式
   3. 包含错误处理
   4. 添加认证和授权说明
   ```

   **输出**: `docs/api/api-spec.md`

4. **生成系统架构设计**

   **AI 提示词示例**:
   ```
   根据 PRD 文档，设计系统架构，包括：
   1. 系统架构图（使用 Mermaid 或 PlantUML）
   2. 技术栈选型
   3. 模块划分
   4. 部署架构
   5. 数据流图
   ```

   **输出**: `docs/design/system-architecture.md`

5. **生成网络架构设计**

   **AI 提示词示例**:
   ```
   设计网络架构，包括：
   1. 网络拓扑图
   2. 服务间通信方式
   3. 负载均衡策略
   4. 安全策略
   5. 监控和日志方案
   ```

   **输出**: `docs/design/network-architecture.md`

#### 2.3 文档评审和优化

**步骤**:

1. **AI 辅助评审**
   ```
   请评审以下文档，检查：
   1. 需求是否完整
   2. 设计是否合理
   3. 是否有遗漏或矛盾
   4. 是否符合最佳实践
   ```

2. **人工评审**
   - 产品经理评审 PRD
   - 架构师评审系统设计
   - 开发团队评审技术方案

3. **迭代优化**
   - 根据反馈更新文档
   - 确保文档之间的一致性
   - 版本控制文档变更

**检查清单**:
- [ ] PRD 文档已生成并评审通过
- [ ] 数据库设计文档已完成
- [ ] API 接口文档已设计完成
- [ ] 系统架构图已绘制
- [ ] 网络架构设计已完成
- [ ] 所有文档已提交到仓库

---

### 阶段 3: 项目计划

#### 3.1 创建 GitHub Project

**步骤**:

1. **创建项目看板**
   - 在 GitHub 仓库中创建 Project
   - 设置看板列：待办、进行中、评审中、已完成

2. **配置项目字段**
   - 优先级（高/中/低）
   - 工作量估算（Story Points）
   - 标签（功能、Bug、优化等）
   - 里程碑（Sprint/版本）

#### 3.2 任务分解和 Issue 创建

**目标**: 将项目分解为可管理的任务，每个任务创建对应的 Issue。

**步骤**:

1. **任务分解原则**
   - 每个任务应该可以在 1-3 天内完成
   - 任务之间依赖关系清晰
   - 任务有明确的验收标准

2. **使用 AI 辅助任务分解**

   **AI 提示词示例**:
   ```
   根据以下 PRD 文档，将项目分解为开发任务：
   
   [粘贴 PRD 内容]
   
   要求：
   1. 每个任务包含：标题、描述、验收标准
   2. 标识任务依赖关系
   3. 估算工作量（Story Points）
   4. 分配优先级
   5. 输出格式：Markdown 表格
   ```

3. **创建 Issue**

   **使用模板**: [`github/github-issue-templates/`](../github/github-issue-templates/)

   ```bash
   # 根据任务类型选择合适的 Issue 模板
   # - issue-01-platform.md: 平台相关
   # - issue-02-database.md: 数据库相关
   # - issue-03-sync-mechanism.md: 同步机制
   # - issue-04-performance.md: 性能优化
   # - issue-05-business.md: 业务功能
   # - issue-06-tech-stack.md: 技术栈
   # - issue-07-security.md: 安全相关
   ```

   **Issue 标题格式**: `[类型] 任务描述 [优先级]`

   **示例**:
   - `[功能] 实现用户登录功能 [高]`
   - `[优化] 优化数据库查询性能 [中]`
   - `[Bug] 修复支付接口超时问题 [高]`

4. **关联 Issue 到 Project**

   - 将 Issue 添加到对应的 Project 看板
   - 设置里程碑和标签
   - 分配负责人

#### 3.3 制定开发计划

**步骤**:

1. **Sprint 规划**
   - 确定 Sprint 周期（通常 1-2 周）
   - 选择本 Sprint 要完成的任务
   - 评估团队容量

2. **依赖关系管理**
   - 识别任务依赖
   - 确定开发顺序
   - 标记阻塞任务

3. **里程碑设置**
   - 设置版本里程碑
   - 定义发布计划
   - 设置关键节点

**检查清单**:
- [ ] GitHub Project 已创建
- [ ] 所有任务已分解并创建 Issue
- [ ] Issue 已关联到 Project
- [ ] 开发计划已制定
- [ ] 里程碑已设置

---

### 阶段 4: 代码分析

> **注意**: 此阶段仅适用于已有代码库的项目。新项目可跳过此阶段。

#### 4.1 代码结构分析

**目标**: 理解现有代码结构，识别需要改进的地方。

**步骤**:

1. **使用 AI 分析代码库**

   **AI 提示词示例**:
   ```
   请分析以下代码库的结构：
   
   [提供代码库路径或关键文件]
   
   分析内容：
   1. 项目架构和技术栈
   2. 模块划分和依赖关系
   3. 代码质量和设计模式
   4. 潜在问题和改进建议
   5. 技术债务识别
   ```

2. **生成代码分析报告**

   **输出**: `docs/analysis/code-analysis.md`

   **包含内容**:
   - 架构分析
   - 代码质量评估
   - 技术债务清单
   - 重构建议
   - 性能瓶颈分析

#### 4.2 完善需求文档

**步骤**:

1. **对比代码和需求**
   ```
   对比现有代码实现和 PRD 文档，识别：
   1. 已实现的功能
   2. 缺失的功能
   3. 实现与需求不一致的地方
   4. 需要重构的部分
   ```

2. **更新需求文档**
   - 补充遗漏的需求
   - 修正不准确的需求
   - 添加技术约束
   - 更新非功能性需求

#### 4.3 完善设计文档

**步骤**:

1. **更新架构设计**
   - 根据实际代码更新架构图
   - 补充模块设计说明
   - 更新技术栈信息

2. **更新数据库设计**
   - 对比实际数据库结构
   - 识别需要优化的表
   - 添加索引优化建议

3. **更新 API 文档**
   - 对比实际 API 实现
   - 补充缺失的接口文档
   - 修正接口规范

**检查清单**:
- [ ] 代码分析报告已完成
- [ ] 需求文档已更新
- [ ] 设计文档已完善
- [ ] 技术债务已识别

---

### 阶段 5: 开发阶段

#### 5.1 创建开发分支

**步骤**:

1. **遵循 Git Flow 规范**

   **参考**: [`github/git-flow.md`](../github/git-flow.md)

   ```bash
   # 从 develop 分支创建功能分支
   git checkout develop
   git pull origin develop
   git checkout -b feature/ABC-123-user-login
   
   # 或创建修复分支
   git checkout -b bugfix/ABC-456-fix-timezone
   ```

2. **分支命名规范**
   - 功能分支: `feature/<id>-<desc>`
   - 修复分支: `bugfix/<id>-<desc>`
   - 热修复: `hotfix/<id|version>-<desc>`

#### 5.2 AI 辅助开发

**步骤**:

1. **代码生成**

   **AI 提示词示例**:
   ```
   根据以下需求实现功能：
   
   需求：[描述具体需求]
   技术栈：[Java/TypeScript/Python 等]
   框架：[Spring Boot/Vue/React 等]
   
   要求：
   1. 遵循编码规范（参考 coding/coding.md）
   2. 添加适当的注释（参考 coding/comment.md）
   3. 包含错误处理
   4. 添加单元测试
   ```

2. **代码审查辅助**

   **AI 提示词示例**:
   ```
   请审查以下代码：
   
   [粘贴代码]
   
   检查：
   1. 代码质量和规范
   2. 潜在 Bug
   3. 性能问题
   4. 安全问题
   5. 最佳实践
   ```

3. **重构建议**

   **AI 提示词示例**:
   ```
   请分析以下代码，提供重构建议：
   
   [粘贴代码]
   
   关注点：
   1. 代码可读性
   2. 设计模式应用
   3. 性能优化
   4. 可维护性
   ```

#### 5.3 提交代码

**步骤**:

1. **代码提交规范**

   ```bash
   # 提交前检查
   git status
   git diff
   
   # 提交代码
   git add .
   git commit -m "feat(auth): implement user login [ABC-123]"
   ```

2. **提交信息格式**

   **参考**: [`github/git-flow.md`](../github/git-flow.md)

   ```
   type(scope): summary [ID]
   
   - type: feat | fix | docs | chore | refactor | test | perf
   - scope: 模块名（可选）
   - summary: 简短描述
   - ID: Issue 编号（可选）
   ```

3. **推送分支**

   ```bash
   git push origin feature/ABC-123-user-login
   ```

#### 5.4 创建 Pull Request

**步骤**:

1. **PR 标题格式**
   ```
   type(scope): summary [ID]
   
   示例：
   feat(auth): support SSO login [ABC-123]
   fix(api): handle 204 response correctly [BUG-456]
   ```

2. **PR 描述模板**
   ```markdown
   ## 变更说明
   - [ ] 功能变更
   - [ ] Bug 修复
   - [ ] 性能优化
   - [ ] 代码重构
   
   ## 变更内容
   [描述具体变更]
   
   ## 测试说明
   [描述测试方法和结果]
   
   ## 相关 Issue
   Closes #123
   
   ## 截图/演示
   [如有 UI 变更，添加截图]
   ```

3. **代码审查**
   - 等待代码审查
   - 根据反馈修改代码
   - 解决冲突
   - 通过审查后合并

**检查清单**:
- [ ] 代码已实现并通过本地测试
- [ ] 代码符合编码规范
- [ ] 已添加必要的注释
- [ ] 已创建 Pull Request
- [ ] 代码审查已通过

---

### 阶段 6: 自动化测试

#### 6.1 测试策略

**参考**: [`testing/automation-testing-guide.md`](../testing/automation-testing-guide.md)

**测试类型**:
- **单元测试**: [`testing/ut-testing-guide.md`](../testing/ut-testing-guide.md)
- **集成测试**: [`testing/integration-testing-guide.md`](../testing/integration-testing-guide.md)
- **E2E 测试**: [`testing/e2e-testing-guide.md`](../testing/e2e-testing-guide.md)

#### 6.2 AI 辅助生成测试用例

**步骤**:

1. **生成单元测试**

   **AI 提示词示例**:
   ```
   为以下函数生成单元测试：
   
   [粘贴函数代码]
   
   要求：
   1. 覆盖所有分支
   2. 包含边界条件测试
   3. 包含异常情况测试
   4. 使用合适的测试框架
   ```

2. **生成集成测试**

   **AI 提示词示例**:
   ```
   为以下 API 接口生成集成测试：
   
   [粘贴 API 接口代码]
   
   要求：
   1. 测试正常流程
   2. 测试错误处理
   3. 测试边界条件
   4. 包含性能测试
   ```

3. **生成 E2E 测试**

   **AI 提示词示例**:
   ```
   为以下用户场景生成 E2E 测试：
   
   场景：[描述用户操作流程]
   
   要求：
   1. 使用 Playwright/Cypress
   2. 包含完整的用户操作步骤
   3. 验证关键结果
   4. 包含错误场景
   ```

#### 6.3 配置 CI/CD 测试

**步骤**:

1. **GitHub Actions 配置**

   ```yaml
   # .github/workflows/test.yml
   name: Tests
   
   on:
     pull_request:
       branches: [develop, main]
     push:
       branches: [develop, main]
   
   jobs:
     test:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         - name: Run unit tests
           run: npm test
         - name: Run integration tests
           run: npm run test:integration
         - name: Run E2E tests
           run: npm run test:e2e
   ```

2. **测试覆盖率要求**
   - 单元测试覆盖率 >= 80%
   - 关键业务逻辑覆盖率 >= 90%
   - 集成测试覆盖主要流程

3. **测试报告**
   - 自动生成测试报告
   - 在 PR 中显示测试结果
   - 失败时阻止合并

#### 6.4 测试执行和验证

**步骤**:

1. **本地测试**
   ```bash
   # 运行所有测试
   npm test
   
   # 运行特定测试
   npm test -- --grep "user login"
   
   # 查看覆盖率
   npm run test:coverage
   ```

2. **CI 测试**
   - 提交代码后自动触发
   - 查看测试结果
   - 修复失败的测试

3. **测试评审**
   - 确保测试覆盖关键场景
   - 验证测试质量
   - 优化测试性能

**检查清单**:
- [ ] 单元测试已编写并通过
- [ ] 集成测试已配置
- [ ] E2E 测试已实现
- [ ] CI/CD 测试已配置
- [ ] 测试覆盖率达标
- [ ] 所有测试通过

---

### 阶段 7: 部署阶段

#### 7.1 部署前检查

**步骤**:

1. **代码检查**
   - [ ] 所有测试通过
   - [ ] 代码审查已通过
   - [ ] 无编译错误
   - [ ] 无安全漏洞

2. **文档检查**
   - [ ] README 已更新
   - [ ] API 文档已更新
   - [ ] 变更日志已更新
   - [ ] 部署文档已准备

3. **配置检查**
   - [ ] 环境变量已配置
   - [ ] 数据库迁移脚本已准备
   - [ ] 配置文件已更新
   - [ ] 依赖已更新

#### 7.2 版本发布

**参考**: [`github/git-release.md`](../github/git-release.md)

**步骤**:

1. **创建 Release 分支**

   ```bash
   git checkout develop
   git pull origin develop
   git checkout -b release/1.0.0
   ```

2. **版本号规范**
   - 语义化版本: `vMAJOR.MINOR.PATCH`
   - 示例: `v1.0.0`, `v1.1.0`, `v2.0.0`

3. **创建 Tag**

   ```bash
   git tag -a v1.0.0 -m "Release version 1.0.0"
   git push origin v1.0.0
   ```

4. **创建 GitHub Release**

   - 在 GitHub 上创建 Release
   - 添加 Release Notes
   - 关联相关 Issue 和 PR

#### 7.3 部署执行

**步骤**:

1. **选择部署方式**

   - **自动化部署**: 使用 CI/CD 自动部署
   - **手动部署**: 参考 [`workflow/deploy-terraform.md`](deploy-terraform.md)

2. **部署到测试环境**

   ```bash
   # 示例：使用 Terraform 部署
   cd terraform
   terraform plan
   terraform apply
   ```

3. **验证部署**

   - 健康检查
   - 功能验证
   - 性能测试
   - 监控检查

4. **部署到生产环境**

   - 确认测试环境验证通过
   - 执行生产部署
   - 监控部署过程
   - 验证生产环境

#### 7.4 部署后验证

**步骤**:

1. **功能验证**
   - 验证关键功能正常
   - 检查 API 接口
   - 验证数据一致性

2. **性能监控**
   - 检查响应时间
   - 监控资源使用
   - 查看错误日志

3. **回滚准备**
   - 准备回滚方案
   - 测试回滚流程
   - 保持回滚版本可用

**检查清单**:
- [ ] 部署前检查已完成
- [ ] 版本号已确定并打 Tag
- [ ] GitHub Release 已创建
- [ ] 测试环境部署成功
- [ ] 生产环境部署成功
- [ ] 部署后验证通过
- [ ] 监控和日志正常

---

## 最佳实践

### 1. AI 工具使用

- ✅ **提供清晰上下文**: 在提示词中包含足够的背景信息
- ✅ **迭代优化**: 根据 AI 输出不断优化提示词
- ✅ **人工审查**: AI 生成的代码和文档必须经过人工审查
- ✅ **保持一致性**: 使用统一的提示词模板和格式

### 2. 文档管理

- ✅ **版本控制**: 所有文档纳入 Git 版本控制
- ✅ **及时更新**: 代码变更时同步更新文档
- ✅ **模板规范**: 使用统一的文档模板
- ✅ **评审流程**: 重要文档必须经过评审

### 3. 代码质量

- ✅ **编码规范**: 遵循团队编码规范
- ✅ **代码审查**: 所有代码必须经过审查
- ✅ **测试覆盖**: 保持足够的测试覆盖率
- ✅ **持续重构**: 定期重构技术债务

### 4. 项目管理

- ✅ **任务细化**: 将大任务分解为小任务
- ✅ **及时更新**: 及时更新 Issue 和 Project 状态
- ✅ **沟通协作**: 保持团队沟通畅通
- ✅ **风险识别**: 及时识别和应对风险

---

## 检查清单

### 项目启动检查

- [ ] 工作空间已准备
- [ ] GitHub 仓库已创建
- [ ] 模板已熟悉
- [ ] AI 工具已配置

### 需求分析检查

- [ ] PRD 文档已完成
- [ ] 数据库设计已完成
- [ ] API 文档已完成
- [ ] 架构设计已完成
- [ ] 所有文档已评审

### 开发检查

- [ ] 开发分支已创建
- [ ] 代码已实现
- [ ] 代码已审查
- [ ] PR 已合并

### 测试检查

- [ ] 单元测试已通过
- [ ] 集成测试已通过
- [ ] E2E 测试已通过
- [ ] 测试覆盖率达标

### 部署检查

- [ ] 版本号已确定
- [ ] Release 已创建
- [ ] 部署已成功
- [ ] 验证已通过

---

## 相关资源

### 模板文档

- [PRD 模板](../prd/prd-template.md)
- [API 接口文档模板](../api/api-template.md)
- [数据库设计模板](../db/db-template.md)
- [测试指南](../testing/automation-testing-guide.md)

### 工作流文档

- [Git Flow 规范](../github/git-flow.md)
- [版本发布规范](../github/git-release.md)
- [Terraform 部署工作流](deploy-terraform.md)
- [编译错误修复工作流](fix-compilation-errors.md)

### 编码规范

- [编码规范](../coding/coding.md)
- [代码注释规范](../coding/comment.md)

### 日志配置

- [多服务 Datadog 配置](../logging/01-多服务-datadog-分布式追踪配置模板.md)
- [单服务日志配置](../logging/02-单服务日志配置模板.md)

---

## 常见问题

### Q1: AI 生成的代码质量如何保证？

**A**: 
1. 必须经过人工代码审查
2. 运行所有测试确保功能正确
3. 遵循团队编码规范
4. 使用静态代码分析工具

### Q2: 如何选择合适的 PRD 模板？

**A**: 
- **标准模板**: 适用于大多数项目
- **敏捷模板**: 适用于 Scrum/Kanban 团队
- **精益模板**: 适用于 MVP 和快速验证
- **瀑布模板**: 适用于传统项目

### Q3: 如何处理 AI 生成文档的准确性？

**A**:
1. 提供准确和完整的上下文
2. 人工审查和验证所有信息
3. 与相关干系人确认
4. 迭代优化文档内容

### Q4: 测试覆盖率要求是多少？

**A**:
- 单元测试覆盖率 >= 80%
- 关键业务逻辑 >= 90%
- 集成测试覆盖主要流程
- E2E 测试覆盖核心用户场景

---

**文档版本**: v1.0.0  
**最后更新**: 2025-01-20  
**维护团队**: 开发规范团队
