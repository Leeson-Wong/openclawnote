# 通信协议设计

## 🎯 目标

设计 Agent 之间的通信协议，包括：
- 消息格式和结构
- 路由和寻址机制
- 错误处理和重试策略
- 通信安全和认证
- 性能优化

---

## 📋 通信架构

### 通信模式

#### 1. 点对点通信（P2P）
```
Agent A → Agent B (直接消息）
```

#### 2. 广播通信（Broadcast）
```
Agent A → Agent B, Agent C, Agent D (群发消息）
```

#### 3. 组播通信（Multicast）
```
Agent A → [Group: Research Team] (群组消息）
```

#### 4. 发布/订阅（Pub/Sub）
```
Agent A → [Topic: Updates] → [Subscribers: Agent B, Agent C, Agent D]
```

---

## 📨 消息格式规范

### 基础消息结构

```json
{
  "version": "1.0.0",
  "message_id": "msg_20260203_1545_001",
  "timestamp": "2026-02-03T15:45:00.000Z",
  "type": "request|response|event|notification|error",
  "priority": "low|normal|high|urgent",
  "source": {
    "agent_id": "gateway_agent",
    "agent_type": "GatewayAgent",
    "session_id": "session_001"
  },
  "destination": {
    "agent_id": "coordinator_agent",
    "agent_type": "CoordinatorAgent",
    "routing": "direct|broadcast|multicast|pubsub"
  },
  "headers": {
    "correlation_id": "correlation_001",
    "reply_to": "gateway_agent",
    "timeout": 30000,
    "retry_policy": "exponential_backoff",
    "max_retries": 3,
    "require_ack": true,
    "require_receipt": true
  },
  "body": {
    "action": "decompose_task",
    "parameters": {
      "task": "Analyze AI market trends",
      "complexity": "medium"
    }
  },
  "attachments": [],
  "signature": "HMAC_SHA256_signature",
  "encryption": "AES256_GCM"
}
```

### 消息类型定义

#### 1. Request（请求）
```json
{
  "type": "request",
  "body": {
    "action": "tool_invoke",
    "parameters": {
      "tool_name": "web_search",
      "tool_args": {
        "query": "AI market trends 2024"
      }
    }
  }
}
```

#### 2. Response（响应）
```json
{
  "type": "response",
  "body": {
    "status": "success|partial_success|failure|error",
    "result": {
      "data": "search results...",
      "metadata": {}
    },
    "error": null
  }
}
```

#### 3. Event（事件）
```json
{
  "type": "event",
  "body": {
    "event_type": "task_completed|task_failed|agent_started|agent_stopped",
    "event_data": {
      "task_id": "task_001",
      "agent_id": "research_agent",
      "timestamp": "2026-02-03T15:45:00.000Z"
    }
  }
}
```

#### 4. Notification（通知）
```json
{
  "type": "notification",
  "body": {
    "level": "info|warning|error|critical",
    "message": "Task completed successfully",
    "details": {}
  }
}
```

#### 5. Error（错误）
```json
{
  "type": "error",
  "body": {
    "error_code": "ERR_001",
    "error_message": "Tool not found: web_search",
    "error_type": "validation_error|runtime_error|network_error",
    "stack_trace": "...",
    "retryable": true
  }
}
```

---

## 🗺️ 路由和寻址机制

### Agent 寻址规范

#### Agent ID 格式
```
{agent_type}_{instance_id}
示例: gateway_agent_001, coordinator_agent_001
```

#### 寻址模式

1. **直接寻址（Direct）**
```json
{
  "destination": {
    "agent_id": "coordinator_agent_001",
    "routing": "direct"
  }
}
```

2. **广播寻址（Broadcast）**
```json
{
  "destination": {
    "agent_type": "CapabilityAgent",
    "routing": "broadcast"
  }
}
```

3. **组播寻址（Multicast）**
```json
{
  "destination": {
    "group": "research_team",
    "routing": "multicast"
  }
}
```

4. **发布/订阅寻址（Pub/Sub）**
```json
{
  "destination": {
    "topic": "task_updates",
    "routing": "pubsub"
  }
}
```

### 路由算法

#### 智能路由器（基于 AutoGen GroupChat）
```python
class IntelligentRouter:
    """智能路由器"""
    
    def __init__(self):
        self.routing_table = {}
        self.group_chat = AIGroupChatManager()
        
    def route_message(self, message: dict) -> dict:
        """路由消息到合适的 Agent"""
        
        # 1. 解析消息
        routing_info = self.parse_message(message)
        
        # 2. 根据路由模式选择路由策略
        if routing_info["routing"] == "direct":
            return self.route_direct(routing_info)
        elif routing_info["routing"] == "broadcast":
            return self.route_broadcast(routing_info)
        elif routing_info["routing"] == "multicast":
            return self.route_multicast(routing_info)
        elif routing_info["routing"] == "pubsub":
            return self.route_pubsub(routing_info)
        else:
            # 智能路由（基于 GroupChat）
            return self.route_intelligent(message)
            
    def route_intelligent(self, message: dict) -> dict:
        """智能路由（基于 AutoGen GroupChat）"""
        
        # 1. 分析消息内容
        content_analysis = self.analyze_message_content(message)
        
        # 2. 确定目标 Agent
        target_agent = self.group_chat.select_next_speaker(
            message,
            content_analysis
        )
        
        # 3. 路由消息
        return {
            "destination": {
                "agent_id": target_agent,
                "routing": "direct"
            },
            "confidence": content_analysis["confidence"],
            "reasoning": content_analysis["reasoning"]
        }
        
    def analyze_message_content(self, message: dict) -> dict:
        """分析消息内容"""
        # 基于 LLM 的内容分析
        analysis = self.llm.analyze({
            "action": message["body"]["action"],
            "parameters": message["body"]["parameters"],
            "context": message.get("context", {})
        })
        
        return {
            "target_agent_type": analysis["agent_type"],
            "confidence": analysis["confidence"],
            "reasoning": analysis["reasoning"]
        }
```

---

## ⚠️ 错误处理和重试策略

### 错误分类

#### 1. 可恢复错误（Retryable）
- 网络超时
- 临时性错误
- 限流错误
- 网络不稳定

#### 2. 不可恢复错误（Non-Retryable）
- 参数错误
- 权限错误
- 严重错误
- 配置错误

### 重试策略

#### 1. 指数退避（Exponential Backoff）
```python
class ExponentialBackoffRetry:
    """指数退避重试策略"""
    
    def __init__(self, max_retries: int = 3, base_delay: int = 1000):
        self.max_retries = max_retries
        self.base_delay = base_delay
        
    async def retry_with_backoff(self, func, *args, **kwargs):
        """带指数退避的重试"""
        last_exception = None
        
        for attempt in range(self.max_retries):
            try:
                return await func(*args, **kwargs)
            except RetryableError as e:
                last_exception = e
                delay = self.base_delay * (2 ** attempt)
                
                if attempt < self.max_retries - 1:
                    await asyncio.sleep(delay)
                    
        raise last_exception
```

#### 2. 线性退避（Linear Backoff）
```python
class LinearBackoffRetry:
    """线性退避重试策略"""
    
    def __init__(self, max_retries: int = 3, base_delay: int = 1000):
        self.max_retries = max_retries
        self.base_delay = base_delay
        
    async def retry_with_backoff(self, func, *args, **kwargs):
        """带线性退避的重试"""
        last_exception = None
        
        for attempt in range(self.max_retries):
            try:
                return await func(*args, **kwargs)
            except RetryableError as e:
                last_exception = e
                delay = self.base_delay * (attempt + 1)
                
                if attempt < self.max_retries - 1:
                    await asyncio.sleep(delay)
                    
        raise last_exception
```

#### 3. 断路器模式（Circuit Breaker）
```python
class CircuitBreaker:
    """断路器模式"""
    
    def __init__(self, failure_threshold: int = 5, recovery_timeout: int = 60):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.failure_count = 0
        self.state = "closed"  # closed, open, half-open
        self.last_failure_time = None
        
    async def call(self, func, *args, **kwargs):
        """通过断路器调用函数"""
        
        # 检查断路器状态
        if self.state == "open":
            if self.is_recovery_timeout_expired():
                self.state = "half-open"
            else:
                raise CircuitBreakerOpenError("Circuit breaker is open")
                
        try:
            result = await func(*args, **kwargs)
            
            # 成功调用
            if self.state == "half-open":
                self.state = "closed"
                
            self.failure_count = 0
            return result
            
        except Exception as e:
            self.failure_count += 1
            self.last_failure_time = time.time()
            
            if self.failure_count >= self.failure_threshold:
                self.state = "open"
                
            raise e
            
    def is_recovery_timeout_expired(self) -> bool:
        """检查恢复超时是否过期"""
        if self.last_failure_time is None:
            return False
            
        return (time.time() - self.last_failure_time) >= self.recovery_timeout
```

---

## 🔐 通信安全和认证

### 认证机制

#### 1. API Key 认证
```python
class APIKeyAuthenticator:
    """API Key 认证器"""
    
    def __init__(self):
        self.api_keys = self.load_api_keys()
        
    def authenticate(self, api_key: str) -> bool:
        """验证 API Key"""
        return api_key in self.api_keys
        
    def generate_api_key(self, agent_id: str) -> str:
        """生成 API Key"""
        return f"agent_{agent_id}_{generate_secret_key()}"
```

#### 2. JWT（JSON Web Token）认证
```python
class JWTAuthenticator:
    """JWT 认证器"""
    
    def __init__(self, secret_key: str):
        self.secret_key = secret_key
        
    def generate_token(self, agent_id: str, expires_in: int = 3600) -> str:
        """生成 JWT Token"""
        payload = {
            "agent_id": agent_id,
            "exp": time.time() + expires_in,
            "iat": time.time()
        }
        
        return jwt.encode(payload, self.secret_key, algorithm="HS256")
        
    def verify_token(self, token: str) -> dict:
        """验证 JWT Token"""
        try:
            payload = jwt.decode(token, self.secret_key, algorithms=["HS256"])
            return payload
        except jwt.ExpiredSignatureError:
            raise AuthenticationError("Token expired")
        except jwt.InvalidTokenError:
            raise AuthenticationError("Invalid token")
```

#### 3. HMAC 签名
```python
class HMACSignature:
    """HMAC 签名器"""
    
    def __init__(self, secret_key: str):
        self.secret_key = secret_key.encode()
        
    def sign(self, message: dict) -> str:
        """对消息进行签名"""
        message_json = json.dumps(message, sort_keys=True)
        signature = hmac.new(
            self.secret_key,
            message_json.encode(),
            hashlib.sha256
        ).hexdigest()
        
        return signature
        
    def verify(self, message: dict, signature: str) -> bool:
        """验证签名"""
        expected_signature = self.sign(message)
        return hmac.compare_digest(signature, expected_signature)
```

### 加密机制

#### 1. AES 加密
```python
class AESEncryptor:
    """AES 加密器"""
    
    def __init__(self, key: str):
        self.key = key.encode()
        self.cipher = AES.new(self.key, AES.MODE_GCM)
        
    def encrypt(self, plaintext: str) -> dict:
        """加密文本"""
        ciphertext, tag = self.cipher.encrypt_and_digest(
            plaintext.encode()
        )
        
        return {
            "ciphertext": ciphertext.hex(),
            "nonce": self.cipher.nonce.hex(),
            "tag": tag.hex()
        }
        
    def decrypt(self, ciphertext: str, nonce: str, tag: str) -> str:
        """解密文本"""
        cipher = AES.new(
            self.key,
            AES.MODE_GCM,
            nonce=bytes.fromhex(nonce)
        )
        
        plaintext = cipher.decrypt_and_verify(
            bytes.fromhex(ciphertext),
            bytes.fromhex(tag)
        )
        
        return plaintext.decode()
```

---

## ⚡ 性能优化

### 1. 消息压缩
```python
class MessageCompressor:
    """消息压缩器"""
    
    def compress(self, message: dict) -> bytes:
        """压缩消息"""
        message_json = json.dumps(message)
        return gzip.compress(message_json.encode())
        
    def decompress(self, compressed_data: bytes) -> dict:
        """解压消息"""
        decompressed = gzip.decompress(compressed_data)
        message_json = decompressed.decode()
        return json.loads(message_json)
```

### 2. 消息批量处理
```python
class MessageBatcher:
    """消息批处理器"""
    
    def __init__(self, batch_size: int = 10, max_wait_time: int = 1000):
        self.batch_size = batch_size
        self.max_wait_time = max_wait_time
        self.batch = []
        self.lock = asyncio.Lock()
        
    async def add_message(self, message: dict):
        """添加消息到批次"""
        async with self.lock:
            self.batch.append(message)
            
            if len(self.batch) >= self.batch_size:
                await self.flush_batch()
                
    async def flush_batch(self):
        """刷新批次"""
        if not self.batch:
            return
            
        batch = self.batch
        self.batch = []
        
        # 处理批次消息
        await self.process_batch(batch)
```

### 3. 消息优先级队列
```python
class PriorityQueue:
    """优先级队列"""
    
    def __init__(self):
        self.queue = []
        self.lock = asyncio.Lock()
        
    async def enqueue(self, message: dict, priority: int):
        """将消息加入队列"""
        async with self.lock:
            heapq.heappush(self.queue, (-priority, time.time(), message))
            
    async def dequeue(self) -> dict:
        """从队列中取出消息"""
        async with self.lock:
            if self.queue:
                _, _, message = heapq.heappop(self.queue)
                return message
            return None
```

---

## 📊 通信协议总结

### 核心组件

| 组件 | 功能 | 技术栈 |
|-----|------|--------|
| **消息格式** | 标准化消息结构 | JSON Schema |
| **路由器** | 智能消息路由 | AutoGen GroupChat |
| **错误处理** | 重试和断路器 | asyncio + Retry policies |
| **认证** | 安全认证 | JWT + API Key |
| **加密** | 消息加密 | AES-GCM + HMAC |
| **压缩** | 消息压缩 | gzip |
| **批量处理** | 批量优化 | asyncio |
| **优先级** | 优先级队列 | heapq |

### 通信流程

```
Agent A
  ↓
[消息格式化] → [签名] → [加密] → [压缩]
  ↓
[路由器] → [选择目标 Agent]
  ↓
[发送] → [接收 Agent B]
  ↓
[解压] → [解密] → [验证签名] → [解析]
  ↓
Agent B
```

---

## ✅ 通信协议设计完成

### 已完成
- [x] 消息格式规范定义
- [x] 路由和寻址机制设计
- [x] 错误处理和重试策略
- [x] 通信安全和认证
- [x] 性能优化机制

### 🔄 下一步
- [ ] API 接口设计
- [ ] 部署架构设计

---

**完成时间**: 2026-02-03 15:55
**设计时间**: 10 分钟
**状态**: ✅ 通信协议设计完成
**下一任务**: API 接口设计
