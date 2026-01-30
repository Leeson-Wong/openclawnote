# OpenClaw 架构笔记

<div align="center">

![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![Status](https://img.shields.io/badge/status-active-success.svg)

**OpenClaw 个人 AI 助手系统的完整架构文档与特性分析**

[架构文档](#openclaw-architecturemd) • [特性分析](#openclaw-featuresmd)

</div>

## 简介

OpenClaw 是一个功能强大的个人 AI 助手系统，采用模块化设计，支持多渠道接入、多模型配置和丰富的扩展功能。

本项目包含了 OpenClaw 系统的完整架构文档和特性分析，帮助开发者理解系统设计和技术实现。

## 文档说明

### openclaw-architecture.md
OpenClaw 的完整架构文档，包含 10 个详细的 Mermaid 架构图：

- **整体架构图** - 系统层级关系与模块划分
- **源码目录结构图** - 52 个源码目录的组织方式
- **协议层架构图** - 17 个协议 Schema 的设计
- **Skills 系统架构图** - 54 个技能的分类与组织
- **Extensions 系统架构图** - 20+ 个扩展的功能划分
- **数据流图** - 消息在系统中的流转过程
- **Gateway 核心文件结构图** - 核心模块的文件组织
- **Config 模块文件结构图** - 配置系统的详细设计
- **Sessions 模块文件结构图** - 会话管理系统的实现
- **部署架构图** - 多平台部署方案

### openclaw-features.md
OpenClaw 的核心特性分析，涵盖：

- **技术栈** - Node.js、TypeScript、pnpm 等技术选型
- **核心模块** - Gateway、Agents、Channels、Plugins、Sessions
- **技能系统** - 内置技能列表与功能说明
- **扩展系统** - 扩展能力与集成方式
- **架构亮点** - 模块化、可扩展性、多渠道支持等

## 快速开始

### 查看文档

直接在 GitHub 上查看文档：
- [openclaw-architecture.md](./openclaw-architecture.md)
- [openclaw-features.md](./openclaw-features.md)

### 本地查看

克隆仓库后使用支持 Mermaid 的 Markdown 查看器：

```bash
git clone https://github.com/Leeson-Wong/openclawnote.git
cd openclawnote
```

推荐查看工具：
- [VS Code](https://code.visualstudio.com/) + Mermaid 插件
- [Typora](https://typora.io/)
- [Obsidian](https://obsidian.md/)

## 技术栈

### 运行时
- **Node.js** ≥ 22
- **pnpm** 10.23.0

### 开发语言
- **TypeScript** 5.9.3

### 通信协议
- **WebSocket** - 实时通信
- **HTTP** - RESTful API

### 数据存储
- **SQLite** - 本地数据库

## 系统架构概览

OpenClaw 采用 **Gateway 控制平面 + Agent 助手产品** 的双架构设计：

```
┌─────────────────────────────────────────────────────────┐
│                      用户交互层                            │
│  WhatsApp | Telegram | Slack | Discord | iMessage | ...  │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│                    Gateway 控制平面                       │
│  • 配置管理  • 通道管理  • 插件系统  • 会话管理           │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│                    Agent 助手产品                         │
│  • 模型配置  • 技能系统  • 内存管理  • 任务调度           │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│                    扩展能力层                             │
│  • TTS  • Canvas  • 浏览器控制  • 文件操作  ...          │
└─────────────────────────────────────────────────────────┘
```

## 核心特性

### 🚀 多渠道支持
- 10+ 个通信渠道
- 统一的消息格式
- 可插拔的通道架构

### 🧠 模型灵活配置
- 支持多个 AI 模型提供商
- 动态模型切换
- 自定义模型参数

### 🔌 扩展系统
- 54 个内置技能
- 20+ 个扩展插件
- 自定义技能开发

### 📊 协议化设计
- 完整的 WebSocket 协议层
- 17 个协议 Schema 定义
- 类型安全的消息传递

### 🛠️ 部署灵活
- 支持本地运行
- Docker 部署
- 云平台部署

## 目录结构

```
openclawnote/
├── README.md                      # 本文件
├── openclaw-architecture.md       # 完整架构文档
├── openclaw-features.md          # 特性分析文档
└── .gitignore
```

## 相关项目

- [Clawdbot](https://github.com/clawdbot/clawdbot) - Clawdbot 主项目
- [OpenClaw](https://github.com/openclaw/openclaw) - OpenClaw 开源版本
- [ClawdbotConfig](https://github.com/Leeson-Wong/OpenclawdConfig) - Web 配置面板

## 贡献

欢迎提交 Issue 和 Pull Request！

## 许可证

MIT License

## 更新日志

### v1.0.0 (2026-01-31)
- ✅ 完成架构文档整理
- ✅ 创建 10 个 Mermaid 架构图
- ✅ 完成特性分析文档

---

<div align="center">

**Made with ❤️ by OpenClaw Community**

[⬆ 回到顶部](#openclaw-架构笔记)

</div>
