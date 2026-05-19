# Tokenomics Strategy: The Utility Economy
**Sustainable Computing Resource Management**

Clipper Gen menggunakan unit **Tokens** untuk mengabstraksi biaya komputasi yang bervariasi (GPU, CPU, API). Sistem ini menjamin margin keuntungan yang sehat sekaligus memberikan fleksibilitas bagi pengguna.

## 1. Token Acquisition Channels

### **A. System-Led Bonuses**
1.  **Welcome Bonus**: 50 Tokens diberikan otomatis saat akun baru memverifikasi email (One-time).
2.  **Subscription Bonus**: Jatah token bulanan yang disertakan dalam paket Basic (650) dan Premium (2,500). Bonus ini hangus setiap akhir siklus billing.

### **B. Growth-Led Rewards (The Viral Loop)**
1.  **Referrer Reward**: Mendapatkan 25 Tokens setiap kali orang yang diajak (referee) melakukan ekspor video pertama mereka.
2.  **Referee Reward**: Mendapatkan tambahan 15 Tokens saat mendaftar (Total 65 Tokens awal).

### **C. User-Led Purchases (Top-up)**
*   Pembelian paket token mandiri (Starter, Pro, Agency).
*   Token Top-up disimpan dalam "Permanent Wallet" yang **tidak memiliki masa kedaluwarsa**.

## 2. Service Burn Rate (Usage Costs)

Sistem akan memotong saldo token (prioritas: Bonus Wallet -> Top-up Wallet) berdasarkan aktivitas:

| Aktivitas Layanan | Biaya (Tokens) | Catatan Khusus |
| :--- | :--- | :--- |
| **Ingestion & Transcription** | 1 / Menit | Berdasarkan durasi video sumber. |
| **YouTube Heatmap Analysis** | 10 / Video | Pengambilan data retensi riil. |
| **Semantic Analysis (AI)** | 2 / Menit | Filter cerdas via Embedding & LLM. |
| **Recommendation Unlocking** | 10 / Batch | Membuka +10 kandidat klip tambahan. |
| **Export Standard (1080p)** | 5 / Klip | Termasuk auto-captioning dasar. |
| **Export Premium (4K)** | 20 / Klip | Beban rendering VRAM ekstrim. |
| **AI Translation & Dubbing** | 10 / Klip | Integrasi model multibahasa. |
| **Shorts Monitoring** | 2 / Hari | Pelacakan statistik per Shorts. |

## 3. Burn Priority Logic (ABAC Integration)
Untuk mengoptimalkan pengeluaran pengguna, sistem menerapkan logika berikut:
1.  **Bonus Wallet First**: Saldo yang akan hangus digunakan terlebih dahulu.
2.  **Top-up Wallet Second**: Saldo permanen digunakan jika Bonus Wallet kosong.
3.  **Tier-Based Cost**: Pengguna Premium mendapatkan diskon burn rate sebesar 10% untuk pemrosesan volume besar (Enterprise Benefit).

---
*Clipper Gen - Transparent Computing Economy*
