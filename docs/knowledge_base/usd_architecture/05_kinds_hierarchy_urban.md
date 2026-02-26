# Guideline 05: Kinds Hierarchy — Urban Scene

The USD `kind` metadata system is the mechanism that makes large scenes navigable. Without it, clicking anywhere in a 30-building street scene randomly selects individual mesh polygons. With it, a single click correctly selects the entire building, the entire city block, or the specific Smart Stop you are inspecting.

## 1. Project UX Rule

> **A `Component` or `Subcomponent` must always live inside a `Group` or an `Assembly`.**

This rule ensures the Asset Validator in Omniverse produces zero `KindChecker` errors and guarantees that scene selection behaves as an urban planner would expect.

## 2. The Urban Hierarchy for Case 01

The Case 01 hierarchy maps to the physical structure of the city:

| Kind | USD Level | Real-World Analogy | Example Prim Path |
| --- | --- | --- | --- |
| `group` | Avenue | The entire Moskovsky Prospekt | `/World/MoskovskyAv` |
| `assembly` | City Block | One block between intersections | `/World/MoskovskyAv/Block_01` |
| `component` | Individual Building or Stop | A single building or shelter | `/World/MoskovskyAv/Block_01/bld_msk_150` |
| `subcomponent` | Named Detail | A specific Smart Stop on a building | `/World/MoskovskyAv/Block_01/bld_msk_150/stop_msk_av_01` |

## 3. Why Kinds Matter in Practice

**Without correct Kinds:**

* Clicking a lamp post selects a single triangle of its mesh.
* "Select All Buildings on Block 01" requires a Python script with name-pattern matching — slow, fragile.
* The Asset Validator throws `KindChecker` warnings on every prim.

**With correct Kinds:**

* Single-click on any part of the building selects the entire `component` — the whole building highlights.
* Double-click descends into the `subcomponent` level — the Smart Stop shelter highlights separately.
* Python tools use `prim.GetKind() == "assembly"` to instantly collect all blocks.

## 4. Houdini Implementation

Use the `Configure Primitive` LOP node in Solaris:

1. Select the avenue root (`/World/MoskovskyAv`) → set Kind to `group`.
2. Select each block group (`Block_01`, `Block_02`, ...) → set Kind to `assembly`.
3. Select each building and Smart Stop root prim → set Kind to `component`.
4. Select named sub-elements requiring independent inspection (e.g., a specific stop shelter attached to a building's footprint) → set Kind to `subcomponent`.

> [!TIP]
> In Houdini Solaris, you can define Kind in the `primSpec` metadata block of a typed LOP. Alternatively, use an **Edit Properties** LOP targeting the root prim and add `kind = "assembly"` as a string metadata entry. This survives USD roundtrips cleanly.

---

## ✅ Definition of Done (DoD)

* [ ] Running the Asset Validator in Omniverse yields zero `KindChecker` errors.
* [ ] Clicking any part of `bld_msk_150` in the Viewport selects the entire `component` (building root), not a polygon.
* [ ] A Python script using `prim.GetKind() == "assembly"` correctly returns all city block prims and only them.
* [ ] Smart Stop prims tagged as `subcomponent` allow focused individual selection without selecting the parent building.
