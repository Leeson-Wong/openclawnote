# AutoGPT 项目 - 操作指南

## 快速开始

### AutoGPT Platform

#### 快速一键安装（推荐）

##### macOS/Linux
```bash
curl -fsSL https://setup.agpt.co/install.sh -o install.sh && bash install.sh
```

##### Windows (PowerShell)
```bash
powershell -c "iwr https://setup.agpt.co/install.bat -o install.bat; ./install.bat"
```

这将自动：
- 安装依赖
- 配置 Docker
- 启动本地实例

#### 手动安装

##### 1. 系统要求
- CPU: 4+ 核心（推荐）
- RAM: 最少 8GB，推荐 16GB
- 存储: 至少 10GB 可用空间

##### 2. 软件要求
- Docker Engine (20.10.0+)
- Docker Compose (2.0.0+)
- Git (2.30+)
- Node.js (16.x+)
- npm (8.x+)
- VSCode (1.60+) 或现代代码编辑器

##### 3. 克隆仓库
```bash
git clone https://github.com/Significant-Gravitas/AutoGPT.git
cd AutoGPT
```

##### 4. 配置环境
```bash
# 复制环境变量文件
cp .env.example .env

# 编辑 .env 文件，配置必要的 API 密钥
nano .env
```

##### 5. 启动服务
```bash
# 使用 Docker Compose
docker-compose up -d

# 或使用脚本（如果有）
./start.sh
```

##### 6. 访问平台
打开浏览器访问: `http://localhost:3000`

### AutoGPT Classic

#### 快速开始

##### 1. 克隆仓库
```bash
git clone https://github.com/Significant-Gravitas/AutoGPT.git
cd AutoGPT
```

##### 2. 安装依赖
```bash
./run setup
```

##### 3. 运行 Agent
```bash
# 列出可用的 Agent
./run agent list

# 启动 Agent
./run agent start <agent-name>
```

## 核心功能使用

### AutoGPT Platform

#### 1. 使用 Agent Builder 创建 Agent

##### 步骤
1. 登录平台
2. 导航到 "Agent Builder"
3. 选择 Agent 模板（或从零开始）
4. 配置 Agent：
   - 名称和描述
   - 系统提示
   - 模型选择
   - 工具配置
5. 保存并测试
6. 部署到生产

##### 配置示例
```json
{
  "name": "内容创作 Agent",
  "description": "自动生成社交媒体内容",
  "system_prompt": "你是一个社交媒体内容创作专家",
  "model": "gpt-4",
  "tools": ["web_search", "text_generation"],
  "settings": {
    "max_tokens": 2000,
    "temperature": 0.7
  }
}
```

#### 2. 使用 Workflow Editor 构建工作流

##### 步骤
1. 导航到 "Workflows"
2. 创建新工作流
3. 从块库拖拽块到画布
4. 连接块定义数据流
5. 配置每个块的参数
6. 测试工作流
7. 保存并部署

##### 常用块
- **触发器**: 定时器、Webhook、手动触发
- **输入**: 文本输入、文件上传、API 请求
- **处理**: LLM 调用、数据处理、条件分支
- **输出**: 邮件发送、Webhook、文件保存

##### 工作流示例
```
[定时器] → [获取话题] → [LLM 生成] → [发布到社交媒体]
```

#### 3. 部署 Agent

##### 步骤
1. 在 Dashboard 选择 Agent
2. 点击 "Deploy"
3. 选择部署环境（测试/生产）
4. 配置部署参数
5. 点击 "Confirm Deploy"
6. 等待部署完成
7. 监控运行状态

#### 4. 与 Agent 交互

##### 通过 Dashboard
1. 导航到已部署的 Agent
2. 点击 "Run" 启动交互
3. 在聊天界面输入任务
4. 查看实时执行过程
5. 获取结果

##### 通过 API
```bash
curl -X POST http://localhost:8000/api/agents/<agent-id>/run \
  -H "Content-Type: application/json" \
  -d '{
    "task": "生成一篇关于 AI 的博客文章",
    "parameters": {
      "length": 1000
    }
  }'
```

#### 5. 监控 Agent

##### Dashboard 监控
- 查看运行状态
- 查看性能指标
- 查看执行日志
- 查看错误和警告

##### 性能指标
- 成功率
- 平均响应时间
- 资源使用（CPU、内存）
- API 调用次数
- 错误率

### AutoGPT Classic

#### 1. 使用 Forge 构建 Agent

##### 快速开始
```bash
# 导航到 Forge 目录
cd classic/forge

# 遵循教程
cat tutorials/001_getting_started.md
```

##### 创建自定义 Agent
```python
from forge import BaseAgent, Tool

class MyAgent(BaseAgent):
    def __init__(self):
        super().__init__()
        self.tools = [
            Tool(name="search", function=self.search)
        ]

    def search(self, query: str) -> str:
        # 实现搜索功能
        return f"搜索结果: {query}"

    def plan(self, task: str):
        # 实现规划逻辑
        return plan

    def execute(self, step):
        # 实现执行逻辑
        return result
```

#### 2. 使用 Benchmark 测试 Agent

##### 运行基准测试
```bash
# 列出所有测试
./run benchmark list

# 运行特定类别
./run benchmark start --category reasoning

# 运行所有测试
./run benchmark start --all

# 生成报告
./run benchmark report --format html
```

##### 查看结果
- 查看生成的 HTML 报告
- 分析得分和排名
- 识别改进领域

#### 3. 使用 CLI

##### Agent 命令
```bash
# 创建新 Agent
./run agent create --name my-agent --template default

# 列出所有 Agent
./run agent list

# 启动 Agent
./run agent start my-agent

# 停止 Agent
./run agent stop my-agent

# 删除 Agent
./run agent delete my-agent
```

##### Benchmark 命令
```bash
# 列出测试类别
./run benchmark categories

# 列出所有测试
./run benchmark list

# 启动基准测试
./run benchmark start

# 查看帮助
./run benchmark --help
```

##### Setup 命令
```bash
# 安装所有依赖
./run setup

# 检查系统要求
./run setup --check

# 安装特定依赖
./run setup --install docker
```

## 配置管理

### AutoGPT Platform

### 环境变量

```bash
# .env 文件示例

# LLM API
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# 数据库
DATABASE_URL=postgres://user:password@localhost:5432/autogpt

# 服务端口
FRONTEND_PORT=3000
BACKEND_PORT=8000

# 日志
LOG_LEVEL=INFO
LOG_FILE=./logs/autogpt.log

# 监控
ENABLE_METRICS=true
METRICS_PORT=9090
```

### AutoGPT Classic

### 配置文件

```yaml
# config.yaml

agent:
  name: MyAgent
  model: gpt-4
  temperature: 0.7
  max_tokens: 2000

tools:
  - name: search
    enabled: true
  - name: web_browse
    enabled: true

memory:
  type: vector
  max_entries: 1000

logging:
  level: INFO
  file: ./logs/agent.log
```

## 常见问题排查

### 1. Docker 相关问题

#### Docker 未运行
```bash
# 启动 Docker
sudo systemctl start docker

# 或 macOS
open -a Docker
```

#### 端口冲突
```bash
# 查看占用端口的进程
lsof -i :3000

# 修改 .env 文件中的端口
FRONTEND_PORT=3001
```

### 2. API 密钥问题

#### 无效的 API 密钥
```bash
# 检查环境变量
echo $OPENAI_API_KEY

# 重新配置
nano .env
```

### 3. 内存不足

#### 增加分配给 Docker 的内存
- 打开 Docker Desktop
- Settings → Resources
- 增加 Memory 到至少 8GB

### 4. Agent 执行失败

#### 查看日志
```bash
# AutoGPT Platform
docker-compose logs backend

# AutoGPT Classic
cat logs/agent.log
```

#### 重启 Agent
```bash
# Platform
# 通过 Dashboard 停止并重启 Agent

# Classic
./run agent stop my-agent
./run agent start my-agent
```

## 开发工作流

### AutoGPT Platform

### 典型开发流程

```
1. 设计 Agent
   ↓
2. 使用 Agent Builder 配置
   ↓
3. 构建 Workflow（如需要）
   ↓
4. 测试（测试环境）
   ↓
5. 调试（查看日志）
   ↓
6. 部署到生产
   ↓
7. 监控性能
```

### AutoGPT Classic

### 典型开发流程

```
1. 克隆仓库
   ↓
2. 使用 Forge 创建 Agent
   ↓
3. 实现自定义逻辑
   ↓
4. 使用 Benchmark 测试
   ↓
5. 通过 UI 测试
   ↓
6. 部署到生产
```

## 测试

### 单元测试

```bash
# AutoGPT Platform
cd backend
pytest tests/

# AutoGPT Classic
cd classic/forge
pytest tests/
```

### 集成测试

```bash
# 测试 API
cd backend
pytest tests/integration/

# 测试 Agent
./run agent start test-agent
./run benchmark start --agent test-agent
```

## 社区资源

### 官方资源
- **文档**: https://docs.agpt.co
- **平台文档**: https://docs.agpt.co/platform/getting-started/
- **博客**: https://agpt.co/blog
- **Discord**: https://discord.gg/autogpt
- **Twitter**: https://twitter.com/Auto_GPT

### 学习资源
- **Forge 教程**: https://medium.com/@aiedge/autogpt-forge-e3de53cc58ec
- **Agent Protocol**: https://agentprotocol.ai/
- **多语言文档**: 多种语言版本

### 贡献
- **贡献指南**: https://github.com/Significant-Gravitas/AutoGPT/blob/master/CONTRIBUTING.md
- **Issues**: https://github.com/Significant-Gravitas/AutoGPT/issues
- **Discussions**: https://github.com/Significant-Gravitas/AutoGPT/discussions

## 最佳实践

### 1. Agent 设计
- 清晰的目标和范围
- 合理的系统提示
- 适当的工具选择
- 测试和迭代

### 2. 工作流设计
- 模块化设计
- 错误处理
- 日志记录
- 性能优化

### 3. 部署
- 先在测试环境验证
- 监控性能指标
- 设置告警
- 版本控制

### 4. 安全
- 保护 API 密钥
- 验证输入
- 限制权限
- 定期更新

## 快速参考

### 常用命令

#### AutoGPT Platform
```bash
# 启动所有服务
docker-compose up -d

# 停止所有服务
docker-compose down

# 查看日志
docker-compose logs -f

# 重启服务
docker-compose restart
```

#### AutoGPT Classic
```bash
# 安装依赖
./run setup

# 启动 Agent
./run agent start <name>

# 停止 Agent
./run agent stop <name>

# 运行基准测试
./run benchmark start
```

---

## 分析日期
2026-02-03
