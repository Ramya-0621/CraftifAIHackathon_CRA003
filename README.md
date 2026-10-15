<<<<<<< HEAD
# halo
=======
# HALO: Fall-Aware Home Simulator

**HALO** (*Human Assistance and Live Observation*) is a single-file, browser-based simulator of a fall-aware home safety system. It models the full loop from a vision pipeline (**PipeGen**) through **MQTT** to an **ESP32** device (**FirmGen**) and on to a caregiver's phone, so the behaviour can be demonstrated and checked without any hardware.

> **The camera never decides alone. The room asks first, and says so when it can't see.**

> **Note:** Every value in this simulator is simulated. A hardware build must feed the same panels from real MQTT messages and UART output.

## Quick start

No build step, no dependencies, no network access needed.

```bash
git clone <your-repo-url>
cd halo-fall-aware-home-simulator
# Option 1: just open the file
open index.html            # macOS  (Linux: xdg-open, Windows: start)
# Option 2: serve it locally (needed if your browser restricts mic access on file://)
python3 -m http.server 8000   # then visit http://localhost:8000
```

Works in any modern browser. Voice input uses the Web Speech API (best in Chrome/Edge); the typed box works everywhere.

## How it works

```
 Camera ──► PipeGen (skeleton keypoints only, no pixels)
                 │  MQTT  halo/room1/state, halo/room1/heartbeat
                 ▼
          ESP32 "Halo" device (FirmGen)  ── LED ring, buzzer, path lights
                 │  BLE + MQTT (two-way)
                 ▼
          Caregiver phone  ◄──► "I'm on my way" acknowledgement
```

### Escalation stages

| Stage | Meaning | Device behaviour |
|-------|---------|------------------|
| 0 | Watching | Idle; amber "confirming" state if a possible fall is seen |
| 1 | Concern | Path lights on, ring breathing |
| 2 | Asking "are you OK?" | Voice prompt, buzzer chirp (muted in quiet hours); waits for wave, button, or voice |
| 3 | Escalated | Strobe, loud alarm, BLE alert to caregiver phone |

Default timings: Stage 1 after the zone grace period, Stage 2 at 8 s, Stage 3 at 25 s (plus any learned extra grace).

### Requirements checked live in the simulator

The page tracks acceptance evidence for these requirements and marks each one as it passes:

| ID | Requirement |
|----|-------------|
| R1 | Subscribe to `halo/+/state` and heartbeat over MQTT |
| R2 | Stage 1 within 200 ms of "down", held ≥ 3 s |
| R3 | Escalate at 8 s and 25 s via RTOS timer |
| R4 | Cancel within 100 ms of button or standing |
| R5 | Eyes offline after 3 s without heartbeat |
| R6 | BLE alert advertised at Stage 3 |
| R7 | Every transition logged with timestamp |
| R8 | Fail safe to Stage 3 if sight is lost mid-escalation |

## Features

- **Skeleton-only pipeline view**: standing, sitting, and lying poses, with no pixel data.
- **ESP32 device view**: 16-LED ring with breathe, strobe, and chase patterns; live hardware-state text.
- **UART log and MQTT feed** (`halo/room1/*`) as acceptance evidence.
- **Room memory**: per-zone grace periods (open floor, yoga mat, pet corner, bathroom). After 2 false alarms in a zone, 3 s are added there. The bathroom is safety-capped and never relaxes.
- **Adjustable floor-confirmation window** (1 to 5 s) and **quiet hours** mode.
- **Caregiver phone panel** with two-way acknowledgement.
- **Voice assistant**: speech output plus voice or typed commands ("I'm okay", "cancel", "help", "stop alarm").
- **Privacy ledger** and a **"Today at home"** summary built from skeletons only.
- **Edge optimisation panel**: FP16 vs INT8 engine (example numbers for layout only; replace with real PipeGen benchmarks).
- **Performance monitor**: inference rate (5/10/20 fps), simulated MQTT loopback latency, and message rate.
- **Fault injection console**: duplicate event, invalid command, silence command, MQTT broker drop, ESP32 reboot, camera unplug.
- **Event ledger**: generated, received, acknowledged, resolved, with filtering.
- **Clock speed** control (1x, 3x, 6x) and light/dark theme support.

## Controls

| Key | Action |
|-----|--------|
| `1` | Standing |
| `2` | Sitting on chair |
| `3` | Lying on floor |
| `B` | Press the big button |
| `W` | Wave arm |

Sound plays automatically on alerts. If your browser blocks audio, click anywhere on the page once.

## Try a demo scenario

1. Click **Lying on floor** (or press `3`) and watch the device move from Watching to Concern, then Asking.
2. Do nothing until Stage 3 fires, then click **Caregiver: I'm on my way** to see two-way acknowledgement.
3. Repeat, but press **B**, wave, or type "I'm okay" to cancel and log a false alarm.
4. Click **Unplug camera** mid-escalation to see the fail-safe behaviour (R5, R8).
5. Switch to **Yoga mat** under Room memory and see the longer grace period.

## Project structure

```
halo-fall-aware-home-simulator/
├── index.html    # the complete simulator (HTML, CSS, JS in one file)
├── README.md
└── .gitignore
```

## Moving to real hardware

The simulator's message shapes are the contract to implement:

```json
// halo/room1/state
{"state": "down", "held_s": 3.2, "zone": "floor", "conf": 0.91, "ts": 12345}

// halo/room1/heartbeat
{"ts": 12345, "fps": 28}
```

The board also publishes retained state on `halo/board/state`. Replace the simulated publishers with the real PipeGen output and the real ESP32 UART/MQTT streams.

## Publishing on GitHub Pages

Settings → Pages → deploy from the `main` branch root. The simulator will be live at `https://<username>.github.io/<repo>/`.

## License

Add a license of your choice (e.g. MIT) as `LICENSE` before publishing.
>>>>>>> b364fa2 (Initial commit)
