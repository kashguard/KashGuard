# KMS API 设计文档

**版本**: v1.0  
**文档类型**: API 规范  
**Base URL**: `https://api.kms.example.com/v1`

---

## 1. API 概述

KMS 提供 RESTful API 和 gRPC API 两种接口，支持密钥管理、加密解密、签名验证等操作。

### 1.1 API 特性

- **RESTful 设计**: 符合 REST 规范
- **JSON 格式**: 请求和响应使用 JSON
- **认证**: 基于 Token 的认证
- **版本控制**: URL 路径版本（/v1/）
- **错误处理**: 统一的错误响应格式
- **限流**: 基于 Token 的请求限流

### 1.2 认证方式

所有 API 请求需要在 Header 中携带认证 Token：

```
Authorization: Bearer <token>
```

---

## 2. 密钥管理 API

### 2.1 创建密钥

**请求**:
```http
POST /v1/keys
Content-Type: application/json
Authorization: Bearer <token>

{
  "alias": "my-encryption-key",
  "description": "用于生产环境数据加密",
  "key_type": "AES_256",
  "key_spec": {
    "algorithm": "AES",
    "key_size": 256
  },
  "tags": {
    "environment": "production",
    "project": "data-encryption"
  },
  "policy": {
    "statements": [
      {
        "effect": "Allow",
        "actions": ["encrypt", "decrypt"],
        "principals": ["user:alice", "role:developer"]
      }
    ]
  }
}
```

**响应**:
```json
{
  "key_id": "key-1234567890abcdef",
  "alias": "my-encryption-key",
  "description": "用于生产环境数据加密",
  "key_type": "AES_256",
  "key_state": "Enabled",
  "created_at": "2024-01-15T10:30:00Z",
  "arn": "arn:kms:us-east-1:123456789012:key/key-1234567890abcdef"
}
```

**状态码**:
- `201 Created`: 创建成功
- `400 Bad Request`: 请求参数错误
- `401 Unauthorized`: 认证失败
- `403 Forbidden`: 权限不足
- `409 Conflict`: 别名已存在

---

### 2.2 获取密钥信息

**请求**:
```http
GET /v1/keys/{key_id}
Authorization: Bearer <token>
```

**响应**:
```json
{
  "key_id": "key-1234567890abcdef",
  "alias": "my-encryption-key",
  "description": "用于生产环境数据加密",
  "key_type": "AES_256",
  "key_state": "Enabled",
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z",
  "arn": "arn:kms:us-east-1:123456789012:key/key-1234567890abcdef",
  "tags": {
    "environment": "production",
    "project": "data-encryption"
  },
  "policy": {
    "statements": [...]
  }
}
```

**状态码**:
- `200 OK`: 成功
- `404 Not Found`: 密钥不存在
- `401 Unauthorized`: 认证失败
- `403 Forbidden`: 权限不足

---

### 2.3 更新密钥

**请求**:
```http
PUT /v1/keys/{key_id}
Content-Type: application/json
Authorization: Bearer <token>

{
  "description": "更新后的描述",
  "tags": {
    "environment": "production",
    "project": "data-encryption",
    "updated": "2024-01-20"
  }
}
```

**响应**:
```json
{
  "key_id": "key-1234567890abcdef",
  "updated_at": "2024-01-20T15:45:00Z"
}
```

---

### 2.4 删除密钥

**请求**:
```http
DELETE /v1/keys/{key_id}
Authorization: Bearer <token>
```

**响应**:
```json
{
  "key_id": "key-1234567890abcdef",
  "deletion_date": "2024-02-15T10:30:00Z",
  "state": "PendingDeletion"
}
```

**说明**:
- 密钥进入计划删除状态
- 等待期（默认 30 天）内可以取消删除
- 等待期后永久删除

---

### 2.5 启用/禁用密钥

**启用密钥**:
```http
POST /v1/keys/{key_id}/enable
Authorization: Bearer <token>
```

**禁用密钥**:
```http
POST /v1/keys/{key_id}/disable
Authorization: Bearer <token>
```

**响应**:
```json
{
  "key_id": "key-1234567890abcdef",
  "key_state": "Enabled",
  "updated_at": "2024-01-20T15:45:00Z"
}
```

---

### 2.6 轮换密钥

**请求**:
```http
POST /v1/keys/{key_id}/rotate
Authorization: Bearer <token>
```

**响应**:
```json
{
  "key_id": "key-1234567890abcdef",
  "new_version": 2,
  "rotated_at": "2024-01-20T15:45:00Z"
}
```

**说明**:
- 创建新版本的密钥
- 旧版本保留用于解密历史数据
- 新版本用于加密新数据

---

### 2.7 列出密钥

**请求**:
```http
GET /v1/keys?limit=50&offset=0&state=Enabled&tag=environment:production
Authorization: Bearer <token>
```

**查询参数**:
- `limit`: 返回数量限制（默认 50，最大 1000）
- `offset`: 偏移量（分页）
- `state`: 密钥状态过滤（Enabled, Disabled, PendingDeletion）
- `tag`: 标签过滤（格式：key:value）
- `alias`: 别名过滤（前缀匹配）

**响应**:
```json
{
  "keys": [
    {
      "key_id": "key-1234567890abcdef",
      "alias": "my-encryption-key",
      "key_type": "AES_256",
      "key_state": "Enabled",
      "created_at": "2024-01-15T10:30:00Z"
    }
  ],
  "total": 100,
  "limit": 50,
  "offset": 0
}
```

---

## 3. 加密解密 API

### 3.1 加密数据

**请求**:
```http
POST /v1/encrypt
Content-Type: application/json
Authorization: Bearer <token>

{
  "key_id": "key-1234567890abcdef",
  "plaintext": "SGVsbG8gV29ybGQ=",  // Base64 编码的明文
  "encryption_context": {
    "project": "data-encryption",
    "environment": "production"
  }
}
```

**响应**:
```json
{
  "ciphertext_blob": "AQICAHh...",  // Base64 编码的密文
  "key_id": "key-1234567890abcdef",
  "key_version": 1
}
```

**说明**:
- `plaintext`: Base64 编码的明文数据
- `encryption_context`: 可选的加密上下文（用于增强安全性）
- `ciphertext_blob`: 包含密钥ID、版本和密文的完整数据

---

### 3.2 解密数据

**请求**:
```http
POST /v1/decrypt
Content-Type: application/json
Authorization: Bearer <token>

{
  "ciphertext_blob": "AQICAHh...",  // Base64 编码的密文
  "encryption_context": {
    "project": "data-encryption",
    "environment": "production"
  }
}
```

**响应**:
```json
{
  "plaintext": "SGVsbG8gV29ybGQ=",  // Base64 编码的明文
  "key_id": "key-1234567890abcdef",
  "key_version": 1
}
```

**说明**:
- 如果提供了 `encryption_context`，必须与加密时一致
- 支持使用旧版本密钥解密历史数据

---

### 3.3 生成数据密钥

**请求**:
```http
POST /v1/generate-data-key
Content-Type: application/json
Authorization: Bearer <token>

{
  "key_id": "key-1234567890abcdef",
  "key_spec": "AES_256",
  "number_of_bytes": 32,
  "encryption_context": {
    "project": "data-encryption"
  }
}
```

**响应（包含明文）**:
```json
{
  "plaintext": "SGVsbG8gV29ybGQ=",  // Base64 编码的明文 DEK
  "ciphertext_blob": "AQICAHh...",  // Base64 编码的加密 DEK
  "key_id": "key-1234567890abcdef"
}
```

**响应（仅密文）**:
```http
POST /v1/generate-data-key
Content-Type: application/json
Authorization: Bearer <token>

{
  "key_id": "key-1234567890abcdef",
  "key_spec": "AES_256",
  "number_of_bytes": 32,
  "encryption_context": {
    "project": "data-encryption"
  },
  "return_plaintext": false  // 不返回明文
}
```

```json
{
  "ciphertext_blob": "AQICAHh...",
  "key_id": "key-1234567890abcdef"
}
```

**说明**:
- 用于信封加密（Envelope Encryption）
- 客户端使用明文 DEK 加密数据后立即删除
- 存储加密后的 DEK（KEK）和密文数据

---

## 4. 签名验证 API

### 4.1 数字签名

**请求**:
```http
POST /v1/sign
Content-Type: application/json
Authorization: Bearer <token>

{
  "key_id": "key-1234567890abcdef",
  "message": "SGVsbG8gV29ybGQ=",  // Base64 编码的消息
  "message_type": "RAW",  // RAW 或 DIGEST
  "signing_algorithm": "ECDSA_SHA_256"
}
```

**响应**:
```json
{
  "signature": "MEUCIQD...",  // Base64 编码的签名
  "key_id": "key-1234567890abcdef",
  "signing_algorithm": "ECDSA_SHA_256"
}
```

**支持的签名算法**:
- `RSA_PSS_SHA_256`
- `RSA_PSS_SHA_384`
- `RSA_PSS_SHA_512`
- `ECDSA_SHA_256`
- `ECDSA_SHA_384`
- `ECDSA_SHA_512`
- `Ed25519`

---

### 4.2 验证签名

**请求**:
```http
POST /v1/verify
Content-Type: application/json
Authorization: Bearer <token>

{
  "key_id": "key-1234567890abcdef",
  "message": "SGVsbG8gV29ybGQ=",
  "message_type": "RAW",
  "signature": "MEUCIQD...",
  "signing_algorithm": "ECDSA_SHA_256"
}
```

**响应**:
```json
{
  "signature_valid": true,
  "key_id": "key-1234567890abcdef",
  "signing_algorithm": "ECDSA_SHA_256"
}
```

---

## 5. 策略管理 API

### 5.1 创建策略

**请求**:
```http
POST /v1/policies
Content-Type: application/json
Authorization: Bearer <token>

{
  "policy_id": "my-policy",
  "description": "开发团队策略",
  "statements": [
    {
      "effect": "Allow",
      "actions": ["encrypt", "decrypt"],
      "resources": ["keys/my-key"],
      "conditions": {
        "encryption_context": {
          "project": ["data-encryption"]
        }
      }
    }
  ]
}
```

**响应**:
```json
{
  "policy_id": "my-policy",
  "created_at": "2024-01-15T10:30:00Z"
}
```

---

### 5.2 获取策略

**请求**:
```http
GET /v1/policies/{policy_id}
Authorization: Bearer <token>
```

**响应**:
```json
{
  "policy_id": "my-policy",
  "description": "开发团队策略",
  "statements": [...],
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

---

### 5.3 更新策略

**请求**:
```http
PUT /v1/policies/{policy_id}
Content-Type: application/json
Authorization: Bearer <token>

{
  "statements": [...]
}
```

---

### 5.4 删除策略

**请求**:
```http
DELETE /v1/policies/{policy_id}
Authorization: Bearer <token>
```

---

## 6. 审计日志 API

### 6.1 查询审计日志

**请求**:
```http
GET /v1/audit-logs?start_time=2024-01-01T00:00:00Z&end_time=2024-01-31T23:59:59Z&key_id=key-1234567890abcdef&limit=100
Authorization: Bearer <token>
```

**查询参数**:
- `start_time`: 开始时间（ISO 8601）
- `end_time`: 结束时间（ISO 8601）
- `key_id`: 密钥ID过滤
- `user_id`: 用户ID过滤
- `event_type`: 事件类型过滤
- `limit`: 返回数量限制
- `offset`: 偏移量

**响应**:
```json
{
  "events": [
    {
      "timestamp": "2024-01-15T10:30:00Z",
      "event_type": "Encrypt",
      "user_id": "user:alice",
      "key_id": "key-1234567890abcdef",
      "operation": "encrypt",
      "result": "Success",
      "ip_address": "192.168.1.100",
      "details": {
        "encryption_context": {...}
      }
    }
  ],
  "total": 1000,
  "limit": 100,
  "offset": 0
}
```

---

### 6.2 导出审计日志

**请求**:
```http
POST /v1/audit-logs/export
Content-Type: application/json
Authorization: Bearer <token>

{
  "start_time": "2024-01-01T00:00:00Z",
  "end_time": "2024-01-31T23:59:59Z",
  "format": "json",  // json, csv, pdf
  "filters": {
    "key_id": "key-1234567890abcdef"
  }
}
```

**响应**:
```json
{
  "export_id": "export-1234567890",
  "status": "Processing",
  "download_url": "https://api.kms.example.com/v1/exports/export-1234567890/download",
  "expires_at": "2024-02-01T00:00:00Z"
}
```

---

## 7. 错误处理

### 7.1 错误响应格式

```json
{
  "error": {
    "code": "InvalidKeyId",
    "message": "The key ID is invalid or does not exist",
    "request_id": "req-1234567890abcdef",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

### 7.2 错误代码

| 错误代码 | HTTP 状态码 | 说明 |
|---------|------------|------|
| `InvalidRequest` | 400 | 请求参数错误 |
| `Unauthorized` | 401 | 认证失败 |
| `Forbidden` | 403 | 权限不足 |
| `NotFound` | 404 | 资源不存在 |
| `Conflict` | 409 | 资源冲突（如别名已存在） |
| `InvalidKeyId` | 400 | 密钥ID无效 |
| `InvalidKeyState` | 400 | 密钥状态不允许此操作 |
| `InvalidCiphertext` | 400 | 密文格式错误 |
| `InvalidSignature` | 400 | 签名验证失败 |
| `KeyDisabled` | 400 | 密钥已禁用 |
| `KeyDeleted` | 400 | 密钥已删除 |
| `RateLimitExceeded` | 429 | 请求频率超限 |
| `InternalError` | 500 | 服务器内部错误 |
| `ServiceUnavailable` | 503 | 服务不可用 |

---

## 8. 限流策略

### 8.1 限流规则

- **默认限流**: 1000 请求/分钟/Token
- **加密解密**: 500 请求/分钟/Token
- **密钥管理**: 100 请求/分钟/Token
- **审计日志查询**: 50 请求/分钟/Token

### 8.2 限流响应

```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
Retry-After: 60

{
  "error": {
    "code": "RateLimitExceeded",
    "message": "Rate limit exceeded. Please retry after 60 seconds.",
    "retry_after": 60
  }
}
```

---

## 9. API 版本管理

### 9.1 版本策略

- URL 路径版本：`/v1/`, `/v2/`
- 向后兼容：保持旧版本 API 可用至少 12 个月
- 弃用通知：提前 6 个月通知 API 弃用

### 9.2 版本信息

```http
GET /v1/version
```

**响应**:
```json
{
  "version": "1.0.0",
  "api_version": "v1",
  "build_date": "2024-01-15T10:30:00Z",
  "git_commit": "abc123def456"
}
```

---

## 10. SDK 示例

### 10.1 Go SDK

```go
package main

import (
    "github.com/kms/sdk-go/kms"
)

func main() {
    client := kms.NewClient(&kms.Config{
        Endpoint: "https://api.kms.example.com",
        Token:    "your-token",
    })
    
    // 创建密钥
    key, err := client.CreateKey(&kms.CreateKeyRequest{
        Alias:       "my-key",
        KeyType:     "AES_256",
        Description: "My encryption key",
    })
    
    // 加密数据
    encrypted, err := client.Encrypt(&kms.EncryptRequest{
        KeyID:     key.KeyID,
        Plaintext: []byte("Hello World"),
    })
    
    // 解密数据
    decrypted, err := client.Decrypt(&kms.DecryptRequest{
        CiphertextBlob: encrypted.CiphertextBlob,
    })
}
```

### 10.2 Python SDK

```python
from kms import KMSClient

client = KMSClient(
    endpoint="https://api.kms.example.com",
    token="your-token"
)

# 创建密钥
key = client.create_key(
    alias="my-key",
    key_type="AES_256",
    description="My encryption key"
)

# 加密数据
encrypted = client.encrypt(
    key_id=key["key_id"],
    plaintext=b"Hello World"
)

# 解密数据
decrypted = client.decrypt(
    ciphertext_blob=encrypted["ciphertext_blob"]
)
```

---

## 11. Webhook 集成

### 11.1 事件类型

- `KeyCreated`: 密钥创建
- `KeyDeleted`: 密钥删除
- `KeyRotated`: 密钥轮换
- `KeyStateChanged`: 密钥状态变更
- `Encrypt`: 加密操作
- `Decrypt`: 解密操作

### 11.2 Webhook 配置

```http
POST /v1/webhooks
Content-Type: application/json
Authorization: Bearer <token>

{
  "url": "https://your-app.com/webhooks/kms",
  "events": ["KeyCreated", "KeyDeleted"],
  "secret": "webhook-secret"
}
```

---

**文档维护**: API 团队  
**最后更新**: 2024

