# 🔐 Security & Privacy Standard — Konveksio

> **Versi:** 2.0 (Modular Refactor)
> **Sumber:** `docs/requirement_spec.md` Bagian 4.2 (Matriks Hak Akses / RBAC)

---

## Definisi Role

| Role | Deskripsi | Cakupan |
|---|---|---|
| **Super Admin / Owner** | Pemilik bisnis, akses penuh ke semua data & semua cabang | Global |
| **Admin Cabang** | Pengelola operasional harian, akses penuh di cabangnya sendiri | Per Cabang |
| **Staff Produksi** | Karyawan lapangan (tukang potong, jahit, dll.), lihat order yang ditugaskan, update progress, handover estafet | Per Cabang |

---

## Matriks Hak Akses (RBAC)

| Fitur | Super Admin | Admin Cabang | Staff Produksi |
|---|---|---|---|
| Dashboard (semua cabang) | ✅ | ❌ | ❌ |
| Dashboard (cabang sendiri) | ✅ | ✅ | ❌ |
| CRUD Order | ✅ | ✅ | ❌ (Read only) |
| Input Pembayaran | ✅ | ✅ | ❌ |
| Handover Estafet | ✅ | ✅ | ✅ |
| Lihat Tugas Produksi | ✅ | ✅ | ✅ (milik sendiri) |
| CRUD Pelanggan | ✅ | ✅ | ❌ |
| CRUD Vendor | ✅ | ✅ | ❌ |
| CRUD Karyawan | ✅ | ✅ | ❌ |
| Laporan Keuangan | ✅ | ✅ | ❌ |
| Rekap Gaji | ✅ | ✅ | ❌ (Read own) |
| Approve Kasbon | ✅ | ✅ | ❌ |
| Ajukan Kasbon | ✅ | ✅ | ✅ |
| Setting Tarif Borongan | ✅ | ❌ | ❌ |
| Kelola Role & User | ✅ | ❌ | ❌ |
| Transaksi Antar Cabang | ✅ | ✅ | ❌ |

---

## Autentikasi

- **Login:** Email + Password / PIN karyawan
- **Biometrik:** Fingerprint / Face ID (opsional setelah login pertama)
- **Session:** Persistent login di mobile app

---

## Aturan Teknis Keamanan

- Enkripsi password menggunakan **bcrypt**
- Autentikasi API menggunakan **JWT token**
- Semua komunikasi melalui **HTTPS**
- Wajib implementasi **input validation & sanitization** di semua endpoint
- Kredensial biometrik disimpan menggunakan **flutter_secure_storage** (Keychain/EncryptedSharedPreferences)
