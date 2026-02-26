# Guideline 06: Instancing for Urban Elements

Moskovsky Prospekt is not a unique sculpture — it is a **mass-produced urban environment**. Soviet-era street furniture, panel building sections, and avenue trees were designed for repetition. USD instancing is the natural fit, and ignoring it will collapse real-time performance the moment the scene reaches full avenue length.

> [!WARNING]
> Without instancing, a 500-metre stretch of Moskovsky Prospekt containing 200 lamp posts, 80 trees, and 40 bench units would load hundreds of complete geometry copies into RAM simultaneously. Viewport frame rates will drop to single digits.

## 1. The Urban Instancing Contract

Instancing loads one **Prototype** (the blueprint) and generates N lightweight **Instance pointers** to it. The rule for Case 01:

1. **The Prototype:** Store the heavy asset under `/World/Prototypes/`. Example: `/World/Prototypes/proto_lamp_post_1960`.
2. **The Payload:** Heavy LOD0 geometry lives inside the Prototype, behind a Payload arc.
3. **The Instances:** Place layout pointers at their street positions (e.g., `/World/MoskovskyAv/Block_01/Lamps/lamp_01`). This is where `instanceable = true` is set — **never** on the Prototype itself.

## 2. Ideal Candidates for Instancing in Case 01

| Asset | Estimated Count | Why Instance It |
| --- | --- | --- |
| Soviet-era lamp post (1960s) | 100–200 per avenue | Identical design, regular spacing |
| Avenue tree (linden / elm) | 80–150 per avenue | Same skeleton, colour varies via primvar |
| Panel building facade section (5m segment) | 50–100 per panel building | Modular construction, exact duplicates |
| Bus stop shelter (non-Smart) | 10–20 | Standard municipal model |
| Pavement tile | Thousands | Ground-plane repetition |

## 3. When NOT to Instance

| Avoid For | Reason |
| --- | --- |
| **Hero buildings** (e.g., n°150 with ornamental detail) | Each is architecturally unique; variants are preferable |
| **Smart Stops** | Each carries unique `primvars:stop:id` telemetry — per-instance data writes conflict with full USD instancing; use primvars approach described below |
| **Vehicles in motion** | Each vehicle position changes every frame |

## 4. Per-Instance Visual Variation via Primvars

When 80 identical linden trees must appear slightly different (seasonal colour, scale variation), use **Primitive Variables** rather than unique geometry:

* Author `primvars:displayColor` on the instance point in Houdini SOP layer.
* The Hydra render engine reads these primvars at render-time and passes them into the shared Prototype shader, allowing 80 geometrically identical trees to render with individual tinting.

```python
# Example: verify instancing stats in Omniverse Script Editor
import omni.usd
stage = omni.usd.get_context().get_stage()
stats = stage.GetSessionLayer()  # Use USD Stage Stats extension for PrototypeCount
```

Enable **Window → Utilities → Statistics** → set scope to **USD Stage** to read:

* `PrototypeCount` — number of unique blueprints
* `TotalInstanceCount` — total number of instance pointers
* Target ratio: at least 10 instances per prototype for meaningful performance gain.

---

## ✅ Definition of Done (DoD)

* [ ] The `instanceable = true` flag is set on all street-prop instance pointers, not on Prototypes.
* [ ] USD Stage Stats confirm: `TotalInstanceCount / PrototypeCount ≥ 10` for lamp posts and trees.
* [ ] Memory footprint confirms prototype geometry is loaded exactly once (single `PrototypeCount` entry per prop type).
* [ ] Tree colour variation is achieved via `primvars:displayColor` without breaking instancing (no unique geometry per tree).
