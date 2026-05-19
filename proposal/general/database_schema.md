# Database Architecture Design
**Scalable Schema for AI Video Operations & Third-Party Auth**

Dokumen ini merinci arsitektur data Clipper Gen yang dioptimalkan untuk performa tinggi, manajemen token yang presisi, dan integrasi autentikasi modern.

## 1. Authentication & Identity Layer
Kami menggunakan pendekatan **Hybrid Auth Architecture**:
*   **Users (SSO-First)**: **Clerk-only**. Backend memverifikasi `Authorization: Bearer <JWT>` melalui **JWKS** (issuer + key rotation). Identitas dipetakan ke `users.clerk_id`.
*   **Admin/Superadmin**: Tetap **Clerk-only**. Peran ditentukan di backend melalui atribut `users.role` (RBAC) yang diturunkan dari aturan organisasi (misalnya allowlist email admin/superadmin atau klaim/metadata Clerk).

## 2. Core Entity Relationship (ERD)

### **Table: `users`**
*   `id` (UUID, PK)
*   `clerk_id` (String, Unique, Index)
*   `google_id` (String, Unique, Index, Nullable) - tidak digunakan jika mode Clerk-only.
*   `email` (String, Unique)
*   `full_name` (String)
*   `role` (Enum: USER, ADMIN, SUPERADMIN)
*   `is_active` (Boolean) - Untuk pembekuan akun (Admin action).
*   `subscription_tier_id` (FK)
*   `subscription_expiry` (DateTime, Nullable)
*   `referral_code` (String, Unique)
*   `referred_by` (UUID, FK -> users.id, Nullable)
*   `created_at` (DateTime)

### **Table: `wallets`**
*   `id` (UUID, PK)
*   `user_id` (UUID, FK, Unique)
*   `bonus_balance` (Int) - Reset bulanan.
*   `topup_balance` (Int) - Non-expiry.
*   `updated_at` (DateTime)

### **Table: `subscription_tiers`**
*   `id` (UUID, PK)
*   `code` (String, Unique) - e.g. FREE, PRO, ENTERPRISE.
*   `display_name` (String)
*   `monthly_bonus_credits` (Int)
*   `metadata` (JSONB, Nullable) - limits, feature flags.
*   `created_at` (DateTime)

`users.subscription_tier_id` → `subscription_tiers.id`.

### **Table: `projects`**
*   `id` (UUID, PK)
*   `user_id` (UUID, FK)
*   `title` (String)
*   `source_url` (String)
*   `yt_metadata` (JSONB) - Title, Channel, Duration, Heatmap data.
*   `status` (Enum: INGESTING, ANALYZING, COMPLETED, FAILED)
*   `visibility` (Enum: PRIVATE, SHARED)
*   `created_at` (DateTime)

### **Table: `project_videos` (sumber YouTube per proyek)**
Menyimpan metadata sumber video YouTube untuk satu `project`. UI memutar via YouTube player; tidak ada penyimpanan file video di server.

*   `id` (UUID, PK)
*   `project_id` (UUID, FK → `projects.id`, on delete cascade)
*   `video_id` (String) - YouTube video ID (bukan URL penuh).
*   `video_title` (String)
*   `video_description` (Text, Nullable)
*   `video_tags` (JSONB atau Text[]) - normalisasi tag konsisten dengan aplikasi.
*   `video_thumbnail_id` (String, Nullable) - referensi thumbnail (mis. kunci YouTube / CDN).
*   `video_thumbnail_url` (String, Nullable)
*   `short_count` (Int, Default 0) - jumlah short yang ter-generate untuk pasangan project+video; boleh dijaga aplikasi atau dihitung dari `generated_shorts`.
*   `created_at` (DateTime)
*   `updated_at` (DateTime)

**Unique**: `(project_id, video_id)`.

### **Table: `clips`**
*   `id` (UUID, PK)
*   `project_id` (UUID, FK)
*   `start_time` (Float)
*   `end_time` (Float)
*   `transcript_chunk` (Text)
*   `embedding_vector` (Vector, Dim: 384/1536) - Untuk semantic search.
*   `viral_score` (Int)
*   `status` (Enum: LOCKED, UNLOCKED, RENDERING, COMPLETED)
*   `output_url` (String, Nullable) - Path ke CDN/S3.

### **Table: `generated_shorts` (hasil short per segmen waktu)**
Satu baris = satu short (nomor `short_number` dari `short_count` total pada video sumber).

*   `id` (UUID, PK)
*   `project_video_id` (UUID, FK → `project_videos.id`, on delete cascade)
*   `clip_id` (UUID, FK → `clips.id`, Nullable) - jika short berasal dari segmen analisis tertentu.
*   `short_number` (Int) - indeks 1..N dalam konteks video sumber.
*   `short_count` (Int) - snapshot N pada saat generate (untuk tampilan "n of N"); boleh disinkronkan dari `project_videos.short_count`.
*   `generated_on` (DateTime)
*   `generated_by` (UUID, FK → `users.id`)
*   `last_updated` (DateTime)
*   `start_time` (Float) - detik pada timeline sumber.
*   `end_time` (Float)
*   `short_thumbnail_id` (String, Nullable)
*   `short_thumbnail_url` (String, Nullable)
*   `short_video_id` (String, Nullable) - YouTube video ID untuk short yang sudah dipublikasikan/diunggah (jika ada); preview di UI tetap bisa memakai player dengan window waktu pada sumber.
*   `short_title` (String)
*   `short_description` (Text, Nullable)
*   `short_tags` (JSONB atau Text[])
*   `subtitle` (Text, Nullable) - teks atau referensi ke asset subtitle ringkas.
*   `subtitle_language` (String, Nullable) - kode BCP 47.
*   `subtitle_style` (String atau JSONB, Nullable) - preset gaya burn-in / tampilan.
*   `subtitle_source` (Enum: AUTO, MANUAL, IMPORTED, Nullable)
*   `export_status` (Enum: NONE, PENDING, PROCESSING, COMPLETED, FAILED)
*   `export_on` (DateTime, Nullable) - waktu ekspor file berhasil (ffmpeg); file hasil tidak disimpan permanen kecuali kebijakan produk menambahkan penyimpanan objek terpisah.

**Unique**: `(project_video_id, short_number)` jika `short_number` stabil per video.

### **Table: `transactions`**
*   `id` (UUID, PK)
*   `user_id` (UUID, FK)
*   `amount` (Int)
*   `type` (Enum: BURN_INGEST, BURN_ANALYSIS, BURN_EXPORT, TOPUP_PURCHASE, REFERRAL_BONUS)
*   `wallet_source` (Enum: BONUS, TOPUP)
*   `metadata` (JSONB) - e.g. { "project_id": "..." }
*   `created_at` (DateTime)

## 3. High-Performance Indexing Strategy
*   **Composite Index**: `(user_id, status)` pada tabel `projects` untuk loading dashboard cepat.
*   **Lookup proyek-video**: Unique `(project_id, video_id)` pada `project_videos`; index `(project_id)` untuk daftar sumber per proyek.
*   **Shorts**: `(project_video_id, short_number)` unique; index `(project_video_id, export_status)` untuk antrian ekspor.
*   **Vector Index (HNSW)**: Pada `embedding_vector` untuk pencarian kemiripan klip lintas proyek.
*   **TTL Index**: Pada data cache transkrip mentah yang berusia >30 hari.

## 3.1 Playback, penyimpanan, dan ekspor

### Pratinjau (sebelum ekstrak)
*   **Perasaan “real-time”**: Pratinjau tidak memerlukan file short yang sudah di-render. UI memanggil **YouTube IFrame / Player API** dengan `video_id` dari `project_videos` dan memosisikan pemutaran pada **`start_time`–`end_time`** (detik, presisi float) yang tersimpan di `generated_shorts`.
*   **Alur**: Saat short dibuat atau dibuka, klien menginisialisasi player → seek/cue ke `start_time` → pembatasan akhir segmen (stop/loop di `end_time` sesuai implementasi). Streaming tetap dari infrastruktur YouTube; tidak ada unduhan penuh ke server aplikasi pada tahap ini.
*   **Data yang dipakai**: Cukup metadata di basis data (`video_id`, `start_time`, `end_time`, teks/thumbnail untuk UI). `short_video_id` opsional untuk kasus short sudah menjadi video terpisah di YouTube; pratinjau default tetap dari sumber panjang + window waktu.

### Ekstrak (ffmpeg, setelah user memutuskan)
*   **Tanpa file video persisten di DB**: Skema menyimpan titik data dan URL/thumbnail; blob video tidak disimpan di basis data.
*   **Ekspor**: Hanya setelah user meminta file unduhan, worker menjalankan unduhan sementara + potong segmen `start_time`–`end_time` dengan ffmpeg, lalu mengisi `export_status` / `export_on`. Biaya operasi dapat dicatat lewat `transactions` dengan `metadata` berisi `generated_short_id` / `project_id`.

## 4. Admin Management (Internal)
Tidak menggunakan tabel kredensial admin terpisah. Semua akses admin menggunakan Clerk (JWT) dan RBAC pada `users.role`.

---
*Clipper Gen - Engineered for High-Volume Data Reliability*
