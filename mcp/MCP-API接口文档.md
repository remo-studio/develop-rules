# MCP 运营分析 API 接口文档

> **Juren Operations MCP REST API Reference**
>
> Version: 1.0.0
> Base URL: `http://localhost:20000/mcp`
> Protocol Version: 2024-11-05

---

## 目录

1. [认证说明](#1-认证说明)
2. [基础接口](#2-基础接口)
3. [工具调用接口](#3-工具调用接口)
4. [快捷查询接口](#4-快捷查询接口)
5. [工具详细说明](#5-工具详细说明)
6. [错误处理](#6-错误处理)
7. [示例代码](#7-示例代码)
8. [更新日志](#8-更新日志)

---

## 1. 认证说明

### 1.1 认证方式

MCP API 支持 API Key + 签名认证。认证通过 HTTP Header 传递：

| Header | 必填 | 说明 |
|--------|------|------|
| `X-MCP-Key` | 是 | API Key ID |
| `X-MCP-Timestamp` | 是 | 请求时间戳 (毫秒) |
| `X-MCP-Signature` | 否* | HMAC-SHA256 签名 |

> *签名在启用签名验证时必填

### 1.2 签名算法

```
签名内容 = METHOD + "\n" + URI + "\n" + TIMESTAMP
签名结果 = Base64(HMAC-SHA256(签名内容, KeySecret))
```

**Python 示例**

```python
import hmac
import hashlib
import base64
import time

def generate_signature(method, uri, key_secret):
    timestamp = str(int(time.time() * 1000))
    content = f"{method}\n{uri}\n{timestamp}"
    signature = base64.b64encode(
        hmac.new(
            key_secret.encode('utf-8'),
            content.encode('utf-8'),
            hashlib.sha256
        ).digest()
    ).decode('utf-8')
    return timestamp, signature

# 使用示例
timestamp, signature = generate_signature(
    "POST",
    "/mcp/tools/call",
    "mcp-secret-key-for-claude-2025"
)
```

### 1.3 请求示例

```bash
curl -X POST http://localhost:20000/mcp/tools/call \
  -H "Content-Type: application/json" \
  -H "X-MCP-Key: claude-desktop" \
  -H "X-MCP-Timestamp: 1708012800000" \
  -H "X-MCP-Signature: abc123..." \
  -d '{"name":"get_station_status","arguments":{}}'
```

### 1.4 认证错误响应

| HTTP 状态码 | 错误信息 | 说明 |
|------------|----------|------|
| 401 | Missing X-MCP-Key header | 缺少 API Key |
| 401 | Invalid API Key | API Key 无效 |
| 401 | Request expired | 请求已过期 |
| 401 | Invalid signature | 签名验证失败 |
| 403 | IP not allowed | IP 不在白名单 |

### 1.5 配置说明

在 `application.yml` 中配置：

```yaml
mcp:
  security:
    enabled: true                    # 启用认证
    signature-enabled: true          # 启用签名验证
    signature-expire-seconds: 300    # 签名有效期 (秒)
    ip-whitelist:                    # IP 白名单
      - "127.0.0.1"
      - "10.0.0.0/8"
    api-keys:                        # API Keys 列表
      - key-id: "claude-desktop"
        key-secret: "your-secret-key"
        client-name: "Claude Desktop"
        active: true
```

---

## 2. 基础接口

### 2.1 获取服务信息

获取 MCP 服务器基本信息和能力。

**请求**

```http
GET /mcp/info
```

**响应**

```json
{
  "code": 200,
  "msg": "操作成功",
  "data": {
    "name": "juren-operations-mcp",
    "version": "1.0.0",
    "description": "Juren 充电宝租赁系统运营分析 MCP 服务",
    "protocol_version": "2024-11-05",
    "capabilities": {
      "tools": true,
      "resources": false,
      "prompts": false
    }
  }
}
```

---

### 2.2 获取工具列表

获取所有可用的 MCP 工具定义。

**请求**

```http
GET /mcp/tools/list
```

**响应**

```json
{
  "code": 200,
  "msg": "操作成功",
  "data": [
    {
      "name": "get_station_status",
      "description": "获取站点在线状态统计，包括总站点数、在线数、离线数、在线率",
      "inputSchema": {
        "type": "object",
        "properties": {},
        "required": []
      }
    },
    {
      "name": "get_revenue_stats",
      "description": "获取收入统计概览，包括今日、本周、本月收入",
      "inputSchema": {
        "type": "object",
        "properties": {},
        "required": []
      }
    }
    // ... 更多工具
  ]
}
```

---

## 3. 工具调用接口

### 3.1 调用工具

通用的工具调用接口。

**请求**

```http
POST /mcp/tools/call
Content-Type: application/json

{
  "name": "工具名称",
  "arguments": {
    // 工具参数
  }
}
```

**响应**

```json
{
  "code": 200,
  "msg": "操作成功",
  "data": {
    "isError": false,
    "content": [
      {
        "type": "text",
        "text": "## 站点在线状态统计\n\n- **总站点数**: 10852\n...",
        "mimeType": null,
        "data": null
      }
    ]
  }
}
```

**错误响应**

```json
{
  "code": 200,
  "msg": "操作成功",
  "data": {
    "isError": true,
    "content": [
      {
        "type": "text",
        "text": "Error: Unknown tool: invalid_tool_name"
      }
    ]
  }
}
```

---

## 4. 快捷查询接口

提供常用查询的直接访问接口，无需构造 JSON 请求体。

| 接口 | 方法 | 说明 |
|------|------|------|
| `/mcp/quick/station-status` | GET | 站点状态概览 |
| `/mcp/quick/user-stats` | GET | 用户统计概览 |
| `/mcp/quick/revenue-stats` | GET | 收入统计概览 |
| `/mcp/quick/rental-stats` | GET | 租借统计概览 |
| `/mcp/quick/dashboard` | GET | 综合仪表盘 |

**示例**

```bash
curl http://localhost:20000/mcp/quick/dashboard
```

---

## 5. 工具详细说明

### 5.1 设备巡检类

#### get_station_status

获取站点在线状态统计。

**参数**: 无

**示例请求**

```json
{
  "name": "get_station_status",
  "arguments": {}
}
```

**示例响应**

```markdown
## 站点在线状态统计

- **总站点数**: 10852
- **在线站点**: 3096
- **离线站点**: 8
- **在线率**: 28.53%
```

---

#### get_abnormal_stations

获取异常站点列表（停止/维护/故障状态）。

**参数**

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| limit | integer | 否 | 20 | 返回数量限制 |

**示例请求**

```json
{
  "name": "get_abnormal_stations",
  "arguments": {"limit": 10}
}
```

---

#### get_high_traffic_stations

获取今日高流量站点排名。

**参数**

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| limit | integer | 否 | 10 | 返回数量限制 |

---

#### get_low_inventory_stations

获取电池库存不足的站点。

**参数**

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| threshold | integer | 否 | 2 | 库存阈值 |
| limit | integer | 否 | 20 | 返回数量限制 |

---

#### get_failure_reports

获取待处理的故障报告。

**参数**

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| limit | integer | 否 | 20 | 返回数量限制 |

---

### 5.2 用户管理类

#### get_user_stats

获取用户统计概览。

**参数**: 无

**示例响应**

```markdown
## 用户统计概览

- **总用户数**: 1478917
- **今日新增**: 0
- **本周新增**: 1310
- **本月新增**: 24752
```

---

#### get_active_user_stats

获取活跃用户统计（DAU/WAU/MAU）。

**参数**: 无

**示例响应**

```markdown
## 活跃用户统计

- **日活 (DAU)**: 1234
- **周活 (WAU)**: 8567
- **月活 (MAU)**: 35420
```

---

#### get_frequent_users

获取高频用户列表。

**参数**

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| minRentals | integer | 否 | 10 | 最小租借次数 |
| limit | integer | 否 | 20 | 返回数量限制 |

---

#### get_pending_refunds

获取待处理的退款列表。

**参数**

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| limit | integer | 否 | 20 | 返回数量限制 |

---

### 5.3 收入财务类

#### get_revenue_stats

获取收入统计概览。

**参数**: 无

**示例响应**

```markdown
## 收入统计

- **今日收入**: ¥0 (0笔)
- **本周收入**: ¥1614910
- **本月收入**: ¥30214360
```

---

#### get_payment_breakdown

获取支付渠道分布。

**参数**

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| days | integer | 否 | 30 | 统计天数 |

**示例响应**

```markdown
## 支付渠道分布 (近30天)

| 支付方式 | 笔数 | 金额 | 占比 |
|----------|------|------|------|
| PayPay | 46338 | ¥18338870 | 61.52% |
| 信用卡(GMO) | 19635 | ¥7819680 | 26.07% |
| Docomo | 3170 | ¥1274130 | 4.21% |
| Au | 2517 | ¥1130800 | 3.34% |
| Softbank | 2004 | ¥880660 | 2.66% |
```

---

#### get_billing_type_breakdown

获取计费类型分布。

**参数**

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| days | integer | 否 | 30 | 统计天数 |

---

#### get_daily_revenue_trend

获取每日收入趋势。

**参数**

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| days | integer | 否 | 7 | 统计天数 |

---

#### get_top_locations

获取 Location 收益排名。

**参数**

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| days | integer | 否 | 30 | 统计天数 |
| limit | integer | 否 | 10 | 返回数量限制 |

---

### 5.4 租借统计类

#### get_rental_stats

获取租借统计概览。

**参数**: 无

**示例响应**

```markdown
## 租借统计概览

- **历史总租借**: 3323965
- **今日租借**: 0
- **本周租借**: 2948
- **已完成**: 3270685
- **逾期未还**: 53271
```

---

#### get_active_rentals

获取当前未归还的租借列表。

**参数**

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| limit | integer | 否 | 50 | 返回数量限制 |

**示例响应**

```markdown
## 当前未归还租借

| 租借ID | 客户 | 电池ID | 租借站点 | 已用时间 |
|--------|------|--------|----------|----------|
| J102440286265 | hirose.takuya | 00004442 | 東京駅 | 52321小时 |
| J102440286267 | HOJO | 00004384 | 渋谷駅 | 52321小时 |
```

---

#### get_customer_rental_history

查询指定客户的租借历史。

**参数**

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| customerId | integer | **是** | - | 客户ID |
| limit | integer | 否 | 20 | 返回数量限制 |

**示例请求**

```json
{
  "name": "get_customer_rental_history",
  "arguments": {"customerId": 12345, "limit": 10}
}
```

---

## 6. 错误处理

### 6.1 HTTP 状态码

| 状态码 | 说明 |
|--------|------|
| 200 | 请求成功 |
| 400 | 请求参数错误 |
| 404 | 接口不存在 |
| 500 | 服务器内部错误 |

### 6.2 业务错误

业务错误通过 `isError` 字段标识：

```json
{
  "code": 200,
  "data": {
    "isError": true,
    "content": [
      {
        "type": "text",
        "text": "Error: Unknown tool: invalid_tool"
      }
    ]
  }
}
```

### 6.3 常见错误

| 错误信息 | 原因 | 解决方案 |
|----------|------|----------|
| Unknown tool: xxx | 工具名称不存在 | 检查工具名称拼写 |
| Tool name is required | 未提供工具名称 | 在请求体中添加 name 字段 |
| SQL error | 数据库查询错误 | 检查参数类型和值 |

---

## 7. 示例代码

### 7.1 cURL

```bash
# 获取站点状态
curl -X POST http://localhost:20000/mcp/tools/call \
  -H "Content-Type: application/json" \
  -d '{"name":"get_station_status","arguments":{}}'

# 获取最近7天收入趋势
curl -X POST http://localhost:20000/mcp/tools/call \
  -H "Content-Type: application/json" \
  -d '{"name":"get_daily_revenue_trend","arguments":{"days":7}}'

# 查询客户租借历史
curl -X POST http://localhost:20000/mcp/tools/call \
  -H "Content-Type: application/json" \
  -d '{"name":"get_customer_rental_history","arguments":{"customerId":12345,"limit":10}}'
```

### 7.2 Python

```python
import requests

BASE_URL = "http://localhost:20000/mcp"

def call_tool(name: str, arguments: dict = None):
    response = requests.post(
        f"{BASE_URL}/tools/call",
        json={"name": name, "arguments": arguments or {}}
    )
    return response.json()

# 获取站点状态
result = call_tool("get_station_status")
print(result["data"]["content"][0]["text"])

# 获取收入统计
result = call_tool("get_revenue_stats")
print(result["data"]["content"][0]["text"])

# 获取支付渠道分布
result = call_tool("get_payment_breakdown", {"days": 30})
print(result["data"]["content"][0]["text"])
```

### 7.3 JavaScript

```javascript
const BASE_URL = "http://localhost:20000/mcp";

async function callTool(name, arguments = {}) {
  const response = await fetch(`${BASE_URL}/tools/call`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ name, arguments })
  });
  return response.json();
}

// 获取站点状态
const stationStatus = await callTool("get_station_status");
console.log(stationStatus.data.content[0].text);

// 获取活跃租借
const activeRentals = await callTool("get_active_rentals", { limit: 10 });
console.log(activeRentals.data.content[0].text);
```

---

## 8. 更新日志

| 版本 | 日期 | 变更内容 |
|------|------|----------|
| 1.0.0 | 2025-02-15 | 初始版本，17个工具 |
