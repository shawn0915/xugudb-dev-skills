# Docker 与容器化部署

## 概述

XuguDB 安装包中包含官方 Docker 文件，支持基于 Debian 和 RedHat 的容器化部署。

安装包中的 Docker 目录结构：
```
Docker/
├── Dockerfile.Debian      # 基于 Debian 的镜像构建文件
├── Dockerfile.RedHat      # 基于 RedHat 的镜像构建文件
└── docker-image-build.md  # 官方构建说明
```

## Docker 镜像构建

### 前置条件

- 安装 Docker Engine 或 Docker Desktop
- XuguDB 安装包已解压

### 构建命令

```bash
# 进入解压后的目录
cd XuguDB/Docker/

# 基于 Debian 构建（推荐，镜像更小）
docker build -f Dockerfile.Debian -t xugudb:12.9 ..

# 基于 RedHat 构建（兼容性更好）
docker build -f Dockerfile.RedHat -t xugudb:12.9 ..

# 带版本标签
docker build -f Dockerfile.Debian -t xugudb:12.9.9-debian ..
```

### 验证镜像

```bash
docker images | grep xugudb
```

## Docker 运行

### 快速启动（开发/测试）

```bash
docker run -d --name xugudb \
    -p 5138:5138 \
    xugudb:12.9
```

### 生产环境运行

```bash
docker run -d --name xugudb \
    -p 5138:5138 \
    -v /data/xugudb/XHOME:/opt/xugudb/XHOME \
    -v /data/xugudb/XGLOG:/opt/xugudb/XGLOG \
    -v /data/xugudb/BACKUP:/opt/xugudb/BACKUP \
    -v /data/xugudb/SETUP:/opt/xugudb/SETUP \
    --memory=8g \
    --cpus=4 \
    --ulimit nofile=65536:65536 \
    --restart=unless-stopped \
    --network=host \
    xugudb:12.9
```

### 关键参数说明

| 参数 | 说明 |
|------|------|
| `-p 5138:5138` | 映射数据库端口 |
| `-v .../XHOME` | 数据文件持久化（必须） |
| `-v .../XGLOG` | 日志文件持久化（建议） |
| `-v .../BACKUP` | 备份文件持久化 |
| `-v .../SETUP` | 配置文件挂载（可覆盖默认配置） |
| `--memory=8g` | 内存限制 |
| `--cpus=4` | CPU 限制 |
| `--restart=unless-stopped` | 自动重启策略 |
| `--network=host` | 使用宿主网络（性能更好） |
| `--ulimit nofile=65536` | 文件描述符限制 |

### 连接容器内数据库

```bash
# 从宿主机连接
xgconsole -s nssl -h 127.0.0.1 -P 5138 -d SYSTEM -u SYSDBA -p SYSDBA

# 进入容器执行
docker exec -it xugudb xgconsole -s nssl -h 127.0.0.1 -P 5138 -d SYSTEM -u SYSDBA -p SYSDBA
```

### 查看日志

```bash
# 容器日志
docker logs xugudb

# 数据库日志（持久化后）
cat /data/xugudb/XGLOG/ERROR.LOG
```

## Docker Compose

### 单机部署

```yaml
version: '3.8'

services:
  xugudb:
    image: xugudb:12.9
    container_name: xugudb
    ports:
      - "5138:5138"
    volumes:
      - xugudb_data:/opt/xugudb/XHOME
      - xugudb_log:/opt/xugudb/XGLOG
      - xugudb_backup:/opt/xugudb/BACKUP
    deploy:
      resources:
        limits:
          memory: 8G
          cpus: '4'
        reservations:
          memory: 4G
          cpus: '2'
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "xgconsole", "-s", "nssl", "-h", "127.0.0.1", "-P", "5138", "-d", "SYSTEM", "-u", "SYSDBA", "-p", "SYSDBA", "-e", "SELECT 1 FROM DUAL;"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  xugudb_data:
    driver: local
  xugudb_log:
    driver: local
  xugudb_backup:
    driver: local
```

### 启动

```bash
docker-compose up -d
docker-compose logs -f xugudb
docker-compose down
```

### 应用 + 数据库联合部署

```yaml
version: '3.8'

services:
  xugudb:
    image: xugudb:12.9
    container_name: xugudb
    ports:
      - "5138:5138"
    volumes:
      - xugudb_data:/opt/xugudb/XHOME
    restart: unless-stopped

  app:
    image: myapp:latest
    depends_on:
      - xugudb
    environment:
      - DB_HOST=xugudb
      - DB_PORT=5138
      - DB_NAME=SYSTEM
      - DB_USER=SYSDBA
      - DB_PASS=SYSDBA
    ports:
      - "8080:8080"

volumes:
  xugudb_data:
```

## Kubernetes 部署

### StatefulSet（推荐用于数据库）

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: xugudb
  labels:
    app: xugudb
spec:
  serviceName: xugudb
  replicas: 1
  selector:
    matchLabels:
      app: xugudb
  template:
    metadata:
      labels:
        app: xugudb
    spec:
      containers:
      - name: xugudb
        image: xugudb:12.9
        ports:
        - containerPort: 5138
          name: xugudb
        resources:
          requests:
            memory: "4Gi"
            cpu: "2"
          limits:
            memory: "8Gi"
            cpu: "4"
        volumeMounts:
        - name: data
          mountPath: /opt/xugudb/XHOME
        - name: log
          mountPath: /opt/xugudb/XGLOG
        livenessProbe:
          exec:
            command:
            - xgconsole
            - -s
            - nssl
            - -h
            - "127.0.0.1"
            - -P
            - "5138"
            - -d
            - SYSTEM
            - -u
            - SYSDBA
            - -p
            - SYSDBA
            - -e
            - "SELECT 1 FROM DUAL;"
          initialDelaySeconds: 30
          periodSeconds: 30
        readinessProbe:
          tcpSocket:
            port: 5138
          initialDelaySeconds: 10
          periodSeconds: 10
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: standard
      resources:
        requests:
          storage: 100Gi
  - metadata:
      name: log
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: standard
      resources:
        requests:
          storage: 20Gi
```

### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: xugudb
spec:
  type: ClusterIP
  ports:
  - port: 5138
    targetPort: 5138
    protocol: TCP
  selector:
    app: xugudb
---
# NodePort（集群外访问）
apiVersion: v1
kind: Service
metadata:
  name: xugudb-nodeport
spec:
  type: NodePort
  ports:
  - port: 5138
    targetPort: 5138
    nodePort: 30138
  selector:
    app: xugudb
```

### ConfigMap（配置注入）

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: xugudb-config
data:
  xugu.ini: |
    listen_port = 5138;
    task_thd_num = 8;
    max_parallel = 2;
    pass_mode = 3;
```

### 部署命令

```bash
kubectl apply -f xugudb-statefulset.yaml
kubectl apply -f xugudb-service.yaml

# 查看状态
kubectl get pods -l app=xugudb
kubectl logs xugudb-0

# 连接
kubectl port-forward svc/xugudb 5138:5138
xgconsole -s nssl -h 127.0.0.1 -P 5138 -d SYSTEM -u SYSDBA -p SYSDBA
```

## 容器化最佳实践

### 数据持久化

- **必须**持久化 XHOME 目录（数据文件）
- 建议持久化 XGLOG（日志）和 BACKUP（备份）
- 使用命名卷或 hostPath，避免匿名卷

### 资源限制

- 设置内存和 CPU 限制，避免资源争抢
- `task_thd_num` 应匹配容器 CPU 限制（不是宿主机 CPU 数）

### 网络

- 开发环境使用端口映射 `-p 5138:5138`
- 生产环境考虑 `--network=host` 减少网络开销
- K8s 中使用 Service 做服务发现

### 健康检查

- 配置 liveness/readiness probe
- 使用 `xgconsole -e "SELECT 1 FROM DUAL;"` 做深度检查
- 使用 TCP 端口检查做快速就绪检查

### 安全

- 不在镜像中硬编码密码
- 使用 K8s Secret 或 Docker Secret 管理敏感信息
- 生产环境启用 SSL（`-s ssl`）

### 集群容器化

- 分布式版可以通过多容器模拟，但需要注意：
  - 容器间需要 UDP 通信（cluster.ini 配置）
  - 使用 `--network=host` 或自定义 overlay 网络
  - 每个容器的 `MY_NID` 必须唯一
  - 建议在 K8s 中使用 StatefulSet + Headless Service
