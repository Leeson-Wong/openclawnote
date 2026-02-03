# 定时任务管理 Skill

## 功能定位
这个技能定义了 OpenClaw 的定时任务系统，使用户能够创建和管理周期性任务。

## 核心功能与操作指导

### Cron 管理
- **添加定时任务**
  - 命令：`/cron add <schedule> <command>`
  - 示例：`/cron add "0 9 * * *" "/reminders morning"`

- **删除定时任务**
  - 命令：`/cron remove <task_id>`
  - 实现：`CronManager.deleteTask(taskId)`

- **更新定时任务**
  - 命令：`/cron update <task_id> --schedule "new_schedule"`
  - 实现：`CronManager.updateSchedule()`

- **列出定时任务**
  - 命令：`/cron list`
  - 输出示例：
    ```
    Scheduled Tasks:
    ├─ Task 1 [ID: abc123] [0 9 * * *]
    ├─ Task 2 [ID: def456] [30 17 * * 5]
    └─ Task 3 [ID: ghi789] [*/10 * * * *]
    ```

### 定时任务特性
- **Cron 表达式支持**
  - 标准 cron 格式
  - 配置路径：`config/cron/tasks.json`
  - 示例配置：
    ```json
    {
      "tasks": [
        {
          "id": "abc123",
          "schedule": "0 9 * * *",
          "command": "/reminders morning"
        }
      ]
    }
    ```

- **任务日志**
  - 查看运行记录
  - 命令：`/cron logs <task_id>`

- **Wake 事件发送**
  - 支持发送唤醒事件
  - 实现：`WakeEventManager.sendEvent()`

## 使用场景
- 创建日常提醒
- 定期执行维护任务
- 监控任务执行情况

## 注意事项
- 正确配置 cron 表达式
- 检查任务冲突
- 监控任务执行日志