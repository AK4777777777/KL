# 夏威夷大岛垃圾填埋场选址优化

本项目旨在使用NSGA-II算法，通过多标准决策分析，为夏威夷大岛识别出新垃圾填埋场的最佳位置。

项目方法论改编自 [NSGA2-GIS-Landfill-Optimization-Cape-Verde](https://github.com/Jazancort/NSGA2-GIS-Landfill-Optimization-Cape-Verde) 项目。

## 数据获取与预处理

所有空间数据均使用QGIS进行处理。所有图层的最终坐标参照系统 (CRS) 均为 **NAD83 / UTM zone 4N (EPSG:26904)** 或其兼容系统，并在必要时应用了坐标变换。

为本次分析准备了以下图层：

### 1. 研究区边界与坡度

* **边界 (`Big_Island_Boundary.shp`)**:
    * **来源**：用户提供的夏威夷大岛海岸线Shapefile文件。
    * **预处理**：该图层被用作所有其他数据集的主要裁剪边界，以确保研究区域保持一致。

* **坡度 (`Big_Island_Slope.tif`)**:
    * **来源**：用户提供的夏威夷大岛数字高程模型 (DEM)。
    * **预处理**：使用QGIS内置的坡度分析工具，基于DEM计算得出坡度（单位：度）。该栅格图层是优化模型的直接输入项。

### 2. 保护区 (`Protected_Areas_Final.shp`)

由于单一数据源信息不完整，通过合并联邦与州两个级别的数据源，创建了一个全面的保护区图层。

* **来源 1 (联邦级)**：美国国家公园
    * **网站**：[Protected Planet](https://www.protectedplanet.net/) (世界保护区数据库 - WDPA)
    * **数据集**：“夏威夷火山国家公园” (`Hawaii Volcanoes National Park`) Shapefile。

* **来源 2 (州级)**：州立保护地
    * **网站**：[夏威夷州GIS项目官网](https://planning.hawaii.gov/gis/download-gis-data/)
    * **数据集**：覆盖全州的 `Conservation District Subzones` Shapefile。

* **预处理步骤**：
    1.  **裁剪州级数据**：使用 `Big_Island_Boundary.shp` 对全州范围的 `Conservation District Subzones` 图层进行裁剪，以仅提取位于大岛上的保护区。
    2.  **合并图层**：在QGIS中使用“合并矢量图层”工具，将裁剪后的州级保护区图层与“夏威夷火山国家公园”图层进行合并。
    3.  **最终输出**：将生成的文件保存为 `Protected_Areas_Final.shp`。

### 3. 地质与渗透性 (`Big_Island_Geology_Permeability.shp`)

获取并处理地质图，旨在将岛屿地表划分为不同渗透性的区域，这是防止地下水污染的关键因素。

* **来源**：美国地质调查局 (USGS)
    * **网站**：[夏威夷州地质图 (USGS Open-File Report 2007-1089)](https://pubs.usgs.gov/of/2007/1089/)
    * **数据集**：下载 `Haw_St_shapefiles.zip` 数据包，并从中识别出 `Haw_St_geo_..._region.shp` 作为主要地质图层。

* **预处理步骤**：
    1.  **裁剪州级数据**：使用 `Big_Island_Boundary.shp` 对全州地质图层进行裁剪。
    2.  **添加渗透性字段**：在裁剪后的地质图层属性表中，添加一个名为 `Permeability` 的新文本字段。
    3.  **数据重分类**：使用“字段计算器”，根据现有的 `ROCK_TYPE` 字段中的描述信息，为 `Permeability` 字段赋值。应用的逻辑如下：
        * `ROCK_TYPE` 描述为 `'Lava flows'` (熔岩流) 的区域赋值为 `'Low'` (低)。
        * `ROCK_TYPE` 描述为 `'Ash'` (火山灰) 或 `'cinder'` (火山渣) 的区域赋值为 `'High'` (高)。
        * 所有其他岩石类型被保守地赋值为 `'Medium'` (中)。
    4.  **最终输出**：将处理后的文件保存为 `Big_Island_Geology_Permeability.shp`。