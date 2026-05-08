# README Update Design

**Date:** 2026-05-08
**Output file:** `README.md` (root)

## Goal

Rewrite `README.md` to serve as a hub page: strong intro, feature overview, concise getting-started section with links to the new docs, and preserved Usage/Terrains content. Remove the outdated per-field configuration listings and superseded installation steps.

## What Changes

### Remove

- The entire **Configuration** subsection under "Running OCAP" — it references `OcapReplaySaver2.cfg.json` (old extension config name) and `userconfig/config.hpp` (no longer exists; settings moved to CBA in-game). Replaced by a one-sentence callout + link to `docs/configuration.md`.
- The entire **Installation** subsection under "Running OCAP" — superseded by `docs/installation.md`. Replaced by a 3-step quick-start summary.

### Keep Verbatim

- Banner image, title, screenshot
- Demo, Maps, Discord links
- "What is it?" paragraph
- Feature Overview bullet list
- Terrains subsection (not in new docs)
- Usage subsection (`ocap_fnc_exportData` examples + warning block)
- Detailed Features section
- Current Developers + Credits

### New Section: "Getting Started" (replaces "Running OCAP")

Structure:

```
## Getting Started

### Installation

Three-line quick-start summary:
1. Download the release archive from GitHub Releases (link to releases page)
2. Install `@ocap/` on your Arma 3 server as a `-serverMod` (CBA_A3 required)
3. Deploy the web server (binary, Docker, or Pterodactyl/Pelican)

→ See the full [Installation Guide](docs/installation.md) for step-by-step instructions.

### Configuration

One-sentence summary: OCAP's settings are split across three places —
in-game CBA options (addon), `ocap_recorder.cfg.json` (extension),
and `setting.json` (web server).

→ See the [Configuration Reference](docs/configuration.md) for all options.

### Terrains
[verbatim from current README]

### Usage
[verbatim from current README including the warning block]
```

## Section Order After Update

1. Banner image + title
2. Screenshot + demo/maps/Discord links
3. What is it?
4. Feature Overview
5. **Getting Started** (new — replaces "Running OCAP")
   - Installation (3-step summary + link)
   - Configuration (1-sentence summary + link)
   - Terrains (verbatim)
   - Usage (verbatim)
6. Detailed Features
7. Current Developers
8. Credits

## Constraints

- Do not invent new feature descriptions or change the tone of existing prose.
- Do not move or reword the Terrains or Usage content — copy it exactly.
- The warning block in Usage ("WARNING: To ensure your recordings are saved...") must be preserved.
- Links to `docs/installation.md` and `docs/configuration.md` use relative paths (no hostname).
- Do not add any content not described in this spec.
