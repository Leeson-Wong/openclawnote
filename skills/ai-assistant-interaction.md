# AI 助手交互 Skill

## 功能概述
这个技能涵盖了 OpenClaw 的核心 AI 助手交互功能，包括：

### 文本对话
- 多轮对话支持
- 流式输出
- Markdown 支持
- 消息引用

### 多模型支持
- Anthropic Claude 系列
- OpenAI 模型（GPT-4、GPT-3.5等）
- Google Gemini
- GitHub Copilot
- Oracle 模型
- Minimax
- 其他兼容模型

### 模型切换与选择
- 动态切换模型
- 模型优先级配置
- 模型测试验证

### 上下文窗口管理
- 最大上下文长度配置
- 智能截断
- Token 计数

### 思考模式控制
- 关闭模式 (off)
- 流式模式 (stream)
- 开启模式 (on)

### 系统提示词配置
- 自定义提示词
- 提示词模板
- 提示词继承

### 多Agent支持
- 主 Agent 和子 Agent
- Agent 间通信
- 并发支持
- 资源限制
- 自动清理

## 使用场景
- 自然语言对话
- 模型切换
- 上下文维护
- 思考过程展示

## 配置方法
通过 `/reasoning` 切换思考模式
通过 `/session new` 创建新会话