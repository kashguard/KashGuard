# MPC 产品文档

**版本**: v1.0  
**最后更新**: 2024

---

## 📚 文档目录

本文档集提供了 MPC（多方安全计算）基础设施的完整产品文档，为 2B（企业多签钱包、托管）和 2C（个人钱包、智能钱包）产品提供底层支持。

### 核心文档

1. **[产品概述](./01-产品概述.md)** 📖
   - 产品简介和定位
   - 核心功能说明（阈值签名、密钥分片管理、分布式签名协议）
   - 技术特性（协议支持、性能特性、架构特性）
   - 应用场景（2B 和 2C）
   - 产品路线图

2. **[架构设计](./02-架构设计.md)** 🏗️
   - 分布式 MPC 架构
   - 核心组件设计（Coordinator、Participant、Storage）
   - 密钥分片管理
   - 签名流程（阈值签名、多方协作协议）
   - 高可用架构
   - 性能优化

3. **[API 设计](./03-API设计.md)** 🔌
   - API 概述（RESTful + gRPC）
   - 密钥管理 API
   - 签名服务 API
   - 节点管理 API
   - 协议管理 API
   - 链特定 API
   - SDK 示例

4. **[安全设计](./04-安全设计.md)** 🔒
   - MPC 特有的安全考虑
   - 密钥分片安全
   - 协议安全（GG18/GG20/FROST）
   - 通信安全
   - 访问控制
   - 审计与监控

5. **[部署指南](./05-部署指南.md)** 🚀
   - 部署概述
   - 环境要求
   - 单节点部署（开发测试）
   - 多节点部署（生产环境）
   - Kubernetes 部署
   - 监控配置
   - 备份与恢复

---

## 🎯 快速开始

### 对于产品经理
1. 阅读 [产品概述](./01-产品概述.md) 了解产品定位和功能
2. 查看应用场景了解 2B 和 2C 的使用场景
3. 查看产品路线图了解开发计划

### 对于架构师
1. 阅读 [架构设计](./02-架构设计.md) 了解系统架构
2. 理解分布式 MPC 架构和组件设计
3. 了解密钥分片管理和签名流程

### 对于开发人员
1. 阅读 [API 设计](./03-API设计.md) 了解 API 接口
2. 阅读 [部署指南](./05-部署指南.md) 进行环境搭建
3. 参考 SDK 示例进行集成开发

### 对于安全人员
1. 阅读 [安全设计](./04-安全设计.md) 了解安全措施
2. 关注密钥分片安全和协议安全
3. 了解 MPC 特有的安全考虑

### 对于运维人员
1. 阅读 [部署指南](./05-部署指南.md) 进行部署
2. 关注监控配置和备份恢复
3. 了解故障排查和性能调优

---

## 🔑 核心特性

### 阈值签名（TSS）
- ✅ 支持多种协议（GG18、GG20、FROST）
- ✅ 支持多种签名算法（ECDSA、EdDSA、Schnorr）
- ✅ 灵活的阈值配置（M-of-N）
- ✅ 无需私钥重构

### 密钥分片管理
- ✅ 分布式密钥生成（DKG）
- ✅ 密钥分片加密存储
- ✅ 安全的分片分发
- ✅ 密钥轮换（无需重新分发）

### 分布式架构
- ✅ 无单点故障
- ✅ 节点间对等通信
- ✅ 阈值容错
- ✅ 高可用部署

### 多链支持
- ✅ Bitcoin（BTC）
- ✅ Ethereum（ETH、Layer 2）
- ✅ EVM 链（BSC、Avalanche 等）
- ✅ Cosmos 生态

### 高性能
- ✅ 低延迟签名（< 200ms 目标）
- ✅ 高吞吐量（1000+ 签名/秒）
- ✅ 水平扩展
- ✅ 并发签名处理

---

## 🏗️ 架构特点

### 分布式架构
- **Coordinator 节点**: 协调签名流程，管理会话
- **Participant 节点**: 存储密钥分片，参与签名
- **无单点故障**: 阈值容错，只要达到阈值即可签名

### 协议支持
- **GG18/GG20**: 成熟的 ECDSA 阈值签名协议
- **FROST**: IETF 标准，支持 Schnorr 签名
- **可扩展**: 支持新协议集成

### 存储架构
- **密钥分片**: 加密存储在 Participant 节点
- **元数据**: PostgreSQL 存储
- **会话缓存**: Redis 缓存

---

## 📊 应用场景

### 2B 应用场景

#### 企业多签钱包
- 多人审批交易
- 灵活的阈值配置（2-of-3、3-of-5 等）
- 无需链上多签（降低 Gas 费）

#### 托管服务
- 机构级数字资产托管
- 去中心化存储（无单点故障）
- 符合合规要求

#### 机构钱包
- 基金、投资机构资产管理
- 多签管理
- 完整的审计追踪

#### 交易所热钱包
- 交易所热钱包管理
- 高性能签名
- 支持多链

### 2C 应用场景

#### 个人钱包
- 个人数字资产管理
- 无单点故障
- 社交恢复支持

#### 智能钱包
- 支持账户抽象
- MPC 签名 + 智能合约
- 更好的用户体验

#### 社交恢复钱包
- 基于 MPC 的恢复机制
- 无需助记词
- 社交验证

#### DeFi 应用
- DeFi 协议密钥管理
- 多签管理
- 自动化交易

---

## 🛠️ 技术栈

### 开发语言
- **推荐**: Go（并发支持好、性能优秀）
- **备选**: Rust（内存安全、性能优秀）

### 协议库
- **tss-lib**: Binance 开源（Go）
- **ZenGo-X/multi-party-ecdsa**: Rust 实现
- **FROST**: IETF 标准实现

### 存储
- **PostgreSQL**: 元数据存储
- **Redis**: 会话缓存
- **加密文件系统**: 密钥分片存储

### 通信
- **gRPC**: 节点间高效通信
- **WebSocket**: 实时通信和事件推送
- **HTTP/REST**: 客户端 API

### 部署
- **Docker**: 容器化部署
- **Kubernetes**: 生产环境部署

---

## 🔐 安全特性

### 密钥分片安全
- 密钥分片加密存储
- 安全的分片传输（TLS）
- 分片永不完整存在

### 协议安全
- 基于成熟的密码学协议
- 恶意节点防护
- 侧信道攻击防护

### 通信安全
- 节点间通信加密（TLS）
- 消息认证
- 防重放攻击

### 访问控制
- 节点认证
- 基于策略的授权
- 完整的审计日志

---

## 📈 产品路线图

### Phase 1: MVP（6个月）
- ✅ 2-of-3 阈值签名（GG18/GG20）
- ✅ 支持 ECDSA（secp256k1）
- ✅ 支持 BTC、ETH 主网
- ✅ 基础节点管理
- ✅ RESTful API

### Phase 2: 生产化（6个月）
- ✅ 通用阈值签名（任意 M-of-N）
- ✅ 支持 EdDSA（Ed25519）
- ✅ 支持更多区块链
- ✅ 密钥轮换
- ✅ 高可用架构

### Phase 3: 高级功能（6个月）
- ✅ FROST 协议支持（Schnorr 签名）
- ✅ 性能优化（< 200ms 延迟）
- ✅ 安全加固
- ✅ 智能合约集成
- ✅ 账户抽象支持

---

## 🚀 开始使用

### 快速部署

```bash
# 使用 Docker Compose 快速部署
git clone https://github.com/your-org/mpc-infrastructure
cd mpc-infrastructure
docker-compose up -d
```

### SDK 集成

```go
// Go SDK 示例
import "github.com/mpc/sdk-go/mpc"

client := mpc.NewClient(&mpc.Config{
    Endpoint: "https://api.mpc.example.com",
    Token:    "your-token",
})

// 创建密钥
key, _ := client.CreateKey(&mpc.CreateKeyRequest{
    Algorithm: "ECDSA",
    Curve:     "secp256k1",
    Threshold: 2,
    TotalNodes: 3,
})

// 签名
signature, _ := client.Sign(&mpc.SignRequest{
    KeyID:  key.KeyID,
    Message: []byte("Hello World"),
})
```

---

## 📞 支持与反馈

### 文档问题
如发现文档问题，请提交 Issue 或联系文档维护团队。

### 技术支持
- 技术文档: 本文档集
- API 文档: [API 设计](./03-API设计.md)
- 部署文档: [部署指南](./05-部署指南.md)

---

## 📝 文档维护

### 版本历史
- **v1.0** (2024): 初始版本

### 维护团队
- 产品管理团队
- 架构团队
- API 团队
- 安全团队
- 运维团队

---

## 📚 参考资料

### 学术论文
- **GG18**: "Fast Multiparty Threshold ECDSA with Fast Trustless Setup" (Gennaro & Goldfeder, 2018)
- **GG20**: "One Round Threshold ECDSA with Identifiable Abort" (Gennaro & Goldfeder, 2020)
- **FROST**: "Two-Round Threshold Schnorr Signatures with Fiat-Shamir via Homomorphic Commitments" (Komlo & Goldberg, 2020)

### 开源项目
- [tss-lib](https://github.com/binance-chain/tss-lib) - Binance 开源的 TSS 库
- [ZenGo-X/multi-party-ecdsa](https://github.com/ZenGo-X/multi-party-ecdsa) - ZenGo 开源的 MPC 实现
- [FROST](https://github.com/ZcashFoundation/frost) - IETF 标准实现

### 相关文档
- [BIP-340: Schnorr Signatures](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki)
- [EIP-191: Signed Data Standard](https://eips.ethereum.org/EIPS/eip-191)
- [EIP-712: Typed Structured Data Hashing](https://eips.ethereum.org/EIPS/eip-712)

---

**开始使用 MPC 基础设施，构建安全可靠的去中心化密钥管理！** 🔐

