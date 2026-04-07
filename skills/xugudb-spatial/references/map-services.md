# 地图服务集成

## GeoServer 集成

GeoServer 是开源地图服务器，支持发布 WMS、WFS、WMTS 等 OGC 标准服务。

### 连接虚谷数据库

1. **安装 GeoServer**
2. **安装虚谷 JDBC 驱动**：将 XuguDB JDBC 驱动 jar 放入 GeoServer 的 `lib` 目录
3. **配置数据存储**：
   - 新建"存储" → 选择"PostGIS"（虚谷兼容 PostGIS 接口）
   - 填写连接参数：
     - 主机：数据库服务器 IP
     - 端口：5138
     - 数据库：SYSTEM
     - 用户名：SYSDBA
     - 密码：SYSDBA

4. **发布图层**：选择空间表 → 配置样式 → 发布

### 支持的服务

| 服务 | 说明 |
|------|------|
| WMS | 地图图片服务 |
| WFS | 要素数据服务 |
| WMTS | 瓦片地图服务 |
| WCS | 覆盖范围服务 |

## SuperMap iServer 集成

SuperMap iServer 是超图的企业级 GIS 服务器。

### 连接配置

1. 安装 SuperMap iServer
2. 获取虚谷数据库驱动
3. 在 iServer 管理界面中添加数据源：
   - 数据源类型：数据库
   - 数据库类型：虚谷（或 PostGIS 兼容模式）
   - 连接参数：IP、端口（5138）、数据库名、用户名、密码

### 功能

- 空间数据发布与管理
- 地图制作与渲染
- 空间分析服务
- REST API 接口

## SuperMap iDesktopX 集成

SuperMap iDesktopX 是桌面端 GIS 应用。

### 连接虚谷数据库

1. 打开 iDesktopX
2. 新建/打开工作空间
3. 添加数据源 → 数据库类型选择虚谷
4. 填写连接参数

### 功能

- 空间数据浏览与编辑
- 地图制图
- 空间分析
- 数据导入/导出

## 注意事项

- 虚谷空间功能兼容 PostGIS 接口，部分 GIS 工具可使用 PostGIS 连接方式
- 确保空间表有正确的 SRID 设置
- 建议为空间列创建 GiST 索引以提高地图渲染性能
- 大数据量场景下建议使用 WMTS 瓦片服务而非实时渲染
