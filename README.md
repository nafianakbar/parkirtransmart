# parkirtransmart

## Aplikasi Parkir - Transmart Yogyakarta

Aplikasi manajemen parkir untuk Transmart Yogyakarta, dibangun dengan **PHP native + MySQL (PDO)** dan tampilan **Tailwind CSS**, mengikuti alur kerja parkir dasar (masuk/keluar/struk) yang dilengkapi fitur booking online oleh pelanggan, serta landing page publik dengan video demo dan ulasan pengunjung.

Selain alur parkir dasar, aplikasi ini juga dilengkapi fitur **booking slot parkir online** oleh pelanggan (role *user*), **registrasi staf tersembunyi** dengan kode rahasia dan status persetujuan admin, serta **landing page publik** dengan video demo dan form ulasan/rating pengunjung.

**Live demo:** `appparkir.infinityfreeapp.com/parkirtransmart/landing.php`

---

## 1. Struktur Folder

```
parkirtransmart/
├── assets/                     -> aset landing page (poster, video demo)
│   ├── demo-aplikasi.mp4
│   └── demo-aplikasi-poster.jpg
├── includes/                   -> komponen bersama
│   ├── header.php
│   ├── footer.php
│   └── sidebar.php
├── sql/
│   └── booking_migration.sql    (migrasi tabel & kolom untuk fitur booking)
├── config.php                   (koneksi PDO + helper get_pdo, catat_log, json_response)
├── auth_guard.php               (helper require_role / proteksi akses per role)
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
├── kelola_tarif.php             (CRUD tarif parkir per jenis kendaraan)
├── log_aktivitas.php            (akses log aktivitas seluruh user, khusus Admin)
├── booking.php                  (booking slot parkir online oleh pelanggan)
├── booking_saya.php             (riwayat & pembatalan booking milik pelanggan)
├── lihat_slot.php               (cek sisa slot area parkir per tanggal)
├── transaksi_masuk.php          (input kendaraan masuk, termasuk dari booking)
├── transaksi_keluar.php         (proses kendaraan keluar & hitung biaya per jam)
├── struk_masuk.php              (cetak struk saat kendaraan masuk)
├── struk_keluar.php             (cetak struk & bukti pembayaran saat keluar)
├── rekap_transaksi.php          (rekap transaksi milik Owner)
└── laporan.php                  (laporan pendapatan & statistik untuk Owner)
```

> Catatan: seluruh halaman berada langsung di folder root project (bukan dipisah per subfolder role), pembatasan akses tiap role dijaga lewat `auth_guard.php` (`require_role([...])`) di baris awal setiap file.

---

## 2. Instalasi Lokal (XAMPP / Laragon)

1. Copy folder `parkirtransmart` ke dalam `htdocs` (XAMPP) atau `www` (Laragon).
2. Buka phpMyAdmin, buat database baru, lalu jalankan skema awal (tabel `tb_user`, `tb_kendaraan`, `tb_tarif`, `tb_area_parkir`, `tb_transaksi`, `tb_log_aktivitas`), kemudian import `sql/booking_migration.sql` untuk menambahkan role `user` dan tabel `tb_booking`.
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
2. Buat database MySQL baru lewat panel InfinityFree (nama database & user otomatis diberi prefix, misal `if0_xxxxxxx_db_parkir_transmart`), lalu import skema + `booking_migration.sql` lewat phpMyAdmin bawaan InfinityFree.
3. Sesuaikan `config.php` dengan kredensial yang diberikan InfinityFree:
   ```php
   define('DB_HOST', 'sqlXXX.infinityfree.com');
   define('DB_NAME', 'if0_xxxxxxx_db_parkir_transmart');
   define('DB_USER', 'if0_xxxxxxx');
   define('DB_PASS', '••••••••');
   ```
4. **Perlu diperbaiki sebelum production:** `config.php` saat ini masih mengaktifkan `display_errors` (komentar di kode menandainya sebagai *"DEBUG SEMENTARA — WAJIB dihapus"*), sehingga detail error PDO (host/kredensial) bisa terlihat publik jika koneksi gagal. Matikan `display_errors` dan ganti pesan error koneksi jadi pesan generik sebelum live.

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
| CRUD Tarif Parkir | ✔ | | | |
| CRUD Area Parkir (kapasitas slot) | ✔ | | | |
| CRUD Kendaraan | ✔ | | | |
| Akses Log Aktivitas | ✔ | | | |
| Booking slot parkir online | | | | ✔ |
| Cek sisa slot & riwayat booking sendiri | | | | ✔ |
| Kendaraan Masuk (Transaksi) | | ✔ | | |
| Kendaraan Keluar + Hitung Biaya | | ✔ | | |
| Cetak Struk Masuk / Keluar | | ✔ | | |
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
| `tb_tarif` | Master tarif parkir per jenis kendaraan (per jam) |
| `tb_booking` | Booking slot parkir oleh pelanggan (status: `confirmed`, `dibatalkan`, `selesai`, `kadaluarsa`); plat nomor & jenis kendaraan disimpan sebagai snapshot, bukan FK ke `tb_kendaraan` |
| `tb_transaksi` | Transaksi parkir masuk/keluar, waktu masuk/keluar, durasi, biaya total |
| `tb_log_aktivitas` | Log aktivitas seluruh user (login, CRUD, dsb.) |
| `tb_ulasan` | Ulasan & rating (1-5 bintang) publik dari landing page, tabel dibuat otomatis (`CREATE TABLE IF NOT EXISTS`) saat landing page pertama kali diakses |

Relasi antar tabel dijaga dengan **FOREIGN KEY** (`tb_booking` ↔ `tb_user` / `tb_area_parkir`).

---

## 7. Alur Kerja Aplikasi

1. Pengunjung publik membuka `landing.php`: melihat profil layanan, video demo aplikasi, dan bisa mengirim ulasan/rating baru (langsung tampil, dilindungi honeypot anti-bot, tanpa moderasi admin).
2. Pelanggan mendaftar mandiri lewat `register_user.php` (nama, plat nomor, jenis kendaraan, no HP, password), akun langsung aktif dan bisa login.
3. Pelanggan login ke `dashboard_user.php`, mengecek sisa slot (`lihat_slot.php`), lalu melakukan booking (`booking.php`) — sistem mengecek kapasitas area secara real-time sebelum booking disimpan. Pelanggan dapat memantau/membatalkan booking di `booking_saya.php`.
4. Saat kendaraan (baik dari booking maupun walk-in) masuk, Petugas mencatatnya di `transaksi_masuk.php` — kendaraan baru otomatis terdaftar jika plat belum ada di database — slot area berkurang otomatis & struk masuk tercetak (`struk_masuk.php`).
5. Saat kendaraan keluar, Petugas memprosesnya di `transaksi_keluar.php` — sistem menghitung durasi (dibulatkan ke atas per jam) & biaya berdasarkan tarif per jenis kendaraan, lalu struk pembayaran dicetak (`struk_keluar.php`) dan slot area bertambah kembali.
6. Admin mengelola seluruh master data (user, tarif, area, kendaraan) dan memantau log aktivitas seluruh pengguna, termasuk menyetujui/mengaktifkan akun staf baru yang mendaftar lewat `register.php`.
7. Owner memantau rekap transaksi (`rekap_transaksi.php`) dan laporan pendapatan/statistik (`laporan.php`) pada dashboard-nya.

---

## 8. Hal yang Perlu Diperhatikan

- Halaman `register.php` untuk staf tidak ditautkan dari navigasi manapun (hidden by design) — pastikan kode rahasia `TRANSMARTSIGMA` tidak bocor keluar tim, dan pertimbangkan memindahkannya ke konfigurasi/env alih-alih hardcode di kode.
- `config.php` masih dalam mode debug (`display_errors` aktif) — wajib dimatikan sebelum benar-benar digunakan publik di production.
- Belum ditemukan file skema SQL awal (`.sql` lengkap) di project, yang tersedia hanya `sql/booking_migration.sql`. Disarankan mengekspor skema awal (`tb_user`, `tb_kendaraan`, `tb_tarif`, `tb_area_parkir`, `tb_transaksi`, `tb_log_aktivitas`) dari database yang sudah berjalan supaya instalasi ulang lebih mudah.
