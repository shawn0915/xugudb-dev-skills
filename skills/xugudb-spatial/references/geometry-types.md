# 几何类型与空间参考系统

## GEOMETRY 类型

GEOMETRY 是虚谷空间数据库的核心数据类型，遵循 OGC SFA 标准。

### 定义

```sql
GEOMETRY[(type_modifier, SRID)]
```

- `type_modifier`：几何子类型约束（可选）
- `SRID`：空间参考系统标识（默认 0）

### 几何子类型

| 类型 | WKT 示例 | 维度 |
|------|----------|------|
| POINT | `POINT(1 2)` | 0D |
| LINESTRING | `LINESTRING(0 0, 1 1, 2 2)` | 1D |
| POLYGON | `POLYGON((0 0, 4 0, 4 4, 0 4, 0 0))` | 2D |
| MULTIPOINT | `MULTIPOINT((0 0), (1 1))` | 0D |
| MULTILINESTRING | `MULTILINESTRING((0 0, 1 1), (2 2, 3 3))` | 1D |
| MULTIPOLYGON | `MULTIPOLYGON(((0 0, 4 0, 4 4, 0 0)))` | 2D |
| GEOMETRYCOLLECTION | `GEOMETRYCOLLECTION(POINT(1 1), LINESTRING(0 0, 1 1))` | 混合 |

### 坐标维度

| 标记 | 坐标 | WKT 示例 |
|------|------|----------|
| 2D | X, Y | `POINT(1 2)` |
| Z | X, Y, Z | `POINT Z(1 2 3)` |
| M | X, Y, M | `POINT M(1 2 3)` |
| ZM | X, Y, Z, M | `POINT ZM(1 2 3 4)` |

### 建表示例

```sql
-- 不限制类型
CREATE TABLE places (id INT, geom GEOMETRY);

-- 限制为点类型，WGS84 坐标系
CREATE TABLE cities (id INT, location GEOMETRY(POINT, 4326));

-- 限制为面类型
CREATE TABLE districts (id INT, boundary GEOMETRY(POLYGON, 4326));
```

## GEOGRAPHY 类型

GEOGRAPHY 类型用于地理坐标系（经纬度），计算基于椭球面。

```sql
GEOGRAPHY[(type_modifier, SRID)]
```

- 默认 SRID = 4326（WGS84）
- 距离单位为米，面积单位为平方米
- 自动处理跨日期线等边界情况

### GEOMETRY vs GEOGRAPHY

| 特性 | GEOMETRY | GEOGRAPHY |
|------|----------|-----------|
| 坐标系 | 平面/投影 | 地理（经纬度） |
| 默认 SRID | 0 | 4326 |
| 距离单位 | 坐标单位 | 米 |
| 计算精度 | 平面近似 | 椭球面精确 |
| 函数支持 | 全部 | 部分 |
| 适用场景 | 工程/CAD | 地图/导航 |

## BOX2D / BOX3D

```sql
-- BOX2D: 二维边界框
SELECT 'BOX(xmin ymin, xmax ymax)'::BOX2D;
SELECT 'BOX(1 2, 5 6)'::BOX2D;

-- BOX3D: 三维边界框
SELECT 'BOX3D(xmin ymin zmin, xmax ymax zmax)'::BOX3D;
SELECT 'BOX3D(1 2 3, 4 5 6)'::BOX3D;

-- 计算几何对象的边界框
SELECT ST_Extent(geom) FROM geo_table;      -- 返回 BOX2D
SELECT ST_3DExtent(geom) FROM geo_table;    -- 返回 BOX3D
```

## 空间参考系统（SRS）

### SRID（空间参考标识符）

| SRID | 名称 | 说明 |
|------|------|------|
| 0 | 未定义 | 默认，无坐标系 |
| 4326 | WGS84 | GPS 经纬度，最常用 |
| 3857 | Web Mercator | Web 地图投影 |
| 4490 | CGCS2000 | 中国大地坐标系 |

### 设置/查询 SRID

```sql
-- 设置 SRID
SELECT ST_SetSRID(ST_Point(116.4, 39.9), 4326);

-- 查询 SRID
SELECT ST_SRID(geom) FROM geo_table;

-- 坐标转换
SELECT ST_Transform(geom, 3857) FROM geo_table WHERE ST_SRID(geom) = 4326;
```

### EWKT 格式（含 SRID）

```sql
-- EWKT = SRID + WKT
SELECT ST_GeomFromEWKT('SRID=4326;POINT(116.4 39.9)');
```

## 元数据视图

### GEOMETRY_COLUMNS

记录所有 GEOMETRY 类型列的元信息：

| 字段 | 说明 |
|------|------|
| F_TABLE_CATALOG | 数据库名 |
| F_TABLE_SCHEMA | Schema 名 |
| F_TABLE_NAME | 表名 |
| F_GEOMETRY_COLUMN | 列名 |
| COORD_DIMENSION | 坐标维度（2/3/4） |
| SRID | 空间参考 ID |
| TYPE | 几何类型 |

### GEOGRAPHY_COLUMNS

类似结构，记录 GEOGRAPHY 类型列的元信息。

### SPATIAL_REF_SYS

空间参考系统定义表，可查询支持的 SRID。

## 几何有效性

```sql
-- 检查几何有效性
SELECT ST_IsValid(geom);

-- 获取无效原因
SELECT ST_IsValidReason(geom);

-- 获取详细信息
SELECT ST_IsValidDetail(geom);

-- 修复无效几何
SELECT ST_MakeValid(geom);
```

### 有效性规则

- POLYGON 必须闭合（首尾点相同）
- POLYGON 边界不能自交
- LINESTRING 至少 2 个不同点
- 内环必须在外环内部
