# Phase 0 Discovery — QRCC FQMS/F-COST

Tanggal: 2026-05-14
Status: Final discovery notes untuk dasar implementasi berikutnya.

## 1) Audit dashboard template yang sudah ada

Template yang ada masih berbasis dashboard demo Nuxt UI, dengan struktur utama:

- `app/layouts/default.vue` memakai `UDashboardGroup`, `UDashboardSidebar`, `UDashboardSearch`, dan `NotificationsSlideover`.
- `app/pages/index.vue` adalah dashboard home dengan `UDashboardPanel`, `UDashboardNavbar`, `UDashboardToolbar`, `HomeStats`, `HomeChart`, dan `HomeSales`.
- `app/pages/customers.vue`, `app/pages/inbox.vue`, dan `app/pages/settings.vue` masih berupa halaman demo/template yang bisa dijadikan referensi pola layout, tapi belum sesuai domain FQMS/F-COST.
- `app/app.vue` sudah mengatur metadata dasar aplikasi dan layout global.

Kesimpulan audit: struktur dashboard Nuxt UI 4 sudah tersedia dan cukup dekat dengan kebutuhan shell aplikasi internal, tetapi isi navigasi, halaman, dan identity masih template generik.

## 2) Komponen Nuxt UI 4 yang bisa dipakai ulang

Komponen yang sudah ada dan relevan untuk domain FQMS/F-COST:

- Layout/shell: `UDashboardGroup`, `UDashboardSidebar`, `UDashboardPanel`, `UDashboardNavbar`, `UDashboardToolbar`, `UDashboardSidebarCollapse`.
- Navigasi: `UNavigationMenu`, `UDashboardSearch`.
- Kontrol data/form: `UButton`, `UInput`, `USelect`, `UTextarea`, `UCalendar`, `UPopover`, `UTabs`, `UCheckbox`, `UPagination`.
- Display data: `UTable`, `UCard`, `UPageGrid`, `UPageCard`, `UBadge`, `UAvatar`.
- Overlay/aksi: `UDropdownMenu`, `UTooltip`, `USlideover`.

Komponen custom yang bisa dijadikan basis reuse:

- `HomeStats` → pola KPI cards untuk dashboard FQMS/F-COST.
- `HomeDateRangePicker` → pola selector periode/bulan.
- `HomePeriodSelect` → pola filter granularitas periode.
- `MembersList` dan `customers.vue` → referensi pola tabel, filter, dropdown action, dan form controls.
- `TeamsMenu` dan `UserMenu` → referensi sidebar header/footer.

## 3) Menu baru untuk domain FQMS/F-COST

Mengacu pada PRD section 9, menu utama yang akan dipakai:

- Dashboard
- Report Months
- Models
- References
  - Defect Categories
  - Non-defect Categories
  - Grouping Rules
- Entry
  - Summary
  - Defect
  - Non-defect
  - Repair Action
  - F-COST
- Validation
- Reports
  - Preview
  - Export
- Settings
  - Backup

Menu template lama seperti `Home`, `Market Quality`, `F-Cost`, dan `Sales` belum cukup spesifik untuk workflow PRD dan sebaiknya diganti dengan struktur domain di atas pada Phase 1.

## 4) Referensi output FQMS

File referensi yang dipakai:

- `FQMS - LCD LOCAL.xlsx`
- `FQMS - LCD LOCAL_rev.pdf`
- `penjelasan FQMS.pdf`

PDF berasal dari Microsoft Excel 2016, 1 halaman A4 portrait. Workbook Excel berisi 1 sheet bernama `FQMS`, print area `A1:S83`, dan 2 chart. Untuk FQMS monthly report, file Excel adalah template/source utama, sedangkan PDF adalah hasil print/export dari sheet yang sama.

Report FQMS adalah report monitoring Field Quality untuk product `LCD`, dengan manufacturer/vendor yang bisa bernilai `LOCAL` atau `IMPORT`. Report untuk `LOCAL` dan `IMPORT` harus dipisah. Product + manufacturer menjadi konteks/filter report, model monitoring, input bulanan, preview, dan export.

## 5) Target monthly PPM dan fiscal period

Target monthly PPM adalah satu nilai global untuk semua model dalam satu report. Contoh `383` berlaku untuk seluruh model di report tersebut, bukan per-model.

Nilai target global di-update setiap awal fiscal half berjalan. Contoh:

- `26F` berlaku April 2026 – September 2026, target mulai/update di April 2026.
- `26L` berlaku Oktober 2026 – Maret 2027, target mulai/update di Oktober 2026.

Saat generate report, aplikasi memilih target berdasarkan report month/fiscal half, lalu target itu dipakai untuk semua model aktif di Section C dan chart target di Section A.

## 6) Input data bulanan dan akumulasi

Input utama disimpan per bulan, bukan hanya angka akumulasi. User menginput hasil bulanan agar setiap bulan punya dedicated data yang mudah dibaca dan diaudit. Aplikasi menghitung akumulasi otomatis dari launching month model sampai report month yang dipilih.

Karena itu, `Sales`, `Defect qty`, dan `Non Defect qty` yang tampil di Section C adalah hasil kalkulasi akumulasi dari data bulanan. Aplikasi tidak perlu menyimpan formula Excel seperti `=218948+12517`; formula seperti itu hanya representasi manual di template lama untuk menjumlahkan data dari beberapa bulan/periode.

## 7) Section C — Detail Model

Section C adalah section inti laporan. Daftar model bersifat dinamis: model bisa masuk atau tidak masuk daftar monitoring. Section C mengambil model yang aktif/dimonitor untuk product + manufacturer report.

Field utama per model:

- Model
- Launching Month
- Launching Period
- Sales
- Defect Qty
- Non Defect Qty
- Total Claim
- AVG Defect PPM
- AVG Non Defect PPM
- Target Monthly PPM
- Defect Quality Level

Definisi dan formula:

```text
launching_period = selisih bulan antara report_month dan launching_month
sales = akumulasi sales bulanan dari launching_month sampai report_month
defect_qty = akumulasi defect bulanan dari launching_month sampai report_month
nondefect_qty = akumulasi non-defect bulanan dari launching_month sampai report_month
total_claim = defect_qty + nondefect_qty
avg_defect_ppm = defect_qty / (sales × launching_period) × 1.000.000
avg_nondefect_ppm = nondefect_qty / (sales × launching_period) × 1.000.000
defect_quality_level = NG jika avg_defect_ppm >= target_monthly_ppm, selain itu OK
```

Total row Section C:

```text
total_sales = sum(sales semua model aktif)
total_defect = sum(defect_qty semua model aktif)
total_nondefect = sum(nondefect_qty semua model aktif)
total_claim = total_defect + total_nondefect
total_exposure = sum(sales × launching_period semua model aktif)
total_avg_defect_ppm = total_defect / total_exposure × 1.000.000
total_avg_nondefect_ppm = total_nondefect / total_exposure × 1.000.000
```

Contoh angka `335` di report berasal dari:

```text
total_defect = 3.493
total_exposure = 10.438.222
total_avg_defect_ppm = 3.493 / 10.438.222 × 1.000.000 = 334,6355 ≈ 335
```

Jadi total AVG defect bukan average sederhana dari PPM per model, melainkan PPM total berbobot exposure `sales × launching_period`.

## 8) Section A — Quality Trend

Section A adalah grafik PPM bulanan `Target` vs `Result`. Bulan paling kanan adalah report month aktif atau bulan laporan terakhir yang dipilih. Periode bulan pada grafik bersifat dinamis mengikuti report month.

Mapping:

```text
month
result_ppm = total_avg_defect_ppm seluruh model aktif pada bulan tersebut
target_ppm = target global fiscal half yang berlaku pada bulan tersebut
```

Result dihitung memakai formula AVG defect PPM Section C:

```text
avg_defect_ppm = defect_qty / (sales_qty × launching_period) × 1.000.000
```

## 9) Section B — Acceptance Ratio

Section B memakai periode bulan yang dinamis seperti Section A. Nilai OK dan NG dihitung dari snapshot quality level model yang aktif/masuk monitoring pada bulan tersebut. `OK` dan `NG` bukan input manual, tetapi hasil perbandingan AVG defect PPM terhadap target monthly PPM global pada bulan snapshot tersebut.

Mapping:

```text
total_model = jumlah model aktif/monitoring pada bulan tersebut
ok_models = jumlah model dengan quality_level OK
ng_models = jumlah model dengan quality_level NG
acceptance_ratio = ok_models / total_model
```

Label fiscal half seperti `2025FH` mengambil nilai dari data bulan terakhir di fiscal half tersebut. Contoh: `2025FH` sama dengan data September 2025.

## 10) Section D — Worst Defect

Section D memakai data defect bulanan per model × defect category. User menginput data per bulan. Untuk report month tertentu, aplikasi menampilkan top 3 defective untuk masing-masing model aktif yang tampil di Section C.

Kolom bulan dibatasi 4 kolom:

- Kolom pertama seperti `~Dec'25` adalah total semua bulan sebelum 3 bulan terakhir.
- Tiga kolom berikutnya adalah 3 bulan terakhir sampai report month.

Contoh jika report month `Mar-26`:

```text
~Dec'25 = total semua bulan sebelum Jan-26
Jan'26
Feb'26
Mar'26
```

Formula Section D:

```text
total = bucket_older_months + month_minus_2 + month_minus_1 + report_month
defect_occup = total_defect_category / total_defect_model
defect_ppm = total_defect_category / (accumulated_sales_model × launching_period_model) × 1.000.000
```

Untuk implementasi baru, `defect_ppm` Section D memakai denominator `accumulated_sales_model × launching_period_model` agar konsisten dengan Section C dan menghasilkan nilai rata-rata yang proporsional terhadap masa monitoring.

## 11) Quality Issue and Follow Up

Bagian `Quality Issue and Follow Up` belum masuk scope FQMS MVP saat ini karena akan dibahas terpisah sebagai bagian dari Market Quality Issue. Untuk implementasi FQMS awal, bagian ini tidak perlu menjadi sumber data utama.

## 12) Konsekuensi struktur data untuk implementasi

Struktur data yang perlu dipertimbangkan pada phase implementasi berikutnya:

- Report month/context: report month, product, manufacturer, status.
- Master model: model name, product, manufacturer, launching month, active/monitoring flag.
- Fiscal target: fiscal half, period start/end, target monthly PPM global.
- Monthly model input: report month, model, sales qty bulan tersebut, defect qty bulan tersebut, non-defect qty bulan tersebut.
- Monthly defect category input: report month, model, defect category, qty bulan tersebut.
- Calculation/view layer: membangun snapshot Section C, Section A, Section B, dan Section D berdasarkan report month, product, dan manufacturer.

## 13) Referensi output F-COST

File referensi yang dipakai:

- `FCOST - LCD LOCAL.xlsx`
- `FCOST - LCD LOCAL_rev.pdf`

PDF berasal dari Microsoft Excel 2016, 1 halaman A4 portrait. Workbook Excel berisi 1 sheet bernama `F-Cost`, print area `A1:R39`, dan 1 chart. Untuk F-COST monthly report, file Excel adalah template/source utama, sedangkan PDF adalah hasil print/export dari sheet yang sama.

Report F-COST adalah report monitoring Failure Cost untuk product `LCD` dan vendor/manufacturer `LOCAL`. Seperti FQMS, product + vendor/manufacturer menjadi konteks/filter report, input bulanan, preview, dan export.

## 14) F-COST reporting month dan fiscal half

Contoh file memakai report month `Mar-26`, dengan current fiscal half `2025LH` yang berjalan dari Oktober 2025 sampai Maret 2026. Template menampilkan tiga tipe periode sekaligus:

- Kolom agregat previous fiscal half, contoh `2025FH`.
- Kolom bulanan current fiscal half, contoh `Oct-25` sampai `Mar-26`.
- Kolom total current fiscal half, contoh `2025LH`.

Kartu ringkasan di bagian atas memakai current fiscal half dan report month aktif:

```text
total_sales_current_half = sum(sales_amount bulanan current half)
total_fcost_current_half = sum(fcost_amount bulanan current half)
fcost_report_month = fcost_amount pada report_month
cost_vs_target_report_month = fcost_amount report_month / target_fcost report_month
```

Contoh angka di file `Mar-26`:

```text
total_sales_current_half = 1.022.755.085
total_fcost_current_half = 7.571.386
fcost_report_month = 984.681
target_fcost_report_month = 1.494.000
cost_vs_target_report_month = 984.681 / 1.494.000 = 65,9% ≈ 66%
```

Istilah template lama `Achievement` disepakati diganti menjadi `Cost vs Target` karena metrik ini membandingkan realisasi biaya terhadap target biaya. Untuk F-COST, angka lebih rendah dari 100% berarti biaya aktual berada di bawah target, sehingga istilah `Achievement` bisa ambigu.

Nilai amount pada template memakai satuan `Amount < Rp. K>`, sehingga angka disimpan/ditampilkan sebagai ribuan rupiah sesuai workbook.

## 15) Section A F-COST — Monthly F-Cost

Section A adalah tabel utama F-COST. Data yang tampil adalah kombinasi agregat previous fiscal half, detail bulan current fiscal half, dan total current fiscal half.

Field utama per periode:

- Sales Amount
- F-Cost Amount
- Target F-Cost
- Ratio vs Sales
- Cost vs Target
- Sales Qty
- F-Cost Qty
- Ratio %
- LY F-Cost
- Ratio vs LY F-Cost

Definisi dan formula:

```text
ratio_vs_sales = fcost_amount / sales_amount
cost_vs_target = fcost_amount / target_fcost
fcost_qty_ratio = fcost_qty / sales_qty
ratio_vs_ly_fcost = fcost_amount / ly_fcost
ly_fcost = fcost_amount dari bulan yang sama satu tahun sebelumnya
```

Total row current fiscal half:

```text
total_sales_amount = sum(sales_amount bulanan current half)
total_fcost_amount = sum(fcost_amount bulanan current half)
total_target_fcost = sum(target_fcost bulanan current half)
total_sales_qty = sum(sales_qty bulanan current half)
total_fcost_qty = sum(fcost_qty bulanan current half)
total_ly_fcost = sum(ly_fcost dari bulan yang sama satu tahun sebelumnya untuk setiap bulan current half)
total_ratio_vs_sales = total_fcost_amount / total_sales_amount
total_cost_vs_target = total_fcost_amount / total_target_fcost
total_fcost_qty_ratio = total_fcost_qty / total_sales_qty
total_ratio_vs_ly_fcost = total_fcost_amount / total_ly_fcost
```

Contoh total `2025LH`:

```text
total_sales_amount = 1.022.755.085
total_fcost_amount = 7.571.386
total_target_fcost = 8.964.000
total_ratio_vs_sales = 7.571.386 / 1.022.755.085 = 0,740% ≈ 0,74%
total_cost_vs_target = 7.571.386 / 8.964.000 = 84,46% ≈ 84,5%
total_sales_qty = 378.156
total_fcost_qty = 15.214
total_fcost_qty_ratio = 15.214 / 378.156 = 4,02% ≈ 4,0%
```

Previous fiscal half seperti `2025FH` dihitung dari data bulanan fiscal half sebelumnya jika data historis tersedia. Untuk kebutuhan awal atau ketika data bulanan historis belum lengkap, nilai agregat pembanding dapat disimpan sebagai fiscal half snapshot.

LY F-Cost bukan input manual terpisah. Sistem mengambil nilai dari F-COST bulan yang sama satu tahun sebelumnya. Contoh untuk `Mar-26`, LY F-Cost mengambil data `Mar-25`. Untuk total current half, LY F-Cost menjumlahkan pasangan bulan yang sama tahun sebelumnya untuk setiap bulan current half. Jika data LY tidak tersedia, aplikasi menampilkan `CHECK` atau nilai kosong dengan status validasi `LY data missing`, bukan menyediakan input manual LY. Konsekuensinya, saat inisialisasi data pertama, data historis satu tahun sebelumnya perlu disiapkan agar report dan validasi F-COST bisa lengkap.

Target F-COST adalah satu nilai global per fiscal half untuk product + vendor/manufacturer report. Nilai target di-update pada bulan pertama fiscal half, mengikuti pola target monthly PPM FQMS. Saat report dibuat, aplikasi memakai target fiscal half yang berlaku untuk setiap bulan dalam periode tersebut.

## 16) Section B F-COST — F-Cost Trend

Section B adalah grafik trend untuk current fiscal half. Pada contoh `Mar-26`, grafik mengambil 6 bulan `Oct-25` sampai `Mar-26`.

Mapping:

```text
month
fcost_amount = fcost_amount bulanan pada Section A
target_fcost = target_fcost bulanan pada Section A
```

Grafik tidak memakai kolom agregat previous fiscal half dan tidak memakai total current fiscal half. Series yang tampil adalah `F-Cost` dan `Target`.

## 17) Section C F-COST — Detail F-Cost & Part Contribution

Section C pada F-COST berbeda dari Section C FQMS. Di F-COST, section ini adalah breakdown kontribusi biaya selama current fiscal half.

Breakdown dicatat per bulan agar jelas dan terdokumentasi. Report current fiscal half menjumlahkan breakdown bulanan dari awal fiscal half sampai report month.

Bagian pertama memecah total F-COST current half berdasarkan item biaya utama:

```text
items = Part, Labor, Trip
item_ratio = item_amount / total_fcost_current_half
total_fcost_current_half = sum(item_amount)
```

Contoh `2025LH`:

```text
Part = 6.216.357 → 82,1%
Labor = 924.892 → 12,2%
Trip = 430.137 → 5,7%
Total = 7.571.386 → 100%
```

Bagian kedua memecah kontribusi `Part` berdasarkan kategori part:

```text
part_category_amount
part_category_qty
part_category_ratio = part_category_amount / total_part_amount
```

Contoh kategori part yang muncul di template:

- Panel
- Main Unit
- Power Unit
- Other

Keputusan final: rasio kategori part dihitung dari `part_category_amount / total_part_amount`. Aplikasi tidak menyimpan rasio kategori part sebagai input manual; rasio selalu dihitung dari amount agar konsisten dan total rasio aktual kembali ke 100%.

## 18) Konsekuensi struktur data F-COST untuk implementasi

Struktur data F-COST final memakai long-format bulanan agar konsisten dengan template dan rule report. Implementasi perlu mempertimbangkan data berikut pada phase backend nanti:

- Monthly F-COST summary: report month, product, vendor/manufacturer, sales amount, sales qty, fcost amount, fcost qty. LY F-Cost dihitung dari data bulan yang sama satu tahun sebelumnya.
- Fiscal F-COST target: fiscal half, product, vendor/manufacturer, target fcost, effective start/end. Target berlaku sama untuk semua bulan dalam satu fiscal half dan di-update pada bulan pertama fiscal half.
- Fiscal half snapshot: fiscal half, product, vendor/manufacturer, sales amount, fcost amount, target fcost, sales qty, fcost qty untuk kolom agregat pembanding seperti `2025FH` jika data bulanan historis belum tersedia.
- Monthly F-COST item breakdown: report month, product, vendor/manufacturer, item type (`Part`, `Labor`, `Trip`), amount.
- Monthly F-COST part category breakdown: report month, product, vendor/manufacturer, part category (`Panel`, `Main Unit`, `Power Unit`, `Other`), amount, qty.
- Calculation/view layer: membangun snapshot Section A, Section B, Section C, dan kartu summary berdasarkan report month, fiscal half, product, dan vendor/manufacturer.

Validasi F-COST minimum yang perlu dipertimbangkan:

```text
sum(monthly fcost amount current half) = total_fcost_current_half
sum(monthly target fcost current half) = total_target_fcost_current_half
sum(monthly item breakdown amount current half) = total_fcost_current_half
sum(monthly part category amount current half) = item breakdown amount untuk Part
sales_amount > 0 jika ratio_vs_sales perlu dihitung
sales_qty > 0 jika fcost_qty_ratio perlu dihitung
data F-COST bulan yang sama satu tahun sebelumnya wajib tersedia untuk menghitung ratio_vs_ly_fcost
jika data LY tidak tersedia, tampilkan CHECK atau kosong dengan status validasi `LY data missing`
ly_fcost > 0 jika ratio_vs_ly_fcost perlu dihitung
```

Jika denominator bernilai 0, tampilan sebaiknya mengikuti prinsip PRD untuk PPM: tampilkan `CHECK` atau status validasi, bukan angka rasio yang menyesatkan.

Phase 0 final: discovery dan mapping FQMS sudah dikunci. Discovery F-COST sudah ditambahkan dari contoh export PDF dan template Excel. Belum ada implementasi kode aplikasi dari dokumen ini.
