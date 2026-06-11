---
target: Vinyl Now Playing.html
total_score: 35
p0_count: 0
p1_count: 1
timestamp: 2026-06-11T03-10-39Z
slug: vinyl-now-playing-html
---
## Design Health Score

| # | Heuristic | Score | Key Issue |
|---|-----------|-------|-----------|
| 1 | Visibility of System Status | 4 | Six-state model (playing/between/paused/idle/boot/error) maps cleanly to color + label + animation. Exemplary. |
| 2 | Match System / Real World | 4 | "Side A · 04 OF 06", "PAUSED · TONEARM UP", "SST Records · SST-134". This design IS the physical vocabulary. |
| 3 | User Control and Freedom | 4 | Passive display with zero false affordances. No implied interactivity — correctly non-interactive by design. |
| 4 | Consistency and Standards | 4 | Palette contrast AA across all 6 albums (post-fix). Font fallbacks complete across all type roles. Mono-firewall, full-opacity rule, and accent rule all enforced uniformly. |
| 5 | Error Prevention | 3 | `album.prev.track` and `album.next.track` will throw if prev/next is null (first/last track on a side). `album.genres` has no null guard either. Production crash risk under real data. |
| 6 | Recognition Rather Than Recall | 4 | All information visible at a glance. State-appropriate suppression of metadata in empty states is clean and consistent. |
| 7 | Flexibility and Efficiency | 3 | Five layout variants + multiple state modes. No end-user configurability (appropriate for the device context). |
| 8 | Aesthetic and Minimalist Design | 4 | Genuinely excellent. Track name dominates, hierarchy cascades cleanly, no decoration for its own sake. Every element earns its pixel. |
| 9 | Error Recovery | 3 | Error state clearly communicated (static arc vs rotating boot arc). No user recovery path — appropriate for passive display. Boot state provides no elapsed time signal, so listener can't distinguish "still identifying" from "hung" after 30+ seconds. |
| 10 | Help and Documentation | 2 | Developer docs (DESIGN.md, design.json) are exemplary. No in-display help or timeout messaging for the room listener. |
| **Total** | | **35/40** | **Good** |

---

## Anti-Patterns Verdict

**LLM assessment:** Not AI slop — confidently not. The design has a specific, defended concept ("The Listening Room Monitor") with coherent vocabulary, unusual type pairing (Inter Tight + Newsreader + JetBrains Mono is a non-obvious choice), and per-album palette extraction that makes every state feel materially different. The copy is specific and warm ("PAUSED · TONEARM UP", not "Paused"). The status-state model spans 6 distinct states with semantic color mapping. The 64×2px accent rule as deliberate punctuation (not a full-width divider) is a design opinion, not a template. This reads as handmade.

**Deterministic scan:** Detector returned 2 warnings on `Vinyl Now Playing.html`:
- `overused-font` (line 9): flagged "Google Fonts: Inter" — **false positive**. The project uses Inter Tight, a compressed variant distinct from generic Inter, as the deliberate display/body face in a 3-family system.
- `single-font`: "only font used is inter tight" — **false positive**. The HTML file loads Inter Tight from Google Fonts, but Newsreader and JetBrains Mono are also loaded and actively used. The detector reads only the first font import line.

No real anti-patterns detected. Both findings dismissed.

**Browser visualization:** Skipped — browser injection not available in this environment. CLI scan is the sole Assessment B signal; both findings are false positives as noted above.

---

## Overall Impression

This design is working at a high level. The concept is clear and defended, the system is internally consistent, and the post-session improvements (WCAG AA contrast on all 6 palettes, complete font fallbacks, `textWrap: balance` on artist name, Catalog type step formalized) have meaningfully tightened the craft. The null-guard gap in the adjacent panel is the one production-threatening issue remaining. Everything else is P3 polish or philosophical questions about the between-state design.

---

## What's Working

**The six-state model (Heuristic #1, 4/4).** Playing → amber → paused → idle → boot → error. Each state has a distinct color, label, and animation signature. The `PAUSED · TONEARM UP` copy is the best label in the entire system — specific, physical, warm. The error arc (static) vs boot arc (rotating) is a small but precise semantic distinction that would help a listener troubleshoot from across the room.

**Per-album palette extraction.** Sonic Youth's orange vs Cavetown's dusty rose vs Fugazi's steel blue vs Aimee Mann's olive — these feel genuinely distinct, and the post-session contrast fixes mean all six pass WCAG AA. The Hue Diversity Rule (≥60° separation) kept the collection from clustering toward warm orange. This is the design's defining feature and it's earned.

**Information hierarchy.** Track name at 72px → artist at 48px → album title at 32px serif italic → mono metadata at 13/11px. The scale jumps are large enough to read correctly from 6-10 feet. The accent rule (64×2px) between track and artist is the one decorative element that earns its place: it creates a reading pause that helps the eye separate "what" from "who."

---

## Priority Issues

**[P1] Null guard missing on adjacent tracks and genres**
- **What**: `album.prev.track` and `album.next.track` are accessed without null checks at line 229/237 of DirectionA.jsx. `album.genres` has no null guard before `.slice(0, 3)` at line 187.
- **Why it matters**: In production, the first track on Side A has no previous track, and the last track on Side B has no next. With `showAdjacent: true` active, the component throws — the entire display crashes. A vinyl listener staring at a blank screen can't tell if the Pi died or ShazamIO lost signal.
- **Fix**: Replace `album.prev.track` with `album.prev?.track ?? '—'` and `album.next?.track ?? '—'`. Replace `album.genres.slice(0, 3)` with `(album.genres ?? []).slice(0, 3)`. Consider hiding the PREV or NEXT div entirely when the respective track is null (first/last track on the side), rather than showing a dash.
- **Suggested command**: `/impeccable harden`

**[P2] Boot state provides no elapsed-time signal**
- **What**: The "IDENTIFYING…" boot spinner gives no indication of how long identification has been running. ShazamIO can take 10–60+ seconds depending on signal quality.
- **Why it matters**: After ~15 seconds, a listener can't distinguish "still working" from "something broke." The boot and error states are visually distinct (rotating vs static arc), but the listener doesn't know which one they're heading toward. This creates anxiety at the exact moment when reassurance matters.
- **Fix**: Add an elapsed timer to the boot state — something like a subtly incrementing seconds count or a label change ("IDENTIFYING… 0:34") using a `setInterval` hook. Alternatively, a second label transition after N seconds ("STILL LISTENING…") that acknowledges the delay without implying failure. Either approach stays within the JetBrains Mono / `p.muted` vocabulary.
- **Suggested command**: `/impeccable harden`

**[P3] Empty cover font size inconsistency**
- **What**: The idle state cover placeholder shows "NO RECORD ON PLATTER" at `fontSize: 13, letterSpacing: '0.24em'`. Boot ("WARMING UP") and error ("NO MATCH FOUND") use `fontSize: 11, letterSpacing: '0.2em'`. These are all status labels inside the same 440×440 empty cover area.
- **Why it matters**: Three components inside `DirAEmptyCover` should share a consistent typographic treatment. The inconsistency is invisible to casual viewers but breaks the system's own rules (Label scale is 11px).
- **Fix**: Change the idle cover label to `fontSize: 11, letterSpacing: '0.16em'` to match Label spec. Or — if the 13px feels better at distance — update boot/error to 13px/0.24em and document as a Cover Label variant.
- **Suggested command**: `/impeccable polish`

**[P3] Vestigial `elapsed` / `duration` fields in data.js**
- **What**: `data.js` includes `duration` and `elapsed` fields on every album object. These are never rendered in DirectionA.jsx (per spec — no progress bars allowed).
- **Why it matters**: A future contributor who reads the data schema without knowing the rationale might reasonably add a progress bar. The fields are a footgun.
- **Fix**: Either remove the fields from `data.js`, or add a comment making the no-render intent explicit: `// duration/elapsed are NOT rendered — the display is intentionally static. See DESIGN.md §"No progress bars".`
- **Suggested command**: `/impeccable harden`

---

## Persona Red Flags

**Riley (Stress Tester)** — walks through first/last track edge cases, then genre edge cases:

- Starts an album from Side A Track 1 (`position: 1`, `prev: null`) with `showAdjacent: true`. DirectionA.jsx crashes: `Cannot read properties of null (reading 'track')`. Black screen. No recovery without refreshing the page/restarting the process.
- Sets `album.genres: []` (Discogs returns no genres for this pressing). Renders correctly — `minHeight: 28` preserves chip area layout, zero chips shown. ✓ (Passes for empty array case.)
- Sets `album.genres: undefined` (null return from resolver). `undefined.slice(0,3)` throws. Same crash vector as the prev/next null guard.
- Types an album with a very long "Up next" track name during `between` state: "Up next — Bachelor No. 2 or, the Last Remains of the Dodo" renders at 72px with the 3-line clamp. The "Up next — " prefix consumes ~280px of the first line alone. The actual track name gets compressed into lines 2-3 and may be nearly illegible at 72px in 2 lines. The prefix should probably be a smaller-scale eyebrow, not part of the 72px display text.

**The Vinyl Listener (Project-specific persona)** — seated 8 feet from the display, glancing at it mid-track:

- During `boot` state for 45 seconds: "IDENTIFYING…" is visible, spinner is rotating. Can't tell if it's still working or stuck. Might walk over to check the Pi. The lack of elapsed time is the only thing that breaks immersion.
- During `between` state: sees "Up next — Merchandise". Artist/album shown is still Fugazi / Repeater (correct for same-album context). If the between state were cross-side or cross-album (future v1.5 side-flip), the artist/album metadata shown would be for the previous album while the track says "Up next — [next album's first track]". Confusing. But this is a documented v1.5 design gap, not a current bug.
- Notices genre chips disappear when `isEmpty: true` (correct — metadata is suppressed in boot/idle/error). No red flag here.

---

## Minor Observations

- `fmtTime()` is defined in `data.js` but unused anywhere in the component tree. Either remove it or add a comment that it's reserved for future use (idle screen, v1.4.0).
- `aria-live` is absent from the status strip. For a room display this is low priority, but if the prototype is ever demoed on a machine with VoiceOver, state changes won't be announced. Adding `aria-live="polite"` to the status strip would be a one-line fix.
- The `between` state's `trackText` returns `"Up next — ${album.next.track}"` rendered at 72px Display scale. The "Up next — " prefix at 72px is heavy — it's roughly 280px wide before the track name begins. A more space-efficient treatment: separate the label ("NEXT") from the track name at different type scales, similar to how the PREV/NEXT adjacent panel works but as the primary display area content.
- The detector's `overused-font` and `single-font` warnings are both false positives (Inter Tight in a 3-family system). No action needed; suppress in ignore.md if desired.
