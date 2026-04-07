---
name: 虚谷数据库空间数据库
name_for_command: xugudb-spatial
description: |
  XuguDB 空间数据库（GIS）完整指南：几何模型（GEOMETRY/GEOGRAPHY/BOX2D/BOX3D）、
  空间参考系统（SRS/SRID）、空间索引（GiST/R-Tree）、300+ 空间函数
  （构造/访问/关系/测量/处理/叠加/输入输出/仿射变换等）、SFCGAL 3D 函数、
  地图服务集成（GeoServer/SuperMap）。适用于 GIS 应用开发和空间数据分析。
tags: xugudb, spatial, gis, geometry, geography, postgis, srid, index
---

# 虚谷数据库空间数据库

虚谷数据库空间功能兼容 PostGIS 接口，支持 OGC/SFA 标准，提供 300+ 空间函数用于几何对象的创建、查询、分析和处理。

## 几何模型

### GEOMETRY

```sql
-- GEOMETRY 类型（平面坐标系）
GEOMETRY[(type_modifier, SRID)]

-- 支持的几何子类型
-- Point, LineString, Polygon, MultiPoint, MultiLineString, MultiPolygon, GeometryCollection

-- 示例
CREATE TABLE geo_table (
    id INT PRIMARY KEY,
    geom GEOMETRY(POINT, 4326)  -- 指定为点类型，SRID=4326(WGS84)
);

INSERT INTO geo_table VALUES (1, ST_Point(116.4, 39.9, 4326));
```

### GEOGRAPHY

```sql
-- GEOGRAPHY 类型（地理坐标系，球面计算）
GEOGRAPHY[(type_modifier, SRID)]
-- 默认 SRID=4326 (WGS84)
-- 距离和面积计算基于椭球体，结果为米/平方米
```

### BOX2D / BOX3D

```sql
-- 二维边界框
SELECT 'BOX(1 2, 5 6)'::BOX2D;

-- 三维边界框
SELECT 'BOX3D(1 2 3, 4 5 6)'::BOX3D;
```

> 详细参考：[几何类型与空间参考](references/geometry-types.md)

## 空间索引

支持 GiST（R-Tree）索引加速空间查询：

```sql
-- 创建空间索引
CREATE INDEX idx_geom ON geo_table USING GIST (geom);

-- 构建选项
CREATE INDEX idx_geom ON geo_table USING GIST (geom)
WITH (MODE = 'auto');  -- auto/buffer/sort/basic
```

支持的空间索引操作符：`&&`（边界框相交）、`@`（包含于）、`~`（包含）等。

> 详细参考：[空间索引](references/spatial-indexes.md)

## 空间函数分类总览（339 + 18 个函数）

### 几何构造函数（15）

创建几何对象：

```sql
SELECT ST_Point(116.4, 39.9);                                    -- 点
SELECT ST_MakeLine(ST_Point(0,0), ST_Point(1,1));                -- 线
SELECT ST_MakePolygon('LINESTRING(0 0, 4 0, 4 4, 0 4, 0 0)');   -- 面
SELECT ST_MakeEnvelope(0, 0, 10, 10, 4326);                     -- 矩形
SELECT ST_Collect(geom) FROM geo_table;                          -- 集合
```

### 空间关系函数（21）

判断几何对象间的空间关系：

```sql
ST_Contains(A, B)    -- A 包含 B
ST_Within(A, B)      -- A 在 B 内
ST_Intersects(A, B)  -- A 与 B 相交
ST_Crosses(A, B)     -- A 与 B 交叉
ST_Overlaps(A, B)    -- A 与 B 重叠
ST_Touches(A, B)     -- A 与 B 相切
ST_Disjoint(A, B)    -- A 与 B 不相交
ST_DWithin(A, B, d)  -- A 与 B 距离 <= d
ST_Equals(A, B)      -- A 与 B 几何相等
```

### 测量函数（32）

```sql
ST_Distance(A, B)         -- 距离
ST_Area(geom)             -- 面积
ST_Length(geom)            -- 长度
ST_Perimeter(geom)         -- 周长
ST_Azimuth(A, B)           -- 方位角
ST_ClosestPoint(A, B)      -- 最近点
ST_HausdorffDistance(A, B) -- Hausdorff 距离
ST_DistanceSphere(A, B)    -- 球面距离（米）
```

### 几何处理函数（26）

```sql
ST_Buffer(geom, distance)     -- 缓冲区
ST_ConvexHull(geom)           -- 凸包
ST_ConcaveHull(geom, target)  -- 凹包
ST_Centroid(geom)             -- 质心
ST_Simplify(geom, tolerance)  -- 简化
ST_VoronoiPolygons(geom)      -- Voronoi 图
ST_DelaunayTriangles(geom)    -- Delaunay 三角剖分
```

### 叠加函数（9）

```sql
ST_Intersection(A, B)   -- 交集
ST_Union(A, B)          -- 并集
ST_Difference(A, B)     -- 差集
ST_SymDifference(A, B)  -- 对称差
ST_Split(A, blade)      -- 切割
```

### 几何输入（34）/ 输出（15）

```sql
-- 输入：从文本/二进制创建几何
ST_GeomFromText('POINT(1 2)', 4326)
ST_GeomFromGeoJSON('{"type":"Point","coordinates":[1,2]}')
ST_GeomFromEWKT('SRID=4326;POINT(1 2)')
ST_GeomFromWKB(wkb_data)
ST_GeomFromGML(gml_xml)
ST_GeomFromKML(kml_xml)

-- 输出：将几何转为文本/二进制
ST_AsText(geom)        -- WKT
ST_AsEWKT(geom)        -- EWKT (含SRID)
ST_AsGeoJSON(geom)     -- GeoJSON
ST_AsGML(geom)         -- GML
ST_AsKML(geom)         -- KML
ST_AsBinary(geom)      -- WKB
ST_AsSVG(geom)         -- SVG
ST_GeoHash(geom)       -- GeoHash
```

> 完整函数列表参考：[空间函数参考](references/spatial-functions.md)

## SFCGAL 3D 函数（18）

基于 SFCGAL 库的三维空间分析函数，包括：
- 3D 交集/差集/并集
- 3D 距离计算
- 凸包/Minkowski 和
- 体积/面积计算

> 详细参考：[空间函数参考](references/spatial-functions.md)

## 地图服务集成

- **GeoServer**：通过 JDBC 连接虚谷空间数据，发布 WMS/WFS/WMTS 服务
- **SuperMap iServer**：通过虚谷驱动连接空间数据
- **SuperMap iDesktopX**：桌面端 GIS 应用集成

> 详细参考：[地图服务集成](references/map-services.md)

## 工作流程

当用户咨询空间数据库相关问题时：

1. 确定问题类别（几何类型 / 空间查询 / 空间分析 / 索引优化 / 地图服务）
2. 提供基于 PostGIS 兼容语法的 SQL 示例
3. 对性能敏感的查询建议使用空间索引
4. 标注 SRID（空间参考系统）的重要性
5. 对 3D 分析需求推荐 SFCGAL 函数

## 参考文档

- [几何类型与空间参考](references/geometry-types.md) — GEOMETRY/GEOGRAPHY/BOX/SRS/SRID
- [空间索引](references/spatial-indexes.md) — GiST 索引创建、操作符、优化
- [空间函数参考](references/spatial-functions.md) — 339+18 个函数完整分类列表
- [地图服务集成](references/map-services.md) — GeoServer/SuperMap 集成配置
