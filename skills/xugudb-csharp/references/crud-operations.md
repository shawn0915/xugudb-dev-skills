# C# (.NET) 驱动 CRUD 操作与接口参考

## 执行 DML/DDL — ExecuteNonQuery

```csharp
XGCommand cmd = conn.CreateCommand();

// DDL
cmd.CommandText = "CREATE TABLE test(id INT, name VARCHAR(100))";
cmd.ExecuteNonQuery();

// DML
cmd.CommandText = "INSERT INTO test VALUES(1, 'hello')";
int affected = cmd.ExecuteNonQuery();
```

## 查询 — ExecuteReader

```csharp
cmd.CommandText = "SELECT * FROM test";
XGDataReader reader = cmd.ExecuteReader();
while (reader.Read()) {
    int id = reader.GetInt32(0);
    string name = reader.GetString(1);
    Console.WriteLine($"{id}, {name}");
}
reader.Close();
```

### 常用 DataReader 方法

| 方法 | 说明 |
|------|------|
| `Read()` | 移动到下一行 |
| `GetInt32(i)` | 获取整数列值 |
| `GetString(i)` | 获取字符串列值 |
| `GetDateTime(i)` | 获取日期列值 |
| `GetValue(i)` | 获取通用对象值 |
| `IsDBNull(i)` | 判断是否为 NULL |
| `GetName(i)` | 获取列名 |
| `FieldCount` | 列数 |
| `Close()` | 关闭 Reader |

## 标量查询 — ExecuteScalar

```csharp
cmd.CommandText = "SELECT COUNT(*) FROM test";
int count = Convert.ToInt32(cmd.ExecuteScalar());
```

## 参数化查询

使用 `?` 占位符：

```csharp
cmd.CommandText = "INSERT INTO test(pid, pname) VALUES(?, ?)";
cmd.Parameters.Add("pid", XGDbType.Int).Value = 1;
cmd.Parameters.Add("pname", XGDbType.VarChar).Value = "test";
cmd.ExecuteNonQuery();

// 清除参数后可复用
cmd.Parameters.Clear();
```

## DataAdapter + DataSet

### 查询

```csharp
XGDataAdapter adapter = new XGDataAdapter("SELECT * FROM test", conn);
DataSet ds = new DataSet();
adapter.Fill(ds, "test");

foreach (DataRow row in ds.Tables["test"].Rows) {
    Console.WriteLine(row["NAME"]);
}
```

### DataSet 更新

```csharp
// 修改 DataSet 数据
ds.Tables["test"].Rows[0]["NAME"] = "updated";

// 删除行
ds.Tables["test"].Rows[1].Delete();

// 新增行
ds.Tables["test"].Rows.Add(new object[] { 10, "new_row" });

// 提交更新到数据库
adapter.Update(ds, "test");
```

## 事务控制

```csharp
XGTransaction trans = conn.BeginTransaction();
XGCommand cmd = conn.CreateCommand();
cmd.Transaction = trans;

try {
    cmd.CommandText = "INSERT INTO test VALUES(1, 'data')";
    cmd.ExecuteNonQuery();
    cmd.CommandText = "INSERT INTO test VALUES(2, 'data2')";
    cmd.ExecuteNonQuery();
    trans.Commit();
} catch (Exception ex) {
    trans.Rollback();
    Console.WriteLine(ex.ToString());
}
```

### 指定隔离级别

```csharp
XGTransaction trans = conn.BeginTransaction(IsolationLevel.ReadUncommitted);
```

## 存储过程调用

### 基本存储过程

```csharp
// 数据库中: CREATE OR REPLACE PROCEDURE P1(pid INT, pname VARCHAR) AS BEGIN
//   INSERT INTO test VALUES(pid, pname); END;

cmd.CommandType = CommandType.StoredProcedure;
cmd.CommandText = "P1";
cmd.Parameters.Add("pid", XGDbType.Int).Value = 2;
cmd.Parameters.Add("pname", XGDbType.VarChar).Value = "test";
cmd.ExecuteNonQuery();
```

### 带 IN/OUT 参数的存储过程

```csharp
cmd.CommandType = CommandType.StoredProcedure;
cmd.CommandText = "P1";

XGParameters par_id = new XGParameters("IN_ID", XGDbType.Int);
par_id.Direction = ParameterDirection.InputOutput;
par_id.Value = 1;
cmd.Parameters.Add(par_id);

XGParameters par_name = new XGParameters("OSS", XGDbType.VarChar, 500);
par_name.Direction = ParameterDirection.Output;
cmd.Parameters.Add(par_name);

cmd.ExecuteNonQuery();

Console.WriteLine("IN_ID=" + par_id.Value);
Console.WriteLine("OSS=" + par_name.Value);
```

### 包中的函数调用

```csharp
cmd.CommandType = CommandType.StoredProcedure;
cmd.CommandText = "PACK_NAME1.PA_FUNC1";  // 包名.函数名

XGParameters aa = new XGParameters("AA", XGDbType.Int);
aa.Direction = ParameterDirection.Input;
aa.Value = 2;
cmd.Parameters.Add(aa);

XGParameters ret = new XGParameters("RET", XGDbType.DateTime);
ret.Direction = ParameterDirection.ReturnValue;
cmd.Parameters.Add(ret);

cmd.ExecuteNonQuery();
Console.WriteLine("返回值=" + ret.Value);
```

### RefCursor 处理

```csharp
// 数据库中: CREATE OR REPLACE FUNCTION myfunc(refcur OUT sys_refcursor) RETURN sys_refcursor

cmd.CommandType = CommandType.StoredProcedure;
cmd.CommandText = "myfunc_outrefur";

XGParameters pa1 = new XGParameters("refcur1", XGDbType.RefCursor);
pa1.Direction = ParameterDirection.Output;
cmd.Parameters.Add(pa1);

XGParameters pa2 = new XGParameters("refcur2", XGDbType.RefCursor);
pa2.Direction = ParameterDirection.ReturnValue;
cmd.Parameters.Add(pa2);

cmd.ExecuteNonQuery();

// 获取结果集
XGDataReader reader1 = ((XGRefCursor)pa1.Value).GetDataReader();
XGDataReader reader2 = ((XGRefCursor)pa2.Value).GetDataReader();

reader1.Close();
reader2.Close();
```

## 大对象（BLOB）处理

### 写入 BLOB

```csharp
cmd.CommandText = "UPDATE test_blob SET ss=? WHERE id=1";

XGBlob xb = new XGBlob();
using (FileStream fs = new FileStream(filepath, FileMode.Open, FileAccess.Read)) {
    byte[] buffer = new byte[64768];
    xb.BeginChunkWrite();
    int bytesRead;
    while ((bytesRead = fs.Read(buffer, 0, buffer.Length)) > 0) {
        xb.write(buffer, 0, bytesRead);
    }
    xb.EndChunkWrite();
}

cmd.Parameters.Add("?", XGDbType.LongVarBinary, xb, ParameterDirection.Input);
cmd.ExecuteNonQuery();
xb.Close();
```

## 服务端游标

```csharp
XGCommand cmd = conn.CreateCommand();
cmd.Use_Server_Cursor = true;  // 启用服务端游标
cmd.CommandText = "SELECT * FROM large_table";
XGDataReader reader = cmd.ExecuteReader();
```

## XGCommand 接口参考

| 属性/方法 | 说明 |
|-----------|------|
| `CommandText` | SQL 语句或存储过程名 |
| `CommandType` | Text / StoredProcedure / TableDirect |
| `Connection` | 关联的 XGConnection |
| `Transaction` | 关联的 XGTransaction |
| `Parameters` | 参数集合 |
| `Use_Server_Cursor` | 是否启用服务端游标 |
| `ExecuteNonQuery()` | 执行 DML，返回影响行数 |
| `ExecuteReader()` | 执行查询，返回 XGDataReader |
| `ExecuteScalar()` | 返回第一行第一列值 |
| `CreateParameter()` | 创建 XGParameters |
| `Clone()` | 复制 XGCommand |
| `Dispose()` | 释放资源 |

## XGTransaction 接口参考

| 方法 | 说明 |
|------|------|
| `Commit()` | 提交事务 |
| `Rollback()` | 回滚事务 |
