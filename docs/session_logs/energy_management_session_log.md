# Energy Management System - Session Log
## March 13, 2026

---

## What Was Built

### Helpers Created (Settings → Helpers)
- `input_select.household_state` — Dropdown: Home / Away / Returning / Leaving / Extended Away
- `input_boolean.guest_house_occupied` — Toggle: guest house occupancy
- `input_boolean.vacation_mode` — Toggle: manual extended away override

### Automations Added (15 total, appended to /config/automations.yaml)
File is now 557 lines. Future work: split into packages before Hood River.

**Presence/Household State:**
- `Household State - Both Away` — triggers when both phones away for 5 min
- `Household State - Someone Arrives Home` — triggers on either phone arriving
- `Household State - Extended Away Auto` — escalates Away → Extended Away after 8 hrs
- `Household State - Vacation Mode On` — manual override to Extended Away
- `Household State - Vacation Mode Off` — re-evaluates presence on toggle off

**Guest House:**
- `Guest House - Occupied` — sets Rheem EcoNet to 49°C (120°F)
- `Guest House - Unoccupied` — drops Rheem EcoNet to 43°C (110°F)

**Main House Water Heater (Bradford White heat pump):**
- `Main WH - Away Mode On` — enables away mode when Away or Extended Away
- `Main WH - Away Mode Off` — disables away mode, sets eco when Home
- `Main WH - Pre-heat on Return` — boosts to high_demand when Returning, if hot water < 70%

**Pool Pump:**
- `Pool Pump - Solar Peak On` — starts at 10:00am if battery SOC > 20%
- `Pool Pump - Solar Peak Off` — stops at 3:30pm
- `Pool Pump - Low Battery Stop` — emergency stop below 20% SOC
- `Pool Pump - Battery Recovered` — restarts during solar hours when SOC > 30%
- `Pool Pump - Extended Away Minimum` — runs 2 hrs at 11am for water quality only

**Solar Surplus:**
- `Solar Surplus - Heat Water` — switches Bradford White to electric mode when battery > 95% SOC for 15 min AND PV > 2000W

---

## Key Entity IDs Confirmed

| Device | Entity ID |
|---|---|
| Bill's phone | `person.bill_babcock` / `device_tracker.ponophone` |
| Diane's phone | `person.diane` / `device_tracker.dianes_iphone_17_air` |
| Bradford White HP WH | `water_heater.heat_pump_water_heater` |
| Rheem EcoNet WH | `water_heater.electric_water_heater` |
| Pool pump | `switch.pool_pump` |
| Battery SOC | `sensor.luxpower_lxp_battery_state_of_charge` |
| PV power | `sensor.luxpower_lxp_pv_power_1` |
| Hot water available | `sensor.electric_water_heater_available_hot_water` |

## Bradford White Modes (fully controllable)
- `electric` — resistance only, fastest, ~4.5kW
- `heat_pump` — most efficient, slowest, ~1kW
- `eco` — heat pump first, resistance assist (default)
- `high_demand` — both elements, maximum recovery
- `off`
- away_mode: true/false (separate attribute)

---

## Integrations Confirmed Active
- Bradford White Connect (main house heat pump WH)
- Rheem EcoNet (guest house electric WH)
- Jandy iAqualink (pool — 26 entities)
- Refoss (16-circuit panel monitor, guest house)
- Enphase Envoy (33 devices, main house grid-tied)
- Solar Assistant via MQTT (EG4 18kPV — 71 entities)
- Mobile App — both phones via Nabu Casa

---

## Tested and Verified
- `input_select.household_state` set to Away → Bradford White switches to away mode ✅
- `input_select.household_state` set to Home → Bradford White returns to eco mode ✅
- All 15 automations loaded and enabled in HA ✅
- `ha core check` passes clean ✅

---

## Decisions Made

**Pool pump treated as infrastructure, not comfort load**
The lap pool is rarely used but serves as the fire system water reservoir and adds property value. Pool pump scheduling is therefore driven by water quality maintenance (ORP) and battery protection, not occupancy.

**Guest house occupancy affects water heater only**
Since pool use is minimal regardless of guests, guest_house_occupied only controls the Rheem EcoNet setpoint.

**Jandy internal timer to be disabled**
To avoid conflicts between Jandy's internal scheduler and HA automations, the Jandy timer should be disabled and HA should own the pool pump schedule entirely. Pending conversation with pool guy.

**Presence detection: phones only**
Bill's person entity previously had 5 device trackers (Macs, iPad, phone). Non-phone trackers removed to prevent false "home" readings when computers are on but Bill is out. Macs/iPad moved to person.billadmin.

**Copy-paste method for nano**
Wait for the "copy" icon to appear on code blocks before copying — ensures the full block is rendered before copying. Prevents partial pastes that corrupt YAML.

---

## Pending / To Do

1. **Returning state automations** — need to confirm distance sensor entity IDs:
   - Expected: `sensor.ponophone_distance_from_home`
   - Expected: `sensor.dianes_iphone_17_air_distance_from_home`
   - Verify these exist in Developer Tools → States

2. **Pool pump entity name** — verify `switch.pool_pump` matches actual Jandy entity

3. **Disable Jandy internal timer** — ask pool guy current schedule and pump model

4. **VSP speed control** — pool pump is variable speed but iAqualink only exposes on/off.
   Ask pool guy: pump model, speed programs, RS-485 availability

5. **Split automations.yaml into packages** — before Hood River deployment:
   - `packages/fire_system.yaml`
   - `packages/energy_management.yaml`
   - `packages/presence.yaml`

6. **Bradford White available hot water sensor** — confirm entity ID for tank status
   (currently using Rheem's `sensor.electric_water_heater_available_hot_water` in main WH pre-heat condition — may need correction)

7. **Hood River HA instance** — new build starting April 2026:
   - Home + shop as separate but related systems
   - Shop: EG4 18kPV + wall battery, container solar + ground mounts, 3-phase
   - Tesla charging as primary dispatchable load
   - HVAC present — presence/occupancy automation will have major impact

---

## System Architecture Summary (Maui)

**Off-grid (EG4 18kPV):** Guest house, guest house water heater, pool equipment room, fire sprinkler pump
**Grid-tied (Enphase):** Main house — 32 Sanyo 250W panels, grandfathered net metering with HECO. Do not modify.
**Main house water heater:** Bradford White heat pump WH — on grid but HA-controlled for efficiency
**Fire water source:** Lap pool (~20,000+ gal) with municipal gravity-fed makeup water
**No HVAC:** Natural ventilation only in both structures

---

## Notes on Future Energy Management System (Standalone Project)

Discussed during lunch session: the energy management work being built here should eventually become a standalone open-source project, separate from but cross-linked to the fire resilience system. Key design principles agreed:
- Context-aware, not schedule-based (occupancy, solar production, SOC, fire risk)
- Start with no-solar use case, work up to solar/battery
- Presence detection approach needs to be opinionated — phone-based presence is the most common failure point
- Own GitHub repo, own name
- Hood River deployment in April will be the second implementation and real-world test
