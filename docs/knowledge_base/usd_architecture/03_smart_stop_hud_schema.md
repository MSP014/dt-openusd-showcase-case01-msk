# Guideline 03: Smart Stop HUD Schema

The **Smart Stop** is the centrepiece of the Case 01 Digital Twin. It is the prim that demonstrates L1 Digital Twin capability: real-time state visualisation on a physical object. This document defines the JSON telemetry schema, the USD primvars contract, and the HUD wiring logic.

## 1. What Makes a Stop "Smart"

A standard bus stop is geometry. A Smart Stop prim is geometry **plus a live data layer**:

* It carries a `primvars:` telemetry namespace readable by both the Hydra renderer (for shader tinting) and the Omniverse Extension (for HUD panel overlay).
* It has a `StopAnchor` child Xform that pins the HUD panel in world space above the shelter roof.
* It can be queried programmatically — a Python script traverses the stage and collects all stops with routes to generate a departure board.

## 2. USD Asset Hierarchy

```text
stop_msk_av_01.usda
└── /stop_msk_av_01                    # Root Xform — Kind: component
    ├── /stop_msk_av_01/Geom           # Physical shelter geometry
    │   ├── /stop_msk_av_01/Geom/Shelter     # Roof + walls mesh
    │   └── /stop_msk_av_01/Geom/Pole        # Sign pole mesh
    ├── /stop_msk_av_01/Looks          # Material binding → m_stop_panel_metal
    └── /stop_msk_av_01/StopAnchor    # guide-purpose Xform — HUD attaches here
```

The `StopAnchor` prim is positioned 2.5 m above the shelter roof during Houdini authoring. The Omniverse Extension reads its world transform to place the floating HUD panel.

## 3. The Transport Telemetry Schema (v1.0)

These `primvars:` attributes are the contract between the data simulation layer and the visualisation layer. They live on the **root prim** of the Stop component.

```usda
def Xform "stop_msk_av_01" (
    kind = "component"
)
{
    custom string primvars:telemetry:schemaVersion = "1.0"
    custom string primvars:stop:id               = "MSK_AV_STOP_12"
    custom string primvars:transport:routeId     = "65"
    custom float  primvars:transport:etaSeconds  = 142.0
    custom string primvars:transport:vehicleType = "bus"
    custom string primvars:transport:direction   = "Moskovskaya Metro"
}
```

### Field Definitions

| Field | Type | Constraint | Notes |
| --- | --- | --- | --- |
| `primvars:telemetry:schemaVersion` | String | Must be `"1.0"` | Version guard for future parser migration |
| `primvars:stop:id` | String | Unique per stop | Format: `"MSK_AV_STOP_<N>"` |
| `primvars:transport:routeId` | String | Non-empty | Route number or name |
| `primvars:transport:etaSeconds` | Float | `≥ 0.0` | Remaining time to arrival |
| `primvars:transport:vehicleType` | String | Enum: `bus`, `tram`, `trolleybus` | Used by HUD to pick icon |
| `primvars:transport:direction` | String | Non-empty | Human-readable terminus name |

> [!IMPORTANT]
> All six fields must be present on every Smart Stop root prim. A stop missing `primvars:stop:id` will fail stage traversal validation and will not render a HUD panel.

## 4. The Data Update Loop (Python Simulation)

Since Case 01 operates on **simulated telemetry** (not a live API), a Python generator script runs inside Omniverse and updates `etaSeconds` on each stop at 1 Hz.

```python
# Conceptual outline — full implementation in src/transport_sim.py
def tick_transport_data(stage, delta_seconds: float):
    """Advance all Smart Stop ETA counters by delta_seconds."""
    for prim in stage.Traverse():
        if prim.HasAttribute("primvars:transport:etaSeconds"):
            attr = prim.GetAttribute("primvars:transport:etaSeconds")
            current_eta = attr.Get()
            new_eta = max(0.0, current_eta - delta_seconds)
            attr.Set(new_eta)
            # When ETA reaches 0: reset to next scheduled arrival time
```

## 5. HUD Visualisation Logic

The Omniverse Extension reads the primvars and renders a floating FUI panel:

1. **Traversal**: Find all prims where `primvars:telemetry:schemaVersion == "1.0"` and `primvars:stop:id` exists.
2. **World Transform**: Read the `StopAnchor` child prim's world matrix to position the panel.
3. **Render**: Draw a translucent HUD panel showing: route icon (`vehicleType`), route number (`routeId`), countdown timer (`etaSeconds`), direction string.
4. **Colour coding**: `etaSeconds > 120` → green. `30–120` → amber. `< 30` → red.

---

## ✅ Definition of Done (DoD)

* [ ] Every Smart Stop root prim carries all six `primvars:transport:` attributes in the `primvars:` namespace.
* [ ] `StopAnchor` child Xform exists and is positioned 2.5 m above the shelter roof.
* [ ] Stage traversal script correctly identifies all stops and returns their telemetry data.
* [ ] HUD panels appear above all stops in the Omniverse viewport without manual placement.
