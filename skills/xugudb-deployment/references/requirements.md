# 软硬件环境需求

## 操作系统支持

| 操作系统 | CPU 架构 |
|----------|----------|
| Windows 7/10/11 | x86_64 |
| Red Hat Enterprise Linux 8.x | x86_64, aarch64 |
| 麒麟 V10 SP1/SP2/SP3 | x86_64, aarch64 |
| 统信 UOS V20 | x86_64, aarch64 |
| SUSE Enterprise Linux | x86_64, aarch64 |
| Debian | x86_64, aarch64 |
| Ubuntu | x86_64, aarch64 |
| 凝思 LinxOS | x86_64, aarch64 |
| 龙蜥 AnolisOS | x86_64, aarch64 |
| 中科方德 NFSChina | x86_64, aarch64 |
| Fedora | x86_64, aarch64 |

> XuguDB 同时支持 Intel x86_64 和 ARM aarch64 架构

## 安装包类型

| 安装包 | 内容 | 说明 |
|--------|------|------|
| XuguDB-12.0.0-linux-x86_64-*.zip | Client + Docker + Driver + Server | 全量包（Windows/Linux） |
| XuguDB-Server-12.9.9-linux-x86_64-*.zip | 仅 Server | 服务端独立包 |
| XuguDB-Server-12.9.9-linux-aarch64-*.zip | 仅 Server (ARM) | ARM 架构服务端 |

## 依赖库

### OpenSSL

XuguDB 依赖 OpenSSL 库。如系统未安装，需从以下地址获取：

- https://pkgs.org/search/?q=openssl
- https://rpmfind.net/linux/rpm2html/search.php?query=openssl

## 环境检查清单

### 磁盘 I/O 检查

```bash
# 测试 8K 块顺序写入速度
dd bs=8k count=4k if=/dev/zero of=test01 oflag=dsync

# 建议指标：
# 8K 块写入 >= 10MB/s（最低 3.5MB/s）
```

### 网络检查

```bash
# 查看网卡信息
ethtool enp189s0f0

# 测试延迟（建议 < 0.1ms）
ping 10.28.20.151

# 大包丢包率测试（确认 0% 丢包）
ping -s 10240 192.168.1.13 -c 50000 -f
```

### 网络参数优化

```bash
# /etc/sysctl.conf
net.core.rmem_default = 20971520
net.core.wmem_default = 20971520
net.core.rmem_max = 83886080
net.core.wmem_max = 83886080

# 生效
sysctl -p /etc/sysctl.conf
```

### 防火墙配置

```bash
# 方式1：关闭防火墙
systemctl stop firewalld.service
systemctl disable firewalld.service

# 方式2：开放端口（推荐生产环境）
firewall-cmd --zone=public --add-port=5138/tcp --permanent
firewall-cmd --reload
```

> 注意：集群部署还需开放 UDP 端口（cluster.ini 中 PORTS 端口 +20）

### SELinux

```bash
# 检查状态
getenforce

# 临时关闭
setenforce 0

# 永久关闭
vi /etc/selinux/config
# SELINUX=disabled
```

### 时间同步（集群必须）

```bash
# 主节点配置（chrony）
vi /etc/chrony.conf
# allow 192.168.1.0/16
# local stratum 10
systemctl restart chronyd.service

# 从节点配置
vi /etc/chrony.conf
# server 192.168.1.12 iburst
systemctl restart chronyd.service

# 验证同步
chronyc sources -v
```

### SSH 免密登录（集群部署）

```bash
# 生成密钥
ssh-keygen -t rsa

# 分发公钥到其他节点
ssh-copy-id -i ~/.ssh/id_rsa.pub xugu@192.168.1.13

# 验证
ssh xugu@192.168.1.13
```

### 资源限制

```bash
# /etc/security/limits.conf
xugu soft nofile 65536
xugu hard nofile 65536
xugu soft nproc 65536
xugu hard nproc 65536
```

### 端口检查

```bash
# 检查 5138 端口是否被占用
netstat -anp | grep 5138
```

## 硬件建议

| 组件 | 最低要求 | 生产推荐 |
|------|----------|----------|
| CPU | 2 核 | 8+ 核 |
| 内存 | 4 GB | 32+ GB |
| 磁盘 | 20 GB | SSD 500+ GB |
| 网络 | 100 Mbps | 万兆网络 |
