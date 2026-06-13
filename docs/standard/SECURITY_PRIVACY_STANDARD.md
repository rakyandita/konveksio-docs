# 🔐 Security & Privacy Standard — Konveksio

> **Versi:** 3.0 (Branch Isolation Update)
> **Sumber:** `docs/requirement_spec.md` Bagian 4.2 (Matriks Hak Akses / RBAC)
> **Diperbarui:** 12 Juni 2026

---

## Definisi Role

| Role | Deskripsi | Cakupan Data |
|---|---|---|
| **Super Admin / Owner** | Pemilik bisnis, akses penuh ke semua data & semua cabang | **Global** (lintas cabang) |
| **Admin Cabang** | Pengelola operasional harian, akses penuh di cabangnya sendiri | **Per Cabang** (hanya data cabang sendiri) |
| **Staff Produksi** | Karyawan lapangan (tukang potong, jahit, dll.), lihat order yang ditugaskan, update progress, handover estafet | **Per Cabang** (hanya data cabang sendiri) |

---

## Kebijakan Isolasi Data Cabang (Branch Data Isolation)

> ⚠️ **ATURAN ABSOLUT:** Setiap cabang memiliki manajemen independen. Seluruh data operasional **WAJIB terikat dan terisolasi per cabang**. Tidak ada data yang "bocor" atau terlihat oleh cabang lain, **kecuali** diakses oleh Super Admin.

### Prinsip Dasar

1. **Setiap entitas data memiliki `cabang_id`** — tanpa kecuali. Data yang dibuat di Cabang Serang hanya bisa dilihat, diubah, dan dihapus oleh user yang terdaftar di Cabang Serang.
2. **Super Admin adalah satu-satunya role yang bisa melihat dan mengakses data lintas cabang.** Super Admin memiliki filter cabang di setiap halaman untuk berpindah konteks.
3. **Admin Cabang & Staff Produksi hanya bisa mengakses data milik cabangnya sendiri.** Sistem secara otomatis mem-filter data berdasarkan `cabang_id` user yang sedang login (implicit scope).
4. **Tidak ada fitur "share data antar cabang"** kecuali melalui modul Transaksi Antar Cabang (M-12) yang memang dirancang khusus untuk itu.

### Matriks Isolasi Per Entitas

| Entitas | Terikat Cabang? | Mekanisme Isolasi | Catatan |
|---|---|---|---|
| **Karyawan** | ✅ `cabang_id` | Langsung | Karyawan terdaftar di 1 cabang |
| **Pelanggan** | ✅ `cabang_id` | Langsung | Pelanggan Serang ≠ pelanggan Solo. Jika pelanggan yang sama order di 2 cabang, dibuat sebagai 2 record terpisah. |
| **Vendor** | ✅ `cabang_id` | Langsung | Vendor lokal per cabang. Jika vendor yang sama melayani 2 cabang, dibuat sebagai 2 record terpisah. |
| **Produk & Katalog** | ✅ `cabang_id` | Langsung | Katalog produk & tarif per tahap bisa berbeda antar cabang (Serang=borongan, Solo=CMT). |
| **Order** | ✅ `cabang_id` | Langsung | Order dibuat di cabang tertentu |
| **Order Item (SPK)** | ✅ via `order.cabang_id` | Inherit | Mengikuti order induknya |
| **Tahap Produksi** | ✅ via `order_item.order.cabang_id` | Inherit | Mengikuti order item induknya |
| **Pembayaran** | ✅ via `order.cabang_id` | Inherit | Mengikuti order induknya |
| **Invoice** | ✅ via `order.cabang_id` | Inherit | Mengikuti order induknya |
| **Pengiriman** | ✅ via `order.cabang_id` | Inherit | Mengikuti order induknya |
| **Kasbon** | ✅ via `karyawan.cabang_id` | Inherit | Admin hanya approve kasbon karyawan cabangnya |
| **Gaji** | ✅ via `karyawan.cabang_id` | Inherit | Admin hanya rekap gaji karyawan cabangnya |
| **Rekening** | ✅ `cabang_id` | Langsung | Setiap cabang punya rekening bank sendiri |
| **Notifikasi** | ✅ via `user.cabang_id` | Inherit | Notifikasi hanya terkirim ke user di cabang yang relevan |
| **Transaksi Antar Cabang** | ✅ `cabang_asal_id` + `cabang_tujuan_id` | Khusus | Satu-satunya entitas yang melibatkan 2 cabang sekaligus |
| **Log Handover** | ✅ via `tahap_produksi` chain | Inherit | Mengikuti rantai order item → order → cabang |
| **Porsi Pekerjaan** | ✅ via `tahap_produksi` chain | Inherit | Mengikuti rantai order item → order → cabang |

### Aturan Implementasi Teknis

1. **Backend (Laravel) — Dua Lapis Global Scope:**
   - **Lapis 1 — Tenant Isolation (`BelongsToTenant` trait):** Setiap model yang memiliki `company_id` WAJIB menggunakan trait ini. Mencegah data bocor antar perusahaan (krusial untuk SaaS).
   - **Lapis 2 — Branch Isolation (`BelongsToBranch` trait):** Setiap model yang memiliki `branch_id` (Pelanggan, Vendor, Produk, Order, dll.) WAJIB menggunakan trait ini. Mencegah data bocor antar cabang dalam 1 perusahaan.
2. **Super Admin Bypass:** Jika `user->role === 'super_admin'`, filter `branch_id` (Lapis 2) **tidak diterapkan** secara otomatis. Super Admin memilih cabang secara manual via filter UI.
3. **Validasi Tulis (Create/Update/Delete):** Setiap operasi tulis WAJIB memvalidasi bahwa resource target milik cabang yang sama dengan user yang melakukan operasi. Jika tidak cocok → **403 Forbidden**.
4. **Foreign Key Cross-Check:** Saat membuat relasi (misal: assign karyawan ke order), sistem WAJIB memvalidasi bahwa kedua entitas milik cabang yang sama.

---

## Matriks Hak Akses (RBAC)

> Semua fitur di bawah ini tunduk pada **Kebijakan Isolasi Data Cabang** di atas. Admin Cabang & Staff hanya melihat data cabang sendiri.

| Fitur | Super Admin | Admin Cabang | Staff Produksi |
|---|---|---|---|
| Dashboard (semua cabang) | ✅ | ❌ | ❌ |
| Dashboard (cabang sendiri) | ✅ | ✅ | ❌ |
| CRUD Order | ✅ (semua cabang) | ✅ (cabang sendiri) | ❌ (Read only, cabang sendiri) |
| Input Pembayaran | ✅ (semua cabang) | ✅ (cabang sendiri) | ❌ |
| Handover Estafet | ✅ | ✅ (cabang sendiri) | ✅ (cabang sendiri) |
| Lihat Tugas Produksi | ✅ | ✅ (cabang sendiri) | ✅ (milik sendiri, cabang sendiri) |
| CRUD Pelanggan | ✅ (semua cabang) | ✅ (cabang sendiri) | ❌ |
| CRUD Vendor | ✅ (semua cabang) | ✅ (cabang sendiri) | ❌ |
| CRUD Karyawan | ✅ (semua cabang) | ✅ (cabang sendiri) | ❌ |
| Laporan Keuangan | ✅ (semua cabang) | ✅ (cabang sendiri) | ❌ |
| Rekap Gaji | ✅ (semua cabang) | ✅ (cabang sendiri) | ❌ (Read own slip) |
| Approve Kasbon | ✅ (semua cabang) | ✅ (cabang sendiri) | ❌ |
| Ajukan Kasbon | ✅ | ✅ | ✅ |
| Setting Tarif Borongan | ✅ (semua cabang) | ❌ | ❌ |
| Kelola Role & User | ✅ | ❌ | ❌ |
| Transaksi Antar Cabang | ✅ | ✅ (cabang sendiri sebagai asal/tujuan) | ❌ |
| Katalog Produk & Bundle | ✅ (semua cabang) | ✅ (cabang sendiri) | ❌ |
| Pengaturan Rekening | ✅ (semua cabang) | ✅ (cabang sendiri) | ❌ |

---

## Autentikasi

- **Login:** Email + Password / PIN karyawan
- **Biometrik:** Fingerprint / Face ID (opsional setelah login pertama)
- **Session:** Persistent login di mobile app
- **Konteks Cabang:** Setelah login berhasil, sistem otomatis set konteks `cabang_id` berdasarkan data karyawan. Super Admin default melihat semua cabang, bisa switch via filter.

---

## Aturan Teknis Keamanan

- Enkripsi password menggunakan **bcrypt**
- Autentikasi API menggunakan **JWT token** (payload wajib berisi `user_id`, `cabang_id`, `role`)
- Semua komunikasi melalui **HTTPS**
- Wajib implementasi **input validation & sanitization** di semua endpoint
- Kredensial biometrik disimpan menggunakan **flutter_secure_storage** (Keychain/EncryptedSharedPreferences)
- **Branch scope enforcement** di setiap API endpoint (Laravel Global Scope / Middleware)
