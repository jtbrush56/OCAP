# Installation Documentation Design

**Date:** 2026-05-08
**Output file:** `docs/installation.md`

## Goal

Create a single installation reference for OCAP that covers all deployment methods. Targets general server operators who know their way around servers but may not be familiar with Arma 3-specific conventions.

## Audience Assumptions

- Comfortable running commands and editing config files
- May not know Arma 3 server concepts (serverMod, bikey, RPT log, BattlEye)
- May be choosing between bare metal, Docker, or a game server panel

## Structure

### Section 1 — Overview

What gets installed and why both components are always required:

- The release bundles everything into one `@ocap/` folder: the **addon** (SQF PBOs, game event hooks) and the **extension** (the `ocap_recorder_x64.dll` or `.so` recording backend). They are installed as a unit.
- The **web server** receives uploaded recordings and serves the playback UI. It can run on a different machine than the Arma 3 server.
- Neither component is optional: the addon without the extension has nowhere to store data; the web server without the addon receives nothing.

Simple text diagram:

```
Arma 3 server
  └── @ocap/ (addon PBOs + extension DLL/SO)
        └── HTTP upload ──► web server ◄── browser playback
```

### Section 2 — Downloads

- Source: GitHub Releases on the OCAP2/OCAP repository
- Artifacts:
  - `windows.zip` — `@ocap/` (PBOs + `ocap_recorder_x64.dll`) + `web/` (`ocap-webserver.exe` + assets)
  - `linux.tar.gz` — `@ocap/` (PBOs + `ocap_recorder_x64.so`) + `web/` (`ocap-webserver` binary + assets)
- Docker images (web server only):
  - `ghcr.io/ocap2/web:latest` — slim, web server only
  - `ghcr.io/ocap2/web:full` — includes GDAL, tippecanoe, pmtiles for server-side map tile processing

### Section 3 — Arma 3 Server: Addon + Extension

Applies regardless of web server deployment method. Numbered steps:

1. Extract the release archive and locate the `@ocap/` folder.
2. Copy `@ocap/` to the server's mods directory (same place as other server mods, e.g. next to `@CBA_A3`).
3. Copy all `.bikey` files from `@ocap/keys/` into the server root's `keys/` directory. Brief explanation: Arma 3 signature verification requires public keys in `keys/` or the mod will be blocked.
4. Add `@ocap` to the `-serverMod` startup parameter. Explain `-serverMod` vs `-mod`: `-serverMod` loads the mod only on the server; clients do not need to install or download anything. Example:
   ```
   -serverMod="@CBA_A3;@ocap"
   ```
5. Configure the extension: copy `@ocap/ocap_recorder.cfg.json.example` to `@ocap/ocap_recorder.cfg.json`. Set `api.serverUrl` to the web server's address and `api.apiKey` to a shared secret. Link to `docs/configuration.md` for all options.
6. Start the server. Verify by checking the RPT log for `OCAP` initialization messages.
7. BattlEye note: if BattlEye is enabled, add the extension to the BattlEye whitelist (`bans.txt` or equivalent). Without this, `callExtension` calls will be blocked.

### Section 4 — Web Server: Bare Metal / Binary

1. Extract the `web/` directory from the release archive.
2. Copy `setting.json.example` to `setting.json`. Set `secret` to the same value as `api.apiKey` in the extension config.
3. Run `./ocap-webserver` (Linux) or `ocap-webserver.exe` (Windows). Default port: 5000.
4. Example `systemd` unit file for Linux persistent service.
5. Firewall note: open port 5000 (or your configured port) so the Arma 3 server can upload recordings and clients can access the web UI.
6. Link to `docs/configuration.md` for all `setting.json` options.

### Section 5 — Web Server: Docker Compose

Full working `docker-compose.yml` example:
- Image: `ghcr.io/ocap2/web:latest` (or `:full` for map tools)
- Named volumes: `ocap_db`, `ocap_data`, `ocap_maps`
- Key environment variables: `OCAP_SECRET`, `OCAP_LISTEN`
- Port mapping: `5000:5000`
- `restart: unless-stopped`

Commands: `docker compose up -d`, `docker compose logs -f`.

Note on `:full` vs `:latest`: use `:full` if you want to process map terrain tiles through the admin UI; `:latest` otherwise.

### Section 6 — Web Server: Pterodactyl / Pelican

Two eggs are included in the web server release:
- `egg-ocap2-web.json` — Pelican Panel (PLCN_v3 format)
- `egg-ocap2-web-pterodactyl.json` — Pterodactyl Panel (PTDL_v2 format)

Steps:
1. In the panel, import the egg file matching your panel version.
2. Create a new server using the OCAP2 Web egg.
3. Set the key variables: **OCAP Secret** (must match `api.apiKey` in extension config), and optionally **Allowed Steam IDs** and **Steam API Key** for admin access.
4. Start the server. The install script automatically creates the data, db, and maps directories.
5. Note: the egg uses `ghcr.io/ocap2/web:full` by default (includes map tools). Switch to `ghcr.io/ocap2/web:latest` for a smaller image if map processing is not needed.

## Constraints

- Do not document Arma 3 server setup from scratch — link to BI wiki for that.
- BattlEye whitelist steps are version-dependent; note the requirement but don't prescribe exact file paths.
- All config values belong in `docs/configuration.md`, not this doc — link there instead of repeating.
- Do not invent Docker Compose field names; use only what's supported by the Docker image's ENV defaults from the Dockerfile.
