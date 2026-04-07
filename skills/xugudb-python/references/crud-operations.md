# CRUD 操作

## cursor 类概述

游标对象由 `connection.cursor()` 创建，在整个生命周期内绑定到连接。所有命令在连接所包裹的数据库会话上下文中执行。

**重要：** 游标不是线程安全的。多线程应用程序可以从同一连接创建多个游标，但应在单个线程中使用每个游标。

## 执行方法

### cursor.execute(sql [, parameters])

执行数据库语句。

**参数：**
- `sql`：待执行的 SQL 语句
- `parameters`（可选）：绑定参数，可以是基础类型、一维元组、列表、字典

**占位符规则：**
- 使用 `?` 作为占位符（元组、列表参数）
- 使用 `:参数名` 作为占位符（字典参数）

```python
import xgcondb

conn = xgcondb.connect(host="192.168.2.216", port="5138",
    database="SYSTEM", user="SYSDBA", password="SYSDBA", charset='utf8')
cur = conn.cursor()

# 执行无参数 SQL
cur.execute("create table t_execute(id int, name varchar);")

# 参数为基础类型
cur.execute("insert into t_execute values(1, ?)", "test")

# 参数为一维元组
cur.execute("insert into t_execute values(2, ?)", ("test2",))
cur.execute("insert into t_execute values(?, ?)", (3, "test3"))

# 参数为一维列表
cur.execute("insert into t_execute values(4, ?)", ["test4"])
cur.execute("insert into t_execute values(?, ?)", [5, "test5"])

# 参数为一维字典（使用 :name 占位符）
cur.execute("insert into t_execute values(:id, :name)", {"id": 6, "name": "test6"})
```

### cursor.executemany(sql, parameters)

以 prepare 方式批量执行数据库语句。

**参数：**
- `sql`：待执行的 SQL 语句
- `parameters`：可以是一维/二维元组、列表、字典

**占位符规则同 execute。**

```python
# 参数为一维元组
cur.executemany("insert into t values(1, ?)", ("test1",))
cur.executemany("insert into t values(?, ?)", (2, "test2"))

# 参数为二维元组（批量插入多行）
cur.executemany("insert into t values(?, ?)", ((3, "test3"), (4, "test4")))

# 参数为一维列表
cur.executemany("insert into t values(5, ?)", ["test5"])
cur.executemany("insert into t values(?, ?)", [6, "test6"])

# 参数为二维列表（批量插入多行）
cur.executemany("insert into t values(?, ?)", [[7, "test7"], [8, "test8"]])

# 参数为一维字典
cur.executemany("insert into t values(:id, :name)", {"id": 9, "name": "test9"})

# 参数为二维字典（批量插入多行）
cur.executemany("insert into t values(:id, :name)", {
    "param1": {"id": 10, "name": "test10"},
    "param2": {"id": 11, "name": "test11"}
})
```

### cursor.executebatch(sql, parameters)

以批量发送参数方式执行数据库语句。参数组织方式与 executemany 不同——每列的值集中到一个列表中。

**参数：**
- `sql`：待执行的 SQL 语句
- `parameters`：二维元组/列表，每个元素为单列参数的集合

**占位符：** 使用 `?`

```python
cur.execute("create table t_executebatch(id int, name varchar);")

param_1 = [1, 2, 3, 4, 5]
param_2 = ["name1", "name2", "name3", "name4", "name5"]

# 按列传参，一次性插入5行
cur.executebatch("insert into t_executebatch values(?, ?)", [param_1, param_2])
```

## 查询方法

### cursor.fetchone()

提取查询结果集一行数据。

**返回值：** 一行数据的元组，无数据时返回 None。

```python
cur.execute("select 1, 'test1' from dual")
print(cur.fetchone())
# (1, 'test1')
```

### cursor.fetchmany([size])

提取查询结果集多行数据。

**参数：**
- `size`（可选）：获取的行数，默认由 `cursor.arraysize` 决定（arraysize 默认为 1）

**返回值：** 多行数据的二维集合，每行为元组。

```python
cur.execute("create table t_fetch(id int, name varchar);")
cur.execute("insert into t_fetch values(1,'t1'),(2,'t2'),(3,'t3'),(4,'t4'),(5,'t5'),(6,'t6')")
cur.execute("select * from t_fetch")

# 通过 size 参数控制行数
print(cur.fetchmany(3))
# [(1, 't1'), (2, 't2'), (3, 't3')]

# 通过修改 cursor.arraysize 控制行数
cur.arraysize = 3
print(cur.fetchmany())
# [(4, 't4'), (5, 't5'), (6, 't6')]
```

### cursor.fetchall()

一次性提取结果集的所有数据。

**返回值：** 所有数据的二维集合，每行为元组。

```python
cur.execute("select * from t_fetch")
print(cur.fetchall())
# [(1, 't1'), (2, 't2'), (3, 't3'), (4, 't4'), (5, 't5'), (6, 't6')]
```

## 结果集元数据方法

### cursor.getResultcolname(colno)

获取对应列号的列名（列号从 1 开始）。

```python
cur.execute("select 1 as col1 from dual")
print(cur.getResultcolname(1))  # COL1
```

### cursor.getResultcolseq(colname)

获取对应列名的列序号（从 1 开始）。

```python
cur.execute("select 1 as col1 from dual")
print(cur.getResultcolseq("COL1"))  # 1
```

### cursor.getResultcolsize(colno)

获取对应列号的精度。

```python
cur.execute("select 1 as col1 from dual")
print(cur.getResultcolsize(1))  # 1
```

### cursor.getResultcolscale(colno)

获取对应列号的标度。

```python
cur.execute("select 1 as col1 from dual")
print(cur.getResultcolscale(1))  # 0
```

## cursor 属性

### cursor.description

获取结果集的列属性。执行查询后存在结果集则返回列属性列表，否则返回 None。

每列属性为元组：`(列名, 列类型, 占位符, 占位符, 精度, 标度, 占位符)`

```python
cur.execute("select 1, 'test1' from dual")
print(cur.description)
# [('EXPR1', 'TINYINT', None, None, 1, 0, None),
#  ('EXPR2', 'VARCHAR', None, None, 7, 0, None)]
```

### cursor.rowcount

结果集行数，执行查询后可通过此属性获取。

```python
cur.execute("select 1 from dual")
print(cur.rowcount)  # 1
```

### cursor.arraysize

指定 fetchmany 默认提取行数，默认为 1。

```python
cur.arraysize = 6
print(cur.fetchmany())  # 获取6行
```

## 参数类型设置

### cursor.setinputsizes(sizes)

事先设置传入执行方法的参数长度。

```python
cur.setinputsizes((10, 10))
cur.execute("select 1 from dual where ? = ?;", (10, 10))
```

### cursor.setinputtype(types)

事先设置传入参数对应的数据库类型（不由驱动自动检测）。

**虚谷数据库类型与参数类型常量映射：**

| 数据库类型 | 参数类型常量 |
|-----------|------------|
| NULL | `XG_C_NULL` |
| BOOLEAN | `XG_C_BOOL` |
| VARCHAR | `XG_C_CHAR` |
| CHAR | `XG_C_NCHAR` |
| TINYINT | `XG_C_TINYINT` |
| SMALLINT | `XG_C_SHORT` |
| INTEGER | `XG_C_INTEGER` |
| BIGINT | `XG_C_BIGINT` |
| DOUBLE | `XG_C_DOUBLE` |
| NUMERIC | `XG_C_NUMERIC` |
| DATE | `XG_C_DATE` |
| TIME | `XG_C_TIME` |
| DATETIME | `XG_C_DATETIME` |
| BINARY | `XG_C_BINARY` |
| INTERVAL_YEAR_TO_MONTH | `XG_C_INTERVAL_YEAR_TO_MONTH` |
| INTERVAL_DAY_TO_SECOND | `XG_C_INTERVAL_DAY_TO_SECOND` |
| CLOB | `XG_C_CLOB` |
| BLOB | `XG_C_BLOB` |
| SYS_REFCURSOR | `XG_C_REFCUR` |

```python
cur.setinputtype((xgcondb.XG_C_INTEGER, xgcondb.XG_C_INTEGER))
cur.execute("select 1 from dual where ? = ?;", (10, 10))
```

## 存储过程与存储函数

### cursor.callproc(procname [, parameters, parametertypes])

执行存储过程。支持带参数（in/out/inout）、无参数、返回结果集的存储过程。

**参数：**
- `procname`：存储过程名称
- `parameters`（可选）：参数元组
- `parametertypes`（可选）：参数类型元组（1=IN, 2=OUT, 3=INOUT）

**返回值：** 参数列表对象。

```python
# 执行无参数存储过程
print(cur.callproc("test_no_parameter"))

# 执行 IN 参数存储过程
print(cur.callproc("test_in", (20,), (1,)))

# 执行 OUT 参数存储过程
print(cur.callproc("test_out", ("xugu",), (2,)))

# 执行 INOUT 参数存储过程
print(cur.callproc("test_inout", (20, 'xugu'), (3, 2)))
```

### 存储过程返回结果集示例

```python
import xgcondb

conn = xgcondb.connect(host="127.0.0.1", port="5138",
    database="PYTHON3", user="SYSDBA", password="SYSDBA")
cur = conn.cursor()

cur.execute('create table test(arg1 int, arg2 varchar);')

# 创建返回 SYS_REFCURSOR 的存储过程
cur.execute('''CREATE or replace procedure pro_test(col1 int, col2 OUT SYS_REFCURSOR) as
declare
    par1 int;
    str2 varchar;
begin
    par1 := col1;
    for i in 1..par1 loop
        str2 := 'insert into test values(' || i || ',''' || par2 || ''');';
        execute immediate str2;
    end loop;
    OPEN col2 FOR SELECT * FROM test;
end;''')

# 设置参数类型
cur.setinputtype((xgcondb.XG_C_INTEGER, xgcondb.XG_C_REFCUR))
cur.setinputsizes((4, 10))

# 调用存储过程并获取结果集
print(cur.callproc('pro_test', (100, 'refcur'), (1, 2)))
row = cur.fetchall()
for i in row:
    print(i)

# 清理类型和大小设置
cur.clearsize()
cur.cleartype()
```

### cursor.nextset()

多结果集情况下，跳到下一结果集。

**返回值：** 存在下一结果集返回 True，不存在返回 None。

```python
cur.execute("select 1 from dual; select 2 from dual;")
cur.nextset()
print(cur.fetchone())  # (2,)
```

### cursor.close()

关闭游标对象。

```python
cur.close()
```

## 大对象（LOB）操作

### BLOB/CLOB 插入

```python
import xgcondb

conn = xgcondb.connect(host="127.0.0.1", port="5138",
    database="SYSTEM", user="SYSDBA", password="SYSDBA")
cur = conn.cursor()

cur.execute('create table t_lob(B BLOB, C CLOB);')

# 读取文件内容
clob_fp = open("./clob_test.txt", "r")
blob_fp = open("./blob_test.jpg", "rb")
clob_buf = clob_fp.read()
blob_buf = blob_fp.read()

# 插入大对象
cur.execute('insert into t_lob values(?, ?);', (blob_buf, clob_buf))
```

### BLOB/CLOB 导出

```python
cur.execute('select * from t_lob;')
row = cur.fetchone()

# 导出到文件
blob_fd = open("./blob_select.jpg", "wb+")
clob_fd = open("./clob_select.txt", "w+")
blob_fd.write(row[0])
clob_fd.write(row[1])

cur.close()
conn.close()
```

## 完整 CRUD 示例

```python
#!/usr/bin/python3
import xgcondb

conn = xgcondb.connect(
    host="127.0.0.1", port="5138",
    database="SYSTEM", user="SYSDBA", password="SYSDBA"
)
cur = conn.cursor()

try:
    # CREATE - 建表
    cur.execute("""CREATE TABLE TAB_DEMO(
        A INT, B INT, C VARCHAR, D DATETIME, E NUMBER(4,2)
    )""")

    # INSERT - 插入数据
    cur.execute("INSERT INTO TAB_DEMO VALUES(1001, 2001, 'XUGU1', '2019-01-01', 23.54);")
    cur.execute("INSERT INTO TAB_DEMO VALUES(1002, 2002, 'XUGU2', '2019-01-02', 2.354);")
    cur.execute("INSERT INTO TAB_DEMO VALUES(1003, 2003, 'XUGU3', '2019-01-03', 35.4);")
    cur.execute("INSERT INTO TAB_DEMO VALUES(1004, 2004, 'XUGU4', '2019-01-04', 54.00);")

    # SELECT - 逐行查询
    cur.execute("SELECT * FROM TAB_DEMO;")
    row = cur.fetchone()
    while row is not None:
        print(row)
        row = cur.fetchone()

    # SELECT - 批量查询
    cur.execute("SELECT * FROM TAB_DEMO;")
    rows = cur.fetchall()
    for row in rows:
        print(row)

finally:
    cur.close()
    conn.close()
```
