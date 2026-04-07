---
title: 虚谷数据库部署指南
description: XuguDB 各版本部署步骤、cluster.ini 参数详解、目录结构与高可用部署方案
tags: xugudb, deployment, cluster, installation, enterprise, distributed
---

# 部署指南

## 版本介绍

XuguDB 提供五种不同形态的数据库，以响应不同用户的场景需求：

| 版本 | 节点规模 | 说明 |
|------|----------|------|
| **个人版** | 单机/多机 | 免费试用 360 天，仅支持部分功能项 |
| **标准版** | 单机 | 高度兼容 PostgreSQL/MySQL/Oracle，支持多用户资源隔离，可无损扩展至分布式 |
| **企业版** | 双节点 | 在标准版基础上支持节点故障自动切换，确保高可用性和业务连续性 |
| **分布式版** | 3~N 节点 | 在企业版基础上支持水平扩展，动态调整节点数量，适用于大规模数据处理和高并发场景 |
| **安全版** | 单机/双机 | 提供数据库审计功能，实时收集各节点用户访问行为，形成细粒度审计结果，适用于高安全要求场景 |

版本功能差异通过 **License** 控制，详细功能对比参见产品版本文档。

## 分布式版部署步骤（Linux）

以搭建 4 节点集群为示例（IP: 192.168.30.236~239）。

### 步骤 1：创建用户

在各服务器上创建专用数据库用户：

```bash
# 创建用户（默认家目录 /home/xugu，可通过 -d 指定）
useradd xugu
passwd xugu
```

### 步骤 2：解压产品包

使用 xugu 用户登录，上传安装包并解压：

```bash
# 以 v12.9 某版本为例
unzip XuguDB-Server-12.9.9-linux-x86_64-20250624.zip
```

解压后目录结构包含 BIN、SETUP 等子目录。

### 步骤 3：初始化 cluster.ini

进入 `XuguDB/Server/BIN` 目录，生成集群配置文件：

```bash
./xugu_linux_x86_64_20250624 --init-cluster
```

执行后在 BIN 同级目录生成 `SETUP/cluster.ini.temp` 文件。根据服务器配置编辑该文件，完成后重命名为 `cluster.ini`。

**关键规则**：
- 不同节点之间的 cluster.ini 区别仅在于 `MY_NID` 配置不同
- cluster.ini 中所有参数项位置顺序不能改变，且首行 `#` 不能删除
- 具备管理角色（M）的节点只有 2 个，其他节点不能配置管理角色

### 步骤 4：（可选）初始化其他配置文件

```bash
./xugu_linux_x86_64_20250624 --init-setup
```

初始化 SETUP 目录下的 xugu.ini、mount.ini、types.ini 和 trust.ini。若无需更改默认配置可跳过。

### 步骤 5：（可选）存储文件映射配置

通过 `mount.ini` 和 `datafile.ini` 规划存储路径：

```ini
# mount.ini 示例 - 路径映射
/XGLOG  ./XGLOG
/CATA   ./XHOME/CATA
/DATA   ./XHOME/DATA
/TEMP   ./XHOME/TEMP
/REDO   ./XHOME/REDO
/BACKUP ./XHOME/BACKUP
/ARCH   ./XHOME/ARCH
/UNDO   ./XHOME/UNDO
/MODI   ./XHOME/XMODI
/DATA1  /data1/data
/TEMP1  /data1/temp
```

```ini
# datafile.ini 示例 - 手动创建
#DATA_FILES
/DATA1/DATA1.DBF
/DATA1/DATA2.DBF
/DATA1/DATA3.DBF
/DATA1/DATA4.DBF
#TEMP_FILES
/TEMP1/TEMP1.DBF
/TEMP1/TEMP2.DBF
```

> **注意**：映射的路径必须提前手动创建。若默认安装可跳过此步骤。

### 步骤 6：常用参数检查

编辑 `xugu.ini` 文件，检查以下关键参数（个人及测试环境可使用默认值）：

| 参数 | 默认值 | 配置建议 |
|------|--------|----------|
| `listen_port` | 5138 | 保持默认 5138 |
| `pass_mode` | 3 | 建议 3（必须含字母、数字和特殊符号） |
| `task_thd_num` | 16 | 业务最大并发数 * 2 |
| `max_parallel` | 2 | CPU 核数 * 2（集群场景取决于 LPU 总和） |
| `cata_parti_num` | 32 | CPU 核数（核数 > 32 时为核数 / 2） |
| `rsync_thd_num` | 8 | CPU 核数 / 2 |
| `rtran_thd_num` | 8 | CPU 核数 / 2 |
| `data_buff_mem` | 102400 | 物理内存 60%（单位：MB） |
| `system_sga_mem` | 51200 | 物理内存 20%（单位：MB） |
| `default_copy_num` | 3 | 分布式集群设置为 3 |
| `safely_copy_num` | 2 | 分布式集群设置为 2 |
| `init_data_space_num` | 4 | 保持默认 |
| `init_temp_space_num` | 2 | 保持默认 |
| `init_undo_space_num` | 2 | 保持默认 |
| `def_data_space_size` | 1024 | 按磁盘大小与 IO 评估（单位：MB） |
| `def_temp_space_size` | 1024 | 可根据业务调整（单位：MB） |
| `def_undo_space_size` | 1024 | 可根据业务调整（单位：MB） |
| `def_redo_file_size` | 1024 | 可根据业务调整（单位：MB） |
| `max_trans_modify` | 200000 | 单事务最大变更行数（0 为不限） |
| `enable_node_degrade` | true | 是否启用节点降级 |

### 步骤 7：导入 License

将申请的 `license.dat` 文件放在管理节点的 BIN 目录下。

### 步骤 8：启动数据库

在各节点 BIN 目录下同时执行：

```bash
./startdb.sh
```

**启动方式说明**：
- `--child`：前端监控方式启动
- `--service` / `--server`：后台服务方式启动
- 可编辑 `startdb.sh` 修改启动方式

验证启动成功：查看各节点 `stdout.txt` 日志文件，出现监听成功信息即说明服务启动成功。

## cluster.ini 关键参数说明

| 参数名 | 说明 |
|--------|------|
| `MASTER_GRPS` | 主控角色组数（两个一组构成主备关系），目前仅支持 1 组 |
| `MSG_PORT_NUM` | 集群间通信端口个数，须与节点 PORTS 一致（最大 16） |
| `MAX_SEND_WIN` | 消息发送窗口大小，用于节点间通信流量控制（最大 2046） |
| `MY_NID` | 当前节点号，必须与当前 IP 对应的 PORTS 配置匹配 |
| `NID` | 集群节点号，连续递增数值 |
| `PORTS` | 节点网络通讯端口，格式 `IP1:PORT1,IP2:PORT2`。端口组数须与 MSG_PORT_NUM 一致。发送端口号 = 接收端口号 + 20，部署时需验证端口未被占用 |
| `ROLE` | 节点角色：M（主控）、S（存储）、Q（查询，建立 TCP 监听对外服务）、W（工作，并发并行任务调度计算）、G（变更收集）。Q 和 W 必须组合部署（QW），M 必须配置 2 个且分别在奇偶节点 |
| `LPU` | 参与数据运算的最大逻辑 CPU 数，建议为物理 CPU 核数 - 1 |
| `STORE_WEIGHT` | 节点存储权重，按磁盘容量比配置；磁盘容量相同则配相同数值 |

## 企业版部署（双机热备）

企业版仅支持部署为 **2 节点** 数据库集群。部署步骤与分布式版基本一致：

1. 创建用户 → 解压产品包 → 初始化 cluster.ini → 配置参数 → 导入 License → 启动集群

**企业版与分布式版的关键差异**：

| 对比项 | 企业版 | 分布式版 |
|--------|--------|----------|
| 节点数 | 固定 2 节点 | 3 节点及以上 |
| `default_copy_num` | 2 | 3 |
| `safely_copy_num` | 2 | 2 |
| 水平扩展 | 不支持 | 支持动态增加节点 |
| 多网络通道 | 1~4 路 | 1~16 路 |

## 集群高可用部署方案

XuguDB 采用基于 Shared-Nothing 的多副本架构，支持多级高可用部署：

### 数据中心部署

- 存算分离架构：存储节点专注数据持久化（多副本），计算节点负责查询和事务
- 管理节点双活冗余设计：一个管理节点故障时另一个自动接管
- 支持 Binlog 进行数据备份、恢复和主从同步

### 同城双活部署

- 两个数据中心对称部署，同时运行并承载相同业务负载
- 通过高速低延迟网络互联，增量数据毫秒级实时双向同步，保持数据强一致性
- 任一数据中心故障时可无缝切换，用户无感知
- 故障中心修复后可平滑重新接入

### 同城灾备部署

- 核心生产数据库 + 同城灾备中心
- 基于 Binlog 增量同步，毫秒级实时同步，RPO=0
- 故障时自动切换至灾备中心，RTO < 30s
- 故障恢复后支持数据反向同步，快速重建

### 两地三中心部署

- 在同城灾备基础上增建异地灾备中心
- 异地采用异步复制（避免对核心库性能影响）
- 同城故障：切换至同城灾备（RPO 约 0，RTO < 30s）
- 区域级灾难：异地灾备启动应急流程
- 灾后支持数据回迁，恢复多中心冗余架构

## 目录结构

分布式版与企业版数据库目录结构一致：

```
├── BACKUP/              # 数据库备份文件
├── BIN/
│   ├── license.dat      # License 文件
│   ├── startdb.sh       # 启动脚本
│   ├── weak_pass_dictionary.txt  # 弱口令字典
│   ├── xugu_init.sql    # 初始化文件
│   └── xugu_linux_x86_64_*      # 数据库程序
├── SETUP/
│   ├── cluster.ini      # 集群配置文件
│   ├── mount.ini        # 映射路径文件
│   ├── trust.ini        # 黑白名单配置
│   ├── types.ini        # 数据类型映射文件
│   └── xugu.ini         # 参数配置文件
├── XGLOG/
│   ├── ERROR.LOG        # 错误日志
│   ├── EVENT.LOG        # 事件日志
│   └── TRACE.LOG        # 跟踪日志
├── XHOME/
│   ├── ARCH/            # 归档日志
│   ├── CATA/            # 系统结构性控制文件
│   │   ├── CTL.BAK      # 控制文件备份
│   │   ├── GSYS1.SYS    # 全局存储控制文件
│   │   ├── LSYS1.SYS    # 本地存储控制文件
│   │   ├── UNDO.SYS     # 回滚日志控制文件
│   │   ├── XUGU.CTL     # 系统配置与目录结构
│   │   └── XUGU.ENC     # 加密机对象信息
│   ├── DATA/            # 数据文件（DATA1~4.DBF）
│   ├── REDO/            # 重做日志（REDO1~2.LOG）
│   ├── TEMP/            # 临时表空间（TEMP1~2.DBF）
│   ├── UNDO/            # 回滚日志（UNDO1~2.DBF）
│   └── XMODI/           # 变更数据文件
└── XVOCA/               # 词表目录（全文索引词典）
```

## 与主流数据库部署对比

| 对比项 | XuguDB 分布式版 | Oracle RAC | MySQL Cluster (NDB) | PostgreSQL Citus |
|--------|-----------------|------------|---------------------|------------------|
| **架构类型** | Shared-Nothing 原生分布式 | Shared-Everything（共享存储） | Shared-Nothing | Shared-Nothing（插件扩展） |
| **最小集群规模** | 3 节点 | 2 节点 + 共享存储 | 3 节点（管理 + 数据 + SQL） | 1 协调节点 + N 工作节点 |
| **配置文件** | cluster.ini + xugu.ini | init.ora + OCR/VD | config.ini + my.cnf | postgresql.conf + citus 配置 |
| **节点角色** | M/W/S/Q/G 五类角色灵活配置 | 所有实例对等 | 管理节点 / 数据节点 / SQL 节点 | 协调节点 / 工作节点 |
| **共享存储依赖** | 不依赖 | 依赖 ASM 或共享文件系统 | 不依赖 | 不依赖 |
| **部署复杂度** | 中等（cluster.ini 配置） | 高（需 Grid Infrastructure） | 高（多种节点类型配置） | 中等（需安装 Citus 扩展） |
| **在线扩展** | 动态增加节点，数据自动均衡 | 需停机添加实例 | 支持在线添加数据节点 | 需 rebalance 操作 |
| **副本机制** | 内置多副本（default_copy_num） | 依赖共享存储冗余 | NDB 自动同步副本 | 依赖流复制或 Citus 副本 |
| **故障切换** | 管理节点自动接管，存储节点副本切换 | 透明应用故障转移（TAF） | 自动故障转移 | 需 Patroni 等外部工具 |
| **高可用方案** | 内置双活 / 灾备 / 两地三中心 | Data Guard + RAC | NDB 内置 + MySQL Replication | Patroni + Citus HA |
