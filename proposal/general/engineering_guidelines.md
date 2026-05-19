# Engineering Guidelines & Development Strategy
**Standard Operating Procedures (SOP) for the Clipper Gen Dev Team**

Dokumen ini mendefinisikan standar teknis, alur kerja kolaborasi, dan strategi pengembangan untuk memastikan kode yang dihasilkan bersifat *maintainable*, *scalable*, dan *secure*.

## 1. Coding Standards & Style Guides
Setiap pengembang wajib mengikuti standar berikut:

*   **Backend (Python/FastAPI)**:
    *   Mengikuti **PEP 8** (Style Guide for Python Code).
    *   Wajib menggunakan **Type Hints** untuk semua fungsi dan variabel.
    *   Linter & Formatter: `ruff` (untuk kecepatan) atau `flake8` + `black`.
*   **Frontend (TypeScript/Next.js)**:
    *   Wajib menggunakan **TypeScript** (No `any` allowed).
    *   Komponen berbasis **Functional Components** dengan React Hooks.
    *   Linter & Formatter: `ESLint` + `Prettier`.
*   **API Design**:
    *   Wajib menggunakan standar **RESTful API**.
    *   Dokumentasi otomatis via **Swagger/OpenAPI** (FastAPI default).
    *   Setiap endpoint wajib memiliki skema request/response menggunakan **Pydantic** (Backend) dan **Zod** (Frontend).

## 2. Git Flow & Branching Strategy
Kami menggunakan model **Feature Branching** untuk menjaga stabilitas branch utama.

*   **`master`**: Branch produksi (Stable & Protected). Hanya menerima merge dari `develop`.
*   **`develop`**: Branch integrasi fitur baru.
*   **`feature/[nama-fitur]`**: Branch pengerjaan fitur baru.
*   **`bugfix/[nama-bug]`**: Branch untuk perbaikan bug.
*   **Pull Request (PR) Rules**:
    *   Minimal 1 *Peer Review* sebelum merge ke `develop`.
    *   Wajib melewati CI Checks (Linting & Unit Tests).
    *   Pesan commit wajib mengikuti **Conventional Commits** (e.g., `feat: ...`, `fix: ...`, `docs: ...`).

## 3. Testing Strategy
Kualitas kode divalidasi melalui tiga lapisan pengujian:

*   **Unit Testing**: Menguji fungsi individual (Backend: `pytest`, Frontend: `vitest`). Target cakupan: **80% coverage**.
*   **Integration Testing**: Menguji komunikasi antar komponen (e.g., API ke Database).
*   **E2E Testing (End-to-End)**: Menguji alur pengguna secara utuh (YouTube link -> Analysis -> Export) menggunakan **Playwright** atau **Cypress**.

## 4. Access Control Strategy (Hybrid Security)
Kami menerapkan tiga lapis keamanan untuk melindungi sumber daya dan integritas sistem:

*   **RBAC (Role-Based Access Control)**:
    *   Digunakan untuk mendefinisikan izin statis berdasarkan profil pengguna.
    *   Roles: `SUPERADMIN`, `ADMIN`, `USER`.
    *   Contoh: Hanya `SUPERADMIN` yang dapat mengakses logs sistem global dan manajemen database admin.
*   **ABAC (Attribute-Based Access Control)**:
    *   Digunakan untuk otorisasi dinamis berdasarkan atribut objek dan pengguna.
    *   Atribut: `subscription_tier`, `video_visibility`, `owner_id`, `current_time`.
    *   Contoh: Pengguna hanya dapat melakukan "Direct Upload" jika `user.subscription_tier == 'PREMIUM'`.
*   **Token-Guarded Access**:
    *   Setiap request pemrosesan berat (Transcription, LLM, Rendering) wajib melewati middleware validator token.
    *   Logika: `Request Proceed IF (user.token_balance >= resource.cost)`.

## 4.1 Identity Provider Standard
Untuk menyederhanakan implementasi dan mengurangi kompleksitas operasional:
*   **User Authentication**: **Clerk-only** (JWT Bearer). Backend wajib memverifikasi token menggunakan JWKS dan memetakan `sub` → `users.clerk_id`.
*   **Internal Admin Authentication**: Tetap terpisah menggunakan tabel `internal_admins` dan JWT internal.

## 5. Environment & Containerization
Memastikan konsistensi antar lingkungan pengembangan (Local, Staging, Prod).

*   **Local Setup**: Wajib menggunakan **Docker Compose**. Satu perintah (`docker-compose up`) harus bisa menjalankan seluruh stack (API, DB, Redis, Worker).
*   **Configuration**: Semua rahasia dan konfigurasi wajib ditaruh di `.env` (Jangan pernah commit file `.env`).
*   **CI/CD**: Setiap push ke `develop` akan otomatis di-deploy ke **Staging Environment** untuk pengujian internal.

## 5. Technical Debt & Documentation
*   **Issue Tracking**: Menggunakan GitHub Issues dengan label yang jelas (`bug`, `enhancement`, `high-priority`).
*   **Code Documentation**: Setiap fungsi kompleks wajib memiliki *docstring* yang menjelaskan parameter dan return value.
*   **Architecture Decision Record (ADR)**: Setiap perubahan besar pada arsitektur (e.g., ganti Vector DB) wajib didokumentasikan alasannya.

## 6. Development Roadmap (Technical Evolution)
*   **Sprint 1-4**: Fokus pada MVP (Core AI Pipeline + Basic UI).
*   **Sprint 5-8**: Optimasi Latency (GPU batching) & Implementasi Tokenomics.
*   **Sprint 9+**: Eksperimen model lokal (Llama 3) & Fitur advanced (AI B-roll).

