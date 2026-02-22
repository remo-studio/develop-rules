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
