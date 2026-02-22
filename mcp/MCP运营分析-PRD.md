# MCP 运营分析系统 PRD

> **Model Context Protocol (MCP) Operations Analytics System**
>
> Version: 1.0
> Created: 2025-02-15
> Status: 已实现

---

## 1. 概述

### 1.1 背景

随着 Juren 充电宝租赁业务规模扩大，运营团队需要更高效的方式获取和分析业务数据。传统的数据查询方式依赖 SQL 或预设报表，存在以下问题：

- 非技术人员难以自主获取数据
- 定制化查询需要开发介入
- 数据分析效率低下

### 1.2 解决方案

基于 MCP (Model Context Protocol) 协议，构建 REST API 接口，使 AI 助手（如 Claude Desktop）能够直接查询运营数据，实现：

- **自然语言查询**：用户用自然语言描述需求，AI 自动调用相应工具
- **实时数据访问**：直接连接生产数据库，获取最新数据
- **多维度分析**：覆盖设备、用户、收入、租借四大维度

### 1.3 目标用户

| 角色 | 使用场景 |
|------|----------|
| 运营经理 | 日常业务监控、KPI 追踪 |
| 客服团队 | 客户问题排查、租借历史查询 |
| 财务人员 | 收入统计、支付渠道分析 |
| 技术运维 | 设备状态监控、故障排查 |

---

## 2. 功能模块

### 2.1 设备巡检与维护

| 工具名称 | 功能描述 | 典型问题 |
|----------|----------|----------|
| `get_station_status` | 站点在线状态统计 | "目前有多少站点在线？" |
| `get_abnormal_stations` | 异常站点列表 | "哪些站点处于故障状态？" |
| `get_high_traffic_stations` | 高流量站点排名 | "今天租借量最高的站点是哪些？" |
| `get_low_inventory_stations` | 低库存站点 | "哪些站点电池不足需要补充？" |
| `get_failure_reports` | 待处理故障报告 | "有多少未处理的故障报告？" |

### 2.2 用户与账户管理

| 工具名称 | 功能描述 | 典型问题 |
|----------|----------|----------|
| `get_user_stats` | 用户统计概览 | "本月新增了多少用户？" |
| `get_active_user_stats` | 活跃用户统计 (DAU/WAU/MAU) | "今天有多少活跃用户？" |
| `get_frequent_users` | 高频用户列表 | "租借次数最多的用户是谁？" |
| `get_pending_refunds` | 待处理退款 | "有多少笔退款待处理？" |

### 2.3 收入与财务

| 工具名称 | 功能描述 | 典型问题 |
|----------|----------|----------|
| `get_revenue_stats` | 收入统计概览 | "本月收入是多少？" |
| `get_payment_breakdown` | 支付渠道分布 | "PayPay 占比多少？" |
| `get_billing_type_breakdown` | 计费类型分布 | "延长费收入占多少？" |
| `get_daily_revenue_trend` | 每日收入趋势 | "最近7天的收入趋势如何？" |
| `get_top_locations` | Location 收益排名 | "哪个场所收益最高？" |

### 2.4 租借统计

| 工具名称 | 功能描述 | 典型问题 |
|----------|----------|----------|
| `get_rental_stats` | 租借统计概览 | "今天有多少笔租借？" |
| `get_active_rentals` | 当前未归还租借 | "有多少用户还没还电池？" |
| `get_customer_rental_history` | 客户租借历史 | "用户 xxx 的租借记录" |

---

## 3. 技术实现

### 3.1 架构设计

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Claude Desktop │────▶│  MCP REST API   │────▶│  MySQL Database │
│  (MCP Client)   │     │  (juren-backend)│     │  (Juren_Prod)   │
└─────────────────┘     └─────────────────┘     └─────────────────┘
         │                       │
         │                       ▼
         │              ┌─────────────────┐
         └──────────────│   Markdown      │
                        │   Response      │
                        └─────────────────┘
```

### 3.2 技术栈

| 组件 | 技术 |
|------|------|
| 后端框架 | Spring Boot 2.2 |
| ORM | MyBatis Plus 3.4 |
| 数据库 | MySQL 8.0 |
| 协议 | MCP over REST |
| 响应格式 | Markdown 表格 |

### 3.3 代码位置

```
juren-backend/
└── src/main/java/com/greenutility/adsread/mcp/
    ├── controller/
    │   └── McpController.java          # REST 接口
    ├── service/
    │   └── McpToolsService.java        # 工具注册与调用
    ├── dao/
    │   └── OperationAnalyticsMapper.java  # SQL 查询
    └── model/
        ├── McpToolDefinition.java      # 工具定义
        ├── McpToolCallRequest.java     # 请求模型
        ├── McpToolCallResponse.java    # 响应模型
        └── McpContent.java             # 内容模型
```

---

## 4. 使用方式

### 4.1 直接 API 调用

```bash
# 获取服务信息
curl http://localhost:20000/mcp/info

# 列出所有工具
curl http://localhost:20000/mcp/tools/list

# 调用工具
curl -X POST http://localhost:20000/mcp/tools/call \
  -H "Content-Type: application/json" \
  -d '{"name":"get_station_status","arguments":{}}'
```

### 4.2 Claude Desktop 集成

在 Claude Desktop 配置中添加 MCP Server：

```json
{
  "mcpServers": {
    "juren-operations": {
      "type": "http",
      "url": "http://your-server:20000/mcp"
    }
  }
}
```

### 4.3 自然语言示例

| 用户提问 | 调用工具 | 返回结果 |
|----------|----------|----------|
| "今天的站点在线率是多少？" | `get_station_status` | 总站点: 10,852 / 在线率: 28.53% |
| "本周收入多少钱？" | `get_revenue_stats` | 本周收入: ¥1,614,910 |
| "PayPay 支付占比多少？" | `get_payment_breakdown` | PayPay: 61.52% |
| "有多少用户还没还电池？" | `get_active_rentals` | 返回未归还租借列表 |

---

## 5. 数据安全

### 5.1 访问控制

- MCP 接口部署在内网，不对外暴露
- 生产环境通过 VPN 或内网访问
- 可配置 Token 认证 (`need-token: true`)

### 5.2 数据脱敏

- 用户手机号、邮箱等敏感信息不在响应中完整展示
- LineID 等标识符用于定位，不展示详细个人信息

### 5.3 审计日志

- 所有 MCP 调用记录在应用日志中
- 包含调用时间、工具名称、参数

---

## 6. 后续计划

### 6.1 短期 (Q1 2025)

- [ ] 增加更多分析维度（小时级趋势、地区分布）
- [ ] 支持数据导出 (CSV/Excel)
- [ ] 增加缓存层提升查询性能

### 6.2 中期 (Q2 2025)

- [ ] 集成到管理后台 Dashboard
- [ ] 支持定时报表自动生成
- [ ] 增加异常告警触发

### 6.3 长期

- [ ] 构建 AI 驱动的智能运营助手
- [ ] 预测性分析（收入预测、库存预测）
- [ ] 自动化运营建议生成

---

## 7. 附录

### 7.1 计费类型代码说明

| 代码 | 日文 | 中文 | 金额 |
|------|------|------|------|
| I | 初回 | 初次租借 | ¥330 |
| E1 | 延長1 | 延长1 (24h) | ¥330 |
| E2 | 延長2 | 延长2 (48h) | ¥330 |
| P | 買取 | 购买 | ¥5,500 |
| R | 赤伝 | 红传 (退款) | - |
| B | 黒伝 | 黑传 | - |

### 7.2 支付方式代码说明

| 代码 | 说明 |
|------|------|
| C | 信用卡 (GMO) |
| D | Docomo 运营商支付 |
| A | Au 运营商支付 |
| S | Softbank 运营商支付 |
| L | LINE Pay |
| P | PayPay |

### 7.3 站点状态代码说明

| 状态 | 说明 |
|------|------|
| A | Active - 正常运营 |
| S | Stopped - 暂停服务 |
| M | Maintenance - 维护中 |
| E | Error - 故障 |
| D | Deleted - 已删除 |
