# 部署架构设计

## 🎯 目标

设计完整的部署架构，包括：
- 容器化部署
- 服务架构
- 数据库设计
- 监控和日志
- 扩展性和高可用

---

## 🐳 容器化部署

### Docker 镜像策略

#### 基础镜像

```dockerfile
# 基础镜像（统一）
FROM python:3.11-slim AS base

WORKDIR /app

# 系统依赖
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    && rm -rf /var/lib/apt/lists/*

# Python 依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 应用代码
COPY . .

# 环境变量
ENV PYTHONUNBUFFERED=1
ENV PYTHONPATH=/app
```

#### 服务镜像

```dockerfile
# Gateway Agent 镜像
FROM base AS gateway_agent

ENV AGENT_TYPE=gateway
EXPOSE 8000
CMD ["python", "-m", "agents.gateway.main"]

# Coordinator Agent 镜像
FROM base AS coordinator_agent

ENV AGENT_TYPE=coordinator
EXPOSE 8001
CMD ["python", "-m", "agents.coordinator.main"]

# Execution Agent 镜像
FROM base AS execution_agent

ENV AGENT_TYPE=execution
EXPOSE 8002
CMD ["python", "-m", "agents.execution.main"]

# Memory Agent 镜像
FROM base AS memory_agent

ENV AGENT_TYPE=memory
EXPOSE 8003
CMD ["python", "-m", "agents.memory.main"]

# API Server 镜像
FROM base AS api_server

ENV SERVICE_TYPE=api
EXPOSE 8080
CMD ["python", "-m", "api.main"]

# WebSocket Server 镜像
FROM base AS websocket_server

ENV SERVICE_TYPE=websocket
EXPOSE 8081
CMD ["python", "-m", "websocket.main"]
```

### Docker Compose 配置

```yaml
version: '3.8'

services:
  # API Server
  api-server:
    build:
      context: .
      target: api_server
    image: registry.cn-hangzhou.aliyuncs.com/leesonwong/openclaw:api-server-latest
    ports:
      - "8080:8080"
    environment:
      - LOG_LEVEL=INFO
      - DATABASE_URL=postgresql://user:password@postgres:5432/openclaw
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - postgres
      - redis
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped
    networks:
      - openclaw-network

  # WebSocket Server
  websocket-server:
    build:
      context: .
      target: websocket_server
    image: registry.cn-hangzhou.aliyuncs.com/leesonwong/openclaw:websocket-server-latest
    ports:
      - "8081:8081"
    environment:
      - LOG_LEVEL=INFO
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - redis
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped
    networks:
      - openclaw-network

  # Gateway Agent
  gateway-agent:
    build:
      context: .
      target: gateway_agent
    image: registry.cn-hangzhou.aliyuncs.com/leesonwong/openclaw:gateway-agent-latest
    environment:
      - AGENT_ID=gateway_agent_001
      - LOG_LEVEL=INFO
      - DATABASE_URL=postgresql://user:password@postgres:5432/openclaw
      - REDIS_URL=redis://redis:6379/0
      - WEBSOCKET_URL=ws://websocket-server:8081
    depends_on:
      - postgres
      - redis
      - websocket-server
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped
    networks:
      - openclaw-network
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '1'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M

  # Coordinator Agent
  coordinator-agent:
    build:
      context: .
      target: coordinator_agent
    image: registry.cn-hangzhou.aliyuncs.com/leesonwong/openclaw:coordinator-agent-latest
    environment:
      - AGENT_ID=coordinator_agent_001
      - LOG_LEVEL=INFO
      - DATABASE_URL=postgresql://user:password@postgres:5432/openclaw
      - REDIS_URL=redis://redis:6379/0
      - WEBSOCKET_URL=ws://websocket-server:8081
    depends_on:
      - postgres
      - redis
      - websocket-server
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped
    networks:
      - openclaw-network
    deploy:
      replicas: 1
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '1'
          memory: 512M

  # Execution Agent
  execution-agent:
    build:
      context: .
      target: execution_agent
    image: registry.cn-hangzhou.aliyuncs.com/leesonwong/openclaw:execution-agent-latest
    environment:
      - AGENT_ID=execution_agent_001
      - LOG_LEVEL=INFO
      - DATABASE_URL=postgresql://user:password@postgres:5432/openclaw
      - REDIS_URL=redis://redis:6379/0
      - DOCKER_ENABLED=true
    depends_on:
      - postgres
      - redis
    volumes:
      - ./logs:/app/logs
      - /var/run/docker.sock:/var/run/docker.sock
    restart: unless-stopped
    networks:
      - openclaw-network
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '1'
          memory: 512M

  # Memory Agent
  memory-agent:
    build:
      context: .
      target: memory_agent
    image: registry.cn-hangzhou.aliyuncs.com/leesonwong/openclaw:memory-agent-latest
    environment:
      - AGENT_ID=memory_agent_001
      - LOG_LEVEL=INFO
      - DATABASE_URL=postgresql://user:password@postgres:5432/openclaw
      - REDIS_URL=redis://redis:6379/0
      - VECTOR_DB_URL=chroma
    depends_on:
      - postgres
      - redis
      - chromadb
    volumes:
      - ./logs:/app/logs
      - chroma-data:/app/chroma_data
    restart: unless-stopped
    networks:
      - openclaw-network
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '1.5'
          memory: 768M
        reservations:
          cpus: '0.5'
          memory: 256M

  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=openclaw
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    restart: unless-stopped
    networks:
      - openclaw-network

  # Redis Cache
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    restart: unless-stopped
    networks:
      - openclaw-network

  # ChromaDB (Vector Database)
  chromadb:
    image: chromadb/chroma:latest
    ports:
      - "8000:8000"
    volumes:
      - chroma-data:/chroma/chroma
    restart: unless-stopped
    networks:
      - openclaw-network

  # Nginx Load Balancer
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - api-server
      - websocket-server
    restart: unless-stopped
    networks:
      - openclaw-network

  # Prometheus Monitoring
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    restart: unless-stopped
    networks:
      - openclaw-network

  # Grafana Dashboard
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana
    restart: unless-stopped
    networks:
      - openclaw-network

  # ELK Stack (Logs)
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.8.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    restart: unless-stopped
    networks:
      - openclaw-network

  logstash:
    image: docker.elastic.co/logstash/logstash:8.8.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
      - ./logs:/var/log/app:ro
    depends_on:
      - elasticsearch
    restart: unless-stopped
    networks:
      - openclaw-network

  kibana:
    image: docker.elastic.co/kibana/kibana:8.8.0
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
    restart: unless-stopped
    networks:
      - openclaw-network

networks:
  openclaw-network:
    driver: bridge

volumes:
  postgres-data:
  redis-data:
  chroma-data:
  prometheus-data:
  grafana-data:
  elasticsearch-data:
```

---

## 🏗️ 服务架构

### 架构图

```
                    ┌─────────────┐
                    │   Nginx     │
                    │ Load Balancer│
                    └──────┬──────┘
                           │
              ┌────────────┴────────────┐
              │                         │
       ┌──────▼──────┐          ┌─────▼──────┐
       │ API Server  │          │   WebSocket │
       │  (REST)     │          │   Server    │
       └──────┬──────┘          └─────┬──────┘
              │                        │
              └────────────┬───────────┘
                           │
              ┌────────────┴────────────┐
              │                         │
       ┌──────▼──────┐          ┌─────▼──────┐
       │  Postgres   │          │    Redis    │
       │  Database   │          │    Cache    │
       └─────────────┘          └─────────────┘

              ┌─────────────────────────┐
              │  Agent Network         │
              └─────────┬───────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
   ┌───▼───┐     ┌───▼───┐     ┌───▼───┐
   │Gateway│     │Coordin│     │Executi│
   │ Agent │     │  ator │     │ on Agt│
   └───┬───┘     └───┬───┘     └───┬───┘
       │             │             │
       └─────────────┼─────────────┘
                     │
              ┌──────▼──────┐
              │  Memory     │
              │   Agent    │
              └──────┬──────┘
                     │
              ┌──────▼──────┐
              │  ChromaDB   │
              │ (Vector DB) │
              └─────────────┘

              ┌─────────────────────────┐
              │  Monitoring & Logging  │
              └─────────┬───────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
   ┌───▼─────┐    ┌───▼────┐    ┌───▼────┐
   │Prometheus│    │Grafana │    │  ELK   │
   │  Metrics │    │Dashboard│    │  Logs  │
   └──────────┘    └────────┘    └────────┘
```

### 服务依赖关系

```
API Server
  └─> Postgres Database
  └─> Redis Cache
  └─> Gateway Agent
  └─> Coordinator Agent
  └─> Memory Agent

WebSocket Server
  └─> Redis Cache
  └─> All Agents

Gateway Agent
  └─> Postgres Database
  └─> Redis Cache
  └─> WebSocket Server

Coordinator Agent
  └─> Postgres Database
  └─> Redis Cache
  └─> WebSocket Server
  └─> Execution Agent
  └─> Memory Agent

Execution Agent
  └─> Postgres Database
  └─> Redis Cache
  └─> Docker (for code execution)

Memory Agent
  └─> Postgres Database
  └─> Redis Cache
  └─> ChromaDB (Vector Database)
```

---

## 💾 数据库设计

### PostgreSQL Schema

#### agents 表
```sql
CREATE TABLE agents (
  id VARCHAR(255) PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  role VARCHAR(255) NOT NULL,
  goal TEXT NOT NULL,
  backstory TEXT,
  agent_type VARCHAR(255) NOT NULL,
  status VARCHAR(50) NOT NULL DEFAULT 'inactive',
  configuration JSONB,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  INDEX idx_agent_type (agent_type),
  INDEX idx_status (status)
);
```

#### tasks 表
```sql
CREATE TABLE tasks (
  id VARCHAR(255) PRIMARY KEY,
  description TEXT NOT NULL,
  task_type VARCHAR(50) NOT NULL,
  priority VARCHAR(50) NOT NULL DEFAULT 'normal',
  status VARCHAR(50) NOT NULL DEFAULT 'pending',
  progress INTEGER DEFAULT 0,
  assigned_to VARCHAR(255),
  deadline TIMESTAMP,
  dependencies JSONB,
  result JSONB,
  error TEXT,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  completed_at TIMESTAMP,
  FOREIGN KEY (assigned_to) REFERENCES agents(id),
  INDEX idx_status (status),
  INDEX idx_assigned_to (assigned_to),
  INDEX idx_created_at (created_at)
);
```

#### messages 表
```sql
CREATE TABLE messages (
  id VARCHAR(255) PRIMARY KEY,
  source_agent_id VARCHAR(255) NOT NULL,
  destination_agent_id VARCHAR(255) NOT NULL,
  message_type VARCHAR(50) NOT NULL,
  content TEXT NOT NULL,
  metadata JSONB,
  status VARCHAR(50) NOT NULL DEFAULT 'pending',
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  sent_at TIMESTAMP,
  received_at TIMESTAMP,
  FOREIGN KEY (source_agent_id) REFERENCES agents(id),
  FOREIGN KEY (destination_agent_id) REFERENCES agents(id),
  INDEX idx_source_agent (source_agent_id),
  INDEX idx_destination_agent (destination_agent_id),
  INDEX idx_created_at (created_at)
);
```

#### sessions 表
```sql
CREATE TABLE sessions (
  id VARCHAR(255) PRIMARY KEY,
  user_id VARCHAR(255),
  session_type VARCHAR(50) NOT NULL,
  status VARCHAR(50) NOT NULL DEFAULT 'active',
  metadata JSONB,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  closed_at TIMESTAMP,
  INDEX idx_user_id (user_id),
  INDEX idx_status (status),
  INDEX idx_created_at (created_at)
);
```

#### groupchats 表
```sql
CREATE TABLE groupchats (
  id VARCHAR(255) PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  agents JSONB NOT NULL,
  status VARCHAR(50) NOT NULL DEFAULT 'active',
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  INDEX idx_status (status)
);
```

#### memory 表
```sql
CREATE TABLE memory (
  id VARCHAR(255) PRIMARY KEY,
  memory_type VARCHAR(50) NOT NULL,
  content TEXT NOT NULL,
  metadata JSONB,
  embedding VECTOR(1536),
  importance FLOAT DEFAULT 0.5,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  expires_at TIMESTAMP,
  INDEX idx_memory_type (memory_type),
  INDEX idx_importance (importance),
  INDEX idx_created_at (created_at)
);
```

---

## 📊 监控和日志

### Prometheus 监控指标

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'api-server'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['api-server:8080']

  - job_name: 'gateway-agent'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['gateway-agent:8000']

  - job_name: 'coordinator-agent'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['coordinator-agent:8001']

  - job_name: 'execution-agent'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['execution-agent:8002']

  - job_name: 'memory-agent'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['memory-agent:8003']

  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres:5432']

  - job_name: 'redis'
    static_configs:
      - targets: ['redis:6379']
```

### Grafana Dashboard 配置

```json
{
  "dashboard": {
    "title": "AI Agent Group Dashboard",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total[1m])"
          }
        ]
      },
      {
        "title": "Agent Status",
        "targets": [
          {
            "expr": "agent_status{status=\"active\"}"
          }
        ]
      },
      {
        "title": "Task Completion Rate",
        "targets": [
          {
            "expr": "rate(tasks_completed_total[5m])"
          }
        ]
      },
      {
        "title": "Memory Usage",
        "targets": [
          {
            "expr": "process_resident_memory_bytes"
          }
        ]
      },
      {
        "title": "CPU Usage",
        "targets": [
          {
            "expr": "rate(process_cpu_seconds_total[1m])"
          }
        ]
      }
    ]
  }
}
```

### ELK Stack 日志配置

```conf
# logstash.conf
input {
  file {
    path => "/var/log/app/*.log"
    type => "app-log"
    start_position => "beginning"
  }
}

filter {
  if [type] == "app-log" {
    grok {
      match => {
        "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:log_message}"
      }
    }
    date {
      match => [ "timestamp", "ISO8601" ]
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "openclaw-logs-%{+YYYY.MM.dd}"
  }
}
```

---

## 🚀 扩展性和高可用

### 水平扩展策略

```yaml
# Kubernetes 配置示例（生产环境）
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gateway-agent
spec:
  replicas: 3
  selector:
    matchLabels:
      app: gateway-agent
  template:
    metadata:
      labels:
        app: gateway-agent
    spec:
      containers:
      - name: gateway-agent
        image: registry.cn-hangzhou.aliyuncs.com/leesonwong/openclaw:gateway-agent-latest
        resources:
          requests:
            cpu: "500m"
            memory: "256Mi"
          limits:
            cpu: "1000m"
            memory: "512Mi"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 10
          periodSeconds: 5
```

### 负载均衡策略

```nginx
# nginx.conf
upstream api_servers {
    least_conn;
    server api-server:8080 weight=1;
    server api-server-2:8080 weight=1;
    keepalive 32;
}

upstream websocket_servers {
    ip_hash;
    server websocket-server:8081;
    server websocket-server-2:8081;
}

server {
    listen 80;
    server_name api.aiagentgroup.com;

    location /api/ {
        proxy_pass http://api_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 健康检查
        health_check interval=30s fails=3 passes=2;
    }

    location /ws/ {
        proxy_pass http://websocket_servers;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 超时设置
        proxy_connect_timeout 7d;
        proxy_send_timeout 7d;
        proxy_read_timeout 7d;
    }
}
```

---

## 📊 部署架构总结

### 服务统计

| 服务 | 实例数 | 资源限制 | 职责 |
|-----|--------|---------|------|
| **API Server** | 2 | 1 CPU, 512M | REST API |
| **WebSocket Server** | 2 | 512M, 256M | 实时通信 |
| **Gateway Agent** | 2 | 1 CPU, 512M | 网关和路由 |
| **Coordinator Agent** | 1 | 2 CPU, 1G | 任务协调 |
| **Execution Agent** | 3 | 2 CPU, 1G | 代码执行 |
| **Memory Agent** | 2 | 1.5 CPU, 768M | 记忆管理 |
| **Postgres** | 1 | 4 CPU, 2G | 主数据库 |
| **Redis** | 1 | 1 CPU, 512M | 缓存 |
| **ChromaDB** | 1 | 2 CPU, 1G | 向量数据库 |
| **Nginx** | 2 | 1 CPU, 512M | 负载均衡 |
| **Prometheus** | 1 | 1 CPU, 512M | 监控 |
| **Grafana** | 1 | 1 CPU, 512M | Dashboard |
| **ELK** | 3 | 4 CPU, 2G | 日志 |

### 资源总计

| 资源类型 | 总量 |
|---------|-----|
| **CPU** | 约 18 核 |
| **内存** | 约 12G |
| **存储** | 约 50G |

### 高可用策略

- **负载均衡**: Nginx + 多实例
- **健康检查**: Liveness + Readiness probes
- **自动恢复**: Restart policy
- **数据备份**: 定期备份 + 主从复制
- **监控告警**: Prometheus + Grafana + ELK

---

## ✅ 部署架构设计完成

### 已完成
- [x] 容器化部署（Docker Compose + Kubernetes）
- [x] 服务架构设计
- [x] 数据库设计（PostgreSQL + Redis + ChromaDB）
- [x] 监控和日志系统
- [x] 扩展性和高可用策略

### 🔄 下一步
- [ ] 最终总结
- [ ] 提交所有文档

---

**完成时间**: 2026-02-03 16:15
**设计时间**: 10 分钟
**状态**: ✅ 部署架构设计完成
**下一任务**: 最终总结和提交
