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
