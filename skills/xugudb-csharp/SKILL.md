---
name: 虚谷数据库 C# (.NET) 驱动开发
name_for_command: xugudb-csharp
description: |
  XuguDB C# (.NET) 驱动开发完整指南：XuguClient 驱动安装（Windows/Linux）、
  ADO.NET 接口（XGConnection/XGCommand/XGDataReader/XGDataAdapter）、
  参数化查询、事务控制、存储过程调用、大对象处理、DataSet 操作、数据类型映射。
  适用于使用 C#/.NET 连接和操作虚谷数据库的开发场景。
tags: xugudb, csharp, dotnet, ado-net, xuguclient, driver
---

# 虚谷数据库 C# (.NET) 驱动开发

XuguClient 是虚谷数据库的官方 .NET 驱动程序，支持 .NET Framework 和 .NET Core，实现标准 ADO.NET API 接口。

## 驱动安装

### Windows
1. 将 `XuguClient.dll` 和 `xugusql.dll` 放入项目
2. 在项目中添加 `XuguClient.dll` 引用
3. 将 `xugusql.dll` 放到 BIN 目录（与 exe 同目录）

### Linux
1. 将 `XuguClient.dll` 和 `libxugusql.so` 放到与 exe 同级目录

**平台要求：**
- Windows: x86, 64-bit（Windows 10+）
- Linux: x86 64-bit 或 ARM 64-bit

> 详细参考：[安装与环境搭建](references/installation.md)

## 连接管理

连接字符串格式（分号分隔键值对）：
```
IP=192.168.1.1;PORT=5138;DB=SYSTEM;USER=SYSDBA;PWD=SYSDBA;CHAR_SET=GBK
```

```csharp
XGConnection conn = new XGConnection();
conn.ConnectionString = "IP=127.0.0.1;PORT=5138;DB=SYSTEM;USER=SYSDBA;PWD=SYSDBA;CHAR_SET=GBK";
conn.Open();
// ... 操作
conn.Close();
```

**连接参数：**

| 参数 | 说明 |
|------|------|
| IP | 服务器 IP 地址 |
| IPS | 多 IP 负载均衡（逗号分隔） |
| PORT | 端口号（默认 5138） |
| DBNAME/DB | 数据库名 |
| USER | 用户名 |
| PASSWORD/PWD | 密码 |
| CHAR_SET | 字符集（GBK/UTF8） |
| USESSL | 是否启用 SSL（TRUE/FALSE） |

> 详细参考：[连接管理](references/connection.md)

## CRUD 操作

### 执行 DML/DDL

```csharp
XGCommand cmd = conn.CreateCommand();
cmd.CommandText = "INSERT INTO test VALUES(1, 'hello')";
int affected = cmd.ExecuteNonQuery();
```

### 查询 — XGDataReader

```csharp
cmd.CommandText = "SELECT * FROM test";
XGDataReader reader = cmd.ExecuteReader();
while (reader.Read()) {
    Console.WriteLine($"{reader.GetInt32(0)}, {reader.GetString(1)}");
}
reader.Close();
```

### 查询 — DataSet/DataAdapter

```csharp
XGDataAdapter adapter = new XGDataAdapter("SELECT * FROM test", conn);
DataSet ds = new DataSet();
adapter.Fill(ds, "test");

foreach (DataRow row in ds.Tables["test"].Rows) {
    Console.WriteLine(row["NAME"]);
}
```

### 参数化查询

```csharp
cmd.CommandText = "INSERT INTO test(pid, pname) VALUES(?, ?)";
cmd.Parameters.Add("pid", XGDbType.Int).Value = 1;
cmd.Parameters.Add("pname", XGDbType.VarChar).Value = "test";
cmd.ExecuteNonQuery();
```

### 事务控制

```csharp
XGTransaction trans = conn.BeginTransaction();
cmd.Transaction = trans;
try {
    cmd.CommandText = "INSERT INTO test VALUES(1, 'data')";
    cmd.ExecuteNonQuery();
    trans.Commit();
} catch {
    trans.Rollback();
}
```

> 详细参考：[CRUD 操作](references/crud-operations.md)

## 高级特性

- **存储过程/函数调用** — `CommandType.StoredProcedure` + IN/OUT/INOUT 参数
- **包调用** — `PACK_NAME.FUNC_NAME` 格式
- **大对象处理** — XGBlob 分块写入/读取
- **RefCursor** — 通过 XGRefCursor.GetDataReader() 获取结果集
- **服务端游标** — `Use_Server_Cursor = true`
- **DataSet 更新** — DataAdapter.Update() 批量更新
- **Schema 查询** — GetSchema() 查询数据库元信息
- **SSL 加密** — USESSL=TRUE 启用

> 详细参考：[CRUD 操作](references/crud-operations.md)

## XGDbType 数据类型映射

| XGDbType | C# 类型 | 说明 |
|----------|---------|------|
| Bool | System.Boolean | 布尔 |
| TinyInt | System.SByte | 微整数 |
| SmallInt | System.Int16 | 短整数 |
| Int | System.Int32 | 整数 |
| BigInt | System.Int64 | 长整数 |
| Real | System.Single | 单精度浮点 |
| Double | System.Double | 双精度浮点 |
| Numeric | System.Decimal | 定点数 |
| Char/VarChar | System.String | 字符串 |
| DateTime/Date/Time | System.DateTime | 日期时间 |
| Binary/VarBinary | System.Byte[] | 二进制 |
| LongVarChar | System.String | 大文本(CLOB) |
| LongVarBinary | System.Byte[] | 大二进制(BLOB) |
| RefCursor | XGRefCursor | 游标引用 |
| Guid | System.Guid | 全局唯一标识 |

## 与其他数据库 .NET 驱动的差异

| 特性 | XuguDB (XuguClient) | SQL Server (SqlClient) | MySQL (MySqlConnector) |
|------|---------------------|----------------------|----------------------|
| 连接类 | XGConnection | SqlConnection | MySqlConnection |
| 命令类 | XGCommand | SqlCommand | MySqlCommand |
| 参数占位符 | `?` | `@param` | `@param` |
| 参数类 | XGParameters | SqlParameter | MySqlParameter |
| 数据类型枚举 | XGDbType | SqlDbType | MySqlDbType |

## 工作流程

当用户咨询 C# 驱动开发相关问题时：

1. 确定问题类别（安装配置 / 连接管理 / CRUD / 事务 / 存储过程 / 高级特性）
2. 提供基于 ADO.NET 标准接口的代码示例
3. 标注虚谷特有的连接参数、参数化语法（`?` 占位符）和类型映射
4. 对存储过程和 LOB 操作给出完整示例

## 参考文档

- [安装与环境搭建](references/installation.md) — Windows/Linux 环境配置、驱动安装
- [连接管理](references/connection.md) — 连接字符串、连接参数、SSL、负载均衡
- [CRUD 操作](references/crud-operations.md) — 增删改查、事务、存储过程、LOB、DataSet、完整接口参考
