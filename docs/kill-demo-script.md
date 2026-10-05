# 30-second "Kill" Demo Script — The Living Map (Command Post)

Goal: in 30 seconds the jury sees the whole story — **a robot finds something, the Command Post knows it, an Executor goes there, and the system is honest about what it no longer knows.**

## Before you go on stage (rehearse this once)

1. `docker compose up -d` and open `http://localhost:5173` full screen (map visible, side panels visible).
2. Open Foxglove (layout `foxglove/living_map_robot_health.json`) on a second screen or tab, connected to `ws://localhost:8765`.
3. **Reset the simulation clock right before you start:** `docker compose restart ros`. The simulator's beats are timed from container start, so this makes the script below repeatable.
4. Wait ~3 s until the Writer appears on the map. Press record / start talking. **T = 0 s.**

The simulator timeline (from the container start) that the script relies on:

| Sim time | What happens |
|---|---|
| 0 s | `target_01` moves, `target_02` appears and is static, `target_04` arrives already ~40 s old |
| 10 s | Beacon **B-1** (victim, critical) appears → **Critical = 1** |
| 25 s | `target_02` stops being published → starts to age |
| 30 s | `target_03` appears; B-2 (hazard, warning) appears |
| 50–60 s | Writer LiDAR fault (sensor/hardware event) |

## The script (speak while you click)

| T | On screen | Say (English, 1 breath each) |
|---|---|---|
| **0–5 s** | Map with Writer, live targets (green), `target_04` already LOST (ancient timestamp, large ellipse) | "Inside a collapsed building there is no GPS and no network. Our Writer robot explores and every target here reports through the Outside Network Area — no direct link to us." |
| **5–10 s** | Point at the target list: PoD %, ±σ, LIVE/STALE | "Every point carries its age. Confidence decays with time, and the uncertainty ellipse grows — so we never confuse *old* with *true*." |
| **10–14 s** | **B-1 appears** on the map, **Critical pill turns red (1)** | "A victim beacon, left by the Writer minutes ago, just reached us. Critical: one." |
| **14–18 s** | **Click B-1 → press "Send Executor"** | "One click briefs the Executor through the ONA." |
| **18–25 s** | Executor line draws and the Executor moves towards B-1; mission status active | "The Executor enters later and goes straight to the inherited beacon — it does not start from zero." |
| **25–30 s** | `target_02` starts to age: ellipse visibly grows, state LIVE → STALE; (if time) Executor arrives, mission **done**, Critical returns to 0 | "And when a robot stops reporting, we say so: the ellipse grows, probability decays, and below 25 % the Command Post flags **re-scout**. Victim confirmed, mission done." |

**Closing line (2 s, optional):** "Sense. Communicate. Preserve. Continue the mission."

## Q&A insurance beat (only if asked "is it a sensor or your code?")

Switch to Foxglove: the **Diagnostics** and **Raw Messages** panels.

- One topic silent or a DiagnosticStatus ERROR ⇒ **sensor/hardware** (the LiDAR fault at 50–60 s of each minute shows this live).
- All Writer topics silent at once ⇒ **node-down** (software).
- Foxglove sees the data but the dashboard does not ⇒ the problem is downstream (bridge, MQTT, backend, WebSocket).

Optional live proof of the node-down case (**not yet tested — rehearse before using**): freeze the Writer simulator with `docker compose exec ros bash -c 'pkill -STOP -f writer_sim_node'`, show the single "node-down" alert, then resume with `pkill -CONT -f writer_sim_node`.

## Plan B if something misbehaves on stage

| Problem | Fix in 5 seconds |
|---|---|
| Dashboard empty | `docker compose restart ros` (the bridge republishes immediately) |
| Beacon B-1 does not show | Wait 10 s after the reset; B-1 is time-triggered |
| Executor does not move | Check a beacon is selected (highlighted) before pressing "Send Executor" |
| Browser shows "LINK LOST" banner | Backend restarting; the client reconnects on its own within seconds |

## Recording tips for the Phase 1 video

- 1920×1080, browser zoom 110 %, hide bookmarks bar.
- Record two takes: one with the 30-second script, one slower (60–90 s) with the fault-isolation beat in Foxglove.
- Keep the terminal showing `docker compose logs -f ros` in a corner of the second take: it proves the chain is live.
