# Configuration Documentation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Write `docs/configuration.md` — a complete configuration reference for all three OCAP components (addon, extension, web server).

**Architecture:** Single markdown file with a file-placement summary at the top, followed by one section per component. Each component section has a brief "where to put it" note and a full reference table with every setting, its type, default, and description sourced directly from the config files.

**Tech Stack:** Markdown only. No code changes. Sources: `addon/addons/main/XEH_preInit.sqf`, `addon/addons/recorder/XEH_preInit.sqf`, `extension/ocap_recorder.cfg.json.example`, `web/setting.json.example`.

---

### Task 1: Write `docs/configuration.md`

**Files:**
- Create: `docs/configuration.md`

- [ ] **Step 1: Create the file with the file-placement summary**

Create `docs/configuration.md` with:

```markdown
# Configuration Reference

OCAP has three components that each need configuration. This document covers all settings, their defaults, and where each file lives.

## File Placement

| Component | File | Location |
|-----------|------|----------|
| Addon | *(CBA Settings — no file)* | In-game: Options → Addon Options → OCAP |
| Extension | `ocap_recorder.cfg.json` | Alongside the DLL, inside `@ocap/` |
| Web server | `setting.json` | Working directory of the `ocap-webserver` binary |

> **Important:** `setting.json` → `secret` and `ocap_recorder.cfg.json` → `api.apiKey` must be set to the same value, or the extension will fail to upload recordings.

---
```

- [ ] **Step 2: Add the Addon / CBA Settings section**

Append to `docs/configuration.md`:

```markdown
## Addon — CBA Settings

All addon settings are configured in-game. No file to edit.

**Location:** Options → Addon Options → OCAP

Settings are server-side and synchronised to all clients automatically.

### Core

| Setting (variable name) | Type | Default | Description |
|-------------------------|------|---------|-------------|
| Recording Enabled (`OCAP_enabled`) | Boolean | `true` | Master toggle. Stops recording new data without resetting the current session. To pause/resume mid-session use the CBA events instead. |
| Debug Mode (`OCAP_isDebug`) | Boolean | `false` | Enables increased log output from the addon. |
| Administrators (`OCAP_administratorList`) | Stringified array | `[]` | Player UIDs with access to extra briefing diary controls. Example: `"['76561198000000000']"`. Can also reference a server-visible variable name. |

### Auto-start Settings

| Setting (variable name) | Type | Default | Description |
|-------------------------|------|---------|-------------|
| Auto Start Recording (`OCAP_settings_autoStart`) | Boolean | `true` | Start recording automatically when the minimum player count is reached at mission start. Requires restart to apply. |
| Minimum Player Count (`OCAP_settings_minPlayerCount`) | Number (1–150) | `15` | Player count required to trigger auto-start. |

### Capture

| Setting (variable name) | Type | Default | Description |
|-------------------------|------|---------|-------------|
| Frame Capture Delay (`OCAP_settings_frameCaptureDelay`) | Number (0.10–10.00) | `1` | Seconds between each position/state snapshot of all units and vehicles. Requires restart to apply. |
| Use ACE3 Medical (`OCAP_settings_preferACEUnconscious`) | Boolean | `true` | Track ACE3 unconscious state when ACE3 is loaded. Falls back to vanilla revive system if false or ACE3 is absent. |

### Exclusions

| Setting (variable name) | Type | Default | Description |
|-------------------------|------|---------|-------------|
| Classnames to Exclude (`OCAP_settings_excludeClassFromRecord`) | Stringified array | `"['ACE_friesAnchorBar','WeaponHolderSimulated']"` | Exact object classnames to skip. Use single quotes inside the string. |
| Object KindOfs to Exclude (`OCAP_settings_excludeKindFromRecord`) | Stringified array | `"['WeaponHolder']"` | Classnames where this class and all child classes are excluded. Use single quotes inside the string. |
| Marker Prefixes to Exclude (`OCAP_settings_excludeMarkerFromRecord`) | Stringified array | `"['SystemMarker_','ACE_BFT_','RscAttribute']"` | Markers whose names start with any of these prefixes are not recorded. Use single quotes inside the string. |

### Extra Tracking

| Setting (variable name) | Type | Default | Description |
|-------------------------|------|---------|-------------|
| Enable Ticket Tracking (`OCAP_settings_trackTickets`) | Boolean | `true` | Records respawn ticket counts for all playable factions every 30 frames. |
| Enable Mission Time Tracking (`OCAP_settings_trackTimes`) | Boolean | `false` | Continuously records in-game world time. Enable when missions use time acceleration or time skips. |
| Mission Time Tracking Interval (`OCAP_settings_trackTimeInterval`) | Number (5–25) | `10` | How many capture frames between time snapshots. Only used when time tracking is enabled. |
| Enable Sector Capture Tracking (`OCAP_settings_trackSectors`) | Boolean | `true` | Records ownership changes on `ModuleSector_F` objects. |

### Save / Export Settings

| Setting (variable name) | Type | Default | Description |
|-------------------------|------|---------|-------------|
| Mission Type Tag (`OCAP_settings_saveTag`) | String | `"TvT"` | Tag applied to recordings in the web UI. Can be overridden per-save via the `ocap_exportData` CBA event. |
| Auto-save on MPEnded Event (`OCAP_settings_saveMissionEnded`) | Boolean | `true` | Automatically save and upload when the `MPEnded` mission event fires. |
| Auto-Save When No Players (`OCAP_settings_saveOnEmpty`) | Boolean | `true` | Save automatically when the server empties, provided the recording meets the minimum duration. |
| Required Duration to Save (`OCAP_settings_minMissionTime`) | Number (1–120 min) | `20` | Recordings shorter than this many minutes are not auto-saved. Calling `ocap_exportData` directly bypasses this limit. Requires restart to apply. |

---
```

- [ ] **Step 3: Add the Extension config section**

Append to `docs/configuration.md`:

```markdown
## Extension — `ocap_recorder.cfg.json`

Copy `ocap_recorder.cfg.json.example` (found inside `@ocap/`) to `ocap_recorder.cfg.json` in the same directory and edit as needed.

```json
{
  "logLevel": "info",
  "logsDir": "./ocaplogs",
  "defaultTag": "TvT",
  "api": {
    "serverUrl": "http://127.0.0.1:5000",
    "apiKey": "same-secret",
    "uploadTimeout": "10m"
  },
  "storage": {
    "type": "memory",
    "memory": {
      "outputDir": "./recordings",
      "compressOutput": true
    },
    "sqlite": {
      "dumpInterval": "3m"
    },
    "postgres": {
      "host": "127.0.0.1",
      "port": "5432",
      "username": "postgres",
      "password": "postgres",
      "database": "ocap"
    }
  }
}
```

### Top-level

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `logLevel` | String | `"info"` | Log verbosity. One of `debug`, `info`, `warn`, `error`. |
| `logsDir` | String | `"./ocaplogs"` | Directory where log files are written, relative to the DLL. |
| `defaultTag` | String | `"TvT"` | Fallback tag applied to recordings when the addon does not supply one. |

### `api`

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `api.serverUrl` | String | `"http://127.0.0.1:5000"` | Base URL of the OCAP web server. |
| `api.apiKey` | String | `"same-secret"` | Must match `secret` in `setting.json`. |
| `api.uploadTimeout` | Duration string | `"10m"` | HTTP timeout for uploading a completed recording. |

### `storage`

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `storage.type` | String | `"memory"` | Storage backend. One of `memory`, `sqlite`, `postgres`, `websocket`. |

**`memory` backend** — records entirely in RAM and exports a gzipped JSON file at mission end.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `storage.memory.outputDir` | String | `"./recordings"` | Directory for exported JSON files. |
| `storage.memory.compressOutput` | Boolean | `true` | Gzip the exported file. |

**`sqlite` backend** — records to an in-memory SQLite database and periodically dumps to disk.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `storage.sqlite.dumpInterval` | Duration string | `"3m"` | How often to flush the in-memory DB to disk. |

**`postgres` backend** — writes directly to a PostgreSQL database.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `storage.postgres.host` | String | `"127.0.0.1"` | PostgreSQL host. |
| `storage.postgres.port` | String | `"5432"` | PostgreSQL port. |
| `storage.postgres.username` | String | `"postgres"` | Database user. |
| `storage.postgres.password` | String | `"postgres"` | Database password. |
| `storage.postgres.database` | String | `"ocap"` | Database name. |

**`websocket` backend** — streams events in real time to the OCAP web server. No additional fields.

---
```

- [ ] **Step 4: Add the Web Server config section**

Append to `docs/configuration.md`:

```markdown
## Web Server — `setting.json`

Copy `setting.json.example` to `setting.json` in the same directory as the `ocap-webserver` binary and edit as needed.

Any field can also be set via an environment variable using the prefix `OCAP_` with nested keys concatenated in uppercase (e.g. `auth.steamApiKey` → `OCAP_AUTH_STEAMAPIKEY`). Environment variables take precedence over `setting.json`.

```json
{
  "listen": "127.0.0.1:5000",
  "prefixURL": "",
  "secret": "change-me",
  "db": "data.db",
  "data": "data",
  "static": "",
  "markers": "assets/markers",
  "ammo": "assets/ammo",
  "fonts": "assets/fonts",
  "maps": "maps",
  "logger": false,
  "auth": {
    "steamApiKey": "",
    "adminSteamIds": [],
    "sessionTTL": "24h"
  },
  "conversion": {
    "enabled": false,
    "batchSize": 1,
    "chunkSize": 300,
    "interval": "5m",
    "retryFailed": false
  },
  "streaming": {
    "enabled": false,
    "pingInterval": "30s",
    "pingTimeout": "10s"
  },
  "customize": {
    "enabled": false,
    "headerTitle": "",
    "headerSubtitle": "",
    "websiteURL": "",
    "websiteLogo": "",
    "websiteLogoSize": "32px",
    "disableKillCount": false,
    "cssOverrides": {}
  },
  "httpServer": {
    "readTimeout": "120s",
    "readHeaderTimeout": "30s",
    "writeTimeout": "120s",
    "idleTimeout": "120s"
  }
}
```

### Core

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `listen` | String | `"127.0.0.1:5000"` | Address and port the server binds to. Use `0.0.0.0:5000` to listen on all interfaces. |
| `prefixURL` | String | `""` | URL prefix for all routes, e.g. `"/ocap"` when running behind a reverse proxy at a subpath. |
| `secret` | String | `"change-me"` | Shared secret for recording uploads. Must match `api.apiKey` in the extension config. |
| `db` | String | `"data.db"` | Path to the SQLite database file for mission metadata. |
| `data` | String | `"data"` | Directory where uploaded recording files are stored. |
| `static` | String | `""` | Override the embedded frontend with files from this directory. Leave empty to use the built-in UI. |
| `markers` | String | `"assets/markers"` | Directory containing marker SVG assets. |
| `ammo` | String | `"assets/ammo"` | Directory containing ammo/equipment icon assets. |
| `fonts` | String | `"assets/fonts"` | Directory containing font assets. |
| `maps` | String | `"maps"` | Directory containing terrain tile folders. |
| `logger` | Boolean | `false` | Enable HTTP request logging. |

### `auth`

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `auth.steamApiKey` | String | `""` | Steam Web API key used for admin login via Steam OpenID. Required to use the admin UI. Obtain one at https://steamcommunity.com/dev/apikey. |
| `auth.adminSteamIds` | Array of strings | `[]` | Steam 64-bit IDs (as strings) that are granted admin access. |
| `auth.sessionTTL` | Duration string | `"24h"` | How long an admin session remains valid before requiring re-login. |

### `conversion`

Background worker that converts uploaded JSON recordings to chunked Protobuf format for faster browser streaming.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `conversion.enabled` | Boolean | `false` | Enable the background conversion worker. |
| `conversion.batchSize` | Number | `1` | Number of recordings to convert per worker run. |
| `conversion.chunkSize` | Number | `300` | Frames per Protobuf chunk (~5 minutes at 1 fps). |
| `conversion.interval` | Duration string | `"5m"` | How often the worker checks for unconverted recordings. |
| `conversion.retryFailed` | Boolean | `false` | Re-attempt previously failed conversions. |

### `streaming`

WebSocket-based live streaming of recording data from the extension.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `streaming.enabled` | Boolean | `false` | Accept WebSocket connections from the extension's `websocket` storage backend. |
| `streaming.pingInterval` | Duration string | `"30s"` | How often to send a WebSocket ping to connected extensions. |
| `streaming.pingTimeout` | Duration string | `"10s"` | How long to wait for a pong before closing the connection. |

### `customize`

Optional UI branding overrides shown to all visitors.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `customize.enabled` | Boolean | `false` | Enable customization. Fields below are ignored when false. |
| `customize.headerTitle` | String | `""` | Text shown in the page header. |
| `customize.headerSubtitle` | String | `""` | Subtitle shown beneath the header title. |
| `customize.websiteURL` | String | `""` | URL the header logo/title links to. |
| `customize.websiteLogo` | String | `""` | URL to a logo image shown in the header. |
| `customize.websiteLogoSize` | String | `"32px"` | CSS size of the header logo. |
| `customize.disableKillCount` | Boolean | `false` | Hide kill count information during playback. |
| `customize.cssOverrides` | Object | `{}` | Key/value pairs of CSS custom properties to override. |

### `httpServer`

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `httpServer.readTimeout` | Duration string | `"120s"` | Maximum time to read an entire request. |
| `httpServer.readHeaderTimeout` | Duration string | `"30s"` | Maximum time to read request headers. |
| `httpServer.writeTimeout` | Duration string | `"120s"` | Maximum time to write a response. |
| `httpServer.idleTimeout` | Duration string | `"120s"` | Maximum time to wait for the next request on a keep-alive connection. |
```

- [ ] **Step 5: Verify coverage against source files**

Check that every setting in the source files appears in the doc:

```bash
# Addon settings — count CBA settings in source vs doc
grep -c "QGVAR\|QEGVAR" /home/jacob/OCAP/addon/addons/main/XEH_preInit.sqf
grep -c "QGVAR\|QEGVAR" /home/jacob/OCAP/addon/addons/recorder/XEH_preInit.sqf

# Extension — verify all top-level keys in example appear in doc
cat /home/jacob/OCAP/extension/ocap_recorder.cfg.json.example

# Web — verify all top-level keys in example appear in doc
cat /home/jacob/OCAP/web/setting.json.example
```

Expected: every field seen in the example files has a row in the reference tables.

- [ ] **Step 6: Commit**

```bash
git add docs/configuration.md
git commit -m "docs: add configuration reference for addon, extension, and web server"
```
