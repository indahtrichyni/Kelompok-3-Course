# Kelompok 3 - Course Service

Layanan data mata kuliah untuk demo arsitektur microservices 7 kelompok.
Kelompok lain tidak perlu mengutak-atik database langsung, cukup memanggil
API di bawah ini.

> **Catatan IP:** Alamat IP LAN berubah setiap ganti Wi-Fi. Di README ini
> ditulis `<IP-LAN>` sebagai placeholder. Ganti `<IP-LAN>` dengan IP Wi-Fi
> laptop Course saat demo (lihat dengan `ipconfig` di Windows, bagian
> *Wireless LAN adapter Wi-Fi → IPv4*). Jangan menulis IP lama yang sudah
> tidak berlaku.

## Teknologi

- PHP + MySQL di XAMPP
- Database: `course_db`
- Tabel: `courses` (`id`, `code`, `name`, `credits`)
- Data awal: PTIK301 Sistem Terdistribusi (3 SKS), PTIK302 Pemrograman Web (3 SKS)

## Struktur Folder

```
course-service/
  index.php       # Kode untuk server port 8103
  db.php          # Koneksi database
  .htaccess       # Routing Apache
  router.php      # Router server built-in PHP
index.php         # Kode untuk Apache di htdocs (versi dengan integrasi Assignment)
db.php            # Salinan koneksi database untuk mode Apache
.htaccess         # Salinan routing untuk mode Apache
router.php        # Salinan router untuk mode Apache
database/
  course_db.sql   # Skema tabel + data awal
postman/
  Kelompok3-Course-Service-v2-Gateway.postman_collection.json
  *.postman_environment.json
jalankan-course-8103-lan.bat
cek-ip-lan.bat
registrasi-course-LAN-8103.json
README-CARA-PAKAI.txt
Uji_Coba_Course_Service_v2_Gateway.html   # Simulator latihan, bukan server asli
```

## Prasyarat

- XAMPP (Apache + MySQL) terinstal
- PHP 8.x (bawaan XAMPP, di `C:\xampp\php\php.exe`)
- Postman (untuk pengujian, opsional)

## Instalasi

1. Start **Apache** dan **MySQL** di XAMPP Control Panel.
2. Buka phpMyAdmin, import `database/course_db.sql`.
   Pastikan database `course_db` dan tabel `courses` berisi PTIK301 dan PTIK302.
3. Pilih salah satu mode menjalankan di bawah ini.

## Cara Menjalankan

### Mode 1 - Apache (uji mandiri di satu laptop)

1. Salin **hanya** `index.php`, `db.php`, dan `.htaccess` dari `course-service/`
   ke `C:\xampp\htdocs\Kelompok3-Course-Service\`.
2. Buka di browser / Postman:
   ```
   http://localhost/Kelompok3-Course-Service/api/courses
   http://localhost/Kelompok3-Course-Service/api/health
   ```

### Mode 2 - Server port 8103 (lintas laptop, mode demo utama)

1. Tutup dulu jendela server 8103 yang lama, karena satu port hanya bisa
   dipakai satu server.
2. Klik dua kali `jalankan-course-8103-lan.bat` (bind `0.0.0.0:8103`).
   Biarkan jendela hitam server tetap terbuka selama demo.
   Izinkan lewat Windows Firewall kalau muncul popup untuk `php.exe`.
3. Kedua laptop (Course dan yang mengaksesnya) harus di **Wi-Fi yang sama**.
4. Buka di browser / Postman:
   ```
   http://127.0.0.1:8103/api/courses        # dari laptop Course sendiri
   http://<IP-LAN>:8103/api/courses         # dari laptop lain di Wi-Fi yang sama
   http://<IP-LAN>:8103/api/health
   ```

## Endpoint

Base URL contoh di bawah memakai mode 8103. Ganti bagian depannya sesuai
mode yang dipakai (`http://localhost/Kelompok3-Course-Service` untuk Apache,
`http://127.0.0.1:8103` untuk satu laptop, `http://<IP-LAN>:8103` untuk
lintas laptop). Bagian belakangnya (`/api/...`) selalu sama.

| Method | Endpoint | Fungsi |
|--------|----------|--------|
| GET | `/api/health` | Cek layanan hidup, untuk Kelompok 7 Monitoring |
| GET | `/api/courses` | Lihat semua mata kuliah |
| GET | `/api/courses/{id}` | Lihat satu mata kuliah, contoh `/api/courses/1` |
| POST | `/api/courses` | Tambah mata kuliah |
| PATCH | `/api/courses/{id}` | Ubah sebagian data mata kuliah |
| DELETE | `/api/courses/{id}` | Hapus mata kuliah, contoh `/api/courses/3` |
| GET | `/api/assignments` | Lihat daftar tugas dari Kelompok 4 |
| GET | `/api/courses/{id}/assignments` | Lihat satu mata kuliah beserta tugasnya |

Tidak ada PUT. Method lain di endpoint di atas akan menjawab `405 Method tidak diizinkan`.

### Contoh Response

**GET /api/health**
```json
{"success": true, "status": "UP", "service": "course-service"}
```

**GET /api/courses**
```json
{
  "success": true,
  "data": [
    {"id": 1, "code": "PTIK301", "name": "Sistem Terdistribusi", "credits": 3},
    {"id": 2, "code": "PTIK302", "name": "Pemrograman Web", "credits": 3}
  ]
}
```

**POST /api/courses**
```json
{"code": "PTIK303", "name": "Basis Data", "credits": 3}
```
Berhasil: `201 Course berhasil ditambahkan`.

**PATCH /api/courses/3**

PATCH hanya mengubah field yang dikirim. Field yang tidak dikirim
dibiarkan tetap seperti semula.

```json
{"name": "Basis Data Lanjut", "credits": 4}
```
Berhasil: `200 Course berhasil diubah`.

**DELETE /api/courses/3**
Berhasil: `200 Course berhasil dihapus`.

### Kode Status

| Kode | Artinya |
|------|---------|
| 200 | Berhasil (GET, PATCH, DELETE, health) |
| 201 | Berhasil ditambahkan (POST) |
| 400 | Body tidak valid / ID tidak valid |
| 404 | Endpoint tidak ditemukan / data tidak ada |
| 405 | Method tidak diizinkan |
| 409 | `code` sudah digunakan course lain |
| 500 | Gagal di server / koneksi database gagal |
| 503 | Assignment Service tidak merespons (untuk endpoint assignments) |

## Cara Tes di Postman

1. Import `postman/Kelompok3-Course-Service-v2-Gateway.postman_collection.json`.
2. Atur variabel `courseUrl` sesuai mode, misalnya `http://127.0.0.1:8103`
   atau `http://<IP-LAN>:8103`.
3. Kirim urut:
   1. GET `/api/courses` → 200, 2 data
   2. POST `/api/courses` dengan PTIK303 → 201
   3. GET `/api/courses` lagi → 200, 3 data
   4. GET `/api/courses/3` → 200, PTIK303 saja
   5. PATCH `/api/courses/3` → 200, data berubah
   6. DELETE `/api/courses/3` → 200
   7. GET `/api/courses` lagi → 200, kembali 2 data
   8. GET `/api/health` → UP

Catatan: Course langsung tidak memakai API key dan tidak memeriksa token.
Token diperiksa di Gateway / Authentication, bukan di Course.

## Integrasi dengan Kelompok Lain

### Kelompok 1 - API Gateway

Daftarkan Course di `routes.php` Gateway, lalu **restart Gateway**:

```php
'course' => 'http://<IP-LAN>:8103',
```

Jangan kirim file ZIP / kode ke Kelompok 1, cukup baris alamat di atas
plus daftar endpoint di README ini. Registry (`/api/registry/course`)
adanya di Gateway, bukan di laptop Course.

### Kelompok 4 - Assignment

- Course memanggil Assignment untuk endpoint `/api/assignments` dan
  `/api/courses/{id}/assignments`.
- Assignment memverifikasi Course sebelum membuat tugas dengan
  `GET /api/courses/{id}`: `200` berarti course ada, `404` berarti
  jangan buat tugasnya.

### Kelompok 7 - Monitoring

Monitoring cukup memanggil:

```
GET http://<IP-LAN>:8103/api/health
```

Hasil yang benar: `200`, `status: UP`.

### Kelompok 2 - Authentication

Course langsung tidak memeriksa token. Kalau diakses lewat Gateway,
token diperiksa di Gateway / Authentication.

## Troubleshooting

| Gejala | Penyebab paling mungkin |
|--------|--------------------------|
| 404 HTML Apache | Salah server: URL tanpa `:8103` dan tanpa nama folder, yang dituju Apache port 80, bukan server Course 8103 |
| 404 JSON `Endpoint tidak ditemukan` | Salah nama endpoint, misalnya `/api/course` tanpa `s`, yang benar `/api/courses` |
| `Request timed out` dari laptop lain | Beda Wi-Fi, server Course belum jalan, atau Firewall belum mengizinkan `php.exe` |
| `ECONNREFUSED` | Server Course / Gateway belum dijalankan |
| `405 Method tidak diizinkan` | Method tidak tersedia di endpoint itu, misalnya PUT |
| `409 Kode course sudah digunakan` | `code` yang di-POST / di-PATCH sudah dipakai course lain |
| `500 Koneksi database gagal` | MySQL belum Start atau `course_db.sql` belum di-import |

## Catatan Keamanan

Jangan menulis di README publik, di kode, atau di chat kelompok:
IP lama yang sudah tidak berlaku, token ngrok, API key Postman, atau
password apa pun. Untuk demo, cukup kirim alamat Course yang sedang
berlaku saat itu.
