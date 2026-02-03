# 模型管理 Skill

## 功能定位
这个技能定义了 OpenClaw 的模型管理系统，使用户能够配置、切换和监控多个 AI 模型。

## 核心功能与操作指导

### 模型支持
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

### 模型配置
- **默认模型**
  - 配置路径：`config/models/default.json`
  - 示例配置：
    ```json
    {
      "primary": "anthropic/claude-3-5-sonnet-20241022",
      "fallback": "openai/gpt-4o"
    }
    ```

- **上下文窗口**
  - 配置路径：`config/models/context.json`
  - 示例配置：
    ```json
    {
      "maxTokens": 8192,
      "windowStrategy": "sliding"
    }
    ```

### 认证管理
- **OAuth认证**
  - 配置路径：`config/auth/oauth.json`
  - 示例配置：
    ```json
    {
      "provider": "google",
      "clientId": "xxxxxx",
      "clientSecret": "yyyyyy"
    }
    ```

- **API密钥认证**
  - 配置路径：`config/auth/api-keys.json`
  - 示例配置：
    ```json
    {
      "anthropic": "sk-ant-xxxxxx",
      "openai": "sk-xxxxxx"
    }
    ```

### 认证提供者
- **Anthropic OAuth**
  - 实现：`AuthProvider.anthropic()`

- **Google Gemini CLI**
  - 命令：`/auth google setup`

## 使用场景
- 添加新的模型支持
- 配置模型参数
- 管理认证信息

## 注意事项
- 不同模型的上下文窗口限制不同
- 认证信息需要妥善保管
- 修改模型配置后需要重启服务