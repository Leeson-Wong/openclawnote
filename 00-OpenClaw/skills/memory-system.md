# 记忆系统 Skill

## 功能定位
这个技能定义了 OpenClaw 的记忆管理系统，使用户能够存储、检索和管理长期记忆。

## 核心功能与操作指导

### 长期记忆存储
- **添加记忆**
  - 命令：`/memory add <text>`
  - 实现：`MemoryManager.store()`

- **检索记忆**
  - 命令：`/memory search <query>`
  - 实现：`MemorySearchEngine.find()`

### 短期记忆
- **临时存储**
  - 自动维护短期对话历史
  - 实现：`ShortTermMemory.trackConversation()`

### 记忆更新
- **更新记忆**
  - 命令：`/memory update <id> <new_text>`
  - 实现：`MemoryUpdater.modifyEntry()`

### 记忆核心
- **记忆结构**
  - 使用 LanceDB 存储和查询记忆
  - 配置路径：`config/memory/lancedb.json`
  - 示例配置：
    ```json
    {
      "enabled": true,
      "storagePath": "/data/memory"
    }
    ```

## 使用场景
- 存储重要信息
- 检索过往对话
- 更新记忆内容

## 注意事项
- 敏感信息需要加密存储
- 定期清理过期记忆
- 监控记忆系统性能