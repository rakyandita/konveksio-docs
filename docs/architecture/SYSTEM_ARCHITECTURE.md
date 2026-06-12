# 🏗️ System Architecture — Konveksio

> **Versi:** 2.0 (Modular Refactor)
> **Sumber:** `docs/requirement_spec.md` Bagian 6 (Non-Functional Requirements)

---

## 6.1 Performa

- Halaman harus load dalam < 3 detik di koneksi 4G
- API response time < 500ms untuk operasi CRUD standar
- Support offline mode untuk fitur kritis (lihat daftar tugas, update progress)

---

## 6.2 Keamanan

- Enkripsi password (bcrypt)
- JWT token untuk autentikasi API
- Role-based access control (RBAC)
- HTTPS untuk semua komunikasi
- Input validation & sanitization

---

## 6.3 Skalabilitas & SaaS Readiness

- Arsitektur modular agar mudah menambah cabang baru
- **Multi-Tenant Architecture:** Database schema wajib didesain menggunakan pendekatan *multi-tenant* dengan kolom `company_id` (Tenant) di setiap tabel utama (meskipun di Fase 1 MVP hanya diisi 1 *company*). Hal ini merupakan mitigasi dan persiapan transisi menuju SaaS di Fase 2 tanpa perlu merombak ulang database.
- API versioning untuk backward compatibility

---

## 6.4 Offline Support

- **Status:** CRITICAL (wajib berjalan lancar karena sinyal buruk di pabrik Serang)
- Data tugas produksi tersimpan lokal (SQLite/Hive di Flutter)
- Sinkronisasi otomatis saat kembali online
- **Conflict resolution:** Menggunakan **Additive Sync** (penjumlahan data qty untuk progress, contoh: A tambah 20, B tambah 30, total 50) untuk mencegah *race condition* atau data tertimpa jika menggunakan metode *last-write-wins*.

---

## 6.5 Kompatibilitas

- Android 8.0+ (API level 26+)
- iOS 13+
- Backend compatible dengan shared hosting (PHP 8.1+, PostgreSQL 14+)

---

## 7. Constraint & Asumsi

### Constraint
1. Budget hosting terbatas (Rp500rb/tahun untuk tahap awal, space ~7GB)
2. Shared hosting tidak bisa run queue worker 24/7 → gunakan cron job sebagai alternatif. *(Catatan: Di Fase 2 / SaaS, infrastruktur wajib pindah ke VPS)*.
3. Karyawan lapangan mungkin punya device dengan spek rendah → app harus ringan
4. Sinyal internet di area produksi sangat buruk → butuh offline support yang solid (CRITICAL)
5. **Storage Media:** Wajib implementasi kompresi gambar (*image compression*) agresif di sisi Mobile App (Flutter) sebelum di-upload. Disarankan menggunakan arsitektur Hybrid Storage (menggabungkan hosting lokal dan CDN gratis).

### Asumsi
1. Semua karyawan memiliki smartphone Android/iOS
2. Owner/Admin memiliki akses internet yang stabil
3. WhatsApp API provider (Fonnte/Wablas) tersedia dan reliable
4. Pelanggan nyaman mengakses public link via browser mobile
