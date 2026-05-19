# Rencana Anggaran Biaya (RAB): Clipper Gen
**Projected Budget for MVP & Beta Launch (6-Month Period)**

Dokumen ini merinci estimasi kebutuhan dana untuk pengembangan dan operasional awal platform Clipper Gen. Angka di bawah adalah estimasi dalam Rupiah (IDR).

## 1. Sumber Daya Manusia (Human Resources)
Fokus pada tim inti yang ramping (lean team) untuk mencapai MVP dalam 3-4 bulan.

| Peran | Estimasi Gaji/Bln | Durasi | Total (6 Bln) |
| :--- | :--- | :--- | :--- |
| **1 Lead AI/Backend Engineer** | Rp 20.000.000 | 6 Bln | Rp 120.000.000 |
| **1 Fullstack Developer (Next.js)** | Rp 12.000.000 | 6 Bln | Rp 72.000.000 |
| **1 UI/UX Designer (Part-time/Contract)** | Rp 7.000.000 | 3 Bln | Rp 21.000.000 |
| **Subtotal SDM** | | | **Rp 213.000.000** |

## 2. Infrastruktur & AI Computing (Cloud & API)
Estimasi biaya untuk server GPU dan pemrosesan data.

| Komponen | Spesifikasi | Estimasi/Bln | Total (6 Bln) |
| :--- | :--- | :--- | :--- |
| **GPU Instance (AWS G4dn)** | 2x NVIDIA T4 (On-demand/Spot) | Rp 10.000.000 | Rp 60.000.000 |
| **Cloud Storage (S3)** | 1 TB High Frequency Access | Rp 500.000 | Rp 3.000.000 |
| **Database & Cache** | RDS PostgreSQL & Redis | Rp 1.500.000 | Rp 9.000.000 |
| **API LLM (GPT-4o)** | ~15M Tokens Usage | Rp 2.000.000 | Rp 12.000.000 |
| **Bandwidth (Data Transfer)** | 2 TB Outbound Traffic | Rp 2.500.000 | Rp 15.000.000 |
| **Subtotal Infra** | | | **Rp 99.000.000** |

## 3. Operasional, Legal & Marketing
Biaya pendukung untuk validasi pasar dan legalitas.

| Komponen | Keterangan | Estimasi |
| :--- | :--- | :--- |
| **Legalitas & Pendirian** | Akta Perusahaan (PT/CV) & Domisili | Rp 15.000.000 |
| **Marketing & User Acquisition** | Ads (Meta/Google) & Referral Rewards | Rp 25.000.000 |
| **Domain & SaaS Tools** | Domain, Email, Workspace, Monitoring | Rp 5.000.000 |
| **Biaya Tak Terduga (Contingency)** | 15% dari Total Project Cost | Rp 53.550.000 |
| **Subtotal Ops** | | **Rp 98.550.000** |

## 4. Ringkasan Total Anggaran (Summary)

| Kategori | Alokasi Dana | Persentase |
| :--- | :--- | :--- |
| **Sumber Daya Manusia** | Rp 213.000.000 | 52% |
| **Infrastruktur & API** | Rp 99.000.000 | 24% |
| **Operasional & Legal** | Rp 98.550.000 | 24% |
| **TOTAL ESTIMASI RAB** | **Rp 410.550.000** | **100%** |

## 5. Strategi Penghematan Biaya (Cost-Optimization)
*   **Local Embedding**: Penggunaan model embedding lokal memangkas biaya API LLM hingga 80%.
*   **Spot Instances**: Menggunakan AWS Spot Instances untuk worker pemrosesan video non-urgent guna menghemat biaya server hingga 60%.
*   **FFmpeg Efficiency**: Optimasi script FFmpeg untuk mengurangi waktu rendering per video, yang secara langsung mengurangi durasi pemakaian GPU.

