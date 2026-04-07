---
name: 虚谷数据库安装部署
name_for_command: xugudb-deployment
description: |
  XuguDB 完整安装部署指南：软硬件环境需求、系统配置检查、License 管理、
  四种版本部署（标准版/企业版/分布式版/安全版）、Docker 容器化部署、
  启动参数、启停管理、集群高可用部署方案（数据中心/同城双活/同城灾备/两地三中心）。
  适用于生产环境部署规划和实施。
tags: xugudb, deployment, installation, docker, container, cluster, ha, license
---

# 虚谷数据库安装部署

## 部署概览

XuguDB 支持从单机到多节点集群的灵活部署，并提供 Docker 容器化方案。

### 版本与部署模式

| 版本 | 节点数 | 定位 | 部署复杂度 |
|------|--------|------|-----------|
| **标准版** | 单机 | 中小规模业务 | 低 |
| **企业版** | 2 节点 | 双机热备 | 中 |
| **分布式版** | 3~N 节点 | 大规模数据/高并发 | 高 |
| **安全版** | 1~2 节点 | 涉密系统 | 中 |
| **个人版** | 单机/多机 | 免费试用（360天） | 低 |

### 安装包内容

XuguDB 安装包解压后包含：

```
XuguDB/
├── BIN/
│   ├── xugu_linux_x86_64_*    # 数据库服务程序
│   ├── xgconsole              # 命令行客户端
│   ├── xugu_init.sql          # 初始化脚本
│   └── startdb.sh             # 启动脚本
├── SETUP/
│   ├── xugu.ini               # 数据库配置
│   ├── mount.ini              # 存储挂载配置
│   ├── types.ini              # 类型定义
│   └── trust.ini              # 信任配置
└── Docker/                    # Docker 部署文件
    ├── Dockerfile.Debian      # Debian 镜像
    ├── Dockerfile.RedHat      # RedHat 镜像
    └── docker-image-build.md  # 构建说明
```

> 详细参考：[软硬件环境需求](references/requirements.md)

## 部署流程

```
1. 环境准备 → 2. 获取安装包 → 3. 校验完整性 → 4. 申请License →
5. 创建用户 → 6. 解压部署 → 7. 配置参数 → 8. 启动数据库 → 9. 验证
```

> 详细参考：[部署流程](references/deployment-guide.md)

## 标准版部署（单机）

```bash
# 1. 创建数据库用户
useradd xugu && passwd xugu

# 2. 解压安装包
su xugu
tar -zxvf XuguDB-12.8_*-linux-x64.tar.gz

# 3. 初始化 SETUP（首次部署）
cd XuguDB/BIN
./xugu_linux_x86_64_* --init-setup

# 4. 配置 xugu.ini
vi ../SETUP/xugu.ini
# listen_port = 5138;
# task_thd_num = 16;  # CPU核数*2

# 5. 配置数据目录（mount.ini + datafile.ini）
vi ../SETUP/mount.ini
vi ../SETUP/datafile.ini

# 6. 启动
./xugu_linux_x86_64_* --child  # 前台启动
# 或
./xugu_linux_x86_64_* --server # 后台启动
```

## 企业版部署（双节点主备）

企业版需要 **2 台服务器**，每台部署一个节点，通过 cluster.ini 配置主备关系。

```bash
# 每台机器都需要：
# 1. 创建用户、解压安装包（同标准版）
# 2. 生成 cluster.ini 模板
./xugu_linux_x86_64_* --init-cluster

# 3. 配置 cluster.ini（关键参数）
# MASTER_GRPS = 1
# MY_NID = <当前节点ID>
# NID = "IP1:PORT1,IP2:PORT2"
# ROLE = M（主控）或 SQW（存储+查询+工作）

# 4. 两台机器同时启动
```

## 分布式版部署（3~N 节点）

分布式版至少需要 **3 台服务器**，推荐 4 台以上。

> 详细参考：[各版本部署指南](references/edition-deployment.md)

## Docker 容器化部署

XuguDB 官方提供 Docker 部署支持，安装包中包含 Dockerfile。

### 构建镜像

```bash
# 解压安装包后进入 Docker 目录
cd Docker/

# 基于 Debian 构建
docker build -f Dockerfile.Debian -t xugudb:12.9 .

# 基于 RedHat 构建
docker build -f Dockerfile.RedHat -t xugudb:12.9 .
```

### 运行容器

```bash
# 基本运行
docker run -d --name xugudb \
    -p 5138:5138 \
    -v /data/xugudb:/xugudb/data \
    xugudb:12.9

# 生产环境推荐（限制资源 + 数据持久化）
docker run -d --name xugudb \
    -p 5138:5138 \
    -v /data/xugudb/XHOME:/xugudb/XHOME \
    -v /data/xugudb/XGLOG:/xugudb/XGLOG \
    -v /data/xugudb/BACKUP:/xugudb/BACKUP \
    --memory=8g \
    --cpus=4 \
    --restart=unless-stopped \
    xugudb:12.9
```

### Docker Compose 示例

```yaml
version: '3.8'
services:
  xugudb:
    image: xugudb:12.9
    container_name: xugudb
    ports:
      - "5138:5138"
    volumes:
      - xugudb_data:/xugudb/XHOME
      - xugudb_log:/xugudb/XGLOG
      - xugudb_backup:/xugudb/BACKUP
    environment:
      - XUGU_PASS=SYSDBA
    deploy:
      resources:
        limits:
          memory: 8G
          cpus: '4'
    restart: unless-stopped

volumes:
  xugudb_data:
  xugudb_log:
  xugudb_backup:
```

### Kubernetes 部署

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: xugudb
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
        resources:
          requests:
            memory: "4Gi"
            cpu: "2"
          limits:
            memory: "8Gi"
            cpu: "4"
        volumeMounts:
        - name: data
          mountPath: /xugudb/XHOME
        - name: log
          mountPath: /xugudb/XGLOG
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 100Gi
  - metadata:
      name: log
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 20Gi
---
apiVersion: v1
kind: Service
metadata:
  name: xugudb
spec:
  type: ClusterIP
  ports:
  - port: 5138
    targetPort: 5138
  selector:
    app: xugudb
```

> 详细参考：[Docker 与容器部署](references/docker-deployment.md)

## 启动与停止

### 启动模式

| 模式 | 参数 | 说明 |
|------|------|------|
| 前台模式 | `--child` | 占用当前终端，Ctrl+C 停止 |
| 后台模式 | `--server` | systemd 管理，后台运行 |
| 守护模式 | `--daemon` | 自动重启 |

### 启动参数

```bash
# 查看帮助
./xugu_linux_x86_64_* --help

# 指定工作目录
./xugu_linux_x86_64_* -D="/data/xugudb" --child

# 启动时覆盖配置
./xugu_linux_x86_64_* --start-ini="listen_port=10578,min_pass_len=10" --child

# 初始化配置文件
./xugu_linux_x86_64_* --init-setup

# 生成 cluster.ini 模板
./xugu_linux_x86_64_* --init-cluster
```

### 停止

```sql
-- 正常关闭（等待事务完成）
SHUTDOWN;

-- 立即关闭（中断当前事务）
SHUTDOWN IMMEDIATE;
```

> 详细参考：[启动参数与管理](references/startup-management.md)

## 集群高可用部署方案

| 方案 | 架构 | RPO | RTO | 适用场景 |
|------|------|-----|-----|----------|
| 数据中心 | 存算分离 + 管理双活 | 0 | 秒级 | 单机房高可用 |
| 同城双活 | 对称双中心实时同步 | 0 | 秒级 | 跨机房负载均衡 |
| 同城灾备 | Binlog 增量同步 | 0 | <30s | 机房级容灾 |
| 两地三中心 | 同城灾备+异地异步 | 近0 | 分钟级 | 区域级容灾 |

> 详细参考：[集群高可用方案](references/ha-deployment.md)

## 工作流程

当用户咨询安装部署问题时：

1. 确定部署模式（单机/主备/分布式/容器化）
2. 检查软硬件环境是否满足要求
3. 提供对应版本的完整部署步骤
4. 指导 License 申请和激活
5. 配置关键参数（xugu.ini / cluster.ini）
6. 验证部署成功
7. 对容器化部署提供 Dockerfile/Compose/K8s 模板

## 参考文档

- [软硬件环境需求](references/requirements.md) — 操作系统、CPU 架构、依赖库、环境检查
- [部署流程](references/deployment-guide.md) — 完整部署流程、License 管理、校验包完整性
- [各版本部署指南](references/edition-deployment.md) — 标准版/企业版/分布式版/安全版详细步骤
- [Docker 与容器部署](references/docker-deployment.md) — Dockerfile、Docker Compose、Kubernetes
- [启动参数与管理](references/startup-management.md) — 启动模式、启动参数、启停管理、Windows 服务
- [集群高可用方案](references/ha-deployment.md) — 数据中心/同城双活/同城灾备/两地三中心
