# Go 驱动连接管理

## 连接字符串格式

虚谷 Go 驱动使用分号分隔的键值对格式（非标准 DSN/URL 格式）：

```
IP=<地址>;DB=<数据库>;User=<用户>;PWD=<密码>;Port=<端口>;AUTO_COMMIT=on;CHAR_SET=UTF8
```

## 连接参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| IP | 服务器 IP 地址 | 无（必填） |
| IPS | 多 IP 负载均衡，逗号分隔多个 IP | 无 |
| PORT | 端口号 | 5138 |
| DBNAME / DB | 数据库名 | SYSTEM |
| USER / User | 用户名 | 无（必填） |
| PASSWORD / PWD | 密码 | 无（必填） |
| CHAR_SET | 字符集编码 | 无（建议 UTF8） |
| AUTO_COMMIT | 自动提交 | on |

## 创建连接

```go
db, err := sql.Open("xugu",
    "IP=127.0.0.1;DB=SYSTEM;User=SYSDBA;PWD=SYSDBA;Port=5138;CHAR_SET=UTF8")
if err != nil {
    log.Fatal(err)
}
defer db.Close()
```

## 连接池配置

`sql.Open()` 返回的 `*sql.DB` 是一个连接池，不是单个连接。可配置：

```go
// 最大打开连接数（0 = 无限制）
db.SetMaxOpenConns(10)

// 最大空闲连接数（默认 2）
// 如果 <= 0，不保留空闲连接
db.SetMaxIdleConns(5)

// 连接最大生存时间（<= 0 表示永不过期）
duration, _ := time.ParseDuration("30m")
db.SetConnMaxLifetime(duration)
```

## 连接检测

```go
// Ping 检测连接是否可用
err = db.Ping()
if err != nil {
    fmt.Println("连接失败:", err)
}

// 带 Context 的 Ping
err = db.PingContext(context.Background())
```

## 获取独立连接

```go
// 从连接池获取一个专用连接
conn, err := db.Conn(context.Background())
if err != nil {
    log.Fatal(err)
}
defer conn.Close()
```

## 负载均衡

通过 `IPS` 参数配置多个服务器 IP 实现负载均衡：

```go
db, err := sql.Open("xugu",
    "IPS=192.168.1.10,192.168.1.11,192.168.1.12;DB=SYSTEM;User=SYSDBA;PWD=SYSDBA;Port=5138;CHAR_SET=UTF8")
```

## 注意事项

- `sql.Open()` 不会立即建立连接，首次查询或调用 `Ping()` 时才建立
- 使用 `defer db.Close()` 确保连接池正确关闭
- 生产环境建议配置连接池参数，避免连接泄漏
- 多 goroutine 可安全共享同一个 `*sql.DB` 实例
