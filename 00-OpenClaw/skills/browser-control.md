# 浏览器控制 Skill

## 功能定位
这个技能定义了 OpenClaw 的浏览器控制系统，使用户能够通过自然语言控制浏览器的各种操作。

## 核心功能与操作指导

### 浏览器启动/停止
- **启动浏览器**
  - 命令：`/browser start`
  - 实现：`BrowserManager.start()`

- **停止浏览器**
  - 命令：`/browser stop`
  - 实现：`BrowserManager.stop()`

### 页面导航
- **打开页面**
  - 命令：`/browser open <url>`
  - 实现：`PageNavigator.openUrl()`

- **返回上一页**
  - 命令：`/browser back`
  - 实现：`PageNavigator.goBack()`

### 页面快照
- **获取快照**
  - 命令：`/browser snapshot`
  - 输出：Markdown 格式的页面内容

### 页面截图
- **截取页面**
  - 命令：`/browser screenshot`
  - 输出：PNG 格式的截图文件

### 元素操作
- **点击元素**
  - 命令：`/browser click <selector>`
  - 示例：`/browser click "button.submit"`

- **输入文本**
  - 命令：`/browser type <selector> <text>`
  - 示例：`/browser type "input.username" "admin"`

### Chrome 扩展中继
- **连接扩展**
  - 需要安装 Browser Relay 扩展
  - 配置路径：`config/browser-relay.json`
  - 示例配置：
    ```json
    {
      "enabled": true,
      "port": 8080
    }
    ```

## 使用场景
- 自动化网页操作
- 获取网页内容
- 截图和快照

## 注意事项
- 需要正确配置 Browser Relay
- 某些网站可能有反爬虫机制
- 监控浏览器资源使用