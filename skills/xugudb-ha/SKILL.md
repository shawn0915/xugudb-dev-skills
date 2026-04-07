---
name: 虚谷数据库高可用与备份恢复
name_for_command: xugudb-ha
description: |
  XuguDB 高可用架构、集群管理、备份恢复、回收站和日志管理。适用于生产环境运维、容灾规划、数据保护和故障排查。
tags: xugudb, ha, high-availability, backup, recovery, cluster, log, recycle-bin
---

# 虚谷数据库高可用与备份恢复

## 概述

虚谷数据库（XuguDB）基于 Shared-Nothing 多副本架构，提供从单机房到两地三中心的多级高可用方案，配合完善的备份恢复机制和日志体系，保障数据安全与业务连续性。

## 高可用架构

XuguDB 支持四种高可用部署方案：

| 方案 | 架构 | RPO | RTO | 适用场景 |
|------|------|-----|-----|----------|
| 数据中心部署 | 存算分离 + 管理双活 | 0 | 秒级 | 单机房高可用 |
| 同城双活 | 对称双中心，实时双向同步 | 0 | 秒级 | 跨机房负载均衡 |
| 同城灾备 | 主备中心，Binlog 增量同步 | 0 | <30s | 机房级容灾 |
| 两地三中心 | 同城灾备 + 异地异步复制 | 近0 | 分钟级 | 区域级容灾 |

### 核心机制

- **管理节点双活**：主备控制节点（Master/Standby）自动故障切换
- **多副本存储**：`default_copy_num` 控制副本数（建议3），数据自动均衡
- **节点降级防脑裂**：控制节点与所有节点失联时自动进入只读模式
- **在线扩缩容**：`ALTER CLUSTER ADD/DROP NODE` 不中断业务

## 工作流程

当用户咨询高可用、备份恢复、集群管理或日志相关问题时：

1. 确定问题类别（高可用架构 / 备份恢复 / 集群运维 / 日志排查 / 回收站）
2. 根据部署模式（单机/企业版/分布式版）给出针对性方案
3. 提供具体的 SQL 语法和操作步骤
4. 标注使用限制和注意事项（权限要求、节点数限制等）
5. 对危险操作（DROP NODE、RESTORE 等）明确提醒确认

## 备份恢复快速参考

### 物理备份（系统级）

```sql
-- 全量备份（SYSDBA 在 SYSTEM 库执行）
BACKUP SYSTEM TO '/BACKUP/SYS.DMP';

-- 增量备份（追加模式）
BACKUP SYSTEM INCREMENT APPEND TO '/BACKUP/SYS.DMP';

-- 在线加密压缩备份
BACKUP SYSTEM TO '/BACKUP/SYS.DMP' ONLINE ENCRYPTOR IS 'key' COMPRESS;

-- 恢复
RESTORE SYSTEM FROM '/BACKUP/SYS.DMP';

-- 按时间点恢复（需开启归档 log_archive_mode > 0）
RESTORE SYSTEM FROM '/BACKUP/SYS.DMP' UNTIL TIME '2025-12-31 23:59:59.999';
```

### 逻辑备份（库/用户/模式/表级）

```sql
-- 库级备份/恢复
BACKUP DATABASE TO '/BACKUP/DB.EXP';
RESTORE DATABASE db_name FROM '/BACKUP/DB.EXP';

-- 用户级备份/恢复
BACKUP USER user_name TO '/BACKUP/USR.EXP';
RESTORE USER user_name FROM '/BACKUP/USR.EXP';

-- 模式级备份/恢复
BACKUP SCHEMA schema_name TO '/BACKUP/SCH.EXP';
RESTORE SCHEMA schema_name FROM '/BACKUP/SCH.EXP';

-- 表级备份/恢复
BACKUP TABLE tab_name TO '/BACKUP/TB.EXP';
RESTORE TABLE tab_name FROM '/BACKUP/TB.EXP';

-- 表级恢复并改名
RESTORE TABLE tab_name RENAME TO new_name FROM '/BACKUP/TB.EXP';
```

## 集群管理快速参考

```sql
-- 查看集群状态
SHOW CLUSTERS;
SELECT * FROM SYS_CLUSTERS;

-- 添加节点
ALTER CLUSTER ADD NODE 4 DESCRIBE 'RACK=0001 PORTS="192.168.2.236:19114" ROLE=SQW LPU=3 STORE_WEIGHT=3 STATE=DETECT';

-- 下线节点（V12+）
ALTER CLUSTER SET NODE 4 OFFLINE;

-- 删除节点（需先下线）
ALTER CLUSTER DROP NODE 4;

-- 检查版本修复状态
SELECT COUNT(*) FROM SYS_GSTORES WHERE store_sta != 41;

-- 清除降级标志
SET CLUSTER_FAULT_LEVEL TO 0;
```

## 日志文件总览

| 日志 | 路径 | 用途 |
|------|------|------|
| ERROR.LOG | /Server/XGLOG/ | 错误和异常信息，故障排查 |
| EVENT.LOG | /Server/XGLOG/ | 关键事件（启停、备份、集群变更） |
| TRACE.LOG | /Server/XGLOG/ | 内部执行路径，深度调试 |
| COMMAND.LOG | /Server/XGLOG/ | 所有 SQL 语句记录 |
| SLOWSQL.LOG | /Server/XGLOG/ | 慢 SQL 分析 |
| stdout.txt | /Server/BIN/ | 标准输出，容器环境日志 |

## 参考文档

- [高可用架构](references/ha-architecture.md) - 数据中心/同城双活/同城灾备/两地三中心部署方案
- [备份恢复](references/backup-recovery.md) - 物理备份、逻辑备份、各级别备份恢复语法与示例
- [集群管理](references/cluster-management.md) - 节点角色、状态查看、扩缩容、降级与故障恢复
- [日志管理](references/log-management.md) - 各类日志文件格式、级别与使用场景
- [回收站](references/recycle-bin.md) - 回收站开关、表恢复与清理操作
