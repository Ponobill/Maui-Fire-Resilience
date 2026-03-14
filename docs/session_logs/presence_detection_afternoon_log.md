# Presence Detection - Afternoon Session Log
## March 14, 2026 (afternoon)

---

## What Was Built

### Companion App Sensors Enabled - Bill (ponophone)
| Entity | State | Use |
|---|---|---|
| `sensor.ponophone_activity` | Stationary | Automotive detection for Returning |
| `sensor.ponophone_distance` | 0m | Distance from home zone |
| `sensor.ponophone_connection_type` | Wi-Fi | Network type confirmation |
| `sensor.ponophone_last_update_trigger` | Manual | Debug - why app sent update |

### Companion App Sensors Enabled - Diane
| Entity | State | Use |
|---|---|---|
| `sensor.dianes_iphone_17_air_activity` | Unknown | Needs movement to calibrate |
| `sensor.dianes_iphone_17_air_distance` | 147m | Active and reporting |
| `sensor.dianes_iphone_17_air_connection_type` | Wi-Fi | ✅ |
| `sensor.dianes_iphone_17_air_last_update_trigger` | Manual | ✅ |
| `sensor.dianes_iphone_17_air_ssid` | ponohouse | ✅ |

Note: Diane's Activity sensor shows Unknown — needs movement to calibrate.
Required enabling location permissions in iOS settings for some sensors.

### Returning Automations Added
Two automations added to automations.yaml:

**Household State - Bill Returning** (`automation.household_state_bill_returning`)
- Triggers when `sensor.ponophone_distance` drops below 2000m
- Condition: household state = Away AND bill = not_home AND activity = Automotive
- Action: set household state to Returning + notify

**Household State - Diane Returning** (`automation.household_state_diane_returning`)
- Triggers when `sensor.dianes_iphone_17_air_distance` drops below 2000m
- Condition: household state = Away AND diane = not_home AND activity = Automotive
- Action: set household state to Returning + notify

### Stale Location Detection Added
Added to configuration.yaml under `template:` block:

**binary_sensor.bill_phone_stale**
- On when person.bill_babcock hasn't updated in > 900 seconds (15 min)
- device_class: problem

**binary_sensor.diane_phone_stale**
- On when person.diane hasn't updated in > 900 seconds (15 min)
- device_class: problem

Both currently: `off` (phones reporting normally) ✅

### Stale Location Warning Automations
Two automations added:

**Presence - Bill Phone Stale**
- Triggers when bill_phone_stale turns on
- Condition: household state = Away
- Action: notify Bill's phone with presence warning

**Presence - Diane Phone Stale**
- Triggers when diane_phone_stale turns on
- Condition: household state = Away
- Action: notify Bill's phone with presence warning

---

## Complete Presence Detection Architecture

```
HOME if ANY:
  - person.bill_babcock = home (GPS)
  - person.diane = home (GPS)
  - sensor.ponophone_ssid in [ponohouse, ponohouse_5GEXT, ponohouse_2GEXT]
  - sensor.dianes_iphone_17_air_ssid in [ponohouse, ponohouse_5GEXT, ponohouse_2GEXT]

AWAY if ALL:
  - person.bill_babcock = not_home
  - person.diane = not_home
  - sensor.ponophone_ssid NOT in home SSIDs
  - sensor.dianes_iphone_17_air_ssid NOT in home SSIDs

RETURNING if:
  - distance < 2000m
  - activity = Automotive
  - household state = Away
  - person = not_home

EXTENDED AWAY if:
  - Away for 8+ hours (auto)
  - OR vacation_mode = on (manual)

STALE WARNING if:
  - Phone not updated in 15+ min while Away
```

---

## Zones Configured
All centered on 20.92922, -156.27217 (confirmed correct via Google Maps + iOS Compass app):

| Zone | Radius | Purpose |
|---|---|---|
| `zone.home` | ~187m | On property |
| `zone.nearby` | 1000m | Almost home |
| `zone.local` | 5000m | In Haiku area |

Note: `zone.bill_home` exists as legacy — left in place pending investigation of what uses it.

---

## Pending / To Do

1. **Diane's Activity sensor** — shows Unknown, needs her to drive somewhere to calibrate
2. **Test Returning automation** — needs real-world test when driving home from away
3. **Pool pump entity** — verify `switch.pool_pump` matches Jandy entity name
4. **Disable Jandy internal timer** — ask pool guy current schedule and pump model
5. **Split automations.yaml into packages** — before Hood River
6. **Dead zone mapping** — longer term, collect data on coverage gaps along common routes

---

## File Status
- automations.yaml: 21KB, ~580 lines
- configuration.yaml: 4570 bytes
- Both pushed to GitHub: https://github.com/Ponobill/Maui-Fire-Resilience

## Keyboard Note
Current keyboard (Logitech MK670) has Alt/Cmd on same key causing conflicts with
nano shortcuts. Replacement: Kinesis Freestyle Pro with Cherry MX Brown switches
+ Lift Kit tenting accessory. Bill's preferred reference: IBM Model M15 (split
buckling spring, declined $1500 offer, irreplaceable).
