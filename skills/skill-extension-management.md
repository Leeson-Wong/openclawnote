# 技能与扩展管理 Skill

## 功能定位
这个技能定义了 OpenClaw 的技能和扩展管理系统，使用户能够安装、更新和配置各种功能模块。

## 核心功能与操作指导

### 技能管理
- **安装技能**
  - 命令：`/skills install <skill_name>`
  - 实现：`SkillManager.install()`

- **更新技能**
  - 命令：`/skills update <skill_name>`
  - 实现：`SkillManager.update()`

- **删除技能**
  - 命令：`/skills remove <skill_name>`
  - 实现：`SkillManager.delete()`

- **列出技能**
  - 命令：`/skills list`
  - 输出示例：
    ```
    Installed Skills:
    ├─ Notion Integration [v1.2.3]
    ├─ GitHub Agent [v2.0.1]
    └─ Weather Skill [v1.0.0]
    ```

### 扩展管理
- **安装扩展**
  - 命令：`/extensions install <extension_name>`
  - 实现：`ExtensionInstaller.run()`

- **更新扩展**
  - 命令：`/extensions update <extension_name>`
  - 实现：`ExtensionUpdater.checkUpdates()`

- **列出扩展**
  - 命令：`/extensions list`
  - 输出示例：
    ```
    Installed Extensions:
    ├─ Core Channels [v3.1.0]
    ├─ Model Providers [v2.5.2]
    └─ Memory System [v1.8.4]
    ```

### 技能分类
- **生产力工具**
  - 包括：Notion、Obsidian、Apple Notes 等
  - 配置路径：`config/skills/productivity.json`

- **社交与通讯**
  - 包括：Discord、iMessage、Slack 等
  - 配置路径：`config/skills/social.json`

### 扩展分类
- **核心渠道扩展**
  - 包括：Telegram、WhatsApp、Slack 等
  - 配置路径：`config/extensions/channels.json`

- **认证提供者扩展**
  - 包括：Anthropic、OpenAI、Google 等
  - 配置路径：`config/extensions/auth-providers.json`

## 使用场景
- 安装新的功能模块
- 更新现有技能
- 查看已安装的扩展列表

## 注意事项
- 不同技能可能有依赖关系
- 更新前建议查看变更日志
- 扩展需要正确配置才能使用