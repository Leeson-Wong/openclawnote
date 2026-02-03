# AI 助手交互 Skill

## 功能定位
这个技能定义了 OpenClaw 的核心 AI 交互能力，使用户能够与多个 AI 模型进行自然语言对话，并管理对话的各个方面。

## 核心功能与操作指导

### 文本对话
- **多轮对话**
  - 自动维护上下文历史
  - 实现：`ConversationManager.maintainContext()`
  - 示例：连续对话中自动保留上下文

- **流式输出**
  - 实时返回模型生成内容
  - 实现：`StreamingResponseHandler.process()`
  - 命令：无需特殊命令，系统默认启用

- **Markdown 支持**
  - 完整支持 Markdown 格式
  - 实现：`MessageFormatter.formatMarkdown()`
  - 示例：
    ```markdown
    # 标题
    - 列表项
    [链接](https://example.com)
    ```

- **消息引用**
  - 支持引用特定消息
  - 实现：`MessageReferencer.handleReplyTags()`
  - 示例：使用 `[[reply_to:<id>]]`

### 多模型支持
- **Anthropic Claude**
  - 配置路径：`config/models/anthropic.json`
  - 示例配置：
    ```json
    {
      "apiKey": "sk-ant-xxxxxx",
      "models": [
        {
          "id": "claude-3-5-sonnet-20241022",
          "name": "Claude 3.5 Sonnet"
        }
      ]
    }
    ```

- **OpenAI 模型**
  - 配置路径：`config/models/openai.json`
  - 示例配置：
    ```json
    {
      "apiKey": "sk-xxxxxx",
      "models": [
        {
          "id": "gpt-4o",
          "name": "GPT-4o"
        }
      ]
    }
    ```

### 模型切换与选择
- **动态切换**
  - 命令：`/model switch <model_alias>`
  - 实现：`ModelSwitcher.changeModel()`

- **模型优先级**
  - 配置路径：`config/models/priority.json`
  - 示例配置：
    ```json
    {
      "primary": "anthropic/claude-3-5-sonnet-20241022",
      "fallback": "openai/gpt-4o"
    }
    ```

### 上下文窗口管理
- **最大上下文长度**
  - 配置路径：`config/models/context.json`
  - 示例配置：
    ```json
    {
      "maxTokens": 8192,
      "windowStrategy": "sliding"
    }
    ```

### 思考模式控制
- **切换模式**
  - 命令：`/reasoning <mode>`
  - 可选模式：off, stream, on

### 系统提示词配置
- **自定义提示词**
  - 配置路径：`config/prompts/system.json`
  - 示例配置：
    ```json
    {
      "default": "You are a helpful assistant.",
      "creative": "Be creative and imaginative."
    }
    ```

### 多Agent支持
- **创建子 Agent**
  - 命令：`/agents spawn <task>`
  - 实现：`AgentManager.createSubAgent()`

- **Agent 间通信**
  - 实现：`AgentCommunicator.sendMessage(agentId, message)`

## 使用场景
- 自然语言对话
- 模型切换
- 上下文维护
- 思考过程展示

## 注意事项
- 不同模型的上下文窗口限制不同
- 思考模式会影响响应速度
- 子 Agent 的资源消耗需要监控