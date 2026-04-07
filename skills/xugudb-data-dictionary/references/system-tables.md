---
title: 虚谷数据库系统表与系统虚表
description: 系统文件表、系统堆表（备份恢复/模式对象/访问控制/其他）、系统虚表（静态/动态）完整列表与查询示例
tags: xugudb, data-dictionary, system-tables, virtual-tables, metadata, catalog
---

# 虚谷数据库系统表与系统虚表

---

## 概述

虚谷数据库的系统表存储了数据库的所有元数据和运行状态信息。按存储方式分为：

- **系统文件表**：基于文件存储的日志类表
- **系统堆表**：基于堆存储的元数据表
- **系统虚表**：不占用物理存储，查询时动态生成数据

命名约定：
- `SYS_` 前缀：当前节点视图
- `SYS_ALL_` 前缀：全局视图（分布式环境下汇总所有节点数据）

---

## 一、系统文件表（10个）

系统文件表用于记录数据库运行过程中的各类日志信息。每类日志提供本节点和全局两个视图。

| 系统表 | 功能说明 |
|--------|----------|
| SYS_ERROR_LOG | 当前节点错误日志，记录数据库运行中的错误信息 |
| SYS_ALL_ERROR_LOG | 全局错误日志，汇总所有节点的错误信息 |
| SYS_EVENT_LOG | 当前节点事件日志，记录数据库关键事件 |
| SYS_ALL_EVENT_LOG | 全局事件日志，汇总所有节点的事件信息 |
| SYS_TRACE_LOG | 当前节点跟踪日志，记录 SQL 执行跟踪信息 |
| SYS_ALL_TRACE_LOG | 全局跟踪日志，汇总所有节点的跟踪信息 |
| SYS_COMMAND_LOG | 当前节点命令日志，记录执行的 DDL/DML 命令 |
| SYS_ALL_COMMAND_LOG | 全局命令日志，汇总所有节点的命令信息 |
| SYS_SLOWSQL_LOG | 当前节点慢 SQL 日志，记录执行时间超过阈值的 SQL |
| SYS_ALL_SLOWSQL_LOG | 全局慢 SQL 日志，汇总所有节点的慢 SQL 信息 |

### 查询示例

```sql
-- 查看最近 10 条错误日志
SELECT * FROM SYS_ERROR_LOG ORDER BY LOG_TIME DESC LIMIT 10;

-- 查看全局慢 SQL（分布式环境）
SELECT * FROM SYS_ALL_SLOWSQL_LOG ORDER BY EXEC_TIME DESC LIMIT 20;

-- 查看最近的 DDL 命令记录
SELECT * FROM SYS_COMMAND_LOG WHERE CMD_TYPE = 'DDL' ORDER BY LOG_TIME DESC LIMIT 10;
```

---

## 二、系统堆表 - 备份恢复（6个）

| 系统表 | 功能说明 |
|--------|----------|
| SYS_MODIFY_LOGS | 数据修改日志，用于增量备份和数据同步 |
| SYS_SUBSCRIBERS | 数据订阅者信息，用于数据复制和同步 |
| SYS_IMPORT_LOGS | 数据导入日志，记录导入操作的历史信息 |
| SYS_STREAMS | 数据流信息，用于数据复制流管理 |
| SYS_BACKUP_PLANS | 备份计划定义，存储自动备份的调度信息 |
| SYS_BACKUP_ITEMS | 备份项详情，记录每次备份的具体内容和状态 |

### 查询示例

```sql
-- 查看所有备份计划
SELECT * FROM SYS_BACKUP_PLANS;

-- 查看最近的备份记录
SELECT * FROM SYS_BACKUP_ITEMS ORDER BY BACKUP_TIME DESC LIMIT 10;

-- 查看数据导入历史
SELECT * FROM SYS_IMPORT_LOGS ORDER BY IMPORT_TIME DESC;
```

---

## 三、系统堆表 - 模式对象（17个）

| 系统表 | 功能说明 |
|--------|----------|
| SYS_TABLES | 所有表的定义信息（表名、所属模式、表类型、行数等） |
| SYS_PARTIS | 表的分区定义信息（分区类型、分区键、分区范围等） |
| SYS_SUBPARTIS | 表的二级分区（子分区）定义信息 |
| SYS_COLUMNS | 表和视图的列定义信息（列名、数据类型、精度、默认值等） |
| SYS_LOBS | 大对象（BLOB/CLOB）列的存储信息 |
| SYS_INDEXES | 索引定义信息（索引类型、索引列、唯一性等） |
| SYS_IDX_PARTIS | 索引分区定义信息 |
| SYS_IDX_SUBPARTIS | 索引二级分区定义信息 |
| SYS_CONSTRAINTS | 约束定义信息（主键、外键、唯一、检查约束等） |
| SYS_VIEWS | 视图定义信息（视图名、定义 SQL 等） |
| SYS_VIEW_COLUMNS | 视图列定义信息 |
| SYS_SEQUENCES | 序列定义信息（起始值、步长、缓存大小等） |
| SYS_PROCEDURES | 存储过程和函数定义信息（名称、参数、源代码等） |
| SYS_PACKAGES | 包定义信息（包头、包体） |
| SYS_TRIGGERS | 触发器定义信息（触发事件、触发时机、触发体等） |
| SYS_TYPES | 自定义类型定义信息（OBJECT/VARRAY/TABLE 类型） |
| SYS_OBJECTS | 所有模式对象的统一视图（表、视图、过程、包等） |

### 查询示例

```sql
-- 查看指定模式下的所有表
SELECT TABLE_NAME, TABLE_TYPE, NUM_ROWS
FROM SYS_TABLES
WHERE SCHEMA_ID = (SELECT SCHEMA_ID FROM SYS_SCHEMAS WHERE SCHEMA_NAME = 'PUBLIC');

-- 查看表的列信息（类似 DESC 命令）
SELECT COL_NAME, TYPE_NAME, PRECISION, SCALE, NULLABLE, DEFAULT_VAL
FROM SYS_COLUMNS
WHERE TABLE_ID = (SELECT TABLE_ID FROM SYS_TABLES WHERE TABLE_NAME = 'MY_TABLE')
ORDER BY COL_NO;

-- 查看表的索引
SELECT I.INDEX_NAME, I.INDEX_TYPE, I.IS_UNIQUE
FROM SYS_INDEXES I
WHERE I.TABLE_ID = (SELECT TABLE_ID FROM SYS_TABLES WHERE TABLE_NAME = 'MY_TABLE');

-- 查看表的约束
SELECT CONS_NAME, CONS_TYPE, COL_LIST
FROM SYS_CONSTRAINTS
WHERE TABLE_ID = (SELECT TABLE_ID FROM SYS_TABLES WHERE TABLE_NAME = 'MY_TABLE');

-- 查看分区表的分区信息
SELECT P.PARTI_NAME, P.PARTI_TYPE, P.HIGH_VALUE
FROM SYS_PARTIS P
JOIN SYS_TABLES T ON P.TABLE_ID = T.TABLE_ID
WHERE T.TABLE_NAME = 'MY_PARTI_TABLE';

-- 查看所有存储过程
SELECT PROC_NAME, PROC_TYPE, SCHEMA_ID
FROM SYS_PROCEDURES;

-- 查看视图定义
SELECT VIEW_NAME, DEFINITION
FROM SYS_VIEWS
WHERE VIEW_NAME = 'MY_VIEW';

-- 查看所有序列
SELECT SEQ_NAME, MIN_VALUE, MAX_VALUE, INCREMENT_BY, CACHE_SIZE
FROM SYS_SEQUENCES;
```

---

## 四、系统堆表 - 访问控制（12个）

| 系统表 | 功能说明 |
|--------|----------|
| SYS_USERS | 数据库用户信息（用户名、密码策略、状态、权限等） |
| SYS_SCHEMAS | 模式信息（模式名、所属用户等） |
| SYS_ROLE_MEMBERS | 角色成员关系（用户-角色的授权关系） |
| SYS_ACLS | 访问控制列表（ACL），定义对象级别的访问权限 |
| SYS_POLICIES | 安全策略定义（行级安全策略、VPD 策略等） |
| SYS_LEVELS | 强制访问控制（MAC）安全级别定义 |
| SYS_CATEGORIES | 强制访问控制（MAC）安全类别定义 |
| SYS_AUDIT_DEFS | 审计定义，指定需要审计的操作和对象 |
| SYS_AUDIT_RESULTS | 审计结果记录，存储审计捕获的操作日志 |
| SYS_FORBIDDEN_IPS | 当前节点的 IP 黑名单 |
| SYS_ALL_FORBIDDEN_IPS | 全局 IP 黑名单（所有节点） |
| SYS_BLACK_WHITE_LIST | 当前节点的 IP 黑白名单配置 |
| SYS_ALL_BLACK_WHITE_LIST | 全局 IP 黑白名单配置（所有节点） |

### 查询示例

```sql
-- 查看所有用户
SELECT USER_NAME, USER_ID, DEFAULT_SCHEMA, ACCOUNT_STATUS
FROM SYS_USERS;

-- 查看角色成员关系
SELECT U.USER_NAME, R.ROLE_NAME
FROM SYS_ROLE_MEMBERS RM
JOIN SYS_USERS U ON RM.MEMBER_ID = U.USER_ID
JOIN SYS_USERS R ON RM.ROLE_ID = R.USER_ID;

-- 查看审计定义
SELECT * FROM SYS_AUDIT_DEFS;

-- 查看 IP 黑白名单
SELECT * FROM SYS_BLACK_WHITE_LIST;
```

---

## 五、系统堆表 - 其他（12个）

| 系统表 | 功能说明 |
|--------|----------|
| SYS_DATABASES | 数据库实例信息 |
| SYS_JOBS | 定时作业定义（作业名、调度表达式、执行内容等） |
| SYS_SYNONYMS | 同义词定义（公共同义词和私有同义词） |
| SYS_DEPENDS | 对象依赖关系（视图依赖表、过程依赖表等） |
| SYS_RECYCLEBIN | 回收站，存储被 DROP 的对象信息（支持 FLASHBACK） |
| SYS_DBLINKS | 数据库链接定义（远程数据库连接信息） |
| SYS_KEYWORDS | 系统关键字和保留字列表 |
| SYS_DT_CONVERT_LIST | 数据类型转换规则列表 |
| SYS_ENCRYPTORS | 加密器定义信息 |
| SYS_DBLINK_SP_TYPES | 数据库链接存储过程类型信息 |
| SYS_SNAPSHOTS | 快照定义信息（物化视图） |
| SYS_SNAPSRCS | 快照源表信息 |

### 查询示例

```sql
-- 查看所有定时作业
SELECT JOB_NAME, JOB_TYPE, ENABLED, NEXT_RUN_TIME
FROM SYS_JOBS;

-- 查看回收站中的对象
SELECT * FROM SYS_RECYCLEBIN;

-- 查看数据库链接
SELECT DBLINK_NAME, HOST, PORT, DB_NAME
FROM SYS_DBLINKS;

-- 查看对象依赖关系
SELECT * FROM SYS_DEPENDS
WHERE OBJ_NAME = 'MY_VIEW';

-- 查看同义词
SELECT SYN_NAME, OBJ_SCHEMA, OBJ_NAME, IS_PUBLIC
FROM SYS_SYNONYMS;
```

---

## 六、静态虚表（9个）

静态虚表提供数据库的配置和元信息，数据在数据库启动后不频繁变化。

| 虚表名 | 功能说明 |
|--------|----------|
| SYS_CHARSETS | 数据库支持的字符集列表（UTF-8、GB18030、GBK 等） |
| SYS_VARS | 当前会话的系统变量（参数名、值、说明） |
| SYS_ALL_VARS | 全局系统变量（所有节点的参数汇总） |
| SYS_METHODS | 系统内置方法列表 |
| SYS_OPERATORS | 系统支持的运算符列表 |
| SYS_DATATYPES | 系统支持的数据类型列表 |
| SYS_ERR_DEFS | 系统错误码定义（错误码、错误消息模板） |
| SYS_SYSTEM_TABLES | 系统表的元信息（系统表名称和描述列表） |
| SYS_SYSTEM_VIEWS | 系统视图的元信息（系统视图名称和描述列表） |

### 查询示例

```sql
-- 查看数据库支持的字符集
SELECT * FROM SYS_CHARSETS;

-- 查看当前会话参数
SELECT * FROM SYS_VARS WHERE VAR_NAME LIKE '%TIMEOUT%';

-- 查看所有系统表列表
SELECT * FROM SYS_SYSTEM_TABLES;

-- 查看所有系统视图列表
SELECT * FROM SYS_SYSTEM_VIEWS;

-- 查看错误码定义
SELECT * FROM SYS_ERR_DEFS WHERE ERR_CODE = 19132;

-- 查看支持的数据类型
SELECT * FROM SYS_DATATYPES;
```

---

## 七、动态虚表 - 任务系统（8个）

| 虚表名 | 功能说明 |
|--------|----------|
| SYS_SESSIONS | 当前节点的会话信息（会话ID、用户、连接时间、状态等） |
| SYS_ALL_SESSIONS | 全局会话信息（所有节点的会话汇总） |
| SYS_THD_STATUS | 当前节点的线程状态（线程ID、执行 SQL、耗时等） |
| SYS_ALL_THD_STATUS | 全局线程状态（所有节点的线程汇总） |
| SYS_THD_SESSION | 当前节点的线程-会话关联信息 |
| SYS_ALL_THD_SESSION | 全局线程-会话关联信息 |
| SYS_TRANS | 当前节点的事务信息（事务ID、状态、开始时间等） |
| SYS_ALL_TRANS | 全局事务信息（所有节点的事务汇总） |

### 查询示例

```sql
-- 查看当前所有会话
SELECT SESS_ID, USER_NAME, CONN_TIME, CURR_SQL
FROM SYS_SESSIONS;

-- 查看全局活跃线程（分布式环境）
SELECT * FROM SYS_ALL_THD_STATUS WHERE STATUS = 'ACTIVE';

-- 查看当前活跃事务
SELECT TRANS_ID, SESS_ID, START_TIME, STATUS
FROM SYS_TRANS;

-- 查看执行时间较长的 SQL
SELECT SESS_ID, USER_NAME, CURR_SQL, EXEC_TIME
FROM SYS_SESSIONS
WHERE EXEC_TIME > 10000
ORDER BY EXEC_TIME DESC;
```

---

## 八、动态虚表 - 全局信息（12个）

| 虚表名 | 功能说明 |
|--------|----------|
| SYS_CTL_VARS | 控制变量（数据库运行时可动态调整的参数） |
| SYS_CLUSTERS | 集群节点信息（节点ID、角色、状态、地址等） |
| SYS_TABLESPACES | 当前节点的表空间信息（名称、大小、使用率等） |
| SYS_ALL_TABLESPACES | 全局表空间信息（所有节点的表空间汇总） |
| SYS_DATAFILES | 当前节点的数据文件信息（文件路径、大小、状态等） |
| SYS_ALL_DATAFILES | 全局数据文件信息（所有节点的数据文件汇总） |
| SYS_MONITORS | 当前节点的监控信息（性能计数器、统计指标等） |
| SYS_ALL_MONITORS | 全局监控信息（所有节点的监控汇总） |
| SYS_RUN_INFO | 当前节点的运行信息（启动时间、版本、运行状态等） |
| SYS_ALL_RUN_INFO | 全局运行信息（所有节点的运行信息汇总） |
| SYS_MEM_STATUS | 当前节点的内存使用状态 |
| SYS_ALL_MEM_STATUS | 全局内存使用状态（所有节点的内存汇总） |

### 查询示例

```sql
-- 查看集群节点状态
SELECT * FROM SYS_CLUSTERS;

-- 查看表空间使用情况
SELECT TS_NAME, TOTAL_SIZE, USED_SIZE, FREE_SIZE
FROM SYS_TABLESPACES;

-- 查看全局表空间使用情况（分布式环境）
SELECT * FROM SYS_ALL_TABLESPACES;

-- 查看数据文件
SELECT FILE_NAME, FILE_SIZE, TS_NAME
FROM SYS_DATAFILES;

-- 查看数据库运行信息
SELECT * FROM SYS_RUN_INFO;

-- 查看内存使用状态
SELECT * FROM SYS_MEM_STATUS;

-- 查看监控指标
SELECT * FROM SYS_MONITORS;
```

---

## 九、动态虚表 - 存储（14个）

| 虚表名 | 功能说明 |
|--------|----------|
| SYS_CACHE_STATUS | 当前节点缓存状态（缓冲池命中率、脏页数等） |
| SYS_ALL_CACHE_STATUS | 全局缓存状态 |
| SYS_PAGE_STATUS | 当前节点页面状态（数据页、索引页的使用情况） |
| SYS_ALL_PAGE_STATUS | 全局页面状态 |
| SYS_LOG_STATUS | 当前节点日志状态（WAL 日志的写入和刷盘状态） |
| SYS_ALL_LOG_STATUS | 全局日志状态 |
| SYS_CHECKPOINT | 当前节点检查点信息（最近检查点时间、LSN 等） |
| SYS_ALL_CHECKPOINT | 全局检查点信息 |
| SYS_REDO_STATUS | 当前节点 Redo 日志状态 |
| SYS_ALL_REDO_STATUS | 全局 Redo 日志状态 |
| SYS_STORAGE_STATUS | 当前节点存储使用状态 |
| SYS_ALL_STORAGE_STATUS | 全局存储使用状态 |
| SYS_IO_STATUS | 当前节点 IO 状态（读写次数、吞吐量等） |
| SYS_ALL_IO_STATUS | 全局 IO 状态 |

### 查询示例

```sql
-- 查看缓存命中率
SELECT * FROM SYS_CACHE_STATUS;

-- 查看 WAL 日志状态
SELECT * FROM SYS_LOG_STATUS;

-- 查看检查点信息
SELECT * FROM SYS_CHECKPOINT;

-- 查看 IO 状态
SELECT * FROM SYS_IO_STATUS;

-- 查看全局存储使用情况（分布式环境）
SELECT * FROM SYS_ALL_STORAGE_STATUS;
```

---

## 十、动态虚表 - 锁（12个）

| 虚表名 | 功能说明 |
|--------|----------|
| SYS_ROW_LOCKS | 当前节点的行锁信息 |
| SYS_ALL_ROW_LOCKS | 全局行锁信息 |
| SYS_TABLE_LOCKS | 当前节点的表锁信息 |
| SYS_ALL_TABLE_LOCKS | 全局表锁信息 |
| SYS_META_LOCKS | 当前节点的元数据锁信息（DDL 操作期间） |
| SYS_ALL_META_LOCKS | 全局元数据锁信息 |
| SYS_TRANS_LOCKS | 当前节点的事务锁信息 |
| SYS_ALL_TRANS_LOCKS | 全局事务锁信息 |
| SYS_LOCK_WAITS | 当前节点的锁等待信息（阻塞关系） |
| SYS_ALL_LOCK_WAITS | 全局锁等待信息 |
| SYS_DEAD_LOCKS | 当前节点的死锁检测信息 |
| SYS_ALL_DEAD_LOCKS | 全局死锁检测信息 |

### 查询示例

```sql
-- 查看当前行锁
SELECT * FROM SYS_ROW_LOCKS;

-- 查看表锁
SELECT * FROM SYS_TABLE_LOCKS;

-- 查看锁等待（排查阻塞问题）
SELECT * FROM SYS_LOCK_WAITS;

-- 查看全局锁等待（分布式环境排查阻塞）
SELECT * FROM SYS_ALL_LOCK_WAITS;

-- 查看死锁信息
SELECT * FROM SYS_DEAD_LOCKS;

-- 查看元数据锁（DDL 操作阻塞排查）
SELECT * FROM SYS_META_LOCKS;
```

---

## 常用运维查询汇总

```sql
-- 1. 查看数据库版本和运行信息
SELECT * FROM SYS_RUN_INFO;

-- 2. 查看所有用户表
SELECT TABLE_NAME, NUM_ROWS FROM SYS_TABLES WHERE TABLE_TYPE = 'HEAP';

-- 3. 查看表空间使用率
SELECT TS_NAME, TOTAL_SIZE, USED_SIZE FROM SYS_TABLESPACES;

-- 4. 查看当前会话及正在执行的 SQL
SELECT SESS_ID, USER_NAME, CURR_SQL FROM SYS_SESSIONS;

-- 5. 查看锁等待和阻塞链
SELECT * FROM SYS_LOCK_WAITS;

-- 6. 查看慢 SQL
SELECT * FROM SYS_SLOWSQL_LOG ORDER BY EXEC_TIME DESC LIMIT 10;

-- 7. 查看内存使用
SELECT * FROM SYS_MEM_STATUS;

-- 8. 查看集群节点状态（分布式环境）
SELECT * FROM SYS_CLUSTERS;

-- 9. 查看所有系统表和系统视图列表
SELECT * FROM SYS_SYSTEM_TABLES;
SELECT * FROM SYS_SYSTEM_VIEWS;
```

---

## 与 Oracle/MySQL/PG 系统表对照

| 查询目的 | XuguDB | Oracle | MySQL | PostgreSQL |
|----------|--------|--------|-------|------------|
| 查看所有表 | SYS_TABLES | DBA_TABLES | information_schema.TABLES | pg_tables |
| 查看列信息 | SYS_COLUMNS | DBA_TAB_COLUMNS | information_schema.COLUMNS | pg_attribute |
| 查看索引 | SYS_INDEXES | DBA_INDEXES | information_schema.STATISTICS | pg_index |
| 查看约束 | SYS_CONSTRAINTS | DBA_CONSTRAINTS | information_schema.TABLE_CONSTRAINTS | pg_constraint |
| 查看视图 | SYS_VIEWS | DBA_VIEWS | information_schema.VIEWS | pg_views |
| 查看序列 | SYS_SEQUENCES | DBA_SEQUENCES | 无（AUTO_INCREMENT） | pg_sequences |
| 查看存储过程 | SYS_PROCEDURES | DBA_PROCEDURES | information_schema.ROUTINES | pg_proc |
| 查看用户 | SYS_USERS | DBA_USERS | mysql.user | pg_roles |
| 查看表空间 | SYS_TABLESPACES | DBA_TABLESPACES | 无 | pg_tablespace |
| 查看会话 | SYS_SESSIONS | V$SESSION | information_schema.PROCESSLIST | pg_stat_activity |
| 查看锁 | SYS_ROW_LOCKS / SYS_TABLE_LOCKS | V$LOCK | information_schema.INNODB_LOCKS | pg_locks |
| 查看事务 | SYS_TRANS | V$TRANSACTION | information_schema.INNODB_TRX | pg_stat_activity |
| 查看参数 | SYS_VARS | V$PARAMETER | SHOW VARIABLES | pg_settings |
