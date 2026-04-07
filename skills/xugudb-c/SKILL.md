---
name: 虚谷数据库 C 语言驱动开发
name_for_command: xugudb-c
description: |
  XuguDB C 语言驱动开发完整指南，涵盖三种 C 接口：
  NCI（虚谷原生接口，兼容 Oracle OCI 命名风格）、
  OCI（Oracle OCI 兼容接口，便于 Oracle 应用迁移）、
  XGCI（虚谷简化接口，API 更简洁）。
  适用于需要高性能 C/C++ 应用连接虚谷数据库的场景。
tags: xugudb, c, nci, oci, xgci, native, driver
---

# 虚谷数据库 C 语言驱动开发

虚谷数据库提供三种 C 语言接口，适用于不同场景：

| 接口 | 全称 | 定位 | 适用场景 |
|------|------|------|----------|
| **NCI** | Native Call Interface | 虚谷原生 C 接口 | 新项目首选，API 风格类似 Oracle OCI |
| **OCI** | Oracle Call Interface | Oracle 兼容接口 | Oracle C 应用迁移到虚谷 |
| **XGCI** | Xugu Call Interface | 虚谷简化接口 | 追求简洁 API 的 C/C++ 项目 |

## NCI 与 OCI 的对比

NCI 和 OCI 接口几乎相同，仅前缀不同（NCI → OCI）：

| NCI | OCI | 说明 |
|-----|-----|------|
| NCIEnvCreate | OCIEnvCreate | 创建环境句柄 |
| NCIHandleAlloc | OCIHandleAlloc | 分配句柄 |
| NCIServerAttach | OCIServerAttach | 连接服务器 |
| NCISessionBegin | OCISessionBegin | 开始会话 |
| NCIStmtPrepare | OCIStmtPrepare | 预编译 SQL |
| NCIStmtExecute | OCIStmtExecute | 执行 SQL |
| NCIBindByName | OCIBindByName | 按名绑定参数 |
| NCIDefineByPos | OCIDefineByPos | 按位定义输出列 |
| NCIStmtFetch | OCIStmtFetch | 获取结果行 |

> 从 Oracle OCI 迁移到虚谷：只需将 OCI 前缀替换为 NCI（或直接使用 OCI 兼容层）

## XGCI 简化接口

XGCI 使用四级句柄模型，API 更简洁：

```
HT_ENV → HT_SERVER → HT_SESSION → HT_STATEMENT
```

核心 API：
- `XGCIHandleAlloc()` — 分配句柄
- `XGCIHandleAttrSet()` — 设置连接属性（IP/端口/库名）
- `XGCISessionBegin()` — 登录
- `XGCIExecDirect()` — 直接执行 SQL
- `XGCIDefineByPos()` — 定义输出列
- `XGCIFetch()` — 获取结果
- `XGCISessionEnd()` — 退出会话
- `XGCIHandleFree()` — 释放句柄

## 快速示例 — NCI

```c
#include "nci.h"

NCIEnv* env; NCIError* err; NCISvcCtx* ctx; NCIStmt* stmt;

// 初始化
NCIEnvCreate(&env, NCI_DEFAULT, NULL, NULL, NULL, NULL, 0, NULL);
NCIHandleAlloc(env, &err, NCI_HTYPE_ERROR, 0, NULL);

// 连接（简化方式）
NCILogon(env, err, &ctx, "SYSDBA", 6, "SYSDBA", 6, "127.0.0.1:5138/SYSTEM", 22);

// 预编译执行
NCIHandleAlloc(env, &stmt, NCI_HTYPE_STMT, 0, NULL);
NCIStmtPrepare(stmt, err, "SELECT id FROM test", 20, NCI_NTV_SYNTAX, NCI_DEFAULT);

int id;
NCIDefine* def;
NCIDefineByPos(stmt, &def, err, 1, &id, 4, SQLT_INT, NULL, NULL, NULL, NCI_DEFAULT);
NCIStmtExecute(ctx, stmt, err, 0, 0, NULL, NULL, NCI_DEFAULT);

while (NCIStmtFetch(stmt, err, 1, NCI_FETCH_NEXT, NCI_DEFAULT) == NCI_SUCCESS) {
    printf("%d\n", id);
}

// 释放
NCIHandleFree(stmt, NCI_HTYPE_STMT);
NCILogoff(ctx, err);
NCIHandleFree(err, NCI_HTYPE_ERROR);
NCIHandleFree(env, NCI_HTYPE_ENV);
```

## 快速示例 — XGCI

```c
#include "xgci.h"

XGCIHANDLE henv, hserver, hsession, hstmt;
unsigned short port = 5138;

// 初始化句柄链
XGCIHandleAlloc(NULL, &henv, HT_ENV);
XGCIHandleAlloc(henv, &hserver, HT_SERVER);

// 设置连接属性
XGCIHandleAttrSet(hserver, XGCI_ATTR_SRV_IP, "127.0.0.1", XGCI_NTS);
XGCIHandleAttrSet(hserver, XGCI_ATTR_SRV_PORT, &port, 0);
XGCIHandleAttrSet(hserver, XGCI_ATTR_SRV_DBNAME, "SYSTEM", XGCI_NTS);

// 创建会话和语句
XGCIHandleAlloc(hserver, &hsession, HT_SESSION);
XGCIHandleAlloc(hsession, &hstmt, HT_STATEMENT);

// 登录
XGCISessionBegin(hsession, "SYSDBA", "SYSDBA");

// 直接执行 SQL
XGCIExecDirect(hstmt, "SELECT id, name FROM test", XGCI_NTS);

int id; char name[20];
int rc[2], len[2];
XGCIDefineByPos(hstmt, 1, &id, 4, XG_C_INTEGER, &rc[0], &len[0]);
XGCIDefineByPos(hstmt, 2, name, 20, XG_C_CHAR, &rc[1], &len[1]);

while (XGCIFetch(hstmt) == XGCI_SUCCESS) {
    printf("%d, %s\n", id, name);
}

// 释放
XGCIHandleFree(hstmt);
XGCISessionEnd(hsession);
XGCIHandleFree(hserver);
XGCIHandleFree(henv);
```

## 工作流程

当用户咨询 C 语言驱动开发相关问题时：

1. 确定使用哪种接口（NCI/OCI/XGCI）
2. 如果是 Oracle 迁移场景，推荐 OCI 兼容接口
3. 如果是新项目，根据需求推荐 NCI（功能全面）或 XGCI（简洁易用）
4. 提供完整的句柄初始化 → 连接 → 操作 → 释放代码模板
5. 标注参数绑定和结果定义的类型常量

## 参考文档

- [NCI 接口指南](references/nci-guide.md) — NCI 句柄管理、连接、SQL 执行、参数绑定、完整示例
- [OCI 接口指南](references/oci-guide.md) — OCI 兼容接口、Oracle 迁移指南
- [XGCI 接口指南](references/xgci-guide.md) — XGCI 简化接口、句柄模型、直接执行、完整示例
