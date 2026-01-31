# 配置管理 Skill

## 功能定位
这个技能定义了 OpenClaw 的配置管理系统，使用户能够查看、修改和应用各种配置。

## 核心功能与操作指导

### 配置操作
- **读取配置**
  - 命令：`/config get <key>`
  - 实现：`ConfigManager.get()`

- **写入配置**
  - 命令：`/config set <key> <value>`
  - 实现：`ConfigManager.set()`

- **应用配置**
  - 命令：`/config apply`
  - 实现：`ConfigApplier.applyChanges()`

- **打补丁**
  - 命令：`/config patch <patch_file>`
  - 实现：`ConfigPatcher.applyPatch()`

### 配置类型
- **Agent配置**
  - 配置路径：`config/agents.json`
  - 示例配置：
    ```json
    {
      "defaultModel": "gpt-4o",
      "maxConcurrentSessions": 5
    }
    ```

- **渠道配置**
  - 配置路径：`config/channels/*.json`
  - 示例配置：
    ```json
    {
      "type": "telegram",
      "token": "123456789:ABCdefGhIJKlmNoPQRstuVWXyz"
    }
    ```

- **模型配置**
  - 配置路径：`config/models/*.json`
  - 示例配置：
    ```json
    {
      "provider": "anthropic",
      "apiKey": "sk-ant-xxxxxx"
    }
    ```

### 配置特性
- **环境变量替换**
  - 支持使用环境变量
  - 示例：`${ENV_VAR_NAME}`

- **配置合并**
  - 自动合并默认配置和用户配置
  - 实现：`ConfigMerger.merge()`

- **遗留配置迁移**
  - 自动迁移旧版本配置
  - 实现：`LegacyConfigMigrator.run()`

## 使用场景
- 修改系统设置
- 更新渠道认证信息
- 调整模型参数

## 注意事项
- 修改配置后需要重启服务生效
- 敏感信息需要加密存储
- 备份重要配置