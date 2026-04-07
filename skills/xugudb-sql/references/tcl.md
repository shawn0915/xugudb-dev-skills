---
title: 虚谷数据库事务控制语言 (TCL)
description: BEGIN/START TRANSACTION、COMMIT、ROLLBACK、SAVEPOINT 事务控制语句语法及示例，与 Oracle/MySQL/PostgreSQL 事务控制对比
tags: xugudb, tcl, transaction, commit, rollback, savepoint, begin, abort
---

# 虚谷数据库事务控制语言 (TCL)

事务控制语言（Transaction Control Language，简称 TCL）是 SQL 的一个重要子集，专门用于管理数据库事务——即由多个操作组成的、需作为原子单元执行的逻辑工作单元。借助 TCL 命令，能够严格保证一组数据库操作的执行结果具有"全部成功或全部失败"的特性，以此确保事务满足 ACID 特性（原子性、一致性、隔离性、持久性）。

---

## 启动事务

开始事务标志着一个事务的开始，在此之后执行的所有 SQL 语句都将成为该事务的一部分，直到事务被提交或回滚。

### 语法格式

```
TransactionStmt ::= ('BEGIN' | 'START') (opt_trans?)
opt_trans ::= ('WORK' | 'TRAN' | 'TRANSACTION')
```

### 参数说明

- `BEGIN` 或 `START`：表示一个事务的开始
- `opt_trans`：可选关键字，用来更清晰地表达"启动事务"

以下四条命令等效：

```sql
BEGIN
BEGIN WORK
BEGIN TRAN
BEGIN TRANSACTION
```

同样，以下命令也等效：

```sql
START TRANSACTION
```

---

## 提交事务

提交事务用于将当前事务的所有修改永久保存到数据库，并释放事务持有的所有锁与结束当前事务。

### 语法格式

```
TransactionStmt ::= 'COMMIT' opt_trans
```

### 参数说明

- `COMMIT`：提交事务
- `opt_trans`：可选关键字，用来强调"提交事务"

以下四条命令等效：

```sql
COMMIT
COMMIT WORK
COMMIT TRAN
COMMIT TRANSACTION
```

---

## 回滚事务

回滚事务用于撤销当前事务的所有修改，并回滚到事务开始前的状态与释放事务持有的锁。

### 语法格式

```
TransactionStmt ::= 'ROLLBACK' (('TO' ('SAVEPOINT')? ColumnName) | opt_trans)
                   | 'ABORT' opt_trans
```

### 参数说明

- `ROLLBACK` 或 `ABORT`：功能完全等效，均会终止当前事务并回滚所有未提交的修改
- `TO SAVEPOINT <名称>`：回滚到指定保存点
- `opt_trans`：可选关键字，用来强调"回滚事务"

以下四条命令等效：

```sql
ROLLBACK
ROLLBACK WORK
ROLLBACK TRAN
ROLLBACK TRANSACTION
```

---

## 保存点

保存点用于在事务内部创建回滚标记点，且允许部分回滚而非整个事务。

### 设置保存点

```
TransactionStmt ::= 'SAVEPOINT' ColumnName
```

### 删除保存点

```
TransactionStmt ::= 'RELEASE' 'SAVEPOINT' ColumnName
```

### 参数说明

- `SAVEPOINT <名称>`：在当前事务中创建一个保存点
- `RELEASE SAVEPOINT <名称>`：删除指定的保存点

> **提示**：保存点作用范围只限于当前连接，如果想要回退到某一个保存点，需在提交事务之前。

---

## 完整示例

```sql
-- 创建测试表
CREATE TABLE tb_test(id INT PRIMARY KEY, val VARCHAR);

-- 示例1：开启事务并提交
BEGIN TRANSACTION;
INSERT INTO tb_test VALUES(1, 'aaa');
INSERT INTO tb_test VALUES(2, 'bbb');
COMMIT;

-- 验证测试表数据
SELECT * FROM tb_test;
-- +----+-----+
-- | ID | VAL |
-- +----+-----+
-- |  1 | aaa |
-- |  2 | bbb |
-- +----+-----+

-- 示例2：开启事务并回滚
START TRANSACTION;
INSERT INTO tb_test VALUES(3, 'ccc');
INSERT INTO tb_test VALUES(4, 'ddd');
ROLLBACK;

-- 验证测试表数据（回滚后数据未变）
SELECT * FROM tb_test;
-- +----+-----+
-- | ID | VAL |
-- +----+-----+
-- |  1 | aaa |
-- |  2 | bbb |
-- +----+-----+

-- 示例3：使用保存点进行部分回滚
BEGIN;
INSERT INTO tb_test VALUES(3, 'ccc');
INSERT INTO tb_test VALUES(4, 'ddd');

-- 设置保存点 S1
SAVEPOINT S1;

INSERT INTO tb_test VALUES(5, 'eee');
INSERT INTO tb_test VALUES(6, 'fff');

-- 回退到保存点 S1（撤销 id=5 和 id=6 的插入）
ROLLBACK TO S1;

COMMIT;

-- 验证测试表数据（仅 id=3 和 id=4 被保留）
SELECT * FROM tb_test;
-- +----+-----+
-- | ID | VAL |
-- +----+-----+
-- |  1 | aaa |
-- |  2 | bbb |
-- |  3 | ccc |
-- |  4 | ddd |
-- +----+-----+
```

---

## 与 Oracle/MySQL/PostgreSQL 事务控制对比

### 启动事务

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 显式启动 | `BEGIN` / `START TRANSACTION` | 不支持 `BEGIN`（BEGIN 用于 PL/SQL 块） | `START TRANSACTION` / `BEGIN` | `BEGIN` / `START TRANSACTION` |
| 隐式启动 | 支持 | 每条 SQL 自动开始事务 | 取决于 autocommit 设置 | 每条 SQL 自动开始事务 |
| `BEGIN WORK` | 支持 | 不支持 | 支持 | 支持 |
| `BEGIN TRAN` | 支持 | 不支持 | 不支持 | 不支持 |

### 提交事务

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| `COMMIT` | 支持 | 支持 | 支持 | 支持 |
| `COMMIT WORK` | 支持 | 支持 | 支持 | 支持 |
| `COMMIT TRAN` | 支持 | 不支持 | 不支持 | 不支持 |
| `COMMIT TRANSACTION` | 支持 | 不支持 | 不支持 | 不支持 |

### 回滚事务

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| `ROLLBACK` | 支持 | 支持 | 支持 | 支持 |
| `ABORT` | 支持（等效于 ROLLBACK） | 不支持 | 不支持 | 支持（等效于 ROLLBACK） |
| `ROLLBACK TO SAVEPOINT` | 支持 | 支持 | 支持 | 支持 |
| `ROLLBACK TO <名称>`（省略 SAVEPOINT 关键字） | 支持 | 支持 | 不支持（必须写 SAVEPOINT） | 支持 |

### 保存点

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| `SAVEPOINT <名称>` | 支持 | 支持 | 支持 | 支持 |
| `RELEASE SAVEPOINT <名称>` | 支持 | 不支持 | 支持 | 支持 |

### 迁移注意事项

- **从 Oracle 迁移**：Oracle 中 `BEGIN` 用于 PL/SQL 匿名块，不能用来启动事务；虚谷数据库中 `BEGIN` 既可启动事务也可用于 PL/SQL 块，需注意上下文区分。Oracle 不支持 `RELEASE SAVEPOINT`。
- **从 MySQL 迁移**：虚谷数据库额外支持 `BEGIN TRAN`、`COMMIT TRAN`、`ABORT` 等语法，MySQL 不支持。MySQL 中 DDL 语句会隐式提交事务，虚谷数据库行为需根据实际配置确认。
- **从 PostgreSQL 迁移**：两者事务控制语法高度兼容，虚谷数据库额外支持 `TRAN` 关键字变体。PostgreSQL 的 `ABORT` 在虚谷数据库中同样可用。
