# 海外主流消息APP技术架构调研

> 调研时间：2026年6月 | 覆盖范围：Telegram / WhatsApp / Signal / Discord / Slack / LINE / iMessage / KakaoTalk

---

## 1. Telegram

**用户规模**：9亿+ MAU | **核心技术**：MTProto 协议、自建分布式数据中心

### 1.1 MTProto 2.0 自研协议

- **自定义二进制协议**：专为移动端优化，基于 AES-256-CTR 加密、RSA-2048 密钥交换、Diffie-Hellman 密钥协商
- **传输层**：支持 HTTP/HTTPS/UDP/TCP 多种传输方式，避免被防火墙封锁
- **授权密钥机制**：设备与数据中心建立长期授权密钥（Authorization Key），后续通信基于该密钥派生会话密钥，避免重复握手
- **消息ID去重**：每个消息有唯一递增 msg_id，服务端自动去重

### 1.2 分布式数据中心架构

- **全球5个DC集群**：分布在迈阿密、阿姆斯特丹、新加坡等地，用户就近接入
- **用户-DC绑定**：每个用户注册时绑定到特定DC，跨DC通信通过内部路由转发
- **云+本地混合存储**：
  - 云聊（Cloud Chat）：消息存储在服务端，支持多设备同步
  - 秘密聊天（Secret Chat）：端到端加密，消息仅存于两端设备，不留服务端痕迹

### 1.3 超大群与频道广播

- **超大群组**：支持最多20万成员，采用分片推送策略，仅推送活跃用户所需的增量更新
- **频道（Channel）**：一对多广播模型，消息仅存储一份，订阅者按需拉取，支持无限订阅者
- **Bot API**：提供 HTTP Bot API 和 MTProto Bot API，支持 inline keyboard、webhook 回调、支付等，生态极为丰富

### 1.4 技术栈概览

| 组件 | 技术 |
|------|------|
| 协议 | MTProto 2.0（自研） |
| 服务端语言 | C/C++（核心）、Go、Java |
| 存储 | 自研分布式存储 |
| 多媒体 | 自建 CDN + 边缘节点 |
| 加密 | AES-256-CTR + RSA-2048 + DH |

---

## 2. WhatsApp

**用户规模**：25亿+ MAU | **核心技术**：Erlang/BEAM、Signal Protocol 加密

### 2.1 Erlang/BEAM 高并发后端

- **Erlang 函数式语言**：基于 Actor 模型，每个用户连接对应一个轻量级进程（约2KB），单机支撑百万级并发连接
- **BEAM 虚拟机**：内置调度器管理百万级进程，支持跨核/跨机水平扩展，"Let it crash" 哲学实现自愈
- **Ejabberd（定制版）**：基于 Erlang 的 XMPP 服务器，WhatsApp 对其做了大量性能优化，负责消息路由、投递、离线存储
- **FreeBSD 操作系统**：选择 FreeBSD 而非 Linux，因其网络栈性能优异且发行版统一

### 2.2 端到端加密（Signal Protocol）

- **Signal Protocol**：采用 Curve25519 椭圆曲线、AES-256-CBC 加密
- **密钥管理**：每个设备生成身份密钥对，通过 PreKey 机制实现首次通信加密
- **群组加密**：采用 Sender Keys 协议，群成员共享对称密钥，发送者加密一次即可
- **默认开启**：所有一对一聊天和群聊默认端到端加密，WhatsApp 无法读取消息内容

### 2.3 消息存储与基础设施

- **Mnesia 分布式数据库**：Erlang 原生数据库，用于存储临时消息队列和用户状态，支持实时 KV 查找和动态重配置
- **本地 SQLite**：客户端本地存储完整聊天记录，服务端仅在消息未投递时暂存
- **YAWS 多媒体服务器**：基于 Erlang 的 Web 服务器，通过 WebSocket 处理多媒体上传/下载
- **Facebook 基础设施**：2017年后迁移至 Facebook 自有数据中心，2014年仅需~550台服务器、~11000核心

### 2.4 技术栈概览

| 组件 | 技术 |
|------|------|
| 后端语言 | Erlang |
| 虚拟机 | BEAM |
| 消息协议 | 定制版 XMPP |
| 数据库 | Mnesia（服务端）+ SQLite（客户端） |
| 加密 | Signal Protocol（Curve25519 + AES-256） |
| 操作系统 | FreeBSD |
| 多媒体 | YAWS + WebSocket |

---

## 3. Signal

**用户规模**：数千万 MAU | **核心技术**：Signal Protocol、极简隐私架构

### 3.1 Signal Protocol 双棘轮算法

- **双棘轮（Double Ratchet）**：结合 DH 棘轮和对称棘轮，每次消息交换派生新密钥，提供前向安全性（Forward Secrecy）和后向安全性（Post-Compromise Security）
- **X3DH 密钥协商**：扩展三重 Diffie-Hellman 握手，基于 Curve25519，支持离线异步通信
- **Sealed Sender（密封发送者）**：
  - 服务端无法看到发送者身份，仅知道接收者
  - 使用发送者证书（Sender Certificate）+ 投递令牌（Delivery Token）实现防伪造和限流
  - 信封使用接收者身份密钥二次加密
- **群组设计**：Sender Keys 协议，群成员共享对称密钥，发送者加密一次

### 3.2 极简架构与最小化数据保留

- **服务端不存储**：联系人、社交图谱、对话列表、位置、头像、群组成员等全部不保留
- **消息暂存即删**：消息在服务端仅存至投递成功，投递后立即删除
- **无广告无追踪**：由 Signal Foundation 基金会运营，完全开源（GPLv3）

### 3.3 技术栈概要

| 组件 | 技术 |
|------|------|
| 后端语言 | Java（服务端）、Rust（加密库） |
| 客户端 | Android（Java/Kotlin）、iOS（Swift）、Desktop（Electron） |
| 加密 | Signal Protocol（Double Ratchet + X3DH） |
| 椭圆曲线 | Curve25519 |
| 对称加密 | AES-256 |
| 推送 | FCM / APNs |

---

## 4. Discord

**用户规模**：2亿+ MAU，1100万+并发 | **核心技术**：Elixir/Rust、ScyllaDB、Guild 架构

### 4.1 Elixir + Rust 混合后端

- **Elixir 核心服务**：基于 BEAM VM，利用 Actor 模型实现高并发实时通信，1100万并发 WebSocket 连接
- **Rust NIF 加速**：通过 Rustler 桥接 Rust 原生函数（NIF），用于性能关键路径：
  - **SortedSet**：Rust 实现的高性能有序集合，支撑20万+成员 Guild 的成员列表实时更新，操作延迟 < 4μs（百万级元素）
  - Rust 负责数据密集型计算，Elixir 负责业务逻辑和并发管理
- **语音引擎**：基于 WebRTC，使用 Rust 实现音频处理（后自研音频编解码器）

### 4.2 ScyllaDB 消息存储

- **万亿级消息迁移**：从177个 Cassandra 节点迁移至72个 ScyllaDB 节点，9天完成
- **ScyllaDB 优势**：C++ 重写 Cassandra 兼容引擎，延迟更低吞吐更高
- **Elasticsearch**：消息全文检索
- **20+ ScyllaDB 集群**，近500个节点

### 4.3 Guild 架构与 Bot 系统

- **Guild（服务器）**：核心逻辑单元，包含频道、角色、权限、成员列表等，所有实时事件围绕 Guild 路由
- **频道模型**：文本频道、语音频道、分类频道，支持细粒度权限控制
- **Slash Command**：应用命令交互框架，Bot 注册斜杠命令，用户通过 UI 触发
- **Bot 系统**：基于 WebSocket Gateway + REST API，支持 Bot 接收所有 Guild 事件并响应
- **Gateway**：WebSocket 长连接，客户端/Bot 通过 Gateway 接收事件推送，支持分片（Sharding）

### 4.4 技术栈概要

| 组件 | 技术 |
|------|------|
| 后端语言 | Elixir + Rust（NIF） |
| 数据库 | ScyllaDB、Elasticsearch |
| 实时通信 | WebSocket（Gateway） |
| 语音/视频 | WebRTC + 自研编解码 |
| 消息协议 | 自定义 JSON over WebSocket |
| 运维 | 自建基础设施 |

---

## 5. Slack

**用户规模**：千万级 DAU | **核心技术**：WebSocket、AWS 微服务、Cell 架构

### 5.1 消息模型与实时平台

- **Channel + Thread 模型**：消息归属 Channel，支持 Thread 回复实现话题式讨论
- **双 API 设计**：
  - **Web API**：基于 HTTP，用于发消息、登录、管理等操作
  - **Real-Time API（RTM）**：基于 WebSocket，用于消息推送、打字提示、在线状态等实时事件
- **登录流程**：客户端通过 Web API 登录，返回 Workspace 快照（snapshot）+ WebSocket URL，客户端连接 WebSocket 接收实时事件
- **Cursor 分页**：聊天历史采用游标分页（cursor-based pagination），支持高效大规模消息翻页

### 5.2 AWS 微服务 + Cell 架构

- **Cell-Based 架构**：从单体迁移至 Cell 架构，每个 Cell 独立承载 Workspace，实现故障隔离
- **核心服务**：
  - **Gateway Server**：有状态内存服务，维护 channel_id→客户端连接映射，Actor 模型
  - **Dispatcher**：Pub/Sub 消息分发，跨数据中心广播
  - **Snapshot Service（Flannel）**：应用级边缘缓存，缓存 Workspace 引导数据，100万+ QPS
- **Vitess 分片**：Vitess 管理 MySQL 分片，按 channel_id 分片，支持自动重新分片
- **Edge Proxy（Envoy）**：SSL 终止 + WebSocket 代理，一致性哈希路由

### 5.3 技术栈概要

| 组件 | 技术 |
|------|------|
| 后端语言 | Java |
| 数据库 | MySQL（Vitess 分片） |
| 缓存 | Redis、Flannel（应用级边缘缓存） |
| 实时通信 | WebSocket（Envoy 代理） |
| 搜索 | Apache Solr |
| 基础设施 | AWS（S3、CloudFront） |
| 架构 | Cell-Based 微服务 |

---

## 6. LINE

**用户规模**：2亿+ MAU | **核心技术**：长连接、Akka Actor、多区域部署

### 6.1 自研消息队列与长连接方案

- **长连接方案**：客户端与服务端维持 TCP 长连接（自研协议），支持消息实时推送
- **Akka Actor System**：基于 Scala/Akka 构建聊天服务器，利用 Actor 模型实现高并发并行处理
  - ChatSupervisor → ChatRoomActor → UserActor 三层 Actor 层级
  - 每个 ChatRoom 一个 Actor，每个 User 一个 Actor，消息通过 Actor 传递异步处理
- **Redis Cluster Pub/Sub**：跨服务器消息同步，聊天室分布在不同服务器上，通过 Redis Pub/Sub 实时同步评论

### 6.2 多区域部署与贴纸系统

- **多区域部署**：日本、韩国、东南亚等多区域，用户就近接入
- **贴纸系统架构**：
  - 贴纸商店为独立微服务，支持购买/赠送/下载
  - 贴图资源通过 CDN 分发，客户端缓存常用贴纸
  - 贴纸消息通过消息通道传输元数据（贴纸ID），客户端从本地/CDN 加载资源
- **技术栈**：Java / Erlang / Redis / HBase / Kafka / Armeria（LINE 自研 RPC 框架）

### 6.3 技术栈概要

| 组件 | 技术 |
|------|------|
| 后端语言 | Java（Scala/Akka）、Erlang |
| 数据库 | HBase、MySQL |
| 缓存/同步 | Redis Cluster（Pub/Sub） |
| 消息队列 | Kafka |
| RPC | Armeria（自研） |
| 负载均衡 | 自研 LBaaS |
| 多媒体 | CDN 分发 |

---

## 7. iMessage

**用户规模**：10亿+ 活跃设备 | **核心技术**：Apple 生态整合、APNs、E2E 加密

### 7.1 Apple 设备间通信架构

- **APNs（Apple Push Notification Service）**：所有消息通过 APNs 推送至目标设备
  - 客户端与 APNs 维持长连接，APNs 负责消息路由和投递
  - 支持静默推送（background push）唤醒应用同步消息
- **CloudKit 同步**：iCloud 中消息通过 CloudKit 端到端加密存储，实现跨设备消息同步
  - 开启"高级数据保护"后，CloudKit 数据端到端加密
  - 使用 iCloud Keychain 同步加密密钥

### 7.2 端到端加密与 SMS 融合

- **E2E 加密**：每条消息使用接收设备公钥加密，仅目标设备硬件密钥可解密
- **SMS 融合**：iMessage 自动检测对方是否 Apple 用户，非 Apple 用户回退 SMS/MMS（绿/蓝气泡）

### 7.3 技术栈概要

| 组件 | 技术 |
|------|------|
| 推送 | APNs |
| 云同步 | CloudKit（E2E 加密） |
| 加密 | Apple 自研（设备公钥加密） |
| 身份服务 | IDS（Identity Services） |
| 回退通道 | SMS/MMS（运营商） |
| 平台 | iOS / macOS / watchOS 原生 |

---

## 8. KakaoTalk

**用户规模**：5000万+ MAU（韩国国民IM） | **核心技术**：超级App平台、微服务架构

### 8.1 超级App平台架构

- **超级App模式**：核心聊天 + Kakao Pay（支付）、Kakao T（出行）、Kakao Friends（电商）、游戏等深度整合
- **微服务架构**：不同服务独立扩展，通过高性能 API 网关通信，统一身份认证和用户图谱
- **SDK/开放API**：提供完善开发者 SDK，外部开发者可接入 KakaoTalk 平台
- **2025年重大改版**：与 ChatGPT 深度整合，引入 AI 功能

### 8.2 消息系统与安全

- **分布式架构**：面向日均数十亿消息的分布式系统，注重可扩展性和可靠性
- **Secret Chat E2E 加密**：秘密聊天模式实现端到端加密，需复杂密钥管理和跨设备同步
- **贴纸系统**：Kakao Friends 等IP贴纸为核心功能和重要收入来源

### 8.3 技术栈概要

| 组件 | 技术 |
|------|------|
| 后端语言 | Java 为主 |
| 架构 | 微服务 + API Gateway |
| 数据库 | 分布式存储（HBase 等） |
| 加密 | 自研 E2E（Secret Chat） |
| AI | JAX + TPU（语言模型训练） |
| 云服务 | OpenStack（Kakao 自建云） |

---

## 横向对比表

| 维度 | Telegram | WhatsApp | Signal | Discord | Slack | LINE | iMessage | KakaoTalk |
|------|----------|----------|--------|---------|-------|------|----------|-----------|
| **后端语言** | C/C++ | Erlang | Java/Rust | Elixir+Rust | Java | Java/Scala | Obj-C/Swift | Java |
| **核心协议** | MTProto 2.0 | 定制XMPP | Signal Protocol | 自定义(WS) | HTTP+WebSocket | 自研TCP | APNs+CloudKit | 自研 |
| **E2E加密** | 仅Secret Chat | 默认开启(Signal) | 默认开启 | 无 | 传输加密(TLS) | Secret Chat | 默认开启 | Secret Chat |
| **加密协议** | 自研(AES+RSA) | Signal Protocol | Signal Protocol | — | TLS 1.3 | — | Apple自研 | 自研 |
| **消息存储** | 云端+本地 | 本地SQLite | 服务端不保留 | ScyllaDB | MySQL(Vitess) | HBase | CloudKit | 分布式存储 |
| **实时通信** | MTProto长连接 | XMPP+WebSocket | WebSocket | WebSocket | WebSocket | TCP长连接 | APNs | TCP长连接 |
| **群组上限** | 20万 | 1024 | 1000 | 20万(主服务器) | 单频道1万 | 500 | 群聊有限 | 有限 |
| **基础设施** | 自建5个DC | Facebook DC | 自建+云 | 自建 | AWS | 自建多区域 | Apple IDC | Kakao云 |
| **开源程度** | 客户端+协议开源 | 不开源 | 完全开源 | 不开源 | 不开源 | 部分开源 | 不开源 | 不开源 |
| **核心特色** | 速度+Bot生态 | 极简+规模 | 隐私安全 | 社区+语音 | 企业协作 | 贴纸+生态 | Apple生态 | 超级App |
| **Bot/插件** | Bot API丰富 | 无 | 无 | Bot+Slash | App+Slash | Bot | 无 | 部分开放 |

---

## 关键技术趋势总结

1. **Erlang/BEAM 家族**：WhatsApp、Discord（Elixir）均选择 BEAM VM 处理海量并发连接，Actor 模型是消息系统的事实标准
2. **Signal Protocol 成行业标准**：WhatsApp、Signal、Facebook Messenger 等均采用 Signal Protocol 作为 E2E 加密基础
3. **云端 vs 本地存储**：WhatsApp/Signal 选择本地存储+服务端暂存，Telegram/Discord/Slack 选择云端持久化
4. **Rust 渗入后端**：Discord 大量使用 Rust NIF 加速性能关键路径，Signal 加密库也使用 Rust
5. **Cell/分片架构**：Slack 的 Cell 架构、Discord 的 Guild 分片、WhatsApp 的 Mnesia 分布式，均体现分布式系统趋势
6. **超级App趋势**：KakaoTalk、LINE 从聊天延伸到支付/出行/电商等全场景，微服务 + API Gateway 成为必选架构
