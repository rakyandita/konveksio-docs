# 🧱 Tech Stack — Konveksio

> **Versi:** 3.0 (Library & Extension Update)
> **Sumber:** `docs/requirement_spec.md` Bagian 3
> **Diperbarui:** 13 Juni 2026

---

## 1. Tabel Tech Stack Utama

| Komponen | Teknologi | Versi |
|---|---|---|
| Backend Framework | Laravel | 13.x (latest stable) |
| PHP | PHP | 8.2+ |
| Database | PostgreSQL | 14+ (Safe Baseline) |
| Mobile App | Flutter | Stable Channel (latest) |
| State Management | Riverpod | Latest stable |
| Hosting (tahap awal) | AnymHost Developer Hosting | Shared hosting, ~Rp500rb/tahun |
| Hosting (tahap lanjut) | VPS (DigitalOcean/Vultr) | Saat SaaS Fase 2 |
| Notifikasi WA | WhatsApp API (Fonnte/Wablas) | REST API |
| Push Notification | Firebase Cloud Messaging (FCM) | Firebase Admin SDK |
| Offline Support | Hive + SQLite (Flutter) | Sync saat online |
| Over-The-Air Update | Shorebird | Auto-update tanpa Play Store |

---

## 2. Backend Libraries (Laravel)

### 2.1 Library Wajib

| Library | Versi | Fungsi | Alasan |
|---|---|---|---|
| `php-open-source-saver/jwt-auth` | ^2.0 | JWT Authentication | Lightweight, well-maintained, cocok untuk API-only |
| `barryvdh/laravel-dompdf` | ^3.0 | Generate PDF invoice | Simple, cukup untuk invoice PDF |
| `intervention/image` | ^3.0 | Kompresi & resize gambar upload | Industry standard untuk image processing |
| `spatie/laravel-activitylog` | ^4.0 | Audit trail (siapa lakukan apa) | Required oleh SECURITY_PRIVACY_STANDARD |

### 2.2 Library Opsional (Tambahkan Jika Perlu)

| Library | Fungsi | Kapan Ditambahkan |
|---|---|---|
| `maatwebsite/excel` | Export Excel laporan | Saat modul M-08 (Laporan Keuangan) |
| `kreait/firebase-php` | Firebase Admin SDK (FCM server) | Saat modul M-10 (Notifikasi) |
| `spatie/laravel-query-builder` | Filter, sort, include di API | Jika query filter makin kompleks |

### 2.3 Library yang DILARANG

| Library | Alasan Dilarang |
|---|---|
| `laravel/sanctum` | Konflik dengan JWT — pilih salah satu. Kita pakai JWT. |
| `laravel/passport` | Overkill untuk use case ini, OAuth2 tidak dibutuhkan |
| `tymon/jwt-auth` | Deprecated, sudah diganti `php-open-source-saver/jwt-auth` |

---

## 3. PHP Extensions Wajib

Extension berikut **WAJIB** terinstal di server (local maupun production):

| Extension | Fungsi | Diperlukan Oleh |
|---|---|---|
| `bcmath` | Kalkulasi presisi tinggi (uang) | Kalkulasi HPP, gaji, invoice |
| `intl` | Format angka, tanggal, currency | Format Rp, tanggal Indonesia |
| `gd` atau `imagick` | Image processing | Kompresi foto upload |
| `pdo_pgsql` | Koneksi PostgreSQL | Database |
| `mbstring` | String multibyte | UTF-8, Bahasa Indonesia |
| `zip` | Composer dependency | Laravel internal |
| `curl` | HTTP request ke API eksternal | WhatsApp API, FCM |
| `openssl` | Enkripsi JWT, HTTPS | Security |
| `tokenizer` | Laravel internal | Framework |
| `xml` | Laravel internal | Framework |
| `ctype` | Laravel internal | Framework |
| `fileinfo` | MIME type detection | Validasi file upload |

### Verifikasi di Server

```bash
# Cek extension yang terinstal
php -m

# Cek versi PHP
php -v

# Cek koneksi PostgreSQL
php -r "new PDO('pgsql:host=localhost;dbname=konveksio', 'user', 'pass');"
```

---

## 4. Flutter Dependencies

### 4.1 Package Wajib

| Package | Fungsi | Keterangan |
|---|---|---|
| `flutter_riverpod` | State management | Wajib sesuai CODING_STANDARD |
| `dio` | HTTP client | Untuk komunikasi dengan API Laravel |
| `hive` / `hive_flutter` | Local NoSQL storage | Offline data cache |
| `sqflite` | Local SQLite database | Offline structured data |
| `flutter_secure_storage` | Simpan token & biometrik | Keychain / EncryptedSharedPreferences |
| `go_router` | Navigation & routing | Type-safe routing |
| `freezed` / `json_serializable` | Model class & JSON serialization | Code generation |
| `cached_network_image` | Cache gambar dari URL | Performa loading gambar |
| `image_picker` | Pilih foto dari gallery/camera | Upload bukti pembayaran, desain |
| `url_launcher` | Buka link external | Public link invoice, WhatsApp |
| `permission_handler` | Request permissions | Camera, storage access |
| `firebase_core` + `firebase_messaging` | FCM push notification | Notifikasi real-time |
| `connectivity_plus` | Cek koneksi internet | Offline mode detection |
| `lucide_icons` | Icon set | Sesuai DESIGN_SYSTEM |

### 4.2 Package Opsional

| Package | Fungsi | Kapan Ditambahkan |
|---|---|---|
| `flutter_pdfview` | Preview PDF in-app | Saat fitur preview invoice |
| `printing` | Print / share PDF | Saat fitur print invoice |
| `flutter_local_notifications` | Local notification | Reminder offline |
| `qr_flutter` | Generate QR code | Saat fitur QR invoice |

---

## 5. Development Tools

### 5.1 Backend

| Tool | Fungsi |
|---|---|
| `laravel/pint` | Code formatter (PSR-12) |
| `larastan/larastan` | Static analysis (level 5) |
| `phpunit/phpunit` | Unit & feature testing |
| `fakerphp/faker` | Generate dummy data untuk test & seeder |

### 5.2 Mobile

| Tool | Fungsi |
|---|---|
| `flutter_lints` | Linting rules standar |
| `build_runner` | Code generation (freezed, json_serializable) |
| `flutter_test` | Widget & unit testing |

---

## 6. Environment Requirements

### 6.1 Local Development

| Komponen | Minimum | Recommended |
|---|---|---|
| PHP | 8.2 | 8.3 |
| PostgreSQL | 14 | 16 |
| Composer | 2.x | Latest |
| Node.js | 18 LTS | 20 LTS |
| Flutter SDK | 3.x stable | Latest stable |
| Android Studio | Latest | Latest (untuk emulator) |
| Xcode (macOS) | Latest | Latest (untuk iOS simulator) |

### 6.2 Production (Shared Hosting)

| Komponen | Nilai |
|---|---|
| PHP | 8.2+ (pastikan tersedia di hosting) |
| PostgreSQL | 14+ |
| PHP Memory Limit | Minimal 256MB |
| PHP Max Execution Time | 60 detik |
| Upload Max Filesize | 10MB |
| Disk Space | ~7GB (sesuai constraint) |
| Cron Job | Wajib (untuk deadline notification) |

---

## 7. Compatibility

| Platform | Minimum Version |
|---|---|
| Android | 8.0 (API level 26) |
| iOS | 13.0 |
| Browser (public invoice) | Chrome/Safari/Firefox latest 2 versions |
