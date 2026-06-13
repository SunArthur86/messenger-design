# 💬 Messenger Design — 全球消息系统技术架构调研与设计

> 从开发工程师视角，深度剖析 **11+ 款** 国内外消息 APP 的技术架构，提炼设计范式，输出系统设计方案

🔗 **在线访问**: [https://sunarthur86.github.io/messenger-design/](https://sunarthur86.github.io/messenger-design/)

## ✨ v2.0 新增

- 🎨 **交互式文档阅读器** — Apple 风格 UI，左侧目录树 + 右侧渲染
- 🔍 **全文搜索** — 搜索所有文档内容，即时结果
- 🌙 **深色模式** — 跟随系统 + 手动切换
- 📖 **术语表** — 30+ IM 领域核心术语速查
- ⌨️ **键盘导航** — ←→ 切换文档，/ 快速搜索
- 📱 **移动端** — 侧边栏可折叠
- 🖨 **打印友好** — 打印时自动隐藏导航

## 📁 文档目录

| # | 文档 | 内容 | 篇幅 |
|---|------|------|------|
| 01 | [飞书/微信/QQ 架构调研](docs/01-research.md) | 三款国民级 IM 核心技术架构差异 | 14KB |
| 02 | [消息系统设计方案](docs/02-design.md) | 千万级 DAU 通用 IM 设计方案 | 14KB |
| 03 | [海外 IM 架构调研](docs/03-overseas.md) | Telegram/WhatsApp/Signal/Discord 等 | 15KB |
| 04 | [国内 IM 架构调研](docs/04-domestic.md) | 钉钉/企微/抖音/微博/第三方 PaaS | 12KB |
| 05 | [全景对比与选型](docs/05-comparison.md) | 技术栈/范式/读写扩散/选型速查 | 9KB |
| 06 | [面试题集 (60题)](docs/06-interview.md) | 6 级难度，基础→开放设计 | 36KB |
| 07 | [IM 术语表](docs/07-glossary.md) | 30+ 核心术语，含详细解释 | 20KB |

📊 **总计**: 7 篇文档，~120KB

## 🎯 覆盖的 IM 产品

### 国内
飞书 (Lark) · 微信 · QQ · 钉钉 · 企业微信 · 抖音私信 · 微博私信 · 网易云信 · 融云 · 环信

### 海外
Telegram · WhatsApp · Signal · Discord · Slack · LINE · iMessage · KakaoTalk

## ⌨ 快捷键

| 快捷键 | 功能 |
|--------|------|
| `←` `→` | 切换文档 |
| `/` | 快速搜索 |
| `D` | 切换深色模式 |
| `Esc` | 关闭搜索/侧边栏 |

## 🛠 技术栈

- 纯前端，零框架依赖
- 内置 Markdown 解析器（JS）
- localStorage 阅读进度
- Service Worker 离线支持 (PWA)
- GitHub Pages 部署

---

📅 2026 · 开发工程师视角 · 基于公开技术资料整理
