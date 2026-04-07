---
name: 虚谷数据库 SQL 语法参考
name_for_command: xugudb-sql
description: |
  虚谷数据库 SQL 语法完整参考：DDL/DML/TCL、数据类型、查询、运算符、表达式。
  包含与 Oracle/MySQL/PostgreSQL 的语法对比和迁移指南。
---

# 虚谷数据库 SQL 语法参考

虚谷数据库兼容 SQL-92/99/2003 标准，同时支持 Oracle/MySQL/PostgreSQL 三种兼容模式。

## SQL 基础

虚谷 SQL 的基础语法元素：关键字（保留字/非保留字）、标识符（命名规则、引号标识符）、字面量（文本/数值/日期/布尔等）、字符集（UTF-8/GB18030/GBK 等 10 种）、类型转换（CAST / :: 语法）。

> 详细参考：[sql-basics](skills/xugudb-sql/references/sql-basics.md)

## 数据类型

| 类别 | 类型 |
|------|------|
| 数值 | TINYINT, SMALLINT, INTEGER, BIGINT, NUMERIC/DECIMAL/NUMBER, FLOAT, DOUBLE |
| 字符 | CHAR, VARCHAR, BINARY |
| 日期时间 | DATE, TIME, DATETIME, TIMESTAMP (±TZ), INTERVAL (13种) |
| 大对象 | BLOB, CLOB |
| 特殊类型 | BOOLEAN, BIT/VARBIT, JSON, XML, GUID, ARRAY, 自定义类型(OBJECT/VARRAY/TABLE) |

> 详细参考：[data-types](skills/xugudb-sql/references/data-types.md)

## DDL（数据定义语言）

- **库管理**：CREATE/ALTER/DROP DATABASE
- **模式管理**：CREATE/ALTER/DROP SCHEMA
- **表管理**：CREATE TABLE（IF NOT EXISTS、IDENTITY 自增列、ENCRYPT 加密、临时表）、ALTER TABLE、DROP TABLE、TRUNCATE TABLE
- **分区表**：范围/列表/哈希分区、二级分区、自动扩展分区
- **索引管理**：BTREE/BITMAP、LOCAL/GLOBAL 分区索引、函数索引、在线创建
- **视图管理**：CREATE VIEW（FORCE、WITH CHECK OPTION）
- **序列管理**：CREATE SEQUENCE（CACHE/CYCLE/IDENTITY）

> 详细参考：[ddl](skills/xugudb-sql/references/ddl.md)

## DML（数据操作语言）

| 语句 | 说明 |
|------|------|
| INSERT | 标准插入、多表插入(INSERT ALL/FIRST)、REPLACE、INSERT IGNORE |
| UPDATE | 单表/关联更新、WHERE CURRENT OF 游标更新 |
| DELETE | 条件删除、WHERE CURRENT OF、RETURNING |
| MERGE INTO | Oracle 风格的 UPSERT 操作 |
| SET/SHOW | 系统参数、会话参数、自定义变量 |
| EXECUTE/PREPARE | 预编译 SQL |

**虚谷独有**：无逗号多行插入语法 `VALUES(1,'a')(2,'b')(3,'c')`

> 详细参考：[dml](skills/xugudb-sql/references/dml.md)

## 查询

- **SELECT 基础**：列选择、Hint 提示
- **FROM 子句**：别名、PIVOT/UNPIVOT 行列转换
- **WHERE 子句**：比较、LIKE、BETWEEN、IN、EXISTS
- **GROUP BY**：ROLLUP/CUBE/GROUPING SETS 扩展
- **ORDER BY**：NULLS FIRST/LAST、SIBLINGS
- **WITH (CTE)**：公共表表达式、WITH FUNCTION/PROCEDURE
- **连接查询**：INNER/LEFT/RIGHT/FULL/CROSS/NATURAL JOIN
- **子查询**：标量/IN/EXISTS/ANY/ALL
- **集合查询**：UNION/INTERSECT/MINUS/EXCEPT
- **层次查询**：CONNECT BY/START WITH/PRIOR（兼容 Oracle）
- **分析函数**：OVER/PARTITION BY/窗口帧(ROWS/RANGE)
- **结果集限定**：LIMIT/OFFSET、TOP、DISTINCT
- **执行计划**：EXPLAIN

> 详细参考：[query](skills/xugudb-sql/references/query.md)

## TCL（事务控制语言）

| 语句 | 说明 |
|------|------|
| BEGIN/START TRANSACTION | 启动事务 |
| COMMIT | 提交事务 |
| ROLLBACK / ABORT | 回滚事务（ABORT 等效 ROLLBACK） |
| SAVEPOINT | 设置保存点 |
| RELEASE SAVEPOINT | 删除保存点 |
| ROLLBACK TO SAVEPOINT | 回滚到保存点 |

> 详细参考：[tcl](skills/xugudb-sql/references/tcl.md)

## 运算符

| 类别 | 运算符 |
|------|--------|
| 算术 | `+` `-` `*` `/` `%` |
| 比较 | `=` `<>` `!=` `>` `<` `>=` `<=` |
| 逻辑 | `AND` `OR` `NOT` |
| 连接 | `\|\|` |
| 赋值 | `=` |
| 位 | `&` `\|` `^` `~` `<<` `>>` |
| JSON | `->` `->>` |
| 日期 | `+` `-`（日期加减数字/间隔） |
| 几何 | `+` `-` `*` `/` `@-@` `@@` `##` `<->` `&&` 等 |

> 详细参考：[operators](skills/xugudb-sql/references/operators.md)

## 表达式

常量、标识符、算术、函数调用、类型转换（CAST / ::）、条件（CASE WHEN）、逻辑、子查询表达式。

> 详细参考：[expressions](skills/xugudb-sql/references/expressions.md)

## 与 Oracle/MySQL/PG 关键差异

| 特性 | XuguDB | Oracle | MySQL | PostgreSQL |
|------|--------|--------|-------|------------|
| 兼容模式 | 三种模式可切换 | - | - | - |
| 多行插入 | `VALUES(1)(2)(3)` | 不支持 | `VALUES(1),(2),(3)` | `VALUES(1),(2),(3)` |
| 类型转换 | `CAST` + `::` | `CAST` | `CAST` | `CAST` + `::` |
| 层次查询 | CONNECT BY | CONNECT BY | 不支持(用CTE) | 不支持(用CTE) |
| 结果集限定 | LIMIT/OFFSET + TOP | ROWNUM/FETCH | LIMIT/OFFSET | LIMIT/OFFSET |
| 集合操作 | MINUS + EXCEPT | MINUS | 不支持MINUS | EXCEPT |
| 序列 | CREATE SEQUENCE + IDENTITY | CREATE SEQUENCE | AUTO_INCREMENT | CREATE SEQUENCE + IDENTITY |
| 分区 | RANGE/LIST/HASH + 二级 | RANGE/LIST/HASH | RANGE/LIST/HASH | RANGE/LIST/HASH |
| MERGE INTO | 支持 | 支持 | 不支持(用REPLACE) | 不支持(用ON CONFLICT) |
| PIVOT/UNPIVOT | 支持 | 支持 | 不支持 | 不支持(用crosstab) |
| `\|\|` 连接 | NULL 视为空串 | NULL 视为空串 | 需设置模式 | NULL→NULL |

## 参考文档

- [SQL 基础](skills/xugudb-sql/references/sql-basics.md) — 关键字、标识符、字面量、字符集、类型转换
- [数据类型](skills/xugudb-sql/references/data-types.md) — 数值、字符、日期、大对象、JSON、数组等
- [DDL](skills/xugudb-sql/references/ddl.md) — 库/模式/表/索引/视图/序列管理
- [DML](skills/xugudb-sql/references/dml.md) — INSERT/UPDATE/DELETE/MERGE/SET/SHOW
- [查询](skills/xugudb-sql/references/query.md) — SELECT/JOIN/子查询/CTE/分析函数/EXPLAIN
- [TCL](skills/xugudb-sql/references/tcl.md) — 事务控制：BEGIN/COMMIT/ROLLBACK/SAVEPOINT
- [运算符](skills/xugudb-sql/references/operators.md) — 算术/比较/逻辑/连接/JSON/位/几何
- [表达式](skills/xugudb-sql/references/expressions.md) — 常量/类型转换/CASE/逻辑/子查询
