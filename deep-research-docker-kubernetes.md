# 深度研究 - Docker/Kubernetes 容器化

## 🎯 研究目标

深入研究 Docker 和 Kubernetes 容器化最佳实践：
1. Docker 容器化最佳实践
2. Kubernetes 部署模式
3. Service Mesh (Istio)
4. 高可用架构

---

## 🐳 Docker 容器化最佳实践

### 1. Dockerfile 最佳实践

#### 多阶段构建

```dockerfile
# 构建阶段
FROM python:3.11-slim AS builder

WORKDIR /app

# 安装构建依赖
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    && rm -rf /var/lib/apt/lists/*

# 复制依赖文件
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制源代码
COPY . .

# 编译类型检查文件（如果需要）
RUN pip install mypy && mypy .

# 运行阶段
FROM python:3.11-slim AS runtime

WORKDIR /app

# 只复制必要的文件
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY --from=builder /app .

# 创建非 root 用户
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

# 设置环境变量
ENV PYTHONUNBUFFERED=1
ENV PYTHONPATH=/app

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

# 暴露端口
EXPOSE 8000

# 启动命令
CMD ["python", "-m", "main"]
```

#### 最小化镜像大小

```dockerfile
# 使用 alpine 版本（更小）
FROM python:3.11-alpine AS base

# 只安装必需的依赖
RUN apk add --no-cache --virtual .build-deps gcc musl-dev && \
    pip install --no-cache-dir --no-deps -U pip && \
    apk del .build-deps

# 使用 .dockerignore 减少构建上下文
# .dockerignore
.git
.gitignore
__pycache__
*.pyc
*.md
README.md
docs/
tests/
```

### 2. Docker Compose 最佳实践

#### 生产级 Docker Compose 配置

```yaml
version: '3.8'

services:
  # API Server
  api-server:
    build:
      context: .
      dockerfile: Dockerfile
      target: runtime
    image: registry.cn-hangzhou.aliyuncs.com/leesonwong/openclaw:api-server-v1.0.0
    ports:
      - "8080:8080"
    environment:
      - LOG_LEVEL=${LOG_LEVEL:-INFO}
      - DATABASE_URL=postgresql://${POSTGRES_USER:-user}:${POSTGRES_PASSWORD:-password}@postgres:5432/${POSTGRES_DB:-openclaw}
      - REDIS_URL=redis://:password@redis:6379/0
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - ./logs:/app/logs:rw
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 10s
    networks:
      - openclaw-network
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine
    container_name: openclaw-postgres
    environment:
      - POSTGRES_USER=${POSTGRES_USER:-user}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD:-password}
      - POSTGRES_DB=${POSTGRES_DB:-openclaw}
      - PGDATA=/var/lib/postgresql/data/pgdata
    volumes:
      - postgres-data:/var/lib/postgresql/data/pgdata:rw
      - ./init-scripts:/docker-entrypoint-initdb.d:ro
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-user} -d ${POSTGRES_DB:-openclaw}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    restart: unless-stopped
    networks:
      - openclaw-network
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '1.0'
          memory: 1G

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: openclaw-redis
    command: redis-server --requirepass ${REDIS_PASSWORD:-password} --appendonly yes
    environment:
      - REDIS_PASSWORD=${REDIS_PASSWORD:-password}
    volumes:
      - redis-data:/data:rw
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "${REDIS_PASSWORD:-password}", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
    restart: unless-stopped
    networks:
      - openclaw-network
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M

networks:
  openclaw-network:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.20.0.0/16

volumes:
  postgres-data:
    driver: local
  redis-data:
    driver: local
```

### 3. 安全最佳实践

#### 安全扫描

```bash
# 使用 Trivy 进行镜像安全扫描
trivy image registry.cn-hangzhou.aliyuncs.com/leesonwong/openclaw:api-server-v1.0.0

# 使用 Grype 进行漏洞扫描
grype registry.cn-hangzhou.aliyuncs.com/leesonwong/openclaw:api-server-v1.0.0

# 集成到 CI/CD
```

```yaml
# .github/workflows/security-scan.yml
name: Security Scan

on: [push, pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Build Docker image
        run: docker build -t openclaw:latest .
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'openclaw:latest'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy results to GitHub Security tab
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
```

---

## ☸️ Kubernetes 部署模式

### 1. Kubernetes 部署清单

#### Deployment 配置

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: openclaw-api-server
  labels:
    app: openclaw
    component: api-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: openclaw
      component: api-server
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
  template:
    metadata:
      labels:
        app: openclaw
        component: api-server
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      serviceAccountName: openclaw-sa
      containers:
      - name: api-server
        image: registry.cn-hangzhou.aliyuncs.com/leesonwong/openclaw:api-server-v1.0.0
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: http
          protocol: TCP
        env:
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: openclaw-config
              key: log_level
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: openclaw-secrets
              key: database_url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: openclaw-secrets
              key: redis_url
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
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 3
          failureThreshold: 3
          successThreshold: 1
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
          successThreshold: 1
        volumeMounts:
        - name: logs
          mountPath: /app/logs
        securityContext:
          runAsNonRoot: true
          runAsUser: 1000
          readOnlyRootFilesystem: true
          allowPrivilegeEscalation: false
      volumes:
      - name: logs
        emptyDir: {}
      terminationGracePeriodSeconds: 30
```

#### Service 配置

```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: openclaw-api-server
  labels:
    app: openclaw
    component: api-server
spec:
  type: LoadBalancer
  selector:
    app: openclaw
    component: api-server
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 8080
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
```

#### ConfigMap 配置

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: openclaw-config
data:
  log_level: "INFO"
  environment: "production"
  max_connections: "1000"
  timeout: "30"
```

#### Secret 配置

```yaml
# k8s/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: openclaw-secrets
type: Opaque
data:
  database_url: cG9zdGdyZXNxbDovL3VzZXI6cGFzc3dvcmRAcG9zdGdyZXM6NTQzMi9vcGVuY2xhdwo=
  redis_url: cmVkaXM6Oi86cGFzc3dvcmRAcmVkaXM6NjM3OS8wCg==
  api_key: eW91ci1hcGkta2V5LWhlcmUK
```

### 2. Horizontal Pod Autoscaler (HPA)

```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: openclaw-api-server-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: openclaw-api-server
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  - type: Pods
    pods:
      metric:
        name: packets-per-second
      target:
        type: AverageValue
        averageValue: 1k
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
      - type: Pods
        value: 2
        periodSeconds: 60
      selectPolicy: Min
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
      - type: Pods
        value: 2
        periodSeconds: 60
      selectPolicy: Max
```

### 3. Pod Disruption Budget (PDB)

```yaml
# k8s/pdb.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: openclaw-api-server-pdb
spec:
  minAvailable: 2
  maxUnavailable: 1
  selector:
    matchLabels:
      app: openclaw
      component: api-server
```

---

## 🌐 Service Mesh (Istio)

### 1. Istio Gateway 配置

```yaml
# istio/gateway.yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: openclaw-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "api.openclaw.com"
    tls:
      mode: SIMPLE
      credentialName: openclaw-credential
```

### 2. Virtual Service 配置

```yaml
# istio/virtualservice.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: openclaw-vs
spec:
  hosts:
  - "api.openclaw.com"
  gateways:
  - openclaw-gateway
  http:
  - match:
    - uri:
        prefix: "/api/"
    route:
    - destination:
        host: openclaw-api-server
        subset: v1
      weight: 90
    - destination:
        host: openclaw-api-server
        subset: v2
      weight: 10
    timeout: 30s
    retries:
      attempts: 3
      perTryTimeout: 10s
      retryOn: 5xx,connect-failure,refused-stream
    fault:
      delay:
        percentage:
          value: 10
        fixedDelay: 5s
```

### 3. Destination Rule 配置

```yaml
# istio/destinationrule.yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: openclaw-destination
spec:
  host: openclaw-api-server
  trafficPolicy:
    loadBalancer:
      simple: ROUND_ROBIN
    connectionPool:
      tcp:
        maxConnections: 100
        connectTimeout: 30s
        tcpKeepalive:
          time: 7200s
          interval: 75s
      http:
        http2MaxRequests: 100
        http1MaxPendingRequests: 50
        idleTimeout: 300s
        h2UpgradePolicy: UPGRADE
    outlierDetection:
      consecutiveErrors: 3
      interval: 10s
      baseEjectionTime: 30s
      maxEjectionPercent: 30
      minHealthPercent: 50
    tls:
      mode: ISTIO_MUTUAL
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

---

## 🚀 高可用架构

### 1. 多区域部署

```yaml
# k8s/multi-region-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: openclaw-api-server
  annotations:
    topology.kubernetes.io/zone: "us-west-1a"
spec:
  replicas: 3
  template:
    metadata:
      annotations:
        topology.kubernetes.io/zone: "us-west-1a"
        topology.kubernetes.io/region: "us-west-1"
```

### 2. 故障转移和恢复

```yaml
# k8s/probes.yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 3
  failureThreshold: 3
  successThreshold: 1
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3
  successThreshold: 1
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  initialDelaySeconds: 0
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 30
  successThreshold: 1
```

---

## 📊 研究总结

### Docker 容器化最佳实践
1. **多阶段构建** - 最小化镜像大小
2. **非 root 用户** - 提升安全性
3. **健康检查** - 自动故障恢复
4. **资源限制** - 防止资源耗尽
5. **安全扫描** - CI/CD 集成

### Kubernetes 部署最佳实践
1. **水平扩展** - HPA 自动扩展
2. **滚动更新** - 零停机部署
3. **Pod Disruption Budget** - 保证可用性
4. **健康检查** - 自动重启不健康的 Pod
5. **资源配额** - 公平分配资源

### Service Mesh (Istio) 最佳实践
1. **流量管理** - 灰度发布
2. **故障注入** - 测试容错性
3. **流量拆分** - A/B 测试
4. **连接池** - 性能优化
5. **熔断机制** - 故障隔离

---

## 🚀 应用到我们的设计

### 容器化我们的 Agent

```dockerfile
# Agent Dockerfile
FROM python:3.11-slim AS base

WORKDIR /app

# 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制代码
COPY . .

# 运行时镜像
FROM python:3.11-slim AS runtime

WORKDIR /app

# 复制依赖和代码
COPY --from=base /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY --from=base /app .

# 非 root 用户
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

# 环境变量
ENV PYTHONUNBUFFERED=1
ENV AGENT_TYPE=gateway_agent

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

# 暴露端口
EXPOSE 8000

# 启动 Agent
CMD ["python", "-m", "agents.gateway.main"]
```

### Kubernetes 部署 Agent

```yaml
# Agent Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gateway-agent
spec:
  replicas: 2
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
        image: registry.cn-hangzhou.aliyuncs.com/leesonwong/openclaw:gateway-agent-v1.0.0
        env:
        - name: AGENT_ID
          value: "gateway_agent_001"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: openclaw-secrets
              key: database_url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: openclaw-secrets
              key: redis_url
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
```

---

**研究状态**: ✅ Docker/Kubernetes 深度研究完成（15 分钟）
**研究时间**: 2026-02-03 16:55
**下一研究**: 工具系统实现细节细化
