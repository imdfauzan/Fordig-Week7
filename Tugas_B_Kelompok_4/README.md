# LAPORAN TUGAS B — COMPUTER FORENSIC
## Investigasi Digital Forensik Image Windows Kelompok 3

**Kelompok Pemeriksa:** 4  
**Kelompok Penyedia Image:** 3  
**Tools:** Autopsy 4.23.1  
**Nama Image:** `evidence_windows_kelompok3.vhd`  
**Tanggal Insiden:** 12 Oktober 2026  
**Hostname:** `NUSA-WS-K3`

---

## 1. Tujuan Investigasi

Investigasi ini dilakukan untuk memenuhi Tugas B mata kuliah Computer Forensic, yaitu menganalisis image sistem operasi dari kelompok lain untuk menemukan:

1. File yang terhapus (*deleted file*).
2. File yang disembunyikan (*hidden file*).
3. File yang dimodifikasi (*modified file*).
4. Username, SID, dan informasi akun yang terkait dengan aktivitas tersebut.
5. Kronologi kejadian berdasarkan artefak digital yang ditemukan.

Image Windows dari Kelompok 3 dianalisis menggunakan Autopsy 4.23.1. Pemeriksaan dilakukan terhadap struktur direktori, konten file, riwayat perintah PowerShell, dan transkrip log audit.

## 2. Metodologi Investigasi

### 2.1 Menambahkan Image ke Autopsy

Image barang bukti ditambahkan ke dalam case Autopsy menggunakan fitur **Add Data Source**. Image yang digunakan adalah `evidence_windows_kelompok3.vhd`.

![Menambahkan data source di Autopsy](https://github.com/user-attachments/assets/36fc733d-d2c8-4ac2-8ee3-0d621518c4c0)

### 2.2 Menemukan File Terhapus

Pada panel **File Views**, dipilih kategori **Deleted Files**. File yang ditemukan kemudian diekstrak menggunakan fitur **Extract File(s)** untuk memeriksa kontennya.

![Proses ekstraksi file terhapus](https://github.com/user-attachments/assets/3d5ce10f-5791-4c9a-bc40-1b81f199e495)

Setelah proses ekstraksi, diperoleh file hasil ekstraksi yang dapat diperiksa lebih lanjut.

![File hasil ekstraksi](https://github.com/user-attachments/assets/4073e8fd-2c38-4b00-8427-61881a571dcd)

### 2.3 Analisis File Terhapus

File `financial_projection_2026.xlsx` ditemukan pada kategori Deleted Files. Metadata yang ditampilkan Autopsy menunjukkan bahwa entri file tidak lagi teralokasi (*unallocated*).

![Hasil pemeriksaan file terhapus](https://github.com/user-attachments/assets/b8ac5d5a-9b74-4095-814b-4f01402dd64f)

Dari konten hasil ekstraksi, ditemukan flag berikut:

`FLAG{W1ND0W5_D3L3T3D_F1L3_K3L3_8891}`

Konten yang diperoleh berupa teks biasa, bukan workbook Excel yang valid meskipun nama file memiliki ekstensi `.xlsx`. Oleh karena itu, temuan ini dicatat sebagai konten file yang berhasil dipulihkan, bukan pemulihan workbook Excel secara utuh.

### 2.4 Pemeriksaan Struktur Direktori

Setelah memeriksa file terhapus, pemeriksaan dilanjutkan melalui bagian **Data Sources** pada panel kiri Autopsy untuk menelusuri direktori lain di dalam image.

![Struktur direktori image Windows](https://github.com/user-attachments/assets/f8f268e3-f9ff-42bd-9ef5-83fd59e4f091)

### 2.5 Analisis File Tersembunyi

Pada struktur direktori image, dipilih folder `System_Security` yang berada di bawah volume `vol2`.

![Folder System_Security](https://github.com/user-attachments/assets/18bfa51e-6c8b-460f-b735-125e726da7a6)

Di dalam folder tersebut ditemukan file `backup_credential.dat`. Konten file menyatakan bahwa file memiliki atribut `HIDDEN & SYSTEM` yang dikaitkan dengan perintah `attrib +h +s`.

![Konten backup_credential.dat](https://github.com/user-attachments/assets/98c5793d-7c12-4513-8c2a-33c895c41e8f)

Flag yang ditemukan:

`FLAG{W1ND0W5_H1DD3N_5Y5T3M_K3L3_4981}`

Konten file juga mencantumkan keyword `EMERGENCY_SUPERUSER_KEYPAIR` dan akun pemilik `hendra_admin`. Atribut Hidden dan System perlu dicocokkan dengan metadata filesystem untuk memastikan atribut tersebut benar-benar tercatat pada file di dalam image.

### 2.6 Analisis Riwayat Perintah PowerShell

Pemeriksaan dilanjutkan ke direktori:

`Users/hendra_admin/AppData/Local/Microsoft/Windows/PowerShell`

Di dalam folder tersebut ditemukan `ConsoleHost_history.txt`, yang memuat riwayat perintah PowerShell.

![File ConsoleHost_history.txt](https://github.com/user-attachments/assets/85dca312-1b75-4eb1-8c1b-6989a160d35b)

Riwayat tersebut mencantumkan perintah-perintah penting berikut:

```powershell
whoami /user
cd C:\Users\hendra_admin\Documents
Remove-Item -Path .\financial_projection_2026.xlsx -Force
cd C:\System_Security
attrib +h +s .\backup_credential.dat
notepad.exe .\access_control_policy.cfg
Get-Date -Format "yyyy-MM-dd HH:mm:ss"
Clear-History
exit
```

Analisis terhadap riwayat perintah menunjukkan:

- `whoami /user` digunakan untuk meminta identitas pengguna dan SID.
- `Remove-Item` menyebut file `financial_projection_2026.xlsx` yang ditemukan dalam kategori Deleted Files.
- `attrib +h +s` digunakan untuk menetapkan atribut Hidden dan System pada `backup_credential.dat`.
- `notepad.exe` digunakan untuk membuka `access_control_policy.cfg`.
- `Clear-History` digunakan untuk membersihkan riwayat perintah pada sesi PowerShell saat itu.

Riwayat ini menghubungkan aktivitas yang diperiksa dengan akun `hendra_admin`. Namun, riwayat tersebut tidak mencantumkan timestamp individual untuk setiap perintah.

### 2.7 Analisis Log Audit

Pemeriksaan dilanjutkan ke direktori:

`Windows/System32/LogFiles/Audit`

Di dalam folder tersebut ditemukan `Security_Audit_Incident.log`.

![File Security_Audit_Incident.log](https://github.com/user-attachments/assets/f9eaca52-3f60-4d24-9b0b-a765ffb7be41)

Konten file memuat transkrip empat kejadian yang berkaitan dengan logon pengguna, penghapusan file, perubahan atribut file, dan akses tulis terhadap konfigurasi keamanan.

File ini merupakan transkrip teks yang mencantumkan Event ID Windows. Karena Windows Event Log asli berformat EVTX tidak diperiksa, informasi Event ID dalam laporan ini diperlakukan sebagai catatan yang tercantum dalam transkrip, bukan hasil validasi independen terhadap EVTX asli.

### 2.8 Pemeriksaan File Termodifikasi

File `access_control_policy.cfg` ditemukan di direktori `System_Security`. Kontennya mencantumkan revisi `v1.1 (MODIFIED)` dan bagian `[TAMPERED_INJECTION_SECTION]`.

Informasi yang ditemukan dalam konten file meliputi:

- Keyword: `TAMPERED_SECURITY_GATEWAY_BYPASS`
- `bypass_credential_guard = TRUE`
- `exfiltration_channel = ACTIVE`
- `exfiltration_port = 4443`
- Akun yang tercantum: `hendra_admin`
- Timestamp yang tercantum: 12 Oktober 2026, 11:30:08 WIB

Flag yang ditemukan:

`FLAG{W1ND0W5_M0D1F13D_P0L1CY_K3L3_7734}`

Riwayat PowerShell menunjukkan bahwa file dibuka menggunakan Notepad. Transkrip log audit juga mencatat akses tulis terhadap file tersebut pada pukul 11:30:08 WIB.

Temuan ini menunjukkan indikasi modifikasi konfigurasi keamanan. Namun, pembuktian perubahan isi secara definitif memerlukan perbandingan dengan versi baseline yang tepercaya.

---

## 3. Hasil Temuan Investigasi

### 3.1 Tabel Temuan File

| No. | Nama File | Jenis Temuan | Lokasi | Flag |
|---|---|---|---|---|
| 1 | `financial_projection_2026.xlsx` | Deleted File | `C:\Users\hendra_admin\Documents\` | `FLAG{W1ND0W5_D3L3T3D_F1L3_K3L3_8891}` |
| 2 | `backup_credential.dat` | Hidden/System | `C:\System_Security\` | `FLAG{W1ND0W5_H1DD3N_5Y5T3M_K3L3_4981}` |
| 3 | `access_control_policy.cfg` | Modified File | `C:\System_Security\` | `FLAG{W1ND0W5_M0D1F13D_P0L1CY_K3L3_7734}` |

### 3.2 Analisis File Terhapus

File `financial_projection_2026.xlsx` ditemukan pada kategori Deleted Files dengan status entri file tidak teralokasi. Konten hasil ekstraksi memuat flag dan informasi dokumen terkait audit transfer dana.

Riwayat PowerShell mencantumkan perintah `Remove-Item` yang secara eksplisit menyebut nama file tersebut. Transkrip log audit juga mencatat kejadian penghapusan pada pukul 11:15:42 WIB.

Kesesuaian nama file, perintah, dan timestamp mendukung kesimpulan bahwa file tersebut dihapus melalui sesi yang dikaitkan dengan akun `hendra_admin`.

### 3.3 Analisis File Tersembunyi

File `backup_credential.dat` berada di folder `System_Security`. Kontennya menyatakan atribut Hidden dan System. Riwayat PowerShell mencantumkan perintah `attrib +h +s`, sedangkan transkrip log audit mencatat perubahan atribut pada pukul 11:22:15 WIB.

Kesesuaian antara konten file, riwayat perintah, dan transkrip audit mendukung temuan bahwa file tersebut disembunyikan menggunakan atribut Hidden dan System. Validasi atribut filesystem tetap diperlukan sebagai pemeriksaan tambahan.

### 3.4 Analisis File Termodifikasi

File `access_control_policy.cfg` memuat konfigurasi yang menunjukkan indikasi bypass kontrol kredensial dan kanal eksfiltrasi yang ditandai aktif. Konten file juga mencantumkan flag modifikasi dan identitas akun `hendra_admin`.

Riwayat PowerShell menunjukkan bahwa file dibuka menggunakan Notepad, sementara transkrip log audit mencatat akses tulis pada pukul 11:30:08 WIB. Bukti tersebut mendukung adanya aktivitas terkait modifikasi file, tetapi belum membuktikan secara independen bahwa perubahan tertentu telah disimpan tanpa adanya versi baseline untuk perbandingan.

---

## 4. Identifikasi Username, Domain, dan SID

Berdasarkan `ConsoleHost_history.txt` dan `Security_Audit_Incident.log`, informasi akun yang tercantum adalah:

| Parameter | Hasil |
|---|---|
| Username | `hendra_admin` |
| Domain yang tercantum | `NUSA-WS-K3` |
| Hostname lengkap pada header log | `NUSA-WS-K3.nusameridian.corp` |
| SID | `S-1-5-21-3829104812-1928374615-1002` |
| Jenis logon | Interactive Console Logon |
| Event ID logon dalam transkrip | `4624` |

SID yang sama tercantum pada beberapa kejadian dalam transkrip audit. Riwayat PowerShell juga mencantumkan perintah `whoami /user`, tetapi output perintah tersebut tidak tersedia di dalam riwayat yang diperiksa.

Dengan demikian, akun yang paling kuat dikaitkan dengan aktivitas dalam artefak adalah `hendra_admin`. Temuan ini merupakan atribusi terhadap akun yang tercatat, bukan identifikasi pasti terhadap individu yang menggunakan akun tersebut.

---

## 5. Rekonstruksi Timeline Insiden

Seluruh timestamp berikut menggunakan WIB (UTC+7), sesuai informasi yang dicantumkan dalam artefak.

| Waktu | Aktivitas | Sumber Bukti |
|---|---|---|
| 10:45:10 | Logon interaktif akun `hendra_admin` | `Security_Audit_Incident.log`, Event ID 4624 |
| 10:45:10 | Sesi PowerShell dimulai | `ConsoleHost_history.txt` |
| 11:15:42 | Penghapusan `financial_projection_2026.xlsx` | PowerShell history dan transkrip audit, Event ID 4660 |
| 11:22:15 | Perubahan atribut `backup_credential.dat` menjadi Hidden dan System | PowerShell history dan transkrip audit, Event ID 4670 |
| 11:30:08 | Akses tulis terhadap `access_control_policy.cfg` | Transkrip audit, Event ID 4663, dan timestamp file |

Berdasarkan urutan waktu yang tercantum, aktivitas dimulai dengan logon interaktif, diikuti penghapusan file, perubahan atribut file, dan akses tulis terhadap konfigurasi keamanan.

Perintah `Clear-History` juga ditemukan pada akhir riwayat PowerShell. Perintah tersebut menunjukkan upaya membersihkan riwayat perintah sesi, tetapi tidak membuktikan bahwa seluruh jejak aktivitas pada sistem berhasil dihapus.

Timestamp pada tabel mengikuti transkrip audit dan metadata yang tersedia. Karena EVTX asli tidak diperiksa, timeline ini merupakan rekonstruksi berdasarkan artefak yang ditemukan, bukan validasi menyeluruh terhadap log sistem asli.

---

## 6. Pemeriksaan Integritas Barang Bukti

Penyedia image mencantumkan hash SHA-256 referensi untuk file VHD:

`c05779484da6a583bc53c92b64a5e13987842b0541b80bb996caacf848c99153`

Nilai tersebut perlu dibandingkan dengan SHA-256 aktual file VHD yang diunduh.

| Parameter | Hasil |
|---|---|
| Hash SHA-256 referensi | Tersedia dari penyedia |
| Hash SHA-256 aktual VHD | Belum dicatat |
| Status perbandingan | Belum diverifikasi |
| Hash file hasil ekstraksi | Dihitung secara terpisah jika diperlukan |

Dengan demikian, integritas kriptografis image belum dinyatakan terverifikasi dalam laporan ini.

---

## 7. Kesimpulan

Berdasarkan analisis image Windows dari Kelompok 3 menggunakan Autopsy 4.23.1, ditemukan tiga artefak utama yang sesuai dengan kategori tugas, yaitu file terhapus, file tersembunyi, dan file yang menunjukkan indikasi modifikasi.

File `financial_projection_2026.xlsx` ditemukan dalam kategori Deleted Files dan konten hasil ekstraksinya memuat flag pertama. File `backup_credential.dat` memuat flag kedua dan informasi yang menyatakan atribut Hidden dan System. File `access_control_policy.cfg` memuat flag ketiga serta konfigurasi yang menunjukkan indikasi bypass kontrol kredensial dan aktivitas eksfiltrasi.

Artefak `ConsoleHost_history.txt` dan `Security_Audit_Incident.log` menghubungkan aktivitas tersebut dengan akun `hendra_admin` dan SID `S-1-5-21-3829104812-1928374615-1002`. Kronologi yang tercantum menunjukkan penghapusan file pada pukul 11:15:42 WIB, perubahan atribut pada pukul 11:22:15 WIB, serta akses tulis ke konfigurasi keamanan pada pukul 11:30:08 WIB, seluruhnya pada 12 Oktober 2026.

Temuan ini mendukung skenario insiden yang diberikan oleh Kelompok 3. Namun, kesimpulan akhir tetap mempertimbangkan keterbatasan validasi hash image, pemeriksaan atribut filesystem secara independen, dan tidak tersedianya Windows Event Log asli berformat EVTX.

## 8. Lampiran

Lampiran yang disertakan dalam dokumentasi ini meliputi:

1. Screenshot proses penambahan image ke Autopsy.
2. Screenshot pemeriksaan dan ekstraksi file terhapus.
3. Screenshot hasil ekstraksi `financial_projection_2026.xlsx`.
4. Screenshot struktur direktori dan folder `System_Security`.
5. Screenshot konten `backup_credential.dat`.
6. Screenshot `ConsoleHost_history.txt`.
7. Screenshot `Security_Audit_Incident.log`.
8. Screenshot konten `access_control_policy.cfg` jika tersedia.
9. Screenshot metadata file yang relevan.
10. Hasil perhitungan dan perbandingan hash image jika telah dilakukan.
