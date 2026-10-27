# Plan: CSV sebagai Sumber Data Dashboard Aset

## Ringkasan

Tukar `dashboard.html` daripada data hardcoded JavaScript kepada membaca fail `data.csv`
menggunakan `fetch()`. Semua chart, KPI card, progress bar dan jadual akan dikira semula
secara dinamik daripada CSV setiap kali halaman dimuatkan.

Fail `data.csv` adalah sumber data tunggal — admin hanya perlu edit CSV di Excel, Google
Sheets atau terus di GitHub.com, kemudian commit. GitHub Pages akan auto-deploy dalam masa
beberapa minit.

**Skop:**
- Cipta `data.csv` dengan data sedia ada
- Ubah suai `dashboard.html` untuk fetch dan parse CSV
- Semua KPI, chart dan jadual dikira dari data CSV (tiada hardcode)
- Tambah paparan mesej ralat jika CSV gagal dimuatkan

**Bukan skop:**
- Tiada antara muka CRUD dalam browser
- Tiada backend atau database
- Tiada perubahan pada `index.html`

---

## Struktur Fail

```
saifuledu/aset/
├── index.html          (tiada perubahan)
├── dashboard.html      (diubah suai)
└── data.csv            (fail baru — sumber data tunggal)
```

---

## Sub-Tugas

---

### Sub-Tugas 1 — Cipta data.csv

**Intent**
Pindahkan semua 16 baris data kategori daripada array hardcoded `tableData` dalam
`dashboard.html` ke dalam fail CSV yang bebas, boleh diedit dengan Excel atau Sheets.

**Expected Outcomes**
- Fail `data.csv` wujud di root folder repo
- Mengandungi header row + 16 baris data kategori
- Semua nilai sepadan dengan data asal dalam `dashboard.html`
- Boleh dibuka dan diedit dengan Excel, Google Sheets atau teks editor

**Format CSV**
```
kategori,jumlah,diperiksa,tidak_diperiksa,A,B,C,D,E,F
Perabot,3761,3757,4,3676,6,7,5,0,0
Peralatan & Kelengkapan Bengkel/Kejuruteraan,247,247,0,202,3,3,3,36,6
Peralatan & Kelengkapan Pejabat,297,297,0,286,2,9,0,0,0
Peralatan & Kelengkapan ICT,155,155,0,147,1,1,0,14,7
Peralatan & Kelengkapan Makmal,40,40,0,37,0,3,0,0,0
Kenderaan,11,11,0,11,0,0,0,0,0
Hiasan / Langsir / Hamparan,120,120,0,120,0,0,0,0,0
Peralatan & Kelengkapan Keselamatan,322,322,0,320,0,2,0,0,0
Peralatan & Kelengkapan Dapur,42,42,0,32,7,3,0,0,0
Peralatan & Kelengkapan Makmal (Bekalan),19,19,0,9,0,10,0,0,0
Peralatan & Kelengkapan Sukan/Rekreasi,13,13,0,13,0,0,0,0,0
Peralatan & Kelengkapan Perubatan,12,12,0,5,0,7,0,0,0
Peralatan & Kelengkapan Penyiaran & Muzik,11,11,0,11,0,0,0,0,0
Peralatan & Kelengkapan Telekomunikasi,8,8,0,8,0,0,0,0,0
Peralatan & Kelengkapan Pertanian/Marin,3,3,0,3,0,0,0,0,0
Loji / Jentera,3,3,0,3,0,0,0,0,0
```

**Todo List**
- [ ] Cipta fail `data.csv` dengan header row dan 16 baris data

**Relevant Context**
- Data asal: `dashboard.html` baris 720–737, array `tableData`
- Format semasa: `[kategori, jumlah, diperiksa, tidakDiperiksa, A, B, C, D, E, F]`

**Status:** `[ ] pending`

---

### Sub-Tugas 2 — Ubah dashboard.html: Fetch & Parse CSV

**Intent**
Gantikan array `tableData` yang hardcoded dengan fungsi `fetch('data.csv')` yang membaca
dan parse CSV semasa halaman dimuatkan. Semua logik render hanya dipanggil selepas data
CSV berjaya dimuatkan.

**Expected Outcomes**
- `dashboard.html` tidak lagi mengandungi sebarang hardcoded data aset
- Fungsi `parseCSV(text)` menukar teks CSV kepada array objek
- Fungsi `initDashboard(data)` dipanggil dengan data yang telah diparse
- Loading state dipaparkan semasa fetch berlaku
- Mesej ralat dipaparkan jika fetch gagal (contoh: fail CSV tidak dijumpai)

**Todo List**
- [ ] Padam array `tableData` hardcoded (baris 720–737)
- [ ] Padam semua hardcoded chart data: `catLabels`, `catValues`, `hbarLabels`, `hbarValues` dan nilai donut/stacked
- [ ] Padam hardcoded KPI values dalam HTML (6,591 / 6,573 / 18 / 19 / 25)
- [ ] Tambah loading spinner ringkas dalam `<body>` sebelum dashboard dipaparkan
- [ ] Tulis fungsi `parseCSV(text)` — split baris, split koma, map ke objek bernama
- [ ] Tulis fungsi `initDashboard(rows)` yang memanggil semua fungsi render
- [ ] Panggil `fetch('data.csv')` pada `window.addEventListener('load', ...)` selepas auth check
- [ ] Handle fetch error — papar mesej ralat yang jelas kepada admin

**Relevant Context**
- Session guard berada di `<head>` (baris 9–13 dashboard.html) — jangan ganggu
- `fetch()` berfungsi di GitHub Pages (HTTPS) tanpa sebarang konfigurasi tambahan
- CSV perlu diparse secara manual (tiada library) — split('\n') kemudian split(',')
- Perlu handle koma dalam nama kategori — guna split(',', 10) atau quote CSV field

**Catatan Teknikal**
Nama kategori seperti `Peralatan & Kelengkapan Bengkel/Kejuruteraan` tidak mengandungi
koma, jadi split biasa selamat digunakan. Jika nama kategori mengandungi koma pada masa
hadapan, perlu wrap dalam double quotes dalam CSV.

**Status:** `[ ] pending`

---

### Sub-Tugas 3 — Kira semula KPI Cards & Progress Bar dari CSV

**Intent**
KPI cards (Jumlah Aset, Diperiksa, Tidak Diperiksa, Rosak, Pinjaman) dan progress bar
kini dikira secara dinamik dengan menjumlahkan lajur berkaitan dari semua baris CSV.

**Expected Outcomes**
- Fungsi `renderKPI(rows)` mengira aggregate dari data CSV
- 5 KPI values dikemas kini dalam DOM
- Peratusan Diperiksa dikira: (jumlah_diperiksa / jumlah_keseluruhan * 100)
- Progress bar width dikemas kini secara dinamik
- Footer jadual `JUMLAH KESELURUHAN` dikira dari CSV bukan hardcoded

**Todo List**
- [ ] Tulis fungsi `renderKPI(rows)` — sum lajur jumlah, diperiksa, tidakDiperiksa, C (rosak), E (pinjaman)
- [ ] Kemas kini DOM: 5 elemen `.kpi-value` dengan nilai yang dikira
- [ ] Kemas kini peratusan dan `.kpi-pill` untuk Diperiksa dan Tidak Diperiksa
- [ ] Kemas kini `.progress-bar-inner` width dan teks peratusan secara dinamik
- [ ] Kemas kini teks `6,573 / 6,591 aset telah diperiksa` secara dinamik

**Relevant Context**
- 5 KPI cards ada dalam HTML dengan class `.kpi-value` — perlu tambah `id` unik pada setiap satu
- Progress bar inner: `.progress-bar-inner` dengan inline style `width: 99.73%`
- Footer tfoot dalam jadual hardcoded di baris 690–701

**Status:** `[ ] pending`

---

### Sub-Tugas 4 — Render Jadual dari CSV

**Intent**
Fungsi `renderTable(rows)` yang sedia ada ditukar untuk menerima parameter `rows` dari CSV
berbanding membaca `tableData` global. Semua 16 baris dirender, termasuk baris jumlah
keseluruhan di tfoot yang dikira secara dinamik.

**Expected Outcomes**
- Jadual dirender dengan data terkini dari CSV
- Warna sel (merah/oren/hijau) masih berfungsi berdasarkan nilai
- Baris tfoot `JUMLAH KESELURUHAN` dikira secara automatik dari semua baris
- Jika admin tambah kategori baru dalam CSV, ia akan muncul dalam jadual tanpa ubah kod

**Todo List**
- [ ] Tukar fungsi render jadual supaya menerima `rows` sebagai parameter
- [ ] Kemas kini tfoot secara dinamik — jumlahkan semua lajur numerik
- [ ] Pastikan format nombor dengan `.toLocaleString()` kekal

**Relevant Context**
- Fungsi render jadual sedia ada bermula di baris 739 dashboard.html
- Menggunakan `document.getElementById('mainTableBody')` dan `appendChild`
- tfoot hardcoded di baris 690–701

**Status:** `[ ] pending`

---

### Sub-Tugas 5 — Kemas kini semua Chart dari CSV

**Intent**
Keempat-empat chart (bar kategori, donut status, horizontal bar top 10, stacked bar)
dikemas kini untuk menggunakan data yang dikira dari CSV, bukan array hardcoded.

**Expected Outcomes**
- Bar chart kategori: labels dan values dikira dari CSV rows
- Donut chart: jumlah A–F dikira dari sum semua baris CSV
- Horizontal bar top 10: ambil 10 kategori dengan jumlah tertinggi, sort descending
- Stacked bar: jumlah A–F dikira dari CSV
- Semua chart akan auto-refresh apabila data CSV dikemas kini dan halaman di-reload

**Todo List**
- [ ] Tulis fungsi `renderBarChart(rows)` — map rows ke labels dan values, sort by jumlah descending
- [ ] Tulis fungsi `renderDonutChart(rows)` — sum lajur A, B, C, D, E dari semua rows
- [ ] Tulis fungsi `renderHBarChart(rows)` — sort by jumlah desc, ambil top 10, bina labels dengan nama + unit
- [ ] Tulis fungsi `renderStackedBar(rows)` — sum A–F untuk stacked dataset
- [ ] Semua fungsi chart dipanggil dari dalam `initDashboard(rows)`
- [ ] Padam semua deklarasi chart data hardcoded lama

**Relevant Context**
- 4 canvas elements: `#barChart`, `#donutChart`, `#hbarChart`, `#statusBar`
- Chart.js 4.4.0 sudah dimuatkan via CDN
- Top 10 horizontal bar perlu sort — gunakan `[...rows].sort((a,b) => b.jumlah - a.jumlah).slice(0,10)`
- Label horizontal bar perlu singkat — gunakan nama kategori sahaja (tanpa unit dalam nama)

**Status:** `[ ] pending`

---

## Susunan Implementasi

```
Sub-Tugas 1 → Sub-Tugas 2 → Sub-Tugas 3 → Sub-Tugas 4 → Sub-Tugas 5
   (CSV)        (fetch)        (KPI)         (Table)       (Charts)
```

Setiap sub-tugas bergantung kepada yang sebelumnya — laksanakan mengikut urutan.

---

## Panduan Edit data.csv untuk Admin

Selepas implementasi selesai, admin (Rohimah) boleh:

1. **Edit di GitHub.com** — pergi ke `saifuledu/aset/data.csv`, klik ikon pensil, edit terus
2. **Edit di Excel** — download CSV, buka di Excel, edit, save as CSV, upload semula ke GitHub
3. **Edit di Google Sheets** — import CSV, edit, export semula sebagai CSV, commit ke GitHub

**Peraturan penting:**
- Jangan padam baris header (baris pertama)
- Pastikan setiap baris ada tepat 10 nilai dipisahkan koma
- Nilai mestilah nombor bulat (integer), bukan teks
- Jumlah A+B+C+D+E+F mestilah sama dengan nilai `diperiksa`
- Nilai `tidak_diperiksa` = `jumlah` - `diperiksa`
