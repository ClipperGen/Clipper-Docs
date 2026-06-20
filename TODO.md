# Clipper Gen — Project TODO

Last updated: 20 June 2026

---

## Status Overview

| Area | Progress |
|------|----------|
| **Go Backend (API)** | 31/45 endpoints done |
| **Python Worker** | 3/6 Celery tasks done — 3 CRITICAL missing |
| **FE User** | Core pages wired, some hardcoded settings remain |
| **FE Admin** | All 5 pages wired |
| **Infrastructure** | Docker compose done, monitoring infra incomplete |
| **CI/CD** | ❌ Not started |
| **Git Hygiene** | ❌ No root .gitignore, nested repos not submodules |

---

## ✅ DONE (27 items)

| # | Item | Track | Completed |
|---|------|-------|-----------|
| 1 | Go Fiber: 31 API endpoints (users, projects, wallet, clips, misc, admin) | BE | 31 May |
| 2 | Python Celery: `process_video_ingest`, `process_video_analysis`, `render_generated_short` | Worker | 31 May |
| 3 | All 12 DB models (auto-migrate via GORM) | BE | 31 May |
| 4 | FE API client + React Query hooks | FE | 16 Jun |
| 5 | Zustand store hydration from API | FE | 16 Jun |
| 6 | File upload dialog wired | FE | 16 Jun |
| 7 | Quick Actions URL import wired | FE | 16 Jun |
| 8 | Editor page + Project Details page | FE | 16 Jun |
| 9 | Docker images (Go API, FE, Admin FE, Worker) | Infra | 16 Jun |
| 10 | Unified docker-compose.yml | Infra | 16 Jun |
| 11 | `GET /clips` endpoint | BE | 16 Jun |
| 12 | ClipsLibrary wired to real API | FE | 16 Jun |
| 13 | Reports translated to Indonesian | Docs | 17 Jun |
| 14 | Admin FE rebuild (5 pages wired) | FE Admin | 17 Jun |
| 15 | DELETE clips (BE + FE) | BE + FE | 17 Jun |
| 16 | Dashboard audit log real-time | BE + FE | 17 Jun |
| 17 | Server-side search (ILIKE) | BE + FE | 17 Jun |
| 18 | CSV export | FE Admin | 17 Jun |
| 19 | Bulk activate/revoke | BE + FE Admin | 17 Jun |
| 20 | Complete audit logging | BE | 17 Jun |
| 21 | Heatmap testing & ingestion report | Docs | 19 Jun |
| 22 | Phase 1: Metadata-only ingestion | BE + Worker | 19 Jun |
| 23 | Phase 2: Whisper WASM + ffmpeg.wasm hooks | FE | 19 Jun |
| 24 | Phase 3: YouTubeEmbed → ClipsLibrary | FE | 19 Jun |
| 25 | Phase 4: Auto-generate clips from heatmap (verified) | Worker + FE | 19 Jun |
| 26 | Notification mark-as-read (BE + FE) | BE + FE | 19 Jun |
| 27 | Grafana dashboards + alerting rules | Infra | 19 Jun |

---

## 🔴 CRITICAL — Blocking pipeline

### C1. Missing Celery Tasks (Worker)
Go handlers dispatch 3 tasks that have NO Python consumer → silent failure.

| Task | Called by | File |
|------|-----------|------|
| `generate_placeholder_shorts` | `GenerateShorts` handler | `project.go:860` |
| `generate_ffmpeg_shorts_dev` | `GenerateShorts` (ffmpeg mode) | `project.go:862` |
| `refine_locked_candidates` | `UnlockMoreShorts` handler | `project.go:929` |

**Action**: Implement 3 task functions in `processing-engine/app/workers/tasks.py`

**Estimate**: 4 hours

### C2. Git Hygiene — Root .gitignore & Submodules
- No root `.gitignore` → `memory.db` and build artifacts untracked
- `Clipper_FE/`, `Clipper_BE/`, `Clipper_Admin_FE/`, `Clipper_Docs/` are nested repos, NOT submodules → invisible to clone

**Action**:
  - Create root `.gitignore` (Go binaries, `memory.db`, `*.log`, `.env`, `node_modules/`, `__pycache__/`, etc.)
  - Convert nested repos to proper submodules OR document clone workflow

**Estimate**: 2 hours

### C3. CI/CD Pipeline
No automation exists. Manual push + docker compose up.

**Action**: Create `.github/workflows/deploy.yml` with:
  - Lint + typecheck
  - Build Docker images
  - Push to registry
  - SSH deploy to server

**Estimate**: 4 hours

---

## 🟡 HIGH — This sprint

### H1. Frontend CORS & Preload Warnings (FE)
- [x] Disable Cloudflare Analytics in next.config.ts
- [x] Add `display: 'swap'` to Geist font preloads
- [ ] Investigate Cloudflare Insights beacon CORS issue
- [ ] Fix woff2 font preload warnings
- [ ] Add YouTube domains to remotePatterns

**Estimate**: 1 hour

### H2. Analytics Pipeline (BE + Worker)
Clips show 0 for social metrics. Needs:
- New `clip_analytics` table
- Celery task to pull from YouTube Data API / TikTok API / Instagram Graph API
- `GET /clips/:id/analytics` endpoint
- Update FE `ApiClip` + ClipsLibrary

**Depends on**: Social platform API keys. **Blocked** until available.

**Estimate**: 1 day

### H3. WebSocket Progress (BE + FE)
Current: 3s polling via `refetchInterval`. Needs:
- Go: WebSocket endpoint (`/ws/projects/:id/status`)
- Python: publish status updates to Redis pub/sub
- Go: subscribe Redis → broadcast to WebSocket clients
- FE: replace polling with WebSocket + reconnection logic

**Depends on**: Redis pub/sub (already running)

**Estimate**: 1 day

### H4. Object Storage R2/S3 (BE)
Uploads saved to local `uploads/`, renders in `exports/renders/` — both lost on restart. Needs:
- Cloudflare R2 or S3 bucket setup
- Replace `os.Create("uploads/...")` with R2 `PutObject`
- Replace `open("exports/renders/...")` with R2 upload
- Signed URL generation
- Migration path for existing files

**Priority reduced** by WASM architecture, but still needed for user uploads.

**Estimate**: 1-2 days

### H5. Settings Page Wiring (FE)
Hardcoded replacements needed:
- Plan badge ("Premium Agency") → real subscription data
- Billing info (price, date) → `useTransactions()`
- Referral section → `useReferralStats()`
- Social connections → placeholder (no API yet)
- Notification preferences → store in `system_configs`

**Estimate**: 3-4 hours

### H6. Prometheus + Grafana Services (Infra)
Grafana dashboards configured but Prometheus & Grafana services NOT in docker-compose. Monitoring dead on arrival.

**Action**: Add `prometheus` and `grafana` services to `docker-compose.yml`

**Estimate**: 2 hours

### H7. Activities & Notifications Never Written (BE)
Tables + models + handlers exist, but NO code ever creates Activity or AppNotification records. Feature is dead.

**Action**: Add `CreateActivity()` calls at key action points (project create, clip generate, render complete, etc.)

**Estimate**: 3 hours

---

## 🟢 MEDIUM — Next sprint

| # | Item | Track | Est. | Depends on |
|---|------|-------|------|------------|
| M1 | Thumbnail generation (extract frame midpoint in render pipeline) | Worker | 2-3h | — |
| M2 | Video caching (LRU: cache downloads, eviction policy) | Worker | 2-3h | — |
| M3 | Input validation middleware | BE | 2h | — |
| M4 | Rate limiting middleware | BE | 2h | — |
| M5 | Referrer bonus grant cron | BE | 1h | — |
| M6 | Public subscription tiers endpoint (`GET /subscriptions/tiers`) | BE | 1h | — |
| M7 | Landing page: hero section login button (currently broken) | FE | 1h | — |
| M8 | Template hub page wiring | FE | 3h | — |
| M9 | My Styles page wiring | FE | 2h | — |

---

## 🔵 LOW — Backlog

| # | Item | Track | Est. |
|---|------|-------|------|
| L1 | Batch render API (`POST /projects/:id/render-all`) | BE + Worker | 2h |
| L2 | Clip reordering (drag-to-reorder + `PATCH` endpoint) | FE + BE | 4h |
| L3 | Payment integration (Stripe/Xendit checkout + webhook) | BE | 2-3d |
| L4 | Data retention cron (exists but not registered) | BE | 30m |
| L5 | Refactor duplicate upload code (UploadProject == UploadProjectVideo) | BE | 1h |
| L6 | Clip detail endpoint (`GET /clips/:id`, `PATCH /clips/:id`) | BE | 1h |
| L7 | Hook Audio TTS (Gemini/OpenAI TTS for hook scene) | Worker | 1d |
| L8 | Compiled binary in repo: add to .gitignore + remove from tracking | BE | 30m |
| L9 | `POST /exports` dedicated export endpoint | BE | 2h |

---

## 📋 Track Assignments

| Track | Person | Items |
|-------|--------|-------|
| **BE** (Go) | | C1 (part), H2 (part), H3 (part), H7, M3-6, L1 (part), L3-6, L8-9 |
| **Worker** (Python) | | C1, H2 (part), M1-2, L1 (part), L7 |
| **FE** (Next.js) | | H1, H5, H3 (part), M7-9, L2 (part) |
| **FE Admin** (Next.js) | | — (all done) |
| **Infra** (Docker/DevOps) | | C2-3, H6 |
| **Docs** | | Move TODO to docs, commit + push |

---

## 🚫 Blocked / Deferred

- **YouTube Bot Detection** — ✅ Resolved by WASM architecture (no yt-dlp on server)
- **`uploads/` directory** — Will be resolved by H4 (Object Storage)
- **Analytics Pipeline (H2)** — Blocked until social platform API keys obtained
- **Payment Integration (L3)** — Blocked until business requirements finalized

---

_Contact: Mema (Frea)_
