[![Lang](https://img.shields.io/badge/Lang-English-2ea043?style=for-the-badge)](README.md)
[![Lang](https://img.shields.io/badge/Lang-%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87-6e7781?style=for-the-badge)](README.zh-CN.md)

# GIS — Geospatial Boundary Data

A working collection of **GeoJSON boundary and point datasets exported from ArcGIS** — historical
Chinese administrative divisions, present-day China, world country boundaries, city markers, a
pre-rendered tile set, and a few sample disaster photographs.

Everything is plain `.json` / `.png` / `.jpg`. There is no code to run and no build step: clone the
repo, point your map library at the files.

---

## Data inventory

| File | Size | Container | Features | Geometry | What it is |
| --- | ---: | --- | ---: | --- | --- |
| `Export_Output.json` | 5.3 MB | FeatureCollection | 32 | Polygon / MultiPolygon | Historical administrative **polygons** (Qing, 1820) |
| `Export_Output_2.json` | 1.1 MB | FeatureCollection | 1,859 | Point | Historical administrative **points** |
| `Export_Output_3.json` | 13.6 MB | FeatureCollection | 5,962 | LineString | **Rivers** (`RIVERS_ID`, `ORDER_`, `MOUTH`) |
| `Export_Output_4.json` | 2.6 MB | FeatureCollection | 353 | Polygon | **Lakes** (`LKS_ID`) |
| `Export_Output_5.json` | 10.2 MB | FeatureCollection | 320 | Polygon / MultiPolygon | Historical administrative polygons, wider extent |
| `Export_Output_6.json` | 181 KB | FeatureCollection | 301 | Point | Historical administrative points |
| `Export_Output_7.json` | 13 KB | FeatureCollection | 24 | Point | **1820 province seats** (直隶, 甘肃, …) |
| `Export_Output_8.json` | 5.7 MB | FeatureCollection | 8,659 | Point | Historical administrative points, densest set |
| `city.json` | 6 KB | FeatureCollection | 29 | Point | City markers with **bilingual** labels — *projected CRS, see below* |
| `ming_city.json` | 3 KB | FeatureCollection | 27 | Point | Ming-dynasty cities (`NAME` only) |
| `ming_mian.json` | 111 KB | **GeometryCollection** | 4 | Polygon / MultiPolygon | Ming-dynasty areas — **no attributes** |
| `world-administrative-boundaries.json` | 8.1 MB | **GeometryCollection** | 256 | Polygon / MultiPolygon | World country boundaries — **no attributes** |
| `中华人民共和国.json` | 568 KB | FeatureCollection | 35 | Polygon / MultiPolygon | Present-day China, province level (AMap/DataV schema) |
| `png/` | 3.3 MB | 84 PNG tiles | — | — | Pre-rendered base map, see [Tiles](#tiles) |
| `Picture/` | 45 KB | 4 JPEGs | — | — | Sample disaster photographs, see [Pictures](#pictures) |

---

## Attribute schema

Most of the `Export_Output*.json` files follow the **CHGIS** (China Historical GIS) field naming —
`GEOCOMP`, `CHECKER` and `SYS_ID` are the giveaway. The recurring fields:

| Field | Meaning |
| --- | --- |
| `NAME_CH` | Name in simplified Chinese (e.g. `直隶`) |
| `NAME_FT` | Name in traditional Chinese (e.g. `直隸`) |
| `NAME_PY` | Name in pinyin (e.g. `Zhili`) |
| `TYPE_CH` / `TYPE_PY` | Feature type (`省` / `Sheng`, `府` / `Fu`, …) |
| `LEV_RANK` | Administrative level, `1` = highest |
| `BEG_YR` / `END_YR` | Validity window, e.g. both `1820` for a single-year snapshot |
| `DYN_CH` / `DYN_PY` | Dynasty (`清朝` / `Qingchao`) |
| `LEV1_CH` / `LEV2_CH` | Parent level-1 / level-2 unit |
| `X_COORD` / `Y_COORD` | Representative point, WGS84 degrees |
| `PRES_LOC` | Present-day location the historical unit falls in |
| `AREA` | Area in km² (polygon files) |
| `GEOCOMP` / `CHECKER` | Compiler / checker of the record |
| `OBJ_TYPE` | `POINT` or `POLYGON` — the geometry the record came from |

`city.json` is different and simpler: `name`, `name_ENG`, `Province`, `Province_E`.

`中华人民共和国.json` uses the **AMap (DataV.GeoAtlas) schema**: `adcode`, `name`, `center`,
`centroid`, `childrenNum`, `level`, `parent.adcode`, `acroutes`.

`ming_city.json` has only `NAME`.

---

## Coordinate systems

> ⚠️ **`city.json` is the exception.** Its coordinates look like `[876377.5, 2979239.3]` — metres in
> a **projected** CRS, not degrees. Reproject to WGS84 (`EPSG:4326`) before putting it on a
> web map, otherwise the points land nowhere. Every other file in this repo is already in
> **WGS84 longitude / latitude**.

---

## Tiles

`png/` holds 84 tiles named `tile_{row}_{col}.png`, forming a **7 × 12** grid of a single
pre-rendered map. Load them as an image overlay / tile layer, or stitch them:

```bash
# stitch a 7×12 grid into one image (ImageMagick)
montage png/tile_{0..6}_{0..11}.png -tile 12x7 -geometry +0+0 basemap.png
```

## Pictures

`Picture/p1.jpg` … `p4.jpg` are small (≈270×180) sample photographs:

| File | Content |
| --- | --- |
| `p1.jpg` | Tornado over open ground |
| `p2.jpg` | Tornado damage to buildings |
| `p3.jpg` | Drought — cracked earth |
| `p4.jpg` | Earthquake / landslide damage to a road |

They are demo assets, not data.

---

## Usage

**GeoPandas**

```python
import geopandas as gpd

gdf = gpd.read_file("中华人民共和国.json")
print(gdf.crs)          # EPSG:4326 for everything except city.json
print(gdf.head())

# city.json needs an explicit source CRS before reprojecting
cities = gpd.read_file("city.json")
# cities = cities.set_crs(<source EPSG>).to_crs(4326)
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

## Caveats

- **`GeometryCollection` files carry no properties.** `ming_mian.json` and
  `world-administrative-boundaries.json` are bare geometry arrays — you cannot style or label them
  per feature without supplying your own attribute table.
- **Historical outputs are snapshots, not a time series.** `BEG_YR` / `END_YR` are usually identical
  (e.g. `1820`–`1820`); each file represents one reference year.
- **Coverage differs per file.** The point sets and the polygon sets were exported separately and do
  not necessarily contain the same features.
- **Attribution.** The CHGIS-style attributes indicate the data derives from the *China Historical
  GIS* project (Fudan University / Harvard Yenching). If you republish derived maps, credit the
  original source. This repository does not carry a licence file.

---

## Related

- [`Southwest-Region-Disaster-System-Front-End`](https://github.com/ChenSK23/Southwest-Region-Disaster-System-Front-End) —
  GeoJSON boundaries of Southwest China across 13 historical periods.
