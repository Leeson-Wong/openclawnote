# LangChain 项目 - 操作指南

## 快速开始

### 安装

#### 基础安装
```bash
pip install langchain
```

#### 带特定模型提供商安装
```bash
# OpenAI
pip install langchain "langchain[openai]"

# Anthropic
pip install langchain "langchain[anthropic]"

# Google
pip install langchain "langchain[google]"

# 多个提供商
pip install langchain "langchain[openai,anthropic]"
```

### 环境配置

```bash
# OpenAI
export OPENAI_API_KEY="your-api-key"

# Anthropic
export ANTHROPIC_API_KEY="your-api-key"

# Google
export GOOGLE_API_KEY="your-api-key"
```

## 创建第一个 Agent

### 简单示例（< 10 行代码）

```python
from langchain.agents import create_agent

def get_weather(city: str) -> str:
    """获取指定城市的天气"""
    return f"{city}总是晴天！"

# 创建 Agent
agent = create_agent(
    model="claude-sonnet-4-5-20250929",
    tools=[get_weather],
    system_prompt="你是一个有帮助的助手",
)

# 运行 Agent
result = agent.invoke({
    "messages": [{"role": "user", "content": "旧金山的天气怎么样？"}]
})
print(result)
```

### 使用 OpenAI

```python
from langchain.agents import create_agent
from langchain_openai import ChatOpenAI

# 定义工具
def search(query: str) -> str:
    """搜索信息"""
    return f"关于 '{query}' 的搜索结果"

# 创建 Agent（使用 OpenAI）
agent = create_agent(
    model="gpt-4",
    tools=[search],
    system_prompt="你是一个有帮助的助手",
)

# 运行
result = agent.invoke({"messages": [{"role": "user", "content": "搜索 AI"}]})
```

## 核心功能使用

### 1. 使用 LLM

```python
from langchain_openai import ChatOpenAI

# 创建 LLM 实例
llm = ChatOpenAI(model="gpt-4")

# 调用
response = llm.invoke("你好！")
print(response.content)
```

### 2. 使用 Prompt Template

```python
from langchain.prompts import PromptTemplate

# 创建模板
prompt = PromptTemplate.from_template("给我讲一个关于 {topic} 的笑话")

# 格式化
formatted_prompt = prompt.format(topic="程序员")
print(formatted_prompt)
```

### 3. 使用 Chain

```python
from langchain.chains import LLMChain
from langchain_openai import ChatOpenAI
from langchain.prompts import PromptTemplate

# 创建组件
llm = ChatOpenAI(model="gpt-4")
prompt = PromptTemplate.from_template("解释: {input}")

# 创建 Chain
chain = LLMChain(llm=llm, prompt=prompt)

# 运行
result = chain.run("什么是人工智能？")
print(result)
```

### 4. 使用 Tools

```python
from langchain.tools import tool

# 定义自定义工具
@tool
def calculator(expression: str) -> str:
    """计算数学表达式"""
    try:
        result = eval(expression)
        return f"结果: {result}"
    except:
        return "计算错误"

# 使用工具
print(calculator.invoke("2 + 2"))
```

### 5. 使用 Memory

```python
from langchain.memory import ConversationBufferMemory
from langchain_openai import ChatOpenAI
from langchain.chains import ConversationChain

# 创建记忆
memory = ConversationBufferMemory()

# 创建对话链
llm = ChatOpenAI(model="gpt-4")
conversation = ConversationChain(llm=llm, memory=memory)

# 对话
response1 = conversation.predict(input="我叫小明")
print(response1)

response2 = conversation.predict(input="我叫什么名字？")
print(response2)  # 会记住之前的对话
```

### 6. 使用 Vector Store

```python
from langchain_openai import OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.docstore.document import Document

# 创建文档
documents = [
    Document(page_content="LangChain 是一个 AI 框架"),
    Document(page_content="Python 是一门编程语言"),
]

# 创建嵌入和向量存储
embeddings = OpenAIEmbeddings()
vectorstore = FAISS.from_documents(documents, embeddings)

# 搜索
query = "LangChain 是什么？"
results = vectorstore.similarity_search(query)
for doc in results:
    print(doc.page_content)
```

### 7. 使用 Retriever

```python
from langchain.chains import RetrievalQA
from langchain_openai import ChatOpenAI

# 创建 LLM
llm = ChatOpenAI(model="gpt-4")

# 创建检索 QA 链
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=vectorstore.as_retriever(),
)

# 查询
result = qa_chain.run("什么是 LangChain？")
print(result)
```

## LangGraph 使用（高级 Agent 编排）

### 创建简单的 State Graph

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

# 定义状态
class State(TypedDict):
    messages: list

# 定义节点
def agent_node(state: State):
    """Agent 节点"""
    return {"messages": ["Agent 响应"]}

def tool_node(state: State):
    """工具节点"""
    return {"messages": ["工具执行结果"]}

# 创建图
workflow = StateGraph(State)

# 添加节点
workflow.add_node("agent", agent_node)
workflow.add_node("tools", tool_node)

# 添加边
workflow.set_entry_point("agent")
workflow.add_edge("agent", "tools")
workflow.add_edge("tools", END)

# 编译
app = workflow.compile()

# 运行
result = app.invoke({"messages": []})
print(result)
```

## 开发工作流

### 典型开发流程

```
1. 设计需求
   ↓
2. 创建 Prompt Templates
   ↓
3. 实现/选择 Tools
   ↓
4. 构建 Chains
   ↓
5. 创建 Agent
   ↓
6. 测试和调试
   ↓
7. 部署和监控
```

### 调试和监控

#### 使用 LangSmith

```python
import os

# 配置 LangSmith
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "your-langsmith-key"
os.environ["LANGCHAIN_PROJECT"] = "my-agent"

# 现在所有的运行都会被 LangSmith 跟踪
agent.invoke({"messages": [{"role": "user", "content": "测试"}]})
```

#### 本地调试

```python
from langchain.debug import debug

# 启用调试
debug(True)

# 运行（会打印详细日志）
result = agent.invoke({"messages": [{"role": "user", "content": "测试"}]})

# 禁用调试
debug(False)
```

## 配置管理

### 环境变量配置

```bash
# API Keys
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
export GOOGLE_API_KEY="..."

# LangSmith（可选）
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_API_KEY="lsv2_..."
export LANGCHAIN_PROJECT="my-project"

# 调试
export LANGCHAIN_VERBOSE="true"
```

### 配置文件（.env）

```bash
# .env 文件
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
LANGCHAIN_TRACING_V2=true
```

```python
from dotenv import load_dotenv

# 加载 .env
load_dotenv()
```

## IDE 集成

### VS Code

推荐扩展：
- Python
- Pylance
- Jupyter

### Jupyter Notebook

```bash
# 安装
pip install jupyter

# 启动
jupyter notebook
```

## 常见问题排查

### 1. API 密钥错误
```python
# 检查环境变量
import os
print(os.getenv("OPENAI_API_KEY"))
```

### 2. 导入错误
```bash
# 确保安装了正确的包
pip install langchain "langchain[openai]"
```

### 3. Token 限制
```python
from langchain_openai import ChatOpenAI

# 设置 max_tokens
llm = ChatOpenAI(
    model="gpt-4",
    max_tokens=1000,
    temperature=0.7
)
```

### 4. 内存问题
```python
from langchain.memory import ConversationBufferWindowMemory

# 限制对话历史长度
memory = ConversationBufferWindowMemory(k=5)
```

## 测试

### 单元测试

```python
import pytest
from langchain_openai import ChatOpenAI

def test_llm():
    llm = ChatOpenAI(model="gpt-4")
    response = llm.invoke("测试")
    assert len(response.content) > 0
```

### 集成测试

```python
def test_agent():
    agent = create_agent(
        model="gpt-4",
        tools=[get_weather],
        system_prompt="测试 Agent",
    )
    result = agent.invoke({"messages": [{"role": "user", "content": "测试"}]})
    assert result is not None
```

## 性能优化

### 1. 缓存
```python
from langchain.cache import InMemoryCache
from langchain.globals import set_llm_cache

# 启用缓存
set_llm_cache(InMemoryCache())

# 第二次调用会使用缓存
llm.invoke("测试")
llm.invoke("测试")  # 使用缓存
```

### 2. 异步执行
```python
import asyncio
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4")

async def async_invoke():
    result = await llm.ainvoke("异步测试")
    print(result.content)

# 运行
asyncio.run(async_invoke())
```

### 3. 批处理
```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4")

# 批量调用
results = llm.batch(["测试1", "测试2", "测试3"])
for result in results:
    print(result.content)
```

## 社区资源

### 官方资源
- **文档**: https://docs.langchain.com
- **API 参考**: https://reference.langchain.com
- **Chat LangChain**: https://chat.langchain.com
- **论坛**: https://forum.langchain.com
- **Twitter/X**: https://x.com/langchain

### 学习资源
- **LangChain Academy**: https://academy.langchain.com (免费课程)
- **GitHub Examples**: https://github.com/langchain-ai/langchain/tree/master/examples

### 贡献
- **贡献指南**: https://docs.langchain.com/oss/python/contributing/overview
- **Issues**: https://github.com/langchain-ai/langchain/issues
- **Discussions**: https://github.com/langchain-ai/langchain/discussions

## 最佳实践

### 1. Prompt 设计
- 清晰明确的指令
- 提供示例（Few-shot）
- 使用结构化输出

### 2. 工具设计
- 工具描述要详细（帮助 Agent 选择）
- 参数要有类型提示
- 错误处理要完善

### 3. Memory 管理
- 选择合适的 Memory 类型
- 限制历史长度
- 定期清理

### 4. 监控
- 使用 LangSmith 跟踪
- 记录性能指标
- 定期评估

### 5. 安全
- 保护 API 密钥
- 输入验证
- 输出过滤
- 内容安全检查

## 快速参考

### 常用导入
```python
from langchain.agents import create_agent
from langchain_openai import ChatOpenAI
from langchain.prompts import PromptTemplate
from langchain.chains import LLMChain
from langchain.memory import ConversationBufferMemory
from langchain.tools import tool
```

### 基本流程
```python
# 1. 定义工具
def my_tool(input: str) -> str:
    return f"工具输出: {input}"

# 2. 创建 Agent
agent = create_agent(
    model="gpt-4",
    tools=[my_tool],
    system_prompt="你是一个有帮助的助手",
)

# 3. 运行 Agent
result = agent.invoke({"messages": [{"role": "user", "content": "你好"}]})
```

---

## 分析日期
2026-02-03
