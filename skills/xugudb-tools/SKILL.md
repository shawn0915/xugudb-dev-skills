---
name: 虚谷数据库客户端工具
name_for_command: xugudb-tools
description: |
  XuguDB 客户端工具使用指南：XGConsole 命令行工具（连接、SQL 执行、管理操作）、
  DBeaver 图形化客户端集成。适用于日常数据库管理和开发操作。
tags: xugudb, tools, xgconsole, dbeaver, cli, gui, client
---

# 虚谷数据库客户端工具

## XGConsole 命令行工具

XGConsole 是虚谷数据库官方命令行客户端，支持 SQL 执行、数据库管理等操作，可在 Linux 和 Windows 上运行。

### 安装

| 平台 | CPU 架构 | 版本 |
|------|----------|------|
| Windows | x86, 64-bit | 2.2.7+ |
| Linux | x86, 64-bit | 2.2.7+ |
| Linux | ARM, 64-bit | 2.2.7+ |

**Windows 安装：**
1. 解压 xgconsole.exe
2. 设置环境变量 `XGCONSOLE_HOME`
3. 将 `%XGCONSOLE_HOME%` 添加到 `Path`
4. 验证：`xgconsole -V`

**Linux 安装：**
```bash
export XGCONSOLE_HOME=/opt/xgconsole
export PATH=$PATH:$XGCONSOLE_HOME
xgconsole -V
```

### 连接

```bash
xgconsole -s <ssl_mode> -h <ip> -P <port> -d <db> -u <user> -p <password> [-e <command>]
```

| 参数 | 说明 | 默认值 |
|------|------|--------|
| -s | SSL 模式（nssl/ssl） | nssl |
| -h | 服务器 IP | - |
| -P | 端口号 | 5138 |
| -d | 数据库名 | SYSTEM |
| -u | 用户名 | - |
| -p | 密码 | - |
| -e | 直接执行 SQL（不进入交互模式） | - |

### 示例

```bash
# 交互模式登录
xgconsole -s nssl -h 127.0.0.1 -P 5138 -d SYSTEM -u SYSDBA -p SYSDBA

# 直接执行 SQL
xgconsole -s nssl -h 127.0.0.1 -P 5138 -d SYSTEM -u SYSDBA -p SYSDBA -e "SELECT 1 FROM DUAL;"

# SSL 连接
xgconsole -s ssl -h 192.168.1.1 -P 5138 -d SYSTEM -u SYSDBA -p SYSDBA
```

### 交互模式功能

进入交互模式后，可以执行：
- SQL 语句（以 `;` 结尾）
- PL/SQL 块（以 `/` 结尾）
- DDL/DML/DQL 操作
- 数据库管理命令

> 详细参考：[XGConsole 使用指南](references/xgconsole.md)

## DBeaver 图形化客户端

DBeaver 是通用数据库管理工具，通过 JDBC 连接各种数据库。虚谷数据库提供 DBeaver 插件（xugu-dbeaver）实现集成。

### 安装步骤

1. **安装 DBeaver**：从 GitHub 下载 DBeaver Community Edition
   - Windows: `dbeaver-ce-*-x86_64-setup.exe`
   - Linux/MacOS: 对应平台包

2. **安装虚谷插件**：
   - 获取 `xugu-dbeaver` 插件包
   - 放入 DBeaver 插件目录
   - 重启 DBeaver

3. **创建连接**：
   - 新建数据库连接 → 选择 XuguDB
   - 填写：主机 IP、端口（5138）、数据库名、用户名、密码
   - 测试连接

### 功能

- SQL 编辑器：语法高亮、自动补全
- 数据浏览器：表数据浏览与编辑
- ER 图：查看表关系
- 数据导入/导出
- SQL 执行计划查看

> 详细参考：[DBeaver 集成指南](references/dbeaver.md)

## 工作流程

当用户咨询客户端工具相关问题时：

1. 确定使用场景（命令行管理 / 图形化操作 / 脚本自动化）
2. 命令行场景推荐 XGConsole，图形化场景推荐 DBeaver
3. 提供连接参数和操作示例
4. 对自动化脚本场景，使用 `-e` 参数直接执行 SQL

## 参考文档

- [XGConsole 使用指南](references/xgconsole.md) — 安装、连接、交互命令、常用操作
- [DBeaver 集成指南](references/dbeaver.md) — 插件安装、连接配置、功能使用
