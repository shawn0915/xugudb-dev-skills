# DBeaver 集成虚谷数据库

## 概述

DBeaver 是开源的通用数据库管理工具，支持通过 JDBC 连接各种数据库。虚谷数据库提供专用的 DBeaver 插件（xugu-dbeaver）实现无缝集成。

## 安装 DBeaver

### 下载

从 GitHub Releases 下载 DBeaver Community Edition：

| 平台 | 文件 |
|------|------|
| Windows (安装版) | dbeaver-ce-*-x86_64-setup.exe |
| Windows (免安装) | dbeaver-ce-*-win32.win32.x86_64.zip |
| Linux | dbeaver-ce-*-linux.gtk.x86_64.tar.gz |
| MacOS | dbeaver-ce-*-macos-x86_64.dmg |

### 安装

- Windows: 运行安装程序或解压免安装包
- Linux: 解压 tar.gz 包，运行 `dbeaver` 可执行文件
- MacOS: 拖拽到 Applications

## 安装虚谷插件

1. 获取 `xugu-dbeaver` 插件包（从虚谷官方渠道获取）
2. 将插件放入 DBeaver 的 plugins 目录
3. 重启 DBeaver
4. 验证：新建连接时应能看到 "XuguDB" 选项

> 注意：如果 DBeaver 中看不到 XuguDB 选项，检查插件是否正确放置

## 创建连接

1. 点击"新建数据库连接"（或 Ctrl+Shift+N）
2. 搜索或选择 "XuguDB"
3. 填写连接参数：

| 参数 | 值 |
|------|-----|
| 主机 | 127.0.0.1 |
| 端口 | 5138 |
| 数据库 | SYSTEM |
| 用户名 | SYSDBA |
| 密码 | SYSDBA |

4. 点击"测试连接"验证
5. 点击"完成"保存连接

## 常用功能

### SQL 编辑器

- 新建 SQL 编辑器：Ctrl+] 或右键连接 → SQL 编辑器
- 执行 SQL：Ctrl+Enter（执行当前语句）或 Alt+X（执行全部）
- 语法高亮和自动补全

### 数据浏览

- 双击表名打开数据查看器
- 支持在线编辑数据
- 支持排序、过滤

### 数据导入/导出

- 右键表 → 导出数据（CSV、SQL、Excel 等格式）
- 右键表 → 导入数据

### ER 图

- 右键数据库/Schema → ER 图
- 可视化查看表关系

## 注意事项

- DBeaver 通过 JDBC 连接虚谷，需确保虚谷 JDBC 驱动正确配置
- 首次连接可能需要下载/配置 JDBC 驱动
- 默认端口 5138，不是其他数据库的常见端口
- xugu-dbeaver 插件版本需与 DBeaver 版本兼容
