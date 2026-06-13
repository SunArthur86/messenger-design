# 全球消息系统架构全景对比

> 整合 11+ 款消息 APP 技术调研，提炼架构设计范式与选型指南
> 调研日期：2026-06-03

---

## 一、技术栈全景

| APP | 后端语言 | 存储引擎 | 连接协议 | 加密方案 |
|-----|---------|---------|---------|---------|
| **微信** | C++ | PaxosStore | Mars 自定义二进制 | 传输层加密 |
| **QQ** | C++ 内核 | 云原生存储 | TCP 长连接 | 传输层加密 |
| **飞书** | Go/Kitex | 自研 NoSQL | WebSocket | 传输层加密 |
| **钉钉** | Go + 阿里云 | 表格存储(LSM) | 长连接+推送 | 企业安全 |
| **企业微信** | C++/hikit | msgkv(LevelDB) | 长连接+短连接 | 企业安全 |
| **Telegram** | C++/MTProto | 自研分布式 | MTProto 2.0 | 可选 E2E |
| **WhatsApp** | Erlang/BEAM | Mnesia+SQLite | 自定义二进制 | Signal Protocol |
| **Signal** | Java/Go | SQLite | Noise+WebSocket | E2E 默认 |
| **Discord** | Elixir+Rust | ScyllaDB | WebSocket+WebRTC | 传输层加密 |
| **Slack** | Go/PHP | Vitess(MySQL) | WebSocket | 传输层加密 |
| **LINE** | Java/Akka | Redis+HBase | 自定义+MQTT | 可选 E2E |

---

## 二、消息同步范式分类

全球 IM 系统可归纳为**四大同步范式**：

### 范式一：信箱模型（Mailbox）

```
每用户一个信箱 → 消息写入所有接收者信箱 → 客户端增量拉取
代表：微信、企业微信
优点：协议简单、天然多端同步
缺点：群消息写放大（N 份存储）
```

### 范式二：Timeline 模型

```
每用户每会话一条 Timeline → 按位点同步 → 支持漫游
代表：飞书、抖音私信、阿里钉钉（混合）
优点：多端漫游、已读回执
缺点：存储成本高
```

### 范式三：MTProto 协议

```
客户端授权密钥 → 服务器 salt → 加密通信 → 差量同步
代表：Telegram
优点：自包含协议栈、可自建服务器
缺点：协议复杂度高
```

### 范式四：E2E 加密优先

```
双棘轮密钥协商 → 每条消息独立密钥 → 密文存储
代表：Signal、WhatsApp
优点：隐私极致保护
缺点：多设备同步困难、不支持服务器漫游
```

---

## 三、读写扩散决策矩阵

```
┌──────────────────────────────────────────────────────┐
│                    读写扩散决策树                      │
├──────────────────────────────────────────────────────┤
│                                                      │
│  群规模 < 500?                                       │
│    ├── 是 → 需要个性化消息状态?                       │
│    │       ├── 是 → 写扩散（微信、企业微信）          │
│    │       └── 否 → 读扩散                           │
│    └── 否 → 群规模 < 5000?                           │
│            ├── 是 → 混合方案（钉钉 DTIM）             │
│            └── 否 → 读扩散 + 按需拉取                 │
│                                                      │
│  业界数据：                                           │
│  • 微信群平均 5 人，写扩散可接受                      │
│  • 钉钉 1:30 扩散比，纯写扩散成本过高                 │
│  • Discord 万人 Guild，必须读扩散                     │
│  • Telegram 20 万人 Channel，纯读扩散 + 推送          │
└──────────────────────────────────────────────────────┘
```

---

## 四、核心技术趋势

### 4.1 后端语言演进

```
传统时代 (2000-2015)      现代时代 (2015-2025)       未来趋势
─────────────────      ──────────────────      ──────────
C++ 主流 (微信/QQ)      Go 微服务 (飞书/钉钉)     Rust 渗透 (Discord NIF)
Erlang (WhatsApp)       Elixir (Discord/Slack)   Zig/Go 混合
Java (LINE)             跨平台 C++ 内核 (QQ NT)
```

### 4.2 存储演进

| 代际 | 方案 | 代表 |
|------|------|------|
| 第一代 | MySQL 主从 | QQ 早期 |
| 第二代 | 自研 KV + 内存 | 微信 PaxosStore |
| 第三代 | 云原生 Serverless | 钉钉表格存储 |
| 第四代 | ScyllaDB/Cassandra 万亿消息 | Discord |

### 4.3 加密演进

```
无加密 → 传输层加密(TLS) → 可选 E2E → 默认 E2E
  │           │                  │           │
  早期QQ    飞书/钉钉         Telegram    Signal/WhatsApp
            微信/Slack
            Discord
```

---

## 五、架构选型速查表

### 场景一：社交 IM（类似微信）

| 模块 | 推荐 |
|------|------|
| 消息同步 | 信箱模型 + Sequence |
| 连接方案 | Mars 式长短竞速 |
| 存储 | 写扩散 + 冷热分离 |
| 加密 | 传输层加密即可 |
| 特点 | 大道至简，协议极简 |

### 场景二：企业 IM（类似飞书/钉钉）

| 模块 | 推荐 |
|------|------|
| 消息同步 | Timeline + 推优先 |
| 连接方案 | WebSocket + 厂商 Push |
| 存储 | 读写混合 + KV 分离 |
| 加密 | 传输层 + 企业审计 |
| 特点 | 多端同步 + 已读回执 + 漫游 |

### 场景三：隐私 IM（类似 Signal）

| 模块 | 推荐 |
|------|------|
| 消息同步 | Signal Protocol + 密封发送 |
| 连接方案 | Noise + WebSocket |
| 存储 | 本地 SQLite + 极简服务端 |
| 加密 | 默认 E2E + 双棘轮 |
| 特点 | 零数据保留，服务端无法解密 |

### 场景四：社区 IM（类似 Discord）

| 模块 | 推荐 |
|------|------|
| 消息同步 | Channel 模型 + WebSocket |
| 连接方案 | WebSocket + WebRTC(语音) |
| 存储 | ScyllaDB 万亿消息 |
| 加密 | 传输层加密 |
| 特点 | Guild 架构 + Bot 生态 + 语音 |

### 场景五：第三方 IM SDK（类似网易云信）

| 模块 | 推荐 |
|------|------|
| 消息同步 | 多租户隔离 + 统一消息路由 |
| 连接方案 | Protobuf/MQTT 多协议适配 |
| 存储 | 分库分表 + Redis 缓存 |
| 加密 | 租户级密钥隔离 |
| 特点 | SDK 多平台 + 低接入成本 |

---

## 六、面试高频考点

### Q1: 为什么微信选信箱模型而非 Timeline？

> 微信好友上限 5000，群平均 5 人，写扩散存储成本可接受。信箱模型协议最简单（单 sequence），客户端复杂度低，适合移动端省电。

### Q2: 为什么钉钉选读写混合？

> 企业 IM 扩散比 1:30（群平均远大于社交 IM），纯写扩散存储成本过高。混合方案：普通消息读扩散存一份，个性化状态（已读/删除）写扩散。

### Q3: WhatsApp 如何用 Erlang 支撑 20 亿用户？

> Erlang/BEAM 的 Actor 模型天然适合 IM：每个连接一个轻量进程，消息传递即进程间通信。WhatsApp 仅 50+ 工程师维护 20 亿用户系统。

### Q4: Discord 为什么从 MongoDB 迁移到 ScyllaDB？

> 2017 年 Discord 存储万亿消息。MongoDB 内存映射文件导致延迟抖动。ScyllaDB（C++ 重写的 Cassandra）提供稳定 P99 延迟 + 自动分片 + 列压缩。

### Q5: Signal 的 E2E 加密如何处理多设备？

> Signal 使用 Sender Keys + Sealed Sender：群消息用对称密钥加密（Sender Key 分发给成员），单聊用 X3DH + Double Ratchet。密封发送者——服务器连发送者是谁都不知道。

### Q6: 如何设计一个支持百万人的超级大群？

> 1. 纯读扩散：消息只存一份，所有成员读取同一流
> 2. 分层推送：在线成员 WebSocket 推送摘要，离线成员不推送
> 3. 按需加载：用户打开群时才拉取最新 N 条
> 4. 消息降级：控制消息/已读回执不扩散
> 5. 写入限流：单群消息频率上限

---

## 七、项目文件索引

| 文件 | 内容 |
|------|------|
| [docs/01-research.md](docs/01-research.md) | 飞书/微信/QQ 技术架构调研 |
| [docs/02-design.md](docs/02-design.md) | 基于 Timeline 的系统设计方案 |
| [docs/03-overseas.md](docs/03-overseas.md) | Telegram/WhatsApp/Signal/Discord/Slack/LINE/iMessage/KakaoTalk 调研 |
| [docs/04-domestic.md](docs/04-domestic.md) | 钉钉/企业微信/抖音/微博/第三方 IM PaaS 调研 |
| [docs/05-comparison.md](docs/05-comparison.md) | 本文件：全球全景对比与选型指南 |
| [docs/06-interview.md](docs/06-interview.md) | 60 道 IM 系统设计面试题 |
| [README.md](README.md) | 项目索引 |
