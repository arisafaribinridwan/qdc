# System Design & MVP Plan — QRCC FQMS/F-COST Workflow

## Context

Project ini akan dibangun sebagai web app internal QRCC untuk menggantikan workflow bulanan FQMS/F-COST yang selama ini masih bergantung pada Excel workbook, sheet helper, copy-paste, dan pengecekan manual lintas file.

Tujuan utamanya bukan membuat platform enterprise, tetapi membuat alat kerja internal QRCC yang lebih cepat, lebih rapi, dan lebih aman untuk menghasilkan report bulanan.

Repo yang sudah ada adalah baseline yang bagus: ini sudah Nuxt 4 + Nuxt UI 4 + TypeScript, dengan dashboard template yang bisa dipakai ulang. Jadi kita tidak mulai dari nol; kita repurpose fondasi yang sudah ada.

## Asumsi MVP

Untuk MVP pertama, kita anggap report yang wajib ditutup end-to-end adalah FQMS dan F-COST dalam satu workflow bulanan. Artinya user bisa:

- memilih atau membuat bulan laporan
- mengisi master model dan reference data
- menginput monthly summary, defect category, non-defect category, repair action, dan F-COST
- menjalankan validasi
- melihat preview report
- export PDF
- export Excel
- backup dan restore database

## Prinsip desain

Aplikasi harus mengikuti prinsip single source of truth. Semua data utama tinggal di SQLite, bukan di Excel.

Data internal harus long-format. Excel wide-format hanya untuk output, bukan model penyimpanan.

Setiap fitur harus mengurangi kerja bulanan, bukan menambah klik atau konfigurasi.

Dashboard yang sudah ada dipakai sebagai starting point UI, tapi isinya harus dirombak agar cocok untuk workflow FQMS/F-COST.

## Rekomendasi arsitektur

Paling cocok adalah Nuxt full-stack monolith.

Alasannya:

- satu repo, satu runtime, lebih sederhana untuk penggunaan internal QRCC yang local-first
- UI, API, query, validation, dan export bisa berada dalam satu project
- cocok untuk SQLite lokal
- mudah dipindahkan ke server internal nanti jika dibutuhkan

Komponen inti arsitektur:

- Nuxt pages/components/layouts untuk UI
- server/api untuk endpoint CRUD, validation, export, backup/restore
- server/db untuk Drizzle schema dan koneksi SQLite
- server/services untuk business logic
- server/reports untuk view model PDF/Excel
- templates untuk file template Excel dan layout report HTML/CSS

## Stack yang direkomendasikan

- Nuxt 4
- Vue 3
- TypeScript strict
- Nuxt UI 4
- SQLite
- Drizzle ORM
- Zod
- ExcelJS untuk export Excel
- Playwright untuk render PDF dari HTML/CSS
- Better Auth hanya jika nanti butuh akses jaringan atau multi-device

## Reuse dari dashboard yang sudah ada

Yang layak dipertahankan dari template sekarang:

- layout dashboard Nuxt UI 4
- sidebar/navigation shell
- style system dan utility components
- pola page-based routing
- pendekatan composable untuk state UI

Yang sebaiknya diganti:

- domain dan menu navigasi lama yang masih terkait customers/inbox/settings generik atau terlalu netral untuk identitas QRCC
- halaman home yang sekarang masih template demo
- chart/stat demo yang tidak relevan
- copywriting dan metadata app yang masih terlalu generik dan belum sepenuhnya menegaskan identitas QRCC

Artinya, kita ambil shell-nya, lalu ganti isi dan struktur menjadi product workflow untuk FQMS/F-COST di dalam QRCC Data Center.

## Copywriting dan metadata app

App identity harus diarahkan ke QRCC, bukan ke aplikasi personal terpisah. Title utama direkomendasikan `QRCC Data Center`. Description awal direkomendasikan `Aplikasi dashboard internal QRCC untuk mengelola workflow FQMS, F-COST, PPM, Sales, Quality Issue, validasi data bulanan, dan report generation.`

Menu dan empty state juga harus memakai bahasa domain QRCC. Contohnya: Market Quality, F-Cost, Sales, Report Month, Validation, Report Preview, Export, Backup, dan Settings. Copywriting harus menekankan bahwa modul ini membantu workflow bulanan QRCC, bukan sekadar dashboard generik.

## Domain model inti

Entitas utama yang masuk MVP:

- report_months
- models
- model_group_rules
- defect_categories
- nondefect_categories
- monthly_model_summaries
- defect_category_entries
- nondefect_category_entries
- repair_action_entries
- fcost_entries
- validation_runs
- export_jobs

Intinya, schema harus cukup untuk menyimpan data bulanan dan memetakan output report tanpa mengandalkan spreadsheet sebagai database.

## Aturan data penting

Model harus punya status aktif dan flag report include.

Grouping rule harus eksplisit, bukan hardcode tersembunyi.

Monthly summary menjadi sumber utama untuk sales qty dan total defect/non-defect.

Defect category dan non-defect category harus konsisten dengan reference table.

Jika total category tidak cocok dengan summary, validasi harus menandai CHECK.

## Page map MVP

Halaman yang saya sarankan untuk versi pertama:

- `/` dashboard ringkas
- `/report-months` daftar dan status bulan laporan
- `/models` master model
- `/references/defect-categories`
- `/references/nondefect-categories`
- `/references/grouping-rules`
- `/entry/summary`
- `/entry/defect`
- `/entry/nondefect`
- `/entry/repair-action`
- `/entry/fcost`
- `/validation`
- `/reports/preview`
- `/reports/export`
- `/settings/backup`

Dashboard utama sebaiknya menjawab tiga hal: bulan apa yang aktif, data apa yang belum lengkap, dan langkah berikutnya apa.

## Service layer

Service yang sebaiknya dibuat dari awal:

- report-month service
- model service
- reference service
- monthly summary service
- defect category service
- non-defect category service
- repair action service
- fcost service
- validation service
- report builder service
- export service
- backup service

Service layer penting supaya logic tidak tercecer di UI atau route handler.

## Validation engine

Validation MVP harus fokus pada hal-hal yang paling mungkin membuat report salah.

Minimum checks:

- model aktif yang wajib report belum punya summary
- duplicate summary untuk kombinasi bulan + model
- total defect category tidak sama dengan defect summary
- total non-defect category tidak sama dengan non-defect summary
- sales quantity kosong atau nol saat ada qty yang harus dihitung
- category entry tidak memakai referensi standar
- data report month tercampur dengan bulan lain
- F-COST belum terisi jika report month mengaktifkannya

Validation harus menghasilkan status jelas: OK atau CHECK, plus alasan dan link ke data yang perlu diperbaiki.

## Report design

Report preview dan PDF harus berasal dari sumber layout yang sama.

Cara paling aman:

- buat HTML/CSS report preview
- gunakan route khusus preview
- render PDF dari route yang sama dengan Playwright

Dengan begitu, user melihat sesuatu yang sangat dekat dengan output final.

Untuk Excel, pendekatan terbaik adalah template workbook. Layout mirip file lama lebih mudah dipertahankan jika kita isi template `.xlsx` daripada membangun semuanya dari nol.

## Backup / restore

Karena ini aplikasi internal QRCC dengan SQLite lokal, backup/restore wajib ada sejak MVP.

Backup minimal harus bisa:

- export file database
- menyimpan copy backup dengan nama yang jelas
- restore dari file backup

## Prioritas implementasi

Urutan yang paling sehat:

1. benahi shell dashboard dan information architecture
2. buat schema database
3. buat CRUD untuk master data
4. buat monthly entry screens
5. buat validation engine
6. buat report preview
7. buat export PDF
8. buat export Excel
9. buat backup/restore

## Task plan komprehensif

### Phase 0 — discovery final

- audit dashboard yang sudah ada
- identifikasi komponen Nuxt UI yang bisa dipakai ulang
- pilih struktur menu baru untuk domain FQMS/F-COST
- tentukan report utama yang akan jadi flow pertama
- kumpulkan contoh output Excel/PDF referensi jika ada

### Phase 1 — foundation

- rapikan app metadata, title, description, dan menu agar konsisten dengan identitas QRCC
- tentukan layout utama dashboard baru
- setup struktur folder yang konsisten
- siapkan konfigurasi strict TypeScript dan linting
- siapkan baseline UI theme

### Phase 2 — database core

- definisikan schema SQLite dengan Drizzle
- buat migration awal
- seed reference categories
- buat helper untuk koneksi dan query dasar

### Phase 3 — master data CRUD

- CRUD report month
- CRUD model master
- CRUD defect categories
- CRUD non-defect categories
- CRUD grouping rules

### Phase 4 — monthly data entry

- CRUD monthly summary
- CRUD defect category entries
- CRUD non-defect category entries
- CRUD repair action entries
- CRUD F-COST entries
- buat UX input cepat dan editable table yang nyaman

### Phase 5 — validation

- implement validation rules
- tampilkan status OK/CHECK
- tampilkan selisih dan sumber error
- tambahkan navigasi cepat ke form yang harus diperbaiki

### Phase 6 — report model

- buat view model report dari database
- mapping grouping model
- susun data untuk preview FQMS
- susun data untuk preview F-COST
- pastikan struktur siap untuk PDF dan Excel

### Phase 7 — preview & PDF

- buat halaman preview report
- buat layout print-friendly
- implement export PDF
- samakan preview dan output PDF

### Phase 8 — Excel export

- siapkan template workbook
- mapping data ke cell/range
- implement export Excel
- verifikasi format mirip report lama

### Phase 9 — backup / restore

- implement export backup database
- implement restore
- validasi hasil restore

### Phase 10 — polish

- perbaiki UX dashboard
- tambah shortcut kerja cepat
- tambah export history
- tambah empty-state dan error-state yang jelas
- rapikan dokumen penggunaan internal QRCC

## Risiko utama

Risiko terbesar adalah scope terlalu cepat melebar.

Cara mencegahnya:

- jangan mulai dari import historis massal
- jangan mulai dari dashboard cantik dulu
- jangan bikin semua section report sekaligus
- fokus ke satu alur kerja yang benar-benar selesai

Risiko berikutnya adalah output Excel/PDF tidak mirip file lama.

Cara mencegahnya:

- pakai template workbook untuk Excel
- pakai HTML/CSS print layout untuk PDF
- iterasi dari output nyata, bukan asumsi

## Definisi selesai untuk MVP

MVP dianggap selesai jika user bisa menjalankan satu siklus bulanan dari awal sampai output tanpa bergantung pada workbook Excel sebagai pusat kerja.

Artinya user bisa:

- membuat bulan laporan
- input data utama
- validasi
- preview
- export PDF
- export Excel
- backup database

Kalau itu sudah bisa, berarti core value app sudah tercapai.
