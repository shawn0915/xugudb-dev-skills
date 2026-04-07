# 虚谷数据库 JDBC 连接管理

## 连接 URL 格式

```
jdbc:xugu://serverIP:portNumber/databaseName[?property=value[&property=value]]
```

- `jdbc:xugu://` — 虚谷数据库专用协议（必需）
- `serverIP` — 数据库服务器 IP 地址（必需）
- `portNumber` — TCP 端口，默认 5138（必需）
- `databaseName` — 数据库名称（必需）
- `property` — 连接属性，多个属性以 `&` 分隔，不可重复（可选）

## 获取连接的三种方式

### 方式一：URL 内嵌凭据

```java
public Connection getConnection() {
    Connection con = null;
    try {
        Class.forName("com.xugu.cloudjdbc.Driver");
        String url = "jdbc:xugu://localhost:5138/SYSTEM?user=SYSDBA&password=SYSDBA";
        con = DriverManager.getConnection(url);
        System.out.println("数据库连接成功！");
    } catch (Exception e) {
        e.printStackTrace();
    }
    return con;
}
```

### 方式二：分离用户名和密码

```java
public Connection getConnection() {
    Connection con = null;
    String url = "jdbc:xugu://localhost:5138/SYSTEM";
    String user = "SYSDBA";
    String password = "SYSDBA";
    try {
        Class.forName("com.xugu.cloudjdbc.Driver");
        con = DriverManager.getConnection(url, user, password);
        System.out.println("数据库连接成功！");
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if (con != null) {
            try { con.close(); } catch (Exception e) { /* ignore */ }
        }
    }
    return con;
}
```

### 方式三：Properties 对象

```java
public Connection getConnection() {
    Connection con = null;
    Properties info = new Properties();
    info.setProperty("user", "SYSDBA");
    info.setProperty("password", "SYSDBA");
    try {
        Class.forName("com.xugu.cloudjdbc.Driver");
        String url = "jdbc:xugu://localhost:5138/SYSTEM";
        con = DriverManager.getConnection(url, info);
        System.out.println("数据库连接成功！");
    } catch (Exception e) {
        e.printStackTrace();
    }
    return con;
}
```

## DataSource 数据源

```java
import com.xugu.pool.XgDataSource;

public DataSource getDatasource() {
    XgDataSource ds = new XgDataSource();
    ds.setUrl("jdbc:xugu://127.0.0.1:5138/SYSTEM");
    ds.setHostName("127.0.0.1");
    ds.setPort(5138);
    ds.setDatabaseName("SYSTEM");
    ds.setUser("SYSDBA");
    ds.setPassword("SYSDBA");
    ds.setMaxWaitTime(60000);   // 最大等待时间（毫秒）
    ds.setMaxActive(10);        // 最大活动连接数
    ds.setMinIdle(5);           // 最少空闲连接数
    return ds;
}
```

XgDataSource 默认属性值：
- 服务器地址：`localhost`
- 数据库：`SYSTEM`
- 端口：`5138`
- 用户名/密码：`GUEST` / `GUEST`
- 默认 URL：`jdbc:xugu://127.0.0.1:5138/SYSTEM?user=GUEST&password=GUEST&version=110`

## 连接池

XuguDB-JDBC 提供内置连接池实现，核心类映射关系：

| XuguDB-JDBC 类 | 标准 JDBC 接口 |
|----------------|---------------|
| `XgDataSource` | `javax.sql.DataSource` |
| `XgPooledConnection` | `javax.sql.PooledConnection` |
| `XgConnectionPoolDataSource` | `javax.sql.ConnectionPoolDataSource` |
| `XgConnectionEventListener` | `javax.sql.ConnectionEventListener` |
| `XgConnectionEvent` | `javax.sql.ConnectionEvent` |

### 连接池配置与使用

```java
import com.xugu.pool.XgConnectionPoolDataSource;
import com.xugu.pool.XgPooledConnection;

public PooledConnection getXgPooledConnection() throws SQLException {
    XgConnectionPoolDataSource cpds = new XgConnectionPoolDataSource();
    cpds.setUrl("jdbc:xugu://localhost:5138/SYSTEM");
    cpds.setHostName("localhost");
    cpds.setPort(5138);
    cpds.setDatabaseName("SYSTEM");
    cpds.setUser("SYSDBA");
    cpds.setPassword("SYSDBA");

    // 连接池参数
    cpds.setMaxActive(10);          // 最大连接数
    cpds.setMinIdle(5);             // 最小空闲连接数
    cpds.setLoginTimeout(3000);     // 登录超时（毫秒）
    cpds.setMaxWaitTime(3000);      // 最大等待超时（毫秒）

    XgPooledConnection pconn = (XgPooledConnection) cpds.getPooledConnection();
    return pconn;
}
```

连接池配置方法：

| 方法 | 说明 | 单位 |
|------|------|------|
| `setMinIdle(int)` | 最小空闲连接数 | 个 |
| `setMaxActive(int)` | 最大活动连接数 | 个 |
| `setLoginTimeout(int)` | 登录超时 | 毫秒 |
| `setMaxWaitTime(long)` | 最大等待连接超时 | 毫秒 |

> **注意**：`setLoginTimeout()` 和 `setMaxWaitTime()` 参数单位为毫秒。

## 连接参数完整列表

### 基本连接参数

| 连接参数名 | 说明 | 默认值 |
|-----------|------|--------|
| `database` | 数据库名 | — |
| `user` | 用户名 | — |
| `password` | 用户密码 | — |
| `version` | 服务器版本 | `301` |
| `encryptor` | 数据库解密密钥 | — |
| `charset` | 客户端字符集（`utf8` 或 `gbk`） | `GBK` |

### 行为控制参数

| 连接参数名 | 说明 | 默认值 |
|-----------|------|--------|
| `autoCommit` | 是否自动提交 | `true` |
| `isoLevel` | 事务隔离级别 | `READ COMMITTED` |
| `lockTimeout` | 最大加锁等候时间 | — |
| `return_rowid` | 是否返回 rowid | `false` |
| `lob_ret` | 大对象返回方式 | — |
| `timeZone` | 客户端时区 | — |

### 结果集与游标参数

| 连接参数名 | 说明 | 默认值 |
|-----------|------|--------|
| `recv_mode` | 结果集接收方式：0=普通，1=网络缓存，2=服务端游标 | `0` |
| `cursorFetchSize` | `recv_mode=2` 时每次读取记录条数 | `1000` |
| `cursorFirstSize` | `recv_mode=2` 时首次读取记录条数 | `10` |
| `discadeRow` | 是否丢弃已接收记录 | `false` |
| `closeCurrResult` | 结果集数据读完后是否自动关闭 | `false` |

### 批处理与性能参数

| 连接参数名 | 说明 | 默认值 |
|-----------|------|--------|
| `useBatch` | 是否在满足条件时使用批处理 | `false` |
| `benchmarkMode` | 是否开启 benchmark 优化模式 | `false` |
| `benchmarkModeWid` | 是否开启 benchmark 分域优化 | `false` |
| `batchOrMoreResultMode` | 1=多括号批量，0=多结果集 | `1` |
| `ddlPrepare` | 是否支持 DDL 语句的 PreparedStatement | `false` |

### 连接活性与安全参数

| 连接参数名 | 说明 | 默认值 |
|-----------|------|--------|
| `validateConAlive` | 是否开启死链接探活 | `true` |
| `validateFrequency` | 活性探测时间间隔（分钟） | `1` |
| `validatePerTimes` | 一次 task 执行探活次数 | `500` |
| `rebuildConnection` | 连接断开后是否重建 | `false` |
| `sockeTimeOut` | socket 超时时间（秒） | `3600` |
| `useSSL` | 是否启用 SSL 加密 | `false` |

### 兼容性参数

| 连接参数名 | 说明 | 默认值 |
|-----------|------|--------|
| `isora` | 是否兼容 Oracle 的 BIGINT/DATE/DATETIME/NULL 类型 | `false` |
| `compatiblemode` | 适配其他数据库（MySQL/ORACLE/PostgreSQL） | `NONE` |
| `compatiblemysql` | 是否兼容 MySQL 将 BIGINT 映射为 Java Long | `false` |
| `emptyStringAsNull` | 是否把空串当空处理 | `false` |
| `useLike` | 是否在 getTables/getColumns 中使用模糊查询 | `false` |

### 其他参数

| 连接参数名 | 说明 | 默认值 |
|-----------|------|--------|
| `Command.debug` | 是否开启调试 | `false` |
| `slowSQLRecordTime` | 慢 SQL 阈值（>0 开启慢 SQL 日志） | `0` |
| `return_schema` | 查询是否返回模式信息 | `on` |
| `identity_mode` | 自增长模式（DEFAULT/NULL_AS_AUTO_INCREMENT/ZERO_AS_AUTO_INCREMENT） | — |
| `keyword_filter` | 开放的关键字串，逗号分隔 | — |
| `current_schema` | 指定连接的模式名 | — |
| `ips` | 负载均衡 IP 列表 | — |
| `replaceProcessingEnabled` | 是否对预处理参数占位符 `?` 前后添加空格 | `true` |
| `iskernel` | 控制数据库元信息返回 | `false` |
| `disable_binlog` | 不记载 binlog 日志 | — |
| `logVersionDate` | 是否在日志中输出版本打包日期 | — |
| `printVersionTimes` | 控制打印驱动打包时间信息（只打印一次） | `0` |

### URL 连接参数示例

```java
// 设置版本号、大对象描述符、时区、关闭自动提交
String url = "jdbc:xugu://localhost:5138/System"
    + "?user=GUEST&password=GUEST"
    + "&version=110"
    + "&auto_commit=false"
    + "&lob_ret=descriptor"
    + "&time_zone=GMT-8:00";
```
