# MU — Maintenance & Reliability Dashboard

Dashboard analisis *downtime*, *breakdown*, pemakaian *sparepart*, dan penjadwalan *Preventive Maintenance* (PM) untuk **PT. Cipta Mortar Utama (Saint‑Gobain)**. Seluruh perhitungan berjalan **di dalam browser** dari file Excel yang di‑upload — tidak ada server, tidak ada database, dan data tidak dikirim ke mana pun.

Dibuat sebagai bagian dari skripsi Teknik Industri bertema *reliability‑based preventive maintenance*.

---

## Ringkasan

- **Satu file** `index.html` — HTML + CSS + JavaScript murni (tanpa framework).
- Membaca **4 jenis file Excel**: data *Downtime*, *Sparepart Usage*, *Stok Sparepart*, dan *PM Monitoring*.
- **Filter otomatis** dari semua kolom berjudul pada sheet Excel — bisa dicentang bebas.
- KPI, grafik tren, analisis per mesin/line, tabel detail kejadian, kalender PM, dan ekspor PDF.
- **Kalender PM Monitoring** menampilkan jadwal PM (kuning) dan kejadian breakdown (merah) per minggu/bulan, dengan panel detail per equipment.
- Tema terang/gelap (AMOLED‑friendly).

## Tumpukan teknologi

| Lapisan | Teknologi |
|---|---|
| Struktur | HTML5 |
| Tampilan | CSS murni (Flexbox, CSS Grid, CSS variables, media queries) — tanpa framework |
| Logika | JavaScript murni (vanilla), manipulasi DOM langsung |
| Baca Excel | [SheetJS (xlsx)](https://sheetjs.com/) 0.18.5 (via CDN) |
| Ikon & font | Tabler Icons, Inter (Google Fonts) |
| Ekspor PDF | html2pdf.js |
| Hosting | Vercel (file statis) |

Tidak ada backend, database, `npm install`, atau proses *build*. Semua diproses di sisi browser.

## Struktur repositori

```
.
├── index.html                       # Aplikasi (buka langsung di browser)
├── logo-MU-new-hires-01.webp        # Logo perusahaan
├── README.md                        # Halaman ini
├── LICENSE
└── docs/
    ├── TECHNICAL_DOCUMENTATION.md   # Arsitektur & pembahasan kode detail
    ├── USER_GUIDE.md                # Panduan penggunaan (manual)
    ├── SOP.md                       # Standard Operating Procedure
    └── DATA_FORMAT.md               # Spesifikasi format file Excel
```

## Cara pakai cepat

1. Buka `index.html` di browser (atau buka situs yang sudah di‑deploy).
2. Klik **"Lihat contoh data"** untuk mencoba dengan data dummy, **atau**
3. Upload file Excel-mu (lihat [DATA_FORMAT.md](docs/DATA_FORMAT.md) untuk format kolom).
4. Gunakan **Filter data**, baca KPI & grafik, dan buka tab **Preventive Maintenance** untuk kalender PM.
5. Klik **Ekspor → Simpan sebagai PDF** untuk mengunduh laporan.

Panduan lengkap ada di [docs/USER_GUIDE.md](docs/USER_GUIDE.md).

## Dokumentasi

- **[Dokumentasi Teknis](docs/TECHNICAL_DOCUMENTATION.md)** — arsitektur, alur data, dan pembahasan tiap modul/fungsi kode.
- **[User Guide](docs/USER_GUIDE.md)** — panduan langkah demi langkah untuk pengguna akhir.
- **[SOP](docs/SOP.md)** — prosedur baku: memperbarui data, deploy, dan pemeliharaan.
- **[Format Data](docs/DATA_FORMAT.md)** — kolom & sheet Excel yang dibutuhkan.

## Privasi data

Seluruh file Excel diproses **lokal di browser** menggunakan `FileReader`. Tidak ada data yang diunggah ke server. Menutup atau me‑refresh halaman akan menghapus data dari memori.

## Kredit

Dibuat oleh **Azmi Dava Alfarizqi (Davaa)** — Teknik Industri, President University.
