# ODBC DSN 与连接配置

## DSN 配置

### Windows — 图形界面

1. 打开"ODBC 数据源管理器"
2. 选择"系统 DSN"（所有用户）或"用户 DSN"（当前用户）
3. 点击"添加"
4. 选择"XuguDB ODBC Driver"
5. 填写连接信息：

| 字段 | 值 | 说明 |
|------|-----|------|
| DSN Name | XuguDB | 数据源名称 |
| Server | 127.0.0.1 | 服务器 IP |
| Port | 5138 | 端口号 |
| Database | SYSTEM | 数据库名 |
| User | SYSDBA | 用户名 |
| Password | SYSDBA | 密码 |

### Linux — odbc.ini 文件

```ini
# /etc/odbc.ini（系统级）或 ~/.odbc.ini（用户级）
[XuguDB]
Description = XuguDB Database
Driver = XuguDB ODBC Driver
Server = 127.0.0.1
Port = 5138
Database = SYSTEM
UID = SYSDBA
PWD = SYSDBA
CharSet = UTF8
```

## 连接字符串格式

### DSN 连接

```
DSN=XuguDB;UID=SYSDBA;PWD=SYSDBA
```

### DSN-less 连接（无需预配置 DSN）

```
DRIVER={XuguDB ODBC Driver};SERVER=127.0.0.1;PORT=5138;DATABASE=SYSTEM;UID=SYSDBA;PWD=SYSDBA
```

## 连接参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| DRIVER | ODBC 驱动名称 | - |
| DSN | 数据源名称 | - |
| SERVER | 服务器 IP 地址 | - |
| PORT | 端口号 | 5138 |
| DATABASE/DB | 数据库名 | SYSTEM |
| UID/USER | 用户名 | - |
| PWD/PASSWORD | 密码 | - |
| CharSet | 字符集 | - |

## 连接测试

### Windows

在 ODBC 数据源管理器中配置 DSN 后，点击"测试连接"按钮。

### Linux

```bash
# 使用 isql 测试
isql -v XuguDB SYSDBA SYSDBA

# 成功后会进入 SQL 交互界面
SQL> SELECT 1 FROM DUAL;
```

## 注意事项

- DSN 名称区分大小写（Linux）
- 连接字符串中的驱动名称必须与 odbcinst.ini 中注册的名称一致
- 如果使用 DSN-less 连接，DRIVER 参数中的驱动名称需用花括号包围
- 端口号默认 5138，不是常见的 3306 或 5432
