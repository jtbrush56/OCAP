# Configuration Documentation Design

**Date:** 2026-05-08
**Output file:** `docs/configuration.md`

## Goal

Create a single reference document that explains all configuration files for OCAP — what they are, where they go, and what every setting does. Targets both first-time setup and day-to-day lookup.

## Structure

### Section 1 — File Placement Summary

A table with three rows (addon, extension, web server) showing:
- Config file name
- Where to place it (path relative to component root)
- What it controls

Followed by a callout: the `secret` field in `setting.json` and `api.apiKey` in `ocap_recorder.cfg.json` must be set to the same value.

### Section 2 — Addon: CBA Settings

Intro line: settings are in-game at `Options > Addon Options > OCAP`. No file to edit.

Reference tables organized by CBA category:

| Category | Settings |
|----------|---------|
| Core | `OCAP_enabled`, `OCAP_isDebug`, `OCAP_administratorList` |
| Auto-start Settings | `OCAP_settings_autoStart`, `OCAP_settings_minPlayerCount` |
| Core (capture) | `OCAP_settings_frameCaptureDelay`, `OCAP_settings_preferACEUnconscious` |
| Exclusions | `OCAP_settings_excludeClassFromRecord`, `OCAP_settings_excludeKindFromRecord`, `OCAP_settings_excludeMarkerFromRecord` |
| Extra Tracking | `OCAP_settings_trackTickets`, `OCAP_settings_trackTimes`, `OCAP_settings_trackTimeInterval`, `OCAP_settings_trackSectors` |
| Save/Export Settings | `OCAP_settings_saveTag`, `OCAP_settings_saveMissionEnded`, `OCAP_settings_saveOnEmpty`, `OCAP_settings_minMissionTime` |

Each table row: setting name | type | default | description (from source).

### Section 3 — Extension: `ocap_recorder.cfg.json`

File location: alongside the DLL (`@ocap/ocap_recorder.cfg.json`). Copy from `ocap_recorder.cfg.json.example`.

Full annotated example block, then a flat reference table covering all fields:
- Top-level: `logLevel`, `logsDir`, `defaultTag`
- `api`: `serverUrl`, `apiKey`, `uploadTimeout`
- `storage.type` (values: `memory`, `sqlite`, `postgres`, `websocket`)
- `storage.memory`: `outputDir`, `compressOutput`
- `storage.sqlite`: `dumpInterval`
- `storage.postgres`: `host`, `port`, `username`, `password`, `database`

### Section 4 — Web Server: `setting.json`

File location: working directory of the web server process (same folder as `ocap-webserver` binary). Copy from `setting.json.example`.

Full annotated example block, then a flat reference table:
- Core: `listen`, `prefixURL`, `secret`, `db`, `data`, `static`, `markers`, `ammo`, `fonts`, `maps`, `logger`
- `auth`: `steamApiKey`, `adminSteamIds`, `sessionTTL`
- `conversion`: `enabled`, `batchSize`, `chunkSize`, `interval`, `retryFailed`
- `streaming`: `enabled`, `pingInterval`, `pingTimeout`
- `customize`: `enabled`, `headerTitle`, `headerSubtitle`, `websiteURL`, `websiteLogo`, `websiteLogoSize`, `disableKillCount`, `cssOverrides`
- `httpServer`: `readTimeout`, `readHeaderTimeout`, `writeTimeout`, `idleTimeout`

Note on env var overrides: any setting can be set via `OCAP_<KEY>` (nested keys concatenated, e.g. `OCAP_AUTH_STEAMAPIKEY`).

## Constraints

- Document covers only deployed config files — no build-time config, no SQF scripting API.
- Settings sourced from: `addon/addons/main/XEH_preInit.sqf`, `addon/addons/recorder/XEH_preInit.sqf`, `extension/ocap_recorder.cfg.json.example`, `web/setting.json.example`.
- No invented defaults — all defaults taken directly from source files.
