# ODBC CRUD 操作

## C/C++ ODBC API

### 完整示例

```c
#include <stdio.h>
#include <sql.h>
#include <sqlext.h>

int main() {
    SQLHENV henv;
    SQLHDBC hdbc;
    SQLHSTMT hstmt;
    SQLRETURN ret;

    // 分配环境句柄
    SQLAllocHandle(SQL_HANDLE_ENV, SQL_NULL_HANDLE, &henv);
    SQLSetEnvAttr(henv, SQL_ATTR_ODBC_VERSION, (void*)SQL_OV_ODBC3, 0);

    // 分配连接句柄
    SQLAllocHandle(SQL_HANDLE_DBC, henv, &hdbc);

    // 连接
    ret = SQLDriverConnect(hdbc, NULL,
        (SQLCHAR*)"DRIVER={XuguDB ODBC Driver};SERVER=127.0.0.1;PORT=5138;DATABASE=SYSTEM;UID=SYSDBA;PWD=SYSDBA",
        SQL_NTS, NULL, 0, NULL, SQL_DRIVER_NOPROMPT);

    if (ret != SQL_SUCCESS && ret != SQL_SUCCESS_WITH_INFO) {
        printf("连接失败\n");
        return 1;
    }

    // 分配语句句柄
    SQLAllocHandle(SQL_HANDLE_STMT, hdbc, &hstmt);

    // 执行查询
    SQLExecDirect(hstmt, (SQLCHAR*)"SELECT id, name FROM test", SQL_NTS);

    // 绑定结果列
    SQLINTEGER id;
    SQLCHAR name[100];
    SQLLEN id_ind, name_ind;

    SQLBindCol(hstmt, 1, SQL_C_LONG, &id, 0, &id_ind);
    SQLBindCol(hstmt, 2, SQL_C_CHAR, name, sizeof(name), &name_ind);

    // 遍历结果
    while (SQLFetch(hstmt) == SQL_SUCCESS) {
        printf("%d, %s\n", id, name);
    }

    // 释放资源
    SQLFreeHandle(SQL_HANDLE_STMT, hstmt);
    SQLDisconnect(hdbc);
    SQLFreeHandle(SQL_HANDLE_DBC, hdbc);
    SQLFreeHandle(SQL_HANDLE_ENV, henv);

    return 0;
}
```

### 参数化执行

```c
SQLAllocHandle(SQL_HANDLE_STMT, hdbc, &hstmt);

SQLPrepare(hstmt, (SQLCHAR*)"INSERT INTO test (id, name) VALUES (?, ?)", SQL_NTS);

SQLINTEGER id = 1;
SQLCHAR name[] = "hello";
SQLLEN id_ind = 0, name_ind = SQL_NTS;

SQLBindParameter(hstmt, 1, SQL_PARAM_INPUT, SQL_C_LONG, SQL_INTEGER, 0, 0, &id, 0, &id_ind);
SQLBindParameter(hstmt, 2, SQL_PARAM_INPUT, SQL_C_CHAR, SQL_VARCHAR, 100, 0, name, sizeof(name), &name_ind);

SQLExecute(hstmt);
```

### 事务控制

```c
// 关闭自动提交
SQLSetConnectAttr(hdbc, SQL_ATTR_AUTOCOMMIT, (SQLPOINTER)SQL_AUTOCOMMIT_OFF, 0);

// 执行操作...
SQLExecDirect(hstmt, (SQLCHAR*)"INSERT INTO test VALUES(1, 'data')", SQL_NTS);

// 提交
SQLEndTran(SQL_HANDLE_DBC, hdbc, SQL_COMMIT);

// 或回滚
// SQLEndTran(SQL_HANDLE_DBC, hdbc, SQL_ROLLBACK);
```

## Python (pyodbc)

```python
import pyodbc

# 连接
conn = pyodbc.connect(
    'DRIVER={XuguDB ODBC Driver};SERVER=127.0.0.1;PORT=5138;'
    'DATABASE=SYSTEM;UID=SYSDBA;PWD=SYSDBA'
)
cursor = conn.cursor()

# DDL
cursor.execute("CREATE TABLE test (id INT, name VARCHAR(100))")
conn.commit()

# 插入
cursor.execute("INSERT INTO test VALUES (?, ?)", 1, "hello")
conn.commit()

# 查询
cursor.execute("SELECT * FROM test")
for row in cursor.fetchall():
    print(row.id, row.name)

# 参数化查询
cursor.execute("SELECT * FROM test WHERE id = ?", 1)

# 事务
conn.autocommit = False
try:
    cursor.execute("INSERT INTO test VALUES (?, ?)", 2, "world")
    cursor.execute("INSERT INTO test VALUES (?, ?)", 3, "test")
    conn.commit()
except:
    conn.rollback()

conn.close()
```

## ODBC API 参考

| 函数 | 说明 |
|------|------|
| SQLAllocHandle | 分配句柄 |
| SQLFreeHandle | 释放句柄 |
| SQLDriverConnect | 连接数据库 |
| SQLDisconnect | 断开连接 |
| SQLExecDirect | 直接执行 SQL |
| SQLPrepare | 预编译 SQL |
| SQLExecute | 执行预编译 SQL |
| SQLBindParameter | 绑定输入参数 |
| SQLBindCol | 绑定输出列 |
| SQLFetch | 获取结果行 |
| SQLGetData | 获取列数据 |
| SQLEndTran | 提交/回滚事务 |
| SQLSetEnvAttr | 设置环境属性 |
| SQLSetConnectAttr | 设置连接属性 |
| SQLGetDiagRec | 获取诊断信息 |
