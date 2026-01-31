# 渠道管理 Skill

## 功能定位
这个技能涵盖了 OpenClaw 的渠道管理系统，使用户能够轻松添加、配置和管理多个通信渠道。

## 核心功能与操作指导

### 添加渠道
- **自动发现**
  - 系统会扫描可用的渠道插件
  - 实现：`ChannelDiscoveryService.scanAvailablePlugins()`
  - 示例：运行 `clawd channels scan` 查看可用渠道

- **向导式添加**
  - 提供交互式向导引导完成配置
  - 实现：`ChannelSetupWizard.run()`
  - 示例：运行 `clawd channels add --wizard`

- **OAuth 登录**
  - 支持通过 OAuth 快速登录（如 Telegram、Slack）
  - 实现：`OAuthManager.authenticate(channelType)`
  - 配置：需要在 `config/channels.json` 中设置 OAuth credentials

- **API 配置**
  - 支持 API Key、Token 等认证方式
  - 配置路径：`config/channels/<channel_name>.json`
  - 示例配置：
    ```json
    {
      "type": "telegram",
      "token": "YOUR_TELEGRAM_BOT_TOKEN"
    }
    ```

### 移除渠道
- **安全移除**
  - 移除前进行确认
  - 实现：`ChannelManager.removeChannel(channelId, { confirm: true })`
  - 命令：`clawd channels remove <channel_id>`

- **配置清理**
  - 自动清理相关配置和缓存
  - 实现：`ConfigCleaner.cleanup(channelId)`

### 渠道列表
- **完整列表**
  - 显示所有已配置渠道及其状态
  - 命令：`clawd channels list`
  - 输出示例：
    ```
    Available Channels:
    ├─ Telegram (Connected)
    ├─ Slack (Disconnected)
    └─ WhatsApp (Error: Invalid token)
    ```

### 渠道状态查询
- **连接状态**
  - 查询具体渠道的连接状态
  - 命令：`clawd channels status <channel_id>`

- **健康检查**
  - 执行渠道健康检查
  - 命令：`clawd channels health <channel_id>`

### 渠道能力查询
- **消息类型支持**
  - 查询渠道支持的消息类型
  - 命令：`clawd channels capabilities <channel_id> --message-types`

- **功能支持**
  - 查询渠道支持的功能（回复、表情反应等）
  - 命令：`clawd channels capabilities <channel_id> --features`

## 使用场景
- 添加新的通信渠道
- 配置和管理现有渠道
- 监控渠道状态
- 查询渠道能力

## 注意事项
- 部分渠道需要额外的服务支持（如 BlueBubbles for WhatsApp/iMessage）
- OAuth 认证需要正确配置回调地址
- 修改渠道配置后需要重启服务