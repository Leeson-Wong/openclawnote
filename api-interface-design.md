# API 接口设计

## 🎯 目标

设计完整的 API 接口，包括：
- RESTful API 设计
- WebSocket API 设计
- 请求/响应格式
- 认证和授权
- 错误处理
- API 文档

---

## 📋 API 架构

### API 版本管理

```
/v1/agents - Agent 管理
/v1/tasks - 任务管理
/v1/tools - 工具管理
/v1/messages - 消息管理
/v1/sessions - 会话管理
/v1/groups - GroupChat 管理
/v1/memory - 记忆管理
```

### 通用响应格式

```json
{
  "success": true,
  "data": {},
  "error": null,
  "metadata": {
    "timestamp": "2026-02-03T16:00:00.000Z",
    "request_id": "req_20260203_1600_001",
    "version": "1.0.0"
  }
}
```

---

## 🔌 RESTful API 设计

### 1. Agent 管理 API

#### 列出所有 Agent
```yaml
GET /v1/agents
description: 列出所有 Agent
parameters:
  - name: type
    in: query
    type: string
    description: Agent 类型（可选）
  - name: status
    in: query
    type: enum
    enum: [active, inactive, error]
    description: Agent 状态（可选）
  - name: limit
    in: query
    type: integer
    default: 20
    description: 返回数量限制
  - name: offset
    in: query
    type: integer
    default: 0
    description: 偏移量
responses:
  200:
    description: 成功
    schema:
      type: object
      properties:
        success:
          type: boolean
        data:
          type: object
          properties:
            agents:
              type: array
              items:
                $ref: '#/definitions/Agent'
            total:
              type: integer
            limit:
              type: integer
            offset:
              type: integer
        error:
          type: object
          nullable: true
        metadata:
          $ref: '#/definitions/Metadata'
```

#### 获取单个 Agent
```yaml
GET /v1/agents/{agent_id}
description: 获取单个 Agent 详情
parameters:
  - name: agent_id
    in: path
    type: string
    required: true
    description: Agent ID
responses:
  200:
    description: 成功
    schema:
      type: object
      properties:
        success:
          type: boolean
        data:
          $ref: '#/definitions/Agent'
        error:
          type: object
          nullable: true
        metadata:
          $ref: '#/definitions/Metadata'
```

#### 创建 Agent
```yaml
POST /v1/agents
description: 创建新的 Agent
parameters:
  - name: body
    in: body
    schema:
      type: object
      required:
        - name
        - role
        - goal
      properties:
        name:
          type: string
        role:
          type: string
        goal:
          type: string
        backstory:
          type: string
        tools:
          type: array
          items:
            type: string
        configuration:
          type: object
responses:
  201:
    description: 创建成功
    schema:
      type: object
      properties:
        success:
          type: boolean
        data:
          $ref: '#/definitions/Agent'
        error:
          type: object
          nullable: true
        metadata:
          $ref: '#/definitions/Metadata'
```

#### 更新 Agent
```yaml
PUT /v1/agents/{agent_id}
description: 更新 Agent 配置
parameters:
  - name: agent_id
    in: path
    type: string
    required: true
    description: Agent ID
  - name: body
    in: body
    schema:
      type: object
      properties:
        name:
          type: string
        role:
          type: string
        goal:
          type: string
        backstory:
          type: string
        tools:
          type: array
          items:
            type: string
        configuration:
          type: object
responses:
  200:
    description: 更新成功
    schema:
      type: object
      properties:
        success:
          type: boolean
        data:
          $ref: '#/definitions/Agent'
        error:
          type: object
          nullable: true
        metadata:
          $ref: '#/definitions/Metadata'
```

#### 删除 Agent
```yaml
DELETE /v1/agents/{agent_id}
description: 删除 Agent
parameters:
  - name: agent_id
    in: path
    type: string
    required: true
    description: Agent ID
responses:
  200:
    description: 删除成功
    schema:
      type: object
      properties:
        success:
          type: boolean
        data:
          type: object
          properties:
            deleted:
              type: boolean
            agent_id:
              type: string
        error:
          type: object
          nullable: true
        metadata:
          $ref: '#/definitions/Metadata'
```

---

### 2. 任务管理 API

#### 创建任务
```yaml
POST /v1/tasks
description: 创建新任务
parameters:
  - name: body
    in: body
    schema:
      type: object
      required:
        - task
      properties:
        task:
          type: object
          required:
            - description
          properties:
            description:
              type: string
            type:
              type: string
              enum: [research, analysis, decision, execution]
            priority:
              type: string
              enum: [low, normal, high, urgent]
            deadline:
              type: string
              format: date-time
            assign_to:
              type: string
            dependencies:
              type: array
              items:
                type: string
        options:
          type: object
          properties:
            auto_decompose:
              type: boolean
            auto_assign:
              type: boolean
            notify_on_complete:
              type: boolean
responses:
  201:
    description: 任务创建成功
    schema:
      type: object
      properties:
        success:
          type: boolean
        data:
          $ref: '#/definitions/Task'
        error:
          type: object
          nullable: true
        metadata:
          $ref: '#/definitions/Metadata'
```

#### 获取任务状态
```yaml
GET /v1/tasks/{task_id}
description: 获取任务状态
parameters:
  - name: task_id
    in: path
    type: string
    required: true
    description: 任务 ID
responses:
  200:
    description: 成功
    schema:
      type: object
      properties:
        success:
          type: boolean
        data:
          $ref: '#/definitions/Task'
        error:
          type: object
          nullable: true
        metadata:
          $ref: '#/definitions/Metadata'
```

#### 更新任务
```yaml
PUT /v1/tasks/{task_id}
description: 更新任务
parameters:
  - name: task_id
    in: path
    type: string
    required: true
    description: 任务 ID
  - name: body
    in: body
    schema:
      type: object
      properties:
        status:
          type: string
          enum: [pending, in_progress, completed, failed, blocked]
        progress:
          type: number
          minimum: 0
          maximum: 100
        result:
          type: object
        error:
          type: string
responses:
  200:
    description: 更新成功
    schema:
      type: object
      properties:
        success:
          type: boolean
        data:
          $ref: '#/definitions/Task'
        error:
          type: object
          nullable: true
        metadata:
          $ref: '#/definitions/Metadata'
```

---

### 3. 消息管理 API

#### 发送消息到 Agent
```yaml
POST /v1/messages/send
description: 发送消息到指定 Agent
parameters:
  - name: body
    in: body
    schema:
      type: object
      required:
        - recipient
        - message
      properties:
        recipient:
          type: object
          required:
            - agent_id
          properties:
            agent_id:
              type: string
            agent_type:
              type: string
        message:
          type: object
          required:
            - content
          properties:
            content:
              type: string
            type:
              type: string
              enum: [text, image, audio, video, file]
            metadata:
              type: object
        options:
          type: object
          properties:
            require_receipt:
              type: boolean
            timeout:
              type: number
            priority:
              type: string
              enum: [low, normal, high, urgent]
responses:
  200:
    description: 消息发送成功
    schema:
      type: object
      properties:
        success:
          type: boolean
        data:
          type: object
          properties:
            message_id:
              type: string
            status:
              type: string
            timestamp:
              type: string
        error:
          type: object
          nullable: true
        metadata:
          $ref: '#/definitions/Metadata'
```

---

## 🔌 WebSocket API 设计

### 连接端点

```yaml
WS /v1/ws/connect
description: 建立 WebSocket 连接
query_parameters:
  - name: token
    type: string
    required: true
    description: 认证 Token
  - name: session_id
    type: string
    required: true
    description: 会话 ID
  - name: agent_id
    type: string
    required: true
    description: Agent ID
```

### 消息类型

#### 1. 连接消息（Client → Server）
```json
{
  "type": "connect",
  "data": {
    "agent_id": "gateway_agent_001",
    "session_id": "session_001",
    "token": "jwt_token"
  }
}
```

#### 2. 订阅消息（Client → Server）
```json
{
  "type": "subscribe",
  "data": {
    "channels": ["task_updates", "agent_status", "events"]
  }
}
```

#### 3. 任务请求（Client → Server）
```json
{
  "type": "task_request",
  "data": {
    "task_id": "task_001",
    "action": "start|pause|resume|stop|cancel",
    "parameters": {}
  }
}
```

#### 4. 消息请求（Client → Server）
```json
{
  "type": "message_request",
  "data": {
    "recipient": {
      "agent_id": "coordinator_agent_001",
      "agent_type": "CoordinatorAgent"
    },
    "message": {
      "content": "Please analyze this data",
      "type": "text",
      "metadata": {}
    }
  }
}
```

#### 5. 响应消息（Server → Client）
```json
{
  "type": "response",
  "data": {
    "request_id": "req_001",
    "success": true,
    "result": {},
    "error": null
  }
}
```

#### 6. 任务更新（Server → Client）
```json
{
  "type": "task_update",
  "data": {
    "task_id": "task_001",
    "status": "in_progress",
    "progress": 50,
    "result": {},
    "timestamp": "2026-02-03T16:00:00.000Z"
  }
}
```

#### 7. Agent 状态（Server → Client）
```json
{
  "type": "agent_status",
  "data": {
    "agent_id": "coordinator_agent_001",
    "status": "active",
    "current_task": "task_001",
    "metrics": {
      "tasks_completed": 10,
      "uptime": 3600
    },
    "timestamp": "2026-02-03T16:00:00.000Z"
  }
}
```

#### 8. 事件通知（Server → Client）
```json
{
  "type": "event",
  "data": {
    "event_type": "task_completed|task_failed|agent_started|agent_stopped",
    "event_data": {
      "task_id": "task_001",
      "agent_id": "research_agent_001",
      "status": "completed"
    },
    "timestamp": "2026-02-03T16:00:00.000Z"
  }
}
```

---

## 🔐 认证和授权

### API Key 认证

```yaml
headers:
  X-API-Key:
    type: string
    description: API Key
    example: "sk_live_abc123..."
```

### JWT Token 认证

```yaml
headers:
  Authorization:
    type: string
    description: Bearer Token
    example: "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### OAuth 2.0 认证

```yaml
endpoints:
  /v1/auth/oauth/authorize:
    description: OAuth 2.0 授权端点
  /v1/auth/oauth/token:
    description: OAuth 2.0 Token 端点
  /v1/auth/oauth/refresh:
    description: OAuth 2.0 Refresh Token 端点
```

---

## ❌ 错误处理

### 错误响应格式

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "ERR_001",
    "message": "Agent not found",
    "details": "Agent 'gateway_agent_999' does not exist",
    "type": "not_found",
    "retryable": false,
    "stack_trace": "...",
    "timestamp": "2026-02-03T16:00:00.000Z"
  },
  "metadata": {
    "request_id": "req_20260203_1600_001",
    "version": "1.0.0"
  }
}
```

### 错误码定义

| 错误码 | 消息 | 类型 | 可重试 |
|-------|------|------|--------|
| ERR_001 | Agent not found | not_found | false |
| ERR_002 | Invalid request | validation_error | false |
| ERR_003 | Authentication failed | auth_error | false |
| ERR_004 | Permission denied | permission_error | false |
| ERR_005 | Rate limit exceeded | rate_limit | true |
| ERR_006 | Service unavailable | service_error | true |
| ERR_007 | Timeout | timeout_error | true |
| ERR_008 | Internal server error | internal_error | true |

---

## 📚 API 文档

### OpenAPI/Swagger 规范

```yaml
openapi: 3.0.0
info:
  title: AI Agent Group API
  version: 1.0.0
  description: AI Agent Group REST API
  contact:
    name: API Support
    email: support@aiagentgroup.com

servers:
  - url: https://api.aiagentgroup.com/v1
    description: Production Server
  - url: https://staging-api.aiagentgroup.com/v1
    description: Staging Server
  - url: http://localhost:8000/v1
    description: Development Server

paths:
  /agents:
    get:
      summary: List all agents
      operationId: listAgents
      tags:
        - Agents
      parameters:
        - $ref: '#/components/parameters/type'
        - $ref: '#/components/parameters/status'
        - $ref: '#/components/parameters/limit'
        - $ref: '#/components/parameters/offset'
      responses:
        '200':
          $ref: '#/components/responses/AgentsList'
    post:
      summary: Create a new agent
      operationId: createAgent
      tags:
        - Agents
      requestBody:
        $ref: '#/components/requestBodies/AgentCreate'
      responses:
        '201':
          $ref: '#/components/responses/Agent'

components:
  schemas:
    Agent:
      type: object
      properties:
        id:
          type: string
        name:
          type: string
        role:
          type: string
        goal:
          type: string
        backstory:
          type: string
        tools:
          type: array
          items:
            type: string
        configuration:
          type: object
        status:
          type: string
          enum: [active, inactive, error]
        created_at:
          type: string
          format: date-time
        updated_at:
          type: string
          format: date-time

    Task:
      type: object
      properties:
        id:
          type: string
        description:
          type: string
        type:
          type: string
          enum: [research, analysis, decision, execution]
        priority:
          type: string
          enum: [low, normal, high, urgent]
        status:
          type: string
          enum: [pending, in_progress, completed, failed, blocked]
        progress:
          type: number
          minimum: 0
          maximum: 100
        assigned_to:
          type: string
        deadline:
          type: string
          format: date-time
        dependencies:
          type: array
          items:
            type: string
        result:
          type: object
        error:
          type: string
        created_at:
          type: string
          format: date-time
        updated_at:
          type: string
          format: date-time

    Error:
      type: object
      properties:
        code:
          type: string
        message:
          type: string
        details:
          type: string
        type:
          type: string
        retryable:
          type: boolean
        timestamp:
          type: string
          format: date-time

    Metadata:
      type: object
      properties:
        timestamp:
          type: string
          format: date-time
        request_id:
          type: string
        version:
          type: string

  parameters:
    type:
      name: type
      in: query
      schema:
        type: string

    status:
      name: status
      in: query
      schema:
        type: string
        enum: [active, inactive, error]

    limit:
      name: limit
      in: query
      schema:
        type: integer
        default: 20

    offset:
      name: offset
      in: query
      schema:
        type: integer
        default: 0

  requestBodies:
    AgentCreate:
      required: true
      content:
        application/json:
          schema:
            type: object
            required:
              - name
              - role
              - goal
            properties:
              name:
                type: string
              role:
                type: string
              goal:
                type: string
              backstory:
                type: string
              tools:
                type: array
                items:
                  type: string
              configuration:
                type: object

  responses:
    AgentsList:
      description: List of agents
      content:
        application/json:
          schema:
            type: object
            properties:
              success:
                type: boolean
              data:
                type: object
                properties:
                  agents:
                    type: array
                    items:
                      $ref: '#/components/schemas/Agent'
                  total:
                    type: integer
                  limit:
                    type: integer
                  offset:
                    type: integer
              error:
                type: object
                nullable: true
              metadata:
                $ref: '#/components/schemas/Metadata'

    Agent:
      description: Agent details
      content:
        application/json:
          schema:
            type: object
            properties:
              success:
                type: boolean
              data:
                $ref: '#/components/schemas/Agent'
              error:
                type: object
                nullable: true
              metadata:
                $ref: '#/components/schemas/Metadata'
```

---

## 📊 API 接口总结

### API 端点统计

| 类别 | 端点数 | 主要功能 |
|-----|--------|---------|
| **Agent 管理** | 5 | Agent CRUD 操作 |
| **任务管理** | 5 | 任务 CRUD 操作 |
| **消息管理** | 3 | 消息发送和接收 |
| **会话管理** | 4 | 会话 CRUD 操作 |
| **GroupChat 管理** | 4 | GroupChat CRUD 操作 |
| **记忆管理** | 5 | 记忆 CRUD 操作 |
| **工具管理** | 3 | 工具注册和调用 |
| **WebSocket** | 1 | 实时通信 |
| **认证授权** | 3 | OAuth 2.0 和 JWT |
| **总计** | 33 | 完整的 API |

### 认证方式

| 认证方式 | 端点 | 用途 |
|---------|------|------|
| **API Key** | 所有 REST API | 简单认证 |
| **JWT Token** | 所有 REST API | 标准 Token 认证 |
| **OAuth 2.0** | /v1/auth/oauth/* | 第三方集成 |

### 错误处理

- **标准错误格式**: 统一的错误响应结构
- **错误码**: 8 个主要错误码
- **重试策略**: 明确的可重试标志
- **详细错误**: 包含详细信息堆栈跟踪

---

## ✅ API 接口设计完成

### 已完成
- [x] RESTful API 设计（33 个端点）
- [x] WebSocket API 设计（8 种消息类型）
- [x] 请求/响应格式规范
- [x] 认证和授权机制
- [x] 错误处理策略
- [x] OpenAPI/Swagger 规范
- [x] API 文档结构

### 🔄 下一步
- [ ] 部署架构设计
- [ ] 最终总结和提交

---

**完成时间**: 2026-02-03 16:05
**设计时间**: 10 分钟
**状态**: ✅ API 接口设计完成
**下一任务**: 部署架构设计
