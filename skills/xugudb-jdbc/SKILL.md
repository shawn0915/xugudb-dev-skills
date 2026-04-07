---
name: 虚谷数据库 JDBC 开发
name_for_command: xugudb-jdbc
description: |
  虚谷数据库 Java JDBC 驱动开发完整指南：驱动安装与配置、连接管理与连接池、
  CRUD 操作（Statement/PreparedStatement/CallableStatement）、事务控制、
  批量操作、大对象处理、负载均衡、数据类型映射、SSL 加密传输、常见异常排查。
---

# 虚谷数据库 JDBC 开发

XuguDB-JDBC 是虚谷数据库的官方 Java 驱动程序，以 JDBC 4.2 协议为标准，已实现 985 个标准接口中的 858 个（实现率超 87%）。支持 JDBC 3.0 功能，JDK 1.8 及以上版本。

## 驱动安装

三种安装方式：
- **Maven** — `com.xugudb:xugu-jdbc:版本号`
- **Gradle** — `implementation("com.xugudb:xugu-jdbc:版本号")`
- **手动导入** — 将 `cloudjdbc-*.*.*.jar` 添加到项目 classpath

驱动类名：`com.xugu.cloudjdbc.Driver`

> 详细参考：[installation](skills/xugudb-jdbc/references/installation.md)

## 连接管理

URL 格式：`jdbc:xugu://serverIP:portNumber/databaseName[?property=value[&property=value]]`

连接方式：
| 方式 | 说明 |
|------|------|
| DriverManager | 三种重载：URL 内嵌凭据、分离参数、Properties 对象 |
| DataSource | 通过 `XgDataSource` 配置数据源属性 |
| 连接池 | 通过 `XgConnectionPoolDataSource` 管理连接池 |

关键连接参数：`charset`、`autoCommit`、`isoLevel`、`recv_mode`、`connect_timeout`、`useSSL`、`ips`（负载均衡）等。

> 详细参考：[connection](skills/xugudb-jdbc/references/connection.md)

## CRUD 操作与事务

| 对象 | 用途 |
|------|------|
| Statement | 执行静态 SQL 语句（DDL/DML/查询） |
| PreparedStatement | 预编译 SQL，参数化查询，防 SQL 注入，批量操作 |
| CallableStatement | 调用存储过程/函数，支持 IN/OUT/INOUT 参数 |

事务控制：
- `setAutoCommit(false)` 开启手动事务
- `commit()` / `rollback()` 提交或回滚
- `setSavepoint()` 设置回滚点，支持部分回滚

> 详细参考：[crud-operations](skills/xugudb-jdbc/references/crud-operations.md)

## 高级特性

- **大对象处理** — BLOB/CLOB 的插入与读取（流式、字符串方式）
- **结果集更新** — 可更新 ResultSet（需 `return_rowid=true` + `CONCUR_UPDATABLE`）
- **服务端游标** — `recv_mode=2` 模式下的大数据量分批读取
- **多结果集** — 一次发送多条 SQL 并依次处理返回结果
- **自增键获取** — `getGeneratedKeys()` 获取自动生成的主键值
- **负载均衡** — URL/Properties/XML 三种多 IP 配置方式
- **SSL 加密** — 通过 `ssl=ssl` 参数启用传输加密

> 详细参考：[advanced](skills/xugudb-jdbc/references/advanced.md)

## 常见问题

常见错误码分类：
- **E500xx** — 连接、参数、类型转换、LOB 操作错误
- **E510xx** — 结果集游标、预处理语句、事务隔离级别错误
- **E520xx** — 元数据查询、数据插入范围错误
- **E530xx** — XA 事务错误
- **E550xx** — 未支持方法、IO 编码错误

> 详细参考：[troubleshooting](skills/xugudb-jdbc/references/troubleshooting.md)

## 参考文档

- [驱动安装与配置](skills/xugudb-jdbc/references/installation.md) — Maven/Gradle/手动导入、环境配置
- [连接管理](skills/xugudb-jdbc/references/connection.md) — 连接字符串、连接池、连接参数
- [CRUD 操作](skills/xugudb-jdbc/references/crud-operations.md) — Statement/PreparedStatement/事务/批量
- [高级特性](skills/xugudb-jdbc/references/advanced.md) — LOB、存储过程、游标、负载均衡、SSL
- [常见问题](skills/xugudb-jdbc/references/troubleshooting.md) — 错误码与排查指南
