# 空间索引

## 概述

虚谷空间数据库使用 GiST（Generalized Search Tree）索引，底层基于 R-Tree 结构，通过 MBR（最小边界矩形）加速空间查询。

## 支持的数据类型

- GEOMETRY
- GEOGRAPHY
- POINT
- BOX
- POLYGON
- CIRCLE

## 创建空间索引

### 基本语法

```sql
CREATE INDEX index_name ON table_name USING GIST (geometry_column);
```

### 带构建选项

```sql
CREATE INDEX index_name ON table_name USING GIST (geometry_column)
WITH (MODE = 'auto');
```

### 构建模式

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| auto | 自动选择（默认） | 通用 |
| buffer | 缓冲区构建 | 大数据量 |
| sort | 排序构建 | 有序数据 |
| basic | 基本构建 | 小数据量 |

### 示例

```sql
-- 创建空间表
CREATE TABLE poi (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    location GEOMETRY(POINT, 4326)
);

-- 插入数据
INSERT INTO poi VALUES (1, '北京', ST_Point(116.4, 39.9, 4326));
INSERT INTO poi VALUES (2, '上海', ST_Point(121.5, 31.2, 4326));

-- 创建空间索引
CREATE INDEX idx_poi_location ON poi USING GIST (location);
```

## 空间操作符（利用索引加速）

| 操作符 | 说明 | 示例 |
|--------|------|------|
| `&&` | 边界框相交 | `A && B` |
| `@` | A 包含于 B | `A @ B` |
| `~` | A 包含 B | `A ~ B` |
| `~=` | 几何相等 | `A ~= B` |
| `<->` | 距离（用于 ORDER BY） | `ORDER BY geom <-> point` |
| `<#>` | 边界框距离 | `A <#> B` |

### 查询示例

```sql
-- 边界框相交（使用索引）
SELECT * FROM poi WHERE location && ST_MakeEnvelope(116, 39, 117, 40, 4326);

-- 范围查询
SELECT * FROM poi WHERE ST_DWithin(location, ST_Point(116.4, 39.9, 4326), 0.1);

-- 最近邻查询（KNN，使用 <-> 操作符）
SELECT name, ST_Distance(location, ST_Point(116.4, 39.9, 4326)) AS dist
FROM poi
ORDER BY location <-> ST_Point(116.4, 39.9, 4326)
LIMIT 5;
```

## 索引优化建议

- 对所有参与空间查询（WHERE/JOIN）的几何列创建 GiST 索引
- 使用 `&&` 操作符做边界框预过滤，再用精确函数（ST_Contains 等）做精过滤
- 大表建索引时使用 `buffer` 模式提高效率
- 更新大量数据后考虑重建索引

## 删除空间索引

```sql
DROP INDEX index_name;
```
