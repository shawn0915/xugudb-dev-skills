---
name: 虚谷数据库概览
name_for_command: xugudb
description: |
  虚谷数据库（XuguDB）产品概览、架构设计、版本选型与快速上手。
  适用场景：了解 XuguDB 产品特性、架构设计、版本差异、容量限制。
---

# 虚谷数据库（XuguDB）概览

虚谷数据库是成都虚谷伟业科技有限公司自主原创的高性能关系型数据库管理系统，基于 20 多年数据库领域研发经验，核心代码自有率 99.6%。

## 产品定位

- **国产自主可控**：通过 EAL4+ 安全认证、国密算法、涉密认证
- **原生分布式**：Shared-Nothing 架构，支持 1~512 节点
- **多模兼容**：兼容 Oracle/MySQL/PostgreSQL 语法，支持 SQL-92/99/2003
- **HTAP 混合负载**：同时支持 OLTP 和 OLAP 场景

## 架构特点

虚谷采用**单进程多线程 + Shared-Nothing** 架构，集群由四类角色组成：

| 角色 | 职责 |
|------|------|
| 主控管理 | 集群管理、心跳检测、全局锁仲裁 |
| 工作计算 | 接收用户请求、SQL 解析执行、结果返回 |
| 存储管理 | 数据持久化、8MB CHUNK 自动切片、多副本 |
| 变更收集 | 事务级变更信息收集（可选部署） |

关键设计：
- **中心管理-对等处理**：所有工作节点对等，连接任一节点即访问全部数据
- **存算融合/分离可选**：一个节点可同时承担多个角色
- **智能单元扫描（算子下推）**：计算向存储靠拢，减少网络开销

> 详细架构说明：[architecture](skills/xugudb/references/architecture.md)

## 产品版本

| 版本 | 集群规模 | 适用场景 |
|------|----------|----------|
| 标准版 | 单机 | 中小规模业务、网站、办公系统 |
| 企业版 | 双机 | 关键核心业务、双机热备 |
| 分布式版 | 3~N 机 | GB~PB 级大数据、HTAP 混合业务 |
| 安全版 | 单机/双机 | 高保密、涉密认证要求 |

> 详细版本对比：[product-versions](skills/xugudb/references/product-versions.md)

## 核心技术

| 能力 | 关键技术 |
|------|----------|
| 强一致 | 优化二阶段提交、数据流同步复制、可靠 UDP |
| 高可用 | 存储多副本（1主+2备）、工作节点对等、管理双活、多网络并行 |
| 高性能 | 无限制行级锁、MVCC、CBO 优化器、RDMA、NUMA 优化、并行扫描 |
| 高安全 | DAC + MAC（记录级）、三权分立、国密、EAL4+、黑白名单 |
| 高扩展 | 在线热加入、自动数据均衡（误差 ≤5%）、计算/存储独立扩展 |

> 详细核心技术：[core-technology](skills/xugudb/references/core-technology.md)

## 常用指标

| 指标 | 限制 |
|------|------|
| 最大节点数 | 512 |
| 单节点连接数 | 10,000 |
| 单库表数量 | 21 亿 |
| 字段数 | 2,040 |
| 记录长度 | 64KB |
| JOIN 表数 | 64 |
| 大对象 | 2GB |
| SQL 长度 | 2MB |

> 完整指标与对比：[common-metrics](skills/xugudb/references/common-metrics.md)

## 兼容性

- **SQL 标准**：SQL-92、SQL-99、SQL-2003
- **异构兼容**：Oracle / MySQL / PostgreSQL 模式
- **接口驱动**：JDBC / Python / C# / Go / PHP / Node.js / ODBC / NCI / OCI / XGCI
- **硬件平台**：X86、ARM、MIPS、POWER、SPARC
- **操作系统**：Windows、Linux、国产 OS（麒麟、统信、欧拉等）
- **字符集**：UTF-8、GB18030、GBK、GB2312、Binary

## 与 Oracle/MySQL/PG 关键差异

| 特性 | XuguDB 特点 | 迁移注意 |
|------|-------------|----------|
| 分布式架构 | 原生 Shared-Nothing | 无需额外中间件 |
| 自动分片 | 物理 CHUNK 自动切片，无需分片键 | 无需设计分片策略 |
| 锁机制 | 行级锁，不允许锁升级 | 无表锁升级风险 |
| 隔离级别 | 基于 MVCC 的 Read Committed | 与 Oracle 默认级别一致 |
| PL/SQL | 兼容 Oracle PL/SQL | 存储过程可较低成本迁移 |
| 安全模型 | DAC + MAC + 三权分立 | 比 Oracle/MySQL/PG 更严格 |

## 参考文档

- [产品简介](skills/xugudb/references/product-intro.md) — 产品定位、认证、发展历程
- [产品架构](skills/xugudb/references/architecture.md) — 无共享架构、集群角色、存算融合
- [产品版本](skills/xugudb/references/product-versions.md) — 四个版本功能对比与选型
- [核心技术](skills/xugudb/references/core-technology.md) — 一致性、高可用、高性能、安全、扩展
- [常用指标](skills/xugudb/references/common-metrics.md) — 容量限制与主流数据库对比
