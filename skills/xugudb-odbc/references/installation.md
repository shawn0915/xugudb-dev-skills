# ODBC 驱动安装

## Windows 安装

1. 从虚谷数据库官方获取 ODBC 驱动安装包
2. 运行安装程序，按照向导完成安装
3. 安装完成后，驱动自动注册到 Windows ODBC Driver Manager
4. 可通过"ODBC 数据源管理器"验证驱动是否安装成功

验证方法：
- 打开"ODBC 数据源管理器"（开始菜单搜索"ODBC"）
- 切换到"驱动程序"标签页
- 确认列表中有"XuguDB ODBC Driver"

## Linux 安装

### 1. 安装 unixODBC

```bash
# Ubuntu/Debian
sudo apt-get install unixodbc unixodbc-dev

# CentOS/RHEL
sudo yum install unixODBC unixODBC-devel

# 验证安装
odbcinst -j
```

### 2. 安装虚谷 ODBC 驱动

将虚谷 ODBC 驱动库文件（如 `libxuguodbc.so`）放到指定目录：

```bash
sudo cp libxuguodbc.so /opt/xugudb/lib/
sudo chmod 755 /opt/xugudb/lib/libxuguodbc.so
```

### 3. 注册驱动

编辑 `/etc/odbcinst.ini`：

```ini
[XuguDB ODBC Driver]
Description = XuguDB ODBC Driver
Driver = /opt/xugudb/lib/libxuguodbc.so
Setup = /opt/xugudb/lib/libxuguodbc.so
UsageCount = 1
```

或使用命令注册：

```bash
odbcinst -i -d -f xugudb_driver.template
```

### 4. 验证

```bash
odbcinst -q -d
# 应显示 [XuguDB ODBC Driver]
```

## 注意事项

- ODBC 驱动架构（32/64位）需与应用程序匹配
- Windows 64 位系统有两个 ODBC 管理器（32位和64位），需选择正确的版本
- Linux 上确保 unixODBC 版本与驱动兼容
- 驱动文件路径中不要有中文或空格
