# 部署流程

## 完整部署步骤

```
1. 确认软硬件环境 → 2. 获取安装包 → 3. 校验包完整性 → 4. 申请 License →
5. 创建系统用户 → 6. 解压部署 → 7. 初始化配置 → 8. 配置参数 → 9. 启动 → 10. 验证
```

## Step 1: 获取安装包

XuguDB 安装包包含：Client（客户端）、Docker（容器文件）、Driver（驱动）、Server（服务端）

| 平台 | 包名示例 |
|------|----------|
| Linux x86_64 全量包 | XuguDB-12.0.0-linux-x86_64-*.zip |
| Linux ARM 全量包 | XuguDB-12.0.0-linux-aarch64-*.zip |
| Windows 全量包 | XuguDB-12.0.0-windows-amd64-*.zip |
| Linux x86_64 Server | XuguDB-Server-12.9.9-linux-x86_64-*.zip |
| Linux ARM Server | XuguDB-Server-12.9.9-linux-aarch64-*.zip |

## Step 2: 校验包完整性

```bash
# 选择以下任一方式校验
md5sum XuguDB-*.zip
sha1sum XuguDB-*.zip
sha256sum XuguDB-*.zip
sha512sum XuguDB-*.zip
```

将计算结果与官方提供的校验值对比，确保一致。

## Step 3: 申请和激活 License

### License 类型

| 版本 | License 要求 |
|------|-------------|
| 个人版 | 免费，有效期 360 天 |
| 标准版 | 需申请 |
| 企业版 | 需申请 |
| 分布式版 | 需申请（3~N 节点） |
| 安全版 | 需申请 |

### 申请流程

1. 获取服务器硬件信息（MAC 地址、CPU 序列号等）
2. 提交给虚谷数据库厂商
3. 获取 License 文件
4. 将 License 文件放入安装目录

> 注意：License 与硬件绑定，更换服务器需重新申请

### License 激活

将 License 文件放入 Server 目录后启动数据库即自动加载。

## Step 4: 创建系统用户

```bash
# 创建专用用户（不建议使用 root 运行数据库）
useradd xugu
passwd xugu

# 切换到 xugu 用户
su xugu
```

## Step 5: 解压与初始化

```bash
# 解压
tar -zxvf XuguDB-12.8_*-linux-x64.tar.gz
# 或
unzip XuguDB-Server-12.9.9-linux-x86_64-*.zip

# 初始化配置文件（首次部署）
cd XuguDB/BIN
./xugu_linux_x86_64_* --init-setup
# 在 SETUP/ 下生成 xugu.ini, mount.ini, types.ini, trust.ini

# 集群部署还需生成 cluster.ini 模板
./xugu_linux_x86_64_* --init-cluster
# 在 SETUP/ 下生成 cluster.ini.temp
```

## Step 6: 配置关键参数

### xugu.ini 核心参数

```ini
listen_port = 5138;        -- 监听端口
task_thd_num = 16;         -- 工作线程数（建议 CPU核数*2）
max_parallel = 2;          -- 最大并行度
cata_parti_num = 32;       -- 目录分区数（建议 CPU核数）
rsync_thd_num = 8;         -- 同步线程数
pass_mode = 3;             -- 密码策略模式
```

### mount.ini 存储配置

```ini
/XGLOG ./XGLOG
/CATA ./XHOME/CATA
/DATA ./XHOME/DATA
/TEMP ./XHOME/TEMP
/REDO ./XHOME/REDO
/BACKUP ./XHOME/BACKUP
/ARCH ./XHOME/ARCH
/UNDO ./XHOME/UNDO
/MODI ./XHOME/XMODI
# 可添加独立数据盘
/DATA1 /data1/data
/TEMP1 /data1/temp
```

### datafile.ini 数据文件分布

```ini
#DATA_FILES
/DATA1/DATA1.DBF
/DATA1/DATA2.DBF
/DATA1/DATA3.DBF
/DATA1/DATA4.DBF
#TEMP_FILES
/TEMP1/TEMP1.DBF
/TEMP1/TEMP2.DBF
```

> 将数据文件分布到多块磁盘可提高 I/O 性能

## Step 7: 启动与验证

```bash
# 前台启动
./xugu_linux_x86_64_* --child

# 后台启动
./xugu_linux_x86_64_* --server

# 验证连接
xgconsole -s nssl -h 127.0.0.1 -P 5138 -d SYSTEM -u SYSDBA -p SYSDBA
SQL> SELECT 1 FROM DUAL;
```

## 清理旧集群

```bash
# Windows
# 1. 登录 SYSTEM 库
# 2. SHUTDOWN IMMEDIATE;
# 3. 删除安装目录

# Linux
# 1. 登录 SYSTEM 库
SQL> SHUTDOWN IMMEDIATE;
# 2. 删除数据目录
rm -rf /path/to/XHOME /path/to/XGLOG /path/to/XVOCA
```

> 警告：此操作不可逆，执行前确保数据已备份
