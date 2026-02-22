# MCP 通用设计方案（接口与验证）

> 适用场景：将业务能力以 MCP（Model Context Protocol）服务形式对外暴露，供 Claude Desktop / 自研 Agent / 插件统一发现与调用。
>
> 设计形态：REST API 网关（可选扩展 WebSocket/SSE）。

---

## 1. 目标与范围

### 1.1 目标

- 以统一协议暴露 **Tools / Resources / Prompts** 三类能力。
- 提供稳定的 **发现能力（list/info）**、**调用能力（call/read/get）**。
- 提供完善的 **安全能力（鉴权、签名、回放防护、权限）**。
- 提供 **可观测能力（日志/审计/追踪/限流）**，方便排障和风控。

### 1.2 范围

- Base URL：`https://{host}/mcp`
- Protocol Version：建议 `2024-11-05`（可按实际升级）
- 数据格式：`application/json; charset=utf-8`
- 兼容现有接口模式：`/mcp/info`、`/mcp/tools/list`、`/mcp/tools/call`、`/mcp/quick/*`

---

## 2. 基础约定（协议与数据模型）

### 2.1 通用响应包裹（Envelope）

建议所有接口统一返回结构：

```json
{
  "code": 200,
  "msg": "ok",
  "data": {}
}
code：业务码（可与 HTTP status 并存）
msg：可读信息
data：实际数据
建议：HTTP status 保持语义化（200/400/401/403/404/429/500），code/msg 体现业务细节。

2.2 工具调用返回（ToolResult）
建议工具调用统一返回：

JSON
{
  "isError": false,
  "content": [
    { "type": "text", "text": "...", "mimeType": "text/markdown", "data": null }
  ]
}
content[].type 建议支持：

text：文本/Markdown（便于直接展示）
json：结构化数据（便于程序消费）
image / binary：需要时以 data（base64）+ mimeType 表示
3. 认证与验证（API Key + HMAC 签名）
3.1 Header 约定
Header	必填	说明
X-MCP-Key	是	API Key ID
X-MCP-Timestamp	是	请求时间戳（毫秒）
X-MCP-Nonce	建议	随机串，用于防重放
X-MCP-Signature	否*	HMAC-SHA256 签名（启用签名验证时必填）
X-MCP-Signature-Version	否	签名版本（如 v1），便于未来升级
3.2 签名算法（推荐 v1：包含 body hash + nonce）
为避免 body 被篡改、并增强防重放能力，建议采用如下 canonical string：

Code
METHOD \n
URI_PATH \n
QUERY_STRING \n
TIMESTAMP \n
NONCE \n
BODY_SHA256
说明：

METHOD：大写，如 POST
URI_PATH：如 /mcp/tools/call
QUERY_STRING：对 query 参数按 key 排序再拼接（无则空串）
TIMESTAMP：毫秒字符串
NONCE：随机串（无则空串）
BODY_SHA256：请求体原始 bytes 的 sha256 hex（GET 可为空串或固定值）
签名计算：

Code
signature = Base64( HMAC-SHA256(canonical, KeySecret) )
3.3 Python 参考实现
Python
import base64
import hashlib
import hmac


def body_sha256(body: bytes) -> str:
    if body is None:
        body = b""
    return hashlib.sha256(body).hexdigest()


def sign_v1(method: str, uri_path: str, query_string: str, timestamp_ms: str, nonce: str, body: bytes, key_secret: str) -> str:
    canonical = "\n".join([
        method.upper(),
        uri_path,
        query_string or "",
        timestamp_ms,
        nonce or "",
        body_sha256(body),
    ])

    digest = hmac.new(
        key_secret.encode("utf-8"),
        canonical.encode("utf-8"),
        hashlib.sha256,
    ).digest()

    return base64.b64encode(digest).decode("utf-8")
3.4 服务端校验流程（建议顺序）
校验 X-MCP-Key 是否存在、是否 active
校验 X-MCP-Timestamp 是否在允许窗口内（例如 ±300s）
（强烈建议）校验 X-MCP-Nonce 未使用过（按 key 维度缓存 5 分钟）
若 signature-enabled=true：
计算 canonical string
使用 key_secret 计算签名
进行常量时间比较（避免计时攻击）
权限校验：该 key 是否允许访问对应 tool/resource/prompt（RBAC/ACL）
限流：按 key/IP/tool 维度
3.5 认证失败响应建议
HTTP 状态码	错误信息示例	说明
401	Missing X-MCP-Key header	缺少 API Key
401	Invalid API Key	API Key 无效
401	Request expired	请求超时
401	Invalid signature	签名错误
403	IP not allowed	IP 不在白名单
429	Too many requests	触发限流
4. 核心接口设计（REST 版本 MCP）
4.1 获取服务信息
GET /mcp/info

响应：

JSON
{
  "code": 200,
  "msg": "ok",
  "data": {
    "name": "xxx-mcp",
    "version": "1.0.0",
    "description": "...",
    "protocol_version": "2024-11-05",
    "capabilities": {
      "tools": true,
      "resources": true,
      "prompts": true
    }
  }
}
4.2 Tools（工具能力）
4.2.1 获取工具列表
GET /mcp/tools/list

响应（示例）：

JSON
{
  "code": 200,
  "msg": "ok",
  "data": [
    {
      "name": "get_station_status",
      "description": "获取站点在线状态统计",
      "inputSchema": {
        "type": "object",
        "properties": {},
        "required": []
      },
      "outputSchema": {
        "type": "object"
      }
    }
  ]
}
4.2.2 调用工具（统一入口）
POST /mcp/tools/call

请求：

JSON
{
  "name": "工具名称",
  "arguments": {
    "param": "value"
  }
}
响应：

JSON
{
  "code": 200,
  "msg": "ok",
  "data": {
    "isError": false,
    "content": [
      {
        "type": "text",
        "mimeType": "text/markdown",
        "text": "..."
      }
    ]
  }
}
错误响应（工具级错误）：

JSON
{
  "code": 200,
  "msg": "ok",
  "data": {
    "isError": true,
    "content": [
      { "type": "text", "text": "Error: Unknown tool: invalid_tool" }
    ]
  }
}
建议：无论成功/失败都返回 requestId/traceId（可放在 header 或 data 中）以便追踪。

5. Resources / Prompts（可选但建议预留）
5.1 Resources
GET /mcp/resources/list
POST /mcp/resources/read
read 请求：

JSON
{ "uri": "db://report/daily?date=2026-02-22" }
5.2 Prompts
GET /mcp/prompts/list
POST /mcp/prompts/get
get 请求：

JSON
{ "name": "ops_daily_summary", "arguments": { "date": "2026-02-22" } }
6. Quick API（快捷查询接口，可选）
用于给人/脚本直接访问常用查询，无需构造 tools/call JSON。

接口	方法	说明
/mcp/quick/station-status	GET	站点状态概览
/mcp/quick/user-stats	GET	用户统计概览
/mcp/quick/revenue-stats	GET	收入统计概览
/mcp/quick/rental-stats	GET	租借统计概览
/mcp/quick/dashboard	GET	综合仪表盘
建议：Quick API 内部复用同一个 tool 实现，以复用权限/限流/审计。

7. 错误处理与可观测性
7.1 HTTP 状态码建议
状态码	说明
200	请求成功
400	请求参数错误
401	未认证或签名失败
403	无权限或 IP 不允许
404	接口不存在
429	触发限流
500	服务器内部错误
7.2 业务错误结构建议
JSON
{
  "code": 40101,
  "msg": "Invalid signature",
  "data": {
    "errorType": "AUTH",
    "requestId": "req_...",
    "detail": "Signature mismatch"
  }
}
7.3 审计日志建议（脱敏）
建议记录：

requestId/traceId
apiKeyId
clientIp
path/method
toolName
timestamp
latencyMs
status/httpStatus
isError
arguments（脱敏/摘要）
8. 配置示例（application.yml）
YAML
mcp:
  server:
    base-path: /mcp
    protocol-version: 2024-11-05

  security:
    enabled: true
    signature-enabled: true
    signature-version: v1
    signature-expire-seconds: 300
    nonce-enabled: true
    nonce-cache-seconds: 300
    ip-whitelist:
      - "127.0.0.1"
      - "10.0.0.0/8"
    api-keys:
      - key-id: "claude-desktop"
        key-secret: "your-secret-key"
        client-name: "Claude Desktop"
        active: true
        permissions:
          - "tools:get_station_status"
          - "tools:get_revenue_stats"

  rate-limit:
    enabled: true
    per-key-rps: 10
    burst: 20
9. MVP 落地清单（最小可用版本）
/mcp/info
/mcp/tools/list
/mcp/tools/call
鉴权：X-MCP-Key + X-MCP-Timestamp + HMAC（含 body hash + nonce）
限流 + requestId + 审计日志
10. 附：cURL 调用示例（含签名 header 占位）
bash
curl -X POST https://{host}/mcp/tools/call \
  -H "Content-Type: application/json" \
  -H "X-MCP-Key: claude-desktop" \
  -H "X-MCP-Timestamp: 1708012800000" \
  -H "X-MCP-Nonce: random-uuid" \
  -H "X-MCP-Signature-Version: v1" \
  -H "X-MCP-Signature: <base64-signature>" \
  -d '{"name":"get_station_status","arguments":{}}'
