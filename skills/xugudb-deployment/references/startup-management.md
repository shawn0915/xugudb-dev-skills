# 启动参数与管理

## 启动模式

| 参数 | 说明 | 适用场景 |
|------|------|----------|
| `--child` / `-child` | 前台模式，占用当前终端 | 开发调试、Docker 容器 |
| `--server` / `-server` | 后台模式，systemd 管理 | 生产环境（Linux） |
| `--daemon` / `-daemon` | 守护模式，异常退出自动重启 | 无 systemd 的环境 |

### 前台启动

```bash
./xugu_linux_x86_64_* --child
# 按 Ctrl+C 停止
```

### 后台启动

```bash
./xugu_linux_x86_64_* --server
# 通过 systemd 管理
```

### 使用启动脚本

```bash
# startdb.sh 内容
#!/usr/bin/bash
echo "启动虚谷数据库..."
chmod +x xugu_linux_x86_64_*
$PWD/xugu_linux_x86_64_* --child

# 执行
./startdb.sh
```

## 启动参数详解

### 信息查询参数

| 参数 | 说明 | 示例输出 |
|------|------|----------|
| `--help` / `-h` | 帮助信息 | 所有参数说明 |
| `--version` / `-v` | 版本号 | `XuguDB 12.0.0` |
| `--build-info` | 编译时间 | `2025-06-19 12:00:00` |
| `--kernel-version` | 内核版本 | `12.9.7` |
| `--start-path` | 启动路径 | 程序安装绝对路径 |

### 初始化参数

| 参数 | 说明 |
|------|------|
| `--init-setup` | 在 SETUP/ 下生成 xugu.ini, mount.ini, types.ini, trust.ini |
| `--init-cluster` | 在 SETUP/ 下生成 cluster.ini.temp 模板 |
| `--reload-package` | 重新加载系统包 |

### 运行参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-D=<path>` | 指定工作目录 | `-D="/data/xugudb"` |
| `--start-ini=<kv>` | 启动时覆盖 xugu.ini 参数 | `--start-ini="listen_port=10578,min_pass_len=10"` |
| `--key-signal=<val>` | 键盘信号（备升主） | `--key-signal='a'` |

### 组合使用

```bash
# 指定端口 + 前台启动
./xugu_linux_x86_64_* --start-ini="listen_port=10578" --child

# 初始化配置时指定端口
./xugu_linux_x86_64_* --start-ini="listen_port=10578" --init-setup

# 指定独立数据目录
./xugu_linux_x86_64_* -D="/data1/XGDBMS" --child
```

## 停止数据库

### SQL 方式

```sql
-- 正常关闭（等待所有事务完成后关闭）
SHUTDOWN;

-- 立即关闭（中断当前事务，立即关闭）
SHUTDOWN IMMEDIATE;
```

### 外部方式

```bash
# 通过 xgconsole 远程关闭
xgconsole -s nssl -h 127.0.0.1 -P 5138 -d SYSTEM -u SYSDBA -p SYSDBA \
    -e "SHUTDOWN IMMEDIATE;"

# 前台模式直接 Ctrl+C
```

> 注意：生产环境建议使用 `SHUTDOWN`（正常关闭），非必要不用 `SHUTDOWN IMMEDIATE`

## Windows 服务管理

```cmd
# 安装为 Windows 服务
xugu_windows_amd64_*.exe --install

# 启动服务
xugu_windows_amd64_*.exe --start

# 停止服务
xugu_windows_amd64_*.exe --stop

# 卸载服务
xugu_windows_amd64_*.exe --remove
```

> Windows 下需使用管理员权限的 CMD 或 PowerShell

## systemd 服务配置（Linux）

```ini
# /etc/systemd/system/xugudb.service
[Unit]
Description=XuguDB Database Server
After=network.target

[Service]
Type=simple
User=xugu
Group=xugu
ExecStart=/opt/xugudb/BIN/xugu_linux_x86_64_20250327 --child
ExecStop=/opt/xugudb/BIN/xgconsole -s nssl -h 127.0.0.1 -P 5138 -d SYSTEM -u SYSDBA -p SYSDBA -e "SHUTDOWN IMMEDIATE;"
Restart=on-failure
RestartSec=10
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

```bash
# 启用并启动
systemctl daemon-reload
systemctl enable xugudb
systemctl start xugudb

# 管理
systemctl status xugudb
systemctl stop xugudb
systemctl restart xugudb
```

## 启动日志示例

```
XuguDB 12.0.0 (Built:2025-03-27 12:00:00 GA)
Start at: 2025-06-24 15:38:17
database server PID=34864
CPU info: Node 0: 0 1 2 3 ... 31
          Node 1: 32 33 34 ... 63
Resource limit:
  File number: hard limit=10240, soft limit=10240
  File size(M): hard limit=-1, soft limit=-1
Loading license ... ok.
init data buffer...ok.
Perform control file checksum...ok.
open all tablespaces...ok.
Load all local stores...ok.
Load all gstores...ok.
Mount all gstore...ok.
Open all system tables...ok
Open all system views...ok
Load all jobs...ok
All service started.
Listening at port 5138
```

> 如果出现 `Loading license ... failed!` 则为试用版（360 天）或 License 过期
