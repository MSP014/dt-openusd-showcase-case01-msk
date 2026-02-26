# Guideline 02: Building Facade Asset Structure

A building in an urban Digital Twin is not a monolithic mesh. It is a **USD Component** composed of several independently swappable layers. This document defines the hierarchy and composition rules for all architectural assets in Case 01.

## 1. The Problem with a Single Mesh

Moskovsky Prospekt contains buildings from different eras: Stalinist classicism, Soviet modernism (panel), and post-Soviet commercial. A naive approach (one `.fbx` dump per building) creates:

* **Texture management chaos** — no way to re-skin facade materials across multiple buildings at once.
* **LOD impossibility** — you cannot swap only the ornamental detail without rebuilding the whole mesh.
* **HUD anchoring failure** — the Smart Stop HUD prim needs a stable, named attachment point on the building footprint, which does not exist in a flat mesh hierarchy.

## 2. The USD Component Hierarchy

Each building is authored as a self-contained USD Component file following this structure:

```text
bld_msk_150.usda
└── /bld_msk_150                    # Root Xform — Kind: component
    ├── /bld_msk_150/Geom           # All mesh geometry
    │   ├── /bld_msk_150/Geom/Shell         # Main facade volume mesh
    │   ├── /bld_msk_150/Geom/Roof          # Roof geometry
    │   ├── /bld_msk_150/Geom/Detail        # Cornices, pilasters, ornament (LOD0 only)
    │   └── /bld_msk_150/Geom/Footprint     # Invisible ground-plane prim for anchoring
    ├── /bld_msk_150/Looks          # Material bindings (references to materials_library.usda)
    └── /bld_msk_150/Metadata       # Custom attributes: era, address, floors count
```

## 3. LOD VariantSet

Each building root carries a `LOD` VariantSet. The switching is managed at the city-block assembly layer, not inside the building file itself.

| Variant | Geometry | When Used |
| --- | --- | --- |
| `LOD0` | Full shell + Detail ornament + Roof geometry | Hero shot, close-up camera |
| `LOD1` | Shell + Roof only, simplified mesh | Mid-range camera, typical avenue shot |
| `LOD2` | Single bounding-box `Cube` prim, no texture | Real-time viewport navigation, distant background |

> [!TIP]
> **Houdini Implementation:** Use the `LOD` VariantSet LOP to wrap geometry subsets inside variants. Export all three as a single `.usda`. The `Detail` sub-prim is authored only within `LOD0`; it is simply absent from `LOD1` and `LOD2` — no visibility toggling needed.

## 4. Building Metadata Schema (Custom Attributes)

All buildings carry static descriptive metadata. These are **not** telemetry primvars — they are authored once in Houdini and never updated at runtime.

| Attribute | Type | Example |
| --- | --- | --- |
| `customData:address` | String | `"Moskovskiy Prospekt, 150"` |
| `customData:era` | String | `"stalinist_1950s"`, `"soviet_panel_1970s"` |
| `customData:floors` | Int | `9` |
| `customData:hasSmartStop` | Bool | `True` / `False` |

## 5. The Footprint Prim

Every building component contains an invisible `Footprint` prim at ground level. This is a zero-height `Plane` or `Xform` prim tagged with `purpose = "guide"`. The Smart Stop asset anchors its HUD to the building's `Footprint/StopAnchor` point — ensuring the HUD survives re-exports or geometry edits to the facade shell.

---

## ✅ Definition of Done (DoD)

* [ ] Each building exports as a single `.usda` with a `LOD` VariantSet containing all three variants.
* [ ] The `Detail` sub-mesh is present only in `LOD0`.
* [ ] All four `customData:` attributes are present and validated on each building root prim.
* [ ] A `Footprint` guide prim exists for every building that carries `customData:hasSmartStop = True`.
