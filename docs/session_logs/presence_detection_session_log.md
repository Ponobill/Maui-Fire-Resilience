# Presence Detection Improvements - Session Log
## March 14, 2026

---

## What Was Built

### Zones Added
Three concentric zones now centered on property (20.92922, -156.27217):
- `zone.home` — existing, corrected center point
- `zone.nearby` — 1km radius, for "almost home" detection
- `zone.local` — 5km radius, for "in the area" detection
- `zone.near_home` — deleted, was incorrectly placed legacy zone
- `zone.bill_home` — legacy duplicate, to be deleted

### Phone Infrastructure
Both phones now have stable network presence:

| Phone | Person Entity | IP | MAC |
|---|---|---|---|
| Bill (iPhone 12, iOS 26) | person.bill_babcock | 192.168.68.203 | A0:FB:C5:A7:6E:BF |
| Diane (iPhone 17 Air) | person.diane | 192.168.68.202 | 64:31:35:A8:49:3D |

- Private WiFi Address disabled on both phones — real hardware MACs now in use
- DHCP reservations added in ER605 for both (entries 15 and 16)
- Ping sensors added via Ping (ICMP) integration:
  - `binary_sensor.192_168_68_202` — Diane's phone WiFi presence
  - `binary_sensor.192_168_68_203` — Bill's phone WiFi presence

### SSID Sensors Enabled
More reliable than ping for iOS — updates on WiFi change, not poll interval:
- `sensor.ponophone_ssid` — currently: `ponohouse`
- `sensor.dianes_iphone_17_air_ssid` — currently: `ponohouse`

Home SSIDs: `ponohouse`, `ponohouse_5GEXT`, `ponohouse_2GEXT`

### Automations Updated
Two existing automations updated to cross-reference SSID:

**Household State - Both Away** — now requires BOTH:
- Both GPS = not_home
- Both SSIDs not in home network list
- Prevents false Away trigger when phone GPS is stale but phone is on home WiFi

**Household State - Someone Arrives Home** — now triggers on ANY of:
- Either GPS = home
- Either SSID in home network list
- Catches cases where person arrives but GPS hasn't updated yet

---

## Key Learnings

**iOS WiFi optimization:** iPhones drop WiFi when screen is off, causing ping sensors
to show `off` even when phone is home. SSID sensor is more reliable — updates on 
WiFi association events, not continuous polling.

**Randomized MAC addresses:** Both iPhones were using private/randomized MACs by 
default. Had to disable Private WiFi Address in Settings → WiFi → ⓘ on each phone 
to get stable MACs for DHCP reservation.

**Angry IP Scanner vs Fing:** Angry IP missed Bill's phone; Fing found it. For iOS 
device discovery, Fing is more reliable.

**HA ping integration:** The `binary_sensor: platform: ping` YAML syntax is deprecated
in recent HA versions. Must use the Ping (ICMP) integration via UI instead. Entity IDs 
are auto-named from IP address (e.g. `binary_sensor.192_168_68_203`).

**nano shortcuts learned:**
- Ctrl+W — search
- Ctrl+K — cut entire line (fast bulk deletion)
- Esc+N — toggle line numbers (Alt+N opens new screen on this keyboard)
- Alt and Cmd share a key on Logitech MK670 — causes conflicts

---

## Companion App Sensors Available (ponophone)

| Entity | State | Use |
|---|---|---|
| `sensor.ponophone_ssid` | ponohouse | Home WiFi detection ✅ |
| `sensor.ponophone_bssid` | 20:23:51:10:79:ff | Access point ID |
| `sensor.ponophone_activity` | unavailable | Need to enable — automotive detection |
| `sensor.ponophone_battery_level` | 100 | Phone battery |
| `sensor.ponophone_battery_state` | Not Charging | |
| `sensor.ponophone_connection_type` | unavailable | WiFi vs cellular |
| `sensor.ponophone_distance` | unavailable | Distance from home zone |
| `sensor.ponophone_last_update_trigger` | unavailable | Why app sent update |
| `sensor.ponophone_location_permission` | Authorized Always | ✅ |
| `binary_sensor.ponophone_focus` | off | Screen focus state |

---

## Pending / To Do

1. **Enable Activity sensor** on both phones in Companion app:
   - Settings → Companion App → Sensors → Activity → Enable
   - Will expose `stationary/walking/automotive` states
   - `automotive` = driving → use for Returning detection

2. **Enable Connection Type sensor** on both phones
   - WiFi vs cellular as additional home confirmation

3. **Returning state automations** — use distance sensor once confirmed:
   - `sensor.ponophone_distance` — needs enabling in Companion app
   - Trigger Returning when distance < 2km and state is Away

4. **Stale location detection** — template sensor:
   - Flag when phone last_updated > 15 minutes
   - Use as signal to distrust GPS state

5. **Dead zone mapping** — longer term:
   - Log location + signal strength along common routes
   - Identify corridors where GPS/cell drops
   - Build dead reckoning compensation

6. **Automotive detection automation** — once Activity sensor enabled:
   - If activity = automotive AND heading toward home → set Returning
   - Combines velocity + direction for predictive presence

7. **Delete zone.bill_home** — legacy duplicate zone

---

## GitHub Sync Workflow (end of session)
Run from Mac terminal:
```bash
scp root@192.168.68.112:/config/automations.yaml ~/Maui-Fire-Resilience/config/automations/
scp root@192.168.68.112:/config/configuration.yaml ~/Maui-Fire-Resilience/config/
cd ~/Maui-Fire-Resilience
git add .
git commit -m "description"
git push
```

Git identity set:
- user.name: Bill Babcock
- user.email: bill@ponostyle.com
