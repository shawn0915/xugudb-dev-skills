# 产品与部署常见问题

## License 问题

### 个人版 License

- 免费使用 360 天
- 过期后数据库进入只读模式
- 可重新申请 License

### License 过期处理

1. 获取新的 License 文件
2. 替换旧的 License 文件
3. 重启数据库服务
4. 验证：检查是否可以执行写入操作

### I/O 性能问题

- 检查 License 是否过期（过期会导致 I/O 降级）
- 确认 License 类型与部署模式匹配

## 兼容性

### 支持的操作系统

| 操作系统 | CPU 架构 | 支持 |
|----------|----------|------|
| Linux | Intel x86-64 | 是 |
| Linux | ARM-64 | 是 |
| Windows | Intel x86-64 | 是 |

### 与其他数据库的兼容性

| 特性 | Oracle | MySQL | PostgreSQL |
|------|--------|-------|-----------|
| PL/SQL | 高度兼容 | - | - |
| SQL 语法 | 部分兼容 | 部分兼容 | 部分兼容 |
| ROWNUM | 支持 | - | - |
| SEQUENCE | 支持 | - | 支持 |
| CONNECT BY | 支持 | - | 不支持 |
| 数据类型 | 高度兼容 | 部分兼容 | 部分兼容 |

### 性能参数

| 参数 | 值 |
|------|-----|
| 最大并发连接 | 2000 |
| 最大并发用户 | 100 |
| TINYINT 范围 | -128 ~ 127 |

## 部署问题

### 端口配置

- 默认端口：5138
- 修改方法：编辑 `xugu.ini` 中的 `port` 参数
- 检查端口占用：`netstat -tlnp | grep 5138`（Linux）

### Page Size

- 在创建数据库时设置
- 创建后不可修改
- 影响大对象存储和查询性能
- 建议根据业务数据特征选择

### UDP 端口

- 虚谷使用 UDP 进行集群节点间心跳检测
- 确保防火墙放行相关 UDP 端口
- 如果出现 `bind-udp-recv-port-0-failed` 错误，检查 UDP 端口是否被占用

### 启动与关闭

```bash
# 启动（Linux）
cd /opt/xugudb/bin
./xugu_server &

# 关闭
xgconsole -s nssl -h 127.0.0.1 -P 5138 -d SYSTEM -u SYSDBA -p SYSDBA -e "SHUTDOWN;"

# 或立即关闭
xgconsole -s nssl -h 127.0.0.1 -P 5138 -d SYSTEM -u SYSDBA -p SYSDBA -e "SHUTDOWN IMMEDIATE;"
```

### 密码重置

如果 SYSDBA 密码忘记：
1. 停止数据库服务
2. 使用单用户模式启动
3. 重置密码
4. 正常重启

## 数据库类型说明

| 版本 | 说明 | 适用场景 |
|------|------|----------|
| 标准版 | 单机部署 | 中小规模业务 |
| 企业版 | 双节点主备 | 高可用需求 |
| 分布式版 | 3~N 节点 | 大规模数据 |
| 安全版 | 增强安全 | 涉密系统 |

### 主备模式

- **主库（Primary）**：处理所有读写请求
- **备库（Replicas）**：只读副本，用于容灾
- 通过 `enable_read_copy` 参数开启备库读
