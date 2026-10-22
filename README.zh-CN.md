[![Lang](https://img.shields.io/badge/Lang-English-6e7781?style=for-the-badge)](README.md)
[![Lang](https://img.shields.io/badge/Lang-%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87-2ea043?style=for-the-badge)](README.zh-CN.md)

# GIS — 地理边界数据

从 **ArcGIS 导出的 GeoJSON 边界与点位数据**合集：中国历史行政区划、当代中国的省界、
世界国界、城市点位、一套预渲染瓦片，以及几张示例灾害照片。

全部是纯 `.json` / `.png` / `.jpg`，没有代码、没有构建步骤——克隆下来直接把文件喂给地图库即可。

---

## 数据清单

| 文件 | 大小 | 容器类型 | 要素数 | 几何类型 | 说明 |
| --- | ---: | --- | ---: | --- | --- |
| `Export_Output.json` | 5.3 MB | FeatureCollection | 32 | Polygon / MultiPolygon | 历史行政区**面**（清，1820） |
| `Export_Output_2.json` | 1.1 MB | FeatureCollection | 1,859 | Point | 历史行政区**点** |
| `Export_Output_3.json` | 13.6 MB | FeatureCollection | 5,962 | LineString | **河流**（`RIVERS_ID`、`ORDER_`、`MOUTH`） |
| `Export_Output_4.json` | 2.6 MB | FeatureCollection | 353 | Polygon | **湖泊**（`LKS_ID`） |
| `Export_Output_5.json` | 10.2 MB | FeatureCollection | 320 | Polygon / MultiPolygon | 历史行政区面，范围更大 |
| `Export_Output_6.json` | 181 KB | FeatureCollection | 301 | Point | 历史行政区点 |
| `Export_Output_7.json` | 13 KB | FeatureCollection | 24 | Point | **1820 年各省治所**（直隶、甘肃……） |
| `Export_Output_8.json` | 5.7 MB | FeatureCollection | 8,659 | Point | 历史行政区点，最密集的一组 |
| `city.json` | 6 KB | FeatureCollection | 29 | Point | 中英双语城市点 —— *投影坐标系，见下文* |
| `ming_city.json` | 3 KB | FeatureCollection | 27 | Point | 明代城市（只有 `NAME`） |
| `ming_mian.json` | 111 KB | **GeometryCollection** | 4 | Polygon / MultiPolygon | 明代面数据——**无属性** |
| `world-administrative-boundaries.json` | 8.1 MB | **GeometryCollection** | 256 | Polygon / MultiPolygon | 世界国界——**无属性** |
| `中华人民共和国.json` | 568 KB | FeatureCollection | 35 | Polygon / MultiPolygon | 当代中国省级（高德 / DataV 格式） |
| `png/` | 3.3 MB | 84 张 PNG | — | — | 预渲染底图，见[瓦片](#瓦片) |
| `Picture/` | 45 KB | 4 张 JPEG | — | — | 示例灾害照片，见[图片](#图片) |

---

## 属性字段

多数 `Export_Output*.json` 用的是 **CHGIS**（中国历史地理信息系统）的字段命名——
`GEOCOMP`、`CHECKER`、`SYS_ID` 就是标志。常见字段：

| 字段 | 含义 |
| --- | --- |
| `NAME_CH` | 简体中文名（如 `直隶`） |
| `NAME_FT` | 繁体中文名（如 `直隸`） |
| `NAME_PY` | 拼音（如 `Zhili`） |
| `TYPE_CH` / `TYPE_PY` | 要素类型（`省` / `Sheng`、`府` / `Fu`……） |
| `LEV_RANK` | 行政层级，`1` 为最高 |
| `BEG_YR` / `END_YR` | 存续年份，单年快照时两者相同（如都是 `1820`） |
| `DYN_CH` / `DYN_PY` | 朝代（`清朝` / `Qingchao`） |
| `LEV1_CH` / `LEV2_CH` | 所属一级 / 二级政区 |
| `X_COORD` / `Y_COORD` | 代表点坐标，WGS84 经纬度 |
| `PRES_LOC` | 该历史政区对应的今地 |
| `AREA` | 面积（km²，仅面文件） |
| `GEOCOMP` / `CHECKER` | 数据编纂人 / 核查人 |
| `OBJ_TYPE` | `POINT` 或 `POLYGON`，标明该记录来自点集还是面集 |

`city.json` 的字段更简单：`name`、`name_ENG`、`Province`、`Province_E`。

`中华人民共和国.json` 用的是**高德（DataV.GeoAtlas）格式**：`adcode`、`name`、`center`、
`centroid`、`childrenNum`、`level`、`parent.adcode`、`acroutes`。

`ming_city.json` 只有一个 `NAME` 字段。

---

## 坐标系

> ⚠️ **`city.json` 是唯一的例外。** 它的坐标形如 `[876377.5, 2979239.3]`——这是**投影坐标系**
> 下的米，不是经纬度。直接用在地图上会跑到离谱的位置，必须先重投影到 WGS84（`EPSG:4326`）。
> 仓库里其他所有文件都已经是 **WGS84 经纬度**。

---

## 瓦片

`png/` 下是 84 张 `tile_{行}_{列}.png`，拼成一个 **7 行 × 12 列**的完整底图。
可以当图片叠加层 / 瓦片层加载，也可以直接拼起来：

```bash
# 把 7×12 的瓦片拼成一张图（需要 ImageMagick）
montage png/tile_{0..6}_{0..11}.png -tile 12x7 -geometry +0+0 basemap.png
```

## 图片

`Picture/p1.jpg` … `p4.jpg` 是 4 张约 270×180 的示例照片：

| 文件 | 内容 |
| --- | --- |
| `p1.jpg` | 旷野上的龙卷风 |
| `p2.jpg` | 龙卷风过后的建筑废墟 |
| `p3.jpg` | 干旱——龟裂的土地 |
| `p4.jpg` | 地震 / 滑坡造成的公路坍塌 |

它们是演示素材，不是数据。

---

## 用法示例

**GeoPandas**

```python
import geopandas as gpd

gdf = gpd.read_file("中华人民共和国.json")
print(gdf.crs)          # 除 city.json 外都是 EPSG:4326
print(gdf.head())

# city.json 需要先指定源坐标系
cities = gpd.read_file("city.json")
# cities = cities.set_crs(<源 EPSG>).to_crs(4326)
```

**Leaflet**

```js
const map = L.map('map').setView([30, 105], 5);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

fetch('中华人民共和国.json')
  .then(r => r.json())
  .then(data => L.geoJSON(data, { style: { color: '#3388ff', weight: 1 } }).addTo(map));
```

**ECharts**

```js
const geoJson = await (await fetch('中华人民共和国.json')).json();
echarts.registerMap('china', geoJson);
option = { series: [{ type: 'map', map: 'china' }] };
```

---

## 注意事项

- **`GeometryCollection` 文件没有属性。** `ming_mian.json` 和
  `world-administrative-boundaries.json` 只有裸几何数组，无法按要素配色或标注，
  需要自备属性表。
- **历史数据是"快照"，不是时间序列。** `BEG_YR` / `END_YR` 通常相同（如 `1820`–`1820`），
  每个文件代表一个基准年。
- **各文件覆盖范围不一致。** 点集和面集是分别导出的，未必包含同一批要素。
- **数据来源。** CHGIS 风格的字段说明数据源自 *中国历史地理信息系统*（CHGIS，复旦大学 /
  哈佛燕京学社）。再发布派生地图时请注明原始来源。本仓库未附带 LICENSE 文件。

---

## 相关仓库

- [`Southwest-Region-Disaster-System-Front-End`](https://github.com/ChenSK23/Southwest-Region-Disaster-System-Front-End) —
  西南六省 13 个历史时期的 GeoJSON 边界数据。
