# Algoritma Aplikasi Parkir Transmart

Dokumen ini berisi algoritma (pseudocode) dari proses-proses inti di aplikasi, sebagai pasangan dari `flowchart_lengkap_parkir_transmart.png`. Ditulis per proses supaya mudah dicocokkan dengan kode aslinya.

---

## 1. Login & Redirect Otomatis Sesuai Role

**File:** `login.php`

```
MULAI
  INPUT username, password
  AMBIL data_user DARI tb_user WHERE username = username

  JIKA data_user TIDAK ADA MAKA
    TAMPILKAN "Username atau password salah"
    KEMBALI ke form login
  SELESAI JIKA

  JIKA verifikasi_password(password, data_user.password_hash) SALAH MAKA
    TAMPILKAN "Username atau password salah"
    KEMBALI ke form login
  SELESAI JIKA

  JIKA data_user.status_aktif SAMA DENGAN 0 MAKA
    TAMPILKAN "Akun belum diaktifkan Admin"
    KEMBALI ke form login
  SELESAI JIKA

  SIMPAN session user_id, role SESUAI data_user
  CATAT log aktivitas "login"

  PILIH KASUS data_user.role:
    KASUS "admin"   : ARAHKAN KE dashboard_admin.php
    KASUS "petugas" : ARAHKAN KE dashboard_petugas.php
    KASUS "owner"   : ARAHKAN KE dashboard_owner.php
    KASUS "user"    : ARAHKAN KE dashboard_user.php
  SELESAI PILIH
SELESAI
```

---

## 2. Registrasi

### 2a. Registrasi Pelanggan (publik)
**File:** `register_user.php`

```
MULAI
  INPUT nama, plat_nomor, jenis_kendaraan, no_hp, password

  JIKA plat_nomor SUDAH ADA DI tb_user MAKA
    TAMPILKAN "Plat nomor sudah terdaftar"
    KEMBALI ke form
  SELESAI JIKA

  hash_password = hash(password)
  SIMPAN akun BARU KE tb_user (role = "user", status_aktif = 1)
  ARAHKAN KE index.php (langsung bisa login)
SELESAI
```

### 2b. Registrasi Staf (tersembunyi)
**File:** `register.php`

```
MULAI
  INPUT nama, username, password, role_dipilih, kode_rahasia

  JIKA kode_rahasia TIDAK SAMA DENGAN "TRANSMARTSIGMA" MAKA
    TAMPILKAN "Kode registrasi salah"
    BERHENTI
  SELESAI JIKA

  SIMPAN akun BARU KE tb_user (role = role_dipilih, status_aktif = 0)
  TAMPILKAN "Akun dibuat, tunggu diaktifkan Admin"
SELESAI
```

---

## 3. Booking Slot Parkir Online

**File:** `booking.php`

```
MULAI
  INPUT id_area, tanggal_booking, jam_booking, plat_nomor, jenis_kendaraan, catatan

  kapasitas   = AMBIL kapasitas_slot DARI tb_area_parkir WHERE id_area = id_area
  jumlah_booking_confirmed = HITUNG baris tb_booking
                             WHERE id_area = id_area
                               AND tanggal_booking = tanggal_booking
                               AND status = "confirmed"

  JIKA jumlah_booking_confirmed >= kapasitas MAKA
    TAMPILKAN "Slot penuh untuk tanggal ini"
    BERHENTI
  SELESAI JIKA

  SIMPAN booking BARU KE tb_booking (status = "confirmed")
  TAMPILKAN "Booking berhasil dan sudah confirmed"
SELESAI
```

---

## 4. Transaksi Kendaraan Masuk

**File:** `transaksi_masuk.php`

```
MULAI
  INPUT plat_nomor, jenis_kendaraan, id_area

  kendaraan = CARI tb_kendaraan WHERE plat_nomor = plat_nomor

  JIKA kendaraan TIDAK DITEMUKAN MAKA
    SIMPAN kendaraan BARU KE tb_kendaraan
  SELESAI JIKA

  id_tarif = CARI id_tarif DARI tb_tarif WHERE jenis_kendaraan = jenis_kendaraan

  SIMPAN transaksi BARU KE tb_transaksi
    (id_kendaraan, id_area, id_tarif, waktu_masuk = SEKARANG, status = "masuk")

  area.terisi = area.terisi + 1
  PERBARUI tb_area_parkir

  CETAK struk_masuk.php (termasuk QR code berisi nomor tiket)
SELESAI
```

---

## 5. Transaksi Kendaraan Keluar (proses paling kompleks)

**File:** `transaksi_keluar.php`, memakai fungsi dari `helper_biaya.php`

### 5a. Mencari kendaraan
```
MULAI
  kata_kunci = INPUT manual (ketik plat/tiket) ATAU hasil SCAN QR struk masuk

  transaksi = CARI tb_transaksi
              JOIN tb_kendaraan, tb_area_parkir, tb_tarif
              WHERE status = "masuk"
                AND (plat_nomor = kata_kunci ATAU id_parkir = kata_kunci)

  JIKA transaksi TIDAK DITEMUKAN MAKA
    TAMPILKAN "Data tidak ditemukan"
    BERHENTI
  SELESAI JIKA
SELESAI
```

### 5b. Menghitung biaya (tarif normal → maksimal → tarif malam)
```
FUNGSI hitung_biaya(transaksi):
  menit        = SEKARANG - transaksi.waktu_masuk (dalam menit)
  durasi_jam   = BULATKAN_KE_ATAS(menit / 60), MINIMAL 1

  // tahap 1: tarif per jam vs batas tarif maksimal
  JIKA transaksi.tarif_maksimal > 0 DAN durasi_jam > transaksi.jam_maksimal MAKA
    biaya_siang = transaksi.tarif_maksimal        // dikunci, tidak numpuk lagi
  LAINNYA
    biaya_siang = durasi_jam * transaksi.tarif_per_jam
  SELESAI JIKA

  // tahap 2: tarif malam / inap
  biaya_inap = hitung_biaya_inap(transaksi.waktu_masuk, SEKARANG,
                                  transaksi.jam_malam, transaksi.tarif_inap)

  biaya_total = biaya_siang + biaya_inap
  KEMBALIKAN (biaya_total, biaya_inap)
AKHIR FUNGSI
```

### 5c. Algoritma `hitung_biaya_inap` (hitung jumlah malam yang terlewati)
```
FUNGSI hitung_biaya_inap(waktu_masuk, waktu_sekarang, jam_malam, tarif_inap):
  JIKA tarif_inap <= 0 MAKA
    KEMBALIKAN 0                      // fitur nonaktif
  SELESAI JIKA

  // titik jam_malam pertama pada TANGGAL yang sama dengan waktu masuk
  batas = TANGGAL(waktu_masuk) + JAM(jam_malam) + ":00:00"

  jumlah_malam = 0
  SELAMA batas <= waktu_sekarang LAKUKAN
    jumlah_malam = jumlah_malam + 1
    batas = batas + 1 HARI
  SELESAI SELAMA

  KEMBALIKAN jumlah_malam * tarif_inap
AKHIR FUNGSI
```
*Contoh: masuk jam 10:00, `jam_malam` = 22, diambil besok jam 07:00 → titik jam 22:00 terlewati 1 kali → kena 1× `tarif_inap`.*

### 5d. Memilih metode bayar & menyelesaikan transaksi
```
MULAI
  TAMPILKAN estimasi biaya (dari hitung_biaya)
  PETUGAS PILIH metode_bayar: "tunai" ATAU "qris"

  JIKA metode_bayar SAMA DENGAN "qris" MAKA
    TAMPILKAN gambar QR (LIHAT bagian 6: buat_qris_dinamis)
    TUNGGU petugas konfirmasi pembayaran diterima (manual)
  SELESAI JIKA

  PETUGAS TEKAN "Proses Keluar"
    (biaya_total, biaya_inap) = hitung_biaya(transaksi)   // dihitung ULANG saat submit

  PERBARUI tb_transaksi SET
    waktu_keluar = SEKARANG, durasi_jam, biaya_total, biaya_inap,
    metode_bayar, status = "keluar"

  area.terisi = area.terisi - 1
  PERBARUI tb_area_parkir

  CETAK struk_keluar.php (rincian biaya + metode bayar)
SELESAI
```

---

## 6. QRIS Dinamis dari QRIS Statis ⚠️ *(fitur belum aktif, lihat README §8)*

**File:** `helper_biaya.php` — fungsi `buat_qris_dinamis`

Konsepnya: QRIS statis toko (nominal bebas diisi pembeli) "disuntik" jadi QRIS dinamis (nominal sudah pasti sesuai tagihan), memakai format TLV (Tag-Length-Value) standar EMVCo.

```
FUNGSI buat_qris_dinamis(qris_statis, nominal):
  string_tanpa_crc = HAPUS 4 karakter CRC terakhir DARI qris_statis
  daftar_tag        = PECAH string_tanpa_crc JADI pasangan (tag, panjang, value)

  UNTUK SETIAP item DALAM daftar_tag LAKUKAN
    JIKA item.tag SAMA DENGAN "01" MAKA
      item.value = "12"                     // 11 = statis, 12 = dinamis
    SELESAI JIKA
  AKHIR UNTUK

  SISIPKAN tag baru "54" (Transaction Amount) = nominal
           TEPAT SETELAH tag "53" (Currency Code)

  string_baru = GABUNGKAN SEMUA tag JADI satu string, tambahkan "6304" di akhir
  crc         = hitung_crc16_qris(string_baru)   // CRC16/CCITT-FFFF

  KEMBALIKAN string_baru + crc
AKHIR FUNGSI
```

```
FUNGSI hitung_crc16_qris(teks):
  crc = 0xFFFF
  UNTUK SETIAP karakter DALAM teks LAKUKAN
    crc = crc XOR (nilai_ascii(karakter) DIGESER KIRI 8 bit)
    ULANGI 8 KALI:
      JIKA bit_tertinggi(crc) SAMA DENGAN 1 MAKA
        crc = (crc DIGESER KIRI 1) XOR 0x1021
      LAINNYA
        crc = crc DIGESER KIRI 1
      SELESAI JIKA
  AKHIR UNTUK
  KEMBALIKAN crc DALAM HEKSADESIMAL, 4 DIGIT, HURUF BESAR
AKHIR FUNGSI
```

Gambar QR akhirnya dibuat dengan mengirim hasil `buat_qris_dinamis(...)` ke API generator QR (`api.qrserver.com`), sama seperti QR tiket di struk masuk.

---

## 7. Kelola Data Master (Admin)

**File:** `kelola_user.php`, `kelola_area.php`, `kelola_kendaraan.php`, `kelola_tarif.php` — pola algoritma yang sama (CRUD standar):

```
MULAI
  JIKA aksi SAMA DENGAN "tambah" ATAU "ubah" MAKA
    VALIDASI input (wajib isi, format benar, dsb.)
    JIKA valid MAKA
      SIMPAN / PERBARUI data KE tabel terkait
      CATAT log aktivitas
    LAINNYA
      TAMPILKAN pesan error
    SELESAI JIKA
  SELESAI JIKA

  JIKA aksi SAMA DENGAN "hapus" MAKA
    HAPUS data DARI tabel terkait WHERE id = id_dipilih
    CATAT log aktivitas
  SELESAI JIKA

  TAMPILKAN daftar data terbaru
SELESAI
```

---

## 8. Rekap & Laporan (Owner)

**File:** `rekap_transaksi.php`, `laporan.php`

```
MULAI
  INPUT tanggal_mulai, tanggal_selesai

  daftar_transaksi = AMBIL tb_transaksi
                      WHERE status = "keluar"
                        AND waktu_keluar ANTARA tanggal_mulai DAN tanggal_selesai

  total_pendapatan = JUMLAHKAN biaya_total DARI daftar_transaksi
  total_kendaraan  = HITUNG baris daftar_transaksi

  TAMPILKAN tabel daftar_transaksi, total_pendapatan, total_kendaraan
SELESAI
```
