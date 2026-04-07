# C# (.NET) 驱动连接管理

## 连接字符串格式

分号分隔的键值对格式：
```
IP=<地址>;PORT=<端口>;DB=<数据库>;USER=<用户>;PWD=<密码>;CHAR_SET=<字符集>
```

## 连接参数

| 参数 | 说明 | 必填 |
|------|------|------|
| IP | 服务器 IP 地址 | 是（与 IPS 二选一） |
| IPS | 多 IP 负载均衡（逗号分隔） | 否 |
| PORT | 端口号（默认 5138） | 否 |
| DBNAME / DB | 数据库名 | 是 |
| USER | 用户名 | 是 |
| PASSWORD / PWD | 密码 | 是 |
| CHAR_SET | 字符集（GBK/UTF8） | 否 |
| USESSL | 是否启用 SSL（TRUE/FALSE） | 否 |

## 创建连接

### 方式一：先创建后设置

```csharp
XGConnection conn = new XGConnection();
conn.ConnectionString = "IP=127.0.0.1;PORT=5138;DB=SYSTEM;USER=SYSDBA;PWD=SYSDBA;CHAR_SET=GBK";
conn.Open();
// ... 操作
conn.Close();
```

### 方式二：构造函数传入

```csharp
string connStr = "IP=127.0.0.1;PORT=5138;DB=SYSTEM;USER=SYSDBA;PWD=SYSDBA;CHAR_SET=GBK";
XGConnection conn = new XGConnection(connStr);
conn.Open();
```

## 负载均衡

通过 IPS 参数配置多个服务器 IP：

```csharp
conn.ConnectionString = "IPS=192.168.1.10,192.168.1.11;DB=SYSTEM;USER=SYSDBA;PWD=SYSDBA;PORT=5138;CHAR_SET=GBK";
```

## SSL 加密连接

```csharp
conn.ConnectionString = "IP=127.0.0.1;PORT=5138;DB=SYSTEM;USER=SYSDBA;PWD=SYSDBA;CHAR_SET=GBK;USESSL=TRUE";
```

## 切换数据库

```csharp
conn.ChangeDatabase("TEST");
```

## Schema 信息查询

```csharp
// 获取所有 Schema 信息
DataTable schema = conn.GetSchema();

// 获取指定集合的 Schema
DataTable tables = conn.GetSchema("Tables");

// 带过滤条件
string[] restrictions = new string[4];
restrictions[2] = "Course";
DataTable filtered = conn.GetSchema("Tables", restrictions);
```

## XGConnection 方法参考

| 方法 | 说明 |
|------|------|
| `Open()` | 打开连接 |
| `Close()` | 关闭连接 |
| `BeginTransaction()` | 开始事务 |
| `BeginTransaction(IsolationLevel)` | 指定隔离级别开始事务 |
| `ChangeDatabase(dbName)` | 切换数据库 |
| `CreateCommand()` | 创建 XGCommand |
| `GetSchema()` | 查询数据库 Schema |

## 注意事项

- 使用完毕应调用 `conn.Close()` 或 `conn.Dispose()` 释放连接
- 推荐使用 `using` 语句自动管理连接生命周期
- 多线程环境下每个线程应使用独立的 XGConnection
