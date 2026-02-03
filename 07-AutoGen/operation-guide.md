# AutoGen 项目 - 操作指南

## 快速开始

### 安装

```bash
pip install pyautogen
```

### 基本使用

```python
from autogen import AssistantAgent, UserProxyAgent

# 配置 LLM
llm_config = {
    "config_list": [{"model": "gpt-4", "api_key": "your-api-key"}],
    "temperature": 0.7
}

# 创建 Assistant Agent
assistant = AssistantAgent(
    "assistant",
    llm_config=llm_config
)

# 创建 User Proxy Agent
user_proxy = UserProxyAgent(
    "user_proxy",
    code_execution_config={"work_dir": "coding"},
    human_input_mode="NEVER"
)

# 开始对话
user_proxy.initiate_chat(
    assistant,
    message="写一个 Python 函数来计算斐波那契数列"
)
```

## 核心功能使用

### 1. 创建不同类型的 Agent

#### AssistantAgent
```python
assistant = AssistantAgent(
    name="assistant",
    llm_config=llm_config,
    system_message="你是一个专业的 Python 程序员。",
    description="专业的代码编写助手"
)
```

#### UserProxyAgent
```python
user_proxy = UserProxyAgent(
    name="user_proxy",
    human_input_mode="TERMINATE",  # 终止时请求人类输入
    code_execution_config={
        "work_dir": "coding",
        "use_docker": True
    },
    max_consecutive_auto_reply=3
)
```

#### 自定义 Agent
```python
from autogen import ConversableAgent

class CustomAgent(ConversableAgent):
    def __init__(self, name, **kwargs):
        super().__init__(name, **kwargs)
        self.custom_capability = True

custom_agent = CustomAgent(
    name="custom",
    llm_config=llm_config
)
```

### 2. 群聊设置

#### 创建群聊
```python
from autogen import GroupChat, GroupChatManager

# 创建多个 Agent
coder = AssistantAgent("coder", llm_config=llm_config)
reviewer = AssistantAgent("reviewer", llm_config=llm_config)
tester = AssistantAgent("tester", llm_config=llm_config)

# 创建群聊
groupchat = GroupChat(
    agents=[coder, reviewer, tester],
    messages=[],
    max_round=12,
    speaker_selection_method="auto"
)

# 创建群聊管理器
manager = GroupChatManager(
    groupchat=groupchat,
    llm_config=llm_config
)

# 开始群聊对话
user_proxy.initiate_chat(
    manager,
    message="开发一个网络爬虫项目，需要编码、审查和测试"
)
```

### 3. 工具使用

#### 函数调用
```python
from autogen import register_function

def get_weather(location: str) -> str:
    """获取指定位置的天气信息"""
    # 实现天气获取逻辑
    return f"{location} 的天气：晴朗，25°C"

# 注册函数
assistant.register_for_execution()(get_weather)

# 使用工具
user_proxy.initiate_chat(
    assistant,
    message="查询北京的天气情况"
)
```

#### 代码执行
```python
# 配置代码执行
user_proxy = UserProxyAgent(
    name="user_proxy",
    code_execution_config={
        "work_dir": "coding",
        "use_docker": True,
        "timeout": 60,
        "last_n_messages": 1
    }
)

# Agent 可以执行代码
user_proxy.initiate_chat(
    assistant,
    message="写一个 Python 脚本来分析 CSV 数据并生成图表"
)
```

### 4. 对话控制

#### 发言顺序控制
```python
# 自定义发言顺序
def custom_speaker_selection(last_speaker, groupchat):
    agents = groupchat.agents
    if last_speaker == agents[0]:
        return agents[1]
    else:
        return agents[0]

groupchat.speaker_selection_method = custom_speaker_selection
```

#### 终止条件
```python
# 自定义终止条件
def termination_check(sender, receiver, groupchat):
    if "任务完成" in receiver.last_message().get("content", ""):
        return True, "任务已成功完成"
    return False, None

groupchat.register_reply(
    assistant,
    reply_func=termination_check,
    trigger=lambda sender, receiver: True
)
```

### 5. 人机协作

#### 人类反馈
```python
# 配置人类输入模式
user_proxy = UserProxyAgent(
    name="user_proxy",
    human_input_mode="ALWAYS",  # 总是请求人类输入
    max_consecutive_auto_reply=2
)

# 在对话中人类可以提供反馈
user_proxy.initiate_chat(assistant, message="开始任务")
# 人类可以介入提供反馈或指导
```

#### 审批流程
```python
# 设置审批
def approval_check(sender, receiver, message):
    if "部署" in message:
        # 需要人工审批
        return input(f"批准部署操作？(y/n): ").lower() == "y"
    return True

# 注册审批函数
user_proxy.register_reply(
    assistant,
    reply_func=approval_check,
    trigger=lambda sender, receiver: "部署" in receiver.last_message().get("content", "")
)
```

## 高级配置

### 1. LLM 配置

#### 多模型配置
```python
llm_config = {
    "config_list": [
        {
            "model": "gpt-4",
            "api_key": "openai-key",
            "base_url": "https://api.openai.com/v1"
        },
        {
            "model": "gpt-3.5-turbo",
            "api_key": "openai-key"
        }
    ],
    "temperature": 0.7,
    "max_tokens": 2000,
    "timeout": 30
}
```

#### Azure OpenAI 配置
```python
llm_config = {
    "config_list": [
        {
            "model": "gpt-4",
            "api_key": "azure-key",
            "base_url": "https://your-resource.openai.azure.com",
            "api_type": "azure",
            "api_version": "2024-02-15-preview"
        }
    ]
}
```

### 2. 系统消息定制

```python
# 详细的系统消息
system_message = """
你是一个专业的软件开发团队，包含以下角色：
1. 架构师：负责系统设计
2. 开发者：负责代码实现
3. 测试员：负责质量保证
4. 产品经理：负责需求分析

请按照敏捷开发流程协作完成任务。
"""

architect = AssistantAgent(
    name="architect",
    system_message=system_message + "\n你是架构师...",
    llm_config=llm_config
)
```

## 常见问题排查

### 1. Agent 不响应

#### 检查 LLM 配置
```python
# 验证 LLM 配置
import autogen
response = autogen.oai.ChatCompletion.create(
    llm_config=llm_config,
    messages=[{"role": "user", "content": "测试消息"}]
)
print(response)
```

### 2. 代码执行失败

#### 检查 Docker 配置
```python
# 确保 Docker 可用
import docker
client = docker.from_env()
try:
    client.ping()
    print("Docker 可用")
except Exception as e:
    print(f"Docker 不可用: {e}")
```

### 3. 群聊发言问题

#### 检查发言顺序
```python
# 手动指定发言顺序
groupchat.speaker_selection_method = "manual"

# 或者使用轮询
groupchat.speaker_selection_method = "round_robin"
```

## 最佳实践

### 1. Agent 设计原则
- 明确的角色定义
- 合理的工具配置
- 适当的系统消息
- 清晰的职责分工

### 2. 对话流程设计
- 合理的终止条件
- 有效的发言控制
- 完善的错误处理
- 必要的人类介入

### 3. 安全考虑
- Docker 隔离执行
- 权限控制
- 审计日志
- 内容过滤

## 快速参考

### 基本 API

```python
# 创建 Agent
assistant = AssistantAgent(name, llm_config)
user_proxy = UserProxyAgent(name, code_execution_config)

# 创建群聊
groupchat = GroupChat(agents, messages, max_round)
manager = GroupChatManager(groupchat, llm_config)

# 开始对话
user_proxy.initiate_chat(manager, message)

# 注册工具
agent.register_for_execution()(function)
```

---

## 分析日期
2026-02-03
