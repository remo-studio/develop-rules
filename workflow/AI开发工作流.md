# AI辅助开发方案

## 第一部分：各阶段AI生成成果物一览

| 阶段 | AI生成成果物 | 格式 | 说明 |
|------|------------|------|------|
| **需求定义** | PRD（产品需求文档） | Markdown | 用户故事、功能需求、非功能需求、约束条件 |
| | 任务分解・WBS | GitHub Issue | 分阶段任务清单、工数估算、依赖关系 |
| | 领域模型分析 | PlantUML + Markdown | 领域实体、限界上下文、业务规则 |
| **UI设计** | 画面流程图 | PlantUML | 画面跳转、用户流程 |
| | UI组件设计 | Vue.js (.vue) | 表单、表格、弹窗的实现代码 |
| | API类型定义 | TypeScript (.d.ts) | 请求/响应的类型定义 |
| **系统设计** | 系统架构图 | PlantUML | 组件图、时序图、部署图 |
| | 数据库设计文档 | Markdown + SQL | ER图、表定义、DDL、迁移脚本 |
| | API设计文档 | Markdown | 接口清单、请求/响应规格 |
| **详细设计** | 类设计 | Java / PHP | Entity、DTO、Mapper、Service、Controller的实现 |
| | 迁移计划 | Markdown + SQL | 4阶段执行指南（dev→test→preprod→prod） |
| | 回滚方案 | SQL + Markdown | 紧急恢复脚本、操作手册 |
| **开发** | 后端代码 | Java (Spring Boot) | Entity、Mapper、Service、Controller、单元测试 |
| | 前端代码 | Vue.js + TypeScript | API层、页面组件、路由配置 |
| | 批处理 | PHP / Java | 结算、自动购买、收益计算等定时任务 |
| | 数据库迁移 | SQL (Flyway) | 版本管理的Schema变更脚本 |
| **测试** | 单元测试 | JUnit / Jest | Service层、DTO校验、Entity校验 |
| | 集成测试 | Spring Boot Test | API集成测试、数据库操作测试 |
| | 测试数据 | SQL | 迁移用初始数据、测试用数据集 |
| **发布** | Docker环境配置 | docker-compose.yml | 本地开发环境（MySQL、Redis、Backend） |
| | 部署手册 | Markdown | 各环境配置、发布检查清单 |
| | PR与提交 | Git | 变更概要、附带测试计划的PR创建 |

---

## 第二部分：料金计划调整项目的实现方案

### 2.1 项目概述

**料金计划调整项目**将充レン（Juren）的收费体系从固定费率模型迁移到基于时段的分段计费模型。通过AI辅助开发，高效推进64天、15人规模的项目。

### 2.2 各阶段AI应用详情

#### Phase 1：需求定义

**输入**: 业务需求（费率改定背景、新收费体系概要）

**AI实施内容**:
- 根据业务需求生成PRD（`juren-documents/prd/` 目录下）
- 分解为9个阶段、98个任务的WBS（GitHub Issue #16）
- 按仓库拆分开发任务（Issue #21～#23）
- 工数估算（Backend 49天、Web 13.5天、Admin 12天）

**生成成果物**:
```
juren-documents/
├── prd/
│   ├── 功能模块划分文档.md          # 功能模块划分
│   └── Wi-Fi项目任务划分与成果物定义.md  # 任务分解与成果物定义
```

**AI工具**: Claude Code（PRD生成）、GitHub Issue（任务管理）

---

#### Phase 2：系统设计

**输入**: PRD、现有数据库Schema（65张表）、现有代码库

**AI实施内容**:
1. **现有代码库分析**: Explore Agent解析现有的架构模式
2. **数据库设计**: 采用不可变版本化方案的新Schema设计
3. **架构图生成**: 通过PlantUML自动生成系统全景图

**生成成果物**:
```
database/docs/
├── 数据库设计-料金改订.md           # 数据库设计文档v2.0（不可变版本化）
├── release-map.md                  # 发布路线图

docs/db/
├── pricing-plan-schema-changes.md  # Schema变更规格（含v1.0→v2.0的设计演进）
├── MIGRATION_GUIDE.md              # 4阶段迁移指南
├── migration_001_create_pricing_tables.sql    # 新建表（5张）
├── migration_002_alter_existing_tables.sql    # 修改现有表（4张）
├── migration_003_insert_default_data.sql      # 初始数据导入
├── migration_rollback.sql                     # 紧急回滚脚本
└── README_PRICING_PLAN_CHANGES.md             # 文档索引

plantuml/
├── system-architecture-overview.puml          # 系统架构图
├── database-erd.puml                          # ER图
└── *.puml                                     # 时序图（8种以上）
```

**设计决策记录**: AI分析并提出了从v1.0（JSON快照方式）到v2.0（不可变版本化方式）的设计变更，最终采用 `parent_id` + `is_current` 的历史记录管理模式。

---

#### Phase 3：详细设计与开发

**输入**: 数据库设计文档、现有代码的架构模式

**AI实施内容**:

##### 后端（Java Spring Boot）

AI通过Explore Agent分析现有代码库，生成符合项目规范的代码:

```
juren-backend/src/main/java/com/greenutility/juren/pricingplan/
├── model/
│   ├── entity/
│   │   ├── PricingPlan.java              # 料金计划Entity（支持版本化）
│   │   └── PricingPlanSegment.java       # 计费段Entity（A1～D, R）
│   └── dto/
│       ├── PricingPlanSaveDTO.java        # 保存用DTO（含数据校验）
│       └── PricingPlanDetailDTO.java      # 详情展示用DTO
├── dao/
│   ├── PricingPlanMapper.java            # MyBatis Mapper
│   └── PricingPlanSegmentMapper.java     # 计费段Mapper
├── service/
│   └── PricingPlanService.java           # 业务逻辑
│       # - 不可变版本化（创建新版本，停用旧版本）
│       # - 自动税额计算（含税/不含税转换）
│       # - Location批量迁移
└── controller/ (集成到ManagerController)
```

##### 前端（Vue.js + TypeScript）

```
juren-admin-front-vue/src/
├── api/system/manager/
│   ├── index.ts                          # API调用函数
│   └── model.d.ts                        # TypeScript类型定义
└── views/acti/pricing-plan/
    ├── index.vue                         # 料金计划列表页
    ├── columns.tsx                       # 表格列定义
    └── components/
        └── plan-dialog.vue               # 创建/编辑弹窗
```

##### 数据库迁移（Flyway）

```
database/
├── sql/
│   ├── V1__create_pricing_tables.sql     # 新建表
│   ├── V2__alter_existing_tables.sql     # 修改现有表（幂等处理）
│   └── V3__insert_default_data.sql       # 导入测试数据
└── conf/
    ├── dev.conf                          # 开发环境配置
    ├── local.conf                        # 本地Docker环境配置
    └── test.conf                         # 测试环境配置
```

**开发流程**:
1. Explore Agent解析现有代码模式（BaseController、ApiResponse等）
2. AI按照现有模式生成代码
3. Code Reviewer Agent执行代码审查
4. Build Error Resolver Agent解决构建错误

---

#### Phase 4：测试

**输入**: 实现代码、业务规则

**AI实施内容**:
- TDD Guide Agent驱动测试先行开发
- 自动生成单元测试（Service层、Entity、DTO）
- 生成测试数据（3个默认料金计划、8个计费段配置）

**生成成果物**:
```
juren-backend/src/test/java/com/greenutility/juren/pricingplan/
├── service/
│   └── PricingPlanServiceTest.java       # Service层测试
│       # - 版本化测试
│       # - 税额计算测试
│       # - Location迁移测试
└── model/
    ├── PricingPlanSegmentTest.java        # Entity测试
    └── PricingPlanSaveDTOTest.java        # DTO校验测试
```

**测试覆盖率目标**: 80%以上

---

#### Phase 5：环境搭建与发布

**输入**: docker-compose.yml、环境配置文件

**AI实施内容**:
- Docker环境搭建（MySQL 8.0 + Redis）
- Flyway自动迁移执行
- 前端↔后端连接配置
- PR创建（含变更概要 + 测试计划）

**生成成果物**:
```
juren-backend/
├── docker-compose.yml                    # 本地开发环境
└── .env.development                      # 环境变量配置

# GitHub PR
├── #2  Backend Java工具类实现 (pricing-plan-backend → dev)
├── #183 Web PHP函数库实现 (pricing-plan-web → dev)
└── #168 Admin管理画面实现 (pricing-plan-admin → dev)
```

---

### 2.3 AI Agent协作模型

本项目中多个AI Agent协作执行任务:

```
┌─────────────────────────────────────────────────────────┐
│                    Claude Code (Main)                     │
│                    任务协调与决策                          │
├─────────────┬──────────────┬──────────────┬──────────────┤
│  Explore    │   Planner    │  TDD Guide   │ Code Review  │
│  Agent      │   Agent      │  Agent       │ Agent        │
│             │              │              │              │
│  代码库     │  实现计划     │  测试先行    │  代码质量    │
│  分析与搜索  │  任务分解     │  开发       │  审查        │
├─────────────┼──────────────┼──────────────┼──────────────┤
│  Architect  │  Security    │ Build Error  │  Backend     │
│  Agent      │  Reviewer    │ Resolver     │  Engineer    │
│             │              │              │              │
│  系统设计   │  安全检查     │  构建错误    │  后端实现    │
│  决策       │              │  自动修复    │              │
└─────────────┴──────────────┴──────────────┴──────────────┘
```

### 2.4 AI应用效果

| 指标 | 传统方式 | AI辅助 | 削减率 |
|------|---------|--------|--------|
| 数据库设计文档编写 | 3～5天 | 0.5天 | 85% |
| 迁移脚本编写 | 2～3天 | 0.5天 | 80% |
| CRUD实现（后端） | 5～7天 | 1天 | 80% |
| CRUD实现（前端） | 3～5天 | 0.5天 | 85% |
| 单元测试编写 | 3～5天 | 0.5天 | 85% |
| 代码审查 | 1～2天/次 | 即时 | 90% |
| 环境搭建 | 1～2天 | 0.5天 | 65% |

### 2.5 质量保障机制

```
代码生成 → Code Review Agent → Security Review → 测试执行 → PR创建
    ↑                                                     │
    └─────────────── 反馈循环 ────────────────────────────┘
```

1. **生成阶段**: 遵循现有代码库的模式（Explore Agent预先分析）
2. **审查阶段**: Code Reviewer Agent自动审查（CRITICAL/HIGH/MEDIUM三级分类）
3. **安全检查**: Security Reviewer Agent在提交前执行安全扫描
4. **测试阶段**: TDD Guide Agent驱动测试先行开发（覆盖率80%以上）
5. **构建阶段**: Build Error Resolver Agent自动修复构建错误

### 2.6 记忆与上下文管理

AI利用**跨会话记忆系统**，在不同会话间保持以下信息:

- 设计决策的来龙去脉（v1.0 JSON快照 → v2.0 不可变版本化的变更原因）
- Git Worktree的配置状态与修复历史
- 环境配置信息（数据库连接、Docker设置）
- 代码库的结构与现有模式

由此实现跨会话的一致性开发，上下文重建成本**削减91%**。
