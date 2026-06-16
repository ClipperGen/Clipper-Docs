# [Nama Proyek] Modernization/Fix Report: [Nama Fase atau Judul Utama]

## Executive Summary / Summary of Work
Ringkasan eksekutif 2-3 kalimat yang menjelaskan *state* proyek saat ini, perubahan masif yang terjadi, dan dampak utamanya bagi infrastruktur sistem.

---

## Current Project Status / Metadata
Gunakan daftar ringkas (bullet points) untuk menyajikan konteks teknis atau metadata laporan saat ini agar scannable:
* **Progress**: (Contoh: Phase 1-4.1 Complete / Staging Selesai)
* **Architecture**: (Contoh: Headless Hybrid / Monolithic Python Backend)
* **Primary Database**: (Contoh: PostgreSQL 15 / SQLite)
* **Deployment Instance**: (Contoh: Container `lilyopencms-staging2` on Port 6002)

---

## Bug Details & Resolusi (Untuk Laporan Perbaikan/Stabilitas)
Gunakan format terstruktur **Masalah -> Penyebab -> Solusi** untuk setiap temuan agar mudah dilacak oleh tim pengembang:

### 1. [Nama Bug / Judul Komponen]
* **The Problem (Masalah)**: Detail error (misal: `404 Not Found`, `SQL Integrity Error`, `Cascading 500 Errors`).
* **The Cause (Penyebab)**: Akar permasalahan secara teknis (misal: foreign key constraint, missing rollback block, salah tipe data).
* **The Solving (Solusi)**: Tindakan refactoring atau perbaikan yang diambil (sertakan cuplikan kode minimal/produksi jika diperlukan).

---

## Completed Phases / Fitur Terimplementasi (Untuk Laporan Fitur/Modernisasi)
Gunakan pendekatan per komponen atau per fase untuk memecah perubahan arsitektur yang masif:

### Phase/Komponen 1: [Nama Komponen]
* **Detail Implementasi**: Apa saja yang diubah atau dimigrasikan.
* **Data Integrity**: Validasi jumlah data atau record yang terdampak.
* **Optimasasi**: Peningkatan performa yang diterapkan (caching, connection pooling, dll).

---

## Technical Metrics & Results
Sajikan data kuantitatif, hasil pengujian, atau status komponen dalam bentuk tabel standar untuk mempermudah pembacaan cepat:

| Komponen / Metrik | Status / Spesifikasi | Hasil / Catatan |
| :--- | :--- | :--- |
| [Contoh: Image Records] | [Standardized] | [570 WebP images generated] |
| [Contoh: API Endpoints] | [Stabil] | [Mendukung PUT/PATCH & Format JSON baru] |

---

## Known Issues & Mitigasi
Catat masalah yang belum terselesaikan selama proses migrasi/perbaikan beserta rencana investigasinya agar transparansi sistem tetap terjaga:
* **[Nama Issue]**: (Contoh: Elasticsearch mapping error untuk indeks tertentu - Investigation in progress).
* **[Nama Issue]**: (Contoh: Depreciation warnings pada library tertentu - Tracked for future refactor).

---

## Future Roadmap: Phase X & Y
Daftar rencana kerja terstruktur menggunakan checklist (`- [ ]`) untuk langkah atau fase pengembangan selanjutnya:
* [ ] **Tugas 1**: Deskripsi singkat tindakan teknis yang akan diambil.
* [ ] **Tugas 2**: Integrasi atau pengujian yang dijadwalkan berikutnya.

---

**Report generated for Pull Request / Branch**: `[nama-branch]` -> `[target-branch]`
**Commit Hash**: `[hash-commit]`
**Date**: [Tanggal Bulan Tahun]
**Prepared by**: Mema
