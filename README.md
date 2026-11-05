# MediaFlow — Local Video & Audio Downloader

A local React + FastAPI app. Paste a **public** supported media URL, review the formats that yt-dlp actually finds, then choose a video quality or audio download. Video heights and audio formats are returned from the media page; unavailable options are not shown.

> Use only for media you own or are authorized to download. This app does not bypass private-account access, logins, DRM, or other restrictions. Platform access can change, and some links may be unavailable because of region, account, or platform rules.

## Easiest setup on Windows

1. Install **Node.js 22.12 or newer** and **Python 3.10 or newer**.
2. Optional but recommended: install FFmpeg (including `ffprobe`) and make sure both are on `PATH`. FFmpeg enables combining separate video/audio streams and MP3 conversion. Without it, the app shows only formats the browser-compatible single-file workflow can use, and audio stays in its original format. See [FFmpeg downloads](https://ffmpeg.org/download.html).
3. Open the extracted project folder in VS Code.
4. Double-click `start-local.bat` (or run it from a terminal). On first run it creates a Python virtual environment and installs the listed dependencies, then opens separate backend and frontend terminals.
5. Open **http://127.0.0.1:5173**. The backend health route is **http://127.0.0.1:8000/api/health** and API documentation is **http://127.0.0.1:8000/docs**. Health JSON reports the installed yt-dlp version and whether `curl-cffi` is available.

Keep both terminal windows open while using the app. Use Ctrl+C in each terminal to stop its server.

## Manual start (Windows, macOS, or Linux)

Terminal 1 — backend:

```bash
cd Mediaflow-backend
python -m venv .venv
# Windows PowerShell: .\.venv\Scripts\Activate.ps1
# macOS/Linux:       source .venv/bin/activate
python -m pip install -r requirements.txt
python -m uvicorn main:app --host 127.0.0.1 --port 8000 --reload
```

Terminal 2 — frontend, from the project root:

```bash
npm ci
npm run dev
```

Vite proxies `/api` calls to the local FastAPI server; the app does not hard-code the API request URLs. Do not expose this development server to a public network.

## Tests and build

Frontend:

```bash
npm test
npm run lint
npm run build
```

Backend (inside the activated virtual environment):

```bash
cd Mediaflow-backend
python -m pip install -r requirements-dev.txt
python -m pytest -q
```

The backend tests mock media extraction/downloads, so they do not fetch any third-party video.

## Troubleshooting

- **Backend will not start:** verify Python 3.10+, then run `python -m pip install -r Mediaflow-backend/requirements.txt` in the backend virtual environment.
- **Frontend reports backend connection errors:** keep both servers running and check `/api/health`. Port 8000 is for FastAPI; port 5173 is for Vite.
- **FFmpeg missing:** the page lists fewer quality options and audio may be M4A/WebM rather than MP3. Install FFmpeg and `ffprobe`, restart the backend terminal, and analyze the URL again.
- **Download start or speed:** the backend reuses the analyzed format choice instead of making a redundant metadata-only lookup, and segmented HLS/DASH downloads use up to 8 concurrent fragments. Direct media streams and the source website's response speed can still be the limiting factor.
- **TikTok, Instagram, or Facebook won't fetch:** run `start-local.bat` again so the backend installs the current yt-dlp stable version with its `curl-cffi` impersonation support, then analyze the link again. Confirm `/api/health` reports `curl_cffi_available: true`. Upstream notes this dependency may be needed by sites using TLS fingerprinting; do not force a client impersonation profile because it can reduce stability or speed ([yt-dlp docs](https://github.com/yt-dlp/yt-dlp#impersonation)). The link must be public and accessible from your device and region. Login-only, private, DRM-protected, or region-blocked media cannot be fetched by this app.
- **A platform extractor changes:** yt-dlp updates its site extractors regularly. Inside `Mediaflow-backend`, run `python -m pip install -r requirements.txt`, then restart FastAPI.
- Completed server-side temporary files are removed after the response is sent.
