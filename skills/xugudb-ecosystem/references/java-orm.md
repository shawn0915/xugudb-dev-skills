# Java ORM 框架集成

## 通用配置

所有 Java 框架连接虚谷数据库的核心参数：

| 参数 | 值 |
|------|-----|
| 驱动类名 | `com.xugu.cloudjdbc.Driver` |
| JDBC URL | `jdbc:xugu://host:port/database` |
| 默认端口 | 5138 |
| Maven GroupId | `com.xugudb` |
| Maven ArtifactId | `xugu-jdbc` |

## MyBatis

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
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.20</version>
</dependency>
```

### mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="local-xugu.properties"/>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    <typeAliases>
        <typeAlias type="com.xugu.datasource.DruidDataSourceFactory" alias="druid"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="druid">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
                <property name="initialSize" value="1"/>
                <property name="minIdle" value="1"/>
                <property name="maxActive" value="10"/>
                <property name="maxWait" value="10000"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="mybatis/mapper/StudentMapper.xml"/>
    </mappers>
</configuration>
```

### 数据源属性文件（local-xugu.properties）

```properties
driver=com.xugu.cloudjdbc.Driver
url=jdbc:xugu://127.0.0.1:5138/SYSTEM
username=SYSDBA
password=SYSDBA
```

## Hibernate

### 适配要求

- Hibernate 6.6.1+
- 需安装 `xugu-dialect` 到 Hibernate 的 dialects 目录

### hibernate.cfg.xml

```xml
<hibernate-configuration>
    <session-factory>
        <property name="connection.driver_class">com.xugu.cloudjdbc.Driver</property>
        <property name="connection.url">jdbc:xugu://127.0.0.1:5138/SYSTEM</property>
        <property name="connection.username">SYSDBA</property>
        <property name="connection.password">SYSDBA</property>
        <property name="dialect">org.hibernate.dialect.XuguDialect</property>
        <property name="show_sql">true</property>
    </session-factory>
</hibernate-configuration>
```

## Spring Boot

### application.yml（HikariCP 连接池）

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
      auto-commit: true
```

### application.yml（Druid 连接池）

```yaml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.xugu.cloudjdbc.Driver
    url: jdbc:xugu://127.0.0.1:5138/SYSTEM
    username: SYSDBA
    password: SYSDBA
    druid:
      initial-size: 5
      min-idle: 5
      max-active: 20
      max-wait: 60000
```

### pom.xml 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>com.xugudb</groupId>
    <artifactId>xugu-jdbc</artifactId>
    <version>12.3.4</version>
</dependency>
```

## 连接池对比

| 连接池 | 推荐版本 | 特点 |
|--------|----------|------|
| HikariCP | 4.0.3 | Spring Boot 默认，性能最优 |
| Druid | 1.1.24 | 阿里巴巴出品，SQL 监控功能强 |
| c3p0 | - | 传统连接池，配置简单 |

## 注意事项

- XuguDB-JDBC 驱动需与 ORM 框架版本兼容
- Hibernate 需要额外安装 xugu-dialect
- Spring Boot 默认使用 HikariCP，需在 YAML 中配置虚谷连接参数
- Druid 的 WallFilter（SQL 防火墙）可能需要针对虚谷语法调整配置
- 所有框架的 `connection-test-query` 建议使用 `SELECT 1`
