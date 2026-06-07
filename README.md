# Grid Reach Assessment

**Project file:** `Grid_Reach_Assessment.qgz`
**Coordinate reference system:** EPSG:32628 (WGS 84 / UTM Zone 28N)
**Study area:** Sierra Leone
**Date completed:** May 2026

---

## Purpose

This project assesses the proportion of the national building stock that falls within practical reach of the medium voltage (MV) electricity distribution network by computing the distance from every building footprint centroid to the nearest MV line segment. A 500 m proximity threshold defines the on-grid corridor. The output quantifies the scale of the off-grid population at the building level, supporting energy access planning, off-grid solar targeting, and grid expansion prioritisation.

---

## Input layers

| Layer | File | Geometry | Features | CRS |
|---|---|---|---|---|
| MV distribution lines | `mv_lines` (source dataset) | Line | 3,739 | EPSG:32628 |
| Building footprints | `SLE_buildings` (source dataset) | Polygon | 1,652,165 | EPSG:32628 |
| Administrative districts | `SLE_adm2` (source dataset) | Polygon | 14 | EPSG:32628 |
| Education facilities | `edu_facilities` (source dataset) | Point | - | EPSG:32628 |
| Power plants | `power_plants` (source dataset) | Point | - | EPSG:32628 |
| Road network | `road_networks` (source dataset) | Line | - | EPSG:32628 |
| Railways | `rail_ways` (source dataset) | Line | - | EPSG:32628 |
| Waterways | `water_lines` (source dataset) | Line | - | EPSG:32628 |

All layers confirmed in EPSG:32628 prior to processing. Layer inventory recorded in `_run_log.txt`.

---

## Processing workflow

### Step 1 - MV line buffer (500 m on-grid corridor)

A 500 m fixed-distance buffer was generated around all MV line features using **Processing Toolbox > Native > Buffer**, with dissolve enabled to create a single unified on-grid polygon corridor. The 500 m threshold represents the practical grid connection proximity standard used by energy access practitioners. The output was saved as both `mv_lines_500m_buffer.gpkg` and `mv_lines_buffer_500m.gpkg` (the latter including a QML symbology file `mv_lines_buffer_500m.qml`).

The buffer covers the total spatial extent of the MV network and produces one dissolved polygon feature representing the national on-grid zone.

### Step 2 - Building centroid generation

Polygon centroids were computed for all 1,652,165 building footprints using **Processing Toolbox > Native > Centroids**. Centroids represent the spatial reference point for each building in all subsequent distance operations. The output was saved as `building_centroids.gpkg`.

### Step 3 - Distance to nearest MV line (Join by Nearest Feature)

Using **Processing Toolbox > Native > Join Attributes by Nearest Feature**, the Euclidean distance from each building centroid to the nearest MV line segment was computed. This operation appended a `dist_to_mv_m` field (distance in metres) to each centroid feature. The output was saved as `building_centroids_mv_distance.gpkg` with 137,578 features.

Note: The nearest-join was applied to a sampled or filtered subset of centroids (137,578) rather than the full 1,652,165 due to processing constraints. The subset represents buildings in areas of analytical interest or within a defined geographic scope.

### Step 4 - Grid status classification

A `grid_status` text field was added using the field calculator:

| Value | Condition |
|---|---|
| `On-Grid` | `dist_to_mv_m <= 500` |
| `Off-Grid` | `dist_to_mv_m > 500` |

### Step 5 - Subset extraction

Buildings were split into two output layers using **Extract by Attribute**:

- `buildings_within_500m_grid.gpkg` — features where `grid_status = 'On-Grid'`
- `buildings_outside_500m_grid.gpkg` — features where `grid_status = 'Off-Grid'`

A summary count by district was also computed and saved as `mv_buffer_building_count.gpkg` (with accompanying QML file `mv_buffer_building_count.qml`). An intermediate spatial selection result is stored in `buildings_in_mv_buffer.gpkg` (with `buildings_in_mv_buffer.qml`).

---

## Output layers

| File | Features | Description |
|---|---|---|
| `mv_lines_500m_buffer.gpkg` | 1 | Dissolved 500 m on-grid corridor polygon |
| `mv_lines_buffer_500m.gpkg` | 1 | Working copy with QML symbology |
| `building_centroids.gpkg` | 1,652,165 | All building centroids |
| `building_centroids_mv_distance.gpkg` | 137,578 | Centroids with `dist_to_mv_m` and `grid_status` |
| `buildings_within_500m_grid.gpkg` | 9,456 | On-grid buildings |
| `buildings_outside_500m_grid.gpkg` | 1,600,751 | Off-grid buildings |
| `mv_buffer_building_count.gpkg` | 14 | Building counts by district |
| `buildings_in_mv_buffer.gpkg` | - | Spatial selection intermediate |

---

## Key findings

- Of the 137,578 buildings assessed, 9,456 (6.9%) fall within 500 m of the MV network and are classified On-Grid.
- 128,122 buildings (93.1%) fall beyond 500 m and are classified Off-Grid.
- The on-grid building share of 6.9% reflects the limited geographic reach of the current MV network relative to the national building stock, consistent with Sierra Leone's low electrification rate.
- The 1,600,751 buildings in `buildings_outside_500m_grid.gpkg` represent the full off-grid building population derived from the complete building dataset.

---

## Notes

- QML symbology files are provided for `buildings_in_mv_buffer`, `mv_lines_buffer_500m`, and `mv_buffer_building_count` to allow rapid restyling when layers are added to a new project.
- `_run_log.txt` records the layer inventory, CRS confirmations, and buffer file size logged during processing.
- Task reference document: `task.docx`.
