# MPC API 设计文档

**版本**: v1.0  
**文档类型**: API 规范  
**Base URL**: `https://api.mpc.example.com/v1`

---

## 1. API 概述

MPC 基础设施提供 RESTful API 和 gRPC API 两种接口，支持密钥分片管理、阈值签名、节点管理等操作。API 设计遵循 RESTful 规范，支持 2B 和 2C 场景的不同需求。

### 1.1 API 特性

- **RESTful 设计**: 符合 REST 规范
- **JSON 格式**: 请求和响应使用 JSON
- **认证**: 基于 Token 的认证
- **版本控制**: URL 路径版本（/v1/）
- **错误处理**: 统一的错误响应格式
- **限流**: 基于 Token 的请求限流
- **gRPC 支持**: 高性能 RPC 接口

### 1.2 认证方式

所有 API 请求需要在 Header 中携带认证 Token：

```
Authorization: Bearer <token>
```

---

## 2. 密钥管理 API

### 2.1 创建密钥分片

**请求**:
```http
POST /v1/keys
Content-Type: application/json
Authorization: Bearer <token>

{
  "key_id": "key-1234567890abcdef",
  "algorithm": "ECDSA",
  "curve": "secp256k1",
  "threshold": 2,
  "total_nodes": 3,
  "chain_type": "ethereum",
  "description": "企业多签钱包密钥",
  "tags": {
    "environment": "production",
    "wallet_type": "multisig"
  }
}
```

**响应**:
```json
{
  "key_id": "key-1234567890abcdef",
  "public_key": "0x1234567890abcdef...",
  "algorithm": "ECDSA",
  "curve": "secp256k1",
  "threshold": 2,
  "total_nodes": 3,
  "chain_type": "ethereum",
  "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  "status": "active",
  "created_at": "2024-01-15T10:30:00Z",
  "nodes": [
    {
      "node_id": "node-1",
      "status": "active",
      "key_share_ready": true
    },
    {
      "node_id": "node-2",
      "status": "active",
      "key_share_ready": true
    },
    {
      "node_id": "node-3",
      "status": "active",
      "key_share_ready": true
    }
  ]
}
```

**状态码**:
- `201 Created`: 创建成功
- `400 Bad Request`: 请求参数错误
- `401 Unauthorized`: 认证失败
- `403 Forbidden`: 权限不足
- `409 Conflict`: 密钥ID已存在

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
  "public_key": "0x1234567890abcdef...",
  "algorithm": "ECDSA",
  "curve": "secp256k1",
  "threshold": 2,
  "total_nodes": 3,
  "chain_type": "ethereum",
  "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  "status": "active",
  "description": "企业多签钱包密钥",
  "tags": {
    "environment": "production",
    "wallet_type": "multisig"
  },
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z",
  "nodes": [
    {
      "node_id": "node-1",
      "status": "active",
      "key_share_ready": true,
      "last_heartbeat": "2024-01-15T10:35:00Z"
    }
  ],
  "signature_count": 1250,
  "last_signature_at": "2024-01-15T10:32:00Z"
}
```

**状态码**:
- `200 OK`: 成功
- `404 Not Found`: 密钥不存在
- `401 Unauthorized`: 认证失败
- `403 Forbidden`: 权限不足

---

### 2.3 列出密钥

**请求**:
```http
GET /v1/keys?limit=50&offset=0&chain_type=ethereum&status=active
Authorization: Bearer <token>
```

**查询参数**:
- `limit`: 返回数量限制（默认 50，最大 1000）
- `offset`: 偏移量（分页）
- `chain_type`: 链类型过滤（ethereum、bitcoin、bsc 等）
- `status`: 状态过滤（active、inactive、deleted）
- `tag`: 标签过滤（格式：key:value）

**响应**:
```json
{
  "keys": [
    {
      "key_id": "key-1234567890abcdef",
      "public_key": "0x1234567890abcdef...",
      "chain_type": "ethereum",
      "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
      "status": "active",
      "threshold": 2,
      "total_nodes": 3,
      "created_at": "2024-01-15T10:30:00Z"
    }
  ],
  "total": 100,
  "limit": 50,
  "offset": 0
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
  "status": "deleted",
  "deleted_at": "2024-01-20T15:45:00Z"
}
```

**说明**:
- 密钥进入删除状态
- 所有节点的密钥分片将被删除
- 操作不可恢复

---

### 2.5 密钥轮换

**请求**:
```http
POST /v1/keys/{key_id}/rotate
Content-Type: application/json
Authorization: Bearer <token>

{
  "new_threshold": 3,
  "new_total_nodes": 5
}
```

**响应**:
```json
{
  "key_id": "key-1234567890abcdef",
  "old_public_key": "0x1234567890abcdef...",
  "new_public_key": "0xfedcba0987654321...",
  "old_threshold": 2,
  "new_threshold": 3,
  "old_total_nodes": 3,
  "new_total_nodes": 5,
  "rotated_at": "2024-01-20T15:45:00Z"
}
```

**说明**:
- 生成新的密钥分片
- 旧密钥保留用于验证历史签名
- 新密钥用于新签名

---

## 3. 签名服务 API

### 3.1 阈值签名

**请求**:
```http
POST /v1/sign
Content-Type: application/json
Authorization: Bearer <token>

{
  "key_id": "key-1234567890abcdef",
  "message": "0x1234567890abcdef...",
  "message_type": "transaction",
  "chain_type": "ethereum",
  "transaction": {
    "to": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "value": "1000000000000000000",
    "gas_limit": 21000,
    "gas_price": "20000000000",
    "nonce": 42
  }
}
```

**响应**:
```json
{
  "signature": "0x1234567890abcdef...",
  "key_id": "key-1234567890abcdef",
  "public_key": "0x1234567890abcdef...",
  "message": "0x1234567890abcdef...",
  "chain_type": "ethereum",
  "session_id": "session-abc123",
  "signed_at": "2024-01-15T10:30:00Z",
  "participating_nodes": ["node-1", "node-2"],
  "transaction": {
    "raw": "0xf86c...",
    "hash": "0x789abc..."
  }
}
```

**说明**:
- `message`: 要签名的消息（Hex 编码）
- `message_type`: 消息类型（transaction、message、raw）
- `transaction`: 交易详情（仅当 message_type=transaction 时）

---

### 3.2 批量签名

**请求**:
```http
POST /v1/sign/batch
Content-Type: application/json
Authorization: Bearer <token>

{
  "key_id": "key-1234567890abcdef",
  "messages": [
    {
      "message": "0x1234567890abcdef...",
      "message_type": "transaction"
    },
    {
      "message": "0xfedcba0987654321...",
      "message_type": "message"
    }
  ],
  "chain_type": "ethereum"
}
```

**响应**:
```json
{
  "signatures": [
    {
      "signature": "0x1234567890abcdef...",
      "message": "0x1234567890abcdef...",
      "signed_at": "2024-01-15T10:30:00Z"
    },
    {
      "signature": "0xfedcba0987654321...",
      "message": "0xfedcba0987654321...",
      "signed_at": "2024-01-15T10:30:01Z"
    }
  ],
  "total": 2,
  "success": 2,
  "failed": 0
}
```

---

### 3.3 签名验证

**请求**:
```http
POST /v1/verify
Content-Type: application/json
Authorization: Bearer <token>

{
  "signature": "0x1234567890abcdef...",
  "message": "0x1234567890abcdef...",
  "public_key": "0x1234567890abcdef...",
  "algorithm": "ECDSA",
  "chain_type": "ethereum"
}
```

**响应**:
```json
{
  "valid": true,
  "public_key": "0x1234567890abcdef...",
  "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  "verified_at": "2024-01-15T10:30:00Z"
}
```

---

## 4. 节点管理 API

### 4.1 注册节点

**请求**:
```http
POST /v1/nodes
Content-Type: application/json
Authorization: Bearer <token>

{
  "node_id": "node-1",
  "node_type": "participant",
  "endpoint": "https://node-1.example.com:8080",
  "public_key": "0x1234567890abcdef...",
  "capabilities": ["ecdsa", "eddsa"],
  "metadata": {
    "region": "us-east-1",
    "zone": "zone-a"
  }
}
```

**响应**:
```json
{
  "node_id": "node-1",
  "node_type": "participant",
  "status": "active",
  "endpoint": "https://node-1.example.com:8080",
  "public_key": "0x1234567890abcdef...",
  "capabilities": ["ecdsa", "eddsa"],
  "registered_at": "2024-01-15T10:30:00Z",
  "last_heartbeat": "2024-01-15T10:35:00Z"
}
```

---

### 4.2 获取节点信息

**请求**:
```http
GET /v1/nodes/{node_id}
Authorization: Bearer <token>
```

**响应**:
```json
{
  "node_id": "node-1",
  "node_type": "participant",
  "status": "active",
  "endpoint": "https://node-1.example.com:8080",
  "public_key": "0x1234567890abcdef...",
  "capabilities": ["ecdsa", "eddsa"],
  "metadata": {
    "region": "us-east-1",
    "zone": "zone-a"
  },
  "registered_at": "2024-01-15T10:30:00Z",
  "last_heartbeat": "2024-01-15T10:35:00Z",
  "key_shares_count": 50,
  "signature_count": 1250,
  "performance": {
    "avg_signature_latency_ms": 180,
    "success_rate": 0.999
  }
}
```

---

### 4.3 列出节点

**请求**:
```http
GET /v1/nodes?node_type=participant&status=active&limit=50
Authorization: Bearer <token>
```

**查询参数**:
- `node_type`: 节点类型（coordinator、participant）
- `status`: 状态过滤（active、inactive、faulty）
- `limit`: 返回数量限制
- `offset`: 偏移量

**响应**:
```json
{
  "nodes": [
    {
      "node_id": "node-1",
      "node_type": "participant",
      "status": "active",
      "endpoint": "https://node-1.example.com:8080",
      "last_heartbeat": "2024-01-15T10:35:00Z"
    }
  ],
  "total": 10,
  "limit": 50,
  "offset": 0
}
```

---

### 4.4 节点健康检查

**请求**:
```http
GET /v1/nodes/{node_id}/health
Authorization: Bearer <token>
```

**响应**:
```json
{
  "node_id": "node-1",
  "status": "healthy",
  "timestamp": "2024-01-15T10:35:00Z",
  "checks": {
    "connectivity": "ok",
    "key_share_storage": "ok",
    "signing_capability": "ok"
  },
  "metrics": {
    "cpu_usage": 45.2,
    "memory_usage": 62.8,
    "disk_usage": 38.5
  }
}
```

---

## 5. 协议管理 API

### 5.1 创建签名会话

**请求**:
```http
POST /v1/sessions
Content-Type: application/json
Authorization: Bearer <token>

{
  "key_id": "key-1234567890abcdef",
  "message": "0x1234567890abcdef...",
  "protocol": "gg20",
  "timeout": 300
}
```

**响应**:
```json
{
  "session_id": "session-abc123",
  "key_id": "key-1234567890abcdef",
  "protocol": "gg20",
  "status": "pending",
  "threshold": 2,
  "total_nodes": 3,
  "participating_nodes": [],
  "created_at": "2024-01-15T10:30:00Z",
  "timeout": 300,
  "expires_at": "2024-01-15T10:35:00Z"
}
```

---

### 5.2 加入签名会话

**请求**:
```http
POST /v1/sessions/{session_id}/join
Content-Type: application/json
Authorization: Bearer <token>

{
  "node_id": "node-1"
}
```

**响应**:
```json
{
  "session_id": "session-abc123",
  "node_id": "node-1",
  "status": "joined",
  "joined_at": "2024-01-15T10:30:05Z",
  "current_round": 1,
  "total_rounds": 4
}
```

---

### 5.3 获取会话状态

**请求**:
```http
GET /v1/sessions/{session_id}
Authorization: Bearer <token>
```

**响应**:
```json
{
  "session_id": "session-abc123",
  "key_id": "key-1234567890abcdef",
  "protocol": "gg20",
  "status": "completed",
  "threshold": 2,
  "total_nodes": 3,
  "participating_nodes": ["node-1", "node-2"],
  "current_round": 4,
  "total_rounds": 4,
  "signature": "0x1234567890abcdef...",
  "created_at": "2024-01-15T10:30:00Z",
  "completed_at": "2024-01-15T10:30:15Z",
  "duration_ms": 15000
}
```

---

### 5.4 取消签名会话

**请求**:
```http
POST /v1/sessions/{session_id}/cancel
Authorization: Bearer <token>
```

**响应**:
```json
{
  "session_id": "session-abc123",
  "status": "cancelled",
  "cancelled_at": "2024-01-15T10:30:10Z"
}
```

---

## 6. 链特定 API

### 6.1 生成地址

**请求**:
```http
POST /v1/keys/{key_id}/address
Content-Type: application/json
Authorization: Bearer <token>

{
  "chain_type": "ethereum",
  "derivation_path": "m/44'/60'/0'/0/0"
}
```

**响应**:
```json
{
  "key_id": "key-1234567890abcdef",
  "chain_type": "ethereum",
  "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  "derivation_path": "m/44'/60'/0'/0/0",
  "public_key": "0x1234567890abcdef..."
}
```

---

### 6.2 构建交易

**请求**:
```http
POST /v1/transactions/build
Content-Type: application/json
Authorization: Bearer <token>

{
  "chain_type": "ethereum",
  "from": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  "to": "0x1234567890123456789012345678901234567890",
  "value": "1000000000000000000",
  "gas_limit": 21000,
  "gas_price": "20000000000",
  "nonce": 42
}
```

**响应**:
```json
{
  "transaction": {
    "to": "0x1234567890123456789012345678901234567890",
    "value": "1000000000000000000",
    "gas_limit": 21000,
    "gas_price": "20000000000",
    "nonce": 42,
    "data": "0x",
    "chain_id": 1
  },
  "message_hash": "0x1234567890abcdef...",
  "ready_to_sign": true
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
    "timestamp": "2024-01-15T10:30:00Z",
    "details": {
      "key_id": "key-invalid"
    }
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
| `InvalidKeyId` | 400 | 密钥ID无效 |
| `InvalidThreshold` | 400 | 阈值配置无效 |
| `InsufficientNodes` | 400 | 节点数量不足 |
| `NodeUnavailable` | 503 | 节点不可用 |
| `SigningSessionTimeout` | 408 | 签名会话超时 |
| `SigningSessionFailed` | 500 | 签名会话失败 |
| `RateLimitExceeded` | 429 | 请求频率超限 |
| `InternalError` | 500 | 服务器内部错误 |
| `ServiceUnavailable` | 503 | 服务不可用 |

---

## 8. 限流策略

### 8.1 限流规则

- **默认限流**: 1000 请求/分钟/Token
- **签名请求**: 100 请求/分钟/Token
- **密钥管理**: 50 请求/分钟/Token
- **节点管理**: 20 请求/分钟/Token

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

## 9. WebSocket API

### 9.1 连接

```javascript
const ws = new WebSocket('wss://api.mpc.example.com/v1/ws?token=your-token');

ws.onopen = () => {
  console.log('Connected');
};

ws.onmessage = (event) => {
  const message = JSON.parse(event.data);
  console.log('Message:', message);
};
```

### 9.2 订阅签名会话

```json
{
  "action": "subscribe",
  "resource": "session",
  "session_id": "session-abc123"
}
```

### 9.3 接收事件

```json
{
  "event_type": "session_status_changed",
  "session_id": "session-abc123",
  "status": "completed",
  "timestamp": "2024-01-15T10:30:15Z",
  "data": {
    "signature": "0x1234567890abcdef..."
  }
}
```

---

## 10. gRPC API

### 10.1 服务定义

```protobuf
service MPCService {
  // 密钥管理
  rpc CreateKey(CreateKeyRequest) returns (CreateKeyResponse);
  rpc GetKey(GetKeyRequest) returns (GetKeyResponse);
  rpc ListKeys(ListKeysRequest) returns (ListKeysResponse);
  rpc DeleteKey(DeleteKeyRequest) returns (DeleteKeyResponse);
  
  // 签名服务
  rpc Sign(SignRequest) returns (SignResponse);
  rpc BatchSign(BatchSignRequest) returns (BatchSignResponse);
  rpc Verify(VerifyRequest) returns (VerifyResponse);
  
  // 节点管理
  rpc RegisterNode(RegisterNodeRequest) returns (RegisterNodeResponse);
  rpc GetNode(GetNodeRequest) returns (GetNodeResponse);
  rpc ListNodes(ListNodesRequest) returns (ListNodesResponse);
  
  // 会话管理
  rpc CreateSession(CreateSessionRequest) returns (CreateSessionResponse);
  rpc JoinSession(JoinSessionRequest) returns (JoinSessionResponse);
  rpc GetSession(GetSessionRequest) returns (GetSessionResponse);
}
```

---

## 11. SDK 示例

### 11.1 Go SDK

```go
package main

import (
    "github.com/mpc/sdk-go/mpc"
)

func main() {
    client := mpc.NewClient(&mpc.Config{
        Endpoint: "https://api.mpc.example.com",
        Token:    "your-token",
    })
    
    // 创建密钥
    key, err := client.CreateKey(&mpc.CreateKeyRequest{
        Algorithm: "ECDSA",
        Curve:     "secp256k1",
        Threshold: 2,
        TotalNodes: 3,
        ChainType: "ethereum",
    })
    
    // 签名
    signature, err := client.Sign(&mpc.SignRequest{
        KeyID:  key.KeyID,
        Message: []byte("Hello World"),
        ChainType: "ethereum",
    })
}
```

### 11.2 Python SDK

```python
from mpc import MPCClient

client = MPCClient(
    endpoint="https://api.mpc.example.com",
    token="your-token"
)

# 创建密钥
key = client.create_key(
    algorithm="ECDSA",
    curve="secp256k1",
    threshold=2,
    total_nodes=3,
    chain_type="ethereum"
)

# 签名
signature = client.sign(
    key_id=key["key_id"],
    message=b"Hello World",
    chain_type="ethereum"
)
```

### 11.3 JavaScript SDK

```javascript
const { MPCClient } = require('@mpc/sdk');

const client = new MPCClient({
  endpoint: 'https://api.mpc.example.com',
  token: 'your-token'
});

// 创建密钥
const key = await client.createKey({
  algorithm: 'ECDSA',
  curve: 'secp256k1',
  threshold: 2,
  totalNodes: 3,
  chainType: 'ethereum'
});

// 签名
const signature = await client.sign({
  keyId: key.keyId,
  message: Buffer.from('Hello World'),
  chainType: 'ethereum'
});
```

---

## 12. API 版本管理

### 12.1 版本策略

- URL 路径版本：`/v1/`, `/v2/`
- 向后兼容：保持旧版本 API 可用至少 12 个月
- 弃用通知：提前 6 个月通知 API 弃用

### 12.2 版本信息

```http
GET /v1/version
```

**响应**:
```json
{
  "version": "1.0.0",
  "api_version": "v1",
  "protocols": ["gg18", "gg20", "frost"],
  "supported_chains": ["ethereum", "bitcoin", "bsc"],
  "build_date": "2024-01-15T10:30:00Z",
  "git_commit": "abc123def456"
}
```

---

**文档维护**: API 团队  
**最后更新**: 2024

