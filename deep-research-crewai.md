# 深度研究 - CrewAI（10 分钟）

## 🎯 研究目标

深入研究 CrewAI 的核心机制：
1. **Role-based 系统** - 如何定义和管理 Agent 角色
2. **Manager 模式** - 如何协调多个 Agent
3. **协作流程** - Agent 之间如何协作
4. **工具集成** - 如何定义和使用工具
5. **任务分配** - 如何分解和分配任务

---

## 📋 CrewAI 核心概念

### 1. Role-based Agent 定义
CrewAI 使用 Role + Goal + Backstory 的角色系统：
- **Role**: Agent 的角色名称
- **Goal**: Agent 的目标
- **Backstory**: Agent 的背景故事
- **Tools**: Agent 可以使用的工具
- **Verbose**: 详细输出模式

### 2. Manager 模式
一个专门的 Manager Agent 来协调其他 Agent：
- 接收用户输入
- 分解任务
- 分配给合适的 Agent
- 整合结果
- 返回最终答案

### 3. 任务驱动协作
通过 Task 来驱动 Agent 协作：
- 每个 Agent 执行特定的任务
- 任务之间有依赖关系
- 可以并行或串行执行

### 4. 工具集成
与 LangChain 的工具系统完美集成：
- 使用 `@tool` 装饰器
- 自动注册到 Agent
- 支持自定义工具

---

## 🧐 核心机制分析

### Role-based Agent 定义
```python
from crewai import Agent, Task, Crew
from langchain.tools import tool

# Agent 定义
researcher = Agent(
    role='Senior Research Analyst',
    goal='Uncover cutting edge developments in AI and Data Science',
    backstory="""You work at a leading tech think tank. Your expertise 
    lies in identifying emerging trends. You have a knack for dissecting 
    complex data and presenting actionable insights.""",
    verbose=True,
    allow_delegation=False,
    tools=[search_tool, analyze_tool]
)

# Task 定义
task1 = Task(
    description='Research latest AI trends',
    agent=researcher,
    expected_output='A comprehensive report on AI trends'
)

# Crew 定义
crew = Crew(
    agents=[researcher, writer],
    tasks=[task1, task2],
    verbose=2,
    process=Process.sequential
)
```

### Manager 模式
```python
# Manager Agent
manager = Agent(
    role='Project Manager',
    goal='Coordinate research and writing tasks',
    backstory="""You're a project manager with expertise in coordinating 
    complex research projects.""",
    allow_delegation=True,
    verbose=True
)

# 使用 Manager 模式
crew = Crew(
    agents=[manager, researcher, writer],
    tasks=[task1, task2, task3],
    manager_agent=manager,
    process=Process.hierarchical
)
```

### 协作流程
```python
# 顺序执行
sequential_crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task],
    process=Process.sequential,
    verbose=True
)

# 并行执行
parallel_crew = Crew(
    agents=[researcher1, researcher2],
    tasks=[research_task1, research_task2],
    process=Process.parallel,
    verbose=True
)
```

---

## 🎯 与我们设计的关联

### 1. Agent 角色定义
```python
# 我们的 Agent 角色定义（基于 CrewAI 模式）
class GatewayAgent:
    role = "Gateway Agent"
    goal = "处理所有用户输入，理解意图，路由到合适的 agent"
    backstory = """你是一个专业的通信网关，负责接收所有用户请求，
    理解用户意图，然后路由到正确的 agent。你对各种协议和消息格式
    都有深入的理解。"""
    tools = [router_tool, parser_tool]
    
class CoordinatorAgent:
    role = "Coordinator Agent"
    goal = "协调所有 agent，管理任务分配和进度跟踪"
    backstory = """你是一个专业的项目经理，擅长分解复杂任务，
    分配给合适的 agent，并跟踪进度确保高质量交付。"""
    tools = [task_decompose_tool, progress_tracker_tool]
```

### 2. 多 Agent 协作
```python
# 使用 CrewAI 的 Manager 模式来实现我们的 Coordinator Agent
manager = CoordinatorAgent()
gateway = GatewayAgent()
research = ResearchAgent()
analysis = AnalysisAgent()
decision = DecisionAgent()

# 创建 Crew
agent_crew = Crew(
    agents=[gateway, manager, research, analysis, decision],
    tasks=[],
    manager_agent=manager,
    process=Process.hierarchical,
    verbose=True
)
```

---

## 📊 关键发现

### 1. Role-based 系统的优势
- **清晰性**: 每个 Agent 有明确的角色和职责
- **专业性**: Agent 可以专注于特定领域
- **可组合**: 可以组合不同的角色形成团队
- **可扩展**: 易于添加新的角色和技能

### 2. Manager 模式的优势
- **协调性**: 统一的任务分解和分配
- **可控性**: Manager 控制执行流程
- **可观测**: 完整的执行轨迹和日志
- **灵活性**: 可以动态调整任务分配

### 3. Task-driven 的优势
- **灵活性**: 任务可以动态定义
- **可组合**: 任务可以串行或并行执行
- **可追踪**: 每个任务都有明确的输入和输出
- **可测试**: 每个任务都可以单独测试

---

## 🚀 应用到我们的设计

### 1. 使用 Role-based 系统
我们可以完全采用 CrewAI 的 Role + Goal + Backstory 模式：
- 每个都有明确的角色定义
- 专业的目标和背景故事
- 合适的工具集

### 2. 使用 Manager 模式
我们的 Coordinator Agent 可以像 CrewAI 的 Manager Agent：
- 接收用户输入
- 分解任务
- 分配给合适的 Agent
- 整合结果

### 3. 使用 Task-driven 协作
我们可以使用 CrewAI 的 Task 机制：
- 明确的任务定义
- 依赖关系管理
- 并行/串行执行

---

## 📝 CrewAI vs 其他框架对比

| 特性 | CrewAI | AutoGen | LangChain | LlamaIndex |
|-----|---------|---------|-----------|------------|
| **角色系统** | ✅ Role+Goal+Backstory | ❌ 需要手动定义 | ❌ 灵活但不标准 | ❌ 不适用 |
| **Manager 模式** | ✅ 专门的 Manager Agent | ❌ GroupChat | ❌ 无专门协调 | ❌ 不适用 |
| **Task 机制** | ✅ 明确的任务定义 | ❌ 对话驱动 | ❌ 链式调用 | ❌ 查询驱动 |
| **工具集成** | ✅ LangChain 兼容 | ✅ 自定义 | ✅ 原生支持 | ✅ LangChain 兼容 |
| **可观测性** | ✅ 详细日志 | ✅ 对话历史 | ✅ LangSmith | ✅ 查询日志 |

---

## 💡 关键洞察

### 1. 标准化的角色系统
CrewAI 的 Role + Goal + Backstory 模式是所有框架中最标准化的，我们可以直接采用。

### 2. Manager 模式的独特性
只有 CrewAI 提供了专门的 Manager 模式，这正是我们需要的 Coordinator Agent。

### 3. Task-driven 的简洁性
相比对话驱动的复杂性，Task-driven 模式更清晰可控。

### 4. 生态系统的兼容性
CrewAI 与 LangChain 完美兼容，可以重用 LangChain 的工具和组件。

---

## 📂 CrewAI 源码位置

- **Agent 实现**: `crewai/agent.py`
- **Crew 实现**: `crewai/crew.py`
- **Task 实现**: `crewai/task.py`
- **Manager 实现**: `crewai/manager.py`
- **Process 实现**: `crewai/process.py`

---

## 🎯 CrewAI 在我们设计中的具体应用

### 1. Agent 角色定义模板
```python
class BaseAgent:
    def __init__(self, role: str, goal: str, backstory: str, tools: list):
        self.role = role
        self.goal = goal
        self.backstory = backstory
        self.tools = tools
        self.verbose = True
```

### 2. 任务分解模板
```python
def create_task(name: str, description: str, agent: BaseAgent, expected_output: str):
    return Task(
        name=name,
        description=description,
        agent=agent,
        expected_output=expected_output
    )
```

### 3. 协调器实现模板
```python
class CoordinatorManager:
    def __init__(self, agents: list):
        self.agents = agents
        self.manager = self.create_manager()
    
    def create_manager(self):
        return Agent(
            role="Coordinator Agent",
            goal="协调所有 agent 的协作",
            backstory="专业的项目协调器",
            tools=[task_decompose_tool, progress_tracker_tool],
            allow_delegation=True
        )
    
    def execute_tasks(self, tasks: list):
        crew = Crew(
            agents=[self.manager] + self.agents,
            tasks=tasks,
            manager_agent=self.manager,
            process=Process.hierarchical
        )
        return crew.kickoff()
```

---

## 🔄 研究结论

### 为什么 CrewAI 是关键框架
1. **Role-based 系统** 是最标准化的，适合我们的 15 个 Agent
2. **Manager 模式** 直接解决了协调问题
3. **Task-driven** 模式比对话驱动更可控
4. **LangChain 兼容** 可以重用工具和组件

### 可直接借鉴的设计模式
1. **Role + Goal + Backstory** 标准化 Agent 定义
2. **Manager 模式** 统一任务协调
3. **Task 机制** 清晰的任务定义和执行
4. **分层协作** Agent 可以按层次组织

---

**研究状态**: ✅ CrewAI 深度研究完成（10 分钟）
**研究时间**: 2026-02-03 13:20-13:30
**下一研究**: Semantic Kernel（10 分钟）
