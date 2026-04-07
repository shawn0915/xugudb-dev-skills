---
name: 虚谷数据库 PHP 驱动开发
name_for_command: xugudb-php
description: |
  XuguDB PHP PDO 驱动开发完整指南：XuguDB-PDO 驱动安装（Windows/Linux）、
  PDO 连接管理、CRUD 操作（exec/query/prepare）、参数绑定（bindValue/bindParam）、
  事务控制、存储过程/函数调用、大对象处理、结果集获取、PDO 常量参考。
  适用于使用 PHP 连接和操作虚谷数据库的开发场景。
tags: xugudb, php, pdo, driver, web-development
---

# 虚谷数据库 PHP 驱动开发

XuguDB-PDO 是虚谷数据库的官方 PHP 驱动程序，基于 PHP PDO 扩展接口实现，支持标准 PDO API。

## 驱动安装

### Windows
1. 安装 PHP（https://windows.php.net/downloads/releases/）
2. 将 `pdo_xugusql.dll` 放入 PHP 的 `ext` 目录
3. 修改 `php.ini`：
   ```ini
   extension_dir = "ext"
   extension=pdo_xugusql
   ```
4. 验证：`php -m` 查看是否包含 `pdo_xugusql`

### Linux
1. 安装 PHP（编译安装或包管理器）
2. 将 `pdo_xugusql.so` 放入 PHP 扩展目录
3. 修改 `php.ini`：
   ```ini
   extension=pdo_xugusql
   ```
4. 验证：`php -m` 查看是否包含 `pdo_xugusql`

> 详细参考：[安装与环境搭建](references/installation.md)

## 连接管理

DSN 格式：
```
xugusql:ip=<地址>;port=<端口>;db=<数据库>
```

```php
$pdo = new PDO("xugusql:ip=127.0.0.1;port=5138;db=SYSTEM", "SYSDBA", "SYSDBA");
```

**DSN 参数：**

| 参数 | 说明 |
|------|------|
| ip | 服务器 IP 地址 |
| ips | 多 IP 负载均衡 |
| port | 端口号（默认 5138） |
| db | 数据库名 |
| user/pwd | 用户名/密码（也可通过构造函数参数传入） |
| char_set | 字符集 |
| auto_commit | 自动提交（on/off） |
| use_srvcursor | 服务端游标（on/off） |
| usessl | SSL 加密（on/off） |

> 详细参考：[连接管理](references/connection.md)

## CRUD 操作

### 执行 DML（exec）

```php
$affected = $pdo->exec("INSERT INTO test (name) VALUES('Tom')('Jerry')");
echo $affected; // 2
```

### 查询（query）

```php
$stmt = $pdo->query("SELECT * FROM test");
while ($row = $stmt->fetch()) {
    print_r($row);
}
```

### 预编译（prepare + execute）

```php
// 方式1：? 占位符 + bindValue
$stmt = $pdo->prepare("INSERT INTO test (name, created) VALUES(?, ?)");
$stmt->bindValue(1, "Tom");
$stmt->bindValue(2, "2000-01-01");
$stmt->execute();

// 方式2：命名参数 + bindValue
$stmt = $pdo->prepare("INSERT INTO test (name, created) VALUES(:name, :created)");
$stmt->bindValue("name", "Tom");
$stmt->bindValue("created", "2000-01-01");
$stmt->execute();

// 方式3：bindParam（绑定变量引用）
$stmt = $pdo->prepare("INSERT INTO test (name) VALUES(?)");
$name = "Tom";
$stmt->bindParam(1, $name);
$stmt->execute();
```

### 事务控制

```php
$pdo->beginTransaction();
try {
    $pdo->exec("INSERT INTO test (name) VALUES('Tom')");
    $pdo->commit();
} catch (PDOException $e) {
    $pdo->rollBack();
}
```

> 详细参考：[CRUD 操作](references/crud-operations.md)

## 高级特性

### 存储过程调用

```php
// CREATE OR REPLACE PROCEDURE proc_add(a int, b int, sum OUT int) ...
$stmt = $pdo->prepare("EXEC proc_add(?, ?, ?)");
$stmt->bindValue(1, 100);
$stmt->bindValue(2, 200);
$stmt->bindParam(3, $sum, PDO::PARAM_INPUT_OUTPUT);
$stmt->execute();
echo $sum; // 300
```

### 函数调用

```php
// CREATE OR REPLACE FUNCTION func_add(a int, b int) RETURN int ...
$stmt = $pdo->prepare("EXEC func_add(?, ?)");
$stmt->bindValue(1, 100);
$stmt->bindValue(2, 200);
$stmt->bindParam(3, $result, PDO::PARAM_INPUT_OUTPUT);
$stmt->execute();
echo $result; // 300
```

### 大对象（LOB）处理

```php
// 写入 BLOB
$lob = fopen("img.png", "rb");
$stmt = $pdo->prepare("INSERT INTO tab_lob (lob) VALUES (:lob)");
$stmt->bindValue(":lob", $lob, PDO::PARAM_LOB);
$stmt->execute();
fclose($lob);

// 读取 BLOB
$stmt = $pdo->query("SELECT lob FROM tab_lob WHERE id = 1");
$stmt->bindColumn(1, $lob, PDO::PARAM_LOB);
$stmt->fetch(PDO::FETCH_BOUND);
$file = fopen("output.png", "wb");
while (!feof($lob)) fwrite($file, fgets($lob));
fclose($file);
```

### 结果集获取方式

| 方法 | 说明 |
|------|------|
| `fetch()` | 获取单行 |
| `fetchAll()` | 获取所有行 |
| `fetchColumn(n)` | 获取单列值 |
| `fetchObject(class)` | 获取为对象 |
| `bindColumn()` | 绑定列到变量 |

### 多结果集

```php
$stmt = $pdo->query("SELECT * FROM t1; SELECT * FROM t2");
// 处理第一个结果集...
$stmt->nextRowset(); // 切换到下一个结果集
```

> 详细参考：[CRUD 操作](references/crud-operations.md)

## 与其他数据库 PHP PDO 的差异

| 特性 | XuguDB (pdo_xugusql) | MySQL (pdo_mysql) | PostgreSQL (pdo_pgsql) |
|------|---------------------|-------------------|----------------------|
| DSN 前缀 | `xugusql:` | `mysql:` | `pgsql:` |
| 存储过程调用 | `EXEC proc_name(?, ?)` | `CALL proc_name(?, ?)` | `SELECT proc_name(?, ?)` |
| LOB 绑定 | PDO::PARAM_LOB | PDO::PARAM_LOB | PDO::PARAM_LOB |
| 多结果集 | nextRowset() | nextRowset() | 不支持 |
| lastInsertId | 返回 rowid | 返回自增 ID | 需指定序列名 |

## 工作流程

当用户咨询 PHP 驱动开发相关问题时：

1. 确定问题类别（安装配置 / 连接管理 / CRUD / 事务 / 存储过程 / LOB）
2. 提供基于 PDO 标准接口的代码示例
3. 标注虚谷特有的 DSN 格式和存储过程调用语法
4. 对 LOB 和存储过程给出完整示例

## 参考文档

- [安装与环境搭建](references/installation.md) — Windows/Linux 环境配置、PDO 扩展安装
- [连接管理](references/connection.md) — DSN 格式、连接参数、SSL
- [CRUD 操作](references/crud-operations.md) — 增删改查、参数绑定、事务、存储过程、LOB、PDO 接口参考
