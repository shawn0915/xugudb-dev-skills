---
name: 虚谷数据库对象管理
name_for_command: xugudb-objects
description: |
  XuguDB 数据库对象管理完整指南：表（创建/修改/分区/临时表）、索引、视图、
  约束、序列、存储过程/函数、触发器、包、同义词、DBLink、用户/角色/模式、
  定时作业、加密机、锁表等。适用于数据库设计、DDL 操作和对象管理。
tags: xugudb, objects, table, index, view, constraint, sequence, trigger, schema
---

# 虚谷数据库对象管理

## 概述

虚谷数据库支持丰富的数据库对象类型，以下是完整的对象分类：

| 对象类型 | 说明 | DDL 关键字 |
|----------|------|-----------|
| 库 | 数据库实例 | CREATE/DROP DATABASE |
| 模式 | 命名空间 | CREATE/ALTER/DROP SCHEMA |
| 用户 | 数据库用户 | CREATE/ALTER/DROP USER |
| 角色 | 权限集合 | CREATE/ALTER/DROP ROLE |
| 表 | 数据存储 | CREATE/ALTER/DROP/TRUNCATE TABLE |
| 视图 | 虚拟表 | CREATE/DROP VIEW |
| 索引 | 查询加速 | CREATE/DROP INDEX |
| 约束 | 数据完整性 | PRIMARY KEY/FOREIGN KEY/UNIQUE/CHECK/NOT NULL |
| 序列 | 自增序号 | CREATE/DROP SEQUENCE |
| 存储过程 | 过程逻辑 | CREATE PROCEDURE |
| 存储函数 | 函数逻辑 | CREATE FUNCTION |
| 触发器 | 事件响应 | CREATE TRIGGER |
| 包 | PL/SQL 模块 | CREATE PACKAGE |
| 同义词 | 对象别名 | CREATE SYNONYM |
| DBLink | 远程链接 | CREATE DATABASE LINK |
| 定时作业 | 调度任务 | DBMS_SCHEDULER |
| 加密机 | 数据加密 | CREATE ENCRYPTOR |

## 表管理快速参考

### 创建表

```sql
CREATE TABLE [IF NOT EXISTS] [schema.]table_name (
    column_name type [IDENTITY(start, step)] [PRIMARY KEY] [NOT NULL] [DEFAULT expr],
    ...
    [CONSTRAINT name constraint_type ...]
)
[PARTITION BY RANGE|LIST|HASH (col) PARTITIONS (...)]
[COMMENT 'description'];
```

### 临时表

```sql
-- 本地临时表（会话结束删除数据）
CREATE TEMP TABLE temp_t (id INT);

-- 全局临时表（需开启 support_global_tab）
SET support_global_tab ON;
CREATE GLOBAL TEMP TABLE g_temp (id INT) ON COMMIT PRESERVE ROWS;
```

### 修改表

```sql
ALTER TABLE t ADD COLUMN col_name type;
ALTER TABLE t ALTER COLUMN col_name SET DATA TYPE new_type;
ALTER TABLE t DROP COLUMN col_name [CASCADE];
ALTER TABLE t ADD CONSTRAINT ck CHECK (expr);
ALTER TABLE t ENABLE|DISABLE CONSTRAINT cons_name;
ALTER TABLE t DROP CONSTRAINT cons_name [KEEP|DROP INDEX];
ALTER TABLE t OWNER TO new_user;
```

### 分区表

```sql
-- 范围分区
CREATE TABLE t (id INT, date DATE)
PARTITION BY RANGE (date) PARTITIONS (
    p1 VALUES LESS THAN ('2025-01-01'),
    p2 VALUES LESS THAN ('2026-01-01'),
    p3 VALUES LESS THAN (MAXVALUES)
);

-- 列表分区
PARTITION BY LIST (status) PARTITIONS (
    p_active VALUES ('A', 'B'),
    p_other VALUES (OTHERVALUES)
);

-- 哈希分区
PARTITION BY HASH (id) PARTITIONS 4;
```

> 详细参考：[表管理](references/tables.md)

## 索引管理

```sql
-- 普通索引
CREATE INDEX idx_name ON table_name (col1, col2);

-- 唯一索引
CREATE UNIQUE INDEX idx_uk ON table_name (col);

-- 函数索引
CREATE INDEX idx_func ON table_name (UPPER(col));

-- IF NOT EXISTS
CREATE INDEX IF NOT EXISTS idx_name ON table_name (col);

-- 删除索引
DROP INDEX idx_name;
```

支持的索引类型：B+ 树索引（默认）、函数索引、唯一索引、复合索引。

> 详细参考：[索引管理](references/indexes.md)

## 视图管理

```sql
CREATE [OR REPLACE] [FORCE|NOFORCE] VIEW view_name [(col_list)]
AS select_stmt
[WITH READ ONLY | WITH CHECK OPTION]
[COMMENT 'desc'];

DROP VIEW [IF EXISTS] view_name;
```

> 详细参考：[视图与其他对象](references/views-and-others.md)

## 约束

| 约束类型 | 语法 | 说明 |
|----------|------|------|
| PRIMARY KEY | `col INT PRIMARY KEY` | 主键（唯一+非空） |
| UNIQUE | `CONSTRAINT uk UNIQUE(col)` | 唯一约束 |
| NOT NULL | `col INT NOT NULL` | 非空约束 |
| CHECK | `CHECK (col > 0)` | 检查约束 |
| FOREIGN KEY | `REFERENCES parent(col)` | 外键约束 |

约束可在创建表时（内联/外联）定义，也可通过 `ALTER TABLE` 添加/启用/禁用/删除。

> 详细参考：[表管理](references/tables.md)

## 序列

```sql
CREATE SEQUENCE seq_name
    [START WITH n] [INCREMENT BY n]
    [MINVALUE n | NOMINVALUE] [MAXVALUE n | NOMAXVALUE]
    [CACHE n | NOCACHE] [CYCLE | NOCYCLE];

-- 使用
SELECT seq_name.NEXTVAL FROM DUAL;
SELECT seq_name.CURRVAL FROM DUAL;

-- IDENTITY 列（自增）
CREATE TABLE t (id INT IDENTITY(1,1) PRIMARY KEY);
```

> 详细参考：[视图与其他对象](references/views-and-others.md)

## 用户与角色

```sql
-- 创建用户
CREATE USER user_name IDENTIFIED BY 'password'
    [DEFAULT ROLE role_name]
    [VALID UNTIL 'datetime']
    [ACCOUNT LOCK|UNLOCK];

-- 预置角色
-- DB_ADMIN（DBA）、DB_POLICY_ADMIN（SSO）、DB_AUDIT_ADMIN（审计）、PUBLIC

-- 创建自定义角色
CREATE ROLE role_name;
GRANT SELECT ON table TO role_name;
GRANT role_name TO user_name;
```

> 详细参考：[用户角色模式](references/users-roles-schemas.md)

## 工作流程

当用户咨询数据库对象管理问题时：

1. 确定操作类型（创建/修改/删除/查询）和对象类型
2. 提供完整的 DDL 语法和示例
3. 标注 IF NOT EXISTS、CASCADE 等安全选项
4. 对危险操作（DROP/TRUNCATE）明确提醒确认
5. 指出与 Oracle/MySQL 的语法差异

## 参考文档

- [表管理](references/tables.md) — 创建表、修改表、分区表、临时表、约束、删除/截断
- [索引管理](references/indexes.md) — B+树索引、唯一索引、函数索引、复合索引
- [视图与其他对象](references/views-and-others.md) — 视图、序列、同义词、DBLink、定时作业、加密机
- [用户角色模式](references/users-roles-schemas.md) — 用户管理、角色管理、模式管理、库管理
- [存储过程与触发器](references/procedures-triggers.md) — 存储过程、存储函数、触发器、包
