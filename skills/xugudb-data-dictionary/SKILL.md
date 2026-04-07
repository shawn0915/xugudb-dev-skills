---
name: 虚谷数据库数据字典
name_for_command: xugudb-data-dictionary
description: |
  虚谷数据库系统字典完整参考：系统表（文件表/堆表）、系统虚表（静态/动态）、
  系统视图（ALL/DBA/USER/SSO/AUDIT）、系统包（16个 DBMS 包）。
  包含与 Oracle/MySQL/PostgreSQL 数据字典的对比和迁移指南。
---

# 虚谷数据库数据字典

虚谷数据库的数据字典存储了数据库所有元数据信息，包括系统表、系统虚表、系统视图和系统包。数据字典是 DBA 管理和开发人员理解数据库结构的核心工具。

## 系统表概览

虚谷数据库的系统表分为两大类：系统文件表和系统堆表。

### 系统文件表（10个）

用于记录数据库运行日志，包括错误日志、事件日志、跟踪日志、命令日志和慢 SQL 日志。每类日志提供当前节点视图和全局视图（ALL_ 前缀）。

### 系统堆表（47个）

按功能分为四大类：
- **备份恢复**（6个）：备份计划、备份项、修改日志、订阅、导入日志、数据流
- **模式对象**（17个）：表、分区、列、索引、约束、视图、序列、存储过程、包、触发器、类型等
- **访问控制**（12个）：用户、模式、角色、ACL、策略、审计、黑白名单等
- **其他**（12个）：数据库、作业、同义词、依赖、回收站、DBLink、快照等

> 详细参考：[system-tables](skills/xugudb-data-dictionary/references/system-tables.md)

## 系统虚表概览

系统虚表不占用物理存储，数据在查询时动态生成。分为静态虚表和动态虚表。

### 静态虚表（9个）

提供数据库配置和元信息，如字符集、系统变量、方法、运算符、数据类型、错误定义等。

### 动态虚表（46个）

提供数据库运行时状态，按功能分为：
- **任务系统**（8个）：会话、线程状态、线程会话、事务
- **全局信息**（12个）：控制变量、集群、表空间、数据文件、监控、运行信息、内存状态
- **存储**（14个）：缓存状态、页面状态、日志状态、检查点等
- **锁**（12个）：行锁、表锁、元数据锁、事务锁等

> 详细参考：[system-tables](skills/xugudb-data-dictionary/references/system-tables.md)

## 系统视图

虚谷数据库提供 110 个系统视图，按前缀分为五类：

| 类别 | 前缀 | 数量 | 说明 |
|------|------|------|------|
| ALL 视图 | ALL_ | 33 | 当前用户可访问的所有对象 |
| DBA 视图 | DBA_ | 38 | 全库所有对象（需 DBA 权限） |
| USER 视图 | USER_ | 33 | 当前用户拥有的对象 |
| SSO 视图 | SSO_ | 4 | 安全管理员相关（强制访问控制） |
| AUDIT 视图 | AUDIT_ | 2 | 审计管理员相关 |

> 详细参考：[system-views](skills/xugudb-data-dictionary/references/system-views.md)

## 系统包

虚谷数据库提供 16 个系统包，兼容 Oracle DBMS 包接口：

| 包名 | 功能 |
|------|------|
| DBMS_BACKUP | 备份恢复 |
| DBMS_CRYPTO | 加密解密 |
| DBMS_DBA | DBA 管理 |
| DBMS_LOB | 大对象操作 |
| DBMS_METADATA | 元数据导出 |
| DBMS_OUTPUT | 输出调试 |
| DBMS_SQL | 动态 SQL |
| DBMS_STAT | 统计信息 |
| DBMS_SCHEDULER | 作业调度 |
| DBMS_UTILITY | 实用工具 |
| UTL_RAW | RAW 数据处理 |

> 详细参考：[system-packages](skills/xugudb-data-dictionary/references/system-packages.md)

## 与 Oracle/MySQL/PG 数据字典对比

| 特性 | XuguDB | Oracle | MySQL | PostgreSQL |
|------|--------|--------|-------|------------|
| 系统表前缀 | SYS_ | SYS./DBA_DATA_ | information_schema | pg_catalog |
| 视图分类 | ALL/DBA/USER/SSO/AUDIT | ALL/DBA/USER | information_schema | pg_catalog + information_schema |
| 系统包 | 16个 DBMS_* | 完整 DBMS_* | 无系统包 | 无系统包（用扩展） |
| 动态视图 | SYS_ 动态虚表 | V$/GV$ | performance_schema | pg_stat_* |
| 全局/本地视图 | SYS_ / SYS_ALL_ | V$ / GV$ | 仅本地 | pg_stat_ / pg_stat_all_ |
| 查看表结构 | SYS_COLUMNS | DBA_TAB_COLUMNS | COLUMNS | pg_attribute |
| 查看索引 | SYS_INDEXES | DBA_INDEXES | STATISTICS | pg_index |
| 查看会话 | SYS_SESSIONS | V$SESSION | PROCESSLIST | pg_stat_activity |
| 查看锁 | SYS_ROW_LOCKS 等 | V$LOCK | INNODB_LOCKS | pg_locks |

## 参考文档

- [系统表](skills/xugudb-data-dictionary/references/system-tables.md) -- 系统文件表、系统堆表、系统虚表完整列表
- [系统视图](skills/xugudb-data-dictionary/references/system-views.md) -- ALL/DBA/USER/SSO/AUDIT 五类视图
- [系统包](skills/xugudb-data-dictionary/references/system-packages.md) -- 16个系统包功能与方法参考
