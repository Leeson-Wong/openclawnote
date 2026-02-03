# 工具系统设计 - 详细定义

## 🎯 目标

详细定义 15 个 Agent 的工具集，每个工具包括：
- 名称和描述
- 输入参数
- 输出格式
- 实现框架
- 依赖项

---

## 📋 工具系统架构

### 工具接口规范（基于 LangChain）

```python
from typing import Any, Dict, Optional
from abc import ABC, abstractmethod

class BaseTool(ABC):
    """基础工具接口（基于 LangChain）"""
    
    def __init__(self):
        self.name: str = ""
        self.description: str = ""
        self.parameters: Dict[str, Any] = {}
        self.required_params: list = []
        
    @abstractmethod
    async def invoke(self, **kwargs) -> Dict[str, Any]:
        """执行工具"""
        pass
        
    @abstractmethod
    async def validate(self, **kwargs) -> bool:
        """验证参数"""
        pass
        
    def to_schema(self) -> Dict[str, Any]:
        """转换为工具 schema（用于 LLM）"""
        return {
            "name": self.name,
            "description": self.description,
            "parameters": {
                "type": "object",
                "properties": self.parameters,
                "required": self.required_params
            }
        }
```

### 工具注册机制（基于 Semantic Kernel）

```python
class ToolRegistry:
    """工具注册器（基于 Semantic Kernel Plugin）"""
    
    def __init__(self):
        self.tools: Dict[str, BaseTool] = {}
        self.categories: Dict[str, list] = {}
        
    def register(self, tool: BaseTool, category: str = "general"):
        """注册工具"""
        self.tools[tool.name] = tool
        if category not in self.categories:
            self.categories[category] = []
        self.categories[category].append(tool.name)
        
    def get_tool(self, tool_name: str) -> Optional[BaseTool]:
        """获取工具"""
        return self.tools.get(tool_name)
        
    def get_category_tools(self, category: str) -> list:
        """获取分类工具"""
        return [self.tools[name] for name in self.categories.get(category, [])]
        
    def search_tools(self, query: str) -> list:
        """搜索工具（基于描述）"""
        results = []
        query_lower = query.lower()
        
        for tool in self.tools.values():
            if (query_lower in tool.name.lower() or
                query_lower in tool.description.lower()):
                results.append(tool)
                
        return results
```

---

## 🤖 第一层：核心层工具定义

### Agent 1: Gateway Agent

#### 工具 1: WebSocketManager
```yaml
name: websocket_manager
description: 管理 WebSocket 连接，包括建立、维护和关闭连接
framework: asyncio + websockets
parameters:
  connection_id:
    type: string
    description: 连接 ID
  action:
    type: enum
    enum: [connect, disconnect, send, broadcast]
    description: 要执行的操作
  message:
    type: object
    description: 要发送的消息（仅当 action 为 send 或 broadcast 时需要）
required_params: [connection_id, action]
output:
  success: boolean
  connection_status: string
  error_message: string (optional)
```

#### 工具 2: HttpHandler
```yaml
name: http_handler
description: 处理 HTTP 请求，支持 GET, POST, PUT, DELETE 方法
framework: aiohttp + httpx
parameters:
  method:
    type: enum
    enum: [GET, POST, PUT, DELETE, PATCH]
    description: HTTP 方法
  url:
    type: string
    description: 请求 URL
  headers:
    type: object
    description: 请求头
  body:
    type: object
    description: 请求体
  timeout:
    type: number
    default: 30
    description: 超时时间（秒）
required_params: [method, url]
output:
  status_code: number
  headers: object
  body: object
  response_time: number
```

#### 工具 3: ProtocolParser
```yaml
name: protocol_parser
description: 解析各种通信协议（WebSocket, HTTP, gRPC 等）
framework: protocol parsers
parameters:
  protocol:
    type: enum
    enum: [websocket, http, grpc, mqtt, kafka]
    description: 协议类型
  raw_data:
    type: string
    description: 原始数据
  parse_options:
    type: object
    description: 解析选项
required_params: [protocol, raw_data]
output:
  parsed_data: object
  protocol_version: string
  metadata: object
  errors: array (optional)
```

#### 工具 4: RoutingEngine
```yaml
name: routing_engine
description: 路由消息到合适的 Agent 或服务
framework: 自定义路由引擎
parameters:
  message:
    type: object
    description: 要路由的消息
  routing_rules:
    type: object
    description: 路由规则
  fallback_route:
    type: string
    description: 失败时的默认路由
required_params: [message]
output:
  route: string
  routed_to: string
  confidence: number
  reasoning: string
```

#### 工具 5: VideoInterface
```yaml
name: video_interface
description: 基于 Vibecast 的实时视频接口
framework: Vibecast + WebRTC
parameters:
  action:
    type: enum
    enum: [connect, disconnect, stream, screenshot]
    description: 视频操作
  session_id:
    type: string
    description: 会话 ID
  stream_config:
    type: object
    description: 流配置（视频质量、分辨率等）
required_params: [action, session_id]
output:
  connection_id: string
  stream_url: string (optional)
  status: string
  error_message: string (optional)
```

---

### Agent 2: Execution Agent

#### 工具 1: CodeExecutor
```yaml
name: code_executor
description: 在安全的沙箱环境中执行代码
framework: Docker + OpenDevin
parameters:
  code:
    type: string
    description: 要执行的代码
  language:
    type: enum
    enum: [python, javascript, typescript, java, go, rust]
    description: 编程语言
  timeout:
    type: number
    default: 30
    description: 超时时间（秒）
  memory_limit:
    type: number
    default: 512
    description: 内存限制（MB）
required_params: [code, language]
output:
  success: boolean
  stdout: string
  stderr: string
  exit_code: number
  execution_time: number
```

#### 工具 2: ToolCaller
```yaml
name: tool_caller
description: 动态调用其他工具
framework: LangChain Tool Calling
parameters:
  tool_name:
    type: string
    description: 工具名称
  tool_args:
    type: object
    description: 工具参数
  timeout:
    type: number
    default: 30
    description: 超时时间（秒）
required_params: [tool_name, tool_args]
output:
  success: boolean
  result: object
  error_message: string (optional)
  execution_time: number
```

#### 工具 3: FileOperator
```yaml
name: file_operator
description: 文件操作工具（读写、删除、列出）
framework: Python built-in + pathlib
parameters:
  operation:
    type: enum
    enum: [read, write, append, delete, list, create_directory]
    description: 文件操作类型
  file_path:
    type: string
    description: 文件路径
  content:
    type: string
    description: 文件内容（仅用于 write 和 append）
  encoding:
    type: string
    default: utf-8
    description: 文件编码
required_params: [operation, file_path]
output:
  success: boolean
  content: string (optional)
  files: array (optional)
  error_message: string (optional)
```

#### 工具 4: ApiCaller
```yaml
name: api_caller
description: 调用外部 API
framework: httpx + aiohttp
parameters:
  method:
    type: enum
    enum: [GET, POST, PUT, DELETE, PATCH]
    description: HTTP 方法
  url:
    type: string
    description: API URL
  headers:
    type: object
    description: 请求头
  body:
    type: object
    description: 请求体
  auth:
    type: object
    description: 认证信息
required_params: [method, url]
output:
  status_code: number
  data: object
  headers: object
  error_message: string (optional)
```

#### 工具 5: ShellExecutor
```yaml
name: shell_executor
description: 执行 Shell 命令
framework: Python subprocess + asyncio
parameters:
  command:
    type: string
    description: Shell 命令
  working_directory:
    type: string
    description: 工作目录
  timeout:
    type: number
    default: 30
    description: 超时时间（秒）
  env:
    type: object
    description: 环境变量
required_params: [command]
output:
  success: boolean
  stdout: string
  stderr: string
  exit_code: number
  execution_time: number
```

---

### Agent 3: Memory Agent

#### 工具 1: ConversationBufferMemory
```yaml
name: conversation_buffer_memory
description: 管理当前对话的短期记忆
framework: LangChain ConversationBufferMemory
parameters:
  action:
    type: enum
    enum: [save, retrieve, clear, search]
    description: 记忆操作
  content:
    type: object
    description: 要保存的内容
  session_id:
    type: string
    description: 会话 ID
  query:
    type: string
    description: 搜索查询（仅用于 search）
required_params: [action]
output:
  success: boolean
  memory: array (optional)
  search_results: array (optional)
  error_message: string (optional)
```

#### 工具 2: ChatMemoryBuffer
```yaml
name: chat_memory_buffer
description: 管理会话级别的中期记忆
framework: LangChain ChatMemoryBuffer
parameters:
  action:
    type: enum
    enum: [save, retrieve, clear, summarize]
    description: 记忆操作
  content:
    type: object
    description: 要保存的内容
  user_id:
    type: string
    description: 用户 ID
  summary_length:
    type: number
    default: 200
    description: 摘要长度（仅用于 summarize）
required_params: [action]
output:
  success: boolean
  memory: array (optional)
  summary: string (optional)
  error_message: string (optional)
```

#### 工具 3: SemanticMemory
```yaml
name: semantic_memory
description: 管理语义级别的长期记忆
framework: MemGPT + LlamaIndex
parameters:
  action:
    type: enum
    enum: [save, retrieve, search, update, delete]
    description: 记忆操作
  content:
    type: object
    description: 要保存的内容
  embedding:
    type: array
    description: 内容的嵌入向量
  query:
    type: string
    description: 搜索查询（仅用于 retrieve 和 search）
  top_k:
    type: number
    default: 5
    description: 返回的结果数量（仅用于 search）
  memory_id:
    type: string
    description: 记忆 ID（仅用于 update 和 delete）
required_params: [action]
output:
  success: boolean
  memory_id: string (optional)
  retrieved_memory: object (optional)
  search_results: array (optional)
  error_message: string (optional)
```

#### 工具 4: ContextManager
```yaml
name: context_manager
description: 管理对话上下文窗口
framework: MemGPT
parameters:
  action:
    type: enum
    enum: [add, remove, compress, get, clear]
    description: 上下文操作
  message:
    type: object
    description: 要添加的消息
  max_tokens:
    type: number
    default: 8000
    description: 最大 token 数量
  compression_ratio:
    type: number
    default: 0.8
    description: 压缩比率（仅用于 compress）
required_params: [action]
output:
  success: boolean
  current_tokens: number
  compressed_context: array (optional)
  error_message: string (optional)
```

#### 工具 5: MemoryCompressor
```yaml
name: memory_compressor
description: 压缩和总结记忆
framework: MemGPT + LLM
parameters:
  content:
    type: string
    description: 要压缩的内容
  target_length:
    type: number
    default: 200
    description: 目标长度
  format:
    type: enum
    enum: [summary, key_points, bullet_points]
    default: summary
    description: 输出格式
required_params: [content]
output:
  success: boolean
  compressed_content: string
  original_length: number
  compressed_length: number
  compression_ratio: number
```

---

## 🤖 第二层：能力层工具定义

### Agent 4: Conversation Agent

#### 工具 1: TurnManager
```yaml
name: turn_manager
description: 管理对话轮次
framework: AutoGen
parameters:
  action:
    type: enum
    enum: [start_turn, end_turn, get_current_turn, get_history]
    description: 轮次操作
  user_id:
    type: string
    description: 用户 ID
  session_id:
    type: string
    description: 会话 ID
  turn_data:
    type: object
    description: 轮次数据（仅用于 start_turn）
required_params: [action, session_id]
output:
  success: boolean
  current_turn: number (optional)
  turn_data: object (optional)
  conversation_history: array (optional)
  error_message: string (optional)
```

#### 工具 2: PromptGenerator
```yaml
name: prompt_generator
description: 生成 LLM 提示词
framework: LangChain Prompt Templates
parameters:
  template:
    type: string
    description: 提示词模板
  variables:
    type: object
    description: 模板变量
  system_message:
    type: string
    description: 系统消息
  format:
    type: enum
    enum: [text, json, xml]
    default: text
    description: 输出格式
required_params: [template]
output:
  success: boolean
  prompt: string
  formatted_prompt: string
  token_count: number
```

#### 工具 3: MessageFormatter
```yaml
name: message_formatter
description: 格式化消息
framework: 自定义格式化器
parameters:
  message:
    type: object
    description: 要格式化的消息
  format:
    type: enum
    enum: [text, markdown, json, xml, html]
    description: 输出格式
  style:
    type: enum
    enum: [concise, detailed, professional, casual]
    default: concise
    description: 消息风格
required_params: [message]
output:
  success: boolean
  formatted_message: string
  metadata: object
```

#### 工具 4: ContextWindowManager
```yaml
name: context_window_manager
description: 管理上下文窗口
framework: MemGPT
parameters:
  action:
    type: enum
    enum: [add_message, remove_oldest, get_window, clear_window]
    description: 窗口操作
  message:
    type: object
    description: 要添加的消息
  max_messages:
    type: number
    default: 20
    description: 最大消息数
  max_tokens:
    type: number
    default: 8000
    description: 最大 token 数
required_params: [action]
output:
  success: boolean
  current_window: array
  current_token_count: number
  error_message: string (optional)
```

#### 工具 5: ConversationAnalytics
```yaml
name: conversation_analytics
description: 分析对话统计信息
framework: 自定义分析器
parameters:
  conversation_id:
    type: string
    description: 对话 ID
  metrics:
    type: array
    items:
      type: enum
      enum: [turn_count, avg_response_time, sentiment, topic_distribution, user_satisfaction]
    description: 要分析的指标
required_params: [conversation_id]
output:
  success: boolean
  metrics: object
  insights: array
  error_message: string (optional)
```

---

由于篇幅限制，我将在下一部分继续定义剩余的 Agent 工具。

**当前进度**:
- ✅ 核心层（3 个 Agent）：15 个工具
- ⏳ 能力层（5 个 Agent）：待继续
- ⏳ 协调层（1 个 Agent）：待继续
- ⏳ 专业层（6 个 Agent）：待继续

**完成时间**: 2026-02-03 15:15
**总工具数**: 预计 75 个工具

**下一部分**: 能力层和协调层工具定义
