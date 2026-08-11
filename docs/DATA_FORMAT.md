# Format Data Excel

Dashboard mengenali jenis file **otomatis dari isi header**‑nya, bukan dari nama file. Selama header di bawah ini ada, file akan terbaca.

---

## 1. File Downtime / Breakdown

**Dikenali dari:** adanya kolom **`Loss Category`**.
**Sheet:** diutamakan bernama **`DATA2025`**. Bila ada beberapa sheet ber‑`Loss Category`, aplikasi memilih yang kolomnya paling lengkap.

**Kolom yang dibaca** (nama header dicocokkan otomatis, tidak harus urut):

| Kolom | Kegunaan |
|---|---|
| `Date2` | Tanggal kejadian (untuk pemetaan minggu di kalender) |
| `Month` | Bulan (grafik tren) |
| `Packer` | Filter |
| `Description` | Detail kejadian (dapat dicari) |
| `Loss Category` | Menentukan baris breakdown ("BKD - Breakdown") |
| `Category BKD` | Kategori breakdown (Mechanical/Electrical/…) |
| `Equipment (Untuk kategori Engineering Downtime)` | Nama mesin |
| `Equipment Detail (Untuk kategori Engineering Downtime)` | Komponen |
| `Duration (menit)` | Menit downtime (semua perhitungan waktu) |
| `Line` | Line produksi (Line 1 / Line 2) |
| `Equipment` | Equipment (CBT1/CBT2) |

**Catatan:**

- Semua kolom **berjudul** lain juga otomatis menjadi filter, kecuali yang sengaja dikecualikan (Total Spout berhenti, Start, End, Duration, #spout/line, Multiple factor, OEE Impact, Cause, Discription, Root Cause, Action Plan, CODE).
- Baris **tanpa `Loss Category`** (baris rekap/total) otomatis diabaikan.
- Perhitungan kunci:
  - **Kejadian breakdown** = baris dengan `Loss Category` mengandung "BKD".
  - **Total downtime** = jumlah `Duration (menit)`.
  - **MTTR** = jumlah Duration baris breakdown ÷ jumlah kejadian breakdown.

---

## 2. File PM Monitoring

**Dikenali dari:** adanya kolom **`Process Step`** atau **`Strategy`**, beserta kolom minggu.

**Kolom yang dibaca:**

| Kolom | Kegunaan |
|---|---|
| `Equipment` | Nama mesin |
| `Component` | Komponen |
| `Action` | Tindakan |
| `Task` | Jenis pekerjaan |
| `PIC` | Penanggung jawab |
| `Status` | Status (mis. Open/Close) |
| `W01` … `W52` | Kolom minggu; sel terisi = ada jadwal PM pada minggu itu |

Nomor minggu (`W01`–`W52`) dipetakan ke posisi **bulan × minggu** pada kalender.

---

## 3. File Sparepart Usage

**Dikenali dari:** adanya kolom **`Item`**. Berisi riwayat pemakaian sparepart.

---

## 4. File Stok Sparepart

**Dikenali dari:** adanya kolom **`Item No.`**. Berisi stok sparepart terkini.

---

## Tips agar data terbaca akurat

1. **Jangan ganti nama header** kolom penting di atas.
2. Pastikan **satu sheet data utama** (idealnya `DATA2025`) — hindari banyak sheet ringkasan ber‑`Loss Category` yang membingungkan.
3. Isi **`Date2`** dengan tanggal yang benar (format tanggal Excel), bukan teks.
4. Hindari baris total/rekap di tengah atau bawah data tanpa `Loss Category`.
5. Nilai menit di `Duration (menit)` harus **angka**, bukan teks.
