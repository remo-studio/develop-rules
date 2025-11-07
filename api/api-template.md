# API 接口文档模板

## 概述
简要描述该 API 的功能和用途。

---

## 基础信息

- **接口地址**: `https://api.example.com/v1`
- **认证方式**: Bearer Token / API Key / OAuth 2.0
- **Content-Type**: `application/json`
- **API 版本**: v1.0.0

---

## 身份认证

描述认证方式:

```http
Authorization: Bearer {token}
```

或

```http
X-API-Key: {api_key}
```

---

## 接口: [接口名称]

### 基本信息

- **请求方法**: `GET` | `POST` | `PUT` | `PATCH` | `DELETE`
- **接口路径**: `/api/resource/{id}`
- **接口描述**: 简要描述该接口的功能
- **需要认证**: 是 / 否

---

### 请求参数

#### 路径参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `id` | integer | 是 | 资源唯一标识符 |

#### 查询参数

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `page` | integer | 否 | 1 | 分页页码 |
| `limit` | integer | 否 | 10 | 每页数量 |
| `sort` | string | 否 | `created_at` | 排序字段 |
| `order` | string | 否 | `desc` | 排序方式: `asc` 或 `desc` |

#### 请求头

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `Authorization` | string | 是 | Bearer token |
| `Content-Type` | string | 是 | `application/json` |

#### 请求体

```json
{
  "field1": "string",
  "field2": 123,
  "field3": true,
  "nested_object": {
    "sub_field1": "value",
    "sub_field2": [1, 2, 3]
  }
}
```

**请求体参数**

| 参数名 | 类型 | 必填 | 校验规则 | 说明 |
|--------|------|------|----------|------|
| `field1` | string | 是 | max: 255 | 字段1的描述 |
| `field2` | integer | 否 | min: 0, max: 999 | 字段2的描述 |
| `field3` | boolean | 否 | - | 字段3的描述 |
| `nested_object` | object | 否 | - | 嵌套对象的描述 |
| `nested_object.sub_field1` | string | 否 | - | 子字段的描述 |
| `nested_object.sub_field2` | array | 否 | - | 整数数组 |

---

### 响应结果

#### 成功响应 (200 OK)

```json
{
  "code": 200,
  "message": "Success",
  "data": {
    "id": 1,
    "field1": "string",
    "field2": 123,
    "field3": true,
    "created_at": "2025-01-01T00:00:00Z",
    "updated_at": "2025-01-01T00:00:00Z"
  },
  "meta": {
    "page": 1,
    "limit": 10,
    "total": 100,
    "total_pages": 10
  }
}
```

**响应字段说明**

| 字段名 | 类型 | 说明 |
|--------|------|------|
| `code` | integer | HTTP 状态码 |
| `message` | string | 响应消息 |
| `data` | object | 响应数据 |
| `data.id` | integer | 资源唯一标识符 |
| `data.field1` | string | 字段描述 |
| `meta` | object | 元数据 (分页信息等) |

#### 错误响应

**400 请求参数错误**

```json
{
  "code": 400,
  "message": "参数校验失败",
  "errors": {
    "field1": ["field1 字段必填"],
    "field2": ["field2 必须是有效的整数"]
  }
}
```

**401 未授权**

```json
{
  "code": 401,
  "message": "未授权访问",
  "errors": null
}
```

**403 禁止访问**

```json
{
  "code": 403,
  "message": "权限不足",
  "errors": null
}
```

**404 资源不存在**

```json
{
  "code": 404,
  "message": "资源不存在",
  "errors": null
}
```

**500 服务器内部错误**

```json
{
  "code": 500,
  "message": "服务器内部错误",
  "errors": null
}
```

---

### 请求示例

#### cURL

```bash
curl -X POST 'https://api.example.com/v1/resource?page=1&limit=10' \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "field1": "value",
    "field2": 123,
    "field3": true
  }'
```

#### JavaScript (fetch)

```javascript
const response = await fetch('https://api.example.com/v1/resource?page=1&limit=10', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer YOUR_TOKEN',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    field1: 'value',
    field2: 123,
    field3: true
  })
});

const data = await response.json();
```

#### Python (requests)

```python
import requests

url = 'https://api.example.com/v1/resource'
headers = {
    'Authorization': 'Bearer YOUR_TOKEN',
    'Content-Type': 'application/json'
}
params = {'page': 1, 'limit': 10}
data = {
    'field1': 'value',
    'field2': 123,
    'field3': True
}

response = requests.post(url, headers=headers, params=params, json=data)
result = response.json()
```

---

## 状态码说明

| 状态码 | 说明 |
|--------|------|
| 200 | 请求成功 |
| 201 | 资源创建成功 |
| 204 | 请求成功，无返回内容 |
| 400 | 请求参数错误 |
| 401 | 未授权或认证失败 |
| 403 | 权限不足 |
| 404 | 资源不存在 |
| 422 | 请求格式正确但语义错误 |
| 429 | 请求过于频繁，超过速率限制 |
| 500 | 服务器内部错误 |
| 503 | 服务暂时不可用 |

---

## 速率限制

- **限制规则**: 每小时 1000 次请求
- **响应头**:
  - `X-RateLimit-Limit`: 每小时最大请求次数
  - `X-RateLimit-Remaining`: 剩余请求次数
  - `X-RateLimit-Reset`: 速率限制重置时间 (Unix 时间戳)

---

## 分页说明

API 使用基于偏移量的分页:

- `page`: 页码 (默认: 1)
- `limit`: 每页数量 (默认: 10, 最大: 100)

**响应包含分页元数据:**

```json
{
  "meta": {
    "page": 1,
    "limit": 10,
    "total": 100,
    "total_pages": 10
  }
}
```

---

## 过滤和排序

### 过滤

```http
GET /api/resource?status=active&type=user
```

### 排序

```http
GET /api/resource?sort=created_at&order=desc
```

---

## 更新日志

### v1.0.0 (2025-01-01)
- 初始版本发布
- 添加基础 CRUD 操作

---

## 注意事项

- 所有时间戳使用 ISO 8601 格式 (UTC)
- 所有响应均为 JSON 格式
- 请求体大小限制: 10MB
- 日期格式: `YYYY-MM-DDTHH:mm:ssZ`
- 支持语言: en, zh-CN

---

## 技术支持

- **文档地址**: https://docs.example.com
- **技术支持邮箱**: support@example.com
- **服务状态页**: https://status.example.com
