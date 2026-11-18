# Lab8Web - Praktikum 8: PHP dan Database MySQL

**Nama:** Astuti

**NIM:** 312510320

**Kelas:** I252A

**Mata Kuliah:** Pemrograman Web OBE

## Tujuan
1. Memahami konsep dasar database.
2. Memahami konsep dasar CRUD menggunakan PHP.
3. Membuat program CRUD sederhana menggunakan PHP dan MySQL.

## Struktur Project
```
Lab8Web/
├── gambar/          (folder hasil upload gambar)
├── screenshots/     (screenshot laporan)
├── database.sql
├── koneksi.php
├── index.php
├── tambah.php
├── ubah.php
├── hapus.php
└── style.css
```

---

## 1. Persiapan: Menjalankan Apache dan MySQL
Web server Apache dan database MySQL dijalankan lewat **XAMPP Control Panel** dengan menekan tombol **Start** pada modul Apache dan MySQL. Keduanya berstatus *running* (berwarna hijau), Apache memakai port 80 dan 443, MySQL memakai port 3306.

<img width="660" height="412" alt="image" src="https://github.com/user-attachments/assets/06e8cd3a-1fcc-4a2b-ab2e-68929620d5e0" />

## 2. Membuat Database dan Tabel
Pada phpMyAdmin (`http://localhost/phpmyadmin/`) dibuat database `latihan1`, kemudian tabel `data_barang` lewat tab **SQL**.

```sql
CREATE DATABASE latihan1;

CREATE TABLE data_barang (
  id_barang int(10) auto_increment PRIMARY KEY,
  kategori varchar(30),
  nama varchar(50),
  gambar varchar(100),
  harga_beli decimal(10,0),
  harga_jual decimal(10,0),
  stok int(4)
);
```

Penjelasan: `id_barang` adalah *primary key* yang bertambah otomatis (`auto_increment`). Kolom `harga_beli` dan `harga_jual` bertipe `decimal`, sedangkan `gambar` menyimpan nama file gambar.

![Tabel](screenshots/02-tabel.png)

## 3. Menambahkan Data Awal
```sql
INSERT INTO data_barang (kategori, nama, gambar, harga_beli, harga_jual, stok)
VALUES
('Elektronik', 'HP Samsung Android', 'hp_samsung.jpg', 2000000, 2400000, 5),
('Elektronik', 'HP Xiaomi Android', 'hp_xiaomi.jpg', 1000000, 1400000, 5),
('Elektronik', 'HP OPPO Android', 'hp_oppo.jpg', 1800000, 2300000, 5);
```

Perintah `INSERT INTO` memasukkan 3 baris data barang. Hasilnya dicek lewat tab **Jelajahi** (Browse).

![Data](screenshots/03-data.png)

## 4. Membuat Folder Project
Folder `lab8_php_database` dibuat di dalam `htdocs`, lalu diakses melalui `http://localhost/lab8_php_database/`. Halaman *Index of* tampil karena folder masih kosong.

![Index folder](screenshots/04-index-folder.png)

## 5. File Koneksi Database (`koneksi.php`)
```php
<?php
$host = "localhost";
$user = "root";
$pass = "";
$db   = "latihan1";

$conn = mysqli_connect($host, $user, $pass, $db);
if ($conn == false) {
    echo "Koneksi ke server gagal.";
    die();
} #else echo "Koneksi berhasil";
?>
```

Penjelasan: `mysqli_connect()` menghubungkan PHP ke MySQL memakai host, user, password, dan nama database. Jika gagal, program menampilkan pesan lalu berhenti (`die()`). Untuk pengujian, tanda `#` pada `echo "Koneksi berhasil"` dihapus sementara sehingga pesan sukses tampil.

![Koneksi](screenshots/05-koneksi.png)

## 6. Menampilkan Data / Read (`index.php`)
Penjelasan:
- `include("koneksi.php")` memuat file koneksi.
- `mysqli_query()` menjalankan `SELECT * FROM data_barang`.
- `mysqli_fetch_array()` di dalam `while` mengambil data baris demi baris dan menampilkannya pada tabel HTML.
- Kolom **Aksi** berisi tautan **Ubah** dan **Hapus** yang membawa `id` barang lewat URL.
- Jika tidak ada data, tampil tulisan "Belum ada data".
- `htmlspecialchars()` dipakai agar output aman dari XSS.

![Index](screenshots/06-index.png)

## 7. Menambah Data / Create (`tambah.php`)
Penjelasan:
- Form memakai `method="post"` dan `enctype="multipart/form-data"` agar bisa mengunggah file.
- Saat tombol **Simpan** ditekan, data dibaca dari `$_POST`, dan file dari `$_FILES`.
- File gambar dipindahkan ke folder `gambar/` dengan `move_uploaded_file()`.
- Data disimpan dengan **prepared statement** (`mysqli_prepare`, `mysqli_stmt_bind_param`, `mysqli_stmt_execute`) supaya aman dari SQL Injection.
- Setelah tersimpan, halaman diarahkan ke `index.php` memakai `header()`.

![Form Tambah](screenshots/07-tambah.png)
![Hasil Tambah](screenshots/08-tambah-hasil.png)

## 8. Mengubah Data / Update (`ubah.php`)
Penjelasan:
- Data lama diambil berdasarkan `id` dari URL (`$_GET['id']`) lalu ditampilkan pada form.
- Fungsi `is_select()` menandai opsi kategori yang sesuai dengan data lama.
- `id` dikirim lewat input `hidden`.
- Query `UPDATE` dijalankan dengan prepared statement. Kolom `gambar` hanya diubah jika ada file baru yang diunggah.

![Form Ubah](screenshots/09-ubah.png)
![Hasil Ubah](screenshots/10-ubah-hasil.png)

## 9. Menghapus Data / Delete (`hapus.php`)
Penjelasan: `id` diambil dari URL, lalu query `DELETE FROM data_barang WHERE id_barang = ?` dijalankan. Setelah itu halaman kembali ke `index.php`. Di `index.php` ada konfirmasi `confirm()` sebelum data dihapus.

![Hapus](screenshots/11-hapus.png)

---

## Perbaikan dari Modul
| Bagian | Masalah di modul | Perbaikan |
|---|---|---|
| `index.php` | Urutan kolom harga tertukar, kolom Aksi hanya menampilkan id | Urutan disesuaikan, ditambah link Ubah/Hapus/Tambah |
| `tambah.php`, `ubah.php` | Path gambar tersimpan `gambar/nama.jpg` sehingga path dobel | Hanya nama file yang disimpan |
| `ubah.php` | `enctype="multipart/formdata"` salah ketik | `multipart/form-data` |
| `ubah.php` | `is_select()` selalu membandingkan dengan `'Komputer'` | Dibandingkan dengan nilai opsi masing-masing |
| Semua query | Rentan SQL Injection | Memakai prepared statement |
| `style.css` | Dipanggil tetapi tidak disediakan | Dibuat file `style.css` |

## Kesimpulan
Dengan PHP dan MySQL dapat dibuat aplikasi CRUD (Create, Read, Update, Delete) sederhana untuk mengelola data barang. Koneksi dilakukan dengan `mysqli_connect()`, pengambilan data dengan `mysqli_query()`, dan penyimpanan data aman dengan prepared statement.
