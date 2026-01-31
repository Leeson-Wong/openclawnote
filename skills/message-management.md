# 消息管理 Skill

## 功能定位
这个技能定义了 OpenClaw 的消息管理系统，使用户能够高效地发送、接收和管理跨平台消息。

## 核心功能与操作指导

### 发送消息
- **多渠道发送**
  - 支持同时向多个渠道发送消息
  - 实现：`MultiChannelSender.send(targets, message)`
  - 示例：
    ```json
    {
      "targets": ["telegram", "slack"],
      "message": "Hello across platforms!"
    }
    ```

- **消息队列**
  - 异步消息队列系统
  - 配置路径：`config/queue.json`
  - 示例配置：
    ```json
    {
      "maxRetries": 5,
      "retryDelay": 3000
    }
    ```

- **延迟发送**
  - 支持定时发送消息
  - 命令：`/send --delay <seconds>`

- **富文本支持**
  - 支持 Markdown 和 HTML 格式
  - 实现：`RichTextFormatter.format()`

### 接收消息
- **实时接收**
  - WebSocket 实时接收
  - 实现：`MessageReceiver.listen()`

- **消息去重**
  - 自动过滤重复消息
  - 配置路径：`config/deduplication.json`

### 消息重试
- **自动重试**
  - 配置路径：`config/retry.json`
  - 示例配置：
    ```json
    {
      "maxAttempts": 3,
      "backoff": "exponential"
    }
    ```

### 消息确认
- **自动确认**
  - 自动发送确认反应（如 👍）
  - 配置路径：`config/ack.json`

### 消息策略控制
- **发送策略**
  - 配置路径：`config/send-policy.json`
  - 示例配置：
    ```json
    {
      "priority": "high",
      "throttle": 100
    }
    ```

## 使用场景
- 跨平台消息发送
- 消息接收和处理
- 消息重试管理
- 消息策略配置

## 注意事项
- 不同渠道的消息格式可能不同
- 延迟发送需要正确配置时间戳
- 消息队列的性能会影响发送速度