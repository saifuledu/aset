# Plan: Editor Data Online via GitHub API

## Ringkasan

Tambah sistem pengurusan data (CRUD) dalam `dashboard.html` yang membolehkan admin
menambah, mengedit dan memadam rekod kategori aset secara terus dalam browser.
Data disimpan ke `data.csv` dalam repo GitHub melalui GitHub Contents API.

GitHub Personal Access Token disimpan dalam `localStorage` browser — tidak dedah
dalam kod dan tidak di-commit ke repo.

**Aliran kerja:**
1. Admin buka tetapan → masuk GitHub token + nama repo (sekali sahaja)
2. Admin klik "Urus Data" → modal CRUD dipaparkan
3. Admin tambah/edit/padam rekod → klik Simpan
4. Dashboard hantar PUT request ke GitHub API → `data.csv` dalam repo dikemas kini
5. Dashboard reload data dari CSV → carta & jadual dikemas kini automatik

**Skop:**
- Butang "Urus Data" di header
- Modal senarai rekod + butang Edit/Padam setiap baris
- Borang tambah rekod baru
- Borang edit rekod sedia ada (pre-filled)
- Konfirmasi padam
- Modal Tetapan GitHub (token + repo + branch)
- Simpan ke GitHub API via fetch PUT
- Reload dashboard selepas save berjaya
- Mesej status (saving / berjaya / gagal)

**Bukan skop:**
- Tiada perubahan pada `index.html`
- Tiada perubahan pada `data.csv`
- Tiada backend atau server
- Tidak menyimpan token dalam kod

---

## Struktur Fail

```
saifuledu/aset/
├── index.html        (tiada perubahan)
├── dashboard.html    (ditambah: CRUD modal, Settings modal, GitHub API logic)
└── data.csv          (dikemas kini via GitHub API apabila admin simpan)
```

---

## Maklumat GitHub API

**Endpoint untuk kemas kini fail:**
```
PUT https://api.github.com/repos/{owner}/{repo}/contents/{path}
```

**Headers diperlukan:**
```
Authorization: Bearer {token}
Content-Type: application/json
```

**Body:**
```json
{
  "message": "Kemas kini data.csv",
  "content": "<base64 encoded CSV content>",
  "sha": "<SHA semasa fail — perlu ambil dahulu via GET>"
}
```

**Langkah teknikal:**
1. GET `/repos/{owner}/{repo}/contents/data.csv` → ambil `sha` semasa
2. Encode CSV baru ke base64
3. PUT dengan `sha` + content baru

---

## Sub-Tugas

---

### Sub-Tugas 1 — Modal Tetapan GitHub

**Intent**
Borang untuk admin masukkan GitHub token, nama owner, nama repo dan branch.
Semua nilai disimpan ke `localStorage`. Perlu diisi sekali sahaja.

**Expected Outcomes**
- Butang ⚙️ Tetapan di header dashboard
- Modal dengan 4 field: Token, Owner, Repo, Branch (default: main)
- Butang "Uji Sambungan" — test GET ke GitHub API, papar status OK/gagal
- Nilai disimpan ke localStorage: `kk_gh_token`, `kk_gh_owner`, `kk_gh_repo`, `kk_gh_branch`
- Notis amaran: "Token ini disimpan dalam browser anda sahaja"
- Jika token belum diset, badge merah pada butang Urus Data

**Todo List**
- [ ] Tambah butang "⚙️ Tetapan" di header (sebelum butang Log Keluar)
- [ ] Bina modal Tetapan dengan 4 input field
- [ ] Fungsi `saveSettings()` — simpan ke localStorage
- [ ] Fungsi `loadSettings()` — isi semula field dari localStorage on modal open
- [ ] Fungsi `testConnection()` — GET repo info, papar nama repo jika berjaya
- [ ] CSS untuk modal tetapan (guna palet warna sedia ada)

**Relevant Context**
- Header di `dashboard.html` — `.header-inner` (lepas `.header-badge`, sebelum `.btn-logout`)
- localStorage keys guna prefix `kk_` untuk konsisten dengan `kk_auth`
- Modal pattern sama seperti reset modal dalam `index.html`

**Status:** `[ ] pending`

---

### Sub-Tugas 2 — Modal Urus Data (Senarai Rekod)

**Intent**
Modal utama yang memaparkan semua rekod kategori dalam jadual ringkas.
Setiap baris ada butang Edit dan Padam. Ada butang "Tambah Rekod Baru" di bahagian atas.

**Expected Outcomes**
- Butang "📋 Urus Data" di header
- Modal penuh (lebar) dengan jadual semua rekod dari CSV terkini
- Setiap baris: nombor, nama kategori, jumlah, butang ✏️ Edit, butang 🗑️ Padam
- Butang "+ Tambah Rekod Baru" di atas jadual
- Jika token belum diset, papar amaran "Sila tetapkan GitHub token dalam ⚙️ Tetapan dahulu"
- Loading state semasa fetch data CSV semasa
- Status bar di bawah modal (mesej saving/berjaya/gagal)

**Todo List**
- [ ] Tambah butang "📋 Urus Data" di header
- [ ] Bina HTML modal Urus Data dengan jadual dinamik
- [ ] Fungsi `openDataManager()` — semak token, fetch CSV semasa, render jadual
- [ ] Fungsi `renderManagerTable(rows)` — bina jadual dengan butang Edit/Padam
- [ ] Fungsi `confirmDelete(index)` — dialog konfirmasi, padam baris, panggil save
- [ ] Status bar dengan animasi (idle / saving / success / error)

**Relevant Context**
- `rows` data diparse dari `parseCSV()` sedia ada dalam dashboard.html
- Modal perlu lebar (max-width: 900px) untuk muat jadual
- Guna z-index lebih tinggi dari modal lain

**Status:** `[ ] pending`

---

### Sub-Tugas 3 — Borang Tambah & Edit Rekod

**Intent**
Borang modal untuk tambah rekod baru atau edit rekod sedia ada.
Field: Kategori, Jumlah, Diperiksa, Status A–F (9 field nombor + 1 text).
Validasi: jumlah mestilah >= diperiksa, nilai negatif tidak dibenarkan.

**Expected Outcomes**
- Modal borang dengan semua 10 field (kategori + 9 nombor)
- Mod Tambah: semua field kosong, tajuk "Tambah Rekod Baru"
- Mod Edit: field pre-filled dengan data baris semasa, tajuk "Edit Rekod"
- Pengiraan automatik: `tidak_diperiksa = jumlah - diperiksa` (dikira auto, readonly)
- Validasi sebelum simpan:
  - Nama kategori tidak boleh kosong
  - Nama kategori tidak boleh duplikasi (kecuali mod edit rekod yang sama)
  - `diperiksa` tidak boleh melebihi `jumlah`
  - `A+B+C+D+E+F` mesti sama dengan `diperiksa`
  - Semua nilai nombor mestilah >= 0
- Mesej ralat inline untuk setiap validasi gagal

**Todo List**
- [ ] Bina HTML borang modal dengan 10 input field
- [ ] Fungsi `openAddForm()` — clear form, set mod tambah, buka modal
- [ ] Fungsi `openEditForm(index)` — isi form dengan data row[index], set mod edit
- [ ] Auto-kira `tidak_diperiksa` pada setiap input change
- [ ] Auto-kira jumlah A+B+C+D+E+F dan bandingkan dengan `diperiksa` (live indicator)
- [ ] Fungsi `validateForm()` — return true/false dengan mesej ralat
- [ ] Fungsi `submitForm()` — tambah/update rows array, panggil `saveToGitHub(rows)`

**Relevant Context**
- `rows` array perlu dikekalkan dalam memori semasa modal terbuka
- Selepas save berjaya, `initDashboard(rows)` dipanggil semula untuk refresh semua chart
- Validasi A+B+C+D+E+F = diperiksa adalah peraturan data yang kritikal

**Status:** `[ ] pending`

---

### Sub-Tugas 4 — GitHub API: Simpan ke data.csv

**Intent**
Fungsi `saveToGitHub(rows)` yang menukar array `rows` kepada format CSV,
kemudian hantar ke GitHub Contents API untuk kemas kini `data.csv` dalam repo.

**Expected Outcomes**
- Fungsi `rowsToCSV(rows)` — tukar array objek ke string CSV dengan header
- Fungsi `getFileSHA()` — GET semasa SHA `data.csv` dari GitHub API
- Fungsi `saveToGitHub(rows)` — orchestrate: get SHA → encode CSV → PUT ke API
- Status bar dikemas kini: "⏳ Menyimpan..." → "✅ Berjaya disimpan" / "❌ Gagal"
- Selepas save berjaya: `initDashboard(rows)` dipanggil untuk refresh dashboard
- Handle HTTP 401 (token salah), 404 (repo/fail tidak jumpa), 422 (SHA salah)
- Mesej ralat yang bermakna untuk setiap kes

**Todo List**
- [ ] Fungsi `rowsToCSV(rows)` — header + baris CSV, join dengan `\n`
- [ ] Fungsi `getFileSHA()` — fetch GET API, return sha string
- [ ] Fungsi `saveToGitHub(rows)` — async, update status bar, call GET then PUT
- [ ] Handle semua HTTP error dengan mesej Bahasa Melayu
- [ ] Selepas PUT berjaya: tutup modal, refresh `initDashboard(rows)`, papar toast "✅ Data berjaya disimpan ke GitHub"

**Relevant Context**
- GitHub API endpoint: `https://api.github.com/repos/{owner}/{repo}/contents/{path}`
- Token dari `localStorage.getItem('kk_gh_token')`
- `btoa(unescape(encodeURIComponent(csvString)))` untuk encode UTF-8 ke base64
- SHA semasa mesti betul atau API akan return 409 Conflict

**Status:** `[ ] pending`

---

## Susunan Implementasi

```
ST1 (Tetapan) → ST2 (Modal Senarai) → ST3 (Borang Tambah/Edit) → ST4 (GitHub API Save)
```

Semua sub-tugas bergantung kepada yang sebelumnya. Laksanakan mengikut urutan.

---

## Nota Keselamatan Token

- Token **tidak** di-hardcode dalam mana-mana fail
- Token disimpan dalam `localStorage` browser pengguna
- Token **tidak** di-commit ke GitHub
- Admin perlu masuk semula token jika clear browser data
- Cadangan scope token GitHub: **repo** (full repo access) atau **contents:write**

---

## Panduan Jana GitHub Personal Access Token

1. Log masuk GitHub → **Settings** → **Developer settings**
2. **Personal access tokens** → **Tokens (classic)**
3. **Generate new token** → beri nama (contoh: "Dashboard Aset KK")
4. Pilih scope: ✅ **repo** (atau minimal: **public_repo** jika repo public)
5. **Generate token** → salin dan simpan (hanya papar sekali)
6. Masukkan token dalam dashboard → ⚙️ Tetapan
