# 虚谷数据库 JDBC 高级特性

## 大对象（LOB）处理

### 插入大对象

LOB 插入必须通过 PreparedStatement 或 CallableStatement 的 `setXXX()` 方法实现。

可用方法：

| 方法 | 适用类型 |
|------|---------|
| `setBinaryStream(int, InputStream)` | BLOB |
| `setBlob(int, Blob)` | BLOB |
| `setBlob(int, InputStream)` | BLOB |
| `setBytes(int, byte[])` | BLOB |
| `setCharacterStream(int, Reader)` | CLOB |
| `setClob(int, Clob)` | CLOB |
| `setClob(int, Reader)` | CLOB |
| `setString(int, String)` | CLOB |

```java
// 插入 BLOB
PreparedStatement pstm = conn.prepareStatement(
    "INSERT INTO t_blob(id, data) VALUES(?, ?)");
pstm.setInt(1, 1);
FileInputStream fis = new FileInputStream("image.jpg");
pstm.setBinaryStream(2, fis);
pstm.execute();
fis.close();
pstm.close();

// 插入 CLOB
PreparedStatement pstm2 = conn.prepareStatement(
    "INSERT INTO t_clob(id, remark) VALUES(?, ?)");
pstm2.setInt(1, 1);
pstm2.setString(2, "这是一段很长的文本内容...");
pstm2.execute();
pstm2.close();
```

### 读取 CLOB — 流式方式

```java
Statement stm = conn.createStatement();
ResultSet rs = stm.executeQuery("SELECT remark FROM t_clob WHERE id=101");
while (rs.next()) {
    Clob data = rs.getClob("remark");
    // 以字符流方式获取
    Reader rd = data.getCharacterStream();
    char[] buf = new char[1000];
    StringBuilder sb = new StringBuilder();
    int len;
    while ((len = rd.read(buf)) != -1) {
        sb.append(new String(buf, 0, len));
    }
    System.out.println(sb.toString());
    rd.close();
}
rs.close();
stm.close();
```

### 读取 CLOB — 字符串方式

```java
Statement stm = conn.createStatement();
ResultSet rs = stm.executeQuery("SELECT text FROM t_clob");
while (rs.next()) {
    // XuguDB-JDBC 允许直接以字符串形式读取 CLOB
    String clobStr = rs.getString("text");
    System.out.println(clobStr);
}
rs.close();
stm.close();
```

### 读取 BLOB

BLOB 只能通过流式方式获取：

```java
Statement stm = conn.createStatement();
ResultSet rs = stm.executeQuery("SELECT data FROM t_blob WHERE id=1");
while (rs.next()) {
    Blob blob = rs.getBlob("data");
    InputStream is = blob.getBinaryStream();
    // 读取二进制数据...
    byte[] buf = new byte[4096];
    int len;
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
    while ((len = is.read(buf)) != -1) {
        bos.write(buf, 0, len);
    }
    byte[] bytes = bos.toByteArray();
    is.close();
}
rs.close();
stm.close();
```

> **注意**：LOB 对象调用 `free()` 方法后将失效，不可再使用。

## 可更新结果集

要启用结果集更新功能，需同时满足两个条件：
1. 创建 Statement 时设置 `ResultSet.CONCUR_UPDATABLE`
2. URL 连接串中设置 `return_rowid=true`

```java
// URL 需包含 return_rowid=true
String url = "jdbc:xugu://localhost:5138/SYSTEM?return_rowid=true";
Connection conn = DriverManager.getConnection(url, "SYSDBA", "SYSDBA");

// 创建可更新 Statement
Statement stm = conn.createStatement(
    ResultSet.TYPE_SCROLL_INSENSITIVE,
    ResultSet.CONCUR_UPDATABLE);

ResultSet rs = stm.executeQuery("SELECT id, price FROM products");
while (rs.next()) {
    // 更新结果集中的列
    rs.updateLong(1, 5);
    rs.updateFloat(2, 12.3f);
    // 将更改写回数据库
    rs.updateRow();
}
rs.close();
stm.close();
```

## 光标移动与结果集行数统计

```java
Statement stm = conn.createStatement();
ResultSet rs = stm.executeQuery("SELECT * FROM test");

// 移动光标到最后一行
rs.last();
// 获取总行数
int rowCount = rs.getRow();
System.out.println("总行数: " + rowCount);

// 移动到指定行
rs.absolute(5);

// 前后移动
rs.previous();   // 上一行
rs.next();       // 下一行
rs.first();      // 第一行
```

## 多结果集处理

XuguDB-JDBC 支持一次发送多条 SQL 并依次处理多个结果集：

```java
String sql = "SELECT * FROM A1 WHERE D2=(SELECT AVG(D2) FROM A1 GROUP BY D1);"
    + "SELECT * FROM A1 WHERE D2>(SELECT MIN(D2) FROM A1 GROUP BY D1)";

Statement stm = conn.createStatement();
boolean hasResultSet = stm.execute(sql);

if (hasResultSet) {
    ResultSet rs1 = stm.getResultSet();
    // 处理第一个结果集...
    while (rs1.next()) {
        // ...
    }
}

// 遍历后续结果集
while (stm.getMoreResults(Statement.CLOSE_CURRENT_RESULT)) {
    ResultSet rs2 = stm.getResultSet();
    if (rs2 != null) {
        while (rs2.next()) {
            // 处理后续结果集...
        }
    }
}
stm.close();
```

## 服务端游标

适用于大数据量查询场景，需满足两个条件：
1. URL 连接串属性 `recv_mode=2`
2. Statement 执行的 SQL 为单条 SELECT 语句

```java
// 方式一：通过虚谷特有 Statement 接口
com.xugu.cloudjdbc.Statement stm =
    (com.xugu.cloudjdbc.Statement) conn.createStatement();

// 开启服务端游标
stm.setServerCursor(true);

// 设置每次从服务端获取的记录条数
stm.setFetchSize(1000);

ResultSet rs = stm.executeQuery("SELECT * FROM t1");
while (rs.next()) {
    // 逐批从服务端获取数据
    // ...
}

// 关闭服务端游标
stm.setServerCursor(false);

rs.close();
stm.close();
```

> **注意**：服务端游标的 ResultSet 只能单向向前移动，不能后退或来回滚动。

## 自增键获取（GeneratedKeys）

```java
Statement stmt = conn.createStatement();

// 指定自增长关键字列名
String[] colNames = {"id"};

// 在执行方法中传入自增长列标识
stmt.executeUpdate(
    "INSERT INTO authors(first_name, last_name) VALUES('Ghq', 'Wxl')",
    colNames);

// 获取自动生成的键值
ResultSet rs = stmt.getGeneratedKeys();
if (rs.next()) {
    int generatedKey = rs.getInt(1);
    System.out.println("自增键值: " + generatedKey);
}
rs.close();
stmt.close();
```

## 负载均衡

XuguDB-JDBC 支持三种负载均衡配置方式，实现多机环境下连接的均匀分布。

### 方式一：URL 连接串配置

```java
Class.forName("com.xugu.cloudjdbc.Driver");
Connection conn = DriverManager.getConnection(
    "jdbc:xugu://192.168.0.201:5138/SYSTEM"
    + "?user=SYSDBA&password=SYSDBA"
    + "&ips=192.168.0.205,192.168.0.204,192.168.1.206");
```

### 方式二：Properties 配置

```java
Properties info = new Properties();
Class.forName("com.xugu.cloudjdbc.Driver");

// Vector 方式
Vector<String> ipsVector = new Vector<>();
ipsVector.add("192.168.1.201");
ipsVector.add("192.168.1.204");
ipsVector.add("192.168.1.205");
info.put("ips", ipsVector);

// 或数组方式
String[] ips = {"192.168.1.205", "192.168.1.204", "192.168.1.206"};
info.put("ips", ips);

Connection conn = DriverManager.getConnection(
    "jdbc:xugu://192.168.1.201:5138/SYSTEM?user=SYSDBA&password=SYSDBA",
    info);
```

### 方式三：XML 配置文件

创建 `xuguClouldListener.xml`：

```xml
<?xml version="1.0" encoding="utf-8"?>
<listeners>
    <listener><ip>192.168.1.206</ip></listener>
    <listener><ip>192.168.1.204</ip></listener>
    <listener><ip>192.168.1.205</ip></listener>
</listeners>
```

使用 XML 配置：

```java
Class.forName("com.xugu.cloudjdbc.Driver");
Connection conn = DriverManager.getConnection(
    "jdbc:xugu:file://xuguClouldListener.xml:5138/SYSTEM"
    + "?user=SYSDBA&password=SYSDBA");
```

## SSL 传输加密

虚谷数据库通过 SSL 加密机制保护传输数据。

### 启用方式

在 URL 中添加 SSL 参数：

```java
// 启用加密
String url = "jdbc:xugu://localhost:5138/SYSTEM?ssl=ssl";
// 禁用加密
String url = "jdbc:xugu://localhost:5138/SYSTEM?ssl=nssl";
// 或使用 useSSL 参数
String url = "jdbc:xugu://localhost:5138/SYSTEM?useSSL=true";
```

### 动态库依赖

使用 SSL 时需将对应平台的动态链接库复制到系统指定位置：

| 操作系统 | 存放路径 |
|---------|---------|
| Linux (x86_64) | `/usr/lib` 或 `/usr/lib4` |
| Linux (ARM) | `/usr/lib` 或 `/usr/lib4` |
| Windows 32 位 | `C:/windows/system32` |
| Windows 64 位 | `C:/windows/system32` |
| 龙芯 3A5000 | `/usr/lib` 或 `/usr/lib4` |

动态链接库位于 XuguDB-JDBC 压缩包中的 `Driver/ssl` 目录下。

> **注意**：`com.xugu.cloudjdbc.Connection.useSSL` 默认为 `false`（关闭状态）。

## DatabaseMetaData 元数据查询

```java
DatabaseMetaData meta = conn.getMetaData();

// 获取数据库信息
System.out.println("数据库产品名称: " + meta.getDatabaseProductName());
System.out.println("数据库版本: " + meta.getDatabaseProductVersion());
System.out.println("驱动名称: " + meta.getDriverName());
System.out.println("驱动版本: " + meta.getDriverVersion());

// 获取所有表
ResultSet tables = meta.getTables(null, null, "%", new String[]{"TABLE"});
while (tables.next()) {
    System.out.println("表名: " + tables.getString("TABLE_NAME"));
}
tables.close();

// 获取表的列信息
ResultSet columns = meta.getColumns(null, null, "TB_STU", "%");
while (columns.next()) {
    System.out.println("列名: " + columns.getString("COLUMN_NAME")
        + ", 类型: " + columns.getString("TYPE_NAME"));
}
columns.close();
```
