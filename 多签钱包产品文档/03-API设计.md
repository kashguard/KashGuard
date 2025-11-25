# 多签钱包 API 设计文档

**版本**: v1.0  
**文档类型**: API 规范  
**Base URL**: `https://api.multisig.example.com/v1`

---

## 1. API 概述

多签钱包提供 RESTful API 和 gRPC API 两种接口，支持钱包管理、交易审批、签名执行等操作。API 设计遵循 RESTful 规范，支持 2B 和 2C 场景的不同需求。

### 1.1 API 特性

- **RESTful 设计**: 符合 REST 规范
- **JSON 格式**: 请求和响应使用 JSON
- **认证**: 基于 Token 的认证
- **版本控制**: URL 路径版本（/v1/）
- **错误处理**: 统一的错误响应格式
- **限流**: 基于 Token 的请求限流
- **WebSocket**: 实时事件推送

### 1.2 认证方式

所有 API 请求需要在 Header 中携带认证 Token：

```
Authorization: Bearer <token>
```

---

## 2. 钱包管理 API

### 2.1 创建钱包

**请求**:
```http
POST /v1/wallets
Content-Type: application/json
Authorization: Bearer <token>

{
  "name": "企业多签钱包",
  "description": "用于企业资金管理",
  "threshold": 2,
  "total_members": 3,
  "chain_type": "ethereum",
  "members": [
    {
      "user_id": "user-1",
      "role": "admin"
    },
    {
      "user_id": "user-2",
      "role": "approver"
    },
    {
      "user_id": "user-3",
      "role": "approver"
    }
  ],
  "approval_rules": {
    "min_amount": "1000000000000000000",
    "max_amount": "100000000000000000000",
    "address_whitelist": ["0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"]
  }
}
```

**响应**:
```json
{
  "wallet_id": "wallet-1234567890abcdef",
  "name": "企业多签钱包",
  "description": "用于企业资金管理",
  "threshold": 2,
  "total_members": 3,
  "chain_type": "ethereum",
  "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  "public_key": "0x1234567890abcdef...",
  "status": "active",
  "created_at": "2024-01-15T10:30:00Z",
  "members": [
    {
      "member_id": "member-1",
      "user_id": "user-1",
      "role": "admin",
      "status": "active"
    },
    {
      "member_id": "member-2",
      "user_id": "user-2",
      "role": "approver",
      "status": "active"
    },
    {
      "member_id": "member-3",
      "user_id": "user-3",
      "role": "approver",
      "status": "active"
    }
  ],
  "mpc_key_id": "key-1234567890abcdef"
}
```

**状态码**:
- `201 Created`: 创建成功
- `400 Bad Request`: 请求参数错误
- `401 Unauthorized`: 认证失败
- `403 Forbidden`: 权限不足

---

### 2.2 获取钱包信息

**请求**:
```http
GET /v1/wallets/{wallet_id}
Authorization: Bearer <token>
```

**响应**:
```json
{
  "wallet_id": "wallet-1234567890abcdef",
  "name": "企业多签钱包",
  "description": "用于企业资金管理",
  "threshold": 2,
  "total_members": 3,
  "chain_type": "ethereum",
  "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  "public_key": "0x1234567890abcdef...",
  "status": "active",
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z",
  "members": [
    {
      "member_id": "member-1",
      "user_id": "user-1",
      "role": "admin",
      "status": "active",
      "joined_at": "2024-01-15T10:30:00Z",
      "last_active_at": "2024-01-15T10:35:00Z"
    }
  ],
  "balance": {
    "eth": "10.5",
    "usdt": "50000"
  },
  "statistics": {
    "total_transactions": 1250,
    "pending_approvals": 5,
    "total_approved": 1200,
    "total_rejected": 45
  }
}
```

---

### 2.3 列出钱包

**请求**:
```http
GET /v1/wallets?limit=50&offset=0&chain_type=ethereum&status=active
Authorization: Bearer <token>
```

**查询参数**:
- `limit`: 返回数量限制（默认 50，最大 1000）
- `offset`: 偏移量（分页）
- `chain_type`: 链类型过滤
- `status`: 状态过滤（active、inactive、deleted）
- `user_id`: 用户ID过滤（用户参与的钱包）

**响应**:
```json
{
  "wallets": [
    {
      "wallet_id": "wallet-1234567890abcdef",
      "name": "企业多签钱包",
      "chain_type": "ethereum",
      "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
      "threshold": 2,
      "total_members": 3,
      "status": "active",
      "created_at": "2024-01-15T10:30:00Z"
    }
  ],
  "total": 100,
  "limit": 50,
  "offset": 0
}
```

---

### 2.4 更新钱包配置

**请求**:
```http
PUT /v1/wallets/{wallet_id}
Content-Type: application/json
Authorization: Bearer <token>

{
  "name": "更新后的钱包名称",
  "description": "更新后的描述",
  "approval_rules": {
    "min_amount": "2000000000000000000",
    "max_amount": "200000000000000000000"
  }
}
```

**响应**:
```json
{
  "wallet_id": "wallet-1234567890abcdef",
  "updated_at": "2024-01-20T15:45:00Z"
}
```

---

### 2.5 删除钱包

**请求**:
```http
DELETE /v1/wallets/{wallet_id}
Authorization: Bearer <token>
```

**响应**:
```json
{
  "wallet_id": "wallet-1234567890abcdef",
  "status": "deleted",
  "deleted_at": "2024-01-20T15:45:00Z"
}
```

---

## 3. 成员管理 API

### 3.1 添加成员

**请求**:
```http
POST /v1/wallets/{wallet_id}/members
Content-Type: application/json
Authorization: Bearer <token>

{
  "user_id": "user-4",
  "role": "approver"
}
```

**响应**:
```json
{
  "member_id": "member-4",
  "wallet_id": "wallet-1234567890abcdef",
  "user_id": "user-4",
  "role": "approver",
  "status": "active",
  "joined_at": "2024-01-20T15:45:00Z"
}
```

---

### 3.2 移除成员

**请求**:
```http
DELETE /v1/wallets/{wallet_id}/members/{member_id}
Authorization: Bearer <token>
```

**响应**:
```json
{
  "member_id": "member-4",
  "status": "removed",
  "removed_at": "2024-01-20T15:45:00Z"
}
```

---

### 3.3 更新成员角色

**请求**:
```http
PUT /v1/wallets/{wallet_id}/members/{member_id}
Content-Type: application/json
Authorization: Bearer <token>

{
  "role": "admin"
}
```

**响应**:
```json
{
  "member_id": "member-4",
  "role": "admin",
  "updated_at": "2024-01-20T15:45:00Z"
}
```

---

### 3.4 获取成员列表

**请求**:
```http
GET /v1/wallets/{wallet_id}/members
Authorization: Bearer <token>
```

**响应**:
```json
{
  "members": [
    {
      "member_id": "member-1",
      "user_id": "user-1",
      "role": "admin",
      "status": "active",
      "joined_at": "2024-01-15T10:30:00Z",
      "last_active_at": "2024-01-15T10:35:00Z"
    }
  ],
  "total": 3
}
```

---

## 4. 交易管理 API

### 4.1 创建交易

**请求**:
```http
POST /v1/wallets/{wallet_id}/transactions
Content-Type: application/json
Authorization: Bearer <token>

{
  "to": "0x1234567890123456789012345678901234567890",
  "value": "1000000000000000000",
  "gas_limit": 21000,
  "gas_price": "20000000000",
  "data": "0x",
  "description": "支付供应商款项"
}
```

**响应**:
```json
{
  "tx_id": "tx-1234567890abcdef",
  "wallet_id": "wallet-1234567890abcdef",
  "chain_type": "ethereum",
  "from": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  "to": "0x1234567890123456789012345678901234567890",
  "value": "1000000000000000000",
  "gas_limit": 21000,
  "gas_price": "20000000000",
  "nonce": 42,
  "data": "0x",
  "status": "pending_approval",
  "description": "支付供应商款项",
  "created_at": "2024-01-15T10:30:00Z",
  "requester_id": "user-1"
}
```

---

### 4.2 获取交易信息

**请求**:
```http
GET /v1/wallets/{wallet_id}/transactions/{tx_id}
Authorization: Bearer <token>
```

**响应**:
```json
{
  "tx_id": "tx-1234567890abcdef",
  "wallet_id": "wallet-1234567890abcdef",
  "chain_type": "ethereum",
  "from": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  "to": "0x1234567890123456789012345678901234567890",
  "value": "1000000000000000000",
  "gas_limit": 21000,
  "gas_price": "20000000000",
  "nonce": 42,
  "data": "0x",
  "status": "approved",
  "description": "支付供应商款项",
  "created_at": "2024-01-15T10:30:00Z",
  "approved_at": "2024-01-15T10:32:00Z",
  "signed_at": "2024-01-15T10:33:00Z",
  "broadcast_at": "2024-01-15T10:33:05Z",
  "confirmed_at": "2024-01-15T10:35:00Z",
  "tx_hash": "0x789abc...",
  "approvals": [
    {
      "approver_id": "user-2",
      "approved_at": "2024-01-15T10:31:00Z"
    },
    {
      "approver_id": "user-3",
      "approved_at": "2024-01-15T10:32:00Z"
    }
  ],
  "requester_id": "user-1"
}
```

---

### 4.3 列出交易

**请求**:
```http
GET /v1/wallets/{wallet_id}/transactions?limit=50&offset=0&status=approved
Authorization: Bearer <token>
```

**查询参数**:
- `limit`: 返回数量限制
- `offset`: 偏移量
- `status`: 状态过滤（pending_approval、approved、signed、broadcast、confirmed、failed）
- `start_time`: 开始时间
- `end_time`: 结束时间

**响应**:
```json
{
  "transactions": [
    {
      "tx_id": "tx-1234567890abcdef",
      "to": "0x1234567890123456789012345678901234567890",
      "value": "1000000000000000000",
      "status": "confirmed",
      "created_at": "2024-01-15T10:30:00Z",
      "tx_hash": "0x789abc..."
    }
  ],
  "total": 100,
  "limit": 50,
  "offset": 0
}
```

---

## 5. 审批管理 API

### 5.1 审批交易

**请求**:
```http
POST /v1/wallets/{wallet_id}/transactions/{tx_id}/approve
Content-Type: application/json
Authorization: Bearer <token>

{
  "comment": "同意支付"
}
```

**响应**:
```json
{
  "approval_id": "approval-1234567890abcdef",
  "request_id": "request-1234567890abcdef",
  "tx_id": "tx-1234567890abcdef",
  "approver_id": "user-2",
  "status": "approved",
  "comment": "同意支付",
  "approved_at": "2024-01-15T10:31:00Z",
  "approval_count": 1,
  "threshold": 2,
  "auto_execute": false
}
```

**说明**:
- 如果达到阈值，`auto_execute` 为 `true`，交易将自动执行签名
- 如果未达到阈值，`auto_execute` 为 `false`，等待更多审批

---

### 5.2 拒绝交易

**请求**:
```http
POST /v1/wallets/{wallet_id}/transactions/{tx_id}/reject
Content-Type: application/json
Authorization: Bearer <token>

{
  "reason": "金额过大，需要更多审批"
}
```

**响应**:
```json
{
  "rejection_id": "rejection-1234567890abcdef",
  "request_id": "request-1234567890abcdef",
  "tx_id": "tx-1234567890abcdef",
  "rejector_id": "user-2",
  "status": "rejected",
  "reason": "金额过大，需要更多审批",
  "rejected_at": "2024-01-15T10:31:00Z"
}
```

---

### 5.3 获取审批状态

**请求**:
```http
GET /v1/wallets/{wallet_id}/transactions/{tx_id}/approval
Authorization: Bearer <token>
```

**响应**:
```json
{
  "request_id": "request-1234567890abcdef",
  "tx_id": "tx-1234567890abcdef",
  "status": "pending",
  "threshold": 2,
  "current_approvals": 1,
  "approvals": [
    {
      "approver_id": "user-2",
      "approved_at": "2024-01-15T10:31:00Z",
      "comment": "同意支付"
    }
  ],
  "rejections": [],
  "pending_approvers": ["user-3"],
  "created_at": "2024-01-15T10:30:00Z",
  "expires_at": "2024-01-16T10:30:00Z"
}
```

---

### 5.4 取消交易

**请求**:
```http
POST /v1/wallets/{wallet_id}/transactions/{tx_id}/cancel
Authorization: Bearer <token>
```

**响应**:
```json
{
  "tx_id": "tx-1234567890abcdef",
  "status": "cancelled",
  "cancelled_at": "2024-01-15T10:31:00Z"
}
```

---

## 6. 地址管理 API

### 6.1 添加白名单地址

**请求**:
```http
POST /v1/wallets/{wallet_id}/addresses/whitelist
Content-Type: application/json
Authorization: Bearer <token>

{
  "address": "0x1234567890123456789012345678901234567890",
  "label": "供应商账户",
  "description": "主要供应商收款地址"
}
```

**响应**:
```json
{
  "address": "0x1234567890123456789012345678901234567890",
  "label": "供应商账户",
  "description": "主要供应商收款地址",
  "type": "whitelist",
  "added_at": "2024-01-15T10:30:00Z"
}
```

---

### 6.2 添加黑名单地址

**请求**:
```http
POST /v1/wallets/{wallet_id}/addresses/blacklist
Content-Type: application/json
Authorization: Bearer <token>

{
  "address": "0x9876543210987654321098765432109876543210",
  "reason": "可疑地址"
}
```

**响应**:
```json
{
  "address": "0x9876543210987654321098765432109876543210",
  "reason": "可疑地址",
  "type": "blacklist",
  "added_at": "2024-01-15T10:30:00Z"
}
```

---

### 6.3 列出地址列表

**请求**:
```http
GET /v1/wallets/{wallet_id}/addresses?type=whitelist
Authorization: Bearer <token>
```

**响应**:
```json
{
  "addresses": [
    {
      "address": "0x1234567890123456789012345678901234567890",
      "label": "供应商账户",
      "type": "whitelist",
      "added_at": "2024-01-15T10:30:00Z"
    }
  ],
  "total": 10
}
```

---

### 6.4 删除地址

**请求**:
```http
DELETE /v1/wallets/{wallet_id}/addresses/{address}
Authorization: Bearer <token>
```

**响应**:
```json
{
  "address": "0x1234567890123456789012345678901234567890",
  "status": "deleted",
  "deleted_at": "2024-01-20T15:45:00Z"
}
```

---

## 7. 资产查询 API

### 7.1 查询余额

**请求**:
```http
GET /v1/wallets/{wallet_id}/balance
Authorization: Bearer <token>
```

**响应**:
```json
{
  "wallet_id": "wallet-1234567890abcdef",
  "chain_type": "ethereum",
  "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  "balances": [
    {
      "token": "ETH",
      "symbol": "ETH",
      "balance": "10.5",
      "balance_wei": "10500000000000000000",
      "usd_value": "21000"
    },
    {
      "token": "USDT",
      "symbol": "USDT",
      "contract_address": "0xdAC17F958D2ee523a2206206994597C13D831ec7",
      "balance": "50000",
      "usd_value": "50000"
    }
  ],
  "total_usd_value": "71000",
  "updated_at": "2024-01-15T10:35:00Z"
}
```

---

### 7.2 查询交易历史

**请求**:
```http
GET /v1/wallets/{wallet_id}/transactions/history?limit=50&offset=0
Authorization: Bearer <token>
```

**响应**:
```json
{
  "transactions": [
    {
      "tx_id": "tx-1234567890abcdef",
      "tx_hash": "0x789abc...",
      "to": "0x1234567890123456789012345678901234567890",
      "value": "1000000000000000000",
      "status": "confirmed",
      "block_number": 18500000,
      "block_hash": "0xdef456...",
      "created_at": "2024-01-15T10:30:00Z",
      "confirmed_at": "2024-01-15T10:35:00Z"
    }
  ],
  "total": 100,
  "limit": 50,
  "offset": 0
}
```

---

## 8. 审批规则 API

### 8.1 设置审批规则

**请求**:
```http
POST /v1/wallets/{wallet_id}/approval-rules
Content-Type: application/json
Authorization: Bearer <token>

{
  "min_amount": "1000000000000000000",
  "max_amount": "100000000000000000000",
  "require_whitelist": true,
  "max_daily_amount": "1000000000000000000000",
  "max_daily_count": 10
}
```

**响应**:
```json
{
  "wallet_id": "wallet-1234567890abcdef",
  "rules": {
    "min_amount": "1000000000000000000",
    "max_amount": "100000000000000000000",
    "require_whitelist": true,
    "max_daily_amount": "1000000000000000000000",
    "max_daily_count": 10
  },
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

---

### 8.2 获取审批规则

**请求**:
```http
GET /v1/wallets/{wallet_id}/approval-rules
Authorization: Bearer <token>
```

**响应**:
```json
{
  "wallet_id": "wallet-1234567890abcdef",
  "rules": {
    "min_amount": "1000000000000000000",
    "max_amount": "100000000000000000000",
    "require_whitelist": true,
    "max_daily_amount": "1000000000000000000000",
    "max_daily_count": 10
  },
  "updated_at": "2024-01-15T10:30:00Z"
}
```

---

## 9. 批量操作 API

### 9.1 批量审批

**请求**:
```http
POST /v1/wallets/{wallet_id}/transactions/batch-approve
Content-Type: application/json
Authorization: Bearer <token>

{
  "tx_ids": ["tx-1", "tx-2", "tx-3"],
  "comment": "批量审批"
}
```

**响应**:
```json
{
  "results": [
    {
      "tx_id": "tx-1",
      "status": "approved",
      "approved_at": "2024-01-15T10:30:00Z"
    },
    {
      "tx_id": "tx-2",
      "status": "approved",
      "approved_at": "2024-01-15T10:30:01Z"
    },
    {
      "tx_id": "tx-3",
      "status": "approved",
      "approved_at": "2024-01-15T10:30:02Z"
    }
  ],
  "total": 3,
  "success": 3,
  "failed": 0
}
```

---

### 9.2 批量转账

**请求**:
```http
POST /v1/wallets/{wallet_id}/transactions/batch-transfer
Content-Type: application/json
Authorization: Bearer <token>

{
  "transfers": [
    {
      "to": "0x1234567890123456789012345678901234567890",
      "value": "1000000000000000000"
    },
    {
      "to": "0x9876543210987654321098765432109876543210",
      "value": "2000000000000000000"
    }
  ],
  "description": "批量支付"
}
```

**响应**:
```json
{
  "batch_id": "batch-1234567890abcdef",
  "transactions": [
    {
      "tx_id": "tx-1",
      "to": "0x1234567890123456789012345678901234567890",
      "value": "1000000000000000000",
      "status": "pending_approval"
    },
    {
      "tx_id": "tx-2",
      "to": "0x9876543210987654321098765432109876543210",
      "value": "2000000000000000000",
      "status": "pending_approval"
    }
  ],
  "total": 2,
  "created_at": "2024-01-15T10:30:00Z"
}
```

---

## 10. WebSocket API

### 10.1 连接

```javascript
const ws = new WebSocket('wss://api.multisig.example.com/v1/ws?token=your-token');

ws.onopen = () => {
  console.log('Connected');
  
  // 订阅钱包事件
  ws.send(JSON.stringify({
    action: 'subscribe',
    resource: 'wallet',
    wallet_id: 'wallet-1234567890abcdef'
  }));
};

ws.onmessage = (event) => {
  const message = JSON.parse(event.data);
  console.log('Message:', message);
};
```

### 10.2 事件类型

#### 交易状态变更
```json
{
  "event_type": "transaction_status_changed",
  "wallet_id": "wallet-1234567890abcdef",
  "tx_id": "tx-1234567890abcdef",
  "status": "approved",
  "timestamp": "2024-01-15T10:32:00Z"
}
```

#### 审批通知
```json
{
  "event_type": "approval_notification",
  "wallet_id": "wallet-1234567890abcdef",
  "tx_id": "tx-1234567890abcdef",
  "approver_id": "user-2",
  "action": "approved",
  "timestamp": "2024-01-15T10:31:00Z"
}
```

#### 交易确认
```json
{
  "event_type": "transaction_confirmed",
  "wallet_id": "wallet-1234567890abcdef",
  "tx_id": "tx-1234567890abcdef",
  "tx_hash": "0x789abc...",
  "block_number": 18500000,
  "timestamp": "2024-01-15T10:35:00Z"
}
```

---

## 11. 错误处理

### 11.1 错误响应格式

```json
{
  "error": {
    "code": "InsufficientApprovals",
    "message": "交易需要 2 个审批，当前只有 1 个",
    "request_id": "req-1234567890abcdef",
    "timestamp": "2024-01-15T10:30:00Z",
    "details": {
      "wallet_id": "wallet-1234567890abcdef",
      "tx_id": "tx-1234567890abcdef",
      "current_approvals": 1,
      "required_approvals": 2
    }
  }
}
```

### 11.2 错误代码

| 错误代码 | HTTP 状态码 | 说明 |
|---------|------------|------|
| `InvalidRequest` | 400 | 请求参数错误 |
| `Unauthorized` | 401 | 认证失败 |
| `Forbidden` | 403 | 权限不足 |
| `NotFound` | 404 | 资源不存在 |
| `InvalidWalletId` | 400 | 钱包ID无效 |
| `InsufficientApprovals` | 400 | 审批数量不足 |
| `AddressNotWhitelisted` | 400 | 地址不在白名单中 |
| `AddressBlacklisted` | 400 | 地址在黑名单中 |
| `AmountExceeded` | 400 | 金额超过限制 |
| `DailyLimitExceeded` | 400 | 每日限额超限 |
| `TransactionFailed` | 500 | 交易执行失败 |
| `RateLimitExceeded` | 429 | 请求频率超限 |
| `InternalError` | 500 | 服务器内部错误 |

---

## 12. SDK 示例

### 12.1 Go SDK

```go
package main

import (
    "github.com/multisig/sdk-go/multisig"
)

func main() {
    client := multisig.NewClient(&multisig.Config{
        Endpoint: "https://api.multisig.example.com",
        Token:    "your-token",
    })
    
    // 创建钱包
    wallet, err := client.CreateWallet(&multisig.CreateWalletRequest{
        Name:         "企业多签钱包",
        Threshold:    2,
        TotalMembers: 3,
        ChainType:    "ethereum",
    })
    
    // 创建交易
    tx, err := client.CreateTransaction(&multisig.CreateTransactionRequest{
        WalletID: wallet.WalletID,
        To:       "0x1234567890123456789012345678901234567890",
        Value:    "1000000000000000000",
    })
    
    // 审批交易
    err = client.ApproveTransaction(wallet.WalletID, tx.TxID, &multisig.ApproveRequest{
        Comment: "同意支付",
    })
}
```

### 12.2 Python SDK

```python
from multisig import MultiSigClient

client = MultiSigClient(
    endpoint="https://api.multisig.example.com",
    token="your-token"
)

# 创建钱包
wallet = client.create_wallet(
    name="企业多签钱包",
    threshold=2,
    total_members=3,
    chain_type="ethereum"
)

# 创建交易
tx = client.create_transaction(
    wallet_id=wallet["wallet_id"],
    to="0x1234567890123456789012345678901234567890",
    value="1000000000000000000"
)

# 审批交易
client.approve_transaction(
    wallet_id=wallet["wallet_id"],
    tx_id=tx["tx_id"],
    comment="同意支付"
)
```

### 12.3 JavaScript SDK

```javascript
const { MultiSigClient } = require('@multisig/sdk');

const client = new MultiSigClient({
  endpoint: 'https://api.multisig.example.com',
  token: 'your-token'
});

// 创建钱包
const wallet = await client.createWallet({
  name: '企业多签钱包',
  threshold: 2,
  totalMembers: 3,
  chainType: 'ethereum'
});

// 创建交易
const tx = await client.createTransaction({
  walletId: wallet.walletId,
  to: '0x1234567890123456789012345678901234567890',
  value: '1000000000000000000'
});

// 审批交易
await client.approveTransaction(wallet.walletId, tx.txId, {
  comment: '同意支付'
});
```

---

## 13. API 版本管理

### 13.1 版本策略

- URL 路径版本：`/v1/`, `/v2/`
- 向后兼容：保持旧版本 API 可用至少 12 个月
- 弃用通知：提前 6 个月通知 API 弃用

### 13.2 版本信息

```http
GET /v1/version
```

**响应**:
```json
{
  "version": "1.0.0",
  "api_version": "v1",
  "supported_chains": ["ethereum", "bitcoin", "bsc"],
  "build_date": "2024-01-15T10:30:00Z",
  "git_commit": "abc123def456"
}
```

---

**文档维护**: API 团队  
**最后更新**: 2024

