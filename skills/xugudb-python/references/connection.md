# 连接管理

## xgcondb 包级方法

### xgcondb.connect()

创建 connection 对象。

```python
def connect(*args, **kwargs)
```

**参数：**

| 参数 | 说明 |
|------|------|
| `host` | 数据库 IP 地址。多 IP 高可用使用逗号分隔，如 `host="ip1,ip2"` |
| `port` | 数据库端口（字符串），默认 `"5138"` |
| `database` | 库名 |
| `user` | 登录用户名 |
| `password` | 登录用户密码 |
| `charset` | 客户端编码，默认为 `utf8` |
| `usessl` | 加密连接设置（字符串），传入 `"true"`、`"on"`、`"yes"` 表示创建加密连接，其余为非加密 |

还可以直接使用数据库会话连接参数名传递其他参数（如 `current_schema`）。

**返回值：** connection 对象

**示例：**

```python
import xgcondb

# 基础连接
conn = xgcondb.connect(
    host="192.168.2.216",
    port="5138",
    database="SYSTEM",
    user="SYSDBA",
    password="SYSDBA",
    charset='utf8'
)

# SSL 加密连接
conn = xgcondb.connect(
    host="192.168.2.216",
    port="5138",
    database="SYSTEM",
    user="SYSDBA",
    password="SYSDBA",
    charset='utf8',
    usessl="true"
)

# 指定 current_schema
conn = xgcondb.connect(
    host="192.168.2.216",
    port="5138",
    database="SYSTEM",
    user="SYSDBA",
    password="SYSDBA",
    charset='utf8',
    current_schema="TEST"
)

# 多 IP 高可用连接
conn = xgcondb.connect(
    host="192.168.2.100,192.168.2.101",
    port="5138",
    database="SYSTEM",
    user="SYSDBA",
    password="SYSDBA"
)
```

### xgcondb.version()

返回驱动版本信息。

```python
print(xgcondb.version())
# 'xgcondb V2.3.7'
```

## connection 类

connection 类封装了数据库会话，通过 `xgcondb.connect()` 创建。连接是线程安全的，可以在多个线程之间共享连接对象。

### connection.cursor()

创建游标对象，用于数据库操作执行。

```python
cur = conn.cursor()
```

### connection.begin()

在连接上开启一个新事务。

```python
conn.begin()
```

### connection.commit()

提交连接上的事务。

```python
conn = xgcondb.connect(host="192.168.2.216", port="5138",
    database="SYSTEM", user="SYSDBA", password="SYSDBA", charset='utf8')
conn.execute("create table t_commit(id int, name varchar);")
conn.begin()
conn.execute("insert into t_commit values(1, 'test');")
conn.commit()
```

### connection.rollback()

回滚连接上的事务。

```python
conn.execute("create table t_rollback(id int, name varchar);")
conn.begin()
conn.execute("insert into t_rollback values(1, 'test');")
conn.rollback()
```

### connection.autocommit(bool)

设置连接的自动提交属性。

```python
# 设置为自动提交
conn.autocommit(True)

# 设置为非自动提交
conn.autocommit(False)
```

### connection.ping([reconnect])

连接判活，并根据参数自动重连。

**参数：**
- `reconnect`：连接失活时自动重连标志位（可选，默认 False）

**返回值：** 存活或重连成功返回 True，失活返回 False

```python
if conn.ping(True):
    print("conn is alive")
```

### connection.select_db(db_name)

切换连接对象所连接的数据库。

```python
conn.select_db("TESTDB")
```

### connection.close()

关闭连接对象。

```python
conn.close()
```

## 事务管理模式

### 手动事务控制

```python
import xgcondb

conn = xgcondb.connect(
    host="127.0.0.1", port="5138",
    database="SYSTEM", user="SYSDBA", password="SYSDBA"
)
cur = conn.cursor()

try:
    conn.begin()
    cur.execute("INSERT INTO my_table VALUES (1, 'data1')")
    cur.execute("INSERT INTO my_table VALUES (2, 'data2')")
    conn.commit()
except Exception as e:
    conn.rollback()
    print(f"事务回滚: {e}")
finally:
    cur.close()
    conn.close()
```

### 自动提交模式

```python
conn = xgcondb.connect(
    host="127.0.0.1", port="5138",
    database="SYSTEM", user="SYSDBA", password="SYSDBA"
)
conn.autocommit(True)
cur = conn.cursor()

# 每条语句自动提交
cur.execute("INSERT INTO my_table VALUES (1, 'auto')")

cur.close()
conn.close()
```
