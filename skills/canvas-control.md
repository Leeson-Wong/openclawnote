# Canvas 控制 Skill

## 功能定位
这个技能定义了 OpenClaw 的 Canvas 管理系统，使用户能够呈现、操作和管理动态画布。

## 核心功能与操作指导

### Canvas 呈现
- **显示画布**
  - 命令：`/canvas present <url>`
  - 实现：`CanvasManager.present()`

- **隐藏画布**
  - 命令：`/canvas hide`
  - 实现：`CanvasManager.hide()`

### Canvas 导航
- **导航到页面**
  - 命令：`/canvas navigate <url>`
  - 实现：`CanvasNavigator.goToUrl()`

### Canvas 执行
- **运行脚本**
  - 命令：`/canvas eval <script>`
  - 示例：
    ```javascript
    document.querySelector('button').click();
    ```

### Canvas 快照
- **获取快照**
  - 命令：`/canvas snapshot`
  - 输出：PNG 格式的快照文件

### A2UI 推送
- **推送更新**
  - 命令：`/canvas push <data>`
  - 实现：`A2UIPusher.sendUpdate()`

## 使用场景
- 展示动态内容
- 远程控制画布
- 获取画布快照

## 注意事项
- 需要正确配置 Canvas 服务
- 监控资源使用情况
- 某些操作可能需要额外权限