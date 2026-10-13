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

## 📦 Link Image

| Item | Keterangan |
|---|---|
| **Link Google Drive** | - |
| **Nama file** | `kelompok4_linux.img.xz` |
| **Ukuran terkompresi** | - |
| **Ukuran setelah ekstrak** | - |

---

## 🔐 Integritas Bukti (SHA-256)

Verifikasi hash **setelah ekstrak** untuk memastikan image tidak berubah:

```
-
```

Cara verifikasi:

```bash
sha256sum kelompok4_linux.img
```

Hasilnya harus identik dengan hash di atas.

---

## 🖥️ Spesifikasi Image

| Properti | Nilai |
|---|---|
| Sistem operasi | Ubuntu <ISI_RELEASE, mis. 22.04 Jammy> (minimal / `debootstrap --variant=minbase`) |
| Format image | Raw disk image (`.img`) |
| Tabel partisi | MBR (msdos) |
| Jumlah partisi | 1 partisi |
| Filesystem | ext4 (label: `kelp4root`) |
| Awal partisi | Sektor 2048 (offset 1 MiB = 1.048.576 byte) |
| Hostname | `forensik-kelp4` |
| Dibuat dengan | WSL2 Ubuntu, `losetup`, `parted`, `mkfs.ext4`, `debootstrap` |

---

## 🚀 Cara Menggunakan

### 1. Ekstrak

```bash
xz -d -k kelompok4_linux.img.xz
```

(User Windows bisa memakai 7-Zip.)

### 2. Lihat tabel partisi

```bash
mmls kelompok4_linux.img
```

### 3. Mount sebagai **read-only** (disarankan agar bukti tidak berubah)

```bash
sudo mkdir -p /mnt/bukti
sudo mount -o ro,loop,offset=$((2048*512)) kelompok4_linux.img /mnt/bukti
# ... analisis ...
sudo umount /mnt/bukti
```

> 💡 Bekerjalah pada **salinan** image, dan simpan image asli tetap utuh.

---

## 🎯 Tugas Analisis (Tugas B)

Dari image ini, temukan:

1. **File yang terhapus / tersembunyi / termodifikasi.**
2. **Credential (username)** yang digunakan untuk melakukan perubahan tersebut, **kapan** kejadiannya, serta data lain yang dianggap relevan.

---

## 🧰 Tools yang Disarankan

| Tujuan | Tool |
|---|---|
| Analisis partisi dan filesystem | The Sleuth Kit (`mmls`, `fls`, `istat`, `icat`) |
| Analisis GUI | Autopsy |
| File carving | `photorec`, `foremost` |
| Pencarian string | `strings`, `grep` |
| Mount image | `mount -o ro,loop`, `losetup` |
