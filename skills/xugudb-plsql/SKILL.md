---
name: 虚谷数据库 PL/SQL 编程
name_for_command: xugudb-plsql
description: |
  虚谷数据库 PL/SQL 编程完整参考：基本语法、数据类型、控制结构、游标、异常处理、
  存储过程/函数、包、触发器、自定义类型、动态 SQL、批量操作与性能优化。
  高度兼容 Oracle PL/SQL，包含与 Oracle/MySQL/PostgreSQL 的语法对比和迁移指南。
---

# 虚谷数据库 PL/SQL 编程

虚谷数据库 PL/SQL 高度兼容 Oracle PL/SQL 语法，支持完整的过程化编程能力，是从 Oracle 迁移到虚谷的重要兼容性保障。

## PL/SQL 基础

虚谷 PL/SQL 的基础语法元素：程序块结构（DECLARE/BEGIN/EXCEPTION/END）、标识符命名规则、分隔符、注释、数据类型（标量类型、复合类型、属性类型）、变量与常量声明、赋值操作、控制结构（IF/CASE/LOOP/WHILE/FOR/FORALL）、游标（隐式/显式/游标变量）、异常处理（预定义/自定义/非预定义异常）、动态 SQL（EXECUTE IMMEDIATE）、批量操作（BULK COLLECT/FORALL）。

> 详细参考：[plsql-basics](skills/xugudb-plsql/references/plsql-basics.md)

## 程序封装

PL/SQL 程序封装对象：存储过程（Procedure）、存储函数（Function）、包（Package，包规范+包体）、触发器（Trigger）、自定义类型（OBJECT，含成员函数/成员过程/静态函数/静态过程）。每种对象包含创建、修改、删除、查看、执行、反汇编等完整操作。

> 详细参考：[plsql-programs](skills/xugudb-plsql/references/plsql-programs.md)

## 优化策略

PL/SQL 性能优化：SQL 查询优化（索引、JOIN 替代子查询、EXISTS 替代 IN 等）、PL/SQL 代码优化（BULK COLLECT/FORALL 批量操作、合适的数据类型、控制结构选择、避免不必要的异常处理）、数据库设计与索引优化（规范化、分区表、索引维护）。

> 详细参考：[plsql-optimization](skills/xugudb-plsql/references/plsql-optimization.md)

## 与 Oracle/MySQL/PG 关键差异

| 特性 | XuguDB | Oracle | MySQL | PostgreSQL |
|------|--------|--------|-------|------------|
| PL/SQL 兼容性 | 高度兼容 Oracle PL/SQL | 原生 PL/SQL | 不支持(用存储过程) | PL/pgSQL(语法不同) |
| 程序块结构 | DECLARE/BEGIN/EXCEPTION/END | 相同 | BEGIN/END | DECLARE/BEGIN/EXCEPTION/END |
| 匿名块 | 支持，末尾用 / 标识结束 | 支持，末尾用 / | 不支持 | 支持(DO $$ ... $$) |
| 包(Package) | 支持(包规范+包体) | 支持 | 不支持 | 不支持(用Schema模拟) |
| DBMS_OUTPUT | 支持 PUT_LINE | 支持 PUT_LINE/PUT | 不支持 | 需扩展 |
| SEND_MSG | 虚谷独有系统函数 | 不支持 | 不支持 | 不支持 |
| BULK COLLECT | 支持 | 支持 | 不支持 | 不支持(用ARRAY) |
| FORALL | 支持 | 支持 | 不支持 | 不支持 |
| 游标 FOR 循环 | 支持(显式+隐式) | 支持 | 支持(语法不同) | 支持(语法不同) |
| SYS_REFCURSOR | 支持 | 支持 | 不支持 | 支持(REFCURSOR) |
| 触发器 | BEFORE/AFTER/INSTEAD OF | 相同 | BEFORE/AFTER | BEFORE/AFTER/INSTEAD OF |
| OBJECT 类型 | 支持(含成员/静态方法) | 支持 | 不支持 | 支持(类型有限) |
| 异常处理 | EXCEPTION WHEN...THEN | 相同 | DECLARE HANDLER | EXCEPTION WHEN...THEN |
| RAISE/THROW | 两种关键字均支持 | RAISE | SIGNAL | RAISE |
| EXECUTE IMMEDIATE | 支持(含 USING/INTO) | 支持 | PREPARE+EXECUTE | EXECUTE(语法不同) |
| 自治事务 | PRAGMA AUTONOMOUS_TRANSACTION | 相同 | 不支持 | 不支持(需dblink) |
| %TYPE/%ROWTYPE | 支持 | 支持 | 不支持 | 支持 |
| 集合类型 | VARRAY/TABLE/ITABLE | VARRAY/TABLE/ITABLE | 不支持 | ARRAY(有限) |

## 参考文档

- [PL/SQL 基础](skills/xugudb-plsql/references/plsql-basics.md) -- 语法结构、数据类型、声明、控制结构、游标、异常处理、动态 SQL、批量操作
- [程序封装](skills/xugudb-plsql/references/plsql-programs.md) -- 存储过程/函数、包、触发器、自定义类型
- [优化策略](skills/xugudb-plsql/references/plsql-optimization.md) -- SQL 优化、PL/SQL 代码优化、数据库设计与索引优化
