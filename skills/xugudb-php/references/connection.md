# PHP PDO 连接管理

## DSN 格式

```
xugusql:ip=<地址>;port=<端口>;db=<数据库>
```

## DSN 参数

| 参数 | 说明 | 必填 |
|------|------|------|
| ip | 服务器 IP 地址 | 是 |
| ips | 多 IP 负载均衡（逗号分隔） | 否 |
| port | 端口号（默认 5138） | 否 |
| db | 数据库名 | 是 |
| user | 用户名（也可通过构造函数） | 否 |
| pwd | 密码（也可通过构造函数） | 否 |
| char_set | 字符集 | 否 |
| auto_commit | 自动提交（on/off） | 否 |
| use_srvcursor | 服务端游标（on/off） | 否 |
| usessl | SSL 加密（on/off） | 否 |

## 创建连接

```php
try {
    $pdo = new PDO(
        "xugusql:ip=127.0.0.1;port=5138;db=SYSTEM",
        "SYSDBA",  // 用户名
        "SYSDBA"   // 密码
    );
    echo "连接成功";
} catch (PDOException $e) {
    echo "连接失败: " . $e->getMessage();
}
```

### 带选项的连接

```php
$pdo = new PDO(
    "xugusql:ip=127.0.0.1;port=5138;db=SYSTEM",
    "SYSDBA",
    "SYSDBA",
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_AUTOCOMMIT => true
    ]
);
```

## 连接属性操作

```php
// 获取属性
$autocommit = $pdo->getAttribute(PDO::ATTR_AUTOCOMMIT);
$driverName = $pdo->getAttribute(PDO::ATTR_DRIVER_NAME);

// 设置属性
$pdo->setAttribute(PDO::ATTR_AUTOCOMMIT, false);

// 检查是否在事务中
$inTrans = $pdo->inTransaction();
```

## 错误处理

```php
// 获取错误码
echo $pdo->errorCode();

// 获取错误详情
print_r($pdo->errorInfo());
```

## 查看可用驱动

```php
print_r(PDO::getAvailableDrivers());
// 应包含 "xugusql"
```

## 常用 PDO 属性常量

| 常量 | 说明 |
|------|------|
| PDO::ATTR_AUTOCOMMIT | 自动提交开关 |
| PDO::ATTR_CASE | 列名大小写 |
| PDO::ATTR_CLIENT_VERSION | 客户端版本 |
| PDO::ATTR_CONNECTION_STATUS | 连接状态 |
| PDO::ATTR_DRIVER_NAME | 驱动名称 |
| PDO::ATTR_ERRMODE | 错误模式 |
| PDO::ATTR_PERSISTENT | 持久连接 |
| PDO::ATTR_SERVER_INFO | 服务器信息 |
| PDO::ATTR_SERVER_VERSION | 服务器版本 |
| PDO::ATTR_TIMEOUT | 超时时间 |

## 注意事项

- PDO 构造函数失败时抛出 PDOException
- 建议设置 `PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION` 便于错误处理
- 使用 `$pdo = null` 或脚本结束时自动关闭连接
