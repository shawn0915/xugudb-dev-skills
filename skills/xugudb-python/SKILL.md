---
name: 虚谷数据库 Python 开发
name_for_command: xugudb-python
description: |
  虚谷数据库 Python 驱动（xgcondb）开发完整指南：驱动安装与环境配置、连接管理（connect/SSL/多IP）、
  CRUD 操作（execute/executemany/executebatch/fetch）、参数化查询（元组/列表/字典）、
  事务控制（begin/commit/rollback/autocommit）、游标属性、存储过程调用、大对象处理、常见问题排查。
---

# 虚谷数据库 Python 开发

虚谷数据库 Python 驱动接口（xgcondb）为 Python 2.7 及以上版本提供数据库访问能力，基于 Python Database API Specification v2.0 协议编写。

## 驱动安装与环境配置

xgcondb 驱动通过产品包或在线下载获取，支持 Windows（x86_64）和 Linux（x86_64 / ARM64）。安装方式为将 xgcondb 目录放置到项目同级目录，Linux 还需将 libxugusql.so 放入系统动态库目录。

> 详细参考：[installation](skills/xugudb-python/references/installation.md)

## 连接管理

通过 `xgcondb.connect()` 创建连接对象，支持基础连接、SSL 加密连接、多 IP 高可用连接。连接对象提供事务控制、判活重连、数据库切换等方法。

> 详细参考：[connection](skills/xugudb-python/references/connection.md)

## CRUD 操作

游标对象（cursor）提供 execute/executemany/executebatch 执行 SQL，支持 ? 占位符和 :name 字典占位符两种参数化方式。查询结果通过 fetchone/fetchmany/fetchall 提取。

> 详细参考：[crud-operations](skills/xugudb-python/references/crud-operations.md)

## 常见问题

Linux 环境下 libxugusql.so 动态库路径配置、Python 版本兼容性等常见问题及解决方案。

> 详细参考：[troubleshooting](skills/xugudb-python/references/troubleshooting.md)
