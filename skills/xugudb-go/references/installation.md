# Go 驱动安装与环境搭建

## 环境要求

- Go 语言运行环境（`go` 命令可用）
- XuguDB Go Driver（go-xugu-driver），CPU 架构需与目标平台匹配
- 虚谷数据库服务端已安装并启动

## 安装驱动

### 方式一：go get（推荐）

```bash
go get gitee.com/XuguDB/go-xugu-driver@latest
```

### 方式二：本地导入

将驱动源码放入项目目录，使用相对路径导入：

```go
import (
    _ "./go-xugu-driver"    // 本地路径导入
    "database/sql"
)
```

## 项目初始化

```go
package main

import (
    _ "gitee.com/XuguDB/go-xugu-driver"
    "database/sql"
    "log"
    "fmt"
)

func main() {
    db, err := sql.Open("xugu",
        "IP=127.0.0.1;DB=SYSTEM;User=SYSDBA;PWD=SYSDBA;Port=5138;CHAR_SET=UTF8")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    err = db.Ping()
    if err != nil {
        fmt.Println("连接失败:", err)
    } else {
        fmt.Println("连接成功")
    }
}
```

## 注意事项

- 驱动通过匿名导入（`_ "gitee.com/XuguDB/go-xugu-driver"`）注册到 `database/sql`
- 驱动名称为 `"xugu"`，在 `sql.Open()` 第一个参数中使用
- 连接字符串使用分号分隔的键值对格式，非标准 DSN 格式
- 建议设置 `CHAR_SET=UTF8` 避免中文乱码
