# 集群与高可用常见问题

## 集群节点角色

| 角色 | 简称 | 数量要求 | 职责 |
|------|------|----------|------|
| Master | M | 必须 2 个 | 集群管理、心跳检测 |
| Query | Q | 可选 | 查询处理 |
| Work | W | 1+ | 工作计算、任务调度 |
| Storage | S | 1+ | 数据存储 |
| Gather | G | 可选，最多 2 | 变更收集 |

> 可在 `SYS_CLUSTERS` 系统表的 `NODE_TYPE` 列查看节点角色

## 节点故障处理

### 查看节点状态

```sql
SHOW CLUSTERS;
SELECT * FROM SYS_CLUSTERS;
```

### 移除故障节点

```sql
-- 1. 移除节点
ALTER CLUSTER DROP NODE node_id;

-- 2. 修复节点硬件/网络问题

-- 3. 重新添加节点
ALTER CLUSTER ADD NODE node_id DESCRIBE 'node_description';

-- 4. 等待数据同步完成
```

### Master 节点故障

- Master 必须配置 2 个（主备）
- 主 Master 故障时，备 Master 自动接管
- 如果两个 Master 同时故障，集群不可用

## 高可用方案

### 方案选择

| 方案 | RPO | RTO | 适用场景 |
|------|-----|-----|----------|
| 数据中心（单机房） | 0 | 秒级 | 单机房高可用 |
| 同城双活 | 0 | 秒级 | 跨机房负载均衡 |
| 同城灾备 | 0 | <30s | 机房级容灾 |
| 两地三中心 | 近0 | 分钟级 | 区域级容灾 |

### 脑裂（Split-Brain）防护

- 控制节点与所有存储节点断连时自动进入只读模式（降级）
- 通过 `SET CLUSTER_FAULT_LEVEL TO 0` 清除降级标志
- 建议至少 3 个节点避免脑裂

### 多副本配置

```sql
-- V11: 最多 4 个副本
-- V12: 最多 16 个副本
-- 建议生产环境至少 3 副本
```

## 备份恢复问题

### 备份类型

| 类型 | 适用级别 | 命令 |
|------|----------|------|
| 物理备份 | SYSTEM | BACKUP SYSTEM |
| 逻辑备份 | DATABASE/USER/SCHEMA/TABLE | BACKUP DATABASE/USER/SCHEMA/TABLE |

### 常见备份问题

**备份失败**：
- 检查磁盘空间是否充足
- 物理备份需要 SYSDBA 权限
- 在线备份需添加 ONLINE 关键字

**恢复失败**：
- 检查备份文件完整性
- 加密备份需提供正确的密钥
- 按时间点恢复需要归档日志

## 日志排查

### 日志文件位置

所有日志位于 `/Server/XGLOG/` 目录：

| 日志 | 文件 | 用途 |
|------|------|------|
| 错误日志 | ERROR.LOG | 错误和异常信息 |
| 事件日志 | EVENT.LOG | 启停、备份、集群变更等 |
| 跟踪日志 | TRACE.LOG | 内部执行路径 |
| 命令日志 | COMMAND.LOG | SQL 语句记录 |
| 慢查询日志 | SLOWSQL.LOG | 慢 SQL 分析 |

### 排查步骤

1. 首先查看 `ERROR.LOG` 定位错误信息
2. 查看 `EVENT.LOG` 了解事件序列
3. 如需深入调试，查看 `TRACE.LOG`
4. 性能问题查看 `SLOWSQL.LOG`

### 网络排查

- 检查节点间网络连通性
- 确认防火墙规则
- 建议使用 BOND 双网卡提高网络可靠性
