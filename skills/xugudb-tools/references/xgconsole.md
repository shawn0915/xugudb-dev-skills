# XGConsole 使用指南

## 安装配置

### Windows

1. 下载 xgconsole 安装包（确保 CPU 架构匹配）
2. 解压到安装目录（如 `C:\xgconsole`）
3. 设置环境变量：
   - `XGCONSOLE_HOME` = 安装目录
   - Path 中添加 `%XGCONSOLE_HOME%`
4. 验证：打开 CMD 执行 `xgconsole -V`

### Linux

```bash
# 解压到目标目录
tar -xzf xgconsole-*.tar.gz -C /opt/xgconsole

# 配置环境变量
echo 'export XGCONSOLE_HOME=/opt/xgconsole' >> ~/.bash_profile
echo 'export PATH=$PATH:$XGCONSOLE_HOME' >> ~/.bash_profile
source ~/.bash_profile

# 验证
xgconsole -V
```

> 注意：安装包 CPU 架构需与操作系统匹配

## 连接命令

```bash
xgconsole -s <ssl_mode> -h <ip> -P <port> -d <db> -u <user> -p <password> [-e <command>]
```

### 参数说明

| 参数 | 说明 | 示例 |
|------|------|------|
| -s | SSL 模式 | nssl（不加密）/ ssl（加密） |
| -h | 服务器 IP | 127.0.0.1 |
| -P | 端口号 | 5138 |
| -d | 数据库名 | SYSTEM |
| -u | 用户名 | SYSDBA |
| -p | 密码 | SYSDBA |
| -e | 直接执行的 SQL 命令 | "SELECT 1 FROM DUAL;" |

### 连接示例

```bash
# 标准连接（交互模式）
xgconsole -s nssl -h 127.0.0.1 -P 5138 -d SYSTEM -u SYSDBA -p SYSDBA

# SSL 连接
xgconsole -s ssl -h 192.168.1.100 -P 5138 -d SYSTEM -u SYSDBA -p SYSDBA

# 免密连接（交互输入密码）
xgconsole -s nssl -h 127.0.0.1 -P 5138 -d SYSTEM -u SYSDBA

# 直接执行 SQL（脚本化）
xgconsole -s nssl -h 127.0.0.1 -P 5138 -d SYSTEM -u SYSDBA -p SYSDBA \
  -e "SELECT COUNT(*) FROM USER_TABLES;"
```

## 交互模式操作

进入交互模式后：

```sql
-- SQL 语句以分号结尾
SQL> SELECT * FROM test;
SQL> INSERT INTO test VALUES (1, 'hello');

-- PL/SQL 块以 / 结尾
SQL> BEGIN
  2>   DBMS_OUTPUT.PUT_LINE('Hello XuguDB');
  3> END;
  4> /

-- DDL 操作
SQL> CREATE TABLE demo (id INT, name VARCHAR(50));
SQL> DROP TABLE demo;
```

## 脚本化使用

```bash
# 批量执行 SQL 文件
xgconsole -s nssl -h 127.0.0.1 -P 5138 -d SYSTEM -u SYSDBA -p SYSDBA \
  -e "$(cat init.sql)"

# 结合 shell 脚本
#!/bin/bash
XGCONSOLE="xgconsole -s nssl -h 127.0.0.1 -P 5138 -d SYSTEM -u SYSDBA -p SYSDBA"
$XGCONSOLE -e "SELECT COUNT(*) FROM user_tables;"
```

## 注意事项

- Windows 版本为 .exe 可执行文件
- Linux 版本需确保有执行权限（`chmod +x xgconsole`）
- CPU 架构必须与操作系统匹配（x86-64 或 ARM-64）
- 默认端口 5138
- 使用 `-e` 参数时 SQL 需以分号结尾
