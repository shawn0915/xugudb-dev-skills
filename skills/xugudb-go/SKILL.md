---
name: 虚谷数据库 Go 驱动开发
name_for_command: xugudb-go
description: |
  XuguDB Go 驱动开发完整指南：驱动安装与环境搭建、连接管理（database/sql）、
  CRUD 操作（Exec/Query/Prepare）、事务控制、大对象处理、多结果集、接口参考。
  适用于使用 Go 语言连接和操作虚谷数据库的开发场景。
tags: xugudb, go, golang, driver, database/sql, crud, transaction
---

# 虚谷数据库 Go 驱动开发

XuguDB Go 驱动（go-xugu-driver）是虚谷数据库的官方 Go 语言驱动程序，基于 Go 标准库 `database/sql` 接口实现，支持 Go 语言的所有标准数据库操作。

## 驱动安装

安装方式：
```bash
go get gitee.com/XuguDB/go-xugu-driver@latest
```

导入方式（匿名导入注册驱动）：
```go
import (
    _ "gitee.com/XuguDB/go-xugu-driver"
    "database/sql"
)
```

**环境要求：**
- Go 语言环境（go 命令可用）
- XuguDB Go Driver 包（CPU 架构需匹配）

> 详细参考：[安装与环境搭建](references/installation.md)

## 连接管理

驱动名称：`"xugu"`

连接字符串格式（分号分隔的键值对）：
```
IP=127.0.0.1;DB=SYSTEM;User=SYSDBA;PWD=SYSDBA;Port=5138;AUTO_COMMIT=on;CHAR_SET=UTF8
```

连接示例：
```go
db, err := sql.Open("xugu", "IP=127.0.0.1;DB=SYSTEM;User=SYSDBA;PWD=SYSDBA;Port=5138;CHAR_SET=UTF8")
if err != nil {
    log.Fatal(err)
}
defer db.Close()
```

**关键连接参数：**

| 参数 | 说明 |
|------|------|
| IP | 服务器 IP 地址 |
| IPS | 多 IP 负载均衡（逗号分隔） |
| PORT | 端口号（默认 5138） |
| DBNAME/DB | 数据库名 |
| USER/User | 用户名 |
| PASSWORD/PWD | 密码 |
| CHAR_SET | 字符集（建议 UTF8） |
| AUTO_COMMIT | 自动提交（on/off） |

**连接池配置：**
```go
db.SetMaxOpenConns(10)          // 最大打开连接数
db.SetMaxIdleConns(5)           // 最大空闲连接数
db.SetConnMaxLifetime(30 * time.Minute)  // 连接最大生存时间
```

> 详细参考：[连接管理](references/connection.md)

## CRUD 操作

### 执行 DDL/DML（Exec）

```go
// DDL
_, err := db.Exec("CREATE TABLE test(c1 INT, c2 VARCHAR)")

// DML
res, err := db.Exec("INSERT INTO test VALUES(1, 'hello')")
affected, _ := res.RowsAffected()
```

### 查询（Query）

```go
rows, err := db.Query("SELECT c1, c2 FROM test")
if err != nil {
    log.Fatal(err)
}
defer rows.Close()

for rows.Next() {
    var c1 int
    var c2 string
    err = rows.Scan(&c1, &c2)
    // 处理数据
}
```

### 预编译（Prepare）

```go
stmt, err := db.Prepare("INSERT INTO test VALUES(?, ?)")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close()

_, err = stmt.Exec(1, "xugusql")
```

### 事务（Transaction）

```go
tx, err := db.Begin()
if err != nil {
    log.Fatal(err)
}

_, err = tx.Exec("INSERT INTO test VALUES(1, 'data')")
if err != nil {
    tx.Rollback()
    return
}

err = tx.Commit()
```

> 详细参考：[CRUD 操作](references/crud-operations.md)

## 高级特性

- **大对象（LOB）处理** — BLOB/CLOB 的插入与读取（通过 `[]byte` 类型）
- **多结果集** — `rows.NextResultSet()` 遍历多条 SQL 返回的多个结果集
- **参数化查询** — `?` 占位符防 SQL 注入
- **负载均衡** — 通过 `IPS` 参数配置多 IP
- **连接检测** — `db.Ping()` / `db.PingContext()` 检查连接可用性

> 详细参考：[CRUD 操作](references/crud-operations.md)

## 与其他数据库 Go 驱动的差异

| 特性 | XuguDB | MySQL (go-sql-driver) | PostgreSQL (pgx) |
|------|--------|----------------------|------------------|
| 驱动名称 | `"xugu"` | `"mysql"` | `"pgx"` |
| 连接串格式 | 分号键值对 | DSN 格式 | URL 格式 |
| 占位符 | `?` | `?` | `$1, $2` |
| 多结果集 | 支持 | 支持 | 不支持 |
| 导入路径 | `gitee.com/XuguDB/go-xugu-driver` | `github.com/go-sql-driver/mysql` | `github.com/jackc/pgx/v5/stdlib` |

## 工作流程

当用户咨询 Go 驱动开发相关问题时：

1. 确定问题类别（安装配置 / 连接管理 / CRUD 操作 / 事务 / 高级特性）
2. 提供基于 `database/sql` 标准接口的代码示例
3. 标注虚谷特有的连接参数和语法差异
4. 对 LOB 等特殊类型操作给出完整示例

## 参考文档

- [安装与环境搭建](references/installation.md) — Go 环境配置、驱动安装、项目初始化
- [连接管理](references/connection.md) — 连接字符串、连接池、连接参数、负载均衡
- [CRUD 操作](references/crud-operations.md) — Exec/Query/Prepare/事务/LOB/多结果集/完整接口参考
