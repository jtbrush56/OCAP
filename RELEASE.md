# OCAP 2.1.0 Release Notes

## Overview

OCAP 2.1.0 is a major release that addresses critical stability issues and introduces significant architectural improvements across all components. This release focuses on solving browser crashes from large recordings, complete extension rewrite, and enhanced deployment options.

## Critical Fixes

### Browser Crash Resolution - Chunked Streaming

The most impactful fix in this release is the implementation of **chunked streaming** for large recordings. Previously, loading recordings with thousands of frames would cause browser crashes due to memory exhaustion.

**How it works:**
1. **Upload:** Mission ends → JSON.gz uploaded → Stored in `data/` directory
2. **Conversion:** Background worker converts to chunked binary format:
   ```
   data/mission_name.gz  →  data/mission_name/
                             ├── manifest.pb (metadata + entities)
                             └── chunks/
                                   ├── 0000.pb (frames 0-299)
                                   ├── 0001.pb (frames 300-599)
                                   └── ...
   ```
3. **Playback:** Only loads chunks as needed, caches in browser storage (OPFS/IndexedDB)

**Benefits:**
- Peak memory usage reduced by up to 95% for large recordings
- Enables smooth playback of multi-hour missions with full detail
- Old recordings can be converted using CLI: `./ocap-webserver convert --all`

## Component Changes

### Extension (v5.0.0) - Complete C++ to Go Rewrite

The ArmA 3 extension has been completely rewritten from C++ to Go, bringing significant improvements:

**Architecture Changes:**
- Pluggable storage backend system (SQLite, memory, WebSocket streaming)
- Event dispatcher replacing channel-based architecture
- OpenTelemetry integration for structured logging (replaced zerolog)
- Panic recovery in all DLL entry points for improved stability

**New Configuration Format:**

The new JSON-based configuration provides better organization and extensibility:

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
  "db": {
    "host": "127.0.0.1",
    "port": "5432",
    "username": "postgres",
    "password": "postgres",
    "database": "ocap"
  },
  "storage": {
    "type": "memory",
    "memory": {
      "outputDir": "./recordings",
      "compressOutput": true
    },
    "sqlite": {
      "dumpInterval": "3m"
    }
  }
}
```

**Old C++ Extension Config (v4.4.2.3):**
```json
{
    "httpRequestTimeout": 120,
    "logAndTmpPrefix": "ocap-",
    "logsDir": "./OCAPLOG",
    "newServerGameType": "TvT",
    "newUrl": "https://127.0.0.1/api/v1/operations/add",
    "newUrlRequestSecret": "pwd1234",
    "traceLog": 0
}
```

**Key Differences:**
- New config separates concerns (API, database, storage)
- Supports pluggable storage backends (memory, SQLite)
- Includes OpenTelemetry configuration
- Better defaults and validation

**New Features:**
- POLYLINE marker support
- Projectile marker support for grenades and smokes
- Structured weapon arrays with derived hit events
- Dynamic groupID and side tracking per unit state update
- WebSocket streaming storage backend for real-time processing

**Note:** The recording JSON format remains **v1** - completely unchanged. The extension and web server are fully backward compatible.

### Addon (v2.1.0)

**Major Changes:**
- Modular architecture with database export support
- Projectile tracking overhaul with lightweight array format
- ACE explosives refactored to use Explode event handler
- Vehicle tracking improvements (ejection seats, disappeared vehicles, parachute lifetime)

**Bug Fixes:**
- Fixed MarkerUpdated exclusion check that was excluding all markers
- Fixed callExtension errors by passing empty array instead of nil
- Fixed marker side format to use string instead of numeric ID
- Fixed re-recording after export losing data (missing NEW:MISSION)
- Fixed vehicle weapon tracking and projectile system
- Fixed global toFixed calls interfering with TFAR radios
- Fixed adminUIcontrol race condition during postInit
- Fixed diary status never updating past initial values
- Fixed PFH interval variables scoping issues
- Fixed kill weapon attribution with HandleDamage for explosives

**Compatibility:**
- Compatibility with Restrict Markers mod
- Support for onLoadName for mission names
- Manual capture start option added

### Web (v2.1.0)

**Performance Improvements:**
- **Chunked streaming** for large recording playback
- Canvas-based entity renderer replacing Leaflet DOM markers
- Render all firelines per frame instead of only the first
- Binary search optimization for kill frame navigation

**New Features:**
- Admin UI with authentication, CRUD operations, and upload
- Steam profile display and role-based authentication (admin/viewer)
- Activity heatmap timeline with vertical playhead
- Admin-defined focus range for recordings
- Vehicle kills column in scoreboard leaderboard
- ViewSettings panel with decluttered bottom bar
- Marker display mode dropdown (including noLabels option)

**Rendering Improvements:**
- Apply marker type/color/brush changes during playback
- Match Arma 3 coordinate grid pattern
- Eliminate grid wobble during zoom animation
- Contextual hover tooltip for timeline heatmap
- Redesigned player transport controls

**Internationalization:**
- Ukrainian (українська) UI translation
- Finnish (Suomi) UI translation
- Italian translation
- Czech localization
- Internationalized map manager UI

**Architecture:**
- Migrated from Echo v4 to Fuego framework with OpenAPI/Swagger support
- Refactored to standard Go project layout
- Runtime base path for serving behind URI prefix

## Backward Compatibility

**Recording Format:**
- The recording JSON format remains **v1** - completely unchanged
- All existing recordings are fully compatible
- Web server is 100% backward compatible with old recordings

## Docker Deployment

Starting with 2.1.0, the web server offers two Docker image variants:

| Variant | Tag | Description |
|---------|-----|-------------|
| **Slim** (default) | `latest`, `v2.1.0` | Web server only |
| **Full** | `full`, `v2.1.0-full` | Web server + integrated Map Manager (GDAL, tippecanoe, pmtiles) |

The **full** variant includes all tools needed for the Map Manager — an admin page that processes Arma 3 map data (grad_meh exports) into PMTiles and MapLibre styles directly from the web UI. The server auto-detects the available tools at startup; no extra configuration is needed.

**Slim example:**
```bash
docker run --name ocap-web -d \
  -p 5000:5000/tcp \
  -e OCAP_SECRET="same-secret" \
  -e OCAP_CONVERSION_ENABLED="true" \
  -v ocap-records:/var/lib/ocap/data \
  -v ocap-maps:/var/lib/ocap/maps \
  -v ocap-database:/var/lib/ocap/db \
  ghcr.io/ocap2/web:latest
```

**Full example:**
```bash
docker run --name ocap-web -d \
  -p 5000:5000/tcp \
  -e OCAP_SECRET="same-secret" \
  -e OCAP_CONVERSION_ENABLED="true" \
  -v ocap-records:/var/lib/ocap/data \
  -v ocap-maps:/var/lib/ocap/maps \
  -v ocap-database:/var/lib/ocap/db \
  ghcr.io/ocap2/web:full
```

See [README.md](https://github.com/OCAP2/web/blob/main/README.md) for detailed Docker configuration and usage instructions.

## Upgrade Notes

1. **Extension Configuration:** If you're upgrading from extension v4.x, review the new configuration format - the structure has changed significantly
2. **Docker Tags:** Update your Docker deployment to use either `latest` (slim) or `full` tag based on your needs
3. **No Recording Migration Needed:** All existing recordings work without any conversion - the web server is 100% backward compatible
4. **Optional Conversion:** For better performance with large recordings, use `./ocap-webserver convert --all` to convert existing JSON recordings to chunked binary format

## Contributors

This release includes contributions from:
- @fank - Core architecture, extension rewrite, web improvements
- @veteran29 - addon improvements, translations
- @Mike-MF - Manual capture, onLoadName support
- @mrschick - Restrict Markers compatibility
- @P4sca1 - Documentation
- @Fratee - Italian translation
- @YetheSamartaka - Czech translation
- @Exirium - Regional time display
- @galevsky - JSON.gz download feature
- @Morriel1 - Ukrainian translation

## Download

- **Addon:** https://github.com/OCAP2/addon/releases/tag/v2.1.0
- **Extension:** https://github.com/OCAP2/extension/releases/tag/v5.0.0
- **Web:** https://github.com/OCAP2/web/releases/tag/v2.1.0
