# 空间函数完整参考

虚谷空间数据库提供 339 个 GEOMETRY 函数和 18 个 SFCGAL 函数，兼容 PostGIS 接口。

## 几何构造函数（15）

创建几何对象。

| 函数 | 说明 |
|------|------|
| ST_Point(x, y [, srid]) | 创建点 |
| ST_PointZ(x, y, z [, srid]) | 创建 3D 点 |
| ST_PointM(x, y, m [, srid]) | 创建带 M 值的点 |
| ST_PointZM(x, y, z, m [, srid]) | 创建 4D 点 |
| ST_MakeLine(geom1, geom2) | 从点创建线 |
| ST_MakePolygon(linestring) | 从闭合线创建面 |
| ST_MakeEnvelope(xmin, ymin, xmax, ymax [, srid]) | 创建矩形 |
| ST_MakePoint(x, y [, z [, m]]) | 创建点 |
| ST_MakePointM(x, y, m) | 创建带 M 的点 |
| ST_Collect(geom) / ST_Collect(geom[]) | 合并为几何集合 |
| ST_LineFromMultiPoint(multipoint) | 多点转线 |
| ST_Polygon(linestring, srid) | 线转面 |
| ST_Hexagon(size, cell_i, cell_j [, origin]) | 创建六边形 |
| ST_Square(size, cell_i, cell_j [, origin]) | 创建正方形 |
| ST_TileEnvelope(zoom, x, y) | 瓦片包络矩形 |

## 几何访问器（44）

获取几何对象的属性。

| 函数 | 说明 |
|------|------|
| ST_X(point) | 获取 X 坐标 |
| ST_Y(point) | 获取 Y 坐标 |
| ST_Z(point) | 获取 Z 坐标 |
| ST_M(point) | 获取 M 值 |
| ST_GeometryType(geom) | 几何类型名 |
| ST_SRID(geom) | 获取 SRID |
| ST_NDims(geom) | 坐标维度数 |
| ST_CoordDim(geom) | 坐标维度 |
| ST_Dimension(geom) | 几何维度（0/1/2） |
| ST_NPoints(geom) | 点数 |
| ST_NumPoints(linestring) | 线的点数 |
| ST_NumGeometries(collection) | 子几何数 |
| ST_GeometryN(collection, n) | 第 n 个子几何 |
| ST_PointN(linestring, n) | 第 n 个点 |
| ST_StartPoint(linestring) | 起点 |
| ST_EndPoint(linestring) | 终点 |
| ST_ExteriorRing(polygon) | 外环 |
| ST_InteriorRingN(polygon, n) | 第 n 个内环 |
| ST_NumInteriorRings(polygon) | 内环数 |
| ST_Envelope(geom) | 边界框（矩形） |
| ST_Boundary(geom) | 边界 |
| ST_Points(geom) | 所有点 |
| ST_IsEmpty(geom) | 是否为空 |
| ST_IsClosed(geom) | 是否闭合 |
| ST_IsRing(geom) | 是否为环 |
| ST_IsSimple(geom) | 是否简单 |
| ST_HasZ(geom) | 是否有 Z |
| ST_HasM(geom) | 是否有 M |
| ST_MemSize(geom) | 内存大小 |
| ST_Summary(geom) | 摘要信息 |
| ST_IsCollection(geom) | 是否为集合 |
| ST_IsPolygonCCW(geom) | 是否逆时针 |
| ST_IsPolygonCW(geom) | 是否顺时针 |

## 空间关系函数（21）

判断两个几何对象的空间关系，返回布尔值。

| 函数 | 说明 |
|------|------|
| ST_Intersects(A, B) | 相交（有公共部分） |
| ST_Contains(A, B) | A 包含 B |
| ST_ContainsProperly(A, B) | A 真包含 B |
| ST_Within(A, B) | A 在 B 内 |
| ST_Covers(A, B) | A 覆盖 B |
| ST_CoveredBy(A, B) | A 被 B 覆盖 |
| ST_Crosses(A, B) | 交叉 |
| ST_Overlaps(A, B) | 重叠 |
| ST_Touches(A, B) | 相切（边界接触） |
| ST_Disjoint(A, B) | 不相交 |
| ST_Equals(A, B) | 几何相等 |
| ST_OrderingEquals(A, B) | 完全相等（含方向） |
| ST_Relate(A, B, pattern) | DE-9IM 关系 |
| ST_RelateMatch(matrix, pattern) | 匹配 DE-9IM 模式 |
| ST_DWithin(A, B, distance) | 距离 <= d |
| ST_DFullyWithin(A, B, distance) | 完全在距离内 |
| ST_3DIntersects(A, B) | 3D 相交 |
| ST_3DDWithin(A, B, distance) | 3D 距离内 |
| ST_3DDFullyWithin(A, B, distance) | 3D 完全距离内 |
| ST_LineCrossingDirection(A, B) | 线交叉方向 |
| ST_PointInsideCircle(pt, cx, cy, r) | 点在圆内 |

## 测量函数（32）

计算距离、面积、长度等度量。

| 函数 | 说明 |
|------|------|
| ST_Distance(A, B) | 距离 |
| ST_DistanceSphere(A, B) | 球面距离（米） |
| ST_DistanceSpheroid(A, B, spheroid) | 椭球面距离 |
| ST_3DDistance(A, B) | 3D 距离 |
| ST_MaxDistance(A, B) | 最大距离 |
| ST_Area(geom) | 面积 |
| ST_Length(geom) | 长度 |
| ST_3DLength(geom) | 3D 长度 |
| ST_Perimeter(geom) | 周长 |
| ST_3DPerimeter(geom) | 3D 周长 |
| ST_Azimuth(A, B) | 方位角（弧度） |
| ST_Angle(A, B, C) | 角度 |
| ST_ClosestPoint(A, B) | A 上离 B 最近的点 |
| ST_ShortestLine(A, B) | 最短连线 |
| ST_LongestLine(A, B) | 最长连线 |
| ST_HausdorffDistance(A, B) | Hausdorff 距离 |
| ST_FrechetDistance(A, B) | Fréchet 距离 |
| ST_MinimumClearance(geom) | 最小间隙 |

## 几何处理函数（26）

| 函数 | 说明 |
|------|------|
| ST_Buffer(geom, distance) | 缓冲区 |
| ST_OffsetCurve(line, distance) | 偏移曲线 |
| ST_ConvexHull(geom) | 凸包 |
| ST_ConcaveHull(geom, target) | 凹包 |
| ST_Centroid(geom) | 质心 |
| ST_PointOnSurface(geom) | 面上的点 |
| ST_Simplify(geom, tolerance) | 简化（Douglas-Peucker） |
| ST_SimplifyPreserveTopology(geom, tolerance) | 保拓扑简化 |
| ST_SimplifyVW(geom, tolerance) | Visvalingam-Whyatt 简化 |
| ST_LineMerge(multiline) | 合并线 |
| ST_BuildArea(geom) | 构建面 |
| ST_Polygonize(geom) | 线转面 |
| ST_VoronoiPolygons(geom) | Voronoi 图 |
| ST_VoronoiLines(geom) | Voronoi 边 |
| ST_DelaunayTriangles(geom) | Delaunay 三角剖分 |
| ST_GeneratePoints(polygon, n) | 随机生成 n 个点 |
| ST_GeometricMedian(multipoint) | 几何中值 |
| ST_SharedPaths(A, B) | 共享路径 |
| ST_ChaikinSmoothing(geom) | Chaikin 平滑 |
| ST_ReducePrecision(geom, size) | 降低精度 |
| ST_FilterByM(geom, min, max) | 按 M 值过滤 |
| ST_MinimumBoundingCircle(geom) | 最小外接圆 |
| ST_OrientedEnvelope(geom) | 有向包络矩形 |
| ST_TriangulatePolygon(geom) | 三角化面 |
| ST_SimplifyPolygonHull(geom, ratio) | 简化面外壳 |

## 叠加函数（9）

| 函数 | 说明 |
|------|------|
| ST_Intersection(A, B) | 交集 |
| ST_Union(A, B) | 并集 |
| ST_UnaryUnion(geom) | 一元并集 |
| ST_MemUnion(geom) | 内存优化并集 |
| ST_Difference(A, B) | 差集（A - B） |
| ST_SymDifference(A, B) | 对称差 |
| ST_Split(geom, blade) | 切割 |
| ST_Node(geom) | 打断交叉线 |
| ST_ClipByBox2D(geom, box2d) | 裁剪 |

## 几何输入函数（34）

从各种格式创建几何对象。主要包括：
- WKT: ST_GeomFromText, ST_PointFromText, ST_LineFromText, ST_PolygonFromText 等
- WKB: ST_GeomFromWKB, ST_PointFromWKB, ST_LineFromWKB 等
- EWKT/EWKB: ST_GeomFromEWKT, ST_GeomFromEWKB
- GeoJSON: ST_GeomFromGeoJSON
- GML/KML: ST_GeomFromGML, ST_GeomFromKML
- GeoHash: ST_GeomFromGeoHash, ST_PointFromGeoHash, ST_Box2DFromGeoHash
- TWKB: ST_GeomFromTWKB
- MARC21: ST_GeomFromMARC21
- Encoded Polyline: ST_LineFromEncodedPolyline

## 几何输出函数（15）

| 函数 | 输出格式 |
|------|----------|
| ST_AsText(geom) | WKT |
| ST_AsEWKT(geom) | EWKT（含 SRID） |
| ST_AsBinary(geom) | WKB |
| ST_AsEWKB(geom) | EWKB |
| ST_AsHexEWKB(geom) | Hex EWKB |
| ST_AsGeoJSON(geom) | GeoJSON |
| ST_AsGML(geom) | GML |
| ST_AsKML(geom) | KML |
| ST_AsSVG(geom) | SVG |
| ST_AsX3D(geom) | X3D |
| ST_AsTWKB(geom) | TWKB |
| ST_GeoHash(geom) | GeoHash |
| ST_AsLatLonText(point) | 经纬度文本 |
| ST_AsMARC21(geom) | MARC21 |
| ST_AsEncodedPolyline(geom) | Encoded Polyline |

## 仿射变换函数（8）

| 函数 | 说明 |
|------|------|
| ST_Translate(geom, dx, dy [, dz]) | 平移 |
| ST_Scale(geom, sx, sy [, sz]) | 缩放 |
| ST_Rotate(geom, angle [, cx, cy]) | 旋转 |
| ST_RotateX(geom, angle) | 绕 X 轴旋转 |
| ST_RotateY(geom, angle) | 绕 Y 轴旋转 |
| ST_RotateZ(geom, angle) | 绕 Z 轴旋转 |
| ST_Affine(geom, ...) | 通用仿射变换 |
| ST_TransScale(geom, dx, dy, sx, sy) | 平移+缩放 |

## 线性参考函数（10）

| 函数 | 说明 |
|------|------|
| ST_LineInterpolatePoint(line, fraction) | 线上按比例取点 |
| ST_LineInterpolatePoints(line, fraction) | 线上按比例取多点 |
| ST_LineLocatePoint(line, point) | 点在线上的比例位置 |
| ST_LineSubstring(line, start, end) | 取线的子段 |
| ST_LocateAlong(geom, m) | 按 M 值定位 |
| ST_LocateBetween(geom, m1, m2) | M 值范围内几何 |
| ST_LocateBetweenElevations(geom, z1, z2) | Z 值范围内几何 |
| ST_InterpolatePoint(line, point) | 插值点的 M 值 |
| ST_AddMeasure(line, start, end) | 添加 M 值 |
| ST_3DLineInterpolatePoint(line, fraction) | 3D 线插值点 |

## 边界框函数（14）

| 函数 | 说明 |
|------|------|
| BOX2D(geom) | 获取 2D 边界框 |
| BOX3D(geom) | 获取 3D 边界框 |
| ST_Extent(geom) | 聚合 2D 边界框 |
| ST_3DExtent(geom) | 聚合 3D 边界框 |
| ST_MakeBox2D(pt_low, pt_high) | 创建 2D 框 |
| ST_3DMakeBox(pt_low, pt_high) | 创建 3D 框 |
| ST_Expand(geom, d) | 扩展边界框 |
| ST_XMin/ST_XMax(box) | X 范围 |
| ST_YMin/ST_YMax(box) | Y 范围 |
| ST_ZMin/ST_ZMax(box) | Z 范围 |

## SFCGAL 3D 函数（18）

基于 SFCGAL 库的 3D 空间分析函数。

### 处理和关系函数（9）

| 函数 | 说明 |
|------|------|
| ST_3DIntersection(A, B) | 3D 交集 |
| ST_3DDifference(A, B) | 3D 差集 |
| ST_3DUnion(A, B) | 3D 并集 |
| ST_MinkowskiSum(A, B) | Minkowski 和 |
| ST_StraightSkeleton(geom) | 直骨架 |
| ST_ApproximateMedialAxis(geom) | 近似中轴线 |
| ST_Extrude(geom, dx, dy, dz) | 拉伸 |
| ST_Tesselate(geom) | 镶嵌 |
| ST_ConstrainedDelaunayTriangles(geom) | 约束 Delaunay |

### 管理函数（2）

| 函数 | 说明 |
|------|------|
| postgis_sfcgal_version() | SFCGAL 版本 |
| postgis_sfcgal_full_version() | SFCGAL 详细版本 |

### 访问器和设置器（7）

| 函数 | 说明 |
|------|------|
| ST_3DArea(geom) | 3D 面积 |
| ST_Volume(geom) | 体积 |
| ST_IsPlanar(geom) | 是否平面 |
| ST_IsSolid(geom) | 是否实体 |
| ST_MakeSolid(geom) | 转为实体 |
| ST_Orientation(geom) | 朝向 |
| ST_ForceLHR(geom) | 强制左手规则 |
