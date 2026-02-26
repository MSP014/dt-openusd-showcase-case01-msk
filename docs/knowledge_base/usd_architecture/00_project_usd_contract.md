# 00. Master USD Contract: Case 01 — Moskovsky Av

This document is the absolute baseline for all USD asset generation, validation, and assembly within Case 01. Every asset authored in Houdini and every scene assembled in Omniverse must conform to this contract. Non-conforming assets will fail the automated pre-flight pipeline.

## 1. Scene Fundamentals

* **MetersPerUnit:** `1.0` (1 Unit = 1 Metre)
* **UpAxis:** `Y`
* **Coordinate Origin:** World origin `(0, 0, 0)` is placed at the south-west corner of the first city block.

## 2. Naming Conventions

All prims must follow strict naming to enable automated pipeline tools.

| Asset Type | Prefix | Example |
| --- | --- | --- |
| Building | `bld_` | `bld_msk_150` |
| Smart Stop | `stop_` | `stop_msk_av_01` |
| Vehicle | `veh_` | `veh_bus_route_65` |
| Street Prop (lamp post) | `prp_` | `prp_lamp_period_soviet` |
| Material | `m_` | `m_brick_silicate_white` |
| Prototype (for instancing) | `proto_` | `proto_lamp_post_1960` |

All names use lowercase with underscores. No spaces. No Cyrillic.

## 3. LODs and Purpose (Geometry)

* **VariantSet Name:** Exactly `LOD` (not `lod`, `Lods`, or `Level_of_Detail`)
* **Variant Names:**
  * `LOD0` — Full detail. Intended for hero shots and close-up renders.
  * `LOD1` — Medium detail. Simplified facades, merged mesh islands. Default for mid-range view.
  * `LOD2` — Proxy / bounding box volume. Used by viewport for real-time navigation.
* **Purpose Application:** The `render`, `proxy`, and `guide` purpose attributes must be applied to `Geom` (Mesh) primitives — **never** to the root `Xform` of the building or component.

## 4. Instancing and Payload Hierarchy

* **Prototypes Location:** `/World/Prototypes/` — all shared blueprints live here.
* **Payload Application:** Heavy building geometry (LOD0 meshes) must be stored as **Payloads** inside the Prototype. The city block layout layer references only the container Xform.
* **Instanceable Flag:** Must be set on the layout-level prim (the instance pointer), never on the Prototype itself.

## 5. Asset Dependencies

* **Materials:** All shader graphs are stored in `assets/local/materials/materials_library.usda`.
* **Textures:** All bitmap maps (Albedo, Roughness, Normal, AO) are stored in `assets/local/textures/`.
* **Binding:** All material bindings use relative paths resolving from the project root (e.g., `../../assets/local/materials/materials_library.usda`). Absolute `C:/` paths are forbidden.

## 6. Transport Telemetry Schema (v1.0)

All Smart Stop prims must carry the following `primvars:` attributes. This schema is the contract between the Houdini assembly layer and the Omniverse HUD visualisation extension.

| Attribute | Type | Description |
| --- | --- | --- |
| `primvars:telemetry:schemaVersion` | String | `"1.0"` — version tracker |
| `primvars:stop:id` | String | Unique stop identifier, e.g., `"MSK_AV_STOP_12"` |
| `primvars:transport:routeId` | String | Route number, e.g., `"65"`, `"К-306"` |
| `primvars:transport:etaSeconds` | Float | Time to arrival in seconds, e.g., `142.0` |
| `primvars:transport:vehicleType` | String | `"bus"`, `"tram"`, or `"trolleybus"` |
| `primvars:transport:direction` | String | Human-readable terminus, e.g., `"Moskovskaya Metro"` |
