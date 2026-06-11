# Critique Ignore List

Findings listed here are suppressed on all future `/impeccable critique` runs.

## False Positives

### `overused-font` — Inter Tight is not generic Inter
The detector flags "Inter" from the Google Fonts import line. This project uses **Inter Tight**,
a compressed variant with different metrics and personality. It is paired with Newsreader (serif)
and JetBrains Mono (monospace) as a deliberate 3-family system. Not a generic AI-default choice.

### `single-font` — Three font families are actively used
The detector reads only the first Google Fonts `@import` line and concludes only one font is used.
In reality the design uses Inter Tight, Newsreader, and JetBrains Mono — each assigned to a
semantic tier (display/headline, album title, and all mono/label/catalog text respectively).
