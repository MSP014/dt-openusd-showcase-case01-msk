# Guideline 01: Scale and Axis Setup

## 1. The Core Standard: Houdini Native

In the Case 01 Urban Digital Twin, the local authoring standard uses Houdini's native settings. This ensures procedural city-block generation proceeds cleanly without continuous conversion arithmetic.

The fundamental laws for all exported assets are:

* **Metres Per Unit**: `1.0` (1 Unit = 1 Metre)
* **Up-Axis**: `Y` (Y-Up)

Real-world reference: Moskovsky Prospekt is approximately 60 metres wide (including sidewalks and lanes). In USD this reads as `60.0` units across the avenue cross-section. Any asset that violates this scale will immediately appear visibly wrong at street level.

## 2. Omniverse Auto-Resolution

OpenUSD is a self-describing format. As long as Houdini correctly embeds the layer metadata on export, compliant platforms (like Omniverse) will read `upAxis = "Y"` and `metersPerUnit = 1` and automatically apply a `unitsResolve` transform. The result: assets appear at the correct scale relative to Z-Up / Centimetre worlds without any manual correction.

## 3. Configuring in Houdini (Solaris)

Before writing geometry to disk, use the `Configure Layer` LOP:

1. Append a **Configure Layer** node before your USD ROP.
2. Set **Up Axis** to `Y`.
3. Set **Metres Per Unit** to `1.0`.

This must be present for every exported layer: building shells, street props, the master assembly scene.

## 4. Pipeline Defence: Ingesting External Assets

> [!CAUTION]
> Purchased or downloaded assets (NVIDIA Asset Browser, TurboSquid, SketchFab) often default to Z-Up and Centimetres (`metersPerUnit = 0.01`).
> **Rule:** All external assets must pass through an ingestion script that reads their `metersPerUnit` and `upAxis` metadata. The script must either bake the transform or apply a rigid root `Xform` correction before the asset enters the Case 01 master assembly.

A building authored at `metersPerUnit = 0.01` would appear at 1% of its real-world size — rendering the entire avenue as a dollhouse.

---

## ✅ Definition of Done (DoD)

* [ ] Every exported root layer explicitly contains `metersPerUnit=1.0` and `upAxis=Y`.
* [ ] Omniverse import requires **zero** manual rotation or scaling adjustments.
* [ ] An automated pre-flight script reads root layer metadata and fails the pipeline if external assets conflict without a resolving `Xform`.
