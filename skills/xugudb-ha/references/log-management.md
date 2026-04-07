# 日志管理

## 概述

日志是数据库系统运行过程中记录的关键信息，用于回溯历史运行状态、排查问题、追踪异常和性能优化。所有日志文件默认最大 100MB，达到阈值后自动归档（追加时间后缀）并生成新文件。

## 日志文件总览

| 日志名称 | 路径 | 类别 | 用途 |
|----------|------|------|------|
| ERROR.LOG | /Server/XGLOG/ | 错误日志 | 错误和异常信息，故障排查 |
| EVENT.LOG | /Server/XGLOG/ | 事件日志 | 关键事件（启停、备份、集群变更等） |
| TRACE.LOG | /Server/XGLOG/ | 追踪日志 | 内部执行路径与调试信息 |
| COMMAND.LOG | /Server/XGLOG/ | 命令日志 | 所有执行的 SQL 语句 |
| SLOWSQL.LOG | /Server/XGLOG/ | 慢 SQL 日志 | 执行时间超过阈值的 SQL |
| stdout.txt | /Server/BIN/ | 输出日志 | 标准输出，容器环境日志查看 |

## 一、ERROR.LOG（错误日志）

### 1.1 记录内容

- SQL 执行异常（语法错误、约束冲突、权限不足等）
- 系统运行错误（I/O 异常、连接超时等）
- 内部断言失败或致命错误（崩溃、内存越界等）
- 后台任务失败（统计分析、表清理、作业执行等）

### 1.2 错误级别

| 编号 | 级别 | 说明 |
|------|------|------|
| 1 | NOTICE | 警告级别，事务正常执行中的警告 |
| 2 | USEREX | 用户定义错误（RAISE_APPLICATION_ERROR / THROW / RAISE） |
| 3 | ERROR | 中止命令级别，违反设计约束 |
| 4 | ABORT | 中止事务级别，事务被 KILL |
| 5 | DLOCK | 死锁错误 |
| 6 | L06 | 陈旧事务异常，最大活动事务号差值过大（默认600万） |
| 7 | SYSEX | 系统内部异常（存取保护事故、存储异常等） |
| 8 | NETER | 网络错误 |
| 9 | MEMER | 内存错乱（分配或释放异常） |

### 1.3 日志格式

```
1 ERROR 624 10049 2025-07-02 11:44:01 127.0.0.1 SYSDBA 1 字段变量或函数"ABC"不存在 select abc from dual;
```

字段含义：

| 字段 | 说明 |
|------|------|
| 1 | 机器节点号 |
| ERROR | 错误级别 |
| 624 | 错误号 |
| 10049 | 错误码 |
| 2025-07-02 11:44:01 | 发生时间 |
| 127.0.0.1 | 连接 IP |
| SYSDBA | 用户名 |
| 1 | 数据库 ID |
| 字段变量或函数"ABC"不存在 | 错误信息 |
| select abc from dual; | 错误命令（最大 16K 字符，超过截断） |

### 1.4 相关参数和系统表

- 参数 `error_level`：控制记录的错误级别
- 参数 `errlog_size`：控制单个日志文件大小（默认 100MB）
- 系统表：`SYS_ERROR_LOG`、`SYS_ALL_ERROR_LOG`

## 二、EVENT.LOG（事件日志）

### 2.1 事件类型

| 事件类型 | 说明 |
|----------|------|
| BACKUP | 备份事件 |
| CKPT / CKPT1 | Checkpoint 事件（全量/增量） |
| CLU_EVENT | 集群节点接入/死亡/Master切换事件 |
| SYS_START / SYS_START_ERR | 系统启动事件 |
| SYS_EXIT | 系统关闭事件 |
| DB_OPEN | 开库事件 |
| DEAD_LOCK | 死锁事件 |
| EXCEPTION | 系统线程异常事件 |
| IB_ERR | Infiniband 通道异常事件 |
| KILL_TRANS | 中止事务事件 |
| MEDIA_ERR | 存储媒体错误事件 |
| MIGRATE | 存储均衡迁移启停事件 |
| MOUNT_ERR / MOUNT_INFO | Master 挂载全局存储异常/信息 |
| REPAIR / REPAIR_ERR / REPAIR_CRASH | REDO 恢复系统事件 |
| RESTORE / RESTORE_ERR | 恢复事件 |
| XLOG_REG | REDO 记载事件 |

### 2.2 日志格式

```
2 CLU_EVENT 2025-06-24 17:52:47 SYSTEM NODE Node 3 deaded.
```

字段含义：节点号 | 事件类型 | 时间 | 数据库名 | 对象名 | 事件描述

### 2.3 关键集群事件

| 事件描述 | 说明 |
|----------|------|
| Switch master from node X to node Y | 主Master死亡，副master切换为主 |
| Node X joined | 新节点加入集群完成 |
| Node X deaded | 集群节点死亡，处理开始 |
| Node X deaded done | 集群节点死亡处理完成 |

### 2.4 相关系统表

- `SYS_EVENT_LOG`、`SYS_ALL_EVENT_LOG`

## 三、TRACE.LOG（追踪日志）

记录数据库内部详细执行路径与调试信息，用于复杂问题的深度排查与开发调试。

## 四、COMMAND.LOG（命令日志）

记录所有执行的 SQL 语句，便于问题重现与查询分析。

## 五、SLOWSQL.LOG（慢 SQL 日志）

记录执行时间超过阈值的 SQL 语句，用于性能优化与瓶颈分析。

## 六、stdout.txt（输出日志）

捕获程序运行过程中的标准输出信息，常用于开发调试或容器环境下的日志查看。路径：`/Server/BIN/stdout.txt`。

## 使用场景总结

| 场景 | 推荐日志 |
|------|----------|
| 程序缺陷定位 | ERROR.LOG |
| 异常 SQL 跟踪 | ERROR.LOG |
| 系统故障排查 | ERROR.LOG + EVENT.LOG |
| 系统审计 | EVENT.LOG |
| 集群变更追溯 | EVENT.LOG |
| 复杂问题调试 | TRACE.LOG |
| SQL 重现与分析 | COMMAND.LOG |
| 性能优化 | SLOWSQL.LOG |
| 容器日志查看 | stdout.txt |
