---
target: Vinyl Now Playing.html
total_score: 38
p0_count: 0
p1_count: 0
timestamp: 2026-06-11T12-29-45Z
slug: vinyl-now-playing-html
---
## Design Health Score

| # | Heuristic | Score | Key Issue |
|---|-----------|-------|-----------|
| 1 | Visibility of System Status | 4 | Six-state model with distinct color/animation/label per state. Boot elapsed timer ("WARMING UP" → "STILL LISTENING…" → "IDENTIFYING… M:SS") seals it. |
| 2 | Match System / Real World | 4 | Physical vocabulary throughout. "PAUSED · TONEARM UP", side/position notation, serif album title. The language is the artifact. |
| 3 | User Control and Freedom | 4 | Passive display; no false affordances. Correctly non-interactive. |
| 4 | Consistency and Standards | 4 | Cover label scale unified at 11px Label spec post-fix. Fallback fonts, palette roles, full-opacity rule, and mono firewall all consistently enforced. |
| 5 | Error Prevention | 4 | Null guards on album.prev, album.next (optional chaining in trackText), and album.genres (nullish coalescing to []). Core prototype crash vectors closed. |
| 6 | Recognition Rather Than Recall | 4 | All information visible at a glance. State-appropriate metadata suppression in empty states is clean and consistent. |
| 7 | Flexibility and Efficiency | 3 | Five layout variants by design. No runtime configurability — appropriate for fixed-deployment passive display. Gap is inherent to the device context, not a design failure. |
| 8 | Aesthetic and Minimalist Design | 4 | NEXT eyebrow at 11px/p.muted cleanly separates state framing from the 72px track hero. Every element earns its pixel. The between-state redesign is a genuine improvement over the embedded "Up next — " prefix approach. |
| 9 | Error Recovery | 4 | Boot: progressive elapsed labels help the listener distinguish "still working" from "hung." Error: static arc + "NO MATCH FOUND" + "REPOSITION NEEDLE TO RETRY" gives clear visual diagnosis and a concrete action. |
| 10 | Help and Documentation | 3 | Error recovery hint and boot progress labels are meaningful in-display signals for the room listener. DESIGN.md and design.json developer documentation remain exemplary. |
| **Total** | | **38/40** | **Excellent** |

---

## Anti-Patterns Verdict

**LLM assessment:** Still not AI slop — and now more confidently not. The between-state redesign demonstrates a specific design opinion: the track name should be the hero even when the context is "what's coming next." Putting "NEXT" at 11px mono above the 72px track name, rather than embedding "Up next — " into the display scale, is a restraint choice that most generators wouldn't make. The error state recovery hint follows the system's established vocabulary (mono, uppercase, p.muted) rather than introducing a new UI pattern. Everything coheres.

**Deterministic scan:** Detector returned the same 2 warnings as the previous run:
- `overused-font` (line 9): flagged "Google Fonts: Inter" — **false positive** (suppressed in ignore.md). Project uses Inter Tight, a compressed variant distinct from generic Inter.
- `single-font`: "only font used is inter tight" — **false positive** (suppressed in ignore.md). Three font families actively used: Inter Tight, Newsreader, JetBrains Mono.

No real anti-patterns detected. Both findings dismissed per ignore.md.

**Browser visualization:** Skipped — browser injection not available in this environment.

---

## Overall Impression

Three clean fixes, three heuristic improvements, no regressions. The prototype is now in "Excellent" territory (38/40), up from 35. The remaining gap — H7 (inherent to a fixed passive display) and H10 (honest limit of what in-display help can do) — has no clear fix that wouldn't add noise. The design is doing exactly what it set out to do.

The one watch item: the prototype doesn't suppress artist/album/chips/catalog metadata in boot/idle/error states (DESIGN.md says it should). In the prototype this doesn't matter — sample data is always present. In production Python renderer alignment, it's worth confirming.

---

## What's Working

**The six-state model, now fully instrumented (H1, H9: 4/4).** Playing → amber → paused → idle → boot → error. Every state has visual diagnosis + label. The boot elapsed timer is the last piece that was missing — a listener can now tell the difference between "1:05 of identifying" and "something hung." The error arc (static) vs boot arc (rotating) distinction plus "REPOSITION NEEDLE TO RETRY" means a listener can troubleshoot without touching the Pi.

**The between-state NEXT eyebrow.** Replacing "Up next — Catholic Block" at 72px with a 11px "NEXT" eyebrow + "Catholic Block" at 72px is a better reading experience and a cleaner design argument. The track name is the hero because it's the thing the listener wants to know. The contextual label is secondary because it's framing. The hierarchy now matches the information value.

**The null guard coverage.** `(album.genres ?? [])`, `album.prev &&`, `album.next &&`, and `album.next?.track ?? '—'` in trackText all guard the optional fields. The production crash vectors from the P1 in the previous run are closed.

---

## Priority Issues

**[P3] Compact variant status eyebrow drifts from Label spec**
- **What**: The compact variant's status eyebrow (lines 119–138) uses `fontSize: 12, letterSpacing: '0.22em'`. The Label spec is 11px / 0.16em.
- **Why it matters**: This is the one place in the component tree where inline styles deviate from the Label scale without documentation. Minor, but the rule is "11px is the floor for all instrumental metadata."
- **Fix**: Change compact eyebrow to `fontSize: 11, letterSpacing: '0.16em'`. Or document the deviation: the compact eyebrow carries more info density than a typical label (status + side position on one line), and the extra pixel is intentional.
- **Suggested command**: `/impeccable polish`

**[P3] Sleeve variant inner catalog text at 8px — below the Label scale floor**
- **What**: The sleeve variant renders catalog + side labels at `fontSize: 8, letterSpacing: '0.24em'` inside the sleeve frame bottom (lines 97–104).
- **Why it matters**: 8px is invisible at 6–10ft viewing distance. The Label spec floor is 11px for a reason. This text is decorative at 8px, but if it's worth rendering at all, it should be readable.
- **Fix**: Either raise to 11px (may not fit in the 16px sleeve bottom bar), or remove the sleeve label entirely — the catalog footer line already carries this info in the meta column.
- **Suggested command**: `/impeccable polish`

**[P3] `fmtTime()` in data.js is vestigial**
- **What**: `fmtTime()` is defined in `data.js` but not called anywhere in the component tree.
- **Why it matters**: A future contributor reading the data schema might assume it's intentional and add elapsed time display, violating the "no progress bars" rule.
- **Fix**: Remove the function, or add a comment: `// fmtTime — defined but NOT rendered. The display is intentionally static. See DESIGN.md §"No progress bars".`
- **Suggested command**: `/impeccable harden`

---

## Persona Red Flags

**Riley (Stress Tester)** — walks the edge cases:
- Sets `album.prev: null` on Side A Track 1 with `showAdjacent: true`. Component now renders correctly — the `album.prev &&` guard hides the PREV div entirely. ✓
- Sets `album.genres: undefined`. `(album.genres ?? []).slice(0, 3)` returns [] safely. ✓
- Enters compact variant during between state. Sees two consecutive label lines before the hero: "[amber dot] BETWEEN TRACKS · SIDE A03/06" (compact eyebrow) then "NEXT" (between eyebrow) then the 72px track name. Technically correct — both labels carry distinct information — but the stacking looks dense on a first glance. Low severity, noted.
- Throws a 40-character track name at between state ("The Ballad of El Goodo (Studio Version)"). At 72px with `WebkitLineClamp: 3`, this wraps to ~2 lines cleanly. ✓

**The Vinyl Listener (Project-specific)** — seated 8 feet from the display mid-vinyl:
- During boot for 25 seconds: reads "STILL LISTENING…" in the cover area. Knows it's still working. No need to walk over to the Pi. ✓ (Previous critique: "might walk over to check the Pi.")
- During boot for 75 seconds: reads "IDENTIFYING… 1:15". Clearly still running, not hung. ✓
- After error state: "REPOSITION NEEDLE TO RETRY." Lifts tonearm, repositions stylus, plays. Expected action, clearly communicated. ✓
- During between state: sees "NEXT" eyebrow then track name at full size. Glances at it from 8 feet — legible as "what's coming." Artist and album below confirm it's still the same record. Metadata continuity is correct.

---

## Minor Observations

- The status strip shows "BETWEEN TRACKS" while the meta column's NEXT eyebrow also signals the between state. Not redundant — they carry different information (state vs. track context) — but worth confirming that both are intentional in the full layout review.
- DESIGN.md §5 states: "All three states replace the cover image and show no album metadata." The implementation currently shows artist/album/chips/catalog even in boot/idle/error states. In the prototype this is fine (sample data always present). For production Python renderer alignment, confirm whether stale album metadata should be cleared or retained when entering idle/boot.
- `aria-live="polite"` on the status strip is in place. If the prototype is demoed with VoiceOver, state changes will be announced. ✓

---

## Questions to Consider

- The sleeve variant's 8px bottom label — is it worth keeping at all, or is it decorative scaffolding that never survived contact with the 11px rule?
- If/when the idle screen redesign (v1.4.0: recent album grid + clock) ships, will the "no album metadata during idle" question resolve itself, since the idle state will have its own distinct layout?
- The compact variant's between state shows two consecutive label lines before the hero. Is compact ever expected to be in between state in a real deployment, or is it a design-tool-only edge case?
