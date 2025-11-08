# LulutResident
Sistem Informasi Manajemen Kavling — aplikasi Laravel untuk pengelolaan unit rumah, pelanggan, reservasi, pembayaran, dan laporan penjualan.  
Tujuan: mendigitalisasi manajemen data penjualan kavling secara sederhana namun fungsional untuk kebutuhan PKL.

---

## 🎯 Tujuan
Membuat aplikasi berbasis web internal untuk pengelolaan dan pelaporan unit perumahan, di mana **semua tipe rumah sama** (1 tipe).  
Aplikasi ini hanya digunakan oleh pihak internal (Admin dan Sales), bukan publik.

---

## 👥 Roles & Permissions
Aplikasi memiliki dua peran utama (role):

### **1. Admin**
- Full access ke seluruh fitur dan data.
- CRUD Units, Pelanggan, Reservations, Payments, Reports.
- Mengelola pengguna (membuat akun Sales baru).
- Menyetujui & mengubah status penjualan menjadi *Terjual*.
- Melihat log aktivitas sistem.

### **2. Sales**
- Membuat dan mengelola data pelanggan (prospek dan pembeli).
- Membuat pemesanan (reservation) dan mencatat pembayaran awal (deposit).
- Mengubah status unit menjadi *Dipesan*.
- Melihat laporan penjualan dan unit yang dikelola.
- Tidak bisa membuat user baru.

> **Tidak ada fitur Register Publik.**  
> Semua akun dibuat oleh **Admin melalui dashboard** (menu Manajemen Pengguna).  
> Login hanya untuk pengguna internal (Admin dan Sales).

---

## 🏠 Alur Status Unit
Unit hanya memiliki satu tipe rumah (denah sama).  
Status unit mengikuti 3 tahapan logis:

| Status | Keterangan | Diubah oleh |
|--------|-------------|-------------|
| **Tersedia** | Unit belum dipesan atau dijual | Sistem default |
| **Dipesan** | Unit dipesan oleh pelanggan (ada reservation aktif) | Sales/Admin |
| **Terjual** | Unit sudah terjual dan dikonfirmasi admin | Admin |

### Transisi status:
- `Tersedia` → `Dipesan` (Sales/Admin)
- `Dipesan` → `Terjual` (Admin)
- `Dipesan` → `Tersedia` (Batal / Expired)
- `Tersedia` → `Terjual` (Langsung, tanpa pemesanan)

Validasi:
- Tidak boleh double booking.
- Saat status `Terjual`, wajib ada `pelanggan_id` dan `tanggal_terjual`.

---

## 🧱 Struktur Database

### **users**
| Field | Type | Info |
|--------|------|------|
| id | BIGINT | PK |
| name | VARCHAR | Nama user |
| email | VARCHAR | Unique |
| password | VARCHAR | Hash |
| role | ENUM('admin','sales') | Hak akses |
| created_at / updated_at | TIMESTAMP | Otomatis |

---

### **units**
| Field | Type | Info |
|--------|------|------|
| id | BIGINT | PK |
| kode_unit | VARCHAR | Mis. A1 |
| blok | VARCHAR | Nama blok |
| status | ENUM('Tersedia','Dipesan','Terjual') | Default Tersedia |
| pelanggan_id | BIGINT (nullable) | FK ke pelanggan |
| tanggal_terjual | DATE | Nullable |
| catatan | TEXT | Opsional |
| created_at / updated_at | TIMESTAMP |  |

---

### **pelanggans**
| Field | Type | Info |
|--------|------|------|
| id | BIGINT | PK |
| nama_lengkap | VARCHAR |  |
| no_hp | VARCHAR |  |
| email | VARCHAR (nullable) |  |
| alamat | TEXT | Nullable |
| created_at / updated_at | TIMESTAMP |  |

---

### **reservations**
| Field | Type | Info |
|--------|------|------|
| id | BIGINT | PK |
| unit_id | BIGINT | FK |
| pelanggan_id | BIGINT | FK |
| user_id | BIGINT | FK (sales yang membuat) |
| status | ENUM('pending','confirmed','cancelled') | Default pending |
| deposit_amount | DECIMAL | Nullable |
| deposit_date | DATETIME | Nullable |
| expired_at | DATETIME | Nullable |
| notes | TEXT | Nullable |
| created_at / updated_at | TIMESTAMP |  |

---

### **payments**
| Field | Type | Info |
|--------|------|------|
| id | BIGINT | PK |
| reservation_id | BIGINT | Nullable |
| unit_id | BIGINT | Nullable |
| amount | DECIMAL | Nominal bayar |
| method | VARCHAR | Tunai / Transfer |
| paid_at | DATETIME |  |
| note | TEXT |  |
| created_at / updated_at | TIMESTAMP |  |

---

### **unit_images**
| Field | Type | Info |
|--------|------|------|
| id | BIGINT | PK |
| unit_id | BIGINT | Nullable (bisa global layout) |
| path | VARCHAR | Path file image |
| caption | VARCHAR |  |
| created_at / updated_at | TIMESTAMP |  |

---

### **activity_logs**
| Field | Type | Info |
|--------|------|------|
| id | BIGINT | PK |
| user_id | BIGINT | FK |
| action | VARCHAR | Mis. “update_status” |
| model | VARCHAR | Mis. “Unit” |
| model_id | BIGINT |  |
| detail | JSON | Data perubahan |
| created_at | TIMESTAMP |  |

---

## 📊 Laporan Minimal (5 Jenis)
1. **Laporan Status Unit**
   - Semua unit beserta statusnya.
   - Filter: blok, status.
2. **Laporan Penjualan**
   - Unit terjual, pelanggan, tanggal, sales.
   - Filter: rentang tanggal.
3. **Laporan Unit Tersedia**
   - Unit dengan status `Tersedia`.
4. **Laporan Data Pelanggan**
   - Nama, kontak, unit yang dibeli.
5. **Laporan Rekapitulasi Penjualan Periodik**
   - Jumlah unit terjual + total penjualan per periode.

> Semua laporan dapat diekspor ke **CSV & PDF**  
> Dilengkapi **filter tanggal, blok, dan sales**.

---

## 📂 API / Routes (ringkas)
| Route | Role | Keterangan |
|--------|------|------------|
| `/login` | Semua | Login internal |
| `/logout` | Semua | Logout |
| `/users` | Admin | CRUD pengguna (buat akun Sales) |
| `/units` | Admin / Sales | CRUD unit |
| `/pelanggans` | Admin / Sales | CRUD pelanggan |
| `/reservations` | Admin / Sales | Pemesanan unit |
| `/payments` | Admin / Sales | Catat pembayaran |
| `/reports/...` | Admin / Sales | Laporan |
| `/activity-logs` | Admin | Log aktivitas sistem |

---

## 🧭 UI/UX (Rencana Tampilan)
- **Dashboard**: statistik ringkas (tersedia, dipesan, terjual).
- **Daftar Unit**: tabel + filter blok/status.
- **Pemesanan (Reservation)**: pilih unit → isi pelanggan → input deposit.
- **Laporan**: filter + tabel + export CSV/PDF.
- **Manajemen User (Admin)**: tambah/edit user (role sales).
- **Landing Page Publik**: denah rumah tunggal, status unit, kontak (tanpa login).

---

## 🔒 Autentikasi & Keamanan
- Gunakan Laravel Breeze / Jetstream.
- **Fitur Register dinonaktifkan.**
- Route register dihapus (`Auth::routes(['register' => false])`).
- Semua route dilindungi middleware `auth` dan `role`.
- Hash password menggunakan bcrypt.
- Log setiap perubahan status unit di `activity_logs`.

---

## ⚙️ Teknologi
- **Backend:** Laravel 11 (PHP 8.2)
- **Frontend:** Blade + Tailwind CSS
- **Database:** MySQL
- **PDF Export:** laravel-dompdf
- **Auth:** Laravel Breeze (tanpa register publik)

---

## 🧩 Urutan Implementasi (MVP)
1. Setup Laravel + Auth (login only)
2. CRUD Users (Admin only)
3. CRUD Units + status logic
4. CRUD Pelanggans
5. Reservations workflow (Sales)
6. Payments
7. Reports (5 jenis)
8. Activity logs
9. UI Polishing + Denah landing

---

## 🏁 Catatan Akhir
- Semua rumah menggunakan **1 tipe/denah yang sama**.
- Sistem **tidak membuka pendaftaran umum**.
- Setiap akun **hanya dibuat oleh Admin**.
- Desain dibuat **modular dan ringan**, cukup untuk PKL dan pengembangan lebih lanjut.

---
