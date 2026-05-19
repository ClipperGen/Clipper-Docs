# Risk Assessment & Mitigation Strategy
**Ensuring Operational Stability and Business Continuity**

Clipper Gen mengidentifikasi risiko teknis, operasional, dan strategis guna meminimalisir kegagalan sistem serta kerugian finansial. Strategi ini didukung oleh cadangan **15% Contingency Fund** dalam anggaran biaya.

## 1. Technical & Infrastructure Risks

| Kategori Risiko | Dampak | Strategi Mitigasi |
| :--- | :--- | :--- |
| **FFmpeg Library Failure** | Gagal rendering subtitle (No libass). | Penggunaan Docker container dengan build FFmpeg statis yang mendukung `libass` & `fontconfig`. |
| **Path & Character Errors** | Kegagalan FFmpeg pada nama file spasi. | Implementasi **Temporary Working Directory** dengan sistem sanitasi nama file otomatis. |
| **GPU Instance Scarcity** | Antrean rendering memanjang. | Multi-cloud strategy (AWS/GCP) dan penggunaan spot instances dengan fallback otomatis ke CPU. |
| **YouTube API Changes** | Data heatmap tidak dapat diakses. | Implementasi scraper cadangan dan fallback ke **Embedding-based Semantic Search** sebagai sinyal viral utama. |

## 2. Business & Economic Risks

| Kategori Risiko | Dampak | Strategi Mitigasi |
| :--- | :--- | :--- |
| **Compute Cost Spike** | Margin keuntungan menurun. | Pembatasan rekomendasi per tier dan strategi **Embedding-First** untuk menekan pemakaian LLM hingga 80%. |
| **Token Inflation** | Nilai token tidak sebanding infrastruktur. | Peninjauan berkala pada *Service Burn Rate* dan pemanfaatan paket pembelian token permanen (Non-expiry). |
| **Competitor Price War** | Kehilangan pangsa pasar. | Fokus pada **Empirical Data (Heatmap)** yang tidak dimiliki kompetitor entry-level. |

## 3. Regulatory & Legal Risks

| Kategori Risiko | Dampak | Strategi Mitigasi |
| :--- | :--- | :--- |
| **Copyright Infringement** | Tuntutan hukum dari pemilik konten. | Syarat Layanan (TOS) yang menegaskan tanggung jawab hak cipta ada di sisi pengguna; sistem filter konten terlarang. |
| **AI Data Privacy** | Kebocoran data suara/video user. | Kebijakan **Zero-Retention** untuk raw data setelah proses rendering selesai (kecuali diminta user). |

## 4. Disaster Recovery & Contingency
*   **Contingency Fund**: Alokasi 15% dari total RAB untuk menangani lonjakan biaya mendadak atau kegagalan perangkat lunak.
*   **Redundancy**: Replikasi database lintas region setiap 24 jam untuk menjamin ketersediaan data pengguna.

---
*Clipper Gen - Resilient Architecture for Global Content*
