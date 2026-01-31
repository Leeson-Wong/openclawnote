# 会话管理 Skill

## 功能定位
这个技能定义了 OpenClaw 的会话管理系统，使用户能够创建、管理和监控多个独立的对话会话。

## 核心功能与操作指导

### 创建新会话
- **CLI 创建**
  - 命令：`/session new --type <type> --model <model>`
  - 实现：`SessionManager.create()`

- **自动创建**
  - 新对话来源自动创建会话
  - 配置路径：`config/sessions/auto-create.json`
  - 示例配置：
    ```json
    {
      "enabled": true,
      "defaultType": "chat",
      "defaultModel": "gpt-4o"
    }
    ```

### 删除会话
- **手动删除**
  - 命令：`/session remove <session_id>`
  - 实现：`SessionManager.delete(sessionId)`

- **自动清理**
  - 配置路径：`config/sessions/cleanup.json`
  - 示例配置：
    ```json
    {
      "inactiveDays": 7,
      "archiveBeforeDelete": true
    }
    ```

### 列出所有会话
- **完整列表**
  - 命令：`/sessions list`
  - 输出示例：
    ```
    Active Sessions:
    ├─ Session 1 (Active) [Model: gpt-4o]
    ├─ Session 2 (Idle) [Model: claude-3.5]
    └─ Session 3 (Archived)
    ```

### 会话历史记录
- **查看历史**
  - 命令：`/sessions history <session_id>`
  - 实现：`SessionHistoryViewer.getHistory()`

### 会话转录
- **导出会话**
  - 命令：`/sessions transcript <session_id> --format markdown`
  - 支持格式：markdown, json, text

### 会话标签管理
- **添加标签**
  - 命令：`/sessions tag <session_id> --add <tag>`
  - 实现：`SessionTagManager.addTags()`

### 会话优先级
- **设置优先级**
  - 命令：`/sessions priority <session_id> --level high`
  - 可选级别：low, medium, high

### 会话活跃度追踪
- **查看活跃度**
  - 命令：`/sessions activity <session_id>`
  - 输出示例：
    ```
    Session Activity Report:
    ├─ Last Active: 2 hours ago
    ├─ Message Count: 45
    └─ Engagement Score: 85%
    ```

## 使用场景
- 管理多个对话会话
- 监控会话状态
- 查看会话历史
- 导出会话记录

## 注意事项
- 删除会话前建议先归档
- 不同会话可以使用不同的模型
- 活跃度报告可以帮助优化资源分配