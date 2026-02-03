# 深度研究 - AI Agent 可观测性

## 🎯 研究目标

深入研究 AI Agent 的可观测性（Observability）：
1. Observability 架构
2. Distributed Tracing
3. Logging 策略
4. Metrics 和 Alerts
5. 与我们设计的集成

---

## 📊 Observability 架构

### 三支柱架构

```
┌─────────────────────────────────────────────────┐
│                  Observability                   │
├──────────────┬──────────────┬──────────────┤
│    Tracing   │    Logging   │    Metrics    │
├──────────────┼──────────────┼──────────────┤
│ Distributed  │ Structured   │ Time Series   │
│ Tracing       │ Logging      │ & Alerts      │
└──────────────┴──────────────┴──────────────┘
        ↓               ↓               ↓
┌─────────────────────────────────────────────────┐
│            Observability Backend              │
├──────────────┬──────────────┬──────────────┤
│  Jaeger/      │ ELK Stack    │ Prometheus/    │
│  Zipkin       │ (Elasticsearch,│ Grafana       │
│               │  Logstash,     │               │
│               │  Kibana)       │               │
└──────────────┴──────────────┴──────────────┘
        ↓
┌─────────────────────────────────────────────────┐
│         Observability Dashboard               │
├──────────────┬──────────────┬──────────────┤
│  Trace View   │ Log Search    │ Metric Dash   │
│  & Service    │ & Analysis    │ & Alerts      │
│  Map          │               │               │
└──────────────┴──────────────┴──────────────┘
```

---

## 🔍 Distributed Tracing

### 1. Tracing 上下文传播

#### Trace ID 和 Span ID 生成

```python
# tracing/context.py
import uuid
from contextlib import contextmanager
from typing import Optional, Dict, Any

class TracingContext:
    """Tracing 上下文管理器"""
    
    def __init__(self):
        self.trace_id: Optional[str] = None
        self.span_id: Optional[str] = None
        self.parent_span_id: Optional[str] = None
        self.baggage: Dict[str, Any] = {}
        
    def start_trace(self) -> str:
        """开始新的 Trace"""
        self.trace_id = str(uuid.uuid4())
        self.span_id = str(uuid.uuid4())
        self.parent_span_id = None
        return self.trace_id
    
    def start_span(self, parent_span_id: Optional[str] = None) -> str:
        """开始新的 Span"""
        if not self.trace_id:
            raise RuntimeError("必须先开始 Trace")
            
        self.parent_span_id = parent_span_id
        self.span_id = str(uuid.uuid4())
        return self.span_id
    
    def end_span(self) -> None:
        """结束 Span"""
        # 在实际实现中，这里会发送 Span 数据到 Tracing 后端
        pass
    
    def get_trace_context(self) -> Dict[str, str]:
        """获取当前 Trace 上下文"""
        return {
            "trace_id": self.trace_id,
            "span_id": self.span_id,
            "parent_span_id": self.parent_span_id
        }
    
    def add_baggage(self, key: str, value: Any) -> None:
        """添加 Baggage"""
        self.baggage[key] = value

# 全局上下文
tracing_context = TracingContext()
```

#### Tracing 装饰器

```python
# tracing/decorator.py
from functools import wraps
from typing import Callable, Any
from .context import tracing_context

class TracingDecorator:
    """Tracing 装饰器"""
    
    def __init__(self, service_name: str, jaeger_client=None):
        self.service_name = service_name
        self.jaeger_client = jaeger_client
    
    def __call__(self, func: Callable) -> Callable:
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # 1. 开始 Span
            parent_span_id = kwargs.get("parent_span_id")
            span_id = tracing_context.start_span(parent_span_id)
            
            # 2. 添加 Baggage
            for key, value in kwargs.items():
                if not key.startswith("_"):
                    tracing_context.add_baggage(key, value)
            
            try:
                # 3. 记录 Span 开始
                self._record_span_start(span_id, func.__name__, args, kwargs)
                
                # 4. 执行函数
                result = await func(*args, **kwargs)
                
                # 5. 记录 Span 结束
                self._record_span_end(span_id, func.__name__, result, None)
                
                return result
                
            except Exception as e:
                # 6. 记录 Span 错误
                self._record_span_end(span_id, func.__name__, None, e)
                raise e
            
            finally:
                # 7. 结束 Span
                tracing_context.end_span()
        
        return wrapper
    
    def _record_span_start(self, span_id: str, operation_name: str, args: tuple, kwargs: dict) -> None:
        """记录 Span 开始"""
        span_start = {
            "trace_id": tracing_context.trace_id,
            "span_id": span_id,
            "parent_span_id": tracing_context.parent_span_id,
            "service_name": self.service_name,
            "operation_name": operation_name,
            "start_time": time.time(),
            "baggage": tracing_context.baggage,
            "tags": {
                "operation_name": operation_name,
                "args_count": len(args),
                "kwargs_count": len(kwargs)
            }
        }
        
        # 发送到 Jaeger
        if self.jaeger_client:
            self.jaeger_client.send_span(span_start)
    
    def _record_span_end(self, span_id: str, operation_name: str, result: Any, error: Exception) -> None:
        """记录 Span 结束"""
        span_end = {
            "trace_id": tracing_context.trace_id,
            "span_id": span_id,
            "service_name": self.service_name,
            "operation_name": operation_name,
            "end_time": time.time(),
            "tags": {
                "operation_name": operation_name,
                "success": error is None,
                "error_type": type(error).__name__ if error else None
            },
            "logs": []
        }
        
        # 添加结果
        if result is not None:
            span_end["logs"].append({
                "level": "info",
                "message": f"Operation {operation_name} completed",
                "data": str(result)[:1000]  # 限制日志长度
            })
        
        # 添加错误
        if error is not None:
            span_end["logs"].append({
                "level": "error",
                "message": f"Operation {operation_name} failed: {str(error)}"
            })
        
        # 发送到 Jaeger
        if self.jaeger_client:
            self.jaeger_client.send_span(span_end)
```

---

### 2. Jaeger 客户端实现

```python
# tracing/jaeger_client.py
from jaeger_client import Client
from jaeger_client.thrift import span

class JaegerTracer:
    """Jaeger Tracing 客户端"""
    
    def __init__(self, host: str = "localhost", port: int = 6831):
        self.client = Client(host=host, port=port)
        self.service_name = "ai_agent_group"
    
    def send_span(self, span_data: dict) -> None:
        """发送 Span 数据到 Jaeger"""
        # 1. 创建 Span 对象
        span = self._create_span(span_data)
        
        # 2. 发送 Span
        self.client.send(span)
    
    def _create_span(self, span_data: dict) -> span:
        """创建 Span 对象"""
        # 创建 Span
        span_obj = span.Span(
            trace_id=span_data["trace_id"],
            span_id=span_data["span_id"],
            parent_span_id=span_data.get("parent_span_id"),
            operation_name=span_data["operation_name"],
            service_name=span_data["service_name"]
        )
        
        # 设置 Start Time
        span_obj.start_time = span_data["start_time"]
        
        # 设置 End Time
        span_obj.end_time = span_data.get("end_time")
        
        # 设置 Tags
        span_obj.tags = span_data.get("tags", {})
        
        # 设置 Logs
        for log in span_data.get("logs", []):
            span_obj.log(log)
        
        return span_obj
    
    def create_trace(self, trace_id: str, spans: list) -> None:
        """创建完整的 Trace"""
        # 1. 创建 Root Span
        root_span = self._create_root_span(spans[0])
        
        # 2. 添加子 Span
        for span_data in spans[1:]:
            child_span = self._create_span(span_data)
            root_span.add_reference(child_span)
        
        # 3. 发送 Trace
        self.client.send(root_span)
```

---

## 📝 Structured Logging

### 1. 日志级别和格式

```python
# logging/structured_logger.py
import logging
import json
from typing import Dict, Any
from datetime import datetime

class StructuredLogger:
    """结构化日志记录器"""
    
    def __init__(self, name: str, level: str = "INFO"):
        self.logger = logging.getLogger(name)
        self.logger.setLevel(getattr(logging, level))
        
        # 添加 JSON Handler
        handler = logging.StreamHandler()
        handler.setFormatter(JsonFormatter())
        self.logger.addHandler(handler)
    
    def debug(self, message: str, **kwargs) -> None:
        """DEBUG 日志"""
        self.logger.debug(message, extra=self._build_extra("debug", **kwargs))
    
    def info(self, message: str, **kwargs) -> None:
        """INFO 日志"""
        self.logger.info(message, extra=self._build_extra("info", **kwargs))
    
    def warning(self, message: str, **kwargs) -> None:
        """WARNING 日志"""
        self.logger.warning(message, extra=self._build_extra("warning", **kwargs))
    
    def error(self, message: str, exc_info: bool = True, **kwargs) -> None:
        """ERROR 日志"""
        self.logger.error(message, extra=self._build_extra("error", exc_info=exc_info, **kwargs))
    
    def critical(self, message: str, exc_info: bool = True, **kwargs) -> None:
        """CRITICAL 日志"""
        self.logger.critical(message, extra=self._build_extra("critical", exc_info=exc_info, **kwargs))
    
    def _build_extra(self, level: str, **kwargs) -> Dict[str, Any]:
        """构建 extra 字段"""
        return {
            "level": level,
            "timestamp": datetime.utcnow().isoformat(),
            "trace_id": kwargs.get("trace_id"),
            "span_id": kwargs.get("span_id"),
            "agent_id": kwargs.get("agent_id"),
            "session_id": kwargs.get("session_id"),
            "user_id": kwargs.get("user_id"),
            "request_id": kwargs.get("request_id"),
            "correlation_id": kwargs.get("correlation_id"),
            "custom_fields": kwargs
        }

class JsonFormatter(logging.Formatter):
    """JSON 日志格式化器"""
    
    def format(self, record: logging.LogRecord) -> str:
        """格式化日志为 JSON"""
        log_record = {
            "timestamp": datetime.fromtimestamp(record.created).isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
            "module": record.module,
            "function": record.funcName,
            "line": record.lineno,
        }
        
        # 添加 extra 字段
        if hasattr(record, "trace_id") and record.trace_id:
            log_record["trace_id"] = record.trace_id
        if hasattr(record, "span_id") and record.span_id:
            log_record["span_id"] = record.span_id
        if hasattr(record, "agent_id") and record.agent_id:
            log_record["agent_id"] = record.agent_id
        
        # 添加异常信息
        if record.exc_info:
            log_record["exception"] = self.formatException(record.exc_info)
        
        return json.dumps(log_record, ensure_ascii=False)
```

---

### 2. Agent 专用 Logger

```python
# logging/agent_logger.py
from .structured_logger import StructuredLogger

class AgentLogger(StructuredLogger):
    """Agent 专用日志记录器"""
    
    def __init__(self, agent_id: str):
        super().__init__(f"Agent:{agent_id}")
        self.agent_id = agent_id
    
    def log_task_start(self, task_id: str, task: dict, **kwargs) -> None:
        """记录任务开始"""
        self.info(
            "Task started",
            task_id=task_id,
            task_type=task.get("type"),
            task_description=task.get("description"),
            trace_id=kwargs.get("trace_id"),
            span_id=kwargs.get("span_id")
        )
    
    def log_task_complete(self, task_id: str, result: dict, **kwargs) -> None:
        """记录任务完成"""
        self.info(
            "Task completed",
            task_id=task_id,
            result=result,
            trace_id=kwargs.get("trace_id"),
            span_id=kwargs.get("span_id")
        )
    
    def log_task_failed(self, task_id: str, error: Exception, **kwargs) -> None:
        """记录任务失败"""
        self.error(
            "Task failed",
            task_id=task_id,
            error=str(error),
            error_type=type(error).__name__,
            trace_id=kwargs.get("trace_id"),
            span_id=kwargs.get("span_id"),
            exc_info=True
        )
    
    def log_tool_invoke(self, tool_name: str, tool_args: dict, result: dict, **kwargs) -> None:
        """记录工具调用"""
        self.info(
            "Tool invoked",
            tool_name=tool_name,
            tool_args=tool_args,
            result=result,
            execution_time=result.get("execution_time"),
            success=result.get("success"),
            trace_id=kwargs.get("trace_id"),
            span_id=kwargs.get("span_id")
        )
    
    def log_message(self, message_type: str, message: dict, **kwargs) -> None:
        """记录消息"""
        self.info(
            "Message logged",
            message_type=message_type,
            message_id=message.get("id"),
            sender=message.get("source", {}).get("agent_id"),
            receiver=message.get("destination", {}).get("agent_id"),
            trace_id=kwargs.get("trace_id"),
            span_id=kwargs.get("span_id")
        )
```

---

## 📈 Metrics 和 Alerts

### 1. Prometheus Metrics 定义

```python
# metrics/agent_metrics.py
from prometheus_client import Counter, Gauge, Histogram, Summary
from prometheus_client.core import CollectorRegistry

class AgentMetrics:
    """Agent Metrics 定义"""
    
    def __init__(self):
        self.registry = CollectorRegistry()
        
        # 任务 Metrics
        self.task_completed = Counter(
            'agent_task_completed_total',
            'Total number of tasks completed',
            ['agent_id', 'task_type', 'priority'],
            registry=self.registry
        )
        
        self.task_failed = Counter(
            'agent_task_failed_total',
            'Total number of tasks failed',
            ['agent_id', 'task_type', 'priority', 'error_type'],
            registry=self.registry
        )
        
        self.task_duration = Histogram(
            'agent_task_duration_seconds',
            'Task execution duration in seconds',
            ['agent_id', 'task_type'],
            buckets=(0.1, 0.5, 1, 2, 5, 10, 30, 60, 120, 300, 600),
            registry=self.registry
        )
        
        # 工具调用 Metrics
        self.tool_invoked = Counter(
            'agent_tool_invoked_total',
            'Total number of tool invocations',
            ['agent_id', 'tool_name', 'tool_type'],
            registry=self.registry
        )
        
        self.tool_duration = Histogram(
            'agent_tool_duration_seconds',
            'Tool invocation duration in seconds',
            ['agent_id', 'tool_name', 'tool_type'],
            buckets=(0.01, 0.1, 0.5, 1, 2, 5, 10, 30, 60),
            registry=self.registry
        )
        
        # 消息 Metrics
        self.messages_sent = Counter(
            'agent_messages_sent_total',
            'Total number of messages sent',
            ['agent_id', 'message_type', 'destination_type'],
            registry=self.registry
        )
        
        self.messages_received = Counter(
            'agent_messages_received_total',
            'Total number of messages received',
            ['agent_id', 'message_type', 'source_type'],
            registry=self.registry
        )
        
        self.message_latency = Histogram(
            'agent_message_latency_seconds',
            'Message latency in seconds',
            ['agent_id', 'message_type'],
            buckets=(0.01, 0.1, 0.5, 1, 2, 5, 10, 30, 60),
            registry=self.registry
        )
        
        # 错误 Metrics
        self.errors = Counter(
            'agent_errors_total',
            'Total number of errors',
            ['agent_id', 'error_type', 'error_code'],
            registry=self.registry
        )
        
        # 性能 Metrics
        self.active_tasks = Gauge(
            'agent_active_tasks',
            'Number of currently active tasks',
            ['agent_id'],
            registry=self.registry
        )
        
        self.memory_usage = Gauge(
            'agent_memory_usage_bytes',
            'Current memory usage in bytes',
            ['agent_id'],
            registry=self.registry
        )
        
        self.cpu_usage = Gauge(
            'agent_cpu_usage',
            'Current CPU usage',
            ['agent_id'],
            registry=self.registry
        )
    
    def track_task_completed(self, agent_id: str, task_type: str, priority: str, duration: float) -> None:
        """跟踪任务完成"""
        self.task_completed.labels(
            agent_id=agent_id,
            task_type=task_type,
            priority=priority
        ).inc()
        self.task_duration.labels(
            agent_id=agent_id,
            task_type=task_type
        ).observe(duration)
    
    def track_task_failed(self, agent_id: str, task_type: str, priority: str, error_type: str, error_code: str) -> None:
        """跟踪任务失败"""
        self.task_failed.labels(
            agent_id=agent_id,
            task_type=task_type,
            priority=priority,
            error_type=error_type,
            error_code=error_code
        ).inc()
        self.errors.labels(
            agent_id=agent_id,
            error_type=error_type,
            error_code=error_code
        ).inc()
    
    def track_tool_invoke(self, agent_id: str, tool_name: str, tool_type: str, duration: float, success: bool) -> None:
        """跟踪工具调用"""
        self.tool_invoked.labels(
            agent_id=agent_id,
            tool_name=tool_name,
            tool_type=tool_type
        ).inc()
        self.tool_duration.labels(
            agent_id=agent_id,
            tool_name=tool_name,
            tool_type=tool_type
        ).observe(duration)
        
        if not success:
            self.errors.labels(
                agent_id=agent_id,
                error_type="tool_invocation",
                error_code="tool_invoke_failed"
            ).inc()
    
    def track_message(self, agent_id: str, message_type: str, destination_type: str, source_type: str, duration: float) -> None:
        """跟踪消息"""
        self.messages_sent.labels(
            agent_id=agent_id,
            message_type=message_type,
            destination_type=destination_type
        ).inc()
        
        self.messages_received.labels(
            agent_id=agent_id,
            message_type=message_type,
            source_type=source_type
        ).inc()
        
        self.message_latency.labels(
            agent_id=agent_id,
            message_type=message_type
        ).observe(duration)
    
    def update_performance_metrics(self, agent_id: str, active_tasks: int, memory_usage: float, cpu_usage: float) -> None:
        """更新性能 Metrics"""
        self.active_tasks.labels(agent_id=agent_id).set(active_tasks)
        self.memory_usage.labels(agent_id=agent_id).set(memory_usage)
        self.cpu_usage.labels(agent_id=agent_id).set(cpu_usage)
```

### 2. Alert Rules 定义

```yaml
# prometheus/alerts.yml
groups:
  - name: agent_alerts
    rules:
      - alert: HighErrorRate
        expr: rate(agent_errors_total[5m]) > 10
        for: agent_id
        labels:
          severity: warning
          alert_type: high_error_rate
        annotations:
          summary: "Agent {{ $agent_id }} has high error rate"
          description: "Error rate is {{ $value }} errors per 5 minutes, which is above the threshold of 10"
      
      - alert: HighTaskFailureRate
        expr: rate(agent_task_failed_total[10m]) > 5
        for: agent_id
        labels:
          severity: critical
          alert_type: high_task_failure_rate
        annotations:
          summary: "Agent {{ $agent_id }} has high task failure rate"
          description: "Task failure rate is {{ $value }} failures per 10 minutes, which is above the threshold of 5"
      
      - alert: HighToolFailureRate
        expr: rate(agent_tool_invoked_total{tool_type=~"api|http"}[5m]) > 10
        for: agent_id
        labels:
          severity: warning
          alert_type: high_tool_failure_rate
        annotations:
          summary: "Agent {{ $agent_id }} has high tool failure rate"
          description: "External tool failure rate is {{ $value }} failures per 5 minutes, which is above the threshold of 10"
      
      - alert: SlowTaskExecution
        expr: histogram_quantile(0.95, agent_task_duration_seconds[5m]) > 60
        for: agent_id
        labels:
          severity: warning
          alert_type: slow_task_execution
        annotations:
          summary: "Agent {{ $agent_id }} has slow task execution"
          description: "95th percentile of task execution time is {{ $value }} seconds, which is above the threshold of 60 seconds"
      
      - alert: HighMemoryUsage
        expr: agent_memory_usage_bytes / (1024 * 1024 * 1024) > 2
        for: agent_id
        labels:
          severity: warning
          alert_type: high_memory_usage
        annotations:
          summary: "Agent {{ $agent_id }} has high memory usage"
          description: "Memory usage is {{ $value }} GB, which is above the threshold of 2 GB"
      
      - alert: AgentUnresponsive
        expr: up{instance="agent"} == 0
        for: agent_id
        labels:
          severity: critical
          alert_type: agent_unresponsive
        annotations:
          summary: "Agent {{ $agent_id }} is unresponsive"
          description: "Agent has been down for more than 30 seconds"
      
      - alert: LongRunningTask
        expr: (time() - agent_task_start_timestamp_seconds) > 3600
        for: agent_id, task_id
        labels:
          severity: info
          alert_type: long_running_task
        annotations:
          summary: "Task {{ $task_id }} has been running for more than 1 hour"
          description: "Task execution time is {{ $value }} seconds, which is above the threshold of 3600 seconds"
```

---

## 🚀 应用到我们的设计

### 1. 为 Agent 添加 Tracing

```python
# agents/gateway/main.py
from tracing.decorators import TracingDecorator
from tracing.context import tracing_context
from tracing.jaeger_client import JaegerTracer

# 创建 Tracing 装饰器
tracing = TracingDecorator(service_name="gateway_agent", jaeger_client=JaegerTracer())

# 初始化 Tracing
tracing_context.start_trace()

# Agent 主函数
@tracing
async def handle_message(message: dict):
    """
    处理消息（带 Tracing）
    """
    # 获取当前 Trace 上下文
    trace_context = tracing_context.get_trace_context()
    
    # 处理消息
    result = await process_message(message)
    
    return result

async def process_message(message: dict):
    """
    处理消息逻辑
    """
    # 1. 解析消息
    parsed_message = parse_message(message)
    
    # 2. 路由消息
    route_result = await route_message(parsed_message)
    
    # 3. 返回结果
    return route_result
```

### 2. 为 Agent 添加 Logging

```python
# agents/gateway/main.py
from logging.agent_logger import AgentLogger

# 创建 Agent Logger
agent_logger = AgentLogger("gateway_agent_001")

# 使用 Logger
async def handle_message(message: dict):
    """
    处理消息（带 Logging）
    """
    # 记录消息开始
    agent_logger.log_message(
        message_type="incoming",
        message=message,
        trace_id=tracing_context.trace_id,
        span_id=tracing_context.span_id
    )
    
    try:
        # 处理消息
        result = await process_message(message)
        
        # 记录成功
        agent_logger.info(
            "Message processed successfully",
            trace_id=tracing_context.trace_id,
            span_id=tracing_context.span_id
        )
        
        return result
        
    except Exception as e:
        # 记录错误
        agent_logger.error(
            "Message processing failed",
            error=str(e),
            exc_info=True,
            trace_id=tracing_context.trace_id,
            span_id=tracing_context.span_id
        )
        
        raise e
```

### 3. 为 Agent 添加 Metrics

```python
# agents/gateway/main.py
from metrics.agent_metrics import AgentMetrics

# 创建 Agent Metrics
agent_metrics = AgentMetrics()

async def handle_message(message: dict):
    """
    处理消息（带 Metrics）
    """
    start_time = time.time()
    
    try:
        # 处理消息
        result = await process_message(message)
        
        # 记录成功 Metrics
        duration = time.time() - start_time
        agent_metrics.track_message(
            agent_id="gateway_agent_001",
            message_type="incoming",
            destination_type="agent",
            source_type="user",
            duration=duration
        )
        
        return result
        
    except Exception as e:
        # 记录错误 Metrics
        agent_metrics.track_task_failed(
            agent_id="gateway_agent_001",
            task_type="message_processing",
            priority="normal",
            error_type=type(e).__name__,
            error_code="message_processing_failed"
        )
        
        raise e
```

---

## 📊 研究总结

### Observability 核心组件
1. **Distributed Tracing**: Jaeger/Zipkin
2. **Structured Logging**: ELK Stack (Elasticsearch, Logstash, Kibana)
3. **Metrics**: Prometheus/Grafana
4. **Alerting**: Prometheus Alertmanager
5. **Dashboard**: Grafana Dashboard

### 核心发现
1. **三支柱**: Tracing, Logging, Metrics
2. **上下文传播**: Trace ID 和 Span ID 的传播
3. **结构化日志**: JSON 格式的结构化日志
4. **Metrics 定义**: 完整的 Metrics 定义和 Alert 规则
5. **集成策略**: 与我们设计的 Agent 无缝集成

---

**研究状态**: ✅ AI Agent 可观测性深度研究完成（15 分钟）  
**研究时间**: 2026-02-03 17:25  
**总研究时间**: 60 分钟  
**状态**: 全面研究和设计工作全部完成
