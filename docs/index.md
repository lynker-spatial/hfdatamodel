# Hydrofabric: From Standards to Service  

Mike Johnson + the Lynker Spatial Team

---

## Executive Summary
The **Lynker Hydrofabric** provides a (1) streamlined data model (2) a operational, concrete dataset and (3) set of software for tools for flexablly manipulating and representing hydrologic + hydraulic networks, catchments, and critical locations across the U.S. water enterprise. It builds upon and extends the **OGC WaterML 2.0 Part 3: Surface Hydrology Features (HY_Features)** standard (OGC 14-111r6), translating conceptual classes into tangible, high-performance data structures ment to meet the needs of multiscale and divergent modeling activities, increasing cloud based access, and provding a backbone for AI/ML modeling in the weather and water space.  This implementation improves upon past efforts by **USGS and NOAA** to position hydrology for the emerging **Data-as-a-Service paradigm** — interoperable, cloud-native, and AI/ML-ready.  It's in this way that the Lynker Hydrofabic moves beyond another hydrographic dataset towards an modeling infastructure in and of itself.


## Background: OGC HY_Features
The OGC HY_Features standard defines a conceptual model for hydrologic entities and their relationships. Central to this model are **catchments**, which represent the areas of land draining into hydrologic features; **hydro networks**, which express the topological connectivity of streams and rivers; and **hydrolocations**, which anchor the system to specific points of observation or control.  

HY_Features provides the critical benefit of **semantic consistency** across implementations worldwide (or at least with respect to the countries/agencies of the contributing authors). It ensures that catchments, stream networks, and observation points can be referenced and exchanged in a common way, supporting global interoperability in languguage and intent. At the same time, HY_Features is intentionally **abstract**. It describes a set of *what* _has__ be represented in the past, but it does not prescribe *how* those entities should be implemented in practice, or what *must* be represeneted. This leaves open the need for operational schemas that respect the conceptual model while delivering on performance, scalability, and integration.  

## The Hydrofabric Data Model
THe Lynker Hydrofabric fills this gap by providing a concrete implementation of HY_Features for U.S. national-scale workflows (which can be extended globally with accompanying `lynker-spatial/hf*` software suite). Our model defines **Flowpaths** as the canonical routed stream segments that carry identifiers, downstream references, and deterministic  ordering. Each flowpath is linked to a polygonal **Divide** that represents its contributing area and associated landscape properties.  

For finer-grained, multi-scale, or disjoint hydrologic/hydraulic analysis - our Hydrofabric introduces **Incremental Areas**, which partition divides into smaller units that can be rolled up or disaggregated as needed for proportional hydrologic accounting. Underlying these incremental areas are **Flowlines**, which represent the most granular routing primitives, which aggregate into flowpaths but maintain the detail required for high-resolution studies (e.g. street scale flood mapping, high resoltuion routing, and detailed cartography).  

Solid hydrologic modeling is as much hydrologic process reprensentation as it is accurate input data whether it be meterlogic forcings, observations data for training/calibration, or even insitu data assimulation. While most hydrographic products "link" or "cross-walk" data primarilty based on spatial proxiity (a dangerous operation when dealing with loosly accurate location data, and high resolution stream networks). Our hydrofabric is equipt with a set of over a quarter millions cafefully currated **Hydrolocations** sourced from systems like NWIS, NBI, NWS, HiLARRI, the WBD or other critical sturcutral features of the network that are mapped to the flowline representation of our primary network. Hydrolocations carry both their native identifiers and crosswalk keys as explicit source metadata (`hl_type`, `hl_link`), derive persisted unique identifiers (PID) and carry semantic classification (`hl_class`) enabling them to be seamlessly integrated into the Hydrofabric and used for (1) data manipulation (e.g. ensuring breaks in the network) (2) data assimulation (3) evaluation and (4) other tasks were multiple data streams can be used to enhance a given applications. The sheer number of relevant - but colaocated *hydrolocations* (e.g. a dam and gage both sitting on the outlet of a waterbody) require entities to be rolled up into critical network locations. Anchoring the hydrofabric network are **Points of Interest (POIs)**, which serve as named aggregation points where 1 or more hydrolocations are embeded in the geofabric.

Finally, Hydrofabric offers a **Network table**, a denormalized, query-ready view that blends flowpaths, divides, POIs, and hydrolocations. This wide representation allows users to execute routing, delineation, and accumulation analyses efficiently, without the need for multiple complex joins.  

## Key Connections

### Catchment Representation
	•	Hydrofabric: Implements **Divides** and **Incremental Areas** store polygonal catchments with attributes (e.g., areasqkm, type) tied directly to  **flowpaths** and **flowlines**.
	•	HY_Features: HY_Catchment and HY_CatchmentAggregate represent hydrologically meaningful drainage units.
	•	Alignment: Hydrofabric catchment entities directly instantiate HY_Catchment concepts, supporting both atomic and aggregated delineations.  
    Further, it provides incremental catchment partitioning, enabling scalable drainage aggregation and proportional accounting not specified in HY_Features.
### Hydrologic Network Connectivity
	•	Hydrofabric: Flowpaths, Flowlines, and the Network table capture directed connectivity, hydrosequence, and mainstem grouping.
	•	HY_Features: HY_HydroNetwork and HY_HydroLocation define feature-to-feature topological connectivity.
	•	Alignment: Hydrofabric uses explicit downstream identifiers (toid) and hydrosequence ordering, consistent with HY_Features’ graph-based model.
### Hydrologic Locations & Observations
	•	Hydrofabric: POIs and Hydrolocations crosswalk external sources (NWIS, NBI, HUC12, HiLARRI) into unified references (hl_reference).
	•	HY_Features: HY_HydroLocation provides the conceptual anchor for monitoring points, dams, and confluences.
	•	Alignment: Hydrofabric operationalizes HY_HydroLocation with a cross-system identifier strategy to integrate diverse data sources.
    
### Denormalization for Efficiency: Network Views
	•	Hydrofabric: Network provides a wide, read-optimized table flowpaths, divides, POIs, hydrolocations, and external Hydrofabric (hf_*) crosswalks.
	•	HY_Features: The standard is conceptual, leaving implementation details open with no prescribed performance optimizations.
	•	Alignment: Hydrofabric’s denormalized view is an implementation convenience but remains fully mappable to HY_Features feature classes.

## How Hydrofabric Extends HY_Features
	1.	Operationalization of Abstract Concepts
	        HY_Features is conceptual; Hydrofabric makes it tangible, with implemented schemas, SQL-friendly keys, and normalized IDs.
	2.	Performance-Oriented Enhancements
	        Adds hydrosequence ordering, denormalized tables, and incremental area partitions for efficient network traversal and large-scale analytics.
	3.	Cross-System Integration
            Introduces stable crosswalks (hl_reference, hf_*) to integrate diverse source systems, enabling interoperability across U.S. agency datasets.
	4.	Scalable Granularity
            Extends HY_Features’ catchment model with fine-scale Flowlines and Incremental Areas that support aggregation/disaggregation workflows.
	5.	Cloud-Native Readiness
            Designed for high-performance, distributed environments (HPC, cloud pipelines), beyond the scope of HY_Features’ conceptual framework.

## Why This Matters

By aligning with, and then extending, the OGC HY_Features model:
	•	Ensures interoperability with international hydrology standards.
	•	Provides a concrete, operational schema for U.S. agencies (NOAA, USGS, Lynker partners) while remaining standards-compliant.
	•	Enables exchange of network, catchment, and observation location data across systems without semantic loss.
	•	Bridges open standards with modern, performance-oriented data engineering practices.

**In short**: The Hydrofabric Data Model can be seen as an implementation profile of OGC HY_Features — optimized for U.S. national-scale workflows and cloud-native geospatial tooling, but conceptually consistent with the international standard.

---

## Background: OGC HY_Features
HY_Features defines a conceptual model for hydrologic entities such as catchments, networks, and hydrolocations as agreed upond by representivites of select organizations from select countries. Its aim was to come to a consensus of how real world featrues should be describes based on the sucesses and challenges encouted in each partiicapants repective countries model. It provides semantic consistency across implementations but intentionally leaves open how these classes should be physically implemented in databases or services.  

**Hydrofabric operationalizes this model** into a practical schema optimized for national-scale workflows and cloud-native infrastructures.

---

## The Hydrofabric Data Model
Hydrofabric defines the following core entities:  
- **Flowpaths** — canonical routed segments with hydrosequence and mainstem grouping.  
- **Divides** — polygonal catchments tied to flowpaths.  
- **Incremental Areas** — fine-grained catchment partitions for proportional analysis.  
- **Flowlines** — fine-scale routing primitives aggregated into flowpaths.  
- **POIs** — anchor points for observations and analysis.  
- **Hydrolocations** — crosswalks to external systems (e.g., NWIS, NBI, HiLARRI).  
- **Network** — denormalized view combining flowpaths, divides, and observation crosswalks.  

---

## Alignment with OGC HY_Features
Hydrofabric implements HY_Features concepts directly while extending them for operational use:  
- **Catchments** → Divides and Incremental Areas implement `HY_Catchment`.  
- **Networks** → Flowpaths and Flowlines instantiate `HY_HydroNetwork`.  
- **Locations** → POIs and Hydrolocations map `HY_HydroLocation` into concrete keys.  
- **Aggregations** → Network table mirrors HY_Features relationships but optimized for query performance.  

✅ **Result:** Hydrofabric is a faithful *implementation profile* of HY_Features, tuned for U.S. operational workflows.  

---

## Extensions Beyond HY_Features
Hydrofabric extends the conceptual model by:  
1. **Operationalizing abstract classes** with schemas and indexes.  
2. **Introducing incremental catchments** for proportional hydrology analysis.  
3. **Providing deterministic hydrosequence ordering** for network traversal.  
4. **Embedding crosswalks** across NOAA and USGS datasets.  
5. **Delivering cloud-native structures** (Parquet/GeoParquet) for distributed analysis.  

---

## Improving Past USGS & NOAA Efforts

Historically, USGS (NHD, NHDPlus, NLDI) and NOAA (NWM, NextGen) have developed powerful hydrologic data assets, but each effort faced limitations in interoperability, scalability, consistency, and enforces limitiations:
	•	Fragmentation: Different schemas and tiling strategies across NHD, NHDPlus HR, and NextGen prototypes.
	•	Performance Bottlenecks: Legacy databases and services optimized for cartography, not large-scale analytics.
	•	Integration Gaps: Difficulty aligning observations (gauges, dams, reservoirs) with network features across agencies.

The Hydrofabric data model addresses here these by:
	•	Unifying schema design that is conceptually aligned with HY_Features but extended for U.S. operational needs.
	•	Providing scalable incremental catchments that enable efficient roll-up/downstream analysis at any resolution.
	•	Embedding crosswalks so that NOAA NWM grids, USGS NWIS stations, and other sources are consistently linked.
	•	Optimizing for cloud analytics — designed to be deployed as parquet/GeoParquet in object stores for high-performance query and modeling.
.  
**Hydrofabric improvements**:  
- Unified schema to integrate across silos.  
- Scalable incremental catchments for proportional budgets.  
- Crosswalk identifiers for interoperability.  
- Cloud-optimized design for continental-scale analysis.  

---

Extending to a Data-as-a-Service Paradigm

As highlighted in GeoAI Unpacked (“The Rise of Geospatial Data as a Service”), modern geospatial infrastructure is shifting from static datasets to API-driven, cloud-native data services. The Hydrofabric model is ideally suited to this evolution:
	•	Interoperability by design: HY_Features alignment means Hydrofabric outputs can be served directly in standards-compliant APIs (OGC Features API, STAC).
	•	Modular access: Catchments, flowpaths, and POIs can be exposed as independent but linkable resources — supporting just-in-time delivery for AI/ML pipelines.
	•	Cloud-native storage: The denormalized Network and flowline tables are optimized for parquet/GeoParquet, enabling serverless queries and data streaming.
	•	Scalable integration: External datasets (e.g., climate projections, risk data) can join directly via stable crosswalks, powering new forms of water intelligence.

Result: Hydrofabric is not just a schema, but the foundation for hydrology as a service — a living, standards-based, cloud-native platform that brings USGS and NOAA efforts into the modern data economy. In short, the Hydrofabric Data Product operationalizes HY_Features, improves upon decades of USGS/NOAA work, and positions hydrology for the next era of Data-as-a-Service and GeoAI-enabled decision making.

---

## Value Proposition Matrix

| **Advantage**              | **Who Benefits**          | **Why It Matters**                                                  |
|-----------------------------|---------------------------|----------------------------------------------------------------------|
| Scalable granularity        | Researchers, Modelers     | Supports analysis from local to continental scale.                   |
| Deterministic traversal     | Developers, Engineers     | Simplifies routing and reduces error.                                |
| Cloud-native optimization   | Agencies, Cloud architects| Enables distributed queries at scale.                                |
| Crosswalk interoperability  | USGS, NOAA, Partners      | Unifies legacy and modern datasets.                                  |
| Unified schema              | Program managers          | Reduces duplication and supports collaboration.                      |
| Incremental catchments      | Scientists, Analysts      | Unlocks proportional hydrologic accounting.                          |
| GeoAI readiness             | AI/ML teams               | Provides structured training data for predictive analytics.           |

---

## Conclusion
The **Lynker Hydrofabric Data Model**:  
- Operationalizes **OGC HY_Features** into a working schema.  
- Extends it with innovations for performance, crosswalks, and scalability.  
- Improves upon decades of USGS and NOAA investments.  
- Positions hydrology for the **next era of Data-as-a-Service and GeoAI**.  

**From Standards → Implementation → Service**: Hydrofabric is the backbone of a modern, interoperable water data enterprise.  
