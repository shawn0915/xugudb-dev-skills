# XGCI (Xugu Call Interface) 简化接口指南

## 概述

XGCI 是虚谷数据库自主设计的简化 C 接口，相比 NCI/OCI 具有更少的句柄层级和更简洁的 API。适合不需要 Oracle 兼容性的新项目。

## 环境要求

- C/C++ 编译器
- XGCI 头文件（xgci.h）和库文件
- 平台：Windows x86-64 / Linux x86-64 / Linux ARM-64

## 句柄模型

XGCI 使用四级句柄链：

```
HT_ENV → HT_SERVER → HT_SESSION → HT_STATEMENT
```

| 句柄类型 | 说明 | 父句柄 |
|----------|------|--------|
| HT_ENV | 环境句柄 | NULL |
| HT_SERVER | 服务器句柄 | HT_ENV |
| HT_SESSION | 会话句柄 | HT_SERVER |
| HT_STATEMENT | 语句句柄 | HT_SESSION |

## 初始化

```c
XGCIHANDLE henv, hserver, hsession, hstmt;

XGCIHandleAlloc(NULL, &henv, HT_ENV);
XGCIHandleAlloc(henv, &hserver, HT_SERVER);
XGCIHandleAlloc(hserver, &hsession, HT_SESSION);
XGCIHandleAlloc(hsession, &hstmt, HT_STATEMENT);
```

## 连接

### 设置连接属性

```c
unsigned short port = 5138;
XGCIHandleAttrSet(hserver, XGCI_ATTR_SRV_IP, "127.0.0.1", XGCI_NTS);
XGCIHandleAttrSet(hserver, XGCI_ATTR_SRV_PORT, &port, 0);
XGCIHandleAttrSet(hserver, XGCI_ATTR_SRV_DBNAME, "SYSTEM", XGCI_NTS);
```

### 连接属性常量

| 常量 | 说明 | 值类型 |
|------|------|--------|
| XGCI_ATTR_SRV_IP | 服务器 IP | 字符串 |
| XGCI_ATTR_SRV_PORT | 端口号 | unsigned short* |
| XGCI_ATTR_SRV_DBNAME | 数据库名 | 字符串 |

### 登录

```c
int ret = XGCISessionBegin(hsession, "SYSDBA", "SYSDBA");
if (ret != XGCI_SUCCESS) {
    char errCode[7] = {0};
    char errMsg[256] = {0};
    int len = 0;
    XGCIErrors(hsession, errCode, errMsg, &len);
    printf("[%s] %s\n", errCode, errMsg);
}
```

## 执行 SQL

### 直接执行（XGCIExecDirect）

```c
// DDL
XGCIExecDirect(hstmt, "CREATE TABLE test(id INT, name VARCHAR(20))", XGCI_NTS);

// DML
XGCIExecDirect(hstmt, "INSERT INTO test VALUES(1, 'hello')", XGCI_NTS);

// 重置语句以复用
XGCIFreeStmt(hstmt, XG_RESET);
```

### 查询

```c
const char* sql = "SELECT id, name, created FROM test";
int ret = XGCIExecDirect(hstmt, (char*)sql, XGCI_NTS);

if (ret == XGCI_SUCCESS) {
    int id;
    char name[20];
    DATE_STRUCT date;
    int rc[3], len[3];

    // 定义输出列
    XGCIDefineByPos(hstmt, 1, &id, 4, XG_C_INTEGER, &rc[0], &len[0]);
    XGCIDefineByPos(hstmt, 2, name, 20, XG_C_CHAR, &rc[1], &len[1]);
    XGCIDefineByPos(hstmt, 3, &date, 20, XG_C_DATE, &rc[2], &len[2]);

    // 遍历结果
    while (XGCIFetch(hstmt) == XGCI_SUCCESS) {
        printf("%d, %s, %d-%d-%d\n", id, name, date.year, date.month, date.day);
    }
}
```

## XGCI 数据类型常量

| 常量 | 说明 |
|------|------|
| XG_C_INTEGER | 整数 |
| XG_C_FLOAT | 浮点数 |
| XG_C_DOUBLE | 双精度浮点 |
| XG_C_CHAR | 字符数组 |
| XG_C_DATE | DATE_STRUCT 日期 |
| XG_C_TIMESTAMP | TIMESTAMP_STRUCT 时间戳 |
| XG_C_BINARY | 二进制数据 |

## 日期结构

```c
typedef struct {
    short year;
    unsigned short month;
    unsigned short day;
} DATE_STRUCT;

typedef struct {
    short year;
    unsigned short month;
    unsigned short day;
    unsigned short hour;
    unsigned short minute;
    unsigned short second;
    unsigned int fraction;
} TIMESTAMP_STRUCT;
```

## 错误处理

```c
char errCode[7] = {0};
char errMsg[256] = {0};
int len = 0;

XGCIErrors(handle, errCode, errMsg, &len);
printf("[%s] %s\n", errCode, errMsg);
```

## 资源释放

```c
XGCIHandleFree(hstmt);       // 释放语句
XGCISessionEnd(hsession);    // 结束会话
XGCIHandleFree(hserver);     // 释放服务器
XGCIHandleFree(henv);        // 释放环境
```

## 核心 API 参考

| 函数 | 说明 | 返回值 |
|------|------|--------|
| XGCIHandleAlloc(parent, &handle, type) | 分配句柄 | XGCI_SUCCESS |
| XGCIHandleFree(handle) | 释放句柄 | XGCI_SUCCESS |
| XGCIHandleAttrSet(handle, attr, value, len) | 设置属性 | XGCI_SUCCESS |
| XGCISessionBegin(session, user, pwd) | 登录 | XGCI_SUCCESS |
| XGCISessionEnd(session) | 退出 | XGCI_SUCCESS |
| XGCIExecDirect(stmt, sql, len) | 直接执行 SQL | XGCI_SUCCESS |
| XGCIDefineByPos(stmt, pos, buf, buflen, type, &rc, &len) | 定义输出列 | XGCI_SUCCESS |
| XGCIFetch(stmt) | 获取一行结果 | XGCI_SUCCESS/XGCI_NO_DATA |
| XGCIFreeStmt(stmt, option) | 重置/关闭语句 | XGCI_SUCCESS |
| XGCIErrors(handle, errCode, errMsg, &len) | 获取错误信息 | - |

## XGCI 与 NCI/OCI 对比

| 操作 | NCI/OCI | XGCI |
|------|---------|------|
| 初始化句柄 | 6 种句柄逐个分配 | 4 级链式分配 |
| 连接 | ServerAttach + AttrSet + SessionBegin | AttrSet + SessionBegin |
| 执行 SQL | StmtPrepare + StmtExecute | ExecDirect（一步） |
| 重置语句 | 重新 Prepare | FreeStmt(XG_RESET) |
| API 复杂度 | 高（兼容 Oracle） | 低（虚谷简化） |
