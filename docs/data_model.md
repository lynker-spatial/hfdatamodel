# Lynker Hydrofabric Data Model

> **Scope:** Defines the schema and relationships for the Hydrofabric network — integrating flowpaths, catchments, points of interest (POIs), and external hydrolocation sources into a consistent, query-friendly model.

## Overview
This model provides the canonical representation of a routed river network with associated catchments and reference points.  
It supports:
- **Hydrologic analysis** (flowpath connectivity, drainage accumulation).
- **Catchment delineation** (divides and incremental areas).
- **External integration** (gauges, reservoirs, and other hydrolocations).
- **Optimized queries** through the denormalized `Network` table.

---

## Table Descriptions

**Flowpaths** — The backbone of the model. Each record is a routed stream segment with downstream connectivity (`flowpath_toid`). Flowpaths store physical metrics (length, drainage area, hydrosequence), belong to processing tiles (`vpuid`), and can be grouped into mainstems. They are the anchor for Divides, POIs, and Hydrolocations.

**Divides** — Polygonal catchments linked to flowpaths. Divides provide area context (`areasqkm`) and type categories (`network`, `coastal`, `internal`). In most datasets, Divides map one-to-one with Flowpaths, though generalization can produce one-to-many.

**POIs** — Logical anchor points (e.g., stream gages, reservoirs, confluences). POIs snap to Flowpaths for routing and serve as aggregation nodes for Hydrolocations. Rollup fields summarize hydrolocation classes, types, and counts.

**Hydrolocations** — Individual features from external systems (NWIS, HUC12, NBI, HiLARRI, etc.). Hydrolocations snap to Flowpaths and group under POIs. They carry both **source metadata** (`hl_type`, `hl_link`) and **semantic class** (`hl_class`, e.g., gage, reservoir), with unique crosswalk identifiers (`hl_reference`).

**Network** — A denormalized, read-optimized view combining Flowpath, Divide, POI, Hydrolocation, and Hydrofabric crosswalks. It mirrors topology (`flowpath_id`, `flowpath_toid`, `hydroseq`) and exposes convenience fields (`hf_*`, `flowline_*`) for faster queries without multiple joins.

**Flowlines** — Fine-scale routing primitives underlying Flowpaths. Each Flowline has its own downstream pointer, hydrosequence, and incremental catchment. They roll up into Flowpaths but preserve detailed granularity for advanced analyses.

**Incremental Area** — Polygonal catchments representing direct contributing areas to Flowlines. These support area partitioning and proportional allocation across flowline networks.

---

## Entities & Attributes

### Flowpaths
| Column           | Type   | Description                                                                 |
|------------------|--------|-----------------------------------------------------------------------------|
| `flowpath_id`    | int / text    | Primary key for the flowpath.                                               |
| `flowpath_toid`  | int / text    | Downstream `flowpath_id` (self-reference); `NULL` for terminal reaches.     |
| `vpuid`         | text    | Vector Processing Unit identifier (processing/tiling key).                  |
| `poi_id`         | int / text    | Optional anchoring POI on this flowpath.                                    |
| `mainstem_id`    | int    | Identifier grouping flowpaths in the same mainstem.                         |
| `areasqkm`       | float  | Area of the associated divide/catchment (sq km).                            |
| `lengthkm`       | float  | Flowpath length (km).                                                       |
| `total_dasqkm`   | float  | Total contributing drainage area at outlet (sq km).                         |
| `hydroseq`       | int    | Hydrosequence ordering (higher = upstream).                                 |
| `streamorder`    | int    | Strahler Stream Order                                                       |
| `slope`          | float  | Slope of flow segment (m/m)                                                 |

---

### Divides
| Column        | Type   | Description                                                                 |
|---------------|--------|-----------------------------------------------------------------------------|
| `divide_id`   | int / text    | Primary key for the divide polygon.                                         |
| `flowpath_id` | int / text    | Associated `flowpath_id` (one-to-one, or one-to-many in generalized sets). |
| `vpuid`       | text     | Vector Processing Unit identifier.                                          |
| `areasqkm`    | float  | Catchment area (sq km).                                                     |
| `type`        | text   | Divide type (e.g., `network`, `coastal`, `internal`).                       |

---

### POIs
| Column         | Type | Description                                                                 |
|----------------|------|-----------------------------------------------------------------------------|
| `poi_id`       | int / text  | Primary key for a Point of Interest.                                        |
| `flowpath_id`  | int / text  | Flowpath this POI anchors to for routing.                                   |
| `vpuid`         | text   | Vector Processing Unit identifier.                                                               |
| `hl_classes`   | text | Comma-separated list of hydrolocation classes (e.g., `gage,reservoir`).     |
| `hl_types`     | text | Comma-separated list of hydrolocation source types (e.g., `nwis,huc12`).    |
| `hl_count`     | int  | Number of hydrolocations aggregated into this POI.                         |

---

### Hydrolocations
| Column          | Type  | Description                                                                                      |
|-----------------|-------|--------------------------------------------------------------------------------------------------|
| `poi_id`        | int / text   | FK to `POIs.poi_id`.                                                                             |
| `flowpath_id`   | int / text   | Flowpath this hydrolocation is snapped to.                                                       |
| `vpuid`         | text   | Vector Processing Unit identifier.                                                               |
| `hl_type`       | text  | Source/dataset type, e.g., `nwis`, `huc12`, `nbi`, `hilarri`.                                    |
| `hl_link`       | text  | Native identifier in the source system.                                                          |
| `hl_class`      | text  | Semantic class of the hydrolocation, e.g., `gage`, `reservoir`, `lake`, `confluence`.            |
| `hl_reference`  | text  | Stable cross-system identifier, typically `{hl_type}-{hl_link}`. Unique across the model.        |
| `hl_uri`        | text  | Optional resolvable URI to the hydrolocation’s online resource.                                  |

---

### Network
| Column                  | Type   | Description                                                                 |
|-------------------------|--------|-----------------------------------------------------------------------------|
| `flowpath_id`           | int / text    | PK; FK → `flowpaths.flowpath_id`.                                           |
| `flowpath_toid`         | int / text    | Downstream flowpath reference.                                              |
| `mainstem_id`           | int    | Mainstem grouping id.                                                       |
| `hydroseq`              | int    | Flowpath hydrosequence.                                                     |
| `areasqkm`              | float  | Associated divide area (sq km).                                             |
| `lengthkm`              | float  | Flowpath length (km).                                                       |
| `total_dasqkm`          | float  | Total contributing drainage area at outlet (sq km).                         |
| `vpuid`                 | text    | Vector Processing Unit identifier.                                          |
| `divide_id`             | int / text    | FK → `divides.divide_id`.                                                   |
| `poi_id`                | int / text    | Optional FK → `pois.poi_id`.                                                |
| `hl_reference`          | text   | Optional crosswalk to a linked hydrolocation.                               |
| `hf_id`                 | text   | External Hydrofabric flowpath id.                                           |
| `hf_toid`               | text   | External Hydrofabric downstream id.                                         |
| `hf_source`             | text   | Source label for `hf_*` fields (e.g., `nhdplusv2`, `merit`).                |
| `hf_hydroseq`           | int    | External Hydrofabric hydrosequence.                                         |
| `flowline_id`           | int    | Underlying flowline id (if multiple are aggregated).                         |
| `flowline_toid`         | int    | Downstream flowline id.                                                     |
| `incremental_area_id`   | int / text    | FK → `incremental_area.incremental_area_id`                                       |
| `flowline_areasqkm`     | float  | Catchment area (sq km) at flowline scale.                                   |
| `flowline_hydroseq`     | int    | Hydrosequence at flowline granularity.                                      |
| `flowline_lengthkm`     | float  | Flowline length (km).                                                       |
| `flowline_total_dasqkm` | float  | Total drainage area at flowline outlet (sq km).                             |

---

### Flowlines
| Column               | Type   | Description                                                                 |
|----------------------|--------|-----------------------------------------------------------------------------|
| `flowline_id`        | int / text    | Unique identifier for the fine-scale flowline.                              |
| `flowline_toid`      | int / text    | Immediate downstream flowline id.                                           |
| `flowpath_id`        | int / text    | FK → `flowpaths.flowpath_id` that this flowline rolls up into.              |
| `vpuid`              | text     | Vector Processing Unit identifier.                                          |
| `mainstem_id`        | int    | Mainstem grouping id.                                                       |
| `flowline_hydroseq`  | int    | Hydrosequence at flowline granularity.                                      |
| `flowpath_hydroseq`  | int    | Hydrosequence of the parent flowpath.                                       |
| `incremental_area_id`| int / text    | FK → `incremental_area.incremental_area_id`.                                |
| `incremental_areasqkm`| float | Incremental catchment area for this flowline (sq km).                       |
| `lengthkm`           | float  | Flowline length (km).                                                       |
| `total_dasqkm`       | float  | Total contributing drainage area at the flowline outlet (sq km).            |
| `streamorder`        | int    | Strahler Stream Order                                                       |
| `slope`              | float  | Slope of flow segment (m/m)                                                 |

---

### Incremental Area
| Column                   | Type   | Description                                                                 |
|--------------------------|--------|-----------------------------------------------------------------------------|
| `incremental_area_id`    | int / text    | Unique identifier for the incremental catchment polygon.                    |
| `incremental_proportion` | float  | Proportion of the incremental area contributing to a flowline (0–1).        |
| `flowline_id`            | int / text    | FK → `flowlines.flowline_id` linked to this incremental catchment.          |
| `vpuid`                  | text    | Vector Processing Unit identifier.                                          |
| `areasqkm`               | float  | Incremental catchment area (sq km).                                         |
| `type`                   | text   | Category of area (e.g., `network`, `coastal`, `internal`).                  |

---

### Nexus
| Column        | Type   | Description                                                                 |
|---------------|--------|-----------------------------------------------------------------------------|
| `nexus_id`    | int / text    | Unique identifier for the nexus feature.                                    |
| `nexus_toid`  | int / text    | Immediate downstream id.                                           |     
| `vpuid`       | text    | Vector Processing Unit identifier.                                          |
---

## Relationships (ERD)

```mermaid
erDiagram
  FLOWPATHS {
    string    flowpath_id
    string    flowpath_toid
    text   vpuid
    string    poi_id
    int    mainstem_id
    float  areasqkm
    float  lengthkm
    float  total_dasqkm
    int    hydroseq
    int    streamorder
    float  slope
  }

  DIVIDES {
    string    divide_id
    string    flowpath_id
    text   vpuid
    float  areasqkm
    text   type
  }

  POIS {
    string    poi_id
    string    flowpath_id
    text   hl_classes
    text   hl_types
    int    hl_count
    text vpuid
  }

  HYDROLOCATIONS {
    string    poi_id
    string    flowpath_id
    text   vpuid
    text   hl_type
    text   hl_link
    text   hl_class
    text   hl_reference
    text   hl_uri
  }

  NETWORK {
    string    flowpath_id
    string    flowpath_toid
    int    mainstem_id
    int    hydroseq
    float  areasqkm
    float  lengthkm
    float  total_dasqkm
    text   vpuid
    string    divide_id
    string    poi_id
    text   hl_reference
    string   hf_id
    string   hf_toid
    text   hf_source
    int    hf_hydroseq
    string    flowline_id
    string    flowline_toid
    string    incremental_area_id
    float  flowline_areasqkm
    int    flowline_hydroseq
    float  flowline_lengthkm
    float  flowline_total_dasqkm
  }

  FLOWLINES {
    string    flowline_id
    string    flowline_toid
    string    flowpath_id
    text   vpuid
    int    mainstem_id
    int    flowline_hydroseq
    int    flowpath_hydroseq
    string    incremental_area_id
    float  incremental_areasqkm
    float  lengthkm
    float  total_dasqkm
    int    streamorder
    float  slope
  }

  INCREMENTAL_AREA {
    string    incremental_area_id
    float  incremental_proportion
    string    flowline_id
    text   vpuid
    float  areasqkm
    text   type
  }

  NEXUS {
    string  nexus_id
    string  nexus_toid
    text vpuid
  }

  %% Relationships
  FLOWPATHS ||--o{ DIVIDES : owns
  FLOWPATHS ||--o{ POIS : hosts
  POIS ||--o{ HYDROLOCATIONS : aggregates
  FLOWPATHS ||--o{ HYDROLOCATIONS : snapped_to
  
  FLOWPATHS ||--|| NETWORK : "canonical view"
  DIVIDES   ||--o{ NETWORK : referenced_by
  POIS      ||--o{ NETWORK : optional_poi
  
  FLOWPATHS ||--o{ FLOWLINES : aggregates
  FLOWLINES ||--|| INCREMENTAL_AREA : has
  
  NEXUS ||--o{ FLOWPATHS : pins
  NEXUS ||--o{ FLOWLINES : pins
