# Guideline 07: Centralised Materials Library

Case 01 operates on a **Single Source of Truth** principle for all surface materials. Every building facade, street shelter, lamp post, and pavement tile references one central library. There are no local material folders nested inside individual asset files.

## 1. The Urban Material Problem

Moskovsky Prospekt contains buildings across several architectural eras. However, many surface materials repeat:

* The same **silicate brick** appears on multiple Stalinist-era facades.
* The same **panel concrete** texture tiles across dozens of Khrushchev-era buildings.
* The same **oxidised copper** roof cladding appears on symmetrical mirrored buildings.

Duplicating `m_silicate_brick.usda` inside 12 separate building Component files creates:

* **Mass-update impossibility** — correcting a reflectance value requires editing 12 files.
* **Memory duplication** — the GPU loads the same texture 12 times.
* **Version drift** — files diverge silently over time as individual tweaks accumulate.

## 2. The One Library Paradigm

There is **one** master materials file: `assets/local/materials/materials_library.usda`.

Rules:

* **All** surface definitions for Case 01 — from ornamental gilding to tarmac — live in this file.
* **Every** asset exported from Houdini references material bindings from this file via relative path.
* All raw bitmap textures (Albedo, Roughness, Normal, AO) live exclusively in `assets/local/textures/`. The materials library file must not contain embedded texture data.

## 3. Material Naming Convention

All materials use the `m_` prefix and follow a descriptive pattern:

```text
m_<material_type>_<variant>

Examples:
m_brick_silicate_white
m_brick_red_clinker
m_concrete_panel_ussr
m_glass_storefront_clear
m_metal_copper_oxidised
m_asphalt_roadway
m_granite_sidewalk_grey
m_stop_panel_metal
m_lamp_post_cast_iron
```

## 4. Binding Strategy

All Component assets bind materials using **relative paths**:

```usda
rel material:binding = </MaterialLibrary/m_brick_silicate_white>
# ... which resolves via the reference arc:
# (../../../assets/local/materials/materials_library.usda)
```

> [!TIP]
> **Houdini Implementation:** When configuring the `Material Library` LOP, explicitly write the **relative material path** rather than the absolute asset path. Format: `../../assets/local/materials/materials_library.usda`. This guarantees portability across workstations and Omniverse Nucleus servers without broken `C:/` drive references.

## 5. Era-Based Material Groups

For clarity within the library, materials are grouped by architectural era using USD `Scope` prims:

```text
/MaterialLibrary
├── /MaterialLibrary/Stalinist_1940s_1960s
│   ├── m_brick_silicate_white
│   ├── m_stone_granite_red
│   └── m_metal_copper_oxidised
├── /MaterialLibrary/Soviet_Panel_1960s_1980s
│   ├── m_concrete_panel_ussr
│   └── m_glass_panel_soviet
├── /MaterialLibrary/Street_Infrastructure
│   ├── m_asphalt_roadway
│   ├── m_granite_sidewalk_grey
│   ├── m_stop_panel_metal
│   └── m_lamp_post_cast_iron
└── /MaterialLibrary/Vegetation
    └── m_foliage_linden_summer
```

---

## ✅ Definition of Done (DoD)

* [ ] No `materials/` folders exist nested inside individual building Component asset directories.
* [ ] Material definitions (`.usda`) and raw texture maps (`.png` / `.exr`) are separated into two distinct root directories.
* [ ] All material bindings on Component assets trace back through relative cross-directory links.
* [ ] Updating `m_brick_silicate_white` roughness in the library instantly affects all buildings referencing it — verified in Omniverse without reimporting assets.
