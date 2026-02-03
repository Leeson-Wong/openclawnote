# CrewAI 项目 - 操作指南

## 快速开始

### 安装

```bash
pip install crewai
```

### 创建第一个 Crew

```python
from crewai import Agent, Task, Crew

# 创建 Agent
researcher = Agent(
    role="高级研究员",
    goal="发现和整合最新信息",
    backstory="你是一位在 AI 研究领域经验丰富的研究员..."
)

writer = Agent(
    role="专业写作者",
    goal="基于研究写文章",
    backstory="你是一位技术文章写作者..."
)

# 创建任务
research_task = Task(
    description="研究 AI 的最新进展",
    agent=researcher
)

write_task = Task(
    description="基于研究写一篇技术文章",
    agent=writer
)

# 创建 Crew
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task]
)

# 执行
result = crew.kickoff()
print(result)
```

## 核心功能使用

### 1. 定义 Agent

#### 基本定义
```python
from crewai import Agent

agent = Agent(
    role="研究员",
    goal="查找最新信息",
    backstory="你是一位经验丰富的研究员..."
)
```

#### 完整定义
```python
agent = Agent(
    role="高级研究员",
    goal="发现和整合最新信息",
    backstory="你是一位在 AI 研究领域经验丰富的研究员，擅长分析和整合复杂信息...",
    tools=[search_tool, scrape_tool],
    llm="gpt-4",
    verbose=True,
    allow_delegation=False,
    max_iter=5
)
```

### 2. 定义 Task

#### 基本任务
```python
from crewai import Task

task = Task(
    description="研究 AI 的最新进展",
    agent=researcher
)
```

#### 完整任务
```python
task = Task(
    description="研究 AI 的最新进展并整合为报告",
    expected_output="一份详细的研究报告",
    agent=researcher,
    async=False,
    context={
        "domain": "AI",
        "timeframe": "最近3个月"
    }
)
```

### 3. 创建 Crew

#### 基本 Crew
```python
from crewai import Crew

crew = Crew(
    agents=[researcher, writer],
    tasks=[research, write]
)
```

#### 完整 Crew
```python
crew = Crew(
    agents=[researcher, writer, editor],
    tasks=[research, write, edit],
    process=Process.hierarchical,
    verbose=True,
    memory=True,
    max_rpm=100
)
```

### 4. 执行 Crew

#### 顺序执行
```python
result = crew.kickoff()
```

#### 使用特定 Process
```python
from crewai import Process

# 顺序执行
crew = Crew(
    agents=[...],
    tasks=[...],
    process=Process.sequential
)

# 层级执行
crew = Crew(
    agents=[manager, worker1, worker2],
    tasks=[...],
    process=Process.hierarchical
)

# 并行执行
crew = Crew(
    agents=[...],
    tasks=[...],
    process=Process.parallel
)

result = crew.kickoff()
```

### 5. 使用 Tools

#### LangChain 工具集成
```python
from langchain.tools import DuckDuckGoSearchRun
from crewai import Agent

agent = Agent(
    role="搜索专家",
    goal="使用搜索工具查找信息",
    backstory="你是一位搜索专家...",
    tools=[DuckDuckGoSearchRun()],
    llm="gpt-4"
)
```

#### 自定义工具
```python
from crewai import tool

@tool
def search_database(query: str) -> str:
    """搜索数据库"""
    # 实现搜索逻辑
    return f"搜索结果: {query}"

agent = Agent(
    role="数据库专家",
    goal="查询数据库信息",
    tools=[search_database],
    llm="gpt-4"
)
```

## 典型工作流

### 内容创作 Crew

```python
# 1. Researcher Agent
researcher = Agent(
    role="研究员",
    goal="收集和研究信息",
    tools=[search_tool],
    llm="gpt-4"
)

# 2. Writer Agent
writer = Agent(
    role="写作者",
    goal="基于研究写内容",
    llm="gpt-4"
)

# 3. Editor Agent
editor = Agent(
    role="编辑",
    goal="编辑和优化内容",
    llm="gpt-4"
)

# 4. Tasks
research_task = Task(
    description="研究最新的 AI 进展",
    agent=researcher
)

write_task = Task(
    description="基于研究写文章",
    agent=writer,
    context={"research_task": research_task}
)

edit_task = Task(
    description="编辑和优化文章",
    agent=editor,
    context={"write_task": write_task}
)

# 5. Crew
crew = Crew(
    agents=[researcher, writer, editor],
    tasks=[research_task, write_task, edit_task],
    process=Process.sequential
)

# 6. Execute
result = crew.kickoff()
```

### 市场调研 Crew

```python
# Manager Agent
manager = Agent(
    role="项目管理者",
    goal="协调整个研究任务",
    llm="gpt-4"
)

# Specialist Agents
analyst = Agent(
    role="市场分析师",
    goal="分析市场趋势",
    tools=[market_tool],
    llm="gpt-4"
)

researcher = Agent(
    role="研究员",
    goal="研究竞争对手",
    tools=[search_tool],
    llm="gpt-4"
)

writer = Agent(
    role="报告写作者",
    goal="撰写市场分析报告",
    llm="gpt-4"
)

# Tasks
analyze_task = Task(
    description="分析市场趋势",
    agent=analyst
)

research_task = Task(
    description="研究主要竞争对手",
    agent=researcher
)

write_task = Task(
    description="撰写综合市场报告",
    agent=writer
)

# Crew with Hierarchical Process
crew = Crew(
    agents=[manager, analyst, researcher, writer],
    tasks=[analyze_task, research_task, write_task],
    process=Process.hierarchical
)

result = crew.kickoff()
```

## 配置管理

### LLM 配置

```python
from langchain_openai import ChatOpenAI
from crewai import Agent

llm = ChatOpenAI(model="gpt-4", temperature=0.7)

agent = Agent(
    role="研究员",
    goal="查找信息",
    llm=llm
)
```

### Process 类型选择

```python
from crewai import Process

# Sequential: Agent 依次执行
crew = Crew(
    agents=[...],
    tasks=[...],
    process=Process.sequential
)

# Hierarchical: Manager 协调其他 Agent
crew = Crew(
    agents=[manager, worker1, worker2],
    tasks=[...],
    process=Process.hierarchical
)

# Parallel: Agent 同时执行
crew = Crew(
    agents=[...],
    tasks=[...],
    process=Process.parallel
)
```

## 常见问题排查

### 1. Agent 不执行任务

#### 检查
- Task 是否正确分配给 Agent
- Agent 工具是否正确配置
- Process 类型是否合适

#### 解决方案
```python
# 确保 Task 分配给正确的 Agent
task = Task(
    description="任务描述",
    agent=correct_agent  # ← 确保正确
)
```

### 2. 并行执行问题

#### 问题
Agent 间没有正确的协调

#### 解决方案
使用 Hierarchical Process
```python
crew = Crew(
    agents=[manager, worker1, worker2],
    tasks=[...],
    process=Process.hierarchical  # ← Manager 协调
)
```

### 3. 记忆不工作

#### 问题
Agent 没有共享信息

#### 解决方案
```python
crew = Crew(
    agents=[...],
    tasks=[...],
    memory=True  # ← 启用记忆
)
```

## 最佳实践

### 1. Agent 设计
- 明确的角色定义
- 相关的背景故事
- 合适的工具集
- 清晰的目标

### 2. Task 设计
- 清晰的任务描述
- 明确的期望输出
- 适当的 Agent 分配
- 相关的上下文

### 3. Crew 设计
- 合适的 Process 类型
- 合理的 Agent 数量（3-5 个最佳）
- 清晰的任务流程
- 启用记忆（如需要）

### 4. Tool 集成
- 使用 LangChain 工具
- 创建自定义工具
- 工具与角色匹配
- 完善的错误处理

## 快速参考

### 基本流程

```python
from crewai import Agent, Task, Crew

# 1. 创建 Agent
agent = Agent(role=..., goal=..., tools=...)

# 2. 创建 Task
task = Task(description=..., agent=agent)

# 3. 创建 Crew
crew = Crew(agents=[agent], tasks=[task])

# 4. 执行
result = crew.kickoff()
```

---

## 分析日期
2026-02-03
