---
name: 虚谷数据库生态集成
name_for_command: xugudb-ecosystem
description: |
  XuguDB 生态集成指南：ORM 框架适配（Java/Go/Python/PHP/Node.js）。
  涵盖 MyBatis、Hibernate、Spring Boot、GORM、Django、SQLAlchemy、Sequelize、ThinkPHP 等
  主流框架与虚谷数据库的集成配置。适用于使用各语言 ORM 框架开发虚谷数据库应用。
tags: xugudb, orm, mybatis, hibernate, spring, gorm, django, sqlalchemy, ecosystem
---

# 虚谷数据库生态集成

## 支持的框架总览

### Java 生态

| 框架 | 版本 | 适配方式 | 说明 |
|------|------|----------|------|
| **MyBatis** | 3.5.10 | JDBC 直连 | 通过 XuguDB-JDBC + mybatis-config.xml |
| **MyBatis-Plus** | - | JDBC 直连 | MyBatis 增强版 |
| **Hibernate** | 6.6.1 | xugu-dialect | 需安装虚谷 Dialect 到 Hibernate |
| **Spring Boot** | 3.3.2 | JDBC + ORM | 支持 JPA/MyBatis/JdbcTemplate |
| **Druid** | 1.1.24 | 连接池 | 阿里巴巴数据库连接池 |
| **HikariCP** | 4.0.3 | 连接池 | Spring Boot 默认连接池 |
| **c3p0** | - | 连接池 | 传统连接池 |
| **ShardingSphere** | - | 分库分表 | 分布式数据库中间件 |
| **Flyway/Liquibase** | - | 数据库迁移 | Schema 版本管理 |
| **Quartz/XXL-JOB/PowerJob** | - | 任务调度 | 定时任务框架 |
| **Nacos** | - | 配置中心 | 服务配置管理 |
| **Activiti/Flowable/Camunda** | - | 工作流 | BPM 流程引擎 |

### Go 生态

| 框架 | 版本 | 适配方式 |
|------|------|----------|
| **GORM** | 1.20.0 | xggorm 适配器 |
| **XORM** | 1.3.1 | xugu.go dialect |

### Python 生态

| 框架 | 版本 | 适配方式 |
|------|------|----------|
| **Django** | 4.2.1 | xgcondb 引擎 |
| **SQLAlchemy** | 1.4.36 | xugu-sqlalchemy 方言 |
| **peewee** | 3.17.1 | xgpeewee 适配器 |

### PHP 生态

| 框架 | 版本 | 适配方式 |
|------|------|----------|
| **ThinkPHP** | 5.0.1 | Xugusql builder |

### Node.js 生态

| 框架 | 版本 | 适配方式 |
|------|------|----------|
| **Sequelize** | 6.37.3 | xugu-dialect |

> 详细参考：[Java ORM 集成](references/java-orm.md) / [其他语言 ORM 集成](references/other-orm.md)

## Java — MyBatis 集成

### Maven 依赖

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.10</version>
</dependency>
<dependency>
    <groupId>com.xugudb</groupId>
    <artifactId>xugu-jdbc</artifactId>
    <version>12.3.4</version>
</dependency>
```

### mybatis-config.xml 关键配置

```xml
<dataSource type="druid">
    <property name="driver" value="com.xugu.cloudjdbc.Driver"/>
    <property name="url" value="jdbc:xugu://127.0.0.1:5138/SYSTEM"/>
    <property name="username" value="SYSDBA"/>
    <property name="password" value="SYSDBA"/>
</dataSource>
```

## Java — Spring Boot + HikariCP

### application.yml

```yaml
spring:
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: com.xugu.cloudjdbc.Driver
    url: jdbc:xugu://127.0.0.1:5138/SYSTEM
    username: SYSDBA
    password: SYSDBA
    hikari:
      pool-name: MyHikariPool
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000
      max-lifetime: 1800000
      idle-timeout: 600000
      connection-test-query: SELECT 1
```

## Go — GORM 集成

```go
import (
    "xggorm"
    "gorm.io/gorm"
    _ "gitee.com/XuguDB/go-xugu-driver"
)

dsn := "IP=127.0.0.1;DB=SYSTEM;User=SYSDBA;PWD=SYSDBA;Port=5138;CHAR_SET=UTF8"
db, err := gorm.Open(xggorm.Open(dsn), &gorm.Config{})
```

## Python — Django 集成

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

## Python — SQLAlchemy 集成

```python
from sqlalchemy import create_engine
engine = create_engine('xugu://SYSDBA:SYSDBA@127.0.0.1:5138/SYSTEM')
```

## 工作流程

当用户咨询生态集成问题时：

1. 确定使用的编程语言和框架
2. 确认框架版本与虚谷适配器版本的兼容性
3. 提供数据源配置（连接字符串、驱动类名）
4. 标注需要额外安装的适配插件（dialect/adapter）
5. 对虚谷特有语法建议使用原生 SQL

## 参考文档

- [Java ORM 集成](references/java-orm.md) — MyBatis/Hibernate/Spring Boot/连接池配置
- [其他语言 ORM 集成](references/other-orm.md) — Go GORM/XORM、Python Django/SQLAlchemy、PHP ThinkPHP、Node.js Sequelize
