# Semantic Kernel 项目 - 操作指南

## 快速开始

### 安装

#### Python
```bash
pip install semantic-kernel
```

#### C#/.NET
```bash
dotnet add package Microsoft.SemanticKernel
dotnet add package Microsoft.SemanticKernel.Connectors.OpenAI
```

#### Java
```xml
<dependency>
    <groupId>com.microsoft.semantic-kernel</groupId>
    <artifactId>semantic-kernel</artifactId>
    <version>latest</version>
</dependency>
```

### 基本使用

#### Python 示例
```python
from semantic_kernel import Kernel
from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion

# 创建 Kernel
kernel = Kernel()

# 添加服务
kernel.add_service(AzureChatCompletion(
    deployment_name="gpt-4",
    api_key="your-key",
    endpoint="https://your-endpoint.openai.azure.com/"
))

# 使用 Prompt
result = kernel.invoke_prompt("Hello {{$name}}!", name="World")
print(result)
```

#### C# 示例
```csharp
using Microsoft.SemanticKernel;

// 创建 Kernel
var kernel = Kernel.CreateBuilder()
    .AddOpenAIChatCompletion(modelId, apiKey)
    .Build();

// 使用 Prompt
var prompt = "Hello {{$input}}!";
var function = kernel.CreateFunctionFromPrompt(prompt);
var result = await kernel.InvokeAsync(function, new() { ["input"] = "World" });
Console.WriteLine(result);
```

## 核心功能使用

### 1. 创建 Semantic Functions

#### Python
```python
from semantic_kernel import KernelFunction

# 创建语义函数
summarize_function = kernel.create_function_from_prompt(
    prompt_template="Summarize the following: {{$input}}",
    function_name="Summarize",
    description="Summarize the given text"
)

# 执行
result = await kernel.invoke_async(summarize_function, input="Long text here...")
```

#### C#
```csharp
// 创建语义函数
var summarizeFunction = kernel.CreateFunctionFromPrompt(
    "Summarize the following: {{$input}}",
    functionName: "Summarize",
    description: "Summarize the given text"
);

// 执行
var result = await kernel.InvokeAsync(summarizeFunction, 
    new() { ["input"] = "Long text here..." });
```

### 2. 使用 Native Functions

#### Python
```python
from semantic_kernel.functions import kernel_function

class MathPlugin:
    @kernel_function
    def add(self, a: float, b: float) -> float:
        """Add two numbers"""
        return a + b

# 导入插件
kernel.import_plugin_from_object(MathPlugin(), "math")

# 调用
result = await kernel.invoke("math.add", a=5, b=3)
print(result)  # 8.0
```

#### C#
```csharp
public class MathPlugin
{
    [KernelFunction, Description("Add two numbers")]
    public float Add(float a, float b)
    {
        return a + b;
    }
}

// 导入插件
kernel.ImportPluginFromObject(new MathPlugin(), "math");

// 调用
var result = await kernel.InvokeAsync("math.add", 
    new() { ["a"] = 5, ["b"] = 3 });
```

### 3. 使用 Planner

#### Python
```python
from semantic_kernel.planning import SequentialPlanner

# 创建规划器
planner = SequentialPlanner(kernel)

# 规划执行
plan = await planner.create_plan_async("Write a poem about AI and then summarize it")

# 执行计划
result = await plan.invoke_async()
print(result)
```

### 4. 使用 Memory

#### Python
```python
from semantic_kernel.memory import SemanticTextMemory

# 创建记忆
memory = SemanticTextMemory()

# 添加到记忆
await memory.save_information_async(
    collection="facts",
    text="The sky is blue",
    id="fact1"
)

# 搜索记忆
results = await memory.search_async(
    collection="facts",
    query="color of sky"
)

for result in results:
    print(f"{result.id}: {result.text}")
```

### 5. 连接外部数据

#### Python
```python
import aiohttp

async def get_weather(city: str) -> str:
    """Get weather information for a city"""
    async with aiohttp.ClientSession() as session:
        async with session.get(f"https://api.weather.com/{city}") as response:
            return await response.text()

# 导入为插件
kernel.import_function(get_weather, "weather")

# 使用
result = await kernel.invoke("weather.get_weather", city="New York")
```

## 高级配置

### 1. 服务配置

#### Python
```python
from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion

# 配置服务
service = AzureChatCompletion(
    deployment_name="gpt-4",
    api_key="your-key",
    endpoint="https://your-endpoint.openai.azure.com/",
    api_version="2024-02-15-preview",
    default_headers={"User-Agent": "MyApp/1.0"}
)

kernel.add_service(service)
```

#### C#
```csharp
// 配置服务
var service = new AzureOpenAIChatCompletionService(
    deploymentName: "gpt-4",
    apiKey: "your-key",
    endpoint: "https://your-endpoint.openai.azure.com/",
    apiVersion: "2024-02-15-preview",
    httpClient: new HttpClient()
);

kernel.Services.AddService(service);
```

### 2. 提示模板配置

#### Python
```python
# 高级提示模板
prompt_template = """
You are a helpful assistant.
Context: {{$context}}
Question: {{$question}}
Answer:"""

function = kernel.create_function_from_prompt(
    prompt_template=prompt_template,
    function_name="Answer",
    description="Answer a question based on context",
    input_variables=[
        {"name": "context", "description": "Context information"},
        {"name": "question", "description": "Question to answer"}
    ],
    execution_settings={"max_tokens": 1000, "temperature": 0.7}
)
```

### 3. 流式输出

#### Python
```python
async def stream_response():
    function = kernel.create_function_from_prompt("Tell me a story")
    
    async for chunk in kernel.invoke_stream_async(function):
        print(chunk, end="", flush=True)

await stream_response()
```

## 常见问题排查

### 1. API 认证问题

#### 问题
```
AuthenticationError: Invalid API key
```

#### 解决方案
```python
# 检查 API key 配置
service = AzureChatCompletion(
    api_key="your-valid-key",  # 确保密钥正确
    endpoint="https://your-endpoint.openai.azure.com/"
)
```

### 2. 插件导入失败

#### 问题
```
ImportError: Plugin not found
```

#### 解决方案
```python
# 确保插件正确导入
kernel.import_plugin_from_object(plugin_object, "plugin_name")
# 或
kernel.import_plugin_from_directory("./plugins", "plugin_name")
```

### 3. 内存使用问题

#### 问题
- 记忆存储占用过多内存

#### 解决方案
```python
# 使用持久化记忆存储
from semantic_kernel.connectors.memory.azure_cognitive_search import AzureCognitiveSearchMemoryStore

memory_store = AzureCognitiveSearchMemoryStore(
    endpoint="https://your-service.search.windows.net",
    api_key="your-key"
)

memory = SemanticTextMemory(storage=memory_store)
```

## 最佳实践

### 1. 组织代码结构

#### 推荐结构
```
project/
├── plugins/
│   ├── __init__.py
│   ├── math_plugin.py
│   └── weather_plugin.py
├── prompts/
│   ├── summarizer.txt
│   └── translator.txt
└── main.py
```

### 2. 使用配置文件

#### 配置示例
```python
# config.py
CONFIG = {
    "azure_openai": {
        "deployment_name": "gpt-4",
        "api_key": os.getenv("AZURE_OPENAI_KEY"),
        "endpoint": os.getenv("AZURE_OPENAI_ENDPOINT")
    },
    "memory": {
        "storage_type": "cosmos_db",
        "connection_string": os.getenv("COSMOS_CONNECTION")
    }
}
```

### 3. 错误处理

```python
from semantic_kernel import SKException

try:
    result = await kernel.invoke_async(function, input="data")
except SKException as e:
    print(f"Semantic Kernel error: {e}")
except Exception as e:
    print(f"General error: {e}")
```

## 快速参考

### 基本 API

```python
# 创建 Kernel
kernel = Kernel()

# 添加服务
kernel.add_service(service)

# 导入插件
kernel.import_plugin_from_object(plugin, "name")

# 创建函数
function = kernel.create_function_from_prompt(prompt)

# 执行
result = await kernel.invoke_async(function, variables)
```

---

## 分析日期
2026-02-03
