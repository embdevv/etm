# `etm` — eat the media
> **by embdev** · CLI media player for your terminal

```
███████╗████████╗███╗   ███╗
██╔════╝╚══██╔══╝████╗ ████║
█████╗     ██║   ██╔████╔██║
██╔══╝     ██║   ██║╚██╔╝██║
███████╗   ██║   ██║ ╚═╝ ██║
╚══════╝   ╚═╝   ╚═╝     ╚═╝
```

Search, stream, and subtitle movies and TV shows — entirely from your terminal.
Powered by **TMDB**, **SubDL**, **VidKing**, and **mpv**.

---

## Features

| | |
|---|---|
| 🔍 **Search** | Find movies & TV shows by title via TMDB |
| 📺 **Stream** | HLS streams via scraper + automatic VidKing fallback |
| 💬 **Subtitles** | Fetch `.srt`/`.ass` from SubDL, load up to 8 tracks at once |
| 📋 **History** | Auto-saved watch history with resume support |
| ⭐ **Favorites** | Save and launch titles from a personal watchlist |
| 📦 **Portable** | Compiles to a standalone `.exe` with mpv bundled |

---

## Quick Start

### 1. Install dependencies

```bash
pip install requests playwright
python -m playwright install chromium
```

Install [mpv](https://mpv.io) and make sure it's on your `PATH`.

### 2. Get your API keys

| Service | Link | Cost |
|---|---|---|
| TMDB | https://www.themoviedb.org/settings/api | Free |
| SubDL | https://subdl.com/api | Free |

### 3. Configure

**Option A — Setup wizard (recommended):**
```bash
python setup.py
```
Walks you through entering your keys and writes `config.py` automatically.

**Option B — Edit `config.py` directly:**
```python
API_KEY        = "your_tmdb_key"
SUBDL_API_KEY  = "your_subdl_key"
VIDKING_BASE   = "https://vidking.net"
```

**Option C — Environment variables:**
```bash
# Linux / macOS
export TMDB_API_KEY="your_tmdb_key"
export SUBDL_API_KEY="your_subdl_key"

# Windows (PowerShell)
$env:TMDB_API_KEY="your_tmdb_key"
$env:SUBDL_API_KEY="your_subdl_key"
```

> `config.py` is gitignored. Your keys never get committed.

### 4. Run

```bash
python main.py
```

If `config.py` is missing, the setup wizard runs automatically on first launch.

---

## Usage

```
Menu
════════════════════════════════════════════════════
  1.  Search and play
  2.  Watch history
  3.  Favorites + watchlist
  q.  Quit
```

- **Search** → type a title → pick movie or TV → select result → pick stream source → pick subtitles → watch
- **TV shows** → prompted for season/episode → auto-prompts to continue to the next episode after playback
- **Subtitles** → choose a specific release, load up to 8 tracks (cycle with `j`/`J` in mpv), or skip
- **History** → resume with saved stream URL or start over with a fresh fetch

---

## Building a Standalone Executable (Windows)

Place `mpv.exe` in the project folder, then:

```powershell
pyinstaller --onefile --console --name etm --add-binary "mpv.exe;." main.py

robocopy "$env:LOCALAPPDATA\ms-playwright\chromium-1208" "dist\browsers\chromium-1208" /E
robocopy "$env:LOCALAPPDATA\ms-playwright\chromium_headless_shell-1208" "dist\browsers\chromium_headless_shell-1208" /E
```

The compiled `etm.exe` resolves mpv and Playwright browsers relative to its own location — no separate installs needed.

> **Note:** The `chromium-1208` folder name may change when Playwright updates.  
> Check `$env:LOCALAPPDATA\ms-playwright\` for the current name if the build fails.

---

## Security

All outbound requests go through `security.py`:

- **Allowlist-only** — requests permitted only to `api.themoviedb.org`, `vidking.net`, `api.subdl.com`, `dl.subdl.com`
- **HTTPS only** — HTTP URLs are rejected outright
- **Redirect validation** — redirect destinations are re-checked against the allowlist
- **Response size cap** — responses over 10 MB are aborted; subtitle zips capped at 12 MB
- **Zip path traversal protection** — archive entries with `..`, leading `/`, or `:` are skipped
- **Filename sanitization** — subtitle filenames are stripped to alphanumeric characters only

---

## File Structure

```
etm/
├── main.py         # entry point, CLI menus, history, favorites
├── config.py       # your API keys (gitignored)
├── setup.py        # first-run setup wizard
├── security.py     # URL allowlist, HTTPS enforcement, size caps
├── tmdb.py         # TMDB search
├── scraper.py      # primary HLS stream scraper (Playwright)
├── vidking.py      # VidKing fallback scraper
├── subtitles.py    # SubDL subtitle fetching and extraction
└── player.py       # mpv launcher with integrity check
```

---

## Requirements

- Python 3.8+
- mpv — on PATH, or bundled as `mpv.exe` for Windows builds
- TMDB API key (free)
- SubDL API key (free)

---

## Disclaimer

Stream sources depend on third-party services that may change or go offline.  
`VIDKING_BASE` is configurable in `config.py` or via environment variable if the domain changes.  
etm is intended for **personal use only**.
