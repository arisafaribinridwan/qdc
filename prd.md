 # PRD — QRCC FQMS/F-COST Monthly Workflow

> **Versi**: 1.0 · **Terakhir diperbarui**: 2026-05-16
>
> Dokumen ini adalah **sumber kebenaran utama (single source of truth)** untuk seluruh pengembangan aplikasi QRCC Data Center modul FQMS/F-COST. Jika ada konflik antara dokumen lain dengan file ini, **ikuti file ini**. Dokumen lain di folder `.doc/` adalah arsip, discovery notes, atau supporting plan yang harus tunduk pada keputusan di file ini.

---

## 1 · Ringkasan Produk

Aplikasi web internal QRCC untuk menggantikan workflow bulanan FQMS/F-COST yang saat ini bergantung pada Excel workbook, copy-paste, dan pengecekan manual. Bukan platform enterprise — ini **alat kerja** yang mempercepat proses bulanan dan menghasilkan report PDF/Excel.

**Identitas aplikasi:**

| Field | Nilai |
|---|---|
| Nama Aplikasi | QRCC Data Center |
| Deskripsi | Aplikasi dashboard internal QRCC untuk mengelola workflow FQMS, F-COST, PPM, Sales, Quality Issue, validasi data bulanan, dan report generation. |
| Target User | Operator tunggal workflow bulanan FQMS/F-COST di QRCC LCD Local |

---

## 2 · Tech Stack

| Layer | Teknologi | Catatan |
|---|---|---|
| Framework | **Nuxt 4** (full-stack monolith) | Satu repo: UI + API + DB |
| Bahasa | **TypeScript strict** | Seluruh codebase |
| UI | **Vue 3 + Nuxt UI 4** | Komponen default untuk semua form/table |
| Database | **SQLite** (local-first) | File tunggal, mudah backup |
| ORM | **Drizzle ORM** | Schema-as-code, migration |
| Validasi Input | **Zod** | Server-side route validation |
| Export Excel | **ExcelJS** | Template `.xlsx` approach |
| Export PDF | **Playwright** | Render HTML/CSS → PDF |
| Auth | **Better Auth** | Opsional untuk MVP lokal, wajib jika deploy ke jaringan |

---

## 3 · Prinsip Kunci

1. **Single source of truth** — Data utama di SQLite, bukan Excel. Report adalah output, bukan input.
2. **Long-format storage** — Data disimpan sebagai baris (`month`, `model`, `category`, `qty`). Wide-format hanya untuk tampilan report.
3. **Flow pendek** — Idealnya: pilih bulan → input data → validasi → preview → export → selesai.
4. **Grouping eksplisit** — Aturan penggabungan model (misal HD1400I → HD1500I) disimpan di tabel `model_group_rules`, bukan hardcode.

---

## 4 · Struktur Folder

```text
qdc/
  app/
    components/        # Komponen Vue reusable
    composables/       # State & logic composable
    layouts/           # Layout halaman (sidebar, dll)
    pages/             # File-based routing
    types/             # TypeScript types khusus frontend
    utils/             # Helper/utility frontend
  server/
    api/               # HTTP endpoint (controller layer)
    db/
      schema.ts        # Drizzle table definitions
      client.ts        # Koneksi SQLite
      migrations/      # SQL migration files
    repositories/      # Database access (query layer)
    services/          # Business logic layer
    reports/
      pdf/             # HTML/CSS layout untuk PDF
      excel/           # Logic pengisian template Excel
      view-models/     # Transform data → report structure
  shared/
    types/             # Types yang dipakai frontend & backend
    constants/         # Konstanta global
    validators/        # Zod schema bersama
  templates/
    excel/             # File template .xlsx
    pdf/               # Template HTML/CSS report
  storage/
    exports/           # Output PDF/Excel
    backups/           # Backup database
    imports/           # File import sementara
```

---

## 5 · Backend Architecture Pattern

Gunakan pola **API → Service → Repository** (layered architecture).

```
HTTP Request → API (Controller) → Service → Repository → Database
```

| Layer | Tanggung Jawab | Dilarang | Nama File |
|---|---|---|---|
| **API / Controller** | Terima request, validasi input (Zod), format response JSON | Business logic, akses DB langsung | `*.controller.ts` atau `server/api/**/*.ts` |
| **Service** | Business logic, rules, kalkulasi, orchestration | Tahu soal HTTP (`req`/`res`), akses DB langsung | `*.service.ts` |
| **Repository** | CRUD ke database, query Drizzle/SQL | Business logic, format HTTP response | `*.repository.ts` |

**Aturan penting:**
- Service **tidak boleh** import dari `h3` atau menerima parameter `event`.
- Repository **tidak boleh** menghitung PPM atau menjalankan validation logic.
- API handler hanya memanggil service, lalu return response.

---

## 6 · Data Model

### 6.1 report_months

| Field | Type | Keterangan |
|---|---|---|
| id | integer PK | Auto-increment |
| month_key | text UNIQUE | Format `YYYYMM`, misal `202604` |
| month_label | text | Label tampilan, misal `Apr-26` |
| period_start | text | ISO date awal periode |
| period_end | text | ISO date akhir periode |
| status | text | `draft` / `validated` / `exported` / `archived` |
| created_at | text | ISO timestamp |
| updated_at | text | ISO timestamp |

### 6.2 models

| Field | Type | Keterangan |
|---|---|---|
| id | integer PK | Auto-increment |
| model_name | text | Nama model |
| model_series | text | Seri (nullable) |
| product | text | Nama produk |
| active_status | text | `Active` / `Inactive` |
| report_include | integer | `1` = masuk report, `0` = tidak |
| effective_from | text | ISO date nullable |
| effective_to | text | ISO date nullable |
| remark | text | Catatan nullable |
| created_at | text | ISO timestamp |
| updated_at | text | ISO timestamp |

### 6.3 model_group_rules

| Field | Type | Keterangan |
|---|---|---|
| id | integer PK | Auto-increment |
| source_model | text | Model asli (misal `HD1400I`) |
| report_model | text | Model di report (misal `HD1500I`) |
| rule_type | text | Tipe aturan |
| active | integer | `1` / `0` |
| remark | text | Catatan nullable |

### 6.4 defect_categories

| Field | Type | Keterangan |
|---|---|---|
| id | integer PK | Auto-increment |
| code | text UNIQUE | Kode kategori |
| name | text | Nama: Power Unit, Main Unit, Panel, Software, Other |
| sort_order | integer | Urutan tampilan |
| active | integer | `1` / `0` |

### 6.5 nondefect_categories

Struktur sama dengan `defect_categories`. Contoh nama: User Usage, Setting, Explanation, Signal.

### 6.6 monthly_model_summaries

| Field | Type | Keterangan |
|---|---|---|
| id | integer PK | Auto-increment |
| report_month_id | integer FK | → report_months.id |
| model_id | integer FK | → models.id |
| sales_period | text | Periode penjualan |
| sales_qty | integer | Jumlah sales |
| total_defect_qty | integer | Total defect |
| total_nondefect_qty | integer | Total non-defect |
| defect_ppm | real | PPM defect (calculated) |
| nondefect_ppm | real | PPM non-defect (calculated) |
| remark | text | Catatan nullable |
| created_at | text | ISO timestamp |
| updated_at | text | ISO timestamp |

**Constraint:** `UNIQUE(report_month_id, model_id)`

### 6.7 defect_category_entries

| Field | Type | Keterangan |
|---|---|---|
| id | integer PK | Auto-increment |
| report_month_id | integer FK | → report_months.id |
| model_id | integer FK | → models.id |
| category_id | integer FK | → defect_categories.id |
| qty | integer | Jumlah |
| sales_qty_snapshot | integer | Snapshot sales qty saat input (audit) |
| ppm | real | Calculated |
| remark | text | Catatan nullable |

**Constraint:** `UNIQUE(report_month_id, model_id, category_id)`

### 6.8 nondefect_category_entries

Struktur sama dengan `defect_category_entries`, FK `category_id` → `nondefect_categories.id`.

### 6.9 repair_action_entries

| Field | Type | Keterangan |
|---|---|---|
| id | integer PK | Auto-increment |
| report_month_id | integer FK | → report_months.id |
| model_id | integer FK | → models.id |
| repair_action | text | Nama action |
| qty | integer | Jumlah |
| total | integer | Total |
| defect_occup | real | Defect occupation |
| defect_ppm | real | PPM |
| remark | text | Catatan nullable |

### 6.10 fiscal_quality_targets

| Field | Type | Keterangan |
|---|---|---|
| id | integer PK | Auto-increment |
| fiscal_half | text | Label fiscal half, misal `2025FH` / `2025LH` |
| product | text | Product report, misal `LCD` |
| manufacturer | text | Vendor/manufacturer report, misal `LOCAL` |
| target_monthly_ppm | real | Target monthly PPM global FQMS untuk fiscal half |
| effective_from | text | ISO date awal fiscal half |
| effective_to | text | ISO date akhir fiscal half |
| remark | text | Catatan nullable |

### 6.11 fiscal_fcost_targets

| Field | Type | Keterangan |
|---|---|---|
| id | integer PK | Auto-increment |
| fiscal_half | text | Label fiscal half, misal `2025FH` / `2025LH` |
| product | text | Product report, misal `LCD` |
| manufacturer | text | Vendor/manufacturer report, misal `LOCAL` |
| target_fcost | real | Target F-COST bulanan yang berlaku sama untuk semua bulan dalam fiscal half |
| effective_from | text | ISO date awal fiscal half |
| effective_to | text | ISO date akhir fiscal half |
| remark | text | Catatan nullable |

### 6.12 monthly_fcost_summaries

| Field | Type | Keterangan |
|---|---|---|
| id | integer PK | Auto-increment |
| report_month_id | integer FK | → report_months.id |
| product | text | Product report, misal `LCD` |
| manufacturer | text | Vendor/manufacturer report, misal `LOCAL` |
| sales_amount | real | Nilai penjualan, satuan mengikuti template `Amount < Rp. K>` |
| sales_qty | integer | Jumlah unit sales |
| fcost_amount | real | Nilai F-COST, satuan mengikuti template `Amount < Rp. K>` |
| fcost_qty | integer | Jumlah unit F-COST |
| remark | text | Catatan nullable |

**Constraint:** `UNIQUE(report_month_id, product, manufacturer)`

### 6.13 monthly_fcost_item_breakdowns

| Field | Type | Keterangan |
|---|---|---|
| id | integer PK | Auto-increment |
| report_month_id | integer FK | → report_months.id |
| product | text | Product report |
| manufacturer | text | Vendor/manufacturer report |
| item_type | text | `Part` / `Labor` / `Trip` |
| amount | real | Amount bulanan item F-COST |
| remark | text | Catatan nullable |

**Constraint:** `UNIQUE(report_month_id, product, manufacturer, item_type)`

### 6.14 monthly_fcost_part_category_breakdowns

| Field | Type | Keterangan |
|---|---|---|
| id | integer PK | Auto-increment |
| report_month_id | integer FK | → report_months.id |
| product | text | Product report |
| manufacturer | text | Vendor/manufacturer report |
| part_category | text | `Panel` / `Main Unit` / `Power Unit` / `Other` |
| amount | real | Amount bulanan kategori part |
| qty | integer | Qty bulanan kategori part |
| remark | text | Catatan nullable |

**Constraint:** `UNIQUE(report_month_id, product, manufacturer, part_category)`

### 6.15 validation_runs

| Field | Type | Keterangan |
|---|---|---|
| id | integer PK | Auto-increment |
| report_month_id | integer FK | → report_months.id |
| status | text | `OK` / `CHECK` |
| checked_at | text | ISO timestamp |
| summary_json | text | JSON snapshot hasil validasi |

### 6.16 export_jobs

| Field | Type | Keterangan |
|---|---|---|
| id | integer PK | Auto-increment |
| report_month_id | integer FK | → report_months.id |
| type | text | `pdf` / `excel` |
| file_name | text | Nama file output |
| file_path | text | Path file |
| status | text | `pending` / `done` / `error` |
| created_at | text | ISO timestamp |
| message | text | Pesan error nullable |

---

## 7 · Aturan Perhitungan

| Aturan | Formula / Ketentuan |
|---|---|
| **Monthly PPM dasar** | `qty / sales_qty × 1.000.000`; dipakai untuk validasi/entry bulanan sederhana jika diperlukan |
| **PPM saat sales_qty = 0** | Tampilkan `CHECK`, jangan tampilkan angka PPM |
| **FQMS accumulated sales** | Akumulasi sales bulanan dari launching month model sampai report month |
| **FQMS launching period** | Selisih bulan antara launching month dan report month |
| **FQMS AVG defect PPM** | `accumulated_defect_qty / (accumulated_sales × launching_period) × 1.000.000` |
| **FQMS AVG non-defect PPM** | `accumulated_nondefect_qty / (accumulated_sales × launching_period) × 1.000.000` |
| **FQMS total AVG defect PPM** | `total_defect / Σ(accumulated_sales_model × launching_period_model) × 1.000.000`; bukan average sederhana antar model |
| **FQMS quality level** | `NG` jika AVG defect PPM >= target monthly PPM fiscal half, selain itu `OK` |
| **Total defect category** | Σ `defect_category_entries.qty` per model per bulan **harus =** `monthly_model_summaries.total_defect_qty` |
| **Total nondefect category** | Σ `nondefect_category_entries.qty` per model per bulan **harus =** `monthly_model_summaries.total_nondefect_qty` |
| **Sales qty sumber utama** | `monthly_model_summaries.sales_qty`. Field `sales_qty_snapshot` di entries hanya untuk audit. |
| **Grouping model** | Saat render report, gabungkan data berdasarkan `model_group_rules`. Data original tetap terpisah di DB. |
| **Target monthly PPM FQMS** | Satu nilai global per product + manufacturer + fiscal half, di-update pada bulan pertama fiscal half |
| **Target F-COST** | Satu nilai global per product + manufacturer + fiscal half, di-update pada bulan pertama fiscal half |
| **F-COST Cost vs Target** | `fcost_amount / target_fcost`; menggantikan istilah template lama `Achievement` |
| **F-COST Ratio vs Sales** | `fcost_amount / sales_amount` |
| **F-COST Qty Ratio** | `fcost_qty / sales_qty` |
| **F-COST LY** | Diambil dari F-COST bulan yang sama satu tahun sebelumnya; bukan input manual |
| **F-COST Ratio vs LY** | `fcost_amount / ly_fcost` |
| **F-COST item ratio** | `item_amount / total_fcost_amount` untuk item `Part`, `Labor`, `Trip` |
| **F-COST part category ratio** | `part_category_amount / total_part_amount`; rasio tidak disimpan sebagai input manual |

---

## 8 · Validation Rules

Semua validasi menghasilkan status **OK** atau **CHECK** + alasan + link ke data.

| # | Rule | Deskripsi |
|---|---|---|
| V1 | Active model completeness | Semua model `active=1, report_include=1` harus punya summary di bulan terpilih |
| V2 | Duplicate summary | Tidak boleh >1 summary untuk kombinasi bulan + model |
| V3 | Defect total match | Σ defect entries = total_defect_qty di summary |
| V4 | Non-defect total match | Σ non-defect entries = total_nondefect_qty di summary |
| V5 | Sales qty presence | Model dengan qty > 0 harus punya sales_qty > 0 |
| V6 | Category standardization | Semua entries harus pakai category_id dari reference table |
| V7 | Report month consistency | Semua data report utama harus dari bulan yang benar; section trend dan LY boleh mengambil bulan historis sesuai rule report |
| V8 | F-COST completeness | Jika report F-COST aktif, data F-COST summary bulan terpilih harus tersedia |
| V9 | F-COST target presence | Target F-COST fiscal half berjalan harus tersedia |
| V10 | F-COST item breakdown match | Σ item breakdown `Part` + `Labor` + `Trip` untuk current half harus = total F-COST current half |
| V11 | F-COST part category match | Σ part category amount untuk current half harus = total item `Part` current half |
| V12 | F-COST LY data presence | Data F-COST bulan yang sama satu tahun sebelumnya wajib tersedia untuk Ratio vs LY; jika tidak ada tampilkan `CHECK` / kosong dengan status `LY data missing` |

---

## 9 · UI Page Map

| Route | Fungsi |
|---|---|
| `/` | Dashboard: bulan aktif, status validasi, shortcut, tombol generate |
| `/report-months` | Daftar bulan laporan + status |
| `/models` | Master model (CRUD + filter active) |
| `/references/defect-categories` | Reference defect categories |
| `/references/nondefect-categories` | Reference non-defect categories |
| `/references/grouping-rules` | Aturan grouping model |
| `/entry/summary` | Input monthly model summary |
| `/entry/defect` | Input defect category per model |
| `/entry/nondefect` | Input non-defect category per model |
| `/entry/repair-action` | Input repair action per model |
| `/entry/fcost` | Input F-COST bulanan |
| `/validation` | Status validasi OK/CHECK + link perbaikan |
| `/reports/preview` | Preview report (sumber layout PDF) |
| `/reports/export` | Export PDF / Excel + history |
| `/settings/backup` | Backup / restore database |

**Prinsip dashboard:** Harus langsung menjawab 3 hal → bulan apa yang aktif, data apa yang belum lengkap, langkah berikutnya apa.

---

## 10 · Report Specification

### PDF
- Dibuat dari **HTML/CSS print layout** via Playwright.
- Route preview (`/reports/fqms/:month/preview`) = sumber layout yang sama untuk preview UI dan PDF output.
- Section FQMS: header, Quality Trend, Acceptance Ratio, Detail Model, Worst Defect. Quality Issue and Follow Up belum masuk scope FQMS MVP awal.
- Section F-COST: header/summary, Monthly F-Cost, F-Cost Trend, Detail F-Cost & Part Contribution.
- Nama file otomatis: `QRCC FQMS - LCD SEID Apr 2026.pdf`

### Excel
- Pendekatan **template workbook** — isi cell/named range dari view model data.
- Template terpisah per jenis: `job4-presentation-template.xlsx`, `fqms-product-template.xlsx`, `fcost-template.xlsx`.
- MVP mulai dari **1 template** yang paling sering dipakai.
- Nama file otomatis: `QRCC Data Presentasi Meeting LCD SEID Apr 2026.xlsx`

---

## 11 · Roadmap Implementasi

> **Urutan: Frontend dulu → Backend menyusul.**
> Setiap phase harus selesai dan di-merge sebelum lanjut ke phase berikutnya.

### Phase 0 — Discovery & Persiapan
- [ ] Audit dashboard template yang sudah ada
- [ ] Identifikasi komponen Nuxt UI 4 yang bisa dipakai ulang
- [ ] Tentukan menu baru untuk domain FQMS/F-COST
- [ ] Kumpulkan contoh output Excel/PDF referensi

### Phase 1 — UI Shell & Identity (Frontend)
- [ ] Update app metadata: title `QRCC Data Center`, description, favicon
- [ ] Rombak sidebar/navigation sesuai page map (Section 9)
- [ ] Buat layout utama baru (sidebar + content area)
- [ ] Buat halaman dashboard kosong dengan placeholder cards
- [ ] Setup tema/warna identitas QRCC
- [ ] Hapus halaman demo lama (customers, inbox)

### Phase 2 — Halaman Master Data UI (Frontend)
- [ ] Halaman `/report-months` — tabel + form create/edit
- [ ] Halaman `/models` — tabel + search + filter active + form drawer
- [ ] Halaman `/references/defect-categories` — tabel + form
- [ ] Halaman `/references/nondefect-categories` — tabel + form
- [ ] Halaman `/references/grouping-rules` — tabel + form
- [ ] Semua halaman pakai mock data / composable state dulu

### Phase 3 — Halaman Data Entry UI (Frontend)
- [ ] Halaman `/entry/summary` — editable table per bulan
- [ ] Halaman `/entry/defect` — editable table per model × category
- [ ] Halaman `/entry/nondefect` — editable table per model × category
- [ ] Halaman `/entry/repair-action` — editable table
- [ ] Halaman `/entry/fcost` — form / editable table
- [ ] Semua support edit cepat, highlight error, dan month selector

### Phase 4 — Halaman Validation & Report UI (Frontend)
- [ ] Halaman `/validation` — daftar check dengan status OK/CHECK (mock)
- [ ] Halaman `/reports/preview` — layout report HTML/CSS
- [ ] Halaman `/reports/export` — tombol export + daftar history
- [ ] Halaman `/settings/backup` — tombol backup/restore

### Phase 5 — Database Core (Backend)
- [ ] Setup Drizzle + SQLite connection
- [ ] Definisikan semua schema table (Section 6)
- [ ] Buat migration awal
- [ ] Seed reference categories (defect, non-defect)
- [ ] Buat repository layer untuk semua entitas

### Phase 6 — API & Service Layer (Backend)
- [ ] CRUD API + Service untuk report_months
- [ ] CRUD API + Service untuk models
- [ ] CRUD API + Service untuk defect_categories & nondefect_categories
- [ ] CRUD API + Service untuk model_group_rules
- [ ] CRUD API + Service untuk monthly_model_summaries
- [ ] CRUD API + Service untuk defect/nondefect entries
- [ ] CRUD API + Service untuk repair_action_entries
- [ ] CRUD API + Service untuk monthly_fcost_summaries
- [ ] CRUD API + Service untuk fiscal_fcost_targets
- [ ] CRUD API + Service untuk monthly_fcost_item_breakdowns
- [ ] CRUD API + Service untuk monthly_fcost_part_category_breakdowns

### Phase 7 — Integrasi Frontend ↔ Backend
- [ ] Ganti mock data di semua halaman frontend dengan API calls
- [ ] Wiring composables → useFetch / useAsyncData ke API routes
- [ ] Pastikan CRUD end-to-end berfungsi di setiap halaman

### Phase 8 — Validation Engine (Backend)
- [ ] Implement semua validation rules (Section 8: V1–V8)
- [ ] API endpoint `/api/validation/run`
- [ ] Simpan hasil ke validation_runs
- [ ] Wiring ke halaman `/validation` — ganti mock dengan data real

### Phase 9 — Report View Model & Preview
- [ ] Buat report builder service: transform long-format → report view model
- [ ] Mapping grouping model
- [ ] Periode trend FQMS dan current fiscal half F-COST sesuai rule discovery
- [ ] Wiring ke halaman `/reports/preview`

### Phase 10 — Export PDF
- [ ] Layout print-friendly HTML/CSS untuk report
- [ ] Endpoint export PDF via Playwright
- [ ] Auto-naming file output
- [ ] Simpan ke export_jobs

### Phase 11 — Export Excel
- [ ] Siapkan template `.xlsx`
- [ ] Mapping data → cell/range via ExcelJS
- [ ] Endpoint export Excel
- [ ] Simpan ke export_jobs

### Phase 12 — Backup / Restore
- [ ] Endpoint export backup database SQLite
- [ ] Endpoint restore dari file backup
- [ ] UI tombol di `/settings/backup`

### Phase 13 — Polish & Finishing
- [ ] Dashboard trend & ringkasan data
- [ ] Export history di `/reports/export`
- [ ] Empty state & error state yang informatif
- [ ] Shortcut keyboard untuk data entry
- [ ] Optimasi performa query

---

## 12 · Scope MVP vs Out of Scope

### ✅ Masuk MVP
- CRUD semua entitas data (Section 6)
- Validation engine 8 rules (Section 8)
- Report preview dari database
- Export PDF 1 report utama
- Export Excel 1 template utama
- Backup / restore database

### ❌ Tidak masuk MVP
- Import otomatis dari workbook historis lama
- Role-based approval / multi-user permission
- Cloud sync / real-time collaboration
- Automated email sending
- AI insight / advanced dashboard
- Full historical migration
- Power Query replacement

---

## 13 · Acceptance Criteria MVP

MVP **selesai** jika user dapat menjalankan **1 siklus bulanan penuh** tanpa membuka Excel sebagai pusat kerja:

1. ✅ Membuat bulan laporan baru
2. ✅ Mengelola master model
3. ✅ Input summary, defect, non-defect, repair action, F-COST
4. ✅ Menjalankan validasi → semua status utama OK
5. ✅ Preview report
6. ✅ Export PDF
7. ✅ Export Excel
8. ✅ Backup database

---

## 14 · Fiscal Calendar (Japanese Corporate)

| Term | Period |
|---|---|
| **FH** (First Half) | 1 April – 30 September |
| **LH** (Last Half) | 1 October – 31 March |
| **Label** | Tahun kalender dari April (awal FY) |

Contoh:
- `2025FH` = 1 Apr 2025 – 30 Sep 2025
- `2025LH` = 1 Oct 2025 – 31 Mar 2026

---

## 15 · Git Workflow

```
Main branch:    main
Feature branch: feature/<nama-fitur>
Bugfix branch:  bugfix/<nama-bug>

Commit message:
  feat:     Tambah fitur X
  fix:      Perbaiki bug Y
  docs:     Update dokumentasi
  refactor: Refactor modul Z
  test:     Tambah test untuk X
```

---

## 16 · Risiko & Mitigasi

| Risiko | Mitigasi |
|---|---|
| Layout PDF/Excel tidak mirip report lama | Pakai template Excel + HTML print layout, iterasi dari output nyata |
| Data model terlalu cepat kompleks | Mulai dari field yang jelas dipakai report, tambah sesuai kebutuhan |
| Import Excel lama memakan waktu besar | Tunda import otomatis, sediakan input manual dulu |
| Database lokal hilang | Backup/restore wajib ada sejak MVP |
| Angka report salah karena grouping | Grouping sebagai reference table eksplisit + validation page |
| Scope melebar terlalu cepat | Fokus 1 alur kerja sampai selesai sebelum perluas |

---

## 17 · Glossary

| Istilah | Definisi |
|---|---|
| **FQMS** | Field Quality Management System — sistem tracking kualitas produk di pasar |
| **F-COST** | Failure Cost — biaya yang timbul dari klaim kualitas |
| **PPM** | Parts Per Million — metrik kualitas (qty / sales × 1.000.000) |
| **Report Month** | Bulan laporan yang sedang dikerjakan (format YYYYMM) |
| **Monthly Summary** | Ringkasan data per model per bulan: sales, defect, non-defect |
| **Defect Category** | Klasifikasi defect: Power Unit, Main Unit, Panel, Software, Other |
| **Non-defect Category** | Klasifikasi non-defect: User Usage, Setting, Explanation, Signal |
| **Repair Action** | Tindakan perbaikan yang dilakukan terhadap defect |
| **Model Group Rule** | Aturan penggabungan model di report (misal HD1400I → HD1500I) |
| **View Model** | Struktur data yang sudah ditransformasi dari DB ke format siap report |
| **Long-format** | Data disimpan sebagai baris per record (DB storage) |
| **Wide-format** | Data disusun sebagai kolom per bulan (report layout) |
| **QRCC** | Quality Reliability Control Center |
