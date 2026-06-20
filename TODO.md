# Clipper Gen — Project TODO

Last updated: 20 June 2026 (Deep Audit)

---

## 🚨 CRITICAL — Fix Now

### C1. 🔑 Secret Keys Leaked in docker-compose.yml (BE/Infra)
**Severity: CRITICAL** — Anyone with repo access can use these.

| Leak | File |
|------|------|
| `CLERK_SECRET_KEY=sk_test_...` | root `docker-compose.yml:141`, `Clipper_Admin_FE/.env.local` |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...` | root `docker-compose.yml:131,140` |
| Hardcoded DB password `clipper` | root `docker-compose.yml:14` |

**Action**: Rotate all keys. Move to `.env` files or CI/CD secrets. Remove from repo.

### C2. 3 Celery Tasks Missing (Worker)
Go handlers dispatch tasks with NO Python consumer → **silent failure**

| Task | Called from Go | Status |
|------|---------------|--------|
| `generate_placeholder_shorts` | `project.go:860` | ❌ Missing |
| `generate_ffmpeg_shorts_dev` | `project.go:862` | ❌ Missing |
| `refine_locked_candidates` | `project.go:929` | ❌ Missing |

`GenerateShorts` and `UnlockMoreShorts` DO NOTHING. **Estimate**: 4h.

### C3. Auth Security Holes (BE)
| Issue | File:Line | Severity |
|-------|-----------|----------|
| SUPERADMIN via trusted host header | `auth.go:399` | 🔴 Critical |
| Dev auth bypass (`CLIPPER_DEV_DISABLE_AUTH=true`) impersonates via headers | `auth.go:308` | 🔴 Critical |
| JWKS lazy init on first request blocks | `auth.go:300` | 🟡 High |
| No token `exp` claim check | `auth.go` | 🟡 High |
| Race condition on user creation | `auth.go:183` | 🟡 High |

### C4. No Cloud Storage — Data Loss on Restart (BE/Worker)
- Uploads saved to ephemeral `uploads/{userID}/`
- Renders saved to ephemeral `exports/renders/`
- `app.Static("/storage/shorts", ...)` serves from local disk
- **Container restart = all files lost**

### C5. Root Docker Compose Issues (Infra)
| Issue | Detail |
|-------|--------|
| Port mismatch | API `8090:8000` but cloudflared expects `8005` |
| Port mismatch | User FE `8100` (caddy) but tunnel expects `3005` |
| Worker vs Go path mismatch | Worker writes `/app/exports/renders`, Go serves from `/app/storage/renders` |
| No restart policy on `api` | Won't recover from crash |
| Go version typo | Dockerfile: `go:1.25-alpine` — Go 1.25 doesn't exist |
| No monitoring services | Dashboards exist but Prometheus/Grafana NOT deployed |

### C6. Root Git Hygiene
- **No root `.gitignore`** — `memory.db`, build artifacts untracked
- **Nested repos NOT submodules** — `Clipper_FE/`, `Clipper_BE/`, `Clipper_Admin_FE/`, `Clipper_Docs/` invisible to clone
- **Go binary tracked** — `go-backend/api` (ELF binary) committed

---

## 🔴 HIGH — This Sprint

### H1. Missing API Endpoints (BE)
| Endpoint | Why |
|----------|-----|
| `GET /clips/:id` | Single clip detail (only list-all exists) |
| `PATCH /clips/:id` | Update clip metadata |
| `PATCH /projects/:id` | Update project title/visibility |
| `GET /subscriptions/tiers` | Public plan listing for pricing page |
| `GET /admin/users/:id` | User detail (admin only has list) |
| `GET /admin/dashboard/stats` | Aggregated platform analytics |
| `PATCH /projects/videos/:id` | Update video metadata |

### H2. Frontend CORS & Preload Warnings (FE)
- [x] Disable Cloudflare Analytics
- [x] Add `display: 'swap'` to Geist font
- [ ] Fix Cloudflare Insights beacon CORS (beacon.min.js)
- [ ] Fix woff2 font preload warnings
- [ ] Add YouTube domains to remotePatterns

### H3. FE: Broken Route + Runtime Bugs (FE)
**404**: `/dashboard/wallet` — sidebar links here, no page exists

**Runtime bugs:**
| Bug | File:Line | Impact |
|-----|-----------|--------|
| `hooks/index.ts` uses `export { default as X }` on **named** exports (5 re-exports) | All break at runtime | ❌ |
| `CollapsedSidebar` defined inside render function | `sidebar.tsx:490` | 🔄 State reset |
| `onboarding-tour` cascading renders | `onboarding-tour.tsx:136` | 📉 Perf |
| Referral section imported but never rendered | `landing-content.tsx` | 🧹 Dead code |

### H4. FE: Settings Page — Mostly Hardcoded (FE)
Only `user.name` and `user.email` dynamic. **Everything else hardcoded:**
- Plan badge ("Premium Agency"), billing info (price/date)
- Referral section (hardcoded code `'ALEX-CRE8'`)
- Notification preferences (useState, no persist)
- API keys (empty state), invoices (hardcoded)
- Danger zone delete (UI only, no API call)

### H5. FE: Analytics, Templates, My Styles — 100% Hardcoded (FE)
| Page | API Calls | Source |
|------|-----------|--------|
| `/dashboard/analytics` | ❌ None | Hardcoded Recharts mock data |
| `/dashboard/templates` | ❌ None | Hardcoded 12-item array, no API |
| `/dashboard/my-styles` | ❌ None | Local store only, no persist |

### H6. FE: Landing Page — 12 Sections All Mock (FE)
Hero, features-grid, how-it-works, pricing-table, testimonials, FAQ, stats-counter, feature-comparison, referral-section, creator-application-dialog, navbar — **all hardcoded mock data, zero API calls**

### H7. FE: Editor — UI Shell Only (FE)
| Feature | State |
|---------|-------|
| SubtitleEditor | 3 hardcoded mock subtitles, local state |
| VideoPreview | Fake 60s timer, no real video |
| StylePresets | 6 hardcoded presets, local state only |
| EditorTimeline | `mockTracks` (2 layers, 6 clips), local drag/resize |
| ExportProgressDialog | Fake `setInterval` progress |
| ExportSettingsDialog | Static UI, no-op export |
| `useFFmpeg` hook | **Stub** — `extractAudio()` and `getVideoMetadata()` return null |
| `useWhisper` hook | **Stub** — `isTranscribing`/`progress` not toggled correctly |

### H8. TS/Config Hidden Issues (FE)
- `typescript.ignoreBuildErrors: true` in next.config.ts — hides ALL TS errors
- `reactStrictMode: false` — hides double-render/stale closure bugs
- Fix hooks/index.ts, then enable both

### H9. Dual ORM Mismatch (BE/Worker)
| Issue | Impact |
|-------|--------|
| GORM AutoMigrate + Tortoise migrations on same tables | Schema drift |
| `MONTHLY_REFILL` exists in Go enums, missing in Python enums | Python can't parse these txns |
| `admin_audit_logs` table unknown to Tortoise | Python worker crashes if touching it |
| No migration sync strategy documented | |

### H10. Activities & Notifications Never Written (BE)
Tables + models + handlers exist. **NO code creates Activity or AppNotification records**. Entire feature dead.

### H11. Wallet: No Top-Up Flow (BE)
- `GetWallet` + `ListTransactions` are read-only
- No checkout/payment/webhook endpoints
- `TxTypeTopupPurchase` defined but never emitted
- Only way to add credits: `AdminGrantTokens`

### H12. Prometheus + Grafana Not Deployed (Infra)
Dashboards + alerting rules configured in `infra/grafana/` but:
- No Prometheus service in any docker-compose
- No Grafana service in any docker-compose
- Monitoring infrastructure = dead

### H13. Cloudflared Config Broken (Infra)
- API port mismatch: tunnel → `8005`, compose → `8090`
- User FE port mismatch: tunnel → `3005`, compose → `8100`
- Duplicated domains: `clipper-admin` AND `clipper_admin`
- Tunnel ID hardcoded (machine-specific)

### H14. CI/CD Pipeline
Zero `.github/workflows/` files. No automation. Manual push + compose up.

---

## 🟡 MEDIUM — Next Sprint

### M1. Zustand Store: Replace Mock Defaults (FE)
All default state is hardcoded mock:
- 4 fake projects, 8 fake activities, 6 fake transactions
- 5 fake notifications, 4 fake team members, 3 fake saved styles
- Missing: `onboardingComplete`, `lastActiveAt`, `preferredTheme`, `locale`, `currentClipId`, `logs`, `startedAt`

### M2. Input Validation + Rate Limiting (BE)
- No request validation middleware
- No rate limiting
- No request size limits (upload OOM risk)
- No CSRF protection
- Inconsistent error format: `"detail"` vs `"error"` vs `"status": "success"`

### M3. Refactor Duplicate Code (BE)
- `UploadProject` + `UploadProjectVideo` share ~70% identical code
- `ListAllClips` has duplicated count JOIN query
- `BurnCredits` + `CreditCredits` maintain separate balance logic

### M4. Race Condition Fixes (BE)
| Issue | File:Line |
|-------|-----------|
| Bonus burned before topup (should be opposite) | `tokenomics.go:37` |
| Balance check not locked | `tokenomics.go:30` |
| No `FOR UPDATE` on wallet query | `tokenomics.go` |
| `DeleteProject` never commits/rolls back on error | `project.go:738` |
| Credits burned before Celery dispatch — no rollback | `project.go:286` |

### M5. Payment Integration (BE)
- Stripe/LemonSqueezy/Paddle checkout endpoint
- Webhook handler for payment confirmation
- Auto-credit wallet on success
- Subscription tier sync

### M6. WebSocket Progress (BE + FE)
- Go: WebSocket endpoint `/ws/projects/:id/status`
- Python: publish to Redis pub/sub on state change
- Go: subscribe Redis → broadcast to WebSocket
- FE: replace 3s polling with WebSocket + reconnect

### M7. Thumbnail Generation (Worker)
After FFmpeg cut, extract frame at midpoint → set `short_thumbnail_url`

### M8. Video Caching (Worker)
- Cache YouTube downloads: `exports/sources/{video_id}.mp4`
- Check cache before download
- LRU eviction, skip >1GB

### M9. render_tasks.py Bug Fixes (Worker)
| Bug | Impact |
|-----|--------|
| `get_recommended_center()` called twice (lines 91, 138) | Double compute |
| Hardcoded `/app/exports/renders` path | Docker-only |
| Sets COMPLETED on failure (line 179) | Masks errors |
| `RuntimeError` on YouTube fail — no FAILED status | No recovery |
| `shutil.copy2` of GB files into memory | OOM risk |

### M10. Migrations Sync (Worker)
- Add Tortoise migration for `admin_audit_logs`
- Sync `MONTHLY_REFILL` enum to Python
- Consider disabling GORM AutoMigrate in production

### M11. Docker Config Fixes (Infra)
- Fix port mapping: cloudflared ↔ compose
- Fix Go version: `1.25` → actual version
- Fix render/audio path: worker ↔ Go
- Add restart policies to all services
- Add monitoring services (Prometheus + Grafana)

### M12. Root .gitignore
Create with: `go-backend/api`, `memory.db`, `*.log`, `.env`, `.env.*`, `node_modules/`, `__pycache__/`, `*.pyc`, `.DS_Store`, `uploads/`, `exports/`

### M13. Submodule Conversion or Clone Docs
4 nested repos need to be either proper submodules or documented:
- `Clipper_FE/` → `github.com/ClipperGen/Clipper-FE.git` (branch: wired)
- `Clipper_BE/` → `github.com/ClipperGen/clipper-be.git` (branch: wired)
- `Clipper_Admin_FE/` → `github.com/ClipperGen/Clipper-Admin-FE.git` (branch: wired)
- `Clipper_Docs/` → `github.com/ClipperGen/Clipper-Docs.git` (branch: master)

---

## 🔵 LOW — Backlog

| # | Item | Track | Est. |
|---|------|-------|------|
| L1 | Batch render API (`POST /projects/:id/render-all`) | BE + Worker | 2h |
| L2 | Clip reordering (drag + `PATCH` endpoint) | FE + BE | 4h |
| L3 | Referrer bonus grant cron | BE | 1h |
| L4 | Register data retention cron in `InitCron()` | BE | 30m |
| L5 | Public subscription tiers endpoint | BE | 1h |
| L6 | Clip detail endpoint (`GET /clips/:id`) | BE | 1h |
| L7 | Hook Audio TTS (Gemini/OpenAI for hook scene) | Worker | 1d |
| L8 | Remove compiled binary from git tracking | BE | 30m |
| L9 | Remove duplicate cloudflared domain entries | Infra | 15m |
| L10 | Consistent pagination metadata response format | BE | 2h |
| L11 | LLM provider: Claude/Ollama/Together AI support | Worker | 4h |
| L12 | Notification preferences persist to API | BE + FE | 2h |
| L13 | Team invite API wiring (store-only currently) | BE + FE | 3h |
| L14 | Dark mode toggle persist to API | BE + FE | 2h |
| L15 | Remove `NEXT_PUBLIC_DEV_AUTH` from production config | FE | 30m |
| L16 | Landing page referral section — render it or remove | FE | 30m |

---

## ✅ DONE

| # | Item | Track | Date |
|---|------|-------|------|
| 1 | Go Fiber: 31 API endpoints | BE | 31 May |
| 2 | 3 Celery tasks (process_video_ingest, analysis, render) | Worker | 31 May |
| 3 | All 12 DB models (auto-migrate via GORM) | BE | 31 May |
| 4 | FE API client + React Query hooks for all endpoints | FE | 16 Jun |
| 5 | Zustand store hydration from API | FE | 16 Jun |
| 6 | File upload dialog | FE | 16 Jun |
| 7 | Quick Actions URL import | FE | 16 Jun |
| 8 | Editor + Project Details pages | FE | 16 Jun |
| 9 | Docker images (Go API, FE, Admin FE, Worker) | Infra | 16 Jun |
| 10 | Unified docker-compose.yml | Infra | 16 Jun |
| 11 | `GET /clips` endpoint | BE | 16 Jun |
| 12 | ClipsLibrary wired to API | FE | 16 Jun |
| 13 | Reports translated to Indonesian | Docs | 17 Jun |
| 14 | Admin FE rebuild — 5 pages, clean build | FE Admin | 17 Jun |
| 15 | DELETE clips (BE + FE) | BE + FE | 17 Jun |
| 16 | Dashboard audit log real-time | BE + FE | 17 Jun |
| 17 | Server-side search (ILIKE) | BE + FE | 17 Jun |
| 18 | CSV export | FE Admin | 17 Jun |
| 19 | Bulk activate/revoke | BE + FE Admin | 17 Jun |
| 20 | Complete audit logging | BE | 17 Jun |
| 21 | Heatmap testing & ingestion report | Docs | 19 Jun |
| 22 | Phase 1: Metadata-only ingestion | BE + Worker | 19 Jun |
| 23 | Phase 2: Whisper WASM + ffmpeg.wasm hooks (stubs) | FE | 19 Jun |
| 24 | Phase 3: YouTubeEmbed → ClipsLibrary | FE | 19 Jun |
| 25 | Phase 4: Auto-generate clips from heatmap (verified) | Worker + FE | 19 Jun |
| 26 | Notification mark-as-read (BE + FE) | BE + FE | 19 Jun |
| 27 | Grafana dashboards + alerting rules (config only) | Infra | 19 Jun |

---

## 📊 Summary

| Metric | Count |
|--------|-------|
| Total items | 65 |
| 🔴 Critical | 6 |
| 🔴 High | 14 |
| 🟡 Medium | 13 |
| 🔵 Low | 16 |
| ✅ Done | 27 |

---

_Contact: Mema (Frea)_
