# CrewAI 项目 - 基本信息

## 项目来源
- GitHub: https://github.com/joaomdmoura/crewAI
- 作者: Joao Moura

## 项目简介

CrewAI 是一个 AI Agent 框架，专注于多 Agent 协作。它允许创建多个专业化 Agent（称为 "Crew"），每个 Agent 有特定角色和工具，通过协同工作完成复杂任务。

## 核心特点

1. **多 Agent 协作** - 多个专业化 Agent 协同工作
2. **角色定义** - 每个 Agent 有特定角色和目标
3. **工具集成** - 每个 Agent 配有特定工具
4. **分层执行** - Agent 可以按顺序或并行执行
5. **流程控制** - Agent 之间可以相互触发
6. **本地执行** - 可以在本地运行 Crew

## 技术栈

- **语言**: Python
- **框架**: LangChain 集成
- **AI 集成**: OpenAI, Anthropic, HuggingFace 等
- **许可证**: 未明确（可能是 MIT）

## 核心概念

### Crew（团队）
- 一组协同工作的 Agent
- 每个 Crew 有特定目标和配置

### Agent（成员）
- 每个 Agent 有特定角色
- 配有特定工具和目标
- 可以有背景故事（Backstory）

### Task（任务）
- 要完成的具体任务
- 可以分配给特定 Agent 或 Crew
- 有描述和预期输出

## 快速开始

```bash
pip install crewai
```

```python
from crewai import Agent, Task, Crew

# 创建 Agent
researcher = Agent(
    role="研究员",
    goal="查找最新信息",
    backstory="你是一位经验丰富的研究员..."
)

writer = Agent(
    role="写作者",
    goal="基于研究写文章",
    backstory="你是一位专业写作者..."
)

# 创建任务
research_task = Task(
    description="研究 AI 的最新进展",
    agent=researcher
)

write_task = Task(
    description="基于研究写文章",
    agent=writer
)

# 创建 Crew
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task]
)

# 执行
result = crew.kickoff()
```

## 关键功能

### 1. Agent 定义
- 角色 (Role)
- 目标 (Goal)
- 背景故事 (Backstory)
- 工具 (Tools)
- LLM 配置

### 2. Task 定义
- 描述 (Description)
- 分配的 Agent
- 期望输出
- 依赖关系

### 3. Crew 定义
- Agent 列表
- Task 列表
- 执行顺序
- 流程控制

### 4. 执行模式
- 顺序执行
- 并行执行
- 层级执行
- 自主执行

## 应用场景

1. **内容创作** - 研究 + 写作分工
2. **市场调研** - 多 Agent 协同研究
3. **代码生成** - 代码审查 + 生成分工
4. **客服** - 多专业 Agent 协同服务

## 资源链接

- **GitHub**: https://github.com/joaomdmoura/crewAI
- **文档**: https://docs.crewai.com/ (推测）

## 核心优势

1. **多 Agent 协作** - 专业分工，协同工作
2. **角色定义清晰** - 每个 Agent 有明确角色
3. **灵活的执行模式** - 顺序、并行、层级
4. **LangChain 集成** - 基于 LangChain 生态
5. **Python 友好** - Python 原生框架
