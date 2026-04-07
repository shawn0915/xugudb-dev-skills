---
name: 虚谷数据库系统配置参考
name_for_command: xugudb-config
description: |
  虚谷数据库系统配置完整参考：xugu.ini 主配置参数、集群配置、数据文件配置、
  路径挂载、黑白名单、类型映射、内置全局变量、连接会话参数。
  涵盖 SQL 引擎/事务/存储/安全/缓冲区/并行度/网络/日志/备份/兼容性等全部配置项。
---

# 虚谷数据库系统配置参考

虚谷数据库系统配置参数集中放置在数据库根路径下的 SETUP 目录里，用于控制存储文件、集群通信、类型映射、黑白名单和其他系统运行时行为。

## 配置文件概览

| 配置文件 | 说明 |
|----------|------|
| xugu.ini | 主配置文件，包含网络监听、系统调度和并行、缓冲、SQL 引擎、存储系统参数、备份同步、系统日志、审计、自动分析等 |
| cluster.ini | 集群配置，包含规模、通信、角色、算力、存储权重等 |
| datafile.ini | 数据文件配置，指定数据/临时/回滚表空间文件的存储位置和个数 |
| mount.ini | 路径挂载配置，数据库逻辑目录到操作系统物理路径的映射 |
| trust.ini | 黑白名单配置，客户端 IP 的受信认证策略 |
| types.ini | 数据类型映射配置，外部类型名到内置基础数据类型的映射 |

## 参数修改方式

虚谷数据库的参数修改分为以下几种方式：

- **在线修改，立即生效** -- 通过 `SET 参数名 TO 值` 修改，无需重启
- **离线修改，重启生效** -- 编辑 xugu.ini 文件后重启数据库服务
- **离线修改，首次启动服务生效** -- 仅在数据库首次初始化启动时读取
- **不可修改** -- 系统内部固定值，只读

查看参数值：`SHOW 参数名`
查看所有参数：`SHOW SYS_VARS`

## xugu.ini 主配置参数

xugu.ini 是虚谷数据库最核心的配置文件，按功能分为以下类别：

| 类别 | 参数数量 | 说明 |
|------|----------|------|
| SQL 引擎参数 | 42 | 并行执行、优化器、游标、预编译、字典匹配、流导入等 |
| 事务子系统参数 | 11 | 隔离级别、自动提交、检查点、死锁检测、WAL 重放等 |
| 存储子系统参数 | 25 | 表空间文件、数据片、数据版本、存储迁移等 |
| 存储子系统附属参数 | 11 | 写回策略、AIO、持久化模式、缓存写穿等 |
| 安全审计参数 | 2 | 审计开关、安全等级 |
| 系统安全参数 | 7 | 口令复杂度策略（长度/大小写/数字/特殊字符/弱口令字典） |
| 系统并行度参数 | 9 | 线程数、CPU 绑定、Hash 表/分区/锁管理器规模 |
| 系统缓冲区参数 | 11 | 数据缓冲区、SGA 内存、排序内存、Hash 内存、交换缓冲区 |
| 网络侦听参数 | 13 | 端口、连接数、超时、字符集、时区、监听模型 |
| 系统运行(跟踪)日志参数 | 9 | 错误日志、命令日志、慢 SQL、Core Dump 等 |
| 数据备份与同步复制参数 | 8 | 归档模式、变更日志、回收站等 |
| 系统分析参数 | 8 | 自动分析、直方图、监控等 |
| 兼容性参数 | 4 | Oracle/MySQL/PostgreSQL 兼容模式、GROUP BY 模式、自增列模式 |

> 详细参考：[xugu-ini](skills/xugudb-config/references/xugu-ini.md)

## 辅助配置文件

cluster.ini、datafile.ini、mount.ini、trust.ini、types.ini 等辅助配置文件，用于集群部署、数据文件管理、路径映射、黑白名单和类型兼容。

> 详细参考：[cluster-config](skills/xugudb-config/references/cluster-config.md)

## 内置全局变量

内置全局变量是在编译和安装时确定的预设参数，通过 `SHOW 变量名` 查看，大部分不可修改。包含版本信息、构建参数、系统表查询类功能等 29 个变量。

> 详细参考：[global-variables](skills/xugudb-config/references/global-variables.md)

## 连接会话参数

连接级参数设置，作用范围为当前连接会话。通过 `SET 参数名 TO 值` 修改，包含字符集、兼容模式、事务隔离级别、优化器模式等 30 个参数。

> 详细参考：[session-parameters](skills/xugudb-config/references/session-parameters.md)

## 常用配置场景

### 性能调优
- **数据缓冲区**：`data_buff_mem` 建议配置为物理内存的 60%
- **并行度**：`max_parallel` 设为 CPU 核数，`auto_use_eje` 开启自动并行
- **Hash 内存**：`system_sga_mem` 和 `max_hash_mem` 根据排序/聚合负载调整
- **连接数**：`use_std_nio = false` 使用 POLL 模型突破 1024 限制

### 安全加固
- **口令策略**：`pass_mode = 4` 启用强口令，配合 `min_pass_len`、`min_pass_mixed_case` 等
- **黑白名单**：编辑 trust.ini 限制客户端 IP
- **审计**：`enable_audit = true` 开启审计功能
- **连接失败锁定**：`conn_fail_cnt` 控制失败次数阈值

### 兼容性
- **Oracle 兼容**：`SET def_compatible_mode TO ORACLE`
- **MySQL 兼容**：`SET def_compatible_mode TO MYSQL`
- **PostgreSQL 兼容**：`SET def_compatible_mode TO POSTGRESQL`

## 参考文档

- [xugu.ini 主配置参数](skills/xugudb-config/references/xugu-ini.md) -- SQL 引擎/事务/存储/安全/缓冲区/并行度/网络/日志/备份/兼容性
- [辅助配置文件](skills/xugudb-config/references/cluster-config.md) -- 集群、数据文件、路径挂载、黑白名单、数据类型映射
- [内置全局变量](skills/xugudb-config/references/global-variables.md) -- 版本、构建时间、集群信息、内存状态、线程状态等
- [连接会话参数](skills/xugudb-config/references/session-parameters.md) -- 字符集、兼容模式、隔离级别、优化器模式、游标等
