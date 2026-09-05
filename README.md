# parkirtransmart

## Aplikasi Parkir - Transmart Yogyakarta

Aplikasi manajemen parkir untuk Transmart Yogyakarta, dibangun dengan **PHP native + MySQL (PDO)** dan tampilan **Tailwind CSS**, mengikuti alur kerja parkir dasar (masuk/keluar/struk) yang dilengkapi fitur booking online oleh pelanggan, landing page publik dengan video demo dan ulasan pengunjung, serta perhitungan tarif progresif (batas tarif maksimal & tarif malam/inap) dan pencarian kendaraan lewat scan QR struk.

**Live demo:** `appparkir.infinityfreeapp.com/parkirtransmart/landing.php`
**Flowchart:** [`docs/flowchart_lengkap_parkir_transmart.png`](./docs/flowchart_lengkap_parkir_transmart.png)
**Mockup:** [`mockup_parkmanager.png`](./docs/mockup_parkmanager.png)
**Algoritma:** [`ALGORITMA.md`](./ALGORITMA.md)

---

## 0. Dokumentasi Terkait

| Dokumen | Isi | Lokasi |
|---|---|---|
| 📊 **Flowchart** | Alur sistem lengkap dari landing page sampai semua dashboard per role, termasuk logika tarif maksimal/malam & status fitur QRIS | `docs/flowchart_lengkap_parkir_transmart.png` |
| 🎨 **Mockup** | Low-fidelity wireframe (branding "ParkManager") — struktur halaman landing page & dashboard tiap role sebelum masuk desain visual final | `docs/mockup_parkmanager.png` |
| 🧮 **Algoritma** | Pseudocode tiap proses inti (login, booking, transaksi masuk/keluar, hitung tarif maksimal + tarif malam, QRIS dinamis, dsb.) | `ALGORITMA.md` |

> Taruh `flowchart_lengkap_parkir_transmart.png` dan `mockup_parkmanager.png` di folder `docs/` pada root project (buat foldernya kalau belum ada) supaya link di atas langsung nyambung. Kalau naruhnya di folder lain, sesuaikan path link-nya.

---

## 1. Struktur Folder

```
parkirtransmart/
├── docs/                        -> dokumentasi non-kode
│   ├── flowchart_lengkap_parkir_transmart.png
│   └── mockup_parkmanager.png
├── ALGORITMA.md                 -> pseudocode tiap proses inti
├── assets/                     -> aset landing page (poster, video demo)
│   ├── demo-aplikasi.mp4
│   └── demo-aplikasi-poster.jpg
├── includes/                   -> komponen bersama
│   ├── header.php
│   ├── footer.php
│   └── sidebar.php
├── sql/
│   ├── booking_migration.sql        (migrasi tabel & kolom untuk fitur booking)
│   ├── tarif_maksimal_migration.sql (migrasi kolom batas tarif maksimal)
│   ├── tarif_malam_migration.sql    (migrasi kolom tarif malam/inap)
│   └── qris_migration.sql           (migrasi tabel pengaturan & metode bayar)
├── config.php                   (koneksi PDO + helper get_pdo, catat_log, json_response)
├── auth_guard.php               (helper require_role / proteksi akses per role)
├── helper_biaya.php             (hitung biaya inap + konversi QRIS statis ke dinamis)
├── landing.php                  -> landing page publik (profil, video demo, form ulasan)
├── index.php                    -> halaman login (satu form untuk semua role)
├── login.php                    (proses autentikasi + redirect otomatis sesuai role)
├── logout.php
├── register_user.php            (registrasi mandiri publik -> hanya role **user**/pelanggan)
├── register.php                 (registrasi staf tersembunyi, butuh kode rahasia, non-aktif
│                                  sampai disetujui Admin — tidak ditautkan dari menu manapun)
├── dashboard_admin.php          -> Dashboard & ringkasan untuk role ADMIN
├── dashboard_petugas.php        -> Dashboard untuk role PETUGAS
├── dashboard_owner.php          -> Dashboard grafik pendapatan untuk role OWNER
├── dashboard_user.php           -> Dashboard untuk role USER (pelanggan)
├── kelola_user.php              (CRUD user: admin/petugas/owner/user)
├── kelola_area.php              (CRUD area parkir & kapasitas slot)
├── kelola_kendaraan.php         (CRUD data kendaraan)
├── kelola_tarif.php             (CRUD tarif: per jam, batas tarif maksimal, tarif malam/inap)
├── pengaturan_qris.php          (Admin: simpan QRIS statis toko) ⚠️ belum ada di server, lihat §8
├── log_aktivitas.php            (akses log aktivitas seluruh user, khusus Admin)
├── booking.php                  (booking slot parkir online oleh pelanggan)
├── booking_saya.php             (riwayat & pembatalan booking milik pelanggan)
├── lihat_slot.php                (cek sisa slot area parkir per tanggal)
├── transaksi_masuk.php          (input kendaraan masuk, termasuk dari booking)
├── transaksi_keluar.php         (proses kendaraan keluar: cari via ketik atau scan QR,
│                                  hitung tarif maksimal + tarif malam, pilih metode bayar)
├── qris_generate.php            (endpoint gambar QRIS dinamis, nominal otomatis) ⚠️ lihat §8
├── struk_masuk.php              (cetak struk saat kendaraan masuk, termasuk QR code tiket)
├── struk_keluar.php             (cetak struk & bukti pembayaran saat keluar, rincian biaya)
├── rekap_transaksi.php          (rekap transaksi milik Owner)
└── laporan.php                  (laporan pendapatan & statistik untuk Owner)
```

> Catatan: seluruh halaman berada langsung di folder root project (bukan dipisah per subfolder role), pembatasan akses tiap role dijaga lewat `auth_guard.php` (`require_role([...])`) di baris awal setiap file.

---

## 2. Instalasi Lokal (XAMPP / Laragon)

1. Copy folder `parkirtransmart` ke dalam `htdocs` (XAMPP) atau `www` (Laragon).
2. Buka phpMyAdmin, buat database baru, lalu jalankan skema awal (tabel `tb_user`, `tb_kendaraan`, `tb_tarif`, `tb_area_parkir`, `tb_transaksi`, `tb_log_aktivitas`), kemudian import berurutan:
   1. `sql/booking_migration.sql` — role `user` dan tabel `tb_booking`
   2. `sql/tarif_maksimal_migration.sql` — kolom `jam_maksimal`, `tarif_maksimal` di `tb_tarif`
   3. `sql/tarif_malam_migration.sql` — kolom `jam_malam`, `tarif_inap` di `tb_tarif`, kolom `biaya_inap` di `tb_transaksi`
   4. `sql/qris_migration.sql` — tabel `tb_pengaturan` dan kolom `metode_bayar` di `tb_transaksi`
3. Buka `config.php`, sesuaikan bila perlu:
   ```php
   define('DB_HOST', 'localhost');
   define('DB_NAME', 'db_parkir_transmart');
   define('DB_USER', 'root');
   define('DB_PASS', '');
   ```
4. Akses aplikasi melalui `http://localhost/parkirtransmart/landing.php`.

---

## 3. Deploy ke Hosting (InfinityFree)

Project ini live di InfinityFree dengan domain `appparkir.infinityfreeapp.com`. Ringkasan langkah deploy-nya:

1. Upload seluruh isi folder `parkirtransmart` (bukan foldernya) langsung ke folder `htdocs` lewat File Manager atau FTP InfinityFree.
2. Buat database MySQL baru lewat panel InfinityFree (nama database & user otomatis diberi prefix, misal `if0_xxxxxxx_db_parkir_transmart`), lalu import skema awal + keempat file migrasi (urutan di atas) lewat phpMyAdmin bawaan InfinityFree.
3. Sesuaikan `config.php` dengan kredensial yang diberikan InfinityFree:
   ```php
   define('DB_HOST', 'sqlXXX.infinityfree.com');
   define('DB_NAME', 'if0_xxxxxxx_db_parkir_transmart');
   define('DB_USER', 'if0_xxxxxxx');
   define('DB_PASS', '••••••••');
   ```
4. Scan QR (struk & QRIS) butuh akses kamera browser, yang **hanya jalan lewat HTTPS** (kecuali di `localhost`). Pastikan akses situs pakai `https://`, bukan `http://`.
5. **Perlu diperbaiki sebelum production:** `config.php` saat ini masih mengaktifkan `display_errors` (komentar di kode menandainya sebagai *"DEBUG SEMENTARA — WAJIB dihapus"*), sehingga detail error PDO (host/kredensial) bisa terlihat publik jika koneksi gagal. Matikan `display_errors` dan ganti pesan error koneksi jadi pesan generik sebelum live.

---

## 4. Akun & Pendaftaran

| Role | Cara mendapatkan akun | Keterangan |
|---|---|---|
| **User (pelanggan)** | Daftar mandiri via `register_user.php` (isi nama, plat nomor, jenis kendaraan, no HP, password) | Akun langsung aktif, bisa langsung login & booking |
| **Petugas** | Registrasi tersembunyi via `register.php` (tidak ada link ke halaman ini) | Wajib kode rahasia `TRANSMARTSIGMA`, akun **non-aktif** sampai diaktifkan Admin lewat Kelola User |
| **Admin** | Sama seperti Petugas | Sama seperti Petugas |
| **Owner** | Sama seperti Petugas | Sama seperti Petugas |

Login memakai satu form yang sama untuk semua role (`index.php` → `login.php`) — sistem membaca kolom `role` di database dan mengarahkan otomatis ke dashboard yang sesuai, pengguna tidak memilih role secara manual saat login.

---

## 5. Hak Akses Fitur

| Fitur | Admin | Petugas | Owner | User |
|---|---|---|---|---|
| Login / Logout | ✔ | ✔ | ✔ | ✔ |
| Registrasi mandiri (publik) | | | | ✔ |
| Registrasi tersembunyi (kode rahasia, perlu approval) | ✔ | ✔ | ✔ | |
| CRUD User | ✔ | | | |
| CRUD Tarif Parkir (per jam, batas maksimal, tarif malam) | ✔ | | | |
| Pengaturan QRIS toko ⚠️ *(belum aktif, lihat §8)* | ✔ | | | |
| CRUD Area Parkir (kapasitas slot) | ✔ | | | |
| CRUD Kendaraan | ✔ | | | |
| Akses Log Aktivitas | ✔ | | | |
| Booking slot parkir online (otomatis confirmed) | | | | ✔ |
| Cek sisa slot & riwayat booking sendiri | | | | ✔ |
| Kendaraan Masuk (Transaksi) | | ✔ | | |
| Kendaraan Keluar — cari via ketik plat/tiket **atau scan QR struk** | | ✔ | | |
| Hitung biaya otomatis (tarif normal → maksimal → tarif malam) | | ✔ | | |
| Pilih metode bayar Tunai / QRIS ⚠️ *(QRIS belum aktif)* | | ✔ | | |
| Cetak Struk Masuk (dengan QR code tiket) / Struk Keluar | | ✔ | | |
| Rekap Transaksi | | | ✔ | |
| Laporan pendapatan & statistik | | | ✔ | |
| Kirim ulasan & rating (landing page) | publik / semua pengunjung situs | | | |

---

## 6. Skema Database

Tabel utama:

| Tabel | Fungsi |
|---|---|
| `tb_user` | Data akun (role: `admin`, `petugas`, `owner`, `user`), `status_aktif` untuk approval staf |
| `tb_kendaraan` | Data kendaraan, terhubung ke pemilik |
| `tb_area_parkir` | Area/zona parkir beserta kapasitas & jumlah terisi |
| `tb_tarif` | Tarif per jenis kendaraan: `tarif_per_jam`, `jam_maksimal` + `tarif_maksimal` (batas flat), `jam_malam` + `tarif_inap` (biaya tambahan per malam, 0 = nonaktif) |
| `tb_booking` | Booking slot parkir oleh pelanggan (status: `confirmed`, `dibatalkan`, `selesai`, `kadaluarsa`, otomatis `confirmed` jika slot masih tersedia); plat nomor & jenis kendaraan disimpan sebagai snapshot, bukan FK ke `tb_kendaraan` |
| `tb_transaksi` | Transaksi parkir masuk/keluar: waktu, durasi, `biaya_total`, `biaya_inap` (rincian bagian tarif malam), `metode_bayar` (`tunai`/`qris`) |
| `tb_pengaturan` | Key-value pengaturan aplikasi; saat ini dipakai untuk menyimpan `qris_statis` (string mentah QRIS toko) |
| `tb_log_aktivitas` | Log aktivitas seluruh user (login, CRUD, dsb.) |
| `tb_ulasan` | Ulasan & rating (1-5 bintang) publik dari landing page, tabel dibuat otomatis (`CREATE TABLE IF NOT EXISTS`) saat landing page pertama kali diakses |

Relasi antar tabel dijaga dengan **FOREIGN KEY** (`tb_booking` ↔ `tb_user` / `tb_area_parkir`).

---

## 7. Alur Kerja Aplikasi

1. Pengunjung publik membuka `landing.php`: melihat profil layanan, video demo aplikasi, dan bisa mengirim ulasan/rating baru (langsung tampil, dilindungi honeypot anti-bot, tanpa moderasi admin).
2. Pelanggan mendaftar mandiri lewat `register_user.php` (nama, plat nomor, jenis kendaraan, no HP, password), akun langsung aktif dan bisa login.
3. Pelanggan login ke `dashboard_user.php`, mengecek sisa slot (`lihat_slot.php`), lalu melakukan booking (`booking.php`) — sistem mengecek kapasitas area secara real-time dan booking langsung berstatus `confirmed` jika slot tersedia. Pelanggan dapat memantau/membatalkan booking di `booking_saya.php`.
4. Saat kendaraan (baik dari booking maupun walk-in) masuk, Petugas mencatatnya di `transaksi_masuk.php` — kendaraan baru otomatis terdaftar jika plat belum ada di database — slot area berkurang otomatis & struk masuk tercetak (`struk_masuk.php`) lengkap dengan **QR code tiket**.
5. Saat kendaraan keluar, Petugas membuka `transaksi_keluar.php` dan mencari kendaraan dengan **mengetik plat/tiket, atau memindai QR code** di struk masuk lewat kamera. Sistem menghitung biaya bertahap:
   - Jam ke-1 s/d batas tertentu dihitung per jam normal.
   - Lewat batas itu, biaya **berhenti bertambah** dan dikunci ke tarif maksimal (flat).
   - Kalau kendaraan masih ada sampai lewat jam malam tertentu (mis. 22:00), ditambahkan **biaya inap** flat di atasnya — nginap 2 malam kena 2× biaya inap.
   - Petugas memilih metode bayar (Tunai / QRIS ⚠️ *lihat §8*), lalu struk keluar dicetak (`struk_keluar.php`) dengan rincian biaya inap dan metode bayar, dan slot area bertambah kembali.
6. Admin mengelola seluruh master data (user, tarif — termasuk batas maksimal & tarif malam, area, kendaraan) dan memantau log aktivitas seluruh pengguna, termasuk menyetujui/mengaktifkan akun staf baru yang mendaftar lewat `register.php`.
7. Owner memantau rekap transaksi (`rekap_transaksi.php`) dan laporan pendapatan/statistik (`laporan.php`) pada dashboard-nya.

---

## 8. Hal yang Perlu Diperhatikan

- **Fitur QRIS belum lengkap/belum aktif.** `qris_generate.php` butuh `helper_biaya.php` untuk jalan, dan Admin butuh `pengaturan_qris.php` untuk menyimpan QRIS statis toko (`tb_pengaturan.qris_statis`) — pastikan kedua file ini benar-benar ter-upload ke server, dan `transaksi_keluar.php` yang dipakai adalah versi yang sudah punya pilihan metode bayar Tunai/QRIS. Kalau salah satu file itu hilang, tombol/link menuju `qris_generate.php` akan menampilkan **fatal error**, bukan gambar QR.
- Nominal QRIS dibuat **dinamis dari QRIS statis toko** (teknik injeksi tag EMVCo tag 54 + hitung ulang CRC), bukan lewat payment gateway resmi — artinya konfirmasi "pembayaran sudah masuk" masih **manual** oleh Petugas (cek notifikasi HP/EDC), belum otomatis dari sistem.
- Halaman `register.php` untuk staf tidak ditautkan dari navigasi manapun (hidden by design) — pastikan kode rahasia `TRANSMARTSIGMA` tidak bocor keluar tim, dan pertimbangkan memindahkannya ke konfigurasi/env alih-alih hardcode di kode.
- `config.php` masih dalam mode debug (`display_errors` aktif) — wajib dimatikan sebelum benar-benar digunakan publik di production.
- Fitur scan QR (baik di struk masuk maupun QRIS) butuh **HTTPS** dan izin kamera browser — kalau kamera gagal diakses padahal sudah HTTPS, cek izin kamera per-situs di browser dan di level OS, serta pastikan link dibuka lewat browser asli (bukan in-app browser WhatsApp/Instagram).
- Belum ditemukan file skema SQL awal (`.sql` lengkap) di project, yang tersedia hanya file-file migrasi di folder `sql/`. Disarankan mengekspor skema awal (`tb_user`, `tb_kendaraan`, `tb_tarif`, `tb_area_parkir`, `tb_transaksi`, `tb_log_aktivitas`) dari database yang sudah berjalan supaya instalasi ulang lebih mudah.
