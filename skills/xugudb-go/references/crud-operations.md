# Go 驱动 CRUD 操作与接口参考

## DDL 操作

```go
// 创建表
_, err := db.Exec("CREATE TABLE go_test(c1 INT, c2 VARCHAR, c3 FLOAT)")
if err != nil {
    log.Fatal(err)
}
```

## DML 操作（Insert/Update/Delete）

### 直接执行

```go
res, err := db.Exec("INSERT INTO go_test VALUES(1, 'hello', 0.99)")
if err != nil {
    log.Fatal(err)
}
affected, _ := res.RowsAffected()
fmt.Printf("影响行数: %d\n", affected)
```

### 参数化执行（推荐，防 SQL 注入）

```go
res, err := db.Exec("INSERT INTO go_test VALUES(?, ?, ?)", 1, "hello", 0.99)
```

### 预编译批量执行

```go
stmt, err := db.Prepare("INSERT INTO go_test VALUES(?, ?, ?)")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close()

for i := 1; i <= 10; i++ {
    _, err = stmt.Exec(i, fmt.Sprintf("item_%d", i), float64(i)*0.1)
    if err != nil {
        log.Fatal(err)
    }
}
```

### 带 Context 的执行

```go
_, err := db.ExecContext(context.Background(),
    "INSERT INTO go_test VALUES(?, ?)", 1, "hello")
```

## 查询操作

### 多行查询（Query）

```go
rows, err := db.Query("SELECT c1, c2 FROM go_test")
if err != nil {
    log.Fatal(err)
}
defer rows.Close()

// 获取列名
cols, _ := rows.Columns()

// 遍历结果
for rows.Next() {
    var c1 int
    var c2 string
    err = rows.Scan(&c1, &c2)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("%d\t%s\n", c1, c2)
}
```

### 通用扫描方式（列数不确定时）

```go
rows, err := db.Query("SELECT * FROM go_test")
if err != nil {
    log.Fatal(err)
}
defer rows.Close()

cols, _ := rows.Columns()
pvals := make([]interface{}, len(cols))
for i := range pvals {
    dest := make([]byte, 216)
    pvals[i] = &dest
}

for rows.Next() {
    err = rows.Scan(pvals...)
    if err != nil {
        log.Fatal(err)
    }
    for _, v := range pvals {
        fmt.Printf("%s\t", string(*(v.(*[]byte))))
    }
    fmt.Println()
}
```

### 单行查询（QueryRow）

```go
row := db.QueryRow("SELECT c1, c2 FROM go_test LIMIT 1")
var c1 int
var c2 string
err := row.Scan(&c1, &c2)
if err == sql.ErrNoRows {
    fmt.Println("无数据")
} else if err != nil {
    log.Fatal(err)
}
```

### 多结果集

```go
rows, err := db.Query("SELECT * FROM table1; SELECT * FROM table2")
if err != nil {
    log.Fatal(err)
}
defer rows.Close()

// 处理第一个结果集
for rows.Next() {
    // ...
}

// 切换到下一个结果集
if rows.NextResultSet() {
    for rows.Next() {
        // ...
    }
}
```

## 事务操作

```go
tx, err := db.Begin()
if err != nil {
    log.Fatal(err)
}

// 在事务中执行操作
_, err = tx.Exec("INSERT INTO go_test VALUES(1, 'data', 0.5)")
if err != nil {
    tx.Rollback()
    log.Fatal(err)
}

// 事务中也支持 Prepare、Query、QueryRow
stmt, _ := tx.Prepare("INSERT INTO go_test VALUES(?, ?, ?)")
_, err = tx.Stmt(stmt).Exec(2, "data2", 0.6)

// 提交事务
err = tx.Commit()
if err != nil {
    log.Fatal(err)
}
```

### 事务中的完整操作

事务对象 `*sql.Tx` 支持以下方法：
- `tx.Exec(query, args...)` — 执行 DML
- `tx.Query(query, args...)` — 查询多行
- `tx.QueryRow(query, args...)` — 查询单行
- `tx.Prepare(query)` — 预编译
- `tx.Stmt(stmt)` — 在事务中使用已有的 Stmt
- `tx.Commit()` — 提交
- `tx.Rollback()` — 回滚

## 大对象（LOB）处理

### 插入 BLOB

```go
// 读取文件到 []byte
data, err := ioutil.ReadAll(file)
if err != nil {
    log.Fatal(err)
}

// 插入 BLOB 数据
_, err = db.Exec("INSERT INTO lob_test VALUES(1, ?, NULL)", data)
```

### 读取 BLOB

```go
// 先查询 BLOB 长度
rows, _ := db.Query("SELECT LEN(c2) FROM lob_test WHERE c1=1")
var length int
if rows.Next() {
    rows.Scan(&length)
}
rows.Close()

// 再读取 BLOB 数据
rows, _ = db.Query("SELECT c2 FROM lob_test WHERE c1=1")
buf := make([]byte, length)
var lob interface{} = &buf
if rows.Next() {
    rows.Scan(lob)
}
rows.Close()

// 写入文件
ioutil.WriteFile("output.bin", *(lob.(*[]byte)), 0644)
```

### 日期时间处理

```go
import "time"

// 插入当前时间
_, err = db.Exec("INSERT INTO date_test VALUES(?, ?)", 1, time.Now())
```

## DB 接口参考

| 方法 | 说明 |
|------|------|
| `sql.Open(driver, dsn)` | 打开数据库连接池 |
| `db.Ping()` | 检测连接 |
| `db.PingContext(ctx)` | 带 Context 检测连接 |
| `db.Exec(query, args...)` | 执行 DML/DDL |
| `db.ExecContext(ctx, query, args...)` | 带 Context 执行 |
| `db.Query(query, args...)` | 查询多行 |
| `db.QueryContext(ctx, query, args...)` | 带 Context 查询多行 |
| `db.QueryRow(query, args...)` | 查询单行 |
| `db.QueryRowContext(ctx, query, args...)` | 带 Context 查询单行 |
| `db.Prepare(query)` | 预编译 SQL |
| `db.PrepareContext(ctx, query)` | 带 Context 预编译 |
| `db.Begin()` | 开始事务 |
| `db.Conn(ctx)` | 获取独立连接 |
| `db.Close()` | 关闭连接池 |
| `db.SetMaxOpenConns(n)` | 设置最大连接数 |
| `db.SetMaxIdleConns(n)` | 设置最大空闲连接数 |
| `db.SetConnMaxLifetime(d)` | 设置连接最大生存时间 |

## Stmt 接口参考

| 方法 | 说明 |
|------|------|
| `stmt.Exec(args...)` | 执行预编译 SQL |
| `stmt.Query(args...)` | 预编译查询多行 |
| `stmt.QueryRow(args...)` | 预编译查询单行 |
| `stmt.Close()` | 关闭预编译语句 |

## Rows 接口参考

| 方法 | 说明 |
|------|------|
| `rows.Next()` | 移动到下一行，返回 bool |
| `rows.Scan(dest...)` | 扫描当前行到目标变量 |
| `rows.Columns()` | 获取列名列表 |
| `rows.NextResultSet()` | 切换到下一个结果集 |
| `rows.Close()` | 关闭结果集 |

## Tx 接口参考

| 方法 | 说明 |
|------|------|
| `tx.Commit()` | 提交事务 |
| `tx.Rollback()` | 回滚事务 |
| `tx.Exec(query, args...)` | 事务内执行 |
| `tx.Query(query, args...)` | 事务内查询 |
| `tx.QueryRow(query, args...)` | 事务内单行查询 |
| `tx.Prepare(query)` | 事务内预编译 |
| `tx.Stmt(stmt)` | 在事务中使用已有 Stmt |
