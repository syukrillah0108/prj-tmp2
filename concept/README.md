# Entitas

* Admin Koperasi
* Anggota - Pembeli
* Anggota - Penjual

# System Design

### Admin

![Alur Pembuatan Akun](./img/system-design-admin.svg)

# Alur Kerja

### Pembuatan Akun

![Alur Pembuatan Akun](./img/pembuatan_akun.svg)

### Input Produk

![Alur Input Produk](./img/input_produk.svg)

### Jual Beli

![Alur Jual Beli](./img/jual_beli.svg)

### Simpanan

![Alur Simpanan](./img/simpanan.svg)

| Aspek                                  | Simpanan Pokok                                   | Simpanan Wajib                                   | Simpanan Sukarela                                                                     |
| -------------------------------------- | ------------------------------------------------ | ------------------------------------------------ | ------------------------------------------------------------------------------------- |
| **Bentuk aset**                  | Dana tunai Simpanan Pokok                        | Dana tunai                                       | Dana tunai                                                                            |
| **Status kepemilikan**           | Menjadi modal sendiri koperasi atas nama anggota | Menjadi modal sendiri koperasi atas nama anggota | Dana titipan anggota, tetapi tercatat sebagai kewajiban koperasi karena dapat ditarik |
| **Tujuan utama**                 | Syarat keanggotaan (komitmen)                    | Partisipasi rutin menambah modal                 | Tabungan fleksibel & cadangan likuid anggota                                          |
| **Frekuensi setor**              | Satu kali saat bergabung                         | Berkala (biasanya bulanan)                       | Bebas kapan saja                                                                      |
| **Hak anggota atas dana/barang** | Tidak dapat ditarik selama masih anggota         | Tidak dapat ditarik selama masih anggota         | Bebas tarik sesuai aturan koperasi                                                    |
| **Keuntungan koperasi**          | SHU* atas modal                                  | SHU* atas modal                                  | Selisih jasa pengelolaan (bila ada)                                                   |

# Database

### 🧾 Tabel: `users`

| Kolom    | Tipe Data    | Keterangan                    |
| -------- | ------------ | ----------------------------- |
| id_user  | CHAR(16)     | Primary Key (NIK)             |
| username | VARCHAR(50)  | Harus unik                    |
| password | VARCHAR(255) | Disimpan dalam bentuk hash    |
| role     | ENUM         | Nilai:`penjual`,`pembeli` |

---

### 👤 Tabel: `anggota`

| Kolom          | Tipe Data     | Keterangan                   |
| -------------- | ------------- | ---------------------------- |
| id_user        | CHAR(16)      | Primary Key, FK ke `users` |
| nama           | VARCHAR(100)  | Nama lengkap anggota         |
| alamat         | TEXT          | Alamat tempat tinggal        |
| no_hp          | VARCHAR(16)   | Nomor telepon                |
| tanggal_gabung | DATE          | Tanggal menjadi anggota      |
| saldo          | DECIMAL(12,2) | Saldo koperasi (default 0)   |

---

### 💰 Tabel: `simpanan`

| Kolom       | Tipe Data     | Keterangan                             |
| ----------- | ------------- | -------------------------------------- |
| id_simpanan | INT (AI)      | Primary Key                            |
| id_user     | CHAR(16)      | FK ke `users`                        |
| jenis       | ENUM          | Nilai:`pokok`,`wajib`,`sukarela` |
| jumlah      | DECIMAL(12,2) | Nominal setoran                        |
| tanggal     | DATE          | Tanggal simpan                         |
| keterangan  | TEXT          | Catatan tambahan                       |

---

### 📦 Tabel: `barang_titipan`

| Kolom         | Tipe Data     | Keterangan                      |
| ------------- | ------------- | ------------------------------- |
| id_barang     | INT (AI)      | Primary Key                     |
| id_user       | CHAR(16)      | FK ke `users`(penjual)        |
| nama_barang   | VARCHAR(100)  | Nama barang                     |
| type_barang   | VARCHAR(50)   | Jenis/kategori barang           |
| harga_titip   | DECIMAL(12,2) | Harga yang diberikan ke penjual |
| harga_jual    | DECIMAL(12,2) | Harga jual koperasi ke pembeli  |
| stok          | INT           | Jumlah barang tersedia          |
| foto          | VARCHAR(50)   | Nama file yang di simpan        |
| tanggal_masuk | DATE          | Tanggal barang masuk koperasi   |

---

### 🛒 Tabel: `penjualan_barang`

| Kolom        | Tipe Data | Keterangan                  |
| ------------ | --------- | --------------------------- |
| id_penjualan | INT (AI)  | Primary Key                 |
| id_barang    | INT       | FK ke `barang_titipan`    |
| id_pembeli   | CHAR(16)  | FK ke `users`(pembeli)    |
| jumlah       | INT       | Jumlah barang yang dibeli   |
| tanggal      | DATE      | Tanggal transaksi penjualan |

---

### 🏦 Tabel: `tabungan`

| Kolom           | Tipe Data     | Keterangan                |
| --------------- | ------------- | ------------------------- |
| id_tabungan     | INT (AI)      | Primary Key               |
| id_user         | CHAR(16)      | FK ke `users`           |
| jenis_transaksi | ENUM          | Nilai:`setor`,`tarik` |
| jumlah          | DECIMAL(12,2) | Nominal transaksi         |
| tanggal         | DATE          | Tanggal transaksi         |
| keterangan      | TEXT          | Catatan tambahan          |

---

### SQL

```sql
-- Tabel Users (id_user = NIK sebagai ID utama)
CREATE TABLE users (
    id_user CHAR(16) PRIMARY KEY,  -- NIK
    username VARCHAR(50) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    role ENUM('penjual', 'pembeli')
);

-- Tabel Anggota
CREATE TABLE anggota (
    id_user CHAR(16) PRIMARY KEY,
    nama VARCHAR(100) NOT NULL,
    alamat TEXT,
    no_hp VARCHAR(16) NOT NULL,
    tanggal_gabung DATE NOT NULL,
    saldo DECIMAL(12,2) DEFAULT 0.00,
    FOREIGN KEY (id_user) REFERENCES users(id_user) ON DELETE CASCADE
);

-- Tabel Simpanan
CREATE TABLE simpanan (
    id_simpanan INT AUTO_INCREMENT PRIMARY KEY,
    id_user CHAR(16) NOT NULL,
    jenis ENUM('pokok', 'wajib', 'sukarela') NOT NULL,
    jumlah DECIMAL(12,2) NOT NULL,
    tanggal DATE NOT NULL,
    keterangan TEXT,
    FOREIGN KEY (id_user) REFERENCES users(id_user) ON DELETE CASCADE
);

-- Tabel Barang Titipan
CREATE TABLE barang_titipan (
    id_barang INT AUTO_INCREMENT PRIMARY KEY,
    id_user CHAR(16) NOT NULL,
    nama_barang VARCHAR(100) NOT NULL,
    type_barang VARCHAR(50) NOT NULL,
    harga_titip DECIMAL(12,2) NOT NULL,
    harga_jual DECIMAL(12,2) NOT NULL,
    stok INT NOT NULL,
    tanggal_masuk DATE NOT NULL,
    FOREIGN KEY (id_user) REFERENCES users(id_user) ON DELETE CASCADE
);

-- Tabel Penjualan Barang
CREATE TABLE penjualan_barang (
    id_penjualan INT AUTO_INCREMENT PRIMARY KEY,
    id_barang INT NOT NULL,
    id_pembeli CHAR(16) NOT NULL,
    jumlah INT NOT NULL,
    tanggal DATE NOT NULL,
    FOREIGN KEY (id_barang) REFERENCES barang_titipan(id_barang) ON DELETE CASCADE,
    FOREIGN KEY (id_pembeli) REFERENCES users(id_user) ON DELETE CASCADE
);

CREATE TABLE tabungan (
    id_tabungan INT AUTO_INCREMENT PRIMARY KEY,
    id_user CHAR(16) NOT NULL,
    jenis_transaksi ENUM('setor', 'tarik') NOT NULL,
    jumlah DECIMAL(12,2) NOT NULL,
    tanggal DATE NOT NULL,
    keterangan TEXT,
    FOREIGN KEY (id_user) REFERENCES users(id_user) ON DELETE CASCADE
);
```

# Alur Sistem

### Registrasi Anggota

![Alur Pembuatan Akun](./img/registrasi.svg)

### Log-in

![Alur Input Produk](./img/login.svg)

### Log-out

![Alur Jual Beli](./img/logout.svg)
