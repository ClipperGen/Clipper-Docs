# Development Strategy & Tech Stack Detail
**Architecting for Performance, Scalability, and Cost-Efficiency**

Dokumen ini merinci pilihan teknologi dan metodologi pengembangan untuk membangun platform Clipper Gen yang robust.

## 1. Backend Architecture (The Hybrid Engine)
Kami memilih pendekatan **Hybrid Microservices** untuk memisahkan antara API Gateway yang efisien dan Heavy Video Processing yang padat komputasi.

*   **Main API (Gateway & CRUD)**: **Go (Golang)**. Dipilih karena konkurensi yang sangat tinggi (goroutines), penggunaan memori yang sangat rendah, dan performa REST API yang maksimal. Go menangani autentikasi, database CRUD, dan tokenomics.
*   **Worker Microservice (AI & Video)**: **Python 3.11+**. Dipilih karena ekosistem library AI (PyTorch, Transformers, FFmpeg-python) yang paling matang. Python berjalan murni sebagai worker tanpa layer HTTP HTTP/FastAPI.
*   **Task Queue**: **Celery + Redis**. Digunakan untuk mengelola antrean panjang pemrosesan video (Ingest, Analyze, Render). API Go mengirim pesan ke Redis yang kemudian dieksekusi oleh worker Python.
*   **Database**: 
    *   **PostgreSQL**: Untuk data relasional (User, Subscription, Transaction). Digunakan bersama (Shared DB) oleh Go dan Python.
    *   **Qdrant / Milvus (Vector DB)**: Untuk menyimpan embedding hasil transkripsi guna pencarian semantik yang cepat dan murah.

## 2. AI Intelligence Stack (The Brain)
Strategi kita adalah meminimalkan dependensi pada API pihak ketiga yang mahal (OpenAI/Claude) dengan menggunakan model lokal yang dioptimalkan.

*   **Transcription**: **Faster-Whisper** (di-host pada GPU instance). Memberikan kecepatan hingga 4-5x lipat dari Whisper standar.
*   **Semantic Search (Embedding)**: **BGE-Small** atau **all-MiniLM-L6-v2**. Model ringan yang berjalan sangat cepat di CPU/GPU untuk memfilter kandidat klip.
*   **Logical Analysis**: **GPT-4o (via API)** sebagai *final refiner*. Kita hanya mengirimkan "top candidate chunks" hasil embedding ke LLM untuk menekan biaya token hingga 80%.
*   **Visual Analysis**: **PySceneDetect (Content-Aware Detector)** terintegrasi via Python API.

## 3. Video Processing Stack (The Muscle)
Seluruh manipulasi video dilakukan secara native untuk efisiensi VRAM.

*   **Core Engine**: **FFmpeg 6.0+**.
*   **Binding**: **FFmpeg-python**.
*   **Strategy**:
    *   **Stream Copying**: Untuk slicing klip dasar (Lossless & Instant).
    *   **Hardware Acceleration**: Menggunakan `h264_nvenc` atau `hevc_nvenc` (NVIDIA) untuk rendering subtitle dan scaling portrait yang cepat.

## 4. Frontend & Identity Stack (The Interface)
Desain fokus pada kecepatan akses, keamanan data, dan feedback real-time.

*   **Authentication**: 
    *   **Users**: **Clerk-only**. Backend memverifikasi JWT via JWKS (issuer + key rotation) dan memetakan `sub` → `users.clerk_id`. Ini menyederhanakan integrasi dan mengurangi permukaan bug dibanding mendukung banyak IdP sekaligus.
    *   **Admin Access**: **Clerk-only**. Admin/Superadmin menggunakan JWT yang sama; backend menerapkan RBAC via `users.role` (mis. allowlist email admin/superadmin atau klaim/metadata Clerk).
*   **Framework**: **Next.js 14+ (App Router)**.
*   **Styling**: **Tailwind CSS** dengan **shadcn/ui**.

## 5. Infrastructure & Deployment
*   **Containerization**: **Docker & Docker Compose** untuk development. **Kubernetes (EKS/GKE)** untuk production.
*   **GPU Orchestration**: Menggunakan **NVIDIA Container Toolkit** untuk mengalokasikan resource GPU ke dalam container worker.
*   **Storage**: **Amazon S3** (Source & Output) dengan **CloudFront** (CDN).

## 6. Development Lifecycle (SDLC)
*   **TDD (Test Driven Development)**: Fokus pada unit test untuk logika pemotongan (timestamps alignment).
*   **CI/CD**: GitHub Actions untuk automated testing dan deploy ke staging environment.
*   **Observability**: **Prometheus & Grafana** untuk memantau suhu GPU, pemakaian VRAM, dan throughput video per menit.

