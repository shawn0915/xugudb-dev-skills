# PHP PDO CRUD 操作与接口参考

## 执行 DML（exec）

```php
// INSERT/UPDATE/DELETE，返回影响行数
$affected = $pdo->exec("INSERT INTO test (name) VALUES('Tom')");
echo "影响行数: $affected";

// 多行插入
$affected = $pdo->exec("INSERT INTO test (name) VALUES('Tom')('Jerry')");
echo $affected; // 2
```

## 查询（query）

```php
$stmt = $pdo->query("SELECT * FROM test");
while ($row = $stmt->fetch()) {
    echo "{$row['ID']}, {$row['NAME']}" . PHP_EOL;
}
```

## 预编译与参数绑定

### ? 占位符 + bindValue

```php
$stmt = $pdo->prepare("INSERT INTO test (name, created) VALUES(?, ?)");
$stmt->bindValue(1, "Tom");
$stmt->bindValue(2, "2000-01-01");
$stmt->execute();
```

### 命名参数 + bindValue

```php
$stmt = $pdo->prepare("INSERT INTO test (name, created) VALUES(:name, :created)");
$stmt->bindValue("name", "Tom");
$stmt->bindValue("created", "2000-01-01");
$stmt->execute();
```

### bindParam（绑定变量引用）

```php
$stmt = $pdo->prepare("INSERT INTO test (name) VALUES(?)");
$name = "Tom";
$stmt->bindParam(1, $name);  // 绑定引用，execute 时取 $name 当前值
$stmt->execute();

$name = "Jerry";  // 改变变量值
$stmt->execute();  // 插入 "Jerry"
```

> `bindValue` 绑定值的拷贝，`bindParam` 绑定变量的引用。

### 数组参数执行

```php
$stmt = $pdo->prepare("INSERT INTO test (name) VALUES(?)");
$stmt->execute(["Tom"]);
```

## 结果集获取

### fetch — 单行

```php
$stmt = $pdo->query("SELECT * FROM test");
while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
    echo $row['NAME'];
}
```

### fetchAll — 所有行

```php
$stmt = $pdo->query("SELECT * FROM test");
$rows = $stmt->fetchAll(PDO::FETCH_ASSOC);
```

### fetchColumn — 单列

```php
$stmt = $pdo->query("SELECT name FROM test");
while ($name = $stmt->fetchColumn()) {
    echo $name . PHP_EOL;
}
```

### fetchObject — 对象

```php
class User {
    private $id;
    private $name;
    public function getName() { return $this->name; }
}

$stmt = $pdo->query('SELECT id as "id", name as "name" FROM test');
while ($user = $stmt->fetchObject("User")) {
    echo $user->getName();
}
```

### bindColumn — 绑定列

```php
$stmt = $pdo->query("SELECT id, name FROM test");
$stmt->bindColumn(1, $id);
$stmt->bindColumn(2, $name);
while ($stmt->fetch(PDO::FETCH_BOUND)) {
    echo "$id, $name" . PHP_EOL;
}
```

## 获取插入 ID

```php
$pdo->exec("INSERT INTO test (name) VALUES('Tom')");
$id = $pdo->lastInsertId();  // 返回 rowid
```

## 事务控制

```php
$pdo->beginTransaction();
try {
    $stmt = $pdo->prepare("INSERT INTO test (name) VALUES(?)");
    $stmt->bindValue(1, "Tom");
    $stmt->execute();

    $stmt->bindValue(1, "Jerry");
    $stmt->execute();

    $pdo->commit();
} catch (PDOException $e) {
    $pdo->rollBack();
    echo "事务失败: " . $e->getMessage();
}
```

## 存储过程调用

```php
// 数据库中: CREATE OR REPLACE PROCEDURE proc_add(a INT, b INT, sum OUT INT)

$stmt = $pdo->prepare("EXEC proc_add(?, ?, ?)");
$stmt->bindValue(1, 100);
$stmt->bindValue(2, 200);
$stmt->bindParam(3, $sum, PDO::PARAM_INPUT_OUTPUT);
$stmt->execute();
echo $sum; // 300
```

## 函数调用

```php
// 数据库中: CREATE OR REPLACE FUNCTION func_add(a INT, b INT) RETURN INT

$stmt = $pdo->prepare("EXEC func_add(?, ?)");
$stmt->bindValue(1, 100);
$stmt->bindValue(2, 200);
$stmt->bindParam(3, $result, PDO::PARAM_INPUT_OUTPUT);
$stmt->execute();
echo $result; // 300
```

> 注意：虚谷 PHP 中调用存储过程/函数使用 `EXEC` 关键字（MySQL 用 `CALL`）

## 大对象（LOB）处理

### 写入 BLOB

```php
$lob = fopen("img.png", "rb");
$stmt = $pdo->prepare("INSERT INTO tab_lob (lob) VALUES (:lob)");
$stmt->bindValue(":lob", $lob, PDO::PARAM_LOB);
$stmt->execute();
fclose($lob);
```

### 读取 BLOB — 方式一（bindColumn）

```php
$stmt = $pdo->query("SELECT lob FROM tab_lob WHERE id = 1");
$stmt->bindColumn(1, $lob, PDO::PARAM_LOB);
$stmt->fetch(PDO::FETCH_BOUND);

$file = fopen("output.png", "wb");
while (!feof($lob)) {
    fwrite($file, fgets($lob));
}
fclose($file);
```

### 读取 BLOB — 方式二（fetch）

```php
$stmt = $pdo->query("SELECT lob FROM tab_lob WHERE id = 1");
$row = $stmt->fetch();

$file = fopen("output.png", "wb");
while (!feof($row[0])) {
    fwrite($file, fgets($row[0]));
}
fclose($file);
```

## 多结果集

```php
$stmt = $pdo->query("SELECT * FROM t1; SELECT * FROM t2");
// 处理第一个结果集
do {
    while ($row = $stmt->fetch()) {
        print_r($row);
    }
} while ($stmt->nextRowset());
```

## PDO 接口参考

| 方法 | 说明 |
|------|------|
| `new PDO(dsn, user, pass)` | 创建连接 |
| `exec(sql)` | 执行 DML，返回影响行数 |
| `query(sql)` | 执行查询，返回 PDOStatement |
| `prepare(sql)` | 预编译 SQL |
| `beginTransaction()` | 开始事务 |
| `commit()` | 提交事务 |
| `rollBack()` | 回滚事务 |
| `inTransaction()` | 是否在事务中 |
| `lastInsertId()` | 最后插入 ID |
| `quote(string)` | 转义字符串 |
| `errorCode()` | 错误码 |
| `errorInfo()` | 错误详情 |
| `getAttribute(attr)` | 获取属性 |
| `setAttribute(attr, val)` | 设置属性 |

## PDOStatement 接口参考

| 方法 | 说明 |
|------|------|
| `execute(params)` | 执行预编译语句 |
| `fetch(mode)` | 获取单行 |
| `fetchAll(mode)` | 获取所有行 |
| `fetchColumn(col)` | 获取单列 |
| `fetchObject(class)` | 获取为对象 |
| `bindValue(param, val)` | 绑定值 |
| `bindParam(param, &var)` | 绑定变量引用 |
| `bindColumn(col, &var)` | 绑定列到变量 |
| `rowCount()` | 影响行数 |
| `columnCount()` | 列数 |
| `closeCursor()` | 关闭游标 |
| `nextRowset()` | 切换结果集 |
| `getColumnMeta(col)` | 列元数据 |
| `errorCode()` | 错误码 |
| `errorInfo()` | 错误详情 |
| `debugDumpParams()` | 调试参数 |

## PDO 参数类型常量

| 常量 | 说明 |
|------|------|
| PDO::PARAM_NULL | SQL NULL |
| PDO::PARAM_BOOL | 布尔 |
| PDO::PARAM_INT | 整数 |
| PDO::PARAM_STR | 字符串（默认） |
| PDO::PARAM_LOB | 大对象 |
| PDO::PARAM_INPUT_OUTPUT | IN/OUT 参数 |

## Fetch 模式常量

| 常量 | 说明 |
|------|------|
| PDO::FETCH_ASSOC | 关联数组 |
| PDO::FETCH_NUM | 数字索引数组 |
| PDO::FETCH_BOTH | 两种（默认） |
| PDO::FETCH_OBJ | 匿名对象 |
| PDO::FETCH_BOUND | 绑定列模式 |
| PDO::FETCH_COLUMN | 单列模式 |
| PDO::FETCH_CLASS | 类实例模式 |
