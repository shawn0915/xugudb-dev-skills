# 各版本部署指南

## 标准版部署（单机）

### 适用场景

- 中小规模业务
- 开发测试环境
- 单机高性能场景

### 部署步骤

```bash
# 1. 创建用户
useradd xugu && passwd xugu
su xugu

# 2. 解压
tar -zxvf XuguDB-12.8_*-linux-x64.tar.gz

# 3. 初始化
cd XuguDB/BIN
./xugu_linux_x86_64_* --init-setup

# 4. 配置 xugu.ini
vi ../SETUP/xugu.ini
# listen_port = 5138;
# task_thd_num = 16;  # CPU核数*2
# pass_mode = 3;

# 5. 配置存储（可选，独立数据盘）
vi ../SETUP/mount.ini
vi ../SETUP/datafile.ini

# 6. 启动
./xugu_linux_x86_64_* --child
```

### 目录结构

```
XuguDB/
├── BIN/        # 可执行文件
├── SETUP/      # 配置文件
├── XHOME/      # 数据文件（启动后创建）
│   ├── CATA/   # 元数据
│   ├── DATA/   # 用户数据
│   ├── TEMP/   # 临时文件
│   ├── REDO/   # 重做日志
│   ├── UNDO/   # 回滚段
│   └── ARCH/   # 归档日志
├── XGLOG/      # 日志文件
├── BACKUP/     # 备份文件
└── XVOCA/      # 词汇表
```

## 企业版部署（双节点主备）

### 适用场景

- 需要高可用的生产环境
- 自动故障切换

### 环境要求

- 2 台服务器
- 互相网络可达
- 时间同步

### 部署步骤

**两台机器都执行：**

```bash
# 1. 创建用户、解压（同标准版）
useradd xugu && passwd xugu
su xugu
unzip XuguDB-Server-12.9.9-linux-x86_64-*.zip

# 2. 生成 cluster.ini 模板
cd XuguDB/BIN
./xugu_linux_x86_64_* --init-cluster
# 将 SETUP/cluster.ini.temp 复制为 cluster.ini
cp ../SETUP/cluster.ini.temp ../SETUP/cluster.ini
```

**配置 cluster.ini：**

```ini
# 节点1（192.168.30.236）
MASTER_GRPS = 1
MSG_PORT_NUM = 16
MY_NID = 1
NID = "192.168.30.236:5158,192.168.30.237:5158"
# 节点1角色
# M = Master（主控）
# SQW = Storage + Query + Work（存储+查询+工作）
```

```ini
# 节点2（192.168.30.237）
MASTER_GRPS = 1
MSG_PORT_NUM = 16
MY_NID = 2
NID = "192.168.30.236:5158,192.168.30.237:5158"
```

> 关键：每台机器的 `MY_NID` 不同，`NID` 列表相同

**两台机器同时启动：**

```bash
./xugu_linux_x86_64_* --child
```

### cluster.ini 参数说明

| 参数 | 说明 |
|------|------|
| MASTER_GRPS | Master 组数（企业版=1） |
| MSG_PORT_NUM | 消息端口数（推荐 16） |
| MAX_SEND_WIN | 最大发送窗口（默认 2046） |
| MY_NID | 当前节点 ID |
| NID | 所有节点的 IP:PORT 列表 |
| ROLE | 节点角色（M/S/Q/W/G），Q 和 W 必须组合部署 |
| LPU | 逻辑处理单元数 |
| STORE_WEIGHT | 存储权重 |

> UDP 端口 = PORTS端口 + 20，需确保防火墙放行

## 分布式版部署（3~N 节点）

### 适用场景

- 大规模数据存储和高并发查询
- 水平扩展需求

### 环境要求

- 至少 3 台服务器（推荐 4 台以上）
- 万兆网络
- 时间同步 + SSH 免密

### 节点角色规划（4 节点示例）

| 节点 | IP | 角色 | 说明 |
|------|-----|------|------|
| Node1 | 192.168.30.236 | M, SQW | Master + 存储查询工作 |
| Node2 | 192.168.30.237 | M, SQW | Master备 + 存储查询工作 |
| Node3 | 192.168.30.238 | SQW | 存储查询工作 |
| Node4 | 192.168.30.239 | SQW | 存储查询工作 |

### cluster.ini 配置要点

```ini
MASTER_GRPS = 1
MSG_PORT_NUM = 16
MY_NID = 1                    # 每个节点不同
NID = "192.168.30.236:5158,192.168.30.237:5158,192.168.30.238:5158,192.168.30.239:5158"
```

### 部署流程

1. 所有节点创建用户、解压安装包
2. 所有节点生成并配置 cluster.ini（MY_NID 不同）
3. 所有节点**同时启动**
4. 验证集群状态：`SHOW CLUSTERS;`

## 安全版部署

### 适用场景

- 涉密系统
- 需要增强安全审计

### 与企业版的区别

- 启用三权分立（SYSDBA/SYSAUDITOR/SYSSSO）
- 强制密码策略
- 强制审计日志
- 部署步骤与企业版类似（2 节点）

## Windows 部署

### 服务管理

```cmd
# 安装为 Windows 服务
xugu_windows_amd64_*.exe --install

# 启动服务
xugu_windows_amd64_*.exe --start

# 停止服务
xugu_windows_amd64_*.exe --stop

# 卸载服务
xugu_windows_amd64_*.exe --remove
```

> Windows 部署使用 CMD 或 PowerShell 操作
