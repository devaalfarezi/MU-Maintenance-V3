# User Guide — MU Maintenance & Reliability Dashboard

Panduan penggunaan untuk pengguna akhir (tim maintenance, engineering, atau reviewer). Tidak perlu pengetahuan pemrograman.

---

## 1. Membuka dashboard

Ada dua cara:

- **Online** buka alamat situs yang sudah di‑deploy (mis. di Vercel).
- **Lokal** klik dua kali file `index.html`, akan terbuka di browser.

> Disarankan memakai Google Chrome atau Microsoft Edge versi terbaru.

## 2. Memasukkan data

Di bagian atas ada **zona upload** dengan empat penanda jenis file:

| Penanda | Isi file |
|---|---|
| Downtime | Data kejadian downtime & breakdown |
| Sparepart usage | Riwayat pemakaian sparepart |
| Stok sparepart | Stok sparepart terkini |
| PM Monitoring | Jadwal Preventive Maintenance per minggu |

Cara mengisi:

1. **Seret file Excel** ke zona upload, atau **klik** zona itu lalu pilih file.
2. Kamu bisa memasukkan **beberapa file sekaligus**; aplikasi mengenali jenisnya otomatis dari isinya.
3. Penanda file akan berubah **hijau** bila berhasil dibaca.

Ingin mencoba tanpa data sendiri? Klik **"Lihat contoh data"**. Untuk menghapus semua data, klik **"Hapus data"**.

> **Format kolom** yang dibutuhkan ada di [DATA_FORMAT.md](DATA_FORMAT.md).

## 3. Tab BREAKDOWN MAP

Tab utama untuk analisis downtime.

### 3.1 Filter data

Deretan tombol filter dibuat **otomatis dari kolom Excel-mu**. Ada dua jenis:

- **Filter centang** (mis. Line, Packer, Loss Category, Equipment) — klik tombolnya, lalu centang/hapus nilai yang diinginkan. Ada kotak **Cari**, tombol **(Pilih semua)**, **Pilih yang tampil**, dan **Kosongkan**. Angka di sebelah tiap nilai = jumlah barisnya, dan ikut menyesuaikan saat filter lain aktif.
- **Filter rentang** (angka) — isi nilai **min** dan **maks**.

Ada juga kotak **Duration (menit) antara min–maks** khusus. Tombol **Reset filter** mengembalikan semua ke semula.

Semua yang tampil di layar (KPI, grafik, ringkasan, tabel, kalender) mengikuti filter aktif.

### 3.2 KPI

Tiga kartu ringkasan:

- **Total downtime** total menit berhenti pada data terfilter.
- **Kejadian breakdown** jumlah kejadian.
- **MTTR rata‑rata** rata‑rata menit per kejadian breakdown.

### 3.3 Ringkasan per Line

Total downtime dan jumlah kejadian breakdown untuk tiap Line.

### 3.4 Tren downtime per bulan

Diagram batang menit downtime tiap bulan.

### 3.5 Analisis downtime

Diagram batang menit downtime yang bisa **dikelompokkan per dimensi** — klik salah satu tombol dimensi (Equipment, Equipment Detail, Line, Packer, Category BKD, Loss Category, Bulan, dll).

### 3.6 Detail kejadian

Tabel baris per baris (Bulan, Line, Packer, Equipment, Description, dll). Gunakan kotak pencarian untuk menyaring. Maksimal 300 baris ditampilkan; persempit dengan filter bila perlu.

## 4. Tab SPAREPARTS USAGE

Menampilkan ringkasan pemakaian dan stok sparepart (bila file terkait di‑upload), termasuk daftar item yang perlu perhatian.

## 5. Tab PREVENTIVE MAINTENANCE

### 5.1 KPI PM & Jadwal perawatan rutin

Ringkasan penjadwalan dan usulan interval perawatan yang diturunkan dari MTBF breakdown.

### 5.2 Calendar PM Monitoring

Kalender **bulan (Jan–Des) × minggu (W1–W5)** yang menampilkan:

- **Bulatan kuning** = jadwal PM (Preventive Maintenance).
- **Segitiga merah** = kejadian breakdown.

Cara pakai:

1. Gunakan tombol **Semua / PM / Breakdown** untuk memilih apa yang ditampilkan.
2. Gunakan kotak **pencarian** untuk menyorot equipment/komponen tertentu.
3. **Klik sebuah sel** berwarna → muncul **panel di sisi kanan** yang berisi dua bagian:
   - **PM (Preventive Maintenance)** — daftar equipment bernomor.
   - **BKD (Breakdown)** — daftar equipment bernomor.
   Keduanya diurutkan per minggu dan diberi chip **W##**.
4. **Klik salah satu equipment** di panel → detailnya: Component, Action, Task, PIC (untuk PM), Line & Durasi (untuk breakdown), nomor minggu, dan rentang tanggal.
5. Tutup panel dengan tombol **×**, klik di luar panel, atau tekan **Esc**.

> Kalender mengikuti **Filter data** di tab Breakdown Map — jadi menyaring Line/kategori juga akan mengubah kejadian breakdown yang tampil di kalender. Jadwal PM (kuning) tidak terpengaruh filter downtime karena berasal dari file PM Monitoring yang terpisah.

## 6. Mengganti tema

Klik ikon **bulan/matahari** di kanan atas untuk beralih **terang ↔ gelap**.

## 7. Mengekspor laporan

1. Klik **Ekspor** di kanan atas.
2. Pilih **Simpan sebagai PDF** (mengunduh berkas PDF) atau **Cetak** (dialog cetak browser).
3. Saat ekspor PDF, tampilan otomatis dialihkan ke mode terang agar hasilnya rapi.

## 8. Pertanyaan umum

**Kenapa filternya banyak sekali?**
Filter dibuat otomatis dari semua kolom berjudul di Excel-mu. Beberapa kolom teknis memang sengaja tidak dijadikan filter.

**Kenapa angka tidak berubah saat saya ganti filter?**
Kemungkinan besar browser masih menampilkan versi lama. Lakukan **hard refresh** (Ctrl+Shift+R) atau buka di jendela **Incognito**.

**Apakah data saya aman?**
Ya. Semua diproses di browser; tidak ada yang dikirim ke server. Menutup halaman menghapus data dari memori.

**Kenapa tahun di kalender selalu 2025?**
Tahun dikunci ke tahun yang paling banyak muncul di datamu. Jika ada tanggal yang salah ketik (mis. 2032), kejadian itu tetap muncul tetapi ditempatkan berdasarkan bulannya.
