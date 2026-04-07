# 其他语言 ORM 框架集成

## Go — GORM

### 要求

- GORM 1.20.0+
- `xggorm` 虚谷适配器
- `go-xugu-driver` 虚谷 Go 驱动

### 连接

```go
import (
    "xggorm"
    "fmt"
    "gorm.io/gorm"
    _ "gitee.com/XuguDB/go-xugu-driver"
)

func main() {
    dsn := "IP=127.0.0.1;DB=SYSTEM;User=SYSDBA;PWD=SYSDBA;Port=5138;CURRENT_SCHEMA=SYSTEM;AUTO_COMMIT=on;CHAR_SET=UTF8"
    db, err := gorm.Open(xggorm.Open(dsn), &gorm.Config{})
    if err != nil {
        fmt.Printf("连接失败: %v\n", err)
        return
    }
    fmt.Println("连接成功")
}
```

### 模型与 CRUD

```go
type User struct {
    ID   int    `gorm:"primaryKey;autoIncrement"`
    Name string `gorm:"type:varchar(100)"`
    Age  int
}

// 自动建表
db.AutoMigrate(&User{})

// CRUD
db.Create(&User{Name: "Tom", Age: 25})
var user User
db.First(&user, 1)
db.Model(&user).Update("Age", 26)
db.Delete(&user)
```

## Go — XORM

### 要求

- XORM 1.3.1
- 需将 `xugu.go` 放入 XORM 的 `dialects` 目录

### 连接

```go
import (
    "xorm.io/xorm"
    _ "gitee.com/XuguDB/go-xugu-driver"
)

engine, err := xorm.NewEngine("xugu",
    "IP=127.0.0.1;DB=SYSTEM;User=SYSDBA;PWD=SYSDBA;Port=5138;CHAR_SET=UTF8")
```

## Python — Django

### 要求

- Django 4.2.1+
- `xgcondb` 虚谷 Django 引擎

### settings.py

```python
DATABASES = {
    'default': {
        'ENGINE': 'xgcondb',
        'NAME': 'SYSTEM',
        'USER': 'SYSDBA',
        'PASSWORD': 'SYSDBA',
        'HOST': '127.0.0.1',
        'PORT': 5138,
    }
}
```

### 模型与 CRUD

```python
from django.db import models

class User(models.Model):
    name = models.CharField(max_length=100)
    age = models.IntegerField()

    class Meta:
        db_table = 'users'

# 建表
python manage.py migrate

# CRUD
User.objects.create(name='Tom', age=25)
users = User.objects.all()
User.objects.filter(id=1).update(age=26)
User.objects.filter(id=1).delete()
```

## Python — SQLAlchemy

### 要求

- SQLAlchemy 1.4.36
- `xugu-sqlalchemy` 虚谷方言插件

### 连接

```python
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

# 创建引擎（xugu:// 协议前缀）
engine = create_engine('xugu://SYSDBA:SYSDBA@127.0.0.1:5138/SYSTEM')
Session = sessionmaker(bind=engine)
session = Session()
```

## Python — peewee

### 要求

- peewee 3.17.1
- `xgpeewee` 虚谷适配器

### 连接

```python
from xgpeewee import *

db = XuguDatabase(
    database='SYSTEM',
    host='127.0.0.1',
    port=5138,
    user='SYSDBA',
    password='SYSDBA'
)
```

## PHP — ThinkPHP

### 要求

- ThinkPHP 5.0.1
- 需将 `Xugusql.php` builder 放入 ThinkPHP 的 builder 目录

### 数据库配置

```php
return [
    'type'     => 'xugusql',
    'hostname' => '127.0.0.1',
    'database' => 'SYSTEM',
    'username' => 'SYSDBA',
    'password' => 'SYSDBA',
    'hostport' => '5138',
    'charset'  => 'utf8',
];
```

## Node.js — Sequelize

### 要求

- Sequelize 6.37.3
- `xugu-dialect` 替换 mysql dialect

### 连接

```javascript
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize('SYSTEM', 'SYSDBA', 'SYSDBA', {
    host: '127.0.0.1',
    dialect: 'mysql',  // xugu-dialect 伪装为 mysql
    logging: false
});
```

详细配置参考 [xugudb-nodejs skill](../../xugudb-nodejs/SKILL.md)

## 适配器获取方式

所有 ORM 框架的虚谷适配器（dialect/adapter/engine）均需从虚谷官方渠道获取：

| 适配器 | 语言 | 框架 |
|--------|------|------|
| xugu-jdbc | Java | 所有 Java 框架 |
| xugu-dialect (Hibernate) | Java | Hibernate |
| xggorm | Go | GORM |
| xugu.go (XORM) | Go | XORM |
| xgcondb | Python | Django |
| xugu-sqlalchemy | Python | SQLAlchemy |
| xgpeewee | Python | peewee |
| Xugusql.php | PHP | ThinkPHP |
| xugu-dialect (Sequelize) | Node.js | Sequelize |
