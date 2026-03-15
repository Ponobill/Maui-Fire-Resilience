# Frigate Session Log

---

## Session: March 15, 2026 — Hood River Frigate Pi Setup

### Hardware
- New Raspberry Pi (hostname: Raspberrypironman)
- Coral USB Accelerator
- Pi IP: 192.168.68.232

### Completed
- Docker installed and verified (hello-world test passed)
- Created ~/frigate/ directory structure
- Created docker-compose.yml with Coral USB passthrough (/dev/bus/usb:/dev/bus/usb)
- Created config/config.yml with Coral edgetpu detector and both Reolink cameras
- Frigate stable image pulled and running — healthy, Coral detected at 10ms inference

### Camera Configuration
- front: 192.168.68.59 (was incorrectly set to .58 initially)
- back: 192.168.68.77
- Credentials: admin / sk1ppy0n0p
- RTSP confirmed enabled in camera web UI (port 554, Server Settings → Network → Advanced)

### Left Unresolved
- Cameras not showing in Frigate UI
- ffprobe confirmed connection refused on .58 (wrong IP)
- ffprobe test on corrected IP .59 was pending when Pi was swapped out
- Likely fix: verify correct RTSP URL path for Reolink model
- URL to test: rtsp://admin:sk1ppy0n0p@192.168.68.59/h264Preview_01_main

### Next Session
- Reconnect Hood River Pi
- Run ffprobe test to confirm correct RTSP URL
- Update config.yml with working stream URLs
- Add remaining cameras once streams confirmed working

---

## Future: Maui Frigate Pi (Original) — Hailo-8L

- Currently running CPU detection due to HailoRT version mismatch
- Host has HailoRT 4.23.0, Frigate container bundles 4.21.0
- Hailo packages held via apt-mark hold
- Plan: roll back Frigate to a version that supports 4.23.0 when available
