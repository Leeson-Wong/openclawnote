# AutoGPT 项目 - 基本信息

## 项目来源
- GitHub: https://github.com/Significant-Gravitas/AutoGPT
- 官网: https://agpt.co/
- Discord: https://discord.gg/autogpt
- Twitter: https://twitter.com/Auto_GPT

## 项目简介
AutoGPT 是一个强大的平台，允许创建、部署和管理持续运行的 AI Agent，以自动化复杂的工作流。其使命是提供工具，让你专注于重要的事情：构建、测试、委派任务。

**核心愿景**: 让 AI 对每个人都可访问，既可用也可构建。

## 许可证（双重许可）

### AutoGPT Platform（新平台）
- **许可**: Polyform Shield License
- **说明**: 开发中的平台，用于构建、部署和管理 Agent
- **限制**: 此许可证禁止在未经许可的情况下用于商业生产环境

### AutoGPT Classic（经典版本）
- **许可**: MIT License
- **包含内容**:
  - 独立的 AutoGPT Agent
  - Forge（Agent 构建工具包）
  - agbenchmark（Agent 基准测试）
  - AutoGPT Classic GUI
- ** Sister 项目**: GravitasML, Code Ability（也使用 MIT 许可）

## 核心特点

### AutoGPT Platform（新平台）
1. **低代码 Agent 构建器** - 直观的低代码界面设计和配置 Agent
2. **工作流管理** - 通过连接块构建自动化工作流
3. **即用型 Agent** - 预配置的 Agent 库，可直接使用
4. **部署控制** - 管理 Agent 生命周期（测试到生产）
5. **监控和分析** - 跟踪 Agent 性能，提供洞察

### AutoGPT Classic
1. **Forge** - 快速构建自定义 Agent 的工具包
2. **Benchmark** - 严格的测试环境，客观评估 Agent 性能
3. **UI** - 用户友好的界面控制和监控 Agent
4. **CLI** - 统一的命令行工具

## 托管选项

### 1. 自托管（免费！）
- 下载代码自行部署
- 需要技术配置

### 2. 云端托管
- **当前状态**: 闭测（Closed Beta）
- **公开发布**: 即将到来
- **加入等待列表**: https://bit.ly/3ZDijAI

## 系统要求

### 硬件要求
- CPU: 4+ 核心（推荐）
- RAM: 最少 8GB，推荐 16GB
- 存储: 至少 10GB 可用空间

### 软件要求
- **操作系统**:
  - Linux (Ubuntu 20.04 或更新版本，推荐)
  - macOS (10.15 或更新版本)
  - Windows 10/11 with WSL2

- **必需软件**:
  - Docker Engine (20.10.0 或更新版本)
  - Docker Compose (2.0.0 或更新版本)
  - Git (2.30 或更新版本)
  - Node.js (16.x 或更新版本)
  - npm (8.x 或更新版本)
  - VSCode (1.60 或更新版本) 或任何现代代码编辑器

### 网络要求
- 稳定的互联网连接
- 访问所需端口（将在 Docker 中配置）
- 能够进行出站 HTTPS 连接

## 核心架构

### AutoGPT Platform
- **前端**: 用户交互界面
  - Agent Builder
  - Workflow Management
  - Deployment Controls
  - Ready-to-Use Agents
  - Agent Interaction
  - Monitoring and Analytics

- **服务器**: Agent 运行环境
  - 源代码（核心逻辑）
  - 基础设施（可靠和可扩展的性能）
  - Marketplace（预构建 Agent 市场）

### AutoGPT Classic
- **Forge**: Agent 构建工具包
- **Benchmark**: 性能测试框架
- **UI**: 前端界面
- **CLI**: 命令行工具

## 安装方式

### 快速一键设置（推荐用于本地托管）

#### macOS/Linux
```bash
curl -fsSL https://setup.agpt.co/install.sh -o install.sh && bash install.sh
```

#### Windows (PowerShell)
```bash
powershell -c "iwr https://setup.agpt.co/install.bat -o install.bat; ./install.bat"
```

这将自动安装依赖、配置 Docker 并启动本地实例。

### 手动安装
参考官方自托管指南: https://docs.agpt.co/platform/getting-started/

## 示例 Agent

### 1. 从热门话题生成病毒式视频
- 读取 Reddit 话题
- 识别热门话题
- 自动基于内容创建短视频

### 2. 从视频识别最佳引语用于社交媒体
- 订阅 YouTube 频道
- 发布新视频时转录
- 使用 AI 识别最有影响力的引语
- 生成摘要并自动发布到社交媒体

## Sister Projects

### Agent Protocol
- **标准**: 由 AI Engineer Foundation 制定的 Agent 协议标准
- **目的**: 统一标准，确保与当前和未来应用程序的无缝兼容
- **作用**: 标准化 Agent 到前端和基准测试的通信路径

## 资源链接

- **官方文档**: https://docs.agpt.co
- **平台文档**: https://docs.agpt.co/platform/getting-started/
- **AutoGPT Platform 介绍**: https://agpt.co/blog/introducing-the-autogpt-platform
- **贡献指南**: https://github.com/Significant-Gravitas/AutoGPT/blob/master/CONTRIBUTING.md
- **Agent Protocol**: https://agentprotocol.ai/
- **Forge 教程**: https://medium.com/@aiedge/autogpt-forge-e3de53cc58ec
- **Star 历史**: https://star-history.com/#Significant-Gravitas/AutoGPT

## 关键指标

- **GitHub Stars**: 高活跃度（具体数字需确认）
- **社区**: 活跃的 Discord 社区
- **贡献者**: 多个贡献者
- **语言支持**: 多语言文档（德语、西班牙语、法语、日语、韩语、葡萄牙语、俄语、中文）

## 技术栈

### AutoGPT Platform
- **前端**: Node.js, React（推测）
- **后端**: Python（推测）
- **容器化**: Docker, Docker Compose
- **协议**: Agent Protocol

### AutoGPT Classic
- **核心**: Python
- **构建工具**: Forge
- **测试**: agbenchmark
- **界面**: Web UI
- **CLI**: 命令行工具

## 核心使命

"提供工具，让你专注于重要的事情"：
- 🏗️ **构建** - 为惊人的事物奠定基础
- 🧪 **测试** - 将你的 Agent 调优到完美
- 🤝 **委派** - 让 AI 为你工作，让你的想法成为现实
