# Guideline 08: Transport Telemetry Schema

A critical distinction between a photorealistic Omniverse render and a **Digital Twin** is live data. Case 01 simulates a transport monitoring system: the Smart Stops on Moskovsky Prospekt carry ETA data for approaching buses, trams, and trolleybuses. This document defines the full telemetry schema, the Houdini authoring workflow, and the Python simulation loop.

## 1. Why Primvars and Not customData

Two USD mechanisms can carry custom data on prims: `customData` dictionaries and `primvars:` (Primitive Variables).

| | `customData` | `primvars:` |
| --- | --- | --- |
| Hydra shader access | ❌ Not readable by shaders | ✅ Directly readable at render time |
| Python USD API | ✅ Readable | ✅ Readable |
| Performance | Slow on large traversals | Fast — GPU-native |
| Use case | Static metadata (address, era) | Dynamic telemetry (ETA, route ID) |

**Rule for Case 01:** All transport data that must be visualised (ETA colours, route labels on HUD) must live in `primvars:`. Static building metadata (address, architectural era) uses `customData`.

## 2. The Full Telemetry Schema (v1.0)

This schema is the binding contract between the data simulation layer (`src/transport_sim.py`) and the HUD visualisation extension. Any Python script that reads or writes transport data must use exactly these field names and types.

```usda
# Declared on the Smart Stop root prim:
custom string primvars:telemetry:schemaVersion = "1.0"
custom string primvars:stop:id                = "MSK_AV_STOP_12"
custom string primvars:transport:routeId      = "65"
custom float  primvars:transport:etaSeconds   = 142.0
custom string primvars:transport:vehicleType  = "bus"
custom string primvars:transport:direction    = "Moskovskaya Metro"
```

### Field Reference

| Field | Type | Valid Values | Notes |
| --- | --- | --- | --- |
| `schemaVersion` | String | `"1.0"` | Increment on breaking schema changes |
| `stop:id` | String | `"MSK_AV_STOP_<N>"` | Must be unique across the entire stage |
| `transport:routeId` | String | Route number string | Supports Cyrillic: `"К-306"` |
| `transport:etaSeconds` | Float | `0.0` – `3600.0` | `0.0` = vehicle at stop |
| `transport:vehicleType` | String | `"bus"`, `"tram"`, `"trolleybus"` | Controls HUD icon selection |
| `transport:direction` | String | Human-readable | Terminus name, Cyrillic allowed |

> [!IMPORTANT]
> The `schemaVersion` field must be present. Script parsers gate on this field — a stop missing `schemaVersion` is treated as a legacy prim and skipped by the HUD visualiser.

## 3. Authoring in Houdini (Batch Initialisation)

When setting up the stage, initialise all Smart Stop prims with v1.0 schema values using an **Attribute Wrangle** LOP or a Python Script LOP:

```python
# Python Script LOP — run once during stage setup
from pxr import Usd, Sdf

stage = hou.node("/stage").editableStage()

stop_prims = [p for p in stage.Traverse()
              if "stop_" in p.GetName() and p.GetKind() == "component"]

for i, prim in enumerate(stop_prims):
    prim.CreateAttribute("primvars:telemetry:schemaVersion",
                         Sdf.ValueTypeNames.String).Set("1.0")
    prim.CreateAttribute("primvars:stop:id",
                         Sdf.ValueTypeNames.String).Set(f"MSK_AV_STOP_{i+1:02d}")
    prim.CreateAttribute("primvars:transport:routeId",
                         Sdf.ValueTypeNames.String).Set("65")
    prim.CreateAttribute("primvars:transport:etaSeconds",
                         Sdf.ValueTypeNames.Float).Set(float(30 + i * 17))
    prim.CreateAttribute("primvars:transport:vehicleType",
                         Sdf.ValueTypeNames.String).Set("bus")
    prim.CreateAttribute("primvars:transport:direction",
                         Sdf.ValueTypeNames.String).Set("Moskovskaya Metro")
```

## 4. The Simulation Tick Loop (Omniverse Runtime)

At runtime inside Omniverse, `src/transport_sim.py` drives the ETA countdown:

```python
import omni.usd
import asyncio

async def transport_simulation_loop(tick_rate_hz: float = 1.0):
    """Update ETA primvars on all Smart Stops at the specified rate."""
    stage = omni.usd.get_context().get_stage()
    delta = 1.0 / tick_rate_hz

    while True:
        for prim in stage.Traverse():
            schema = prim.GetAttribute("primvars:telemetry:schemaVersion")
            if not schema or schema.Get() != "1.0":
                continue  # Skip non-Smart prims

            eta_attr = prim.GetAttribute("primvars:transport:etaSeconds")
            current_eta = eta_attr.Get() or 0.0
            new_eta = current_eta - delta

            if new_eta <= 0.0:
                # Vehicle has arrived — schedule next departure
                new_eta = 480.0  # 8-minute headway default

            eta_attr.Set(new_eta)

        await asyncio.sleep(delta)
```

## 5. HUD Colour Logic Contract

The ETA value drives the HUD panel colour — this contract must be consistent between the Omniverse Extension and any shader that reads `primvars:transport:etaSeconds`:

| ETA Range | Colour | Meaning |
| --- | --- | --- |
| `> 120 s` | 🟢 Green | Comfortable — no rush |
| `30–120 s` | 🟡 Amber | Approaching — be ready |
| `< 30 s` | 🔴 Red | Arriving — move to stop |
| `== 0` | ⬜ White | Vehicle at stop |

---

## ✅ Definition of Done (DoD)

* [ ] `primvars:telemetry:schemaVersion = "1.0"` is present on every Smart Stop root prim.
* [ ] All six schema fields are present and correctly typed upon inspection in Omniverse.
* [ ] `transport_sim.py` updates `etaSeconds` on all stops at 1 Hz without halting or crashing.
* [ ] HUD panel colour matches the ETA range table above across all test stops.
* [ ] A stage traversal filtering on `schemaVersion == "1.0"` correctly returns all Smart Stops and only them.
