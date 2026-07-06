# Bunkr Downloader

**Fast, reliable CLI for downloading Bunkr albums and individual files.**

This CLI tool (`bunkr`) resolves and signs CDN URLs, downloads albums concurrently with adaptive chunking and resumable transfers, tracks offline servers, and shows a live terminal UI.

## ✨ Features

*   **⚡️ Fast & Concurrent:**
    *   **Adaptive Concurrency:** Worker pool that backs off on HTTP 429 and ramps back up automatically.
    *   **Resumable Downloads:** Interrupted transfers continue via HTTP `Range` instead of restarting.
    *   **Optional Rate Limiting:** Cap total download speed with `--rate-limit`.
*   **📦 Flexible Input:**
    *   **Single URL:** Download one file or an entire album (with pagination — every page is crawled).
    *   **Batch Mode:** Process a list of URLs from `URLs.txt` automatically.
    *   **Retry Failed:** Re-queue last run's failures with `--retry-failed`.
*   **🎯 Selective Downloads:**
    *   **Filename Filters:** Use `--ignore` / `--include` to skip or target specific files.
    *   **Dry Run:** Preview what would download without writing anything.
*   **📊 Live Progress:**
    *   **Interactive TUI:** Real-time speed, ETA, and byte progress (bubbletea).
    *   **Plain Mode:** `--disable-ui` for simple log output.
    *   **JSON Mode:** `--json` for NDJSON event streaming (scripting-friendly).
*   **🛡 Resilient:**
    *   **Offline Server Detection:** Automatically skips dead CDN nodes via the Bunkr status page.
    *   **Disk Space Precheck:** Warns before filling up your drive.
    *   **Session Logging:** Failed items logged to `session.log` for later retry.

---

## 📦 Installation

This tool is available via a public tap for macOS and Linux (Homebrew) and Windows (Scoop).

### 🍺 macOS / Linux (Homebrew)

```bash
# 1. Add the tap
brew tap yasithab/homebrew-taps

# 2. Install the tool
brew install bunkr
```

**To Update:**
```bash
brew upgrade bunkr
```

### 🪟 Windows (Scoop)

```powershell
# 1. Add the bucket
scoop bucket add yasithab https://github.com/yasithab/homebrew-taps

# 2. Install the tool
scoop install bunkr
```

**To Update:**
```powershell
scoop update bunkr
```

---

## 🎮 Usage Guide

### 1. Single Download

Download an album or individual file by passing the URL directly.

```bash
# Album
bunkr https://bunkr.si/a/PUK068QE

# Single file
bunkr https://bunkr.fi/f/gBrv5f8tAGlGW
```

### 2. Batch Download

Create a `URLs.txt` file with one URL per line, then run with no arguments.

```bash
bunkr
```

`URLs.txt` is backed up to `Backups/URLs_<timestamp>.bak` before the run and cleared afterward.

### 3. Selective Downloads

Use filters to download only what you need.

```bash
# Skip zip and rar files
bunkr https://bunkr.si/a/PUK068QE --ignore .zip --ignore .rar

# Only download files matching a keyword
bunkr https://bunkr.si/a/PUK068QE --include FullSizeRender
```

### 4. Retry Failures

If some downloads failed, retry them from the session log.

```bash
bunkr --retry-failed
```

---

## ⚙️ Configuration Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--custom-path <dir>` | (cwd) | Save downloads under `<dir>/Downloads`. |
| `--no-download-folder` | `false` | Skip the `Downloads` subfolder. |
| `--disable-ui` | `false` | Plain log output instead of the live TUI. |
| `--disable-disk-check` | `false` | Skip the free-space precheck. |
| `--max-retries <n>` | `5` | Max retries per media file. |
| `--workers <n>` | `3` | Concurrent downloads per album. |
| `--ignore <substr>` | — | Skip files whose names contain this substring (repeatable). |
| `--include <substr>` | — | Only download files whose names contain this substring (repeatable). |
| `--rate-limit <size>` | unlimited | Cap total download speed, e.g. `5MB`, `500KB`. |
| `--dry-run` | `false` | Resolve and list what would download; write nothing. |
| `--json` | `false` | Emit NDJSON events to stdout (implies `--disable-ui`). |
| `--retry-failed` | `false` | Re-download items that failed in the last `session.log`. |
| `--version` | — | Print version and exit. |

---

## 🌐 Environment Variables

Bunkr periodically migrates domains. The endpoints can be overridden without rebuilding:

| Variable | Default |
|----------|---------|
| `BUNKR_STATUS_URL` | `https://status.bunkr.ru/` |
| `BUNKR_SIGN_API` | `https://glb-apisign.cdn.cr/sign` |
| `BUNKR_DOWNLOAD_API` | `https://dl.bunkr.cr/api/_001_v2` |
| `BUNKR_DOWNLOAD_REFERER` | `https://get.bunkrr.su/` |
| `BUNKR_FALLBACK_DOMAIN` | `bunkr.cr` |

---

## 🔍 Examples

**Scenario 1: Download an Album with Rate Limiting**
```bash
bunkr https://bunkr.si/a/PUK068QE --rate-limit 5MB
```

**Scenario 2: High Concurrency with Fewer Retries**
```bash
bunkr https://bunkr.si/a/PUK068QE --workers 6 --max-retries 3
```

**Scenario 3: Save to External Drive**
```bash
bunkr --custom-path /Volumes/External
```

**Scenario 4: Dry Run (Preview Only)**
```bash
bunkr https://bunkr.si/a/PUK068QE --dry-run
```

**Scenario 5: JSON Output for Scripting**
```bash
bunkr https://bunkr.si/a/PUK068QE --json 2>/dev/null | jq .
```

---

## ❓ Troubleshooting

**Downloads stuck or very slow**
*   **Fix:** The CDN may be rate-limiting. Try reducing workers: `--workers 1`. The adaptive concurrency will handle 429s automatically.

**"disk space" warning**
*   **Fix:** Free up space or use `--disable-disk-check` to skip the precheck. Alternatively, use `--custom-path` to point to a drive with more space.

**Some files keep failing**
*   **Fix:** The CDN node may be offline. The tool automatically detects offline servers via the status page. Try again later or use `--retry-failed` to re-queue only the failures.

**No output / blank screen**
*   **Fix:** The interactive TUI requires a terminal. Use `--disable-ui` for plain log output, or `--json` for machine-readable events.
