---
target: Vinyl Now Playing.html
total_score: 34
p0_count: 0
p1_count: 0
timestamp: 2026-06-11T02-30-37Z
slug: vinyl-now-playing-html
---
### Design Health Score

| # | Heuristic | Score | Key Issue |
|---|-----------|-------|-----------|
| 1 | Visibility of System Status | 4 | Five distinct states with color, animation, and text labels — excellent |
| 2 | Match System / Real World | 4 | Vinyl vocabulary throughout; physical metaphors are exact |
| 3 | User Control and Freedom | 4 | N/A by design — passive display correctly shows no phantom interactivity |
| 4 | Consistency and Standards | 3 | Accent rule uses opacity 0.65; compact eyebrow side counter stacks opacity 0.7; catalog footer uses undefined 13px type step |
| 5 | Error Prevention | 3 | State guards solid; no error/timeout state for failed ShazamIO identification |
| 6 | Recognition Rather Than Recall | 4 | Perfect hierarchy; everything readable at a glance |
| 7 | Flexibility and Efficiency | 3 | Five variants; no resilience for variable genre chip count or overlong track names |
| 8 | Aesthetic and Minimalist Design | 3 | Near-perfect; genre chip quantity creates inconsistent lower-section rhythm |
| 9 | Error Recovery | 2 | No "couldn't identify" / timeout state — display stuck in boot indefinitely |
| 10 | Help and Documentation | 4 | N/A — self-documenting passive display |
| **Total** | | **34/40** | **Good** |

### Anti-Patterns Verdict

LLM assessment: Not AI slop. The three-family typographic semantic system, the per-album palette architecture with hue diversity enforcement, and the complete rejection of streaming-app chrome are all genuine design decisions. The one tell is the accent rule's opacity: 0.65 — a hairline inconsistency against a system that explicitly prohibits it.

Deterministic scan: 2 findings, both false positives. overused-font (Inter Tight) is a deliberate compressed display face choice, not a generic body font. single-font is incorrect — all three families (Inter Tight, Newsreader, JetBrains Mono) are loaded in the same Google Fonts link at line 9.

### Overall Impression

Well-reasoned passive display with genuine typographic and palette craft. Happy path is excellent; edge paths (failed identification, long track names, variable genre chips) reveal where the spec ended.

### What's Working

1. The typographic system is genuinely semantic: Inter Tight for human content, Newsreader italic exclusively for album titles (One Serif Rule held), JetBrains Mono exclusively for data. Real theory, executed correctly.
2. Palette architecture scales: five semantic roles, per-album extraction, 1-second lerp, Hue Diversity Rule (all five albums spanning hue wheel at 55/125/225/290/345 degrees), Full-Opacity Rule. Production-ready infrastructure.
3. Vocabulary of restraint held without apology: no rounded corners, no progress bars, no elapsed time, no playback chrome. Every omission is a deliberate decision.

### Priority Issues

**[P2] Accent rule opacity: 0.65 violates Full-Opacity Rule**
DirectionA.jsx line 155; design.json .ds-accent-rule. Opacity compounds across palette — rule reads ghosted on pale accents (Fugazi's #b5cee2). Fix: remove opacity, adjust width/height instead.
Suggested command: /impeccable polish

**[P2] No error/timeout state — ShazamIO failures leave display in perpetual boot**
stateLabel() and DirAEmptyCover handle boot as indefinite "Listening..." with no timeout differentiation. Fix: add error state with distinct status dot color (#c85050), "COULDN'T IDENTIFY" label, static (non-rotating) arc.
Suggested command: /impeccable harden

**[P2] Track hero has no overflow protection**
72px track name with textWrap: balance but no line clamp. Long production titles (e.g., "All Too Well (Ten Minute Version) (The Short Film)") will run 3-4 lines and break the metadata column. Fix: add WebkitLineClamp: 3 consistent with album title treatment.
Suggested command: /impeccable harden

**[P2] Genre chip count variability creates inconsistent lower-section layout**
marginBottom: auto on chip container + 0 or 5+ chips in production = footer floating or off-screen. Fix: clamp to max 3 chips with "+N more" indicator; add minHeight: 28 to reserve space when empty.
Suggested command: /impeccable harden

**[P3] Idle screen remains the weakest surface**
Diagonal stripe placeholder acknowledged as v1.4.0 gap in DESIGN.md. Most-seen surface during non-listening time. Planned redesign (recently played grid, clock/date, random Discogs suggestion) is the highest-impact remaining design work.
Suggested command: /impeccable craft idle-screen

### Persona Red Flags

Sam (Accessibility): Sister's muted (#8c7e88) on bg (#1d1822) is ~3.8:1 — below 4.5:1 WCAG AA for 11px text. All five album palettes need muted/bg contrast verification.

The Record Listener (project-specific, 6-10ft distance): Key hierarchy (72/48/32px) readable at distance. Status strip (11px) and catalog footer (13px) are glanceable-only from across the room — acceptable by design intent. The muted contrast issue compounds here.

Riley (Stress Tester): between state + showAdjacent=true creates duplicate "up next" track name (hero text and NEXT panel both show same track). Production likely handles this by not setting showAdjacent during between state, but design doesn't encode the constraint.

### Minor Observations

- Sleeve variant opacity: 0.7 on catalog text (line 99) — Full-Opacity violation
- Compact eyebrow side counter opacity: 0.7 (line 133) — Full-Opacity violation
- 13px catalog footer is an undocumented type scale step (defined stops: 72/48/32/12/11px)
- Adjacent track fallback chain truncated: "Inter Tight", sans-serif vs full "Inter Tight", "DejaVu Sans", Arial, sans-serif elsewhere
- textWrap: balance only on track hero; artist name at 48px could benefit too
