# Dokumentasi Teknis — MU Maintenance & Reliability Dashboard

Dokumen ini menjelaskan arsitektur, alur data, dan tiap modul kode di `index.html`. Ditujukan untuk pengembang atau siapa pun yang ingin memahami, memelihara, atau melanjutkan pengembangan aplikasi ini.

---

## 1. Filosofi desain

Aplikasi ini sengaja dibuat sebagai **satu berkas `index.html`** tanpa framework, tanpa proses *build*, dan tanpa backend. Alasannya:

- **Portabel** — cukup satu file untuk berjalan; bisa dibuka lokal (double‑click) atau di‑deploy sebagai file statis.
- **Aman** — semua data Excel diproses di browser; tidak ada data yang keluar dari komputer pengguna.
- **Mudah dirawat** — tidak ada dependensi yang perlu di‑*install*; hanya beberapa pustaka via CDN.

Konsekuensinya, seluruh HTML, CSS, dan JavaScript berada dalam satu berkas. Struktur besarnya:

```
<head>   → meta, font, ikon, pustaka CDN
<style>  → seluruh CSS (variabel tema, komponen, media query)
<body>   → struktur UI (header, tab, kartu, tabel, modal)
<script> → seluruh logika aplikasi (86 fungsi)
```

## 2. Pustaka eksternal (CDN)

| Pustaka | Fungsi |
|---|---|
| SheetJS `xlsx` 0.18.5 | Membaca file `.xlsx` menjadi array data |
| Tabler Icons | Ikon antarmuka |
| Inter (Google Fonts) | Tipografi |
| html2pdf.js | Ekspor tampilan ke PDF |

## 3. Arsitektur & alur data

Alur besar dari file Excel hingga tampil di layar:

```
File Excel di-upload
   │
   ▼
FileReader (baca sebagai ArrayBuffer)
   │
   ▼
XLSX.read()  → workbook (SheetJS)
   │
   ▼
classifyAndParse()  → tentukan jenis file & sheet terbaik
   │
   ├── parseDowntime()  → S.dt   (data downtime/breakdown)
   ├── parseUsage()     → S.us   (pemakaian sparepart)
   ├── parseStock()     → S.st   (stok sparepart)
   └── parsePM()        → S_pm   (jadwal PM)
   │
   ▼
Render:
   ├── afterDT() → buildFilterDefs() + renderFilters() + renderDT() + renderCal()
   ├── renderSP()  (sparepart)
   └── renderCal() (kalender PM)
```

**Objek state global** menyimpan hasil parsing:

| Variabel | Isi |
|---|---|
| `S.dt` | Data downtime terparse: `{raw, cols, hasBkd, rows, year}` |
| `S.us` | Data pemakaian sparepart |
| `S.st` | Data stok sparepart |
| `S_pm` | Jadwal PM: `{raw, year}` |
| `F`, `NUMF` | Kondisi filter aktif (teks & rentang angka) |

Setiap baris downtime (`S.dt.raw[i]`) menyimpan **dua representasi**:

- Kolom "penting" bernama pendek (`mo`, `line`, `packer`, `eqe`, `det`, `cat`, `loss`, `dur`, `isb`, `_d`, …) yang dipakai perhitungan inti.
- Peta `_all` (nama header → nilai teks) dan `_num` (nama header → nilai angka) yang dipakai **filter dinamis** agar semua kolom Excel bisa jadi filter.

## 4. Modul kode

Berikut pengelompokan 86 fungsi berdasarkan perannya.

### 4.1 Setup & tema

| Fungsi | Tujuan |
|---|---|
| `effectiveDark()` | Menentukan apakah mode gelap aktif (mengikuti pilihan pengguna / sistem) |
| `apply(t)` | Menerapkan tema terang/gelap ke `<html data-theme>` |
| `paintIcon()` | Mengganti ikon tombol tema (bulan/matahari) |
| `el(id)` | Pintasan `document.getElementById` |
| `css(v)` | Membaca nilai CSS variable |

### 4.2 Utilitas

| Fungsi | Tujuan |
|---|---|
| `norm(x)` | Normalisasi teks: buang spasi berlebih, ubah ke string |
| `num(x)` | Konversi ke angka; mengembalikan `null` jika bukan angka |
| `stripCode(x)` | Membuang prefiks kode mesin (mis. `K01.Packer 3` → `Packer 3`) |
| `esc(s)` | *Escape* HTML agar aman dimasukkan ke DOM |
| `r1(x)` | Pembulatan 1 desimal |

### 4.3 Pembacaan Excel (inti)

| Fungsi | Tujuan |
|---|---|
| `findHeader(rows, key)` | Cari indeks baris yang memuat header tertentu (mis. `Loss Category`) di 15 baris pertama |
| `toObjects(rows, hr)` | Ubah array mentah menjadi array objek (kunci = nama header) |
| `findCol(head, pred)` | Cari nama kolom yang cocok dengan sebuah kondisi (regex) |
| `findHeaderInWb(wb, key)` | Cek apakah sebuah header ada di salah satu sheet workbook |
| `classifyAndParse(wb, fname)` | **Otak intake**: menentukan jenis file, memilih sheet downtime terbaik, dan memanggil parser yang sesuai |
| `handleFiles(files)` | Menerima file dari input/drag‑drop, membaca via `FileReader`, memanggil `classifyAndParse` |

**Pemilihan sheet terbaik** (`classifyAndParse`): sebuah file bisa memiliki beberapa sheet dengan kolom `Loss Category`. Aplikasi memberi skor tiap sheet berdasarkan kelengkapan kolom penting (`duration`, `packer`, `line`, `equipment`, `category bkd`, `month`, `description`) dan mengutamakan sheet bernama **DATA2025**, lalu memilih yang skornya tertinggi / barisnya terbanyak. Ini mencegah aplikasi salah memakai sheet ringkasan yang kolomnya minim.

### 4.4 Parser data

| Fungsi | Menghasilkan | Catatan |
|---|---|---|
| `parseDowntime(obj)` | `S.dt` | Membaca semua kolom berjudul; membuang baris rekap tanpa `Loss Category`; menandai baris breakdown (`isb`), menyimpan tanggal (`_d`) dan tahun; mengklasifikasi tiap kolom sebagai `text`/`num`/`skip` untuk filter |
| `parseUsage(obj)` | `S.us` | Pemakaian sparepart (kolom `Item`) |
| `parseStock(obj)` | `S.st` | Stok sparepart (kolom `Item No.`) |
| `parsePM(wb)` | `S_pm` | Jadwal PM dari sheet PM Monitoring; membaca kolom minggu `W01`–`W52` |

Klasifikasi kolom pada `parseDowntime`:
- **`num`** — mayoritas nilai angka & unik > 12 → menjadi filter *rentang* (min–maks).
- **`text`** — jumlah nilai unik ≤ 400 dan bukan teks bebas → menjadi filter *centang*.
- **`skip`** — teks bebas (unik hampir sebanyak jumlah baris, mis. *Description*) → tidak dijadikan filter.

### 4.5 Mesin filter

Inilah bagian yang membuat **semua kolom Excel bisa menjadi filter** dan saling menyesuaikan.

| Fungsi | Tujuan |
|---|---|
| `labelOf(h)` | Meringkas label header panjang (mis. `Equipment (Untuk kategori Engineering Downtime)` → `Equipment (Eng.)`) |
| `isExcludedFilter(h)` | Menandai kolom yang **sengaja tidak dijadikan filter** (Total Spout berhenti, Start, End, Duration, #spout/line, Multiple factor, OEE Impact, Cause, Discription, Root Cause, Action Plan, CODE) |
| `buildFilterDefs()` | Membangun daftar filter teks (`FDEF`) & rentang angka (`FNUM`) dari `S.dt.cols` |
| `buildDims()` | Membangun daftar dimensi untuk grafik "Analisis downtime" |
| `resetFilters()` | Mengosongkan semua pilihan filter |
| `gv(r, k)` | Ambil nilai kolom `k` dari baris `r` (dari peta `_all`), kosong → `(Kosong)` |
| `universe(k)` | Daftar nilai unik sebuah kolom (untuk isi dropdown), sudah diurutkan |
| `passRow(r, skipKey)` | Cek apakah sebuah baris lolos semua filter aktif |
| `filteredRows()` | Kumpulan baris yang lolos filter — sumber semua angka di layar |
| `optionCounts(k)` | Jumlah baris per nilai (angka di sebelah tiap opsi), menyesuaikan filter lain |
| `toggleVal(k, v, on)` | Centang/hapus satu nilai pada filter |
| `paintList(drop)` | Menggambar isi dropdown (opsi + pencarian + "pilih semua") |
| `placeMenu(drop)` | Mengatur posisi menu agar tidak terpotong layar (buka ke atas/kanan bila sempit) |
| `openDrop(drop)` | Membuka dropdown & fokus ke kotak cari |
| `paintCaps(pill)` | Memperbarui teks ringkas pada tombol filter (mis. `3/9`) & menghitung filter aktif |
| `renderFilters()` | Menggambar seluruh bar filter (dropdown teks + chip rentang angka) |
| `durChanged()` | Menangani kotak filter Duration (min–maks) khusus |

**Prinsip:** `filteredRows()` adalah satu‑satunya sumber kebenaran. Setiap perubahan filter memanggil `renderDT()` yang membaca ulang `filteredRows()` sehingga KPI, grafik, tabel, dan kalender selalu konsisten.

### 4.6 Render tab Breakdown Map

| Fungsi | Tujuan |
|---|---|
| `kpi(id, arr)` / `kpiMeta(label)` | Menggambar kartu KPI + memilih ikon/warna berdasarkan label |
| `bars(id, rows, color)` | Menggambar diagram batang horizontal |
| `bdRender(rows)` | Grafik "Analisis downtime" per dimensi terpilih |
| `renderLine(rows)` | Kartu "Ringkasan per Line" |
| `renderDetail(rows)` | Tabel "Detail kejadian" (maks. 300 baris + pencarian) |
| `renderDT()` | **Render utama tab Breakdown Map**: hitung KPI, tren bulanan, ringkasan line, grafik, tabel, lalu picu `renderCal()` |
| `afterDT()` | Dipanggil setelah data downtime siap: reset & bangun filter, render, set dimensi default |
| `computePM(rows)` | Menurunkan usulan interval PM dari MTBF breakdown |
| `renderPM()` | Kartu "Jadwal perawatan rutin" di tab Preventive Maintenance |

**KPI yang ditampilkan:** Total downtime (jumlah Duration), Kejadian breakdown, dan MTTR rata‑rata (Duration baris breakdown ÷ jumlah breakdown).

### 4.7 Tab Sparepart

| Fungsi | Tujuan |
|---|---|
| `spRender()` / `renderSP()` | Render pemakaian sparepart |
| `renderST()` / `critList()` | Render stok & daftar item kritis |
| `hasCode(x)`, `codeName(x)`, `top(o, n)` | Utilitas pengelompokan/peringkat |

### 4.8 Kalender PM Monitoring

Modul yang memetakan jadwal PM dan kejadian breakdown ke grid **bulan × minggu (W1–W5)**.

| Fungsi | Tujuan |
|---|---|
| `isoMondayY(y, w)` | Senin dari minggu ISO ke‑`w` tahun `y` |
| `mapWeekY(w, y)` | Petakan nomor minggu → {bulan, minggu‑dalam‑bulan, rentang tanggal} |
| `mapDate(dt)` | Petakan sebuah tanggal → {bulan, minggu‑dalam‑bulan, nomor minggu, rentang} |
| `calFmt(d)`, `calYear()` | Format tanggal & penentuan tahun kalender (dikunci ke tahun data dominan) |
| `breakdownEvents()` | Ambil kejadian breakdown **dari hasil filter** (`filteredRows`) untuk kalender |
| `buildCal()` | Susun matriks sel kalender: PM (kuning) + breakdown (merah) |
| `cellItems(c)` | Isi sebuah sel sesuai mode tampilan (Semua/PM/Breakdown) & pencarian |
| `miniMarks(c)` | Susunan penanda kecil (bulat/segitiga) di dalam sel |
| `renderCal()` | Render seluruh tabel kalender + subjudul |
| `pmMatch(e)` | Pencocokan kata kunci pencarian kalender |

**Modal detail (panel samping):**

| Fungsi | Tujuan |
|---|---|
| `openCalCell(wi, mi)` | Buka panel untuk sel (minggu `wi`, bulan `mi`) |
| `cellByKind(wi, mi)` | Kelompokkan isi sel menjadi PM dan BKD, masing‑masing per equipment |
| `sortEq(map)`, `wkMin(arr)`, `wkChip(arr, isB)` | Urutkan equipment per minggu & buat chip W## |
| `colorFor(name)` | Warna turunan nama (dipakai terbatas) |
| `renderCalLevel1()` + `section(...)` | Daftar dua bagian terpisah — **PM** (kuning) & **BKD** (merah), bernomor, terurut minggu |
| `renderCalDetail(kind, eq)` | Detail satu equipment: Component, Action, Task, PIC, Line, Durasi, W##, tanggal |
| `closeCalModal()` | Menutup panel |

**Pemetaan tahun & tanggal janggal:** tahun kalender dikunci ke tahun yang paling sering muncul pada data (mis. 2025). Baris breakdown dengan tanggal di luar tahun itu (mis. salah ketik 2032) tetap ditampilkan namun dipetakan berdasarkan kolom *Month*, bukan tanggalnya, agar tidak merusak grid.

### 4.9 UI helpers & data lifecycle

| Fungsi | Tujuan |
|---|---|
| `toast(msg, type)` | Notifikasi sementara |
| `setPill(state, text)` | Indikator status data (hijau = data asli, merah = contoh) |
| `setChip(id, state, extra)` | Status tiap jenis file di zona upload |
| `dataLoaded()` | Menampilkan konten setelah ada data |
| `loadSample()` + `rnd()` | Membuat data contoh yang deterministik |
| `resetData()` | Menghapus semua data dari memori & mengosongkan tampilan |

### 4.10 Ekspor

| Fungsi | Tujuan |
|---|---|
| `prepExport()` | Menyiapkan tampilan untuk PDF (paksa mode terang, kelas `exporting`) |
| `exportPDF()` | Membuat PDF dengan html2pdf.js (dengan pengaman timeout) |
| `finish(msg, type)` | Mengembalikan tampilan setelah ekspor selesai |

## 5. Konvensi kode

- **Tanpa dependensi build** — semua fungsi berada di satu blok `<script>`.
- **`filteredRows()` = sumber kebenaran** — jangan menghitung KPI dari data mentah; selalu dari hasil filter.
- **Kolom dibaca lewat nama header**, bukan indeks angka — karena baris data berupa objek (`r['Packer']`), bukan array. Ini penting: membaca `r[7]` akan `undefined`.
- **Escape HTML** setiap nilai dari Excel dengan `esc()` sebelum dimasukkan ke DOM.
- **Warna penanda:** PM = kuning `#d99a1f`, Breakdown = merah `#e2574c` — konsisten antara kalender dan panel detail.

## 6. Menambah/mengubah fitur (panduan singkat)

- **Menambah filter untuk kolom baru** — otomatis: selama kolom punya judul di Excel dan tidak termasuk `isExcludedFilter`, ia akan muncul sebagai filter.
- **Mengecualikan kolom dari filter** — tambahkan polanya di `isExcludedFilter()`.
- **Mengubah KPI** — sunting array pada `renderDT()` (bagian `kpi('bdKpi', [...])`).
- **Mengubah tampilan panel kalender** — sunting `renderCalLevel1()` / `renderCalDetail()`; jangan mengubah `miniMarks()`/`renderCal()` bila hanya ingin mengubah panel.
- **Selalu validasi** perubahan JavaScript (mis. `node --check`) dan uji dengan data asli sebelum deploy.

## 7. Uji & validasi

Karena tidak ada *test runner* bawaan, validasi dilakukan dengan:

1. **Cek sintaks** JavaScript (mis. mengekstrak isi `<script>` lalu `node --check`).
2. **Uji fungsional** dengan mem‑*load* file Excel asli dan memastikan angka KPI cocok dengan perhitungan manual di Excel (`SUMIFS`, `COUNTIFS`).
3. **Uji interaksi** (opsional) dengan lingkungan DOM tiruan (mis. jsdom) untuk memastikan filter/kalender bereaksi benar.
