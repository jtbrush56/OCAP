# Usage Documentation Design

**Date:** 2026-05-08
**Output file:** `docs/usage.md`

## Goal

Create a single usage reference for OCAP covering both mission integration (SQF/CBA API for mission designers) and the web playback UI (for players and viewers). Sources: SQF files in `addon/addons/recorder/`, web UI source in `web/ui/src/`.

## Audience

- **Mission designers** — familiar with SQF and Arma 3 mission editing; need to know when recording starts, how to call export at mission end, and how to integrate custom events and counters.
- **Players / viewers** — using the web UI to find and watch recordings; need to know how to navigate, follow units, and use keyboard shortcuts.

---

## Structure

### Part 1 — Mission Integration

#### Section 1: Auto-Start

How the auto-start gate works:
- When `OCAP_settings_autoStart` is true (default), recording begins automatically after the briefing screen once `OCAP_settings_minPlayerCount` players are connected (default: 15).
- When recording starts, every player receives a diary entry under "OCAPInfo" confirming the start time (in-mission time and UTC).
- If auto-start conditions are never met (e.g. too few players), recording does not start unless triggered manually via `OCAP_record`.

#### Section 2: Ending a Mission and Exporting

The primary export function: `ocap_fnc_exportData`.

Parameters (all optional):
- `_side` — the winning side (`west`, `east`, `independent`, `sideUnknown`, or omitted)
- `_message` — a description of the outcome
- `_tag` — overrides `OCAP_settings_saveTag` for this recording; used to filter recordings in the web UI

All four call forms with examples:

```sqf
// No args — records "Mission ended"
[] call ocap_fnc_exportData;

// Winning side only
[west] call ocap_fnc_exportData;

// Side + message
[east, "OPFOR controlled all sectors!"] call ocap_fnc_exportData;

// Side + message + tag (filterable in playback menu)
[east, "OPFOR controlled all sectors!", "PvP"] call ocap_fnc_exportData;
```

Trigger pattern (recommended integration):
```sqf
// In a trigger's On Activation field, server-side:
if (isServer) then {
    [east, "OPFOR has achieved all of their objectives!"] call ocap_fnc_exportData;
    "end1" call BIS_fnc_endMissionServer;
};
```

Async save flow: calling `ocap_fnc_exportData` is non-blocking. Players see an interim "saving…" toast immediately. Once the extension finishes writing and uploading, a final diary entry appears confirming success or failure.

Subject to `OCAP_settings_minMissionTime` (default: 20 min). If the recording is too short, export is skipped and a diary entry explains why. To bypass this limit, use the `OCAP_exportData` CBA server event instead (see Section 3).

#### Section 3: Manual Recording Control (CBA Server Events)

Three CBA server events for programmatic or admin-driven control. All require the caller to be a server admin or listed in `OCAP_administratorList`.

| Event | Effect |
|-------|--------|
| `["OCAP_record"] call CBA_fnc_serverEvent;` | Start or resume recording |
| `["OCAP_pause"] call CBA_fnc_serverEvent;` | Pause recording (data is preserved; resume with `OCAP_record`) |
| `["OCAP_exportData", [side, message, tag]] call CBA_fnc_serverEvent;` | Stop and save — **always bypasses `minMissionTime`** |

The `OCAP_exportData` event takes the same optional parameters as `ocap_fnc_exportData`:
```sqf
// Save with side and message, bypassing minimum duration
["OCAP_exportData", [west, "BLUFOR completed the objectives!", "TvT"]] call CBA_fnc_serverEvent;
```

When to use CBA events vs. `ocap_fnc_exportData`:
- Use `ocap_fnc_exportData` in normal mission triggers — it respects minimum duration and is the intended mission-end path.
- Use `OCAP_exportData` CBA event when you need to force-save regardless of duration (e.g. server restart, admin manual export).
- Use `OCAP_record` / `OCAP_pause` to gate recording around specific mission phases (e.g. don't record during briefing or post-mission lobby).

#### Section 4: Custom Events

Custom events appear in the playback UI's Events tab and on the timeline. Fire them from any server-side script using the `OCAP_customEvent` CBA event.

**General event** — freeform text entry in the event log:
```sqf
["OCAP_customEvent", ["generalEvent", "Player reached the extraction zone"]] call CBA_fnc_serverEvent;
```

**Sector captured** — structured event with sector name, owning side, and position:
```sqf
["OCAP_customEvent", ["captured", ["sector", "Sector Alpha", str west, getPosASL _sector]]] call CBA_fnc_localEvent;
```

**Sector contested** — sector is being fought over (no owning side):
```sqf
["OCAP_customEvent", ["contested", ["sector", "Sector Alpha", "", getPosASL _sector]]] call CBA_fnc_localEvent;
```

**End mission** — log a mission-end message as a custom event (separate from `ocap_fnc_exportData`):
```sqf
// With side
["OCAP_customEvent", ["endMission", [str west, "BLUFOR controlled all sectors!"]]] call CBA_fnc_localEvent;

// Message only
["OCAP_customEvent", ["endMission", "Mission complete!"]] call CBA_fnc_localEvent;
```

> See the [OCAP wiki](https://github.com/OCAP2/OCAP/wiki/Custom-Game-Events) for additional custom event documentation.

#### Section 5: Score Counters

Track arbitrary scores or ticket counts for up to two sides. Values appear as a live counter in the playback UI.

Initialize at mission start:
```sqf
["OCAP_counterInit", [
    [west, 0],
    [east, 0]
]] call CBA_fnc_serverEvent;
```

Update whenever the score changes:
```sqf
// Set BLUFOR score to 3
["OCAP_counterEvent", [west, 3]] call CBA_fnc_serverEvent;
```

This system is separate from `BIS_fnc_respawnTickets`. Wire it to whatever scoring logic you use.

#### Section 6: Focus Windows

Mark a sub-range of the recording for highlighted playback. Useful for drawing attention to the key phase of a mission.

Set the in-point at the current frame:
```sqf
["OCAP_setFocusStart"] call CBA_fnc_serverEvent;
```

Set the out-point at the current frame:
```sqf
["OCAP_setFocusEnd"] call CBA_fnc_serverEvent;
```

Or specify an explicit frame number:
```sqf
["OCAP_setFocusStart", [120]] call CBA_fnc_serverEvent;
["OCAP_setFocusEnd", [850]] call CBA_fnc_serverEvent;
```

In the playback UI, the focused range is visually highlighted on the timeline. The I and O keyboard shortcuts can also set the focus range interactively while watching a recording.

#### Section 7: Admin Controls (In-Game Diary)

Players who are server admins or whose Steam UID is listed in `OCAP_administratorList` get an **OCAP Admin** tab in their briefing diary. It contains three clickable links:

- **Start/Resume Recording** — fires `OCAP_record`
- **Pause Recording** — fires `OCAP_pause`
- **Stop and Export Recording** — fires `OCAP_exportData`, bypassing minimum duration

The tab appears automatically on player connect and is removed if a player logs out of admin. No mission-side setup is required beyond setting `OCAP_administratorList` in CBA settings.

---

### Part 2 — Playback UI

#### Section 8: Finding a Recording

Navigate to the web server address in any browser. The recording selector shows all uploaded missions with:
- Mission name
- Upload date
- Tag (set via `ocap_fnc_exportData` or `OCAP_settings_saveTag`)

Use the search box to filter by mission name. Use the tag dropdown and date pickers to narrow results. Click any recording to open it in the playback view.

#### Section 9: Playback Controls

The bottom bar contains the main playback controls:
- **Play/Pause button** — starts or stops playback
- **Timeline scrubber** — click or drag to jump to any point in the recording; the focused range (if set) is highlighted

#### Section 10: Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Space` | Toggle play/pause |
| `E` | Toggle side panel visibility |
| `←` / `→` | Step back / forward 1 frame |
| `Shift+←` / `Shift+→` | Step back / forward 10 frames |
| `,` | Jump to previous kill event |
| `.` | Jump to next kill event |
| `I` | Set focus in-point (when in focus edit mode) |
| `O` | Set focus out-point (when in focus edit mode) |
| `Escape` | Cancel focus edit |

#### Section 11: Following a Unit

Click any unit icon on the map to lock the camera to that unit. The map will pan to keep the unit centered as the recording plays. A follow indicator appears in the UI. Click the map background or click a different unit to change or release the follow lock.

#### Section 12: Side Panel Tabs

The side panel on the left has four tabs:

**Units** — all players, AI, and vehicles in the mission. Shows alive/dead state, group name, and slotted role. Click a unit in this list to follow them on the map.

**Events** — a chronological, filterable log of game events: kills, deaths, hits, player connects/disconnects, and any custom events fired via `OCAP_customEvent`. Click an event to jump to that frame in the recording. Use the filter controls to show or hide event types (e.g. hide hits to reduce noise).

**Stats** — kill counts and summary statistics for the session.

**Chat** — in-game chat messages recorded during the session, with timestamps.

Press `E` to hide the panel and maximize the map view.

#### Section 13: Focus Mode

If the mission used `OCAP_setFocusStart` / `OCAP_setFocusEnd`, the focused time range is highlighted on the timeline scrubber. You can also set the focus range interactively while watching:

1. Click the focus toolbar button to enter focus edit mode.
2. Navigate to the desired start point, then press `I`.
3. Navigate to the desired end point, then press `O`.
4. Press `Escape` to cancel without saving.

The focus range is visual only in the playback UI — it does not trim the recording.

#### Section 14: Counter Display

If the mission used `OCAP_counterInit`, a score counter appears on-screen during playback, showing the tracked values for each side as they changed throughout the mission.

---

## Constraints

- Do not document Arma 3 mission editing from scratch — assume the reader knows how triggers and server-side script execution work.
- All CBA event examples must be server-side (`CBA_fnc_serverEvent`) unless the source code shows `localEvent` is correct (e.g. sector events).
- Do not invent UI elements or shortcuts not present in `web/ui/src/pages/recording-playback/shortcuts.ts` and the component files.
- Link to `docs/configuration.md` for `OCAP_settings_*` settings rather than repeating their descriptions.
- Link to the OCAP wiki for custom game events extended documentation.
