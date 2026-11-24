# TUGAS-PWEB10

# Spesifikasi Proyek Website: Aplikasi Pendaftaran Siswa Baru

Disusun oleh Alessandro Almaz Filemon (5025251095)

## 1. Nama Proyek

**Aplikasi Pendaftaran Siswa Baru SMK Coding** (repositori: `pendaftaran-siswa-newest`).

## 2. Deskripsi

Website pendaftaran siswa baru berbasis web. Calon siswa mengisi formulir online, datanya langsung tersimpan ke database MySQL di localhost, lalu panitia dapat melihat, mengubah, dan menghapus data pendaftar dari halaman daftar pendaftar. Aplikasi dibangun dengan PHP native (MySQLi) di sisi back end dan HTML, CSS, serta Bootstrap 3 di sisi front end.

## 3. Tujuan

1. Menggantikan pendaftaran manual (kertas) dengan pendaftaran online yang cepat dan rapi.
2. Menyimpan data calon siswa secara terpusat di database sehingga mudah dicari dan dikelola.
3. Menerapkan operasi CRUD (Create, Read, Update, Delete) pada PHP dan MySQL.
4. Menjadi dasar pengembangan fitur lanjutan: login admin, keamanan lebih lanjut, dan pembayaran biaya pendaftaran lewat payment gateway.

## 4. Fitur

### 4.1 Fitur yang sudah ada (diimplementasikan)

| Fitur | Berkas | Keterangan |
|---|---|---|
| Halaman utama | `index.php` | Menu menuju formulir dan daftar pendaftar, menampilkan notifikasi sukses atau gagal |
| Formulir pendaftaran | `form-daftar.php`, `proses-pendaftaran.php` | Input nama, alamat, jenis kelamin, agama, dan sekolah asal (Create) |
| Daftar pendaftar | `list-siswa.php` | Menampilkan seluruh calon siswa dalam tabel beserta jumlahnya (Read) |
| Edit data | `form-edit.php`, `proses-edit.php` | Mengubah data siswa (Update) |
| Hapus data | `hapus.php` | Menghapus data siswa lewat metode POST dengan konfirmasi (Delete) |
| Koneksi database | `config.php` | Koneksi MySQLi ke `localhost`, database `pendaftaran_siswa_newest` |
| Template tampilan | `header.php`, `footer.php`, `css/style.css` | Header, footer, dan gaya yang dipakai semua halaman, responsif |

### 4.2 Fitur yang direncanakan

- Login admin dan pembatasan akses halaman pendaftar.
- Pembayaran biaya pendaftaran melalui payment gateway.
- Pencarian, filter, dan ekspor data pendaftar.

## 5. Arsitektur

### 5.1 Front End

- **Teknologi:** HTML5, CSS3 (`css/style.css`), Bootstrap 3.4.1, Google Fonts (Poppins dan Fredoka), JavaScript dasar untuk menu mobile.
- **Halaman:** Home, Daftar Baru, Pendaftar, Edit Data.
- **Prinsip tampilan:** responsif (menu menjadi tombol pada layar kecil), form memakai atribut `required` dan `maxlength`, label terhubung ke input untuk aksesibilitas.

### 5.2 Back End

- **Teknologi:** PHP native, MySQLi (prepared statement), MySQL/MariaDB (InnoDB), server lokal XAMPP.
- **Alur kerja:** form (HTML) → skrip proses PHP (validasi) → query ke database → redirect dengan parameter `status` → halaman menampilkan notifikasi.

### 5.3 Struktur Database

Database `pendaftaran_siswa_newest`, tabel `calon_siswa`:

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | INT, AUTO_INCREMENT | Primary key |
| `nama` | VARCHAR(64) | Wajib diisi |
| `alamat` | VARCHAR(255) | Wajib diisi |
| `jenis_kelamin` | VARCHAR(16) | `laki-laki` atau `perempuan` |
| `agama` | VARCHAR(16) | Islam, Kristen, Katolik, Hindu, Buddha, Konghucu |
| `sekolah_asal` | VARCHAR(64) | Wajib diisi |

Tabel tambahan untuk rencana pengembangan: `admin` (id, username, password_hash) dan `pembayaran` (id, siswa_id, order_id, jumlah, status, metode, waktu_bayar).

## 6. Ruang Lingkup Proyek

### 6.1 Pengelolaan Data

- Tambah, tampil, ubah, dan hapus data calon siswa (CRUD) pada tabel `calon_siswa`.
- Data ditampilkan berurutan berdasarkan `id` dengan nomor urut dan total pendaftar.
- Rencana: pencarian berdasarkan nama atau sekolah asal, filter jenis kelamin atau agama, paginasi, dan ekspor ke CSV atau Excel.
- Backup database dilakukan dengan ekspor lewat phpMyAdmin (`database.sql` menjadi skrip pembuatan awal).

### 6.2 Logika Bisnis

- Semua kolom wajib diisi, nilai dipangkas spasi berlebih (`trim`).
- Jenis kelamin dan agama hanya boleh bernilai dari daftar yang sudah ditentukan (whitelist), jika tidak formulir ditolak.
- Jika validasi gagal, pengguna dikembalikan ke form dengan pesan galat. Jika berhasil, muncul pesan sukses.
- Penghapusan hanya lewat POST dan harus dikonfirmasi pengguna.
- Rencana: nomor pendaftaran otomatis, status pendaftaran (menunggu pembayaran, terdaftar, diterima), pencegahan data ganda (nama dan sekolah asal yang sama), dan batas kuota.

### 6.3 Autentikasi dan Security

**Sudah diterapkan:**
- Prepared statement MySQLi pada semua query yang memakai input pengguna (mencegah SQL Injection).
- `htmlspecialchars()` saat menampilkan data (mencegah XSS).
- Casting `(int)` pada `id` dan validasi whitelist pada pilihan agama dan jenis kelamin.
- Penghapusan hanya lewat POST, bukan tautan GET.

**Direncanakan:**
- Login admin dengan `password_hash()` dan `password_verify()`, serta session (`session_start`, `session_regenerate_id`) untuk melindungi `list-siswa.php`, `form-edit.php`, dan `hapus.php`.
- Token CSRF pada seluruh form POST.
- Pemisahan user MySQL khusus aplikasi (bukan `root`) dengan password, dan penyimpanan kredensial di luar repositori publik.
- Batas percobaan login, logout, dan session timeout.
- Memakai HTTPS saat dipublikasikan.

### 6.4 API (Application Programming Interface), Contoh Payment Gateway

Rencana integrasi payment gateway (misalnya Midtrans atau Xendit) untuk biaya pendaftaran:

1. Setelah formulir berhasil disimpan, server membuat transaksi ke API payment gateway (kirim `order_id`, jumlah, dan data siswa), lalu menerima token atau URL pembayaran.
2. Siswa diarahkan ke halaman pembayaran (virtual account, e-wallet, QRIS, dan lainnya).
3. Payment gateway mengirim **notifikasi (webhook)** ke endpoint `notifikasi-pembayaran.php`. Server memverifikasi signature, lalu memperbarui status pada tabel `pembayaran`.
4. Halaman pendaftar menampilkan status pembayaran tiap siswa.

| Endpoint | Metode | Fungsi |
|---|---|---|
| `buat-pembayaran.php` | POST | Membuat transaksi ke payment gateway |
| `notifikasi-pembayaran.php` | POST | Menerima webhook dan memperbarui status |
| `status-pembayaran.php?id=` | GET | Mengecek status pembayaran siswa |

Kunci API (server key) disimpan di berkas konfigurasi yang tidak ikut di-commit ke Github, dan tahap awal memakai mode sandbox.

## 7. Rencana Implementasi

| Tahap | Kegiatan | Hasil |
|---|---|---|
| 1. Analisis dan perancangan | Menentukan kebutuhan, rancangan database, dan tampilan halaman | Spesifikasi proyek dan rancangan tabel |
| 2. Setup lingkungan | Memasang XAMPP, membuat database `pendaftaran_siswa_newest` lewat `database.sql`, mengatur `config.php` | Server lokal berjalan |
| 3. Front end | Membuat `header.php`, `footer.php`, `style.css`, halaman Home, form daftar, dan tabel pendaftar | Antarmuka responsif |
| 4. Back end dan pengelolaan data | Membuat proses simpan, ubah, dan hapus serta validasi | CRUD berfungsi (selesai) |
| 5. Autentikasi dan security | Tabel `admin`, login, session, CSRF, user database khusus | Halaman admin terlindungi |
| 6. Integrasi payment gateway | Akun sandbox, pembuatan transaksi, webhook, tabel `pembayaran` | Biaya pendaftaran bisa dibayar online |
| 7. Pengujian | Uji fungsi tiap fitur, uji input tidak valid, uji SQL Injection dan XSS, uji tampilan mobile | Daftar bug dan perbaikan |
| 8. Publikasi | Push source code ke Github dan, bila perlu, deploy ke hosting | Repositori Github dan website online |

## 8. Kebutuhan Sistem

- XAMPP (Apache, PHP 7.4 atau lebih baru, MySQL/MariaDB), browser modern, Git, dan text editor seperti VS Code.
- Koneksi internet untuk memuat Bootstrap dan Google Fonts dari CDN.

## 9. Cara Menjalankan

1. Salin folder `pendaftaran-siswa-newest` ke `C:\xampp\htdocs\`.
2. Nyalakan Apache dan MySQL di XAMPP.
3. Buka phpMyAdmin (`localhost/phpmyadmin`), lalu impor `database.sql` (atau buat database `pendaftaran_siswa_newest` dan jalankan isi berkasnya).
4. Buka `http://localhost/pendaftaran-siswa-newest/` di browser.
