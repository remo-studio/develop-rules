## 📋 信息收集项: 平台与环境信息

**优先级**: P0 (必填)
**截止日期**: _待设定_
**负责人**: @_待分配_

---

### ✅ 任务清单 (Task List)

- [ ] **1.1 平台清单** - 收集所有平台的基本信息 (负责人: @___)
- [ ] **1.2 服务部署清单** - 列出每个平台部署的服务 (负责人: @___)
- [ ] **1.3 网络拓扑** - 绘制平台间网络连接关系 (负责人: @___)
- [ ] **1.4 访问凭证与权限** - 确认跨平台访问能力 (负责人: @___)

> **进度**: GitHub 会自动显示完成进度 (例如: 1/4 完成)

---

### 📝 填写指南

#### 1.1 平台清单
请提供以下信息:
- 平台总数
- 每个平台的详细信息(名称、云服务商、区域、用途、网络类型)

#### 1.2 服务部署清单
对于每个平台,列出:
- 部署的服务名称
- 服务类型(Web/API/Worker/Other)
- 编程语言和框架版本

#### 1.3 网络拓扑
说明:
- 平台间如何连接(公网/VPN/云专线/隔离)
- 网络限制和特殊说明
- (可选)绘制简单的网络拓扑图

#### 1.4 访问凭证与权限
确认:
- 是否有跨平台访问权限
- 当前使用的访问方式
- 访问限制

---

### 💬 收集结果

**请在下方评论中填写收集到的信息**

#### 回复格式示例:

```markdown
### 1.1 平台清单 - @username

**平台总数**: 3 个

| 平台名称 | 云服务商 | 区域/Region | 用途说明 | 网络类型 |
|---------|---------|------------|---------|---------|
| Production-Platform | AWS | ap-northeast-1 | 生产环境-核心业务 | VPC |
| Analytics-Platform | GCP | asia-northeast1 | 数据分析平台 | VPC |
| Legacy-Platform | 自建IDC | - | 遗留系统 | 物理网络 |

**备注**: [任何补充说明]
```

```markdown
### 1.2 服务部署清单 - @username

**Production-Platform (AWS)**:
- 服务名称: user-service
- 服务类型: API
- 编程语言: PHP 7.4
- 框架版本: Laravel 8.x

**Analytics-Platform (GCP)**:
- 服务名称: analytics-worker
- 服务类型: Worker
- 编程语言: Python 3.9
- 框架版本: -
```

```markdown
### 1.3 网络拓扑 - @username

**平台间网络连接方式**:
- ✅ Production-Platform ↔ Analytics-Platform: 云专线(Direct Connect)
- ❌ Production-Platform ↔ Legacy-Platform: 完全隔离,无直接连接
- ⚠️ Analytics-Platform ↔ Legacy-Platform: 通过VPN,带宽受限

**网络限制说明**:
Legacy-Platform 位于自建 IDC,与云平台之间需通过 VPN 访问,带宽仅 100Mbps。
防火墙规则严格,仅允许特定 IP 段访问。
```

```markdown
### 1.4 访问凭证与权限 - @username

**是否有跨平台访问权限**: ⚠️ 部分有

**当前访问方式**:
- AWS → GCP: 通过 API Gateway + IAM 角色
- AWS → Legacy: 通过 VPN + Bastion Host (跳板机)
- GCP → Legacy: 无直接访问能力

**访问限制**:
Legacy-Platform 对外访问需要通过安全审批流程,通常需要 2-3 个工作日。
```

---

### 📊 数据收集表格 (可选)

如果信息较多,可以使用表格形式:

**平台清单表格**:

| 字段 | Platform-A | Platform-B | Platform-C |
|------|-----------|-----------|-----------|
| 云服务商 | | | |
| 区域 | | | |
| 用途 | | | |
| 网络类型 | | | |
| 主要服务数量 | | | |

---

### 🔗 相关文档

- [信息收集清单完整模板](../../agents/task-0001/信息收集清单0001.md#1%EF%B8%8F%E2%83%A3-%E5%B9%B3%E5%8F%B0%E4%B8%8E%E7%8E%AF%E5%A2%83%E4%BF%A1%E6%81%AF--platform--environment-information)
- [需求文档](../../agents/task-0001/需求.md)

---

### 💡 提示

- 如有疑问,请在评论中 @ 相关负责人讨论
- 可以上传截图或附件辅助说明
- 完成填写后,请勾选上方对应的任务清单
- 所有子任务完成后,Issue 将自动进入"待审核"状态
