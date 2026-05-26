# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies (Linux/Pi: add --break-system-packages)
pip install -r requirements.txt

# System audio prerequisite
brew install portaudio          # macOS
sudo apt install libportaudio2  # Raspberry Pi / Linux

# Run the app
python main.py

# Run all tests
pytest

# Run with verbose output
pytest -v

# Stop on first failure
pytest -x

# Run a single test
pytest tests/test_recognizer.py::test_confirmation_gate

# Generate Last.fm session key (one-time setup)
python get_lastfm_session_key.py
```

## Configuration

Copy `config.example.yaml` → `config.yaml` (gitignored — never commit). Required fields:

```yaml
discogs:
  token: <your_discogs_api_token>
  username: <your_discogs_username>

audio:
  device_name: <sounddevice input device name>   # partial match OK

lastfm:                       # optional — omit to disable scrobbling
  api_key: ...
  api_secret: ...
  session_key: ...
```

## Architecture

### Async Event Pipeline

All I/O runs in a single `asyncio` event loop. Three coroutines are gathered in `main.py`:

```
AudioCapture.run()  →  SilenceDetector (sync, per-chunk)  →  RecognitionLoop.run()
                                  ↓
                           AudioEvent enum
                    (MUSIC_STARTED / MUSIC_STOPPED / SESSION_ENDED)
                                  ↓
                          ListenTracker.on_silence_event()
```

`AudioCapture` streams audio from `sounddevice`, calls `SilenceDetector.process()` synchronously on each chunk, and enqueues raw audio for `RecognitionLoop` via an `asyncio.Queue`.

### Recognition & Confirmation Gate

`RecognitionLoop` polls the queue and calls the active `RecognizerBackend` (default: `ShazamIOBackend`). `RecognizerBackend` is an ABC — swapping recognition providers means implementing one method: `recognize(audio, sample_rate) -> Optional[RawRecognitionResult]`.

Before committing a track, the loop requires `confirmation_required` (default: 2) consecutive identical results. A `None` result or a mismatch resets the pending counter. Commit calls `_commit_track()` which: sets raw state → resolves metadata → updates `PlayerState` and `ListenTracker` → scrobbles Last.fm.

### 3-Tier Metadata Resolution (`src/metadata/resolver.py`)

1. **Discogs collection search** — checks user's own collection first (fastest, richest side data)
2. **Discogs database search** — broader search if not in collection
3. **MusicBrainz fallback** — for albums not on Discogs

All paths return a `TrackMetadata` object. The resolver is instantiated in `main.py` and injected into `RecognitionLoop`.

### Core Data Models (`src/metadata/models.py`)

- **`TrackMetadata`** — central data carrier. Side-awareness properties (`side_letter`, `side_position`, `side_total`, `prev_track_title`, `next_track_title`) are computed from the tracklist. Cross-side boundary stitching: B1's `prev_track_title` returns the last track of Side A.
- **`PlaySession`** — maintained by `ListenTracker`. `log_track()` deduplicates and sets `potential_last_track`. The `album_release_id` is latched from the **first** Discogs-sourced track only (conservative: listening to Side A only never triggers a play-count update).
- **`DisplayPalette`** — 5 RGB tuples (bg, surface, accent, text, muted) extracted from album art via Pillow color quantization. Smooth lerp transition on track change.

### Listen Tracking (`src/tracking/listen_tracker.py`)

`on_silence_event` handles `MUSIC_STARTED` → `_start_session()` and `SESSION_ENDED` → `asyncio.create_task(_end_session())`.

`_end_session()` updates Discogs Play Count + Last Played **only if** `potential_last_track AND album_release_id` are both set. Last.fm love runs independently — a Discogs failure doesn't block it. Discogs calls run via `run_in_executor` (blocking client).

### Display (`src/display/`)

- **`layouts.py`** — pure geometry: `NowPlayingLayout` dataclass with named `Rect(x, y, w, h)` regions (header_strip, cover_art, track_text, divider, artist_text, album_text, genre_chips, meta_text, prev_next). All values relative to 1024×600 and scaled proportionally.
- **`renderer.py`** — all drawing logic (Pillow + pygame). Title claims natural height; secondary elements flow downward (dynamic push-down added in v1.2.1).

### Silence Detection (`src/audio/silence.py`)

RMS-based: `float(np.sqrt(np.mean(audio ** 2))) >= threshold`. Emits `SESSION_ENDED` after `session_end_silence_seconds` of continuous silence (configurable).

## Testing

208 tests, all async, using `pytest-asyncio` with `asyncio_mode = auto` (set in `pytest.ini`). Tests live in `tests/` and mirror the `src/` structure.

## GitHub Push Workflow

**Important:** the local git checkout is typically many commits behind the remote. Do NOT use local `git push`. All file changes must go through the GitHub API:

1. Fetch the current file SHA: `mcp__github__get_file_contents` (the `sha` field in the response)
2. Push the update: `mcp__github__create_or_update_file` with that SHA in the `sha` parameter

Omitting the SHA on an existing file will result in a conflict error.

## Roadmap

- **v1.3.1** (current): Async fix (`get_running_loop`), dependency updates
- **v1.4.0** (planned): Idle Screen & Recent Plays display
