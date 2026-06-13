# 🔐 Security & Privacy Standard — Konveksio

> **Versi:** 4.0 (Token, Rate Limit & Audit Trail Update)
> **Sumber:** `docs/requirement_spec.md` Bagian 4.2 (Matriks Hak Akses / RBAC)
> **Diperbarui:** 13 Juni 2026

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
| **Order Item (SPK)** | ✅ `cabang_id` | Langsung (Denorm) | Denormalisasi untuk performa query |
| **Tahap Produksi** | ✅ `cabang_id` | Langsung (Denorm) | Denormalisasi untuk performa query |
| **Pembayaran** | ✅ via `order.cabang_id` | Inherit | Mengikuti order induknya |
| **Invoice** | ✅ `cabang_id` | Langsung (Denorm) | Denormalisasi untuk performa query |
| **Pengiriman** | ✅ via `order.cabang_id` | Inherit | Mengikuti order induknya |
| **Kasbon** | ✅ via `karyawan.cabang_id` | Inherit | Admin hanya approve kasbon karyawan cabangnya |
| **Gaji** | ✅ via `karyawan.cabang_id` | Inherit | Admin hanya rekap gaji karyawan cabangnya |
| **Rekening** | ✅ `cabang_id` | Langsung | Setiap cabang punya rekening bank sendiri |
| **Notifikasi** | ✅ via `user.cabang_id` | Inherit | Notifikasi hanya terkirim ke user di cabang yang relevan |
| **Transaksi Antar Cabang** | ✅ `cabang_asal_id` + `cabang_tujuan_id` | Khusus | Satu-satunya entitas yang melibatkan 2 cabang sekaligus |
| **Log Handover** | ✅ `cabang_id` | Langsung (Denorm) | Denormalisasi untuk performa query |
| **Porsi Pekerjaan** | ✅ `cabang_id` | Langsung (Denorm) | Denormalisasi untuk performa query |

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
| Setting Tarif Borongan | ✅ (semua cabang) | ✅ (cabang sendiri) | ❌ |
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

---

## Token Management

| Aspek | Nilai | Keterangan |
|---|---|---|
| JWT Token TTL | **60 menit** | Token expired setelah 1 jam |
| Refresh Token TTL | **7 hari** | Refresh token berlaku 1 minggu |
| Refresh Endpoint | `POST /api/v1/auth/refresh` | Return token baru |
| Logout | `POST /api/v1/auth/logout` | Blacklist token yang aktif |
| Token Blacklist | Via JWT blacklist cache | Token yang sudah logout tidak bisa dipakai |
| JWT Claims Wajib | `user_id`, `company_id`, `cabang_id`, `role` | Untuk branch isolation & RBAC |
| Token Refresh Rotation | Ya | Token lama di-invalidate saat refresh |

### Aturan Token

1. **Jangan simpan token di localStorage** — gunakan `flutter_secure_storage` (Keychain di iOS, EncryptedSharedPreferences di Android)
2. **Auto-refresh:** Jika token expired, Flutter otomatis coba refresh. Jika refresh gagal → redirect ke login
3. **Token di response login:** Backend return `token`, `token_type`, `expires_in` (dalam detik)
4. **Blacklist setelah logout:** Token yang sudah logout WAJIB masuk blacklist agar tidak bisa dipakai lagi

---

## Rate Limiting

> Mencegah brute force, DDoS, dan penyalahgunaan API.

| Endpoint / Group | Limit | Window | Keterangan |
|---|---|---|---|
| `POST /api/v1/auth/login` | **5 request** | per menit | Per IP, mencegah brute force |
| `POST /api/v1/auth/refresh` | **10 request** | per menit | Per user |
| API umum (authenticated) | **60 request** | per menit | Per user |
| Public endpoint (invoice) | **30 request** | per menit | Per IP |
| File upload | **10 request** | per menit | Per user |

### Implementasi

Gunakan Laravel built-in rate limiter:

```php
// Di bootstrap/app.php atau RouteServiceProvider
RateLimiter::for('login', function (Request $request) {
    return Limit::perMinute(5)->by($request->ip());
});

RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});
```

### Response Saat Limit Tercapai

```json
{
  "success": false,
  "message": "Terlalu banyak percobaan. Silakan coba lagi dalam 1 menit."
}
```

HTTP Status: **429 Too Many Requests**

---

## Audit Trail

> Setiap operasi penting WAJIB tercatat agar bisa ditelusuri.

### Aturan Audit Log

- **Library:** `spatie/laravel-activitylog` (^4.0)
- **Model yang WAJIB di-audit:** Order, Pembayaran, Kasbon, Invoice, User, Pengeluaran, MutasiCabang
- **Event yang di-log:** `created`, `updated`, `deleted`

### Data yang Tercatat

| Field | Keterangan |
|---|---|
| `causer_id` | ID user yang melakukan aksi |
| `causer_type` | Tipe user (App\Models\User) |
| `subject_type` | Tipe model (App\Models\Order) |
| `subject_id` | ID model yang diubah |
| `description` | Deskripsi aksi ("created", "updated", "deleted") |
| `properties` | JSON berisi `attributes` (data baru) dan `old` (data lama) |
| `ip_address` | IP address user |
| `created_at` | Timestamp aksi |

### Implementasi

```php
// Di Model
use Spatie\Activitylog\Traits\LogsActivity;
use Spatie\Activitylog\LogOptions;

class Order extends Model
{
    use LogsActivity;

    public function getActivitylogOptions(): LogOptions
    {
        return LogOptions::defaults()
            ->logOnly(['status', 'total_harga', 'catatan'])
            ->logOnlyDirty(); // Hanya log field yang berubah
    }
}
```

### Query Audit Log

```php
// Lihat semua aktivitas user tertentu
Activitylog::forSubject($order)->get();

// Lihat siapa yang mengubah status order
Activitylog::forSubject($order)
    ->where('description', 'updated')
    ->get();
```

---

## File Upload Validation

| Aspek | Aturan |
|---|---|
| **Max file size** | 5 MB |
| **Allowed MIME types** | `jpg`, `jpeg`, `png`, `pdf` |
| **Storage** | `storage/app/public/` dengan symlink |
| **Filename** | UUID + extension (`a1b2c3d4-e5f6.jpg`) — JANGAN pakai nama asli file |
| **Virus scan** | Opsional di Fase 1, wajib di Fase 2 (SaaS) |

### Validasi di FormRequest

```php
'bukti_transfer' => 'required|file|mimes:jpg,jpeg,png,pdf|max:5120',
'desain_file' => 'nullable|file|mimes:jpg,jpeg,png,pdf|max:5120',
```

### Aturan Keamanan File

1. **Jangan pernah trust MIME type dari client** — validasi ulang di server menggunakan `fileinfo` extension
2. **Jangan simpan file dengan nama asli** — gunakan UUID untuk mencegah path traversal
3. **Jangan expose path absolut** — return path relatif di response
4. **Set permission yang tepat** — file upload tidak boleh executable
