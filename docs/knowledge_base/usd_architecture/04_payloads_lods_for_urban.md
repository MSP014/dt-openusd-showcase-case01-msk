# Guideline 04: Payloads, LODs, and Urban Asset Loading

Managing rendering performance on Moskovsky Prospekt is fundamentally different from a controlled datacenter environment. The city block is open, deep, and infinitely complex. We solve the scale problem using two orthogonal USD mechanics:  **Composition Arcs** (RAM optimisation) and **LOD Purposes** (GPU optimisation).

## 1. The Urban Performance Problem

A single 500-metre stretch of Moskovsky Prospekt contains:

* ~30 unique building facades, each with detailed cornice and ornamental geometry
* Hundreds of instanced street props (lamp posts, trees, benches, kiosks)
* Multiple Smart Stop shelters with HUD anchors
* Road surface markings, kerbs, and lane meshes

Loading all of this at full resolution simultaneously would be impractical for real-time renders. The correct approach separates **when** data loads (Payloads) from **what** the GPU draws once loaded (LODs).

## 2. References vs. Payloads

### References (Immediate Load)

* **What it is:** The asset is loaded into memory when the parent stage opens.
* **When to use:** Lightweight Xform containers, metadata prims, stop anchors, and street prop positioning grids.
* **Rule for Case 01:** The city block assembly layer (`block_01.usda`) references building **containers** (empty Xforms with metadata) immediately. These give the scene its spatial layout without incurring geometry cost.

### Payloads (Deferred Load)

* **What it is:** Geometry is loaded only when explicitly requested by the user or a script.
* **When to use:** ALL heavy architectural geometry — building shells, LOD0 ornamental meshes, vehicle geometry.
* **Rule for Case 01:** The actual facade mesh inside each building Component must be stored behind a Payload. The city block opens instantly with bounding-box placeholders; an artist or script then loads specific buildings for inspection or hero-shot rendering.

## 3. LOD and Purpose Contract

Following the master contract in `00_project_usd_contract.md`:

1. **VariantSet Name:** Exactly `LOD`.
2. **Variants:** `LOD0` (full detail), `LOD1` (simplified), `LOD2` (proxy box).
3. **Purpose Application:** Within each LOD variant, the `render`, `proxy`, and `guide` purpose attributes attach to **Geom mesh prims only** — never to the root `Xform`.

### Viewport Workflow

Set the Omniverse viewport display filter to **Proxy** while navigating the city block. All buildings collapse to lightweight bounding volumes. Switch to **Render** to resolve full detail for the active camera shot — the GPU only resolves what is in the frustum.

## 4. Recommended Layer Stack

```text
master_scene.usda          # Root stage — references all block layers
└── block_01.usda          # City block 1 — references building containers
    └── bld_msk_150.usda   # Building Component — geometry behind Payload
        └── LOD VariantSet
            ├── LOD0: Shell + Detail (heavy, Payload)
            ├── LOD1: Shell only (lighter, Payload)
            └── LOD2: BBox proxy (instant, no Payload)
```

---

## ✅ Definition of Done (DoD)

* [ ] Every building with more than 50,000 triangles in LOD0 stores its geometry behind a Payload arc.
* [ ] Opening `block_01.usda` without loading payloads takes under 5 seconds in Omniverse.
* [ ] The `LOD` VariantSet is spelled identically across all building assets.
* [ ] Purpose tags (`proxy`, `render`) sit on Geom leaf prims, never on root Xforms.
