# PRD — QRCC FQMS/F-COST Monthly Workflow

## Ringkasan produk

Produk ini adalah aplikasi web internal QRCC untuk menggantikan workflow database Excel bulanan FQMS/F-COST yang saat ini masih banyak bergantung pada workbook, sheet helper, copy-paste, refresh manual, dan pengecekan lintas file. Produk ini menjadi bagian dari sistem QRCC Data Center, bukan aplikasi personal yang berdiri sendiri. Tujuan utamanya bukan membuat sistem enterprise yang kompleks, tetapi membuat tool kerja QRCC yang mempercepat pekerjaan rutin bulanan, mengurangi risiko salah input, dan menghasilkan output report PDF/Excel dengan layout yang familiar seperti report Excel saat ini.

Aplikasi ini akan menjadi sumber data utama untuk data FQMS, F-COST, model, kategori defect, kategori non-defect, repair action, sales quantity, PPM, dan report bulanan. Excel tidak lagi diposisikan sebagai database utama, tetapi tetap bisa dipakai sebagai format output, format template, atau sumber import pada masa transisi.

Stack utama yang direkomendasikan adalah Nuxt 4, TypeScript strict, Vue, Nuxt UI 4, SQLite, Drizzle ORM, Better Auth bila dibutuhkan, ExcelJS untuk export Excel, SheetJS untuk import Excel bila dibutuhkan, dan Playwright/Puppeteer untuk generate PDF berbasis HTML/CSS.

## Latar belakang masalah

Workflow saat ini sudah mulai diarahkan ke desain database yang lebih baik, yaitu data panjang, master table, database bulanan, validasi, dan file presentasi yang membaca dari database. Namun selama masih berbasis Excel, proses bulanan tetap memiliki beberapa friction: workbook harus dibuka, query perlu di-refresh, formula/chart perlu dicek, output perlu disimpan manual, dan beberapa proses masih bergantung pada struktur workbook yang rentan berubah.

Sebagai bagian dari sistem QRCC, aplikasi web ini menyederhanakan proses menjadi satu tempat kerja yang konsisten dengan identitas QRCC Data Center. User cukup membuka aplikasi QRCC, memilih bulan laporan, input atau import data, melihat validasi, lalu generate PDF/Excel. Proses yang sebelumnya tersebar di database workbook, job 4, job 5, dan file F-COST dapat diringkas ke dalam flow yang lebih pendek dan lebih terkontrol.

## Tujuan utama

Tujuan produk ini adalah membuat pekerjaan bulanan menjadi lebih cepat, konsisten, dan mudah diaudit. Data cukup masuk satu kali ke database aplikasi, kemudian semua output seperti summary, defect category, non-defect category, repair action, F-COST, PDF, dan Excel report dihasilkan otomatis dari sumber data yang sama.

Aplikasi harus membantu user menghindari copy-paste manual, geser kolom bulan, update angka yang sama di banyak sheet, dan pengecekan manual lintas workbook. Fokus produk adalah produktivitas kerja QRCC: cepat dipakai, tidak banyak konfigurasi, mudah backup, dan mudah diubah mengikuti format report yang sudah ada.

## Non-goals awal

Versi awal tidak bertujuan menjadi sistem multi-user enterprise, tidak perlu approval workflow yang kompleks, tidak perlu deployment cloud sejak hari pertama, dan tidak perlu mengganti semua file Excel historis sekaligus. Aplikasi juga tidak perlu meniru semua perilaku Excel 1:1 di UI. Yang penting adalah data benar, flow bulanan pendek, validasi jelas, dan output report memenuhi kebutuhan meeting.

## Target pengguna

Pengguna utama adalah operator atau pemilik workflow bulanan FQMS/F-COST di lingkungan QRCC LCD Local. Karena targetnya penggunaan internal QRCC yang dominan dijalankan oleh satu operator, desain aplikasi boleh mengutamakan efisiensi operator tunggal dibanding kebutuhan kolaborasi tim besar. Namun struktur data tetap harus rapi agar suatu saat bisa dikembangkan menjadi multi-user jika dibutuhkan.

## Prinsip produk

Aplikasi harus mengikuti prinsip single source of truth. Data utama disimpan di SQLite, bukan tersebar di banyak workbook. Report PDF dan Excel adalah output, bukan tempat input utama. Semua perhitungan penting seperti total defect, total non-defect, sales quantity, PPM, F-COST ratio, dan category summary harus berasal dari database yang sama.

Aplikasi harus mempertahankan konsep long-format data. Data bulanan disimpan sebagai baris dengan kolom `month`, `model`, `category`, `qty`, dan atribut lain, bukan sebagai kolom bulan yang terus bergeser. Layout report boleh tetap wide-format agar mudah dibaca dalam meeting, tetapi database internal harus tetap panjang.

Aplikasi harus dibuat untuk flow pendek. Idealnya pekerjaan bulanan cukup mengikuti pola: pilih bulan laporan, input/import data, cek validasi, preview report, generate PDF/Excel, simpan output.

## MVP core

MVP pertama berfokus pada CRUD data dan report generation. Import Excel otomatis penuh boleh masuk fase berikutnya, karena MVP harus terlebih dahulu membuktikan bahwa aplikasi bisa menjadi database utama dan menghasilkan report yang berguna.

MVP core terdiri dari master data, monthly data entry, validation, report preview, export PDF, export Excel, dan backup database. Fitur dashboard QRCC yang lebih luas, multi-user permission, audit trail detail, dan import semua workbook historis dapat ditunda.

## Scope MVP

MVP harus menyediakan pengelolaan master model. User bisa menambah, mengedit, menonaktifkan, dan menentukan model mana yang masuk report. Field minimal mencakup model name, model series, product, active status, report include, effective from, effective to, dan remark. Aturan khusus seperti HD1400I digabung ke HD1500I pada report harus disimpan sebagai mapping yang eksplisit, bukan hardcode tersembunyi.

MVP harus menyediakan pengelolaan reference table. Reference table minimal mencakup defect category, non-defect category, repair action, factory atau product grouping jika diperlukan, dan comment/category mapping bila masih relevan. Reference table ini menjadi dasar normalisasi data.

MVP harus menyediakan CRUD monthly model summary. User bisa mengisi atau mengedit data per bulan dan per model, seperti sales period, sales quantity, total defect quantity, total non-defect quantity, defect PPM, non-defect PPM, dan remark. PPM sebaiknya dihitung otomatis dari qty dan sales quantity, tetapi user tetap bisa melihat hasilnya dengan jelas.

MVP harus menyediakan CRUD defect category. User bisa mengisi data defect per bulan, model, dan kategori seperti Power Unit, Main Unit, Panel, Software, dan Other. Aplikasi menghitung total per model dan memastikan total kategori sama dengan total defect summary.

MVP harus menyediakan CRUD non-defect category. User bisa mengisi data non-defect per bulan, model, dan kategori seperti User Usage, Setting, Explanation, Signal, atau kategori final lain yang dipakai. Aplikasi menghitung total per model dan memastikan total kategori sama dengan total non-defect summary.

MVP harus menyediakan CRUD repair action. User bisa mengisi repair action per bulan dan model, termasuk action name, qty, total, defect occupation, dan PPM jika dibutuhkan untuk report.

MVP harus menyediakan CRUD F-COST. User bisa mengisi data F-COST bulanan seperti sales amount, sales quantity, F-COST amount, F-COST quantity, target, actual ratio, last year result, dan remark. Jika data F-COST masih berasal dari tool lama, MVP cukup menyediakan input manual atau paste-friendly table terlebih dahulu.

MVP harus menyediakan validation page. Halaman ini menampilkan status OK/CHECK untuk jumlah model aktif yang masuk report, missing data untuk bulan laporan, duplicate model/month, total defect category versus summary, total non-defect category versus summary, konsistensi sales quantity, dan konsistensi bulan laporan.

MVP harus menyediakan report preview. User bisa memilih report month lalu melihat preview halaman report dalam layout web yang mirip output Excel. Preview ini menjadi sumber untuk PDF generation.

MVP harus menyediakan export PDF. PDF harus menggunakan layout yang mirip report Excel saat ini, minimal mencakup report summary, defective, non-defective, defect category by model, repair action, dan F-COST summary. Untuk MVP, PDF tidak harus langsung 100% identik pixel-by-pixel, tetapi harus cukup rapi untuk dipakai review dan sebagai dasar penyempurnaan layout.

MVP harus menyediakan export Excel. Export Excel bisa dibuat dari template `.xlsx` atau dibuat programatis dengan ExcelJS. Untuk menjaga layout familiar, pendekatan template lebih disarankan: aplikasi mengisi data ke template report yang sudah disiapkan, lalu menghasilkan file report baru untuk bulan terpilih.

MVP harus menyediakan backup/restore sederhana. Karena aplikasi memakai SQLite untuk penggunaan internal QRCC yang local-first, harus ada tombol atau command untuk export backup database dan restore dari backup. Ini penting agar aplikasi tidak menjadi single point of failure.

## Out of scope untuk MVP

Import otomatis dari seluruh workbook model active, import Power Query replacement penuh, role-based approval, cloud sync, real-time collaboration, automated email sending, advanced dashboard, AI insight, dan full historical migration tidak masuk MVP pertama. Fitur-fitur tersebut bisa dibuat setelah CRUD dan report generation stabil.

## User stories MVP

Sebagai user, saya ingin membuat bulan laporan baru agar saya bisa mulai proses FQMS/F-COST untuk bulan berjalan tanpa membuat banyak folder dan workbook manual.

Sebagai user, saya ingin mengelola daftar model aktif agar report hanya mengambil model yang memang masuk keputusan management.

Sebagai user, saya ingin menginput sales, defect, non-defect, repair action, dan F-COST di satu aplikasi agar saya tidak perlu update angka yang sama di banyak workbook.

Sebagai user, saya ingin melihat status validasi OK/CHECK sebelum generate report agar kesalahan data diketahui sebelum meeting.

Sebagai user, saya ingin preview report sebelum export agar saya bisa memastikan layout dan angka sudah benar.

Sebagai user, saya ingin generate PDF dengan layout mirip Excel agar report bisa langsung dibagikan atau diarsipkan.

Sebagai user, saya ingin export Excel dengan format mirip report sekarang agar tetap bisa dicek manual atau digunakan jika meeting masih membutuhkan file Excel.

Sebagai user QRCC, saya ingin backup database agar data workflow QRCC aman dan mudah dipindahkan.

## Flow pengguna MVP

Flow utama dimulai dari halaman bulan laporan. User membuat atau memilih report month, misalnya `2026-04`. Setelah itu user membuka halaman master model untuk memastikan daftar model aktif sudah benar. Kemudian user mengisi monthly summary, defect category, non-defect category, repair action, dan F-COST melalui form atau editable table.

Setelah data masuk, user membuka validation page. Jika ada status CHECK, aplikasi menampilkan penyebabnya, misalnya model active belum punya data, total defect category tidak sama dengan total summary, atau sales quantity berbeda antar bagian. User memperbaiki data dari halaman terkait sampai semua validasi penting berstatus OK.

Setelah validasi OK, user membuka report preview. Preview menampilkan layout report dengan data bulan terpilih dan enam bulan terakhir jika data historis tersedia. User kemudian bisa generate PDF atau Excel. File output diberi nama otomatis berdasarkan report type dan bulan, misalnya `QRCC FQMS - LCD SEID Apr 2026.pdf` atau `QRCC Data Presentasi Meeting LCD SEID Apr 2026.xlsx`.

## Rekomendasi tech stack

Stack utama yang direkomendasikan adalah Nuxt sebagai full-stack framework. Nuxt menangani UI, routing, server API, file generation endpoint, dan integrasi database. TypeScript dipakai di seluruh codebase agar schema, query, dan report mapping lebih aman.

Vue menjadi layer UI utama. Untuk komponen, gunakan Nuxt UI 4 karena komponennya terintegrasi dengan ekosistem Nuxt dan cukup cepat untuk kebutuhan internal QRCC. Untuk MVP ini, Nuxt UI 4 adalah pilihan default. Jika ada kebutuhan tampilan report/editor yang sangat custom, tetap usahakan tetap berada dalam pola komponen Nuxt UI 4 agar konsisten.

SQLite menjadi database utama. Untuk penggunaan internal QRCC yang local-first, SQLite sangat cocok karena ringan, tidak butuh server database terpisah, mudah dibackup, dan cukup kuat untuk data bulanan FQMS/F-COST. Jika suatu hari aplikasi berkembang menjadi multi-user atau cloud-hosted, PostgreSQL bisa dipertimbangkan tanpa mengubah konsep data secara besar.

Drizzle ORM dipakai untuk schema dan query. Drizzle cocok karena ringan, TypeScript-friendly, dan membuat struktur database lebih eksplisit. Drizzle migration juga membantu menjaga perubahan schema tetap terkontrol.

Better Auth bisa dipakai jika aplikasi tetap membutuhkan login, walaupun untuk penggunaan internal QRCC. Jika aplikasi hanya berjalan lokal di komputer internal, auth bisa dibuat opsional pada MVP. Namun jika aplikasi di-deploy ke server internal atau jaringan lokal, Better Auth sebaiknya dipakai sejak awal.

ExcelJS dipakai untuk export Excel. Untuk layout report yang mirip file existing, aplikasi sebaiknya membaca template `.xlsx`, mengisi cell/range tertentu, lalu menyimpan file output. Jika template approach kurang fleksibel, ExcelJS juga bisa membuat workbook dari nol, tetapi effort styling akan lebih besar.

SheetJS dipakai untuk import Excel bila diperlukan. Pada MVP, SheetJS bisa ditunda kecuali user ingin paste/import data dari workbook lama. Jika import Excel masuk fase berikutnya, SheetJS membantu membaca workbook, sheet, dan data table menjadi JSON.

Playwright atau Puppeteer dipakai untuk generate PDF. Strategi terbaik adalah membuat halaman report HTML/CSS yang sudah print-friendly, lalu render ke PDF. Dengan pendekatan ini, layout PDF bisa dikontrol melalui CSS, page size, margin, font, border, dan page break.

Zod direkomendasikan untuk validasi input di server route. Komponen table/form dari Nuxt UI 4 menjadi pilihan utama untuk editable grid MVP. Jika data entry menjadi sangat spreadsheet-like, AG Grid Community bisa dipertimbangkan sebagai pengecualian, tetapi untuk MVP sebaiknya mulai dari table/form sederhana yang konsisten dengan Nuxt UI 4.

## Arsitektur aplikasi

Aplikasi dapat dibuat sebagai Nuxt full-stack monolith. Struktur ini paling sederhana untuk penggunaan internal QRCC karena frontend, backend API, database access, dan file generation berada dalam satu project.

Layer UI berada di pages dan components. Halaman utama mencakup dashboard, report months, model master, monthly summary, defect category, non-defect category, repair action, F-COST, validation, report preview, export history, dan settings.

Layer server berada di server/api dan server/services. API route menerima request dari UI, memvalidasi input, lalu memanggil service. Service menangani business logic seperti perhitungan PPM, validasi total, report query, dan export file.

Layer database berada di server/db. Drizzle schema mendefinisikan table, relation, migration, dan query helper. Semua perhitungan yang penting harus mengambil data dari database, bukan dari state UI.

Layer report berada di server/reports. Modul ini menangani transformasi data menjadi view model untuk PDF dan Excel. View model ini penting agar logic report tidak tersebar di komponen UI dan generator file.

Layer template berada di templates. Template Excel dan template HTML/CSS report disimpan terpisah agar mudah diedit tanpa mengganggu schema database.

## Struktur folder awal yang disarankan

```text
qrcc-fqms-fcost-app/
  app/
    components/
    pages/
    layouts/
    composables/
  server/
    api/
    db/
      schema.ts
      client.ts
      migrations/
    services/
      model.service.ts
      monthly-summary.service.ts
      defect-category.service.ts
      nondefect-category.service.ts
      repair-action.service.ts
      fcost.service.ts
      validation.service.ts
      report.service.ts
    reports/
      pdf/
      excel/
      view-models/
  templates/
    excel/
    pdf/
  storage/
    exports/
    backups/
    imports/
  shared/
    types/
    constants/
    validators/
```

## Data model awal

Table `report_months` menyimpan bulan laporan. Field minimal: `id`, `month_key`, `month_label`, `period_start`, `period_end`, `status`, `created_at`, dan `updated_at`. `month_key` sebaiknya memakai format stabil seperti `202604`, sedangkan label bisa `Apr-26` atau `April 2026`.

Table `models` menyimpan master model. Field minimal: `id`, `model_name`, `model_series`, `product`, `active_status`, `report_include`, `effective_from`, `effective_to`, `remark`, `created_at`, dan `updated_at`.

Table `model_group_rules` menyimpan aturan grouping khusus. Field minimal: `id`, `source_model`, `report_model`, `rule_type`, `active`, dan `remark`. Contohnya, source `HD1400I` dapat masuk ke report model `HD1500I`.

Table `defect_categories` menyimpan kategori defect standar. Field minimal: `id`, `code`, `name`, `sort_order`, dan `active`. Contoh kategori: Power Unit, Main Unit, Panel, Software, Other.

Table `nondefect_categories` menyimpan kategori non-defect standar. Field minimal: `id`, `code`, `name`, `sort_order`, dan `active`.

Table `monthly_model_summaries` menyimpan summary per bulan dan model. Field minimal: `id`, `report_month_id`, `model_id`, `sales_period`, `sales_qty`, `total_defect_qty`, `total_nondefect_qty`, `defect_ppm`, `nondefect_ppm`, `remark`, `created_at`, dan `updated_at`. Kombinasi `report_month_id` dan `model_id` harus unik.

Table `defect_category_entries` menyimpan defect per kategori. Field minimal: `id`, `report_month_id`, `model_id`, `category_id`, `qty`, `sales_qty_snapshot`, `ppm`, dan `remark`. Kombinasi `report_month_id`, `model_id`, dan `category_id` harus unik.

Table `nondefect_category_entries` menyimpan non-defect per kategori. Field minimal: `id`, `report_month_id`, `model_id`, `category_id`, `qty`, `sales_qty_snapshot`, `ppm`, dan `remark`.

Table `repair_action_entries` menyimpan repair action. Field minimal: `id`, `report_month_id`, `model_id`, `repair_action`, `qty`, `total`, `defect_occup`, `defect_ppm`, dan `remark`.

Table `fcost_entries` menyimpan F-COST bulanan. Field minimal: `id`, `report_month_id`, `sales_amount`, `sales_qty`, `fcost_amount`, `fcost_qty`, `target_ratio`, `actual_ratio`, `last_year_result`, dan `remark`.

Table `validation_runs` menyimpan hasil validasi. Field minimal: `id`, `report_month_id`, `status`, `checked_at`, dan `summary_json`. Untuk MVP, validation result juga bisa dihitung real-time tanpa disimpan, tetapi menyimpan snapshot membantu audit.

Table `export_jobs` menyimpan riwayat export. Field minimal: `id`, `report_month_id`, `type`, `file_name`, `file_path`, `status`, `created_at`, dan `message`.

## Aturan perhitungan

PPM dihitung dengan rumus `qty / sales_qty * 1,000,000`. Jika sales quantity kosong atau nol, PPM harus ditampilkan sebagai kosong, nol, atau status CHECK sesuai keputusan produk. Untuk menghindari angka yang menyesatkan, lebih aman menampilkan CHECK jika qty ada tetapi sales quantity nol.

Total defect category per model dan bulan harus sama dengan `total_defect_qty` pada monthly summary. Total non-defect category per model dan bulan harus sama dengan `total_nondefect_qty`. Jika tidak sama, report masih boleh dipreview tetapi export final sebaiknya memberi warning besar atau dikunci sampai user override secara sadar.

Sales quantity harus konsisten. Jika sales quantity digunakan di summary, category, dan report, sumber utama harus `monthly_model_summaries.sales_qty`. Field `sales_qty_snapshot` pada category entry boleh ada untuk audit atau performa, tetapi tidak boleh menjadi sumber utama jika berbeda dari summary.

Report model harus mengikuti grouping rule. Jika ada model yang secara data original berbeda tetapi harus tampil sebagai satu model series pada report, aplikasi harus menyatukan data saat membuat report view model, bukan menghapus detail original.

## Validation rules MVP

Validation pertama adalah active model completeness. Semua model dengan `active_status = Active` dan `report_include = true` harus memiliki summary untuk report month terpilih.

Validation kedua adalah duplicate summary. Tidak boleh ada lebih dari satu summary untuk kombinasi report month dan model.

Validation ketiga adalah defect total matching. Jumlah semua defect category entries untuk model dan bulan harus sama dengan total defect quantity pada summary.

Validation keempat adalah non-defect total matching. Jumlah semua non-defect category entries untuk model dan bulan harus sama dengan total non-defect quantity pada summary.

Validation kelima adalah sales quantity presence. Model yang memiliki defect atau non-defect quantity harus memiliki sales quantity yang valid.

Validation keenam adalah category standardization. Semua category entry harus memakai category id dari reference table, bukan free text yang bisa berbeda penulisan.

Validation ketujuh adalah report month consistency. Semua data yang dipakai report harus berasal dari report month yang sama, kecuali bagian trend yang memang mengambil beberapa bulan historis.

Validation kedelapan adalah F-COST completeness. Jika report F-COST diaktifkan untuk bulan tersebut, data F-COST wajib tersedia.

## Report PDF MVP

PDF MVP harus dibuat dari HTML/CSS print layout. Aplikasi membuat route preview, misalnya `/reports/fqms/:month/preview`, lalu Playwright membuka route tersebut secara server-side dan menyimpannya sebagai PDF. Dengan cara ini, tampilan preview dan output PDF berasal dari sumber layout yang sama.

PDF harus memiliki ukuran halaman, margin, font, border, table style, dan page break yang konsisten. Untuk awal, gunakan layout yang mendekati Excel report, lalu sempurnakan berdasarkan hasil print. Komponen report sebaiknya dipisahkan menjadi section seperti cover/header, quality trend, acceptance ratio, model detail, defect category, repair action, dan F-COST.

PDF export harus menerima parameter report month dan report type. Output file harus dinamai otomatis dan masuk ke folder export aplikasi. Setelah export selesai, user bisa membuka atau menyimpan file tersebut.

## Export Excel MVP

Excel export MVP sebaiknya menggunakan template workbook. Template berisi layout report, merged cells, border, style, print area, dan sheet structure. Aplikasi hanya mengisi cell atau named range dengan data dari report view model.

Pendekatan template mengurangi risiko perbedaan tampilan karena styling sudah dibuat di Excel. ExcelJS dapat membuka template, mengisi cell, menambah row jika dibutuhkan, lalu menyimpan workbook baru. Untuk report tabel yang jumlah barisnya berubah, template perlu menyediakan area dinamis atau aplikasi perlu melakukan insert row dengan style copy.

Jika report harus mendukung banyak jenis output, pisahkan template menjadi `job4-presentation-template.xlsx`, `fqms-product-template.xlsx`, dan `fcost-template.xlsx`. Untuk MVP, cukup mulai dari satu template report yang paling sering dipakai.

## UI MVP

Halaman dashboard menampilkan report month aktif, status validasi terakhir, shortcut ke input data, dan tombol generate report. Karena tujuan aplikasi adalah produktivitas, dashboard harus langsung menjawab: bulan apa yang sedang dikerjakan, data apa yang belum lengkap, dan tombol apa yang harus ditekan berikutnya.

Halaman report months memungkinkan user membuat bulan baru, menandai bulan sebagai draft, validated, exported, atau archived. Status ini membantu membedakan bulan yang masih dikerjakan dan bulan yang sudah final.

Halaman master data berisi model, kategori defect, kategori non-defect, dan grouping rule. UI bisa dibuat dengan table sederhana, search, filter active, dan form edit drawer/modal.

Halaman data entry berisi monthly summary, defect category, non-defect category, repair action, dan F-COST. Untuk MVP, masing-masing bisa menjadi tab. Table harus mendukung edit cepat, copy-paste dari spreadsheet jika memungkinkan, dan highlight error.

Halaman validation menampilkan daftar checks dengan status OK/CHECK, expected value, actual value, selisih, dan link ke data yang perlu diperbaiki.

Halaman report preview menampilkan layout report. Tombol export PDF dan export Excel ditempatkan di halaman ini, tetapi jika validasi belum OK, tombol bisa menampilkan warning.

Halaman settings berisi lokasi export, backup database, restore database, dan preferensi format bulan.

## Copywriting dan metadata app

Karena aplikasi ini adalah bagian dari sistem QRCC, seluruh copywriting, metadata, title, description, menu, empty state, dan export naming harus mengarah ke identitas QRCC Data Center. Judul aplikasi direkomendasikan memakai `QRCC Data Center`, dengan subtitle atau deskripsi yang menjelaskan bahwa modul ini menangani FQMS, F-COST, PPM, Sales, dan Quality Issue workflow.

Contoh metadata awal: title `QRCC Data Center`; description `Aplikasi dashboard internal QRCC untuk mengelola workflow FQMS, F-COST, PPM, Sales, Quality Issue, validasi data bulanan, dan report generation.` Menu utama juga harus memakai istilah domain QRCC, seperti Market Quality, F-Cost, Sales, Report Month, Validation, Report Preview, Export, dan Settings.

## Acceptance criteria MVP

MVP dianggap berhasil jika user dapat membuat report month baru, mengelola master model, menginput summary, defect category, non-defect category, repair action, dan F-COST untuk satu bulan, menjalankan validasi sampai status utama OK, melihat preview report, menghasilkan PDF, menghasilkan Excel, dan membuat backup database.

MVP juga dianggap berhasil jika total defect dan non-defect pada report selalu berasal dari database yang sama dan tidak perlu copy-paste antar workbook. User harus bisa menyelesaikan proses utama dengan jauh lebih sedikit langkah dibanding workflow Excel lama.

## Roadmap implementasi

Phase 0 adalah discovery dan template capture. Pada fase ini, kumpulkan contoh report Excel final, tentukan sheet/section yang wajib ditiru, definisikan field utama, dan pilih apakah MVP pertama meniru Job 4, Job 5, atau gabungan minimal keduanya.

Phase 1 adalah project setup. Buat project Nuxt + TypeScript, pilih UI library, setup SQLite, Drizzle, migration, layout dasar, dan halaman dashboard kosong.

Phase 2 adalah database core. Implementasikan schema report months, models, category references, monthly summaries, defect entries, non-defect entries, repair action, dan F-COST. Tambahkan seed data untuk kategori standar.

Phase 3 adalah CRUD core. Buat halaman dan API untuk master model, report month, monthly summary, defect category, non-defect category, repair action, dan F-COST.

Phase 4 adalah validation engine. Buat service validasi, halaman validation, status OK/CHECK, dan link ke data bermasalah.

Phase 5 adalah report view model. Buat query yang mengubah database long-format menjadi struktur yang siap ditampilkan di report, termasuk grouping model dan enam bulan terakhir bila data tersedia.

Phase 6 adalah PDF preview dan export. Buat template HTML/CSS report, preview page, dan endpoint export PDF dengan Playwright/Puppeteer.

Phase 7 adalah Excel export. Siapkan template `.xlsx`, mapping cell/range, dan endpoint export Excel dengan ExcelJS.

Phase 8 adalah backup/restore. Tambahkan export database backup, restore, dan dokumentasi penggunaan internal QRCC.

Phase 9 adalah import Excel opsional. Setelah MVP stabil, tambahkan import dari workbook lama menggunakan SheetJS, dimulai dari file yang paling sering dipakai atau format yang paling stabil.

Phase 10 adalah polish. Tambahkan dashboard trend, export history, data duplicate detection lebih detail, shortcut keyboard, dan optimasi data entry.

## Risiko dan mitigasi

Risiko pertama adalah layout PDF/Excel tidak langsung identik dengan report Excel lama. Mitigasinya adalah memakai template Excel untuk output Excel dan HTML print layout yang disempurnakan bertahap untuk PDF.

Risiko kedua adalah data model terlalu cepat dibuat kompleks. Mitigasinya adalah mulai dari field yang sudah jelas dipakai report, lalu tambah field setelah ada kebutuhan nyata.

Risiko ketiga adalah import Excel lama memakan waktu terlalu besar. Mitigasinya adalah menunda import otomatis penuh dari MVP dan menyediakan input manual/paste-friendly terlebih dahulu.

Risiko keempat adalah aplikasi internal QRCC menjadi sulit dipindahkan karena file database lokal. Mitigasinya adalah backup/restore wajib ada sejak MVP.

Risiko kelima adalah report menghasilkan angka yang salah karena grouping atau category mapping. Mitigasinya adalah menjadikan grouping dan category sebagai reference table yang terlihat di UI, serta membuat validation page yang eksplisit.

## Keputusan teknis yang direkomendasikan

Gunakan Nuxt full-stack monolith, bukan frontend dan backend terpisah, karena target awal adalah penggunaan internal QRCC dan produktivitas. Gunakan SQLite lokal sebagai database utama. Gunakan Drizzle untuk schema dan migration. Gunakan Nuxt UI 4 untuk mempercepat MVP. Gunakan Playwright untuk PDF karena hasil print dari HTML lebih mudah dikontrol. Gunakan ExcelJS dengan template workbook untuk export Excel.

Auth bisa dibuat opsional untuk local-only MVP. Jika aplikasi akan dijalankan di server atau dibuka dari perangkat lain, gunakan Better Auth sejak awal.

## Definisi selesai untuk MVP

MVP selesai ketika aplikasi bisa dipakai untuk satu siklus report bulanan dari awal sampai output. Satu siklus berarti user dapat membuat bulan laporan, input data utama, melihat validasi, memperbaiki error, preview report, export PDF, export Excel, dan backup database. Jika semua itu bisa dilakukan tanpa membuka database Excel lama sebagai pusat kerja, maka MVP sudah memenuhi tujuan utama.

## Catatan untuk project baru

Project baru sebaiknya tidak dimulai dengan membuat semua fitur sekaligus. Mulailah dari database schema dan satu report paling penting. Setelah satu report berhasil dari input sampai PDF/Excel, baru perluas ke section lain. Jangan mulai dari dashboard besar atau import semua file historis, karena itu bisa membuat scope terlalu lebar sebelum core workflow terbukti.

Prioritas pertama adalah membuat workflow bulanan terasa pendek. Jika aplikasi membuat user harus melakukan terlalu banyak klik atau input berulang, berarti produk belum menyelesaikan masalah utama. Setiap fitur harus diuji dengan pertanyaan: apakah ini mengurangi pekerjaan bulanan, mengurangi risiko salah angka, atau membuat output lebih cepat siap meeting? Jika tidak, fitur tersebut bisa ditunda.
