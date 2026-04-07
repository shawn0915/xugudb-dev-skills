# 虚谷数据库 JDBC CRUD 操作

## 开发流程概览

1. 通过 DriverManager 获取 Connection
2. 创建 Statement 或其子类
3. 通过 Statement 执行 SQL 命令
4. 处理执行结果（ResultSet）
5. 关闭 Statement
6. 关闭 Connection

```java
String url = "jdbc:xugu://127.0.0.1:5138/SYSTEM";
Connection conn = DriverManager.getConnection(url, "SYSDBA", "SYSDBA");
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("select 1 from dual;");
while (rs.next()) {
    System.out.println(rs.getObject(1));
}
rs.close();
stmt.close();
conn.close();
```

## Statement — 静态 SQL 执行

Statement 用于执行静态 SQL 语句，通过 `Connection.createStatement()` 创建。

### 基本 CRUD 操作

```java
public static void main(String[] args) throws SQLException {
    Connection conn = DriverManager.getConnection(
        "jdbc:xugu://localhost:5138/SYSTEM", "SYSDBA", "SYSDBA");
    Statement stmt = conn.createStatement();

    // 创建表
    stmt.execute("CREATE TABLE tb_stu("
        + "id INT IDENTITY(1,1) NOT NULL PRIMARY KEY, "
        + "name VARCHAR(10));");

    // 插入数据
    stmt.execute("INSERT INTO tb_stu(name) VALUES('张三');");

    // 更新数据
    stmt.executeUpdate("UPDATE tb_stu SET name='李四' WHERE id=1;");

    // 查询数据
    ResultSet rs = stmt.executeQuery("SELECT name FROM tb_stu WHERE id=1;");
    while (rs.next()) {
        System.out.println(rs.getString(1));
    }

    // 关闭资源
    rs.close();
    stmt.close();
    conn.close();
}
```

### Statement 批量操作

> **注意**：
> - 执行批处理前，自动提交模式须设为 `false`
> - 执行成功后手动 `commit()`
> - 执行完毕后恢复自动提交为 `true`
> - 异常时执行 `rollback()`

```java
public static void main(String[] args) throws SQLException, ClassNotFoundException {
    Connection conn = null;
    Statement stmt = null;
    try {
        conn = DriverManager.getConnection(
            "jdbc:xugu://localhost:5138/SYSTEM", "SYSDBA", "SYSDBA");
        stmt = conn.createStatement();

        // 关闭自动提交
        conn.setAutoCommit(false);

        // 添加多条 SQL 到批处理
        stmt.addBatch("INSERT INTO tb_stu(name) VALUES('张三');");
        stmt.addBatch("INSERT INTO tb_stu(name) VALUES('李四');");
        stmt.addBatch("INSERT INTO tb_stu(name) VALUES('王五');");

        // 执行批处理
        stmt.executeBatch();
        System.out.println("插入成功！");

        // 提交事务
        conn.commit();
        System.out.println("提交成功！");

        // 恢复自动提交
        conn.setAutoCommit(true);
    } catch (SQLException e) {
        if (conn != null && !conn.isClosed()) {
            conn.rollback();
            System.out.println("提交失败，回滚！");
            conn.setAutoCommit(true);
        }
    } finally {
        if (stmt != null) { stmt.close(); }
        if (conn != null) { conn.close(); }
    }
}
```

## PreparedStatement — 预编译 SQL 执行

PreparedStatement 通过 `Connection.prepareStatement()` 创建，支持参数化查询。

优势：
- 动态传入参数值
- 预编译缓存执行计划，比 Statement 更快
- 防止 SQL 注入攻击

### 基本参数化操作

```java
public static void main(String[] args) throws SQLException {
    Connection conn = DriverManager.getConnection(
        "jdbc:xugu://localhost:5138/SYSTEM", "SYSDBA", "SYSDBA");

    // 创建 PreparedStatement，? 为参数占位符
    PreparedStatement pstm = conn.prepareStatement(
        "INSERT INTO tb_stu(name) VALUES(?);");

    // 使用 setXxx() 方法给参数传值
    pstm.setString(1, "张三");

    // 执行
    pstm.execute();

    pstm.close();
    conn.close();
}
```

### PreparedStatement 批量插入

> **注意**：PreparedStatement 执行语句不能为 DDL 语句（除非设置 `ddlPrepare=true`）。

```java
public static void main(String[] args) throws SQLException {
    Connection conn = null;
    PreparedStatement pstm = null;
    try {
        conn = DriverManager.getConnection(
            "jdbc:xugu://localhost:5138/SYSTEM", "SYSDBA", "SYSDBA");
        conn.setAutoCommit(false);

        pstm = conn.prepareStatement("INSERT INTO tb_stu(name) VALUES(?);");

        // 批量插入 10000 条，每 100 条提交一次
        for (int i = 0; i < 10000; i++) {
            pstm.setString(1, "user_" + i);
            pstm.addBatch();

            if ((i + 1) % 100 == 0) {
                pstm.executeBatch();
                pstm.clearBatch();
                conn.commit();
            }
        }

        // 提交剩余数据
        pstm.executeBatch();
        pstm.clearBatch();
        conn.commit();

    } catch (Exception e) {
        if (conn != null && !conn.isClosed()) {
            conn.rollback();
            System.out.println("提交失败，回滚！");
            conn.setAutoCommit(true);
        }
    } finally {
        if (pstm != null) { pstm.close(); }
        if (conn != null) { conn.close(); }
    }
}
```

> **限制**：批处理条数或参数个数不得超过 32767。

## CallableStatement — 存储过程调用

CallableStatement 通过 `Connection.prepareCall()` 创建，支持三种参数类型：

| 参数类型 | 设置方式 | 读取方式 |
|---------|---------|---------|
| IN | `setXXX()` | — |
| OUT | `registerOutParameter()` | `getXXX()` |
| INOUT | `setXXX()` | `getXXX()` |

### IN 参数示例

```java
public static void main(String[] args) throws SQLException {
    Connection conn = DriverManager.getConnection(
        "jdbc:xugu://localhost:5138/SYSTEM", "SYSDBA", "SYSDBA");

    // 创建存储过程
    conn.createStatement().execute(
        "CREATE PROCEDURE p_in(p_id IN INT, p_name IN VARCHAR) AS "
        + "BEGIN "
        + "  UPDATE tb_stu SET name=p_name WHERE id=p_id; "
        + "END;");

    // 调用存储过程
    CallableStatement cstm = conn.prepareCall("CALL p_in(?,?);");
    cstm.setInt(1, 1);
    cstm.setString(2, "张三");
    cstm.execute();

    cstm.close();
    conn.close();
}
```

### OUT 参数示例

```java
public static void main(String[] args) throws SQLException {
    Connection conn = DriverManager.getConnection(
        "jdbc:xugu://localhost:5138/SYSTEM", "SYSDBA", "SYSDBA");

    // 创建带 OUT 参数的存储过程
    conn.createStatement().execute(
        "CREATE PROCEDURE p_out(rs OUT VARCHAR) AS "
        + "BEGIN "
        + "  SELECT name INTO rs FROM tb_stu WHERE id=1; "
        + "END;");

    CallableStatement cstm = conn.prepareCall("CALL p_out(?);");

    // 注册 OUT 参数类型
    cstm.registerOutParameter(1, Types.VARCHAR);
    cstm.execute();

    // 获取返回值
    String result = cstm.getString(1);
    System.out.println(result);

    cstm.close();
    conn.close();
}
```

### 使用参数名绑定（替代参数下标）

```java
// 使用 :参数名 语法
CallableStatement cstm = conn.prepareCall("CALL p_in(:a, :b);");
cstm.setInt("a", 1);
cstm.setString("b", "张三");
cstm.execute();
```

> **注意**：不支持参数下标绑定和参数名绑定混合使用。

## 事务管理

### 手动事务控制

```java
public static void main(String[] args) throws SQLException {
    Connection conn = DriverManager.getConnection(
        "jdbc:xugu://localhost:5138/SYSTEM", "SYSDBA", "SYSDBA");

    // 开启手动事务
    conn.setAutoCommit(false);

    Statement stm = conn.createStatement();

    // 执行静态 SQL
    stm.execute("INSERT INTO tb_stu(name) VALUES('张三');");

    // 执行预处理 SQL
    PreparedStatement pstm = conn.prepareStatement(
        "UPDATE tb_stu SET name='李四' WHERE id=?");
    pstm.setInt(1, 1);
    pstm.execute();

    // 提交事务 — 上面的 INSERT 和 UPDATE 永久生效
    conn.commit();

    // 之后的操作
    stm.execute("DELETE FROM tb_stu WHERE id=1;");

    // 回滚 — DELETE 操作不会生效
    conn.rollback();

    stm.close();
    pstm.close();
    conn.close();
}
```

### Savepoint 事务保存点

```java
public static void main(String[] args) throws SQLException {
    Connection conn = DriverManager.getConnection(
        "jdbc:xugu://localhost:5138/SYSTEM", "SYSDBA", "SYSDBA");
    conn.setAutoCommit(false);

    Statement stm = conn.createStatement();

    stm.execute("INSERT INTO tb_stu(name) VALUES('张三');");
    stm.execute("INSERT INTO tb_stu(name) VALUES('李四');");

    // 设置保存点 save1
    Savepoint save1 = conn.setSavepoint("save_1");

    stm.execute("INSERT INTO tb_stu(name) VALUES('王五');");

    // 设置保存点 save2
    Savepoint save2 = conn.setSavepoint("save_2");

    stm.execute("INSERT INTO tb_stu(name) VALUES('赵六');");

    // 设置保存点 save3
    Savepoint save3 = conn.setSavepoint("save_3");

    stm.execute("INSERT INTO tb_stu(name) VALUES('陈七');");

    // 回滚到 save3 — "陈七" 这条记录被回滚
    conn.rollback(save3);

    // 提交 — "张三"、"李四"、"王五"、"赵六" 保留
    conn.commit();

    stm.close();
    conn.close();
}
```

> **注意**：事务仅对 DML 操作有效。

## 执行 SQL 语句对象的 ResultSet 属性设置

创建 Statement 时可通过构造方法设置 ResultSet 属性：

| 属性类型 | 描述 | 默认值 |
|---------|------|--------|
| ResultSetType | `TYPE_SCROLL_INSENSITIVE`：支持前后滚动，对其他会话的变更不敏感 | `TYPE_SCROLL_INSENSITIVE` |
| ResultSetConcurrency | `CONCUR_READ_ONLY`：不可更新结果集 | `CONCUR_READ_ONLY` |
| ResultSetHoldability | `HOLD_CURSORS_OVER_COMMIT`：commit/rollback 后 ResultSet 仍可用 | `HOLD_CURSORS_OVER_COMMIT` |

```java
// Statement 扩展构造
Statement stm = conn.createStatement(
    ResultSet.TYPE_SCROLL_INSENSITIVE,
    ResultSet.CONCUR_UPDATABLE);

// PreparedStatement 扩展构造
PreparedStatement pstm = conn.prepareStatement(sql,
    ResultSet.TYPE_SCROLL_INSENSITIVE,
    ResultSet.CONCUR_UPDATABLE,
    ResultSet.HOLD_CURSORS_OVER_COMMIT);

// CallableStatement 扩展构造
CallableStatement cstm = conn.prepareCall(sql,
    ResultSet.TYPE_SCROLL_INSENSITIVE,
    ResultSet.CONCUR_UPDATABLE);
```

## 数据类型映射

### 基本数据类型

| 虚谷数据库类型 | 标准 SQL 类型 | Java 类型 |
|--------------|-------------|----------|
| BOOLEAN | BOOLEAN | `Boolean` |
| TINYINT | TINYINT | `Byte` |
| SMALLINT | SMALLINT | `Short` |
| INTEGER | INTEGER | `Integer` |
| BIGINT | BIGINT | `Long` |
| FLOAT | FLOAT | `Float` |
| DOUBLE | DOUBLE | `Double` |
| NUMERIC | NUMERIC/DECIMAL | `BigDecimal` |
| CHAR / NCHAR | CHAR / NCHAR | `String` |
| VARCHAR / VARCHAR2 / NVARCHAR2 | VARCHAR | `String` |
| BLOB | BLOB | `Blob` |
| CLOB / NCLOB | CLOB | `Clob` |
| BINARY | BINARY / VARBINARY / LONGVARBINARY | `byte[]` |
| DATE | DATE | `Date` |
| TIME | TIME | `Time` |
| TIMESTAMP | TIMESTAMP | `Timestamp` |
| JSON | JSON | `String` |
| XML | XML | `String` |
| BIT / VARBIT | BIT / VARBIT | `Byte` |
| ARRAY | ARRAY | `Array` |

### 扩展数据类型

| 虚谷数据库类型 | Java 类型 |
|--------------|----------|
| DATETIME | `Timestamp` |
| DATETIME WITH TIME ZONE | `String` |
| TIME WITH TIME ZONE | `String` |
| INTERVAL YEAR/MONTH/DAY/HOUR/MINUTE/SECOND | `String` |
| INTERVAL YEAR TO MONTH 等复合类型 | `String` |
| GUID | `Timestamp` |
| CURSOR / SYS_REFCURSOR | 使用 `ResultSet` 获取游标数据 |

### 类型转换规则

- **getXxx() 方法**：Y=直接转换（类型匹配）、X=无丢失转换（长精度取短精度）、Z=依赖转换（可能丢失精度）
- **setXxx() 方法**：P=允许插入（无损）、Z=部分插入（类型不完全匹配）

> 建议按照数据库列类型使用对应的 getXxx()/setXxx() 方法，避免隐式转换导致数据精度丢失。
