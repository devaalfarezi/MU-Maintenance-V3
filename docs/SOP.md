# SOP — MU Maintenance & Reliability Dashboard

*Standard Operating Procedure* untuk mengoperasikan, memperbarui data, men‑deploy, dan memelihara dashboard.

---

## SOP‑01 — Memperbarui data bulanan

**Tujuan:** memasukkan data downtime/PM/sparepart terbaru ke dashboard.

**Frekuensi:** setiap periode pelaporan (mis. bulanan).

**Langkah:**

1. Siapkan file Excel sesuai format pada [DATA_FORMAT.md](DATA_FORMAT.md). Pastikan:
   - Sheet data downtime bernama **DATA2025** (atau memuat kolom `Loss Category`, `Duration (menit)`, `Line`, `Packer`, `Equipment`, dll).
   - Tidak ada baris rekap/total tercecer tanpa `Loss Category` (akan otomatis diabaikan, tetapi lebih baik bersih).
   - Kolom tanggal (`Date2`) terisi tanggal yang benar.
2. Buka dashboard.
3. Klik **Hapus data** untuk membersihkan data lama (opsional).
4. **Upload** file Excel — pastikan penanda jenis file berubah **hijau**.
5. Verifikasi angka **Total downtime** dan **Kejadian breakdown** cocok dengan perhitungan Excel (`SUMIFS`/`COUNTIFS`).
6. Jika perlu dibagikan, **Ekspor → Simpan sebagai PDF**.

**Validasi cepat (di Excel):**

```
Total downtime      = SUM(Duration pada baris ber-Loss Category)
Kejadian breakdown  = COUNTIFS(Loss Category; "BKD - Breakdown")
```

---

## SOP‑02 — Deploy / memperbarui situs (Vercel)

**Tujuan:** menerbitkan versi terbaru `index.html`.

**Prasyarat:** akun Vercel terhubung ke repositori GitHub ini.

**Langkah:**

1. Pastikan di root repositori ada **`index.html`** dan **file logo** (`logo-MU-new-hires-01.webp`).
2. *Commit* & *push* perubahan ke branch utama (lihat SOP‑04).
3. Vercel otomatis mem‑build & men‑deploy. Tunggu status **"Ready"**.
4. Buka situs di jendela **Incognito** (Ctrl+Shift+N) untuk memastikan versi terbaru termuat (menghindari cache).
5. Uji singkat: upload data, cek filter & kalender berfungsi.

> **Penting soal cache:** setelah deploy, browser sering masih menyimpan versi lama. Selalu verifikasi lewat Incognito atau **hard refresh** (Ctrl+Shift+R).

---

## SOP‑03 — Verifikasi setelah perubahan kode

**Tujuan:** memastikan tidak ada fitur yang rusak setelah menyunting kode.

**Langkah:**

1. Validasi sintaks JavaScript (ekstrak isi `<script>`, jalankan pemeriksaan sintaks).
2. Buka aplikasi, klik **Lihat contoh data** — pastikan semua tab tampil tanpa error.
3. Upload data asli — pastikan:
   - Filter muncul lengkap dan bisa dicentang; angka berubah saat difilter.
   - KPI cocok dengan Excel.
   - Kalender menampilkan PM (kuning) & breakdown (merah); klik sel membuka panel detail.
   - Ekspor PDF berjalan.
4. Bandingkan daftar fitur dengan versi sebelumnya untuk memastikan tidak ada yang hilang.

---

## SOP‑04 — Alur Git (menyimpan perubahan)

**Tujuan:** menyimpan riwayat perubahan dengan rapi.

**Langkah dasar:**

```bash
git add .
git commit -m "deskripsi singkat perubahan"
git push
```

**Konvensi pesan commit yang disarankan:**

- `feat: ...` untuk fitur baru
- `fix: ...` untuk perbaikan bug
- `docs: ...` untuk perubahan dokumentasi
- `style: ...` untuk perubahan tampilan/CSS

Contoh: `fix: filter angka tidak mengubah KPI saat di-uncheck`.

---

## SOP‑05 — Pemeliharaan berkala

| Item | Tindakan | Frekuensi |
|---|---|---|
| Pustaka CDN | Cek apakah versi SheetJS/html2pdf masih aktif | Semesteran |
| Backup | Simpan salinan `index.html` & data mentah | Tiap update |
| Format Excel | Pastikan tim data memakai format kolom yang sama | Berkelanjutan |
| Dokumentasi | Perbarui docs bila ada fitur baru | Setiap perubahan besar |

---

## Kontak / kepemilikan

Pemilik & pengembang: **Davaa** — Teknik Industri, President University.
