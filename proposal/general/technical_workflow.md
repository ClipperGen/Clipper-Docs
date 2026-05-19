# Technical Workflow & Pipeline
**The Engineering Behind Clipper Gen**

Clipper Gen beroperasi di atas infrastruktur pemrosesan video yang dioptimalkan dengan model AI terkini. Berikut adalah rincian teknis di balik setiap tahap produksinya.

## 1. Ingestion & Pre-processing (YouTube-First)
*   **Primary Source**: URL link dari YouTube. Sistem menggunakan **yt-dlp** untuk mengambil konten video dan metadata secara otomatis.
*   **Premium Feature**: Direct Upload (MP4/MOV) hanya tersedia untuk pengguna paket Premium guna menjaga beban bandwidth dan storage server.
*   **Audio Striping**: Ekstraksi audio track menggunakan `ffmpeg` (`-vn -acodec pcm_s16le`) sebagai langkah awal analisis.

## 2. Intelligence Layer (Embedding & Semantic Search)
Untuk efisiensi biaya (cost-efficiency), kita menggunakan strategi dua lapis:
*   **Tier 1: YouTube Heatmap Analysis (Data-Driven)**: 
    *   Sistem mengakses data **"Most Replayed"** via YouTube API/Scraper untuk menemukan segmen dengan retensi penonton tertinggi.
    *   Memberikan bobot viralitas tinggi pada bagian yang paling sering ditonton ulang oleh audiens asli.
*   **Tier 2: Embedding Analysis (Semantic Search)**: 
    *   Mencocokkan transkrip dengan pola viral jika data heatmap tidak tersedia.
*   **Tier 3: Ranked Refinement & Limitation**: 
    *   LLM hanya melakukan finalisasi narasi untuk sejumlah X kandidat teratas sesuai tier pengguna (misal: 10 untuk Basic).
    *   Sistem menyimpan kandidat potensial lainnya (hasil embedding/heatmap) dalam status "Locked Candidate."
*   **Tier 4: On-Demand Unlocking**:
    *   Jika pengguna menginginkan lebih banyak klip, mereka dapat melakukan "Unlock."
    *   Sistem kemudian mengirimkan batch "Locked Candidates" berikutnya ke LLM untuk diproses menjadi rekomendasi matang. Ini mencegah pemborosan token LLM untuk klip yang tidak diinginkan user.

## 5. Performance Monitoring (Closed-Loop System)
*   **Shorts Tracking**: Pengguna dapat memasukkan link Shorts yang telah diunggah.
*   **Analytics Ingestion**: Sistem memantau perkembangan View, Likes, dan Shares secara berkala untuk memberikan laporan efektivitas "Viral Score" yang diberikan AI sebelumnya.

## 3. Computer Vision Layer (Scene Refinement)
*   **PySceneDetect Analysis**: Melakukan pemindaian scene di sekitar koordinat waktu yang disarankan oleh sistem embedding/LLM.
*   **Boundary Snapping**: Memastikan transisi klip terjadi pada "hard cuts" yang dideteksi secara visual.

## 4. Video Engine (Pure FFmpeg Processing)
Clipper Gen meminimalkan penggunaan library berat dan beralih ke **FFmpeg-native processing** dengan spesifikasi:
*   **libass Support**: Server diwajibkan menggunakan build FFmpeg yang mendukung `libass` untuk rendering subtitle dinamis yang stabil.
*   **Robust Path Handling**: Untuk mencegah error pada path yang mengandung spasi, sistem menggunakan **Temporary Working Directory** dengan penamaan yang bersih (sanitized) selama proses rendering.
*   **Format Compatibility**: Modul internal untuk mengonversi **VTT (YouTube standard)** ke **SRT** secara otomatis, termasuk penyesuaian timestamp pasca-clipping.
*   **Lossless Slicing**: Menggunakan `-ss` dan `-to` dengan `-c copy` untuk pemotongan instan tanpa degradasi kualitas.
*   **Native Filters**: Burn-in subtitles menggunakan filter `drawtext` atau `subtitles` bawaan FFmpeg.
*   **Resolution Scaling**: Optimasi resolusi ke format portrait (9:16) dilakukan langsung via filter `-vf "crop=ih*9/16:ih,scale=1080:1920"`.

## 5. Cost-Optimization & Translation
*   **Batch Subtitle Translation**: Mengurangi biaya API Translation hingga **95%** dengan mengirimkan subtitle dalam batch (20-25 baris per request) alih-alih per baris.
*   **1080p Standard**: Membatasi pemrosesan pada 1080p secara default untuk mengoptimalkan durasi pemakaian GPU dan efisiensi storage.

## 6. Output Management
*   **Metadata Generation**: LLM menghasilkan Judul Klip, Deskripsi YouTube/Instagram, dan Hashtag relevan yang diekspor sebagai file JSON/CSV.
*   **Project Packaging**: Semua aset (video, thumbnail, metadata) dibundel dalam satu objek "Project" di database.

