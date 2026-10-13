# Tugas Computer Forensic — Kelompok 4 (Image Linux)

**Tugas A: Pembuatan Disk Image OS Linux** - Forensik Digital Week 7.

---

## 👥 Anggota Kelompok 4

| No | Nama Lengkap |
|---|---| 
| 1. | Imam Mahmud Dalil Fauzan |
| 2. | M. Hikari Reiziq Rakhmadinta |
| 3. | Ivan Syarifuddin |
| 4. | Nayyara Ashila |
| 5. | Rafika Az Zahra Kusumastuti |
---

# Linux Digital Forensics Case Study

**Jenis Kasus:** Simulasi investigasi filesystem Linux  
**Filesystem:** ext4  
**Format Image:** Raw disk image (`.img`)  
**Format Distribusi:** GZIP (`.img.gz`)

## 1. Deskripsi Proyek

Kasus disusun dalam lingkungan simulasi dengan beberapa jenis artefak, yaitu:

- File yang telah dihapus dari direktori.
- File tersembunyi menggunakan konvensi nama Linux.
- File yang isinya telah dimodifikasi.
- Catatan aktivitas skenario yang dapat digunakan sebagai salah satu sumber informasi investigasi.
- Salinan dokumen versi awal untuk membantu perbandingan perubahan.

## 2. Tujuan Investigasi

Investigator diharapkan mampu:

1. Mengidentifikasi file yang dihapus, tersembunyi, atau dimodifikasi.
2. Memeriksa metadata filesystem, termasuk ukuran file, permission, dan timestamp yang tersedia.
3. Membandingkan dokumen versi awal dan versi akhir untuk menemukan perubahan.
4. Mengidentifikasi username yang tercatat dalam bukti dan menilai kekuatan bukti tersebut.
5. Menyusun timeline kejadian berdasarkan seluruh sumber yang tersedia.
6. Memverifikasi integritas image menggunakan SHA-256.
7. Menulis laporan investigasi yang dapat ditelusuri kembali ke bukti pendukung.

## 3. Struktur File Distribusi

```text
Fordig-Week7/
├── README.md
├── linux_forensic_case.img.gz
└── output/
    ├── SHA256SUMS-IMAGE.txt
    ├── SHA256SUMS-COMPRESSED.txt
    └── SHA256SUMS.txt

```

Keterangan:

| File | Fungsi |
|---|---|
| `linux_forensic_case.img.gz` | Image terkompresi untuk distribusi |
| `SHA256SUMS-IMAGE.txt` | Hash SHA-256 image mentah hasil ekstraksi |
| `SHA256SUMS-COMPRESSED.txt` | Hash SHA-256 file terkompresi |
| `SHA256SUMS.txt` | Salinan hash image mentah untuk kemudahan verifikasi |
| `README.md` | Dokumentasi dan petunjuk investigasi |

## 4. Integritas Barang Bukti

SHA-256 digunakan untuk mendeteksi perubahan byte pada file image.

Sebelum melakukan analisis, investigator wajib:

1. Mengunduh file image dan checksum.
2. Menghitung hash file `.gz` yang diterima.
3. Membandingkan hasilnya dengan `SHA256SUMS-COMPRESSED.txt`.
4. Mengekstrak image.
5. Menghitung hash image mentah.
6. Membandingkan hasilnya dengan `SHA256SUMS-IMAGE.txt`.
7. Mencatat hasil verifikasi dalam laporan.

Hash yang cocok menunjukkan bahwa file yang diperiksa memiliki nilai hash yang sama dengan acuan. Hash tidak membuktikan bahwa isi image sejak awal benar atau bahwa suatu aktivitas dilakukan oleh orang tertentu.

## 5. Persiapan Analisis

Dapat dilakukan menggunakan salah satu atau beberapa tool berikut:

- **Autopsy:** antarmuka grafis untuk pemeriksaan filesystem, pencarian file, dan analisis artefak.
- **Sleuth Kit:** kumpulan command-line tools untuk memeriksa filesystem dan metadata.
- **Linux WSL/Ubuntu:** lingkungan kerja untuk ekstraksi, hashing, dan pemeriksaan dasar.
- **Hex editor:** untuk memeriksa representasi byte ketika diperlukan.

Gunakan salinan kerja dan pertahankan image asli dalam keadaan tidak berubah.

## 6. Ekstraksi Image

Pada Ubuntu atau WSL, jalankan:

```bash
gzip -dk linux_forensic_case.img.gz
```

Perintah tersebut menghasilkan `linux_forensic_case.img` tanpa menghapus file `.gz`.

Verifikasi hash image mentah:

```bash
sha256sum linux_forensic_case.img
```

Pastikan nilai hash sesuai dengan checksum yang disediakan.

## 7. Alur Investigasi

Lakukan pemeriksaan dengan urutan berikut:

### Tahap A: Pemeriksaan awal

- Catat nama file, ukuran, format, dan hash image.
- Identifikasi filesystem.
- Dokumentasikan tool dan versi yang digunakan.

### Tahap B: Pemeriksaan filesystem

- Telusuri struktur direktori.
- Cari entri file yang tersembunyi.
- Periksa entri file yang telah dihapus.
- Periksa metadata dan permission file.

### Tahap C: Analisis perubahan

- Bandingkan dokumen versi awal dan versi akhir.
- Gunakan hash untuk membandingkan integritas file.
- Identifikasi bagian isi yang berubah.
- Catat timestamp dan sumber informasi yang relevan.

### Tahap D: Analisis timeline dan username

- Periksa catatan aktivitas yang tersedia.
- Bandingkan waktu pada catatan dengan metadata filesystem.
- Bedakan waktu modifikasi isi file, perubahan metadata, dan waktu pencatatan aktivitas.
- Jangan menyimpulkan identitas pelaku hanya dari satu artefak.

### Tahap E: Pelaporan

Susun temuan dalam tabel yang memuat:

- Nomor bukti.
- Nama atau lokasi artefak.
- Status file.
- Temuan utama.
- Username yang terindikasi.
- Timestamp dan zona waktu.
- Metode pemeriksaan.
- Hash atau metadata pendukung.
- Tingkat keyakinan dan keterbatasan kesimpulan.

## 8. Prinsip Penanganan Bukti

- Jangan memodifikasi image asli.
- Kerjakan analisis pada salinan.
- Catat hash sebelum dan sesudah proses yang relevan.
- Dokumentasikan setiap langkah pemeriksaan.
- Simpan output analisis secara terpisah.
- Jangan mengklaim atribusi pengguna tanpa dukungan bukti yang memadai.