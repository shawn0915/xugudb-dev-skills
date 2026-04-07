# 虚谷数据库 JDBC 驱动安装与配置

## 环境要求

- JDK 1.8 及以上版本
- 虚谷数据库服务端已启动，默认端口 5138

## 环境变量配置

```
JAVA_HOME = C:\Program Files\Java\jdk1.8.0_451
PATH = %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;
CLASSPATH = .;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar
```

验证配置：

```bash
java -version
# java version "1.8.0_451"
```

## 驱动安装方式

### Maven

```xml
<dependency>
    <groupId>com.xugudb</groupId>
    <artifactId>xugu-jdbc</artifactId>
    <!-- 替换为实际可用版本，如 12.3.4 -->
    <version>*.*.*</version>
</dependency>
```

### Gradle

```groovy
implementation("com.xugudb:xugu-jdbc:*.*.*")
```

### 手动导入 JAR 包

1. 在项目路径下创建 `lib` 目录
2. 将虚谷 JDBC 驱动包（`cloudjdbc-*.*.*.jar`）放入该目录
3. 在 IDE（如 IntelliJ IDEA）中打开项目结构，在 Libraries 中添加该 JAR 包

## 驱动类名

```
com.xugu.cloudjdbc.Driver
```

> 从 JDBC 4.0 起，`DriverManager` 会自动加载驱动，无需手动调用 `Class.forName()`。但为兼容老版本，建议仍显式加载。

## 验证安装

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class Main {
    public static void main(String[] args) {
        String url = "jdbc:xugu://127.0.0.1:5138/SYSTEM";
        String username = "SYSDBA";
        String password = "SYSDBA";
        Connection conn = null;
        try {
            conn = DriverManager.getConnection(url, username, password);
            System.out.println("成功建立连接！");
        } catch (SQLException e) {
            System.out.println("建立连接失败！");
            throw new RuntimeException(e);
        } finally {
            if (conn != null) {
                try { conn.close(); } catch (SQLException e) { /* ignore */ }
            }
        }
    }
}
```

## JDBC 标准兼容性

XuguDB-JDBC 以 JDBC 4.2 协议为标准，截止 xugu-jdbc-12.3.4 版本：

- 已实现标准中 985 个接口中的 858 个
- 实现率超过 87%
- 支持 `java.sql` 和 `javax.sql` 两个包中的主要接口

### 已支持的 java.sql 接口

| 接口 | 说明 |
|------|------|
| `java.sql.Driver` | 驱动程序入口 |
| `java.sql.Connection` | 数据库连接 |
| `java.sql.Statement` | 静态 SQL 执行 |
| `java.sql.PreparedStatement` | 预编译 SQL 执行 |
| `java.sql.CallableStatement` | 存储过程调用 |
| `java.sql.ResultSet` | 结果集 |
| `java.sql.ResultSetMetaData` | 结果集元数据 |
| `java.sql.DatabaseMetaData` | 数据库元数据 |
| `java.sql.Blob` | 二进制大对象 |
| `java.sql.Clob` | 字符大对象 |
| `java.sql.Array` | 数组类型 |
| `java.sql.Savepoint` | 事务保存点 |
| `java.sql.SQLXML` | XML 数据类型 |
| `java.sql.RowId` | 行标识符 |
| `java.sql.ParameterMetaData` | 参数元数据 |
| `java.sql.Ref` | 引用类型 |
| `java.sql.Struct` | 结构化类型 |
| `java.sql.Wrapper` | 包装器 |
| `java.sql.SQLData` | 自定义类型映射 |
| `java.sql.SQLInput` / `java.sql.SQLOutput` | 自定义类型 I/O |
| `java.sql.SQLType` | SQL 类型 |
| `java.sql.DriverAction` | 驱动动作 |

### 已支持的 javax.sql 接口

| 接口 | 说明 |
|------|------|
| `javax.sql.DataSource` | 数据源 |
| `javax.sql.ConnectionPoolDataSource` | 连接池数据源 |
| `javax.sql.PooledConnection` | 池化连接 |
| `javax.sql.CommonDataSource` | 公共数据源 |
| `javax.sql.XADataSource` | XA 数据源 |
| `javax.sql.XAConnection` | XA 连接 |
| `javax.sql.RowSet` | 行集 |
| `javax.sql.RowSetInternal` | 行集内部接口 |
| `javax.sql.RowSetMetaData` | 行集元数据 |
| `javax.sql.RowSetReader` / `javax.sql.RowSetWriter` | 行集读写 |
| `javax.sql.RowSetListener` | 行集监听器 |
| `javax.sql.ConnectionEventListener` | 连接事件监听器 |
| `javax.sql.StatementEventListener` | 语句事件监听器 |
