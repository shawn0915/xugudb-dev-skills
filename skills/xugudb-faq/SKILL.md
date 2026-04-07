---
name: 虚谷数据库常见问题与错误码
name_for_command: xugudb-faq
description: |
  XuguDB 常见问题解答与错误码参考：SQL 使用FAQ、产品FAQ、部署FAQ、
  集群管理FAQ、高可用FAQ、完整错误码分类与解决方案。
  适用于问题排查、故障诊断和技术答疑。
tags: xugudb, faq, troubleshooting, error-codes, diagnostics
---

# 虚谷数据库常见问题与错误码

## SQL 常见问题

### 字符串中的单引号转义

```sql
-- 使用两个单引号表示一个单引号
INSERT INTO emp(name, city) VALUES ('O''Reilly', 'Beijing');
```

### GROUP BY 取每组最新记录

```sql
-- 方式1：窗口函数（推荐）
SELECT c1, c2 FROM (
    SELECT c1, c2, ROW_NUMBER() OVER (PARTITION BY c1 ORDER BY c2 DESC) AS rn
    FROM t
) t WHERE rn = 1;

-- 方式2：子查询 JOIN
SELECT e.* FROM t e
JOIN (SELECT c1, MAX(c2) AS max_c2 FROM t GROUP BY c1) m
ON e.c1 = m.c1 AND e.c2 = m.max_c2;
```

> 大表（>500万行）场景下，JOIN 方式可能优于窗口函数

### IF NOT EXISTS 用法

```sql
-- 建表
CREATE TABLE IF NOT EXISTS t (id INT, name VARCHAR(50));

-- 删表
DROP TABLE IF EXISTS t;

-- 建索引
CREATE INDEX IF NOT EXISTS idx_name ON t (name);
```

> 详细参考：[SQL FAQ](references/sql-faq.md)

## 产品常见问题

### License 相关

- **个人版 License**：免费使用 360 天
- **License 过期**：数据库只读，无法写入
- **License 更新**：替换 License 文件后重启

### 兼容性

- 虚谷数据库兼容 SQL 标准和部分 Oracle/MySQL 语法
- 支持 Intel x86-64 和 ARM-64 架构
- 支持 Linux 和 Windows 操作系统
- 支持最多 2000 个并发连接，100 个并发用户

### 与 Oracle 兼容要点

- 支持 PL/SQL 存储过程/函数/触发器/包
- 支持 ROWNUM 伪列
- 支持 CONNECT BY 层次查询
- 支持 SEQUENCE
- 主备模式类似 Oracle DataGuard

> 详细参考：[产品与部署 FAQ](references/product-faq.md)

## 部署常见问题

### 端口冲突

- 默认端口 5138
- 检查端口占用：`netstat -tlnp | grep 5138`
- 修改端口：编辑 `xugu.ini` 中的 `port` 参数

### Page Size 配置

- 创建数据库时设置，后续不可修改
- 根据数据特征选择合适的 Page Size

### UDP 端口

- 虚谷数据库使用 UDP 进行节点间心跳通信
- 确保防火墙放行 UDP 端口

### 关闭数据库

```sql
-- 正常关闭（等待事务完成）
SHUTDOWN;

-- 立即关闭（中断当前事务）
SHUTDOWN IMMEDIATE;
```

> 详细参考：[产品与部署 FAQ](references/product-faq.md)

## 集群管理常见问题

### 节点角色

| 角色 | 简称 | 职责 |
|------|------|------|
| Master | M | 集群管理，必须配置 2 个 |
| Query | Q | 查询处理 |
| Work | W | 工作计算 |
| Storage | S | 数据存储 |
| Gather | G | 变更收集（可选，最多 2 个） |

### 节点故障处理

```sql
-- 查看节点状态
SHOW CLUSTERS;
SELECT * FROM SYS_CLUSTERS;

-- 移除故障节点
ALTER CLUSTER DROP NODE node_id;

-- 重新添加节点
ALTER CLUSTER ADD NODE node_id DESCRIBE 'node_description';
```

### 日志排查

| 日志文件 | 路径 | 用途 |
|----------|------|------|
| ERROR.LOG | /Server/XGLOG/ | 错误和异常 |
| EVENT.LOG | /Server/XGLOG/ | 关键事件 |
| TRACE.LOG | /Server/XGLOG/ | 执行路径调试 |

> 详细参考：[集群与高可用 FAQ](references/cluster-ha-faq.md)

## 错误码分类

| 错误码范围 | 分类 | 说明 |
|------------|------|------|
| E1001-E1999 | 系统错误 | 内部错误、资源不足 |
| E2001-E2999 | 连接错误 | 连接超时、认证失败 |
| E3001-E3999 | SQL 语法错误 | 语法解析错误 |
| E4001-E4999 | 语义错误 | 对象不存在、类型不匹配 |
| E5001-E5999 | DDL 错误 | 表/索引/视图操作错误 |
| E6001-E6999 | DML 错误 | 插入/更新/删除错误 |
| E7001-E7999 | 约束错误 | 主键/外键/唯一性冲突 |
| E8001-E8999 | PL/SQL 错误 | 存储过程/函数错误 |
| E9001-E9999 | 对象管理错误 | 对象已存在/不存在 |
| E10001-E10999 | 权限错误 | 权限不足 |
| E11001-E11999 | 分布式错误 | 集群/节点错误 |
| E12001-E12999 | 索引错误 | 索引创建/删除错误 |
| E13001-E13999 | 唯一性冲突 | 唯一索引/主键冲突 |
| E14001-E14999 | 锁错误 | 死锁、锁超时 |
| E15001-E15999 | 备份恢复错误 | 备份/恢复操作错误 |
| E16001-E16999 | 约束违反 | NOT NULL、CHECK 等 |
| E17001-E17999 | 类型错误 | 数据类型相关错误 |
| E18001-E18999 | 权限详细错误 | 具体权限操作错误 |
| E19001-E19999 | 对象详细错误 | 触发器/同义词等错误 |
| E20001-E20999 | 事务错误 | 事务管理错误 |
| E21001-E21999 | 空间数据错误 | GIS/空间功能错误 |
| E22001-E22999 | 向量错误 | 向量功能错误 |
| E24001-E24999 | 审计错误 | 审计相关错误 |

### 常见错误码

| 错误码 | 含义 | 解决方案 |
|--------|------|----------|
| E5021 | 表不存在 | 检查表名拼写和 Schema |
| E5025 | 表有依赖对象 | 使用 CASCADE 删除 |
| E9016 | 对象已存在 | 使用 IF NOT EXISTS |
| E12008 | 索引已存在 | 使用 IF NOT EXISTS |
| E13001 | 唯一性冲突 | 检查重复数据 |
| E16005 | NOT NULL 约束违反 | 提供非空值 |
| E17002 | 类型不存在 | 检查数据类型名 |
| E18012 | 权限不足 | 联系 DBA 授权 |
| E19067 | 不支持的同义词目标 | 同义词不能指向触发器 |

> 详细参考：[错误码参考](references/error-codes.md)

## 工作流程

当用户遇到问题时：

1. 根据错误码或现象定位问题分类
2. 查找对应的 FAQ 或错误码说明
3. 提供具体的解决方案和 SQL 示例
4. 对不确定的问题建议查看日志文件（ERROR.LOG）

## 参考文档

- [SQL FAQ](references/sql-faq.md) — SQL 语法问题、查询优化、常用技巧
- [产品与部署 FAQ](references/product-faq.md) — License、兼容性、部署配置
- [集群与高可用 FAQ](references/cluster-ha-faq.md) — 集群管理、高可用、故障处理
- [错误码参考](references/error-codes.md) — 完整错误码分类与解决方案
