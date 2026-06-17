# Vinyl Now Playing

<!-- SHARED-FACTS:START -->
<!-- AUTO-GENERATED shared facts. Do NOT edit between the SHARED-FACTS markers, and do NOT delete the markers themselves. This block is synced into every Projects/*/CLAUDE.md by scripts/sync-shared-facts.py from the master SHARED-MEMORY.md; BOTH live in ~/Documents/Claude/ (one level ABOVE the Projects folder) and are re-run by a scheduled task. To change these facts: edit SHARED-MEMORY.md and rerun the script. Manual edits or deletions here are overwritten/restored on the next sync. -->
## Gmail accounts (two Gmail MCPs connected)
- **Personal** — `lanebecker@gmail.com` → the `mcp__gmail__*` tools (self-hosted ArtyMcLabin Gmail-MCP-Server fork; installed Jun 13 2026; scopes: gmail.modify + settings.basic).
- **Work** — `lbecker@wikimedia.org` → the pre-existing Gmail connector.
- Default: "my Gmail" / "personal Gmail" → the fork (`mcp__gmail__*`). Say "work Gmail" for the WMF account.

## GitHub workflow
- **Default: local-first** (as of 2026-06-16; it's faster than the API). Claude edits files in a local clone that lives **inside the connected project folder**, runs the test suites in the sandbox, then hands Lane a `git add -A && git commit -m "…" && git push` to run on his Mac.
- **The sandbox never runs `git`.** git's lock/hardlink operations fail on the FUSE-mounted folder ("could not lock config file … Operation not permitted") and the sandbox can't authenticate a push anyway. So cloning, committing, and pushing are all done by Lane on his Mac (native filesystem, his credentials).
- **GitHub API / MCP (`push_files`, `create_or_update_file`) is the backup** — use it when Lane is away from his Mac or no local clone is available. (The PAT-carrying MCP tools still work; they're just no longer the default.)
- **Commit email must be the GitHub `noreply` alias.** Lane's account has "Block command line pushes that expose my email" enabled, so any commit stamped with `lanebecker@gmail.com` is rejected on push with `GH007: Your push would publish a private email address`. Each clone's `user.email` must be his noreply address (`<id>+lanebecker@users.noreply.github.com`, from github.com/settings/emails). Set it globally — `git config --global user.email "<id>+lanebecker@users.noreply.github.com"` — so every clone inherits it. If a commit was already made with the wrong email: `git commit --amend --reset-author --no-edit` then push.

## Memory conventions
- **Keep a `log.md`** — an append-only change history, newest at the bottom, each line prefixed with an ISO-8601 UTC timestamp. `scripts/sync-shared-facts.py` creates one per project and records each sync; add your own notable changes too. In a Cowork space, keep it at `memory/log.md`.
- **Timestamp every memory entry.** Any knowledge/memory file with YAML frontmatter carries an ISO-8601 `timestamp` field; the sync script warns about `memory/*.md` entries that don't.
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
