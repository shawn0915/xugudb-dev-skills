---
name: 虚谷数据库 ODBC 驱动开发
name_for_command: xugudb-odbc
description: |
  XuguDB ODBC 驱动配置与开发指南：ODBC 驱动安装、DSN 配置、连接字符串格式、
  通过 ODBC 接口连接虚谷数据库进行 CRUD 操作。适用于需要通过 ODBC 标准接口
  访问虚谷数据库的跨语言场景（C/C++、Python pyodbc、Node.js、Excel、BI 工具等）。
  注：本技能基于虚谷 ODBC 驱动和 ODBC 标准接口推导，无独立 ODBC 开发文档。
tags: xugudb, odbc, driver, dsn, cross-language, bi-tools
---

# 虚谷数据库 ODBC 驱动开发

虚谷数据库提供标准 ODBC 驱动，支持通过 ODBC 接口连接虚谷数据库。ODBC 是跨语言、跨平台的数据库访问标准，广泛用于 BI 工具、报表工具和多语言应用开发。

## ODBC 驱动安装

### Windows

1. 获取虚谷 ODBC 驱动安装包
2. 运行安装程序，按提示完成安装
3. 驱动自动注册到系统 ODBC Driver Manager

### Linux

1. 安装 unixODBC：
   ```bash
   # Ubuntu/Debian
   apt-get install unixodbc unixodbc-dev

   # CentOS/RHEL
   yum install unixODBC unixODBC-devel
   ```
2. 获取虚谷 ODBC 驱动（.so 文件）
3. 配置 `/etc/odbcinst.ini` 注册驱动

> 详细参考：[ODBC 驱动安装](references/installation.md)

## DSN 配置

### Windows — 通过 ODBC 数据源管理器

1. 打开"ODBC 数据源管理器"（控制面板或搜索）
2. 选择"系统 DSN"或"用户 DSN"
3. 点击"添加"，选择"XuguDB ODBC Driver"
4. 填写连接信息：
   - DSN 名称
   - 服务器 IP 地址
   - 端口（默认 5138）
   - 数据库名
   - 用户名/密码

### Linux — 配置 odbc.ini

```ini
# /etc/odbc.ini 或 ~/.odbc.ini
[XuguDB]
Driver = XuguDB ODBC Driver
Server = 127.0.0.1
Port = 5138
Database = SYSTEM
UID = SYSDBA
PWD = SYSDBA
```

```ini
# /etc/odbcinst.ini
[XuguDB ODBC Driver]
Driver = /opt/xugudb/lib/libxuguodbc.so
Setup = /opt/xugudb/lib/libxuguodbc.so
```

> 详细参考：[DSN 配置](references/connection.md)

## 连接字符串格式

无 DSN 连接（DSN-less）：

```
DRIVER={XuguDB ODBC Driver};SERVER=127.0.0.1;PORT=5138;DATABASE=SYSTEM;UID=SYSDBA;PWD=SYSDBA
```

DSN 连接：

```
DSN=XuguDB;UID=SYSDBA;PWD=SYSDBA
```

## 跨语言使用示例

### C/C++ (ODBC API)

```c
#include <sql.h>
#include <sqlext.h>

SQLHENV henv;
SQLHDBC hdbc;
SQLHSTMT hstmt;

SQLAllocHandle(SQL_HANDLE_ENV, SQL_NULL_HANDLE, &henv);
SQLSetEnvAttr(henv, SQL_ATTR_ODBC_VERSION, (void*)SQL_OV_ODBC3, 0);
SQLAllocHandle(SQL_HANDLE_DBC, henv, &hdbc);

SQLDriverConnect(hdbc, NULL,
    "DRIVER={XuguDB ODBC Driver};SERVER=127.0.0.1;PORT=5138;DATABASE=SYSTEM;UID=SYSDBA;PWD=SYSDBA",
    SQL_NTS, NULL, 0, NULL, SQL_DRIVER_NOPROMPT);

SQLAllocHandle(SQL_HANDLE_STMT, hdbc, &hstmt);
SQLExecDirect(hstmt, "SELECT * FROM test", SQL_NTS);
```

### Python (pyodbc)

```python
import pyodbc

conn = pyodbc.connect(
    'DRIVER={XuguDB ODBC Driver};SERVER=127.0.0.1;PORT=5138;DATABASE=SYSTEM;UID=SYSDBA;PWD=SYSDBA'
)
cursor = conn.cursor()
cursor.execute("SELECT * FROM test")
for row in cursor.fetchall():
    print(row)
conn.close()
```

### Node.js (odbc)

```javascript
const odbc = require('odbc');
const conn = await odbc.connect('DSN=XuguDB;UID=SYSDBA;PWD=SYSDBA');
const result = await conn.query('SELECT * FROM test');
await conn.close();
```

> 详细参考：[CRUD 操作](references/crud-operations.md)

## 适用场景

| 场景 | 说明 |
|------|------|
| BI 工具 | Tableau、Power BI 等通过 ODBC 连接 |
| 报表工具 | Crystal Reports、JasperReports 等 |
| Excel | 通过"获取外部数据"功能连接 |
| 跨语言应用 | C/C++、Python、Node.js、Perl 等 |
| ETL 工具 | Informatica、Talend 等 |
| 数据迁移 | 通过 ODBC 桥接不同数据库 |

## 工作流程

当用户咨询 ODBC 相关问题时：

1. 确定使用场景（应用开发 / BI 工具 / 数据迁移）
2. 指导 ODBC 驱动安装和 DSN 配置
3. 根据目标语言/工具提供连接示例
4. 标注虚谷特有的连接参数

## 参考文档

- [ODBC 驱动安装](references/installation.md) — Windows/Linux 驱动安装步骤
- [DSN 与连接配置](references/connection.md) — DSN 配置、连接字符串参数
- [CRUD 操作](references/crud-operations.md) — 跨语言 ODBC CRUD 示例
