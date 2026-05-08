# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

This is a **meta-repository** that coordinates three git submodules. All actual source code lives in the submodules:

| Submodule | Repo | Language | Role |
|-----------|------|----------|------|
| `addon/` | OCAP2/addon | SQF (Arma 3 scripting) | In-game recording addon |
| `extension/` | OCAP2/extension | Go | Native extension called by Arma 3, records missions |
| `web/` | OCAP2/web | Go + SolidJS | HTTP server with playback UI and admin interface |

The root repo holds documentation, the `tiles_tut/` terrain tile creation guide, and the GitHub Actions release bundling workflow.

## Working with Submodules

Submodules are not initialized by default:

```bash
git submodule update --init --recursive   # first-time init
git submodule update --recursive          # sync to pinned SHAs
git submodule update --remote addon       # advance a submodule to remote HEAD
```

After advancing a submodule, commit the root repo to record the new SHA:
```bash
git add addon extension web
git commit -m "chore: bump submodules to ..."
```

## System Architecture

```
Arma 3 server (SQF addon)
  --callExtension--> Go extension (.dll/.so)
                       --HTTP upload--> Go web server
                                          <-- browser playback
```

- **addon**: Hooks Arma 3 event handlers (kills, markers, positions, projectiles) and sends `:RESOURCE:ACTION:` commands to the extension via `callExtension`.
- **extension**: Native DLL/SO (Go, CGo). Async dispatcher routes commands to buffered handlers gated on `:STORAGE:INIT:`. Pluggable storage backends: `memory` (JSON export), `sqlite`, `postgres`, `websocket`.
- **web**: Fuego-based HTTP API (OpenAPI/Swagger at `/swagger`). Recordings uploaded as gzipped JSON, background-converted to chunked Protobuf for efficient streaming. SolidJS+Leaflet frontend embedded in the Go binary.

---

## addon/ — SQF Addon (HEMTT)

### Build

Requires [HEMTT](https://github.com/BrettMayson/HEMTT).

```bash
hemtt check          # lint/validate
hemtt build          # dev build → .hemttout/build/
hemtt release        # signed PBOs → releases/ocap-latest.zip
```

### Structure

Three CBA-style sub-addons under `addons/`:
- `ocap_main` — macros (`script_macros.hpp`), version, CfgPatches
- `ocap_recorder` — 40+ SQF functions for capture loop, event handlers, export
- `ocap_extension` — thin wrapper for `callExtension` calls to the Go extension

The macro system uses CBA prefixes: `FUNC(name)` → `OCAP_recorder_fnc_name`, `GVAR(name)` → `OCAP_recorder_name`. All SQF files must `#include "script_component.hpp"`.

**Key guard macro:** `SHOULDSAVEEVENTS` checks both the recording flag and `startTime > -1` before any event is persisted.

Version numbers are not hardcoded — HEMTT injects them from git tags at release time into `script_version.hpp`.

---

## extension/ — Go Native Extension

> The extension has its own `CLAUDE.md` at `extension/CLAUDE.md` — check it for additional guidance.

### Build

Builds require Docker (cross-compilation via MinGW/Bullseye). There is no local build path without Docker.

```bash
# Windows DLL
docker run --rm -v ${PWD}:/go/work -w /go/work x1unix/go-mingw:1.24 \
  go build -buildvcs=false -o dist/ocap_recorder_x64.dll -buildmode=c-shared ./cmd/ocap_recorder

# Linux SO
docker run --rm -v ${PWD}:/go/work -w /go/work golang:1.24-bullseye \
  go build -buildvcs=false -o dist/ocap_recorder_x64.so -buildmode=c-shared ./cmd/ocap_recorder
```

### Test

```bash
go test ./internal/...
```

### Key Packages

| Package | Role |
|---------|------|
| `pkg/a3interface/` | CGo `RVExtension*` exports; panic recovery to prevent game crashes |
| `pkg/core/` | Storage-agnostic domain types (Soldier, Vehicle, Marker, events) |
| `internal/dispatcher/` | Async event router; sync handlers for entity creation, buffered channels for state/kills/telemetry |
| `internal/parser/` | Parses raw `:RESOURCE:ACTION:` strings into core types |
| `internal/worker/` | Handler registration, DB writer goroutine (batch writes every 2s) |
| `internal/storage/` | Backend implementations: memory, postgres, sqlite, websocket |
| `internal/cache/` | ObjectID → model lookup cache |

**Startup gating:** Buffered handlers don't process until `:STORAGE:INIT:` is received, preventing writes before the DB is ready. Events queue safely in channels during init.

### Configuration

`ocap_recorder.cfg.json` (placed alongside the DLL): storage type, DB credentials, logging level, output directories, API upload URL.

---

## web/ — Go Web Server + SolidJS Frontend

### Build

Frontend must be built before the Go binary embeds it:

```bash
cd ui && npm ci && npm run build   # outputs to internal/frontend/dist/
go build -o ocap-webserver ./cmd/ocap-webserver
```

Docker multi-stage build handles this automatically.

### Test

```bash
go test ./...                  # Go backend
cd ui && npm test              # frontend (Vitest)
cd ui && npm run test:coverage
```

### Dev Server

```bash
# Terminal 1
go run ./cmd/ocap-webserver

# Terminal 2 — Vite dev server with HMR, proxies /api/* to :5000
cd ui && npm run dev           # visit localhost:5173
```

Set `OCAP_STATIC` env var to override the embedded frontend at runtime.

### Key Packages

| Package | Role |
|---------|------|
| `internal/server/` | HTTP handlers, repos (operation/marker/ammo), JWT auth, Steam OpenID |
| `internal/storage/` | JSON.gz reader, chunked Protobuf reader/writer, streaming converter |
| `internal/conversion/` | Background worker converting uploaded JSON → Protobuf chunks |
| `internal/maptool/` | GDAL/tippecanoe/pmtiles pipeline for map tile processing |
| `pkg/schemas/protobuf/v1/` | `.proto` schema + generated Go and TypeScript types |
| `ui/src/playback/` | Playback engine (entities, events) |
| `ui/src/renderers/` | Leaflet + Canvas map renderers |

**Protobuf regen** (after editing `ocap.proto`):
```bash
go generate ./pkg/schemas/...
cd ui && npx protoc --ts_proto_out=./src/generated ... ../pkg/schemas/protobuf/v1/ocap.proto
```

### Configuration

`setting.json` in the working directory, or `OCAP_*` env vars (nested keys use concatenation: `auth.sessionTTL` → `OCAP_AUTH_SESSIONTTL`). Default listen address: `127.0.0.1:5000`.

### Non-Obvious Details

- Recordings upload as JSON.gz → background worker converts to chunked Protobuf (~300 frames/chunk). Browser loads chunks on-demand, caching in OPFS/IndexedDB.
- **slim** Docker image: web server only. **full** image: includes GDAL + tippecanoe for map tile processing.
- Pelican Panel support: `docker/entrypoint.sh` detects Pelican/Wings via `STARTUP` env var for variable substitution.

---

## Release Workflow

`.github/workflows/release.yml` fires on a published GitHub Release. It resolves each submodule's pinned SHA to a matching release tag in the respective component repo, downloads pre-built assets, and assembles `windows.zip` and `linux.tar.gz`. **No compilation happens in this repo** — all builds run in component repo CI.

## Deployed Configuration Files (not tracked here)

- `web/setting.json` — public IP, port, Steam API key, auth secret
- `addon/addons/@ocap/OcapReplaySaver2.cfg.json` — extension upload target (IP/port/secret)
- `addon/userconfig/config.hpp` — recording thresholds, frame rate, exclusion lists
