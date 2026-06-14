# Vinyl Now Playing

<!-- SHARED-FACTS:START -->
<!-- Auto-synced from SHARED-MEMORY.md by scripts/sync-shared-facts.py. Do NOT edit between these markers — edit the master file instead. -->
## Gmail accounts (two Gmail MCPs connected)
- **Personal** — `lanebecker@gmail.com` → the `mcp__gmail__*` tools (self-hosted ArtyMcLabin Gmail-MCP-Server fork; installed Jun 13 2026; scopes: gmail.modify + settings.basic).
- **Work** — `lbecker@wikimedia.org` → the pre-existing Gmail connector.
- Default: "my Gmail" / "personal Gmail" → the fork (`mcp__gmail__*`). Say "work Gmail" for the WMF account.

## GitHub workflow
- Always push/commit changes via the **GitHub API / MCP** (e.g. `create_or_update_file`, `push_files`) — **never** local `git` commands. (Reason: the Cowork sandbox can't authenticate a local `git push`; the GitHub MCP tools carry the PAT. This is the canonical rule — no fuller version exists; reconstructed from project references after the original "global memory" file turned out never to have been created.)
<!-- SHARED-FACTS:END -->

Design prototype and production renderer for a vinyl now-playing display (1024×600 Waveshare, Raspberry Pi).

## Repository Structure

| Path | Purpose |
|------|---------|
| `design/` | React/Babel design prototype — browser-runnable, design tool only |
| `src/` | Production Python/Pillow/pygame renderer |
| `src/display/assets/fonts/` | Bundled OFL fonts (Inter Tight, Newsreader, JetBrains Mono) used by the production renderer |
| `PRODUCT.md` | Product spec |
| `DESIGN.md` | Design system and production handoff spec |
| `design/.impeccable/design.json` | Design system tokens for impeccable tooling |

## Prototype vs. Production

The `design/` directory is a **design tool, not production code.** It runs in a browser via React/Babel CDN imports and is used for design iteration and review. All geometry and specs in `DESIGN.md` are derived from the prototype.

The `src/` directory contains the **production renderer** — Python/Pillow/pygame targeting the Raspberry Pi. When a design decision from the prototype is ready to ship, it gets translated here.

## Running the Prototype

Open `design/index.html` in a browser (serve from the `design/` directory — local `file://` works for most features). No build step.

## Key Design Constraints

- Display: 1024×600px, fixed (no responsive breakpoints)
- Font hierarchy: Inter Tight 600 72px (song) / Inter Tight 500 48px (artist) / Newsreader italic 32px (album) / JetBrains Mono 13px (catalog) / 12px (chip) / 11px (label)
- DisplayPalette: 5 values — bg, surface, accent, text, muted — 1s lerp on track change
- Full-Opacity Rule: `p.muted` never gets additional `opacity` stacked on top
- Hue Diversity Rule: ≥60° OKLCH hue separation between album accent colors
- WCAG AA: 4.5:1 minimum contrast on all text
