# 渠道管理 Skill

## 功能定位
这个技能定义了 OpenClaw 的渠道管理系统，使用户能够添加、配置和监控多个通信渠道的状态和能力。

## 核心功能与操作指导

### 添加渠道
- **自动发现**
  - 扫描可用渠道插件
  - 实现：`ChannelDiscoveryService.scanAvailablePlugins()`
  - 示例：`/channels scan`

- **向导式添加**
  - 交互式配置向导
  - 实现：`ChannelSetupWizard.run()`
  - 示例：`/channels add --wizard`

- **OAuth 登录**
  - 支持主流平台的 OAuth 认证
  - 配置路径：`config/auth/oauth.json`
  - 示例配置：
    ```json
    {
      "provider": "slack",
      "clientId": "xxxxxx",
      "clientSecret": "yyyyyy"
    }
    ```

- **API 配置**
  - 支持多种认证方式
  - 配置路径：`config/channels/<channel_name>.json`
  - 示例配置：
    ```json
    {
      "type": "telegram",
      "token": "123456789:ABCdefGhIJKlmNoPQRstuVWXyz"
    }
    ```

### 移除渠道
- **安全移除**
  - 命令：`/channels remove <channel_id>`
  - 实现：`ChannelManager.safeRemove()`

- **配置清理**
  - 自动清理相关文件
  - 实现：`ConfigCleaner.fullCleanup()`

### 渠道列表
- **完整列表**
  - 命令：`/channels list`
  - 输出示例：
    ```
    Configured Channels:
    ├─ Telegram [Active]
    │  ├─ Token: Valid
    │  └─ Last Activity: 2 hours ago
    ├─ Slack [Inactive]
    └─ WhatsApp [Error]
    ```

### 渠道状态查询
- **实时状态**
  - 命令：`/channels status <channel_id>`
  - 输出示例：
    ```
    Channel Status Report:
    ├─ Connection: Established
    ├─ Latency: 150ms
    └─ Error Rate: 0%
    ```

### 渠道能力查询
- **消息类型**
  - 命令：`/channels capabilities <channel_id> --messages`
  - 输出示例：
    ```
    Supported Message Types:
    ├─ Text
    ├─ Images
    ├─ Files
    └─ Reactions
    ```

- **功能支持**
  - 命令：`/channels capabilities <channel_id> --features`
  - 输出示例：
    ```
    Supported Features:
    ├─ Thread Support: Yes
    ├─ DM Support: Yes
    └─ Inline Buttons: No
    ```

## 使用场景
- 新渠道接入
- 渠道状态监控
- 能力评估
- 配置验证

## 注意事项
- 不同渠道的能力差异较大
- OAuth 需要正确配置回调 URL
- 渠道配置变更需要服务重启