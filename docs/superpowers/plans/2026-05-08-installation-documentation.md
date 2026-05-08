# Installation Documentation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Write `docs/installation.md` — a complete installation guide for OCAP covering the Arma 3 addon+extension setup and three web server deployment methods.

**Architecture:** Single Markdown file with a component overview up front, followed by one section for the Arma 3 server (applies to all deployments) and three self-contained sections for each web server deployment method.

**Tech Stack:** Markdown only. No code changes. Sources: `.github/workflows/release.yml`, `web/Dockerfile`, `web/egg-ocap2-web.json`, `web/egg-ocap2-web-pterodactyl.json`, `extension/ocap_recorder.cfg.json.example`, `web/setting.json.example`.

---

### Task 1: Write `docs/installation.md` — Overview and Downloads

**Files:**
- Create: `docs/installation.md`

- [ ] **Step 1: Create the file with the overview and downloads sections**

Create `/home/jacob/OCAP/docs/installation.md` with:

```markdown
# Installation

OCAP has two parts that need to be installed:

- **Arma 3 server component** — the `@ocap/` folder, which contains both the **addon** (SQF PBOs that hook into game events) and the **extension** (the native `ocap_recorder_x64.dll` / `.so` that records data). These always ship and are installed together.
- **Web server** — receives uploaded recordings from the extension and serves the browser-based playback UI. Can run on a different machine from the Arma 3 server.

Neither component is optional: the addon without the extension has nowhere to send data, and the web server without the addon receives nothing.

```
Arma 3 server
  └── @ocap/  (addon PBOs + extension DLL/SO, loaded via -serverMod)
        └── HTTP upload ──► web server ◄── browser playback
```

---

## Downloads

Download the latest release from the [OCAP2/OCAP GitHub Releases](https://github.com/OCAP2/OCAP/releases) page.

| Archive | Contents | Use for |
|---------|----------|---------|
| `windows.zip` | `@ocap/` (PBOs + `ocap_recorder_x64.dll`) + `web/` (`ocap-webserver.exe` + assets) | Windows Arma 3 server + Windows web server |
| `linux.tar.gz` | `@ocap/` (PBOs + `ocap_recorder_x64.so`) + `web/` (`ocap-webserver` + assets) | Linux Arma 3 server + Linux web server |

If you are running the web server in Docker or on a game server panel, you only need the archive that matches your **Arma 3 server's OS** — you won't use the `web/` folder from it.

### Docker images (web server only)

| Image | Description |
|-------|-------------|
| `ghcr.io/ocap2/web:latest` | Slim — web server only |
| `ghcr.io/ocap2/web:full` | Includes GDAL, tippecanoe, and pmtiles for server-side map terrain tile processing |

Use `:full` if you want to process and host custom Arma 3 terrain tiles through the admin UI. Use `:latest` otherwise.

---
```

- [ ] **Step 2: Commit**

```bash
git -C /home/jacob/OCAP add docs/installation.md
git -C /home/jacob/OCAP commit -m "docs: add installation guide overview and downloads section"
```

---

### Task 2: Arma 3 Server — Addon and Extension Installation

**Files:**
- Modify: `docs/installation.md` (append)

- [ ] **Step 1: Append the Arma 3 server section**

Append to `docs/installation.md`:

```markdown
## Arma 3 Server: Installing the Addon and Extension

This section applies regardless of which web server deployment method you choose. Complete it first.

**Prerequisites:**
- Arma 3 dedicated server (Windows or Linux)
- [CBA_A3](https://steamcommunity.com/workshop/filedetails/?id=450814997) installed as a server mod

### 1. Extract the `@ocap/` folder

Extract your release archive and locate the `@ocap/` directory. It contains:

```
@ocap/
├── addons/              # SQF PBO files (the addon)
├── keys/                # .bikey signature key(s)
├── ocap_recorder_x64.dll   # (Windows) the extension
├── ocap_recorder_x64.so    # (Linux) the extension
└── ocap_recorder.cfg.json.example
```

### 2. Place `@ocap/` on the server

Copy the entire `@ocap/` folder into your server's mods directory — the same location where your other server mods live (e.g. alongside `@CBA_A3`).

### 3. Install the signature key

Copy all `.bikey` files from `@ocap/keys/` into the server's root `keys/` directory.

> **Why:** Arma 3 signature verification requires each mod's public key to be in the server's `keys/` folder. Without this the server will reject the mod at startup.

### 4. Add `@ocap` to the `-serverMod` startup parameter

In your server's start script or configuration, add `@ocap` to the `-serverMod` parameter:

```
./arma3server -serverMod="@CBA_A3;@ocap" ...
```

> **`-serverMod` vs `-mod`:** `-serverMod` loads the mod only on the server. Clients do **not** need to install or download `@ocap`. If you add it to `-mod` by mistake, clients will be required to have it too.

### 5. Configure the extension

Copy the example config and edit it:

```bash
cp @ocap/ocap_recorder.cfg.json.example @ocap/ocap_recorder.cfg.json
```

At minimum, set:

```json
{
  "api": {
    "serverUrl": "http://<your-web-server-ip>:5000",
    "apiKey": "your-shared-secret"
  }
}
```

The `apiKey` value must match the `secret` field in the web server's `setting.json` (or the `OCAP_SECRET` environment variable). See [Configuration Reference](configuration.md) for all options.

### 6. Start the server and verify

Start the Arma 3 server. In the RPT log (`.rpt` file in the server logs directory), look for lines containing `OCAP` to confirm the addon and extension initialized correctly.

### BattlEye

If BattlEye is enabled on your server, the extension must be whitelisted or `callExtension` calls will be silently blocked. Add an exception for `ocap_recorder_x64` in your BattlEye configuration. Refer to the [BattlEye documentation](https://www.battleye.com/) for your server's specific whitelist format.

---
```

- [ ] **Step 2: Commit**

```bash
git -C /home/jacob/OCAP add docs/installation.md
git -C /home/jacob/OCAP commit -m "docs: add Arma 3 server addon and extension installation section"
```

---

### Task 3: Web Server — Bare Metal / Binary

**Files:**
- Modify: `docs/installation.md` (append)

- [ ] **Step 1: Append the bare metal section**

Append to `docs/installation.md`:

```markdown
## Web Server: Bare Metal / Binary

Use this method if you are running the web server directly on a Linux or Windows host without Docker.

### 1. Extract the web server files

From the release archive, extract the `web/` directory. It contains:

```
web/
├── ocap-webserver          # Linux binary (or ocap-webserver.exe on Windows)
├── assets/
│   ├── markers/
│   ├── ammo/
│   └── fonts/
└── setting.json.example
```

### 2. Create `setting.json`

```bash
cp setting.json.example setting.json
```

Edit `setting.json` and set `secret` to the same value as `api.apiKey` in the extension's `ocap_recorder.cfg.json`:

```json
{
  "listen": "0.0.0.0:5000",
  "secret": "your-shared-secret"
}
```

See [Configuration Reference](configuration.md) for all options.

### 3. Run the web server

**Linux:**
```bash
chmod +x ocap-webserver
./ocap-webserver
```

**Windows:**
```
ocap-webserver.exe
```

The server listens on port `5000` by default. Visit `http://<server-ip>:5000` to confirm it is running.

### 4. Run as a persistent service (Linux)

Create a systemd unit to keep the server running across reboots:

```ini
# /etc/systemd/system/ocap-web.service
[Unit]
Description=OCAP2 Web Server
After=network.target

[Service]
Type=simple
User=ocap
WorkingDirectory=/opt/ocap/web
ExecStart=/opt/ocap/web/ocap-webserver
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now ocap-web
```

### 5. Firewall

Open port `5000` (TCP) on the host firewall so that:
- The Arma 3 server can upload recordings to the web server.
- Players can access the playback UI in their browser.

---
```

- [ ] **Step 2: Commit**

```bash
git -C /home/jacob/OCAP add docs/installation.md
git -C /home/jacob/OCAP commit -m "docs: add web server bare metal installation section"
```

---

### Task 4: Web Server — Docker Compose

**Files:**
- Modify: `docs/installation.md` (append)

- [ ] **Step 1: Append the Docker Compose section**

Append to `docs/installation.md`:

```markdown
## Web Server: Docker Compose

Use this method if you are running the web server in Docker on a Linux host.

### `docker-compose.yml`

Create a `docker-compose.yml` file with the following contents:

```yaml
services:
  ocap-web:
    image: ghcr.io/ocap2/web:latest
    restart: unless-stopped
    ports:
      - "5000:5000"
    volumes:
      - ocap_db:/var/lib/ocap/db
      - ocap_data:/var/lib/ocap/data
      - ocap_maps:/var/lib/ocap/maps
    environment:
      OCAP_SECRET: "your-shared-secret"
      OCAP_LISTEN: "0.0.0.0:5000"

volumes:
  ocap_db:
  ocap_data:
  ocap_maps:
```

Set `OCAP_SECRET` to the same value as `api.apiKey` in the extension's `ocap_recorder.cfg.json`.

Any `setting.json` field can be set as an environment variable using the `OCAP_` prefix with nested keys uppercased and concatenated (e.g. `auth.steamApiKey` → `OCAP_AUTH_STEAMAPIKEY`). See [Configuration Reference](configuration.md).

> **Map tools:** To use the server-side map tile processing pipeline (GDAL, tippecanoe), use `ghcr.io/ocap2/web:full` instead of `:latest`.

### Start the server

```bash
docker compose up -d
docker compose logs -f   # watch startup logs
```

Visit `http://<host-ip>:5000` to confirm the server is running.

### Data persistence

The three named volumes store:

| Volume | Contents |
|--------|----------|
| `ocap_db` | SQLite database (mission metadata) |
| `ocap_data` | Uploaded recording files |
| `ocap_maps` | Map terrain tiles |

These persist across container restarts and updates. To update:

```bash
docker compose pull
docker compose up -d
```

---
```

- [ ] **Step 2: Commit**

```bash
git -C /home/jacob/OCAP add docs/installation.md
git -C /home/jacob/OCAP commit -m "docs: add web server Docker Compose installation section"
```

---

### Task 5: Web Server — Pterodactyl / Pelican

**Files:**
- Modify: `docs/installation.md` (append)

- [ ] **Step 1: Append the Pterodactyl/Pelican section**

Append to `docs/installation.md`:

```markdown
## Web Server: Pterodactyl / Pelican

Two panel eggs are included in the web server release archive and also available in the [OCAP2/web repository](https://github.com/OCAP2/web):

| File | Panel |
|------|-------|
| `egg-ocap2-web.json` | [Pelican Panel](https://pelican.dev/) (PLCN_v3) |
| `egg-ocap2-web-pterodactyl.json` | [Pterodactyl Panel](https://pterodactyl.io/) (PTDL_v2) |

### 1. Import the egg

In your panel's admin area, navigate to **Nests** (Pterodactyl) or **Eggs** (Pelican) and import the JSON file that matches your panel version.

### 2. Create a server

Create a new server using the **OCAP2 Web** egg. The panel will prompt you to set the following variables:

| Variable | Description |
|----------|-------------|
| **OCAP Secret** | Shared secret for recording uploads. Must match `api.apiKey` in the extension config. |
| **Enable Conversion** | Enables background JSON → Protobuf conversion for faster playback. Recommended: `true`. |
| **Enable Streaming** | Enables live WebSocket streaming from the extension. Default: `false`. |
| **Allowed Steam IDs** | Steam 64-bit IDs (comma-separated) granted admin access to the web UI. Optional. |
| **Steam API Key** | For displaying Steam display names in the admin UI. Optional. |

The **Listen Address** and storage path variables are pre-configured and should not need to be changed.

### 3. Start the server

Start the server from the panel. The install script automatically creates the required data directories.

Visit the server's allocated address to confirm the web UI is accessible.

> **Docker image:** The egg uses `ghcr.io/ocap2/web:full` by default, which includes the map tile processing tools. To use the smaller slim image, change the Docker image in the egg or server settings to `ghcr.io/ocap2/web:latest`.

---
```

- [ ] **Step 2: Commit**

```bash
git -C /home/jacob/OCAP add docs/installation.md
git -C /home/jacob/OCAP commit -m "docs: add web server Pterodactyl/Pelican installation section"
```
