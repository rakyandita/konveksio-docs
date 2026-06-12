# 🧪 Testing Standard — Konveksio

> **Versi:** 1.0
> **Dibuat:** 11 Juni 2026

---

## Aturan Testing Backend (Laravel)

### Validasi Routing Endpoint
- Setiap selesai membuat endpoint Laravel baru, agent **WAJIB** memvalidasi bahwa routing tidak error dengan menjalankan:
  ```bash
  php artisan route:list --name=[nama_route]
  ```
- Pastikan tidak ada konflik route atau route yang `Missing` sebelum melanjutkan ke implementasi berikutnya.
- Jika endpoint memerlukan middleware (auth, role), wajib diuji bahwa middleware teraplikasi dengan benar.

### Validasi Migrasi Database
- Setelah membuat atau mengubah file migrasi, wajib menjalankan:
  ```bash
  php artisan migrate --pretend
  ```
  untuk memverifikasi SQL yang dihasilkan sebelum menjalankan migrasi sesungguhnya.

### Automated Unit Test (Wajib untuk Finansial)
- Setiap fitur yang menyangkut uang, seperti **Kalkulasi Gaji Borongan**, **Kasbon**, **Potongan**, atau **Kalkulator HPP**, **WAJIB** dibuatkan *Automated Unit Test* (menggunakan PHPUnit untuk Laravel).
- Test harus mencakup skenario normal, skenario batas (edge cases), dan skenario *error*.

---

## Aturan Testing Flutter UI

### Bebas Syntax Error
- Setiap perubahan Flutter UI **WAJIB** bebas dari syntax error sebelum dianggap selesai.
- Validasi dilakukan dengan:
  ```bash
  dart analyze
  ```
  dan pastikan tidak ada error (hanya warning/info yang diizinkan).

### Bebas Overflow Linting Warnings
- Setiap perubahan Flutter UI **WAJIB** bebas dari overflow linting warnings (`RenderFlex overflowed`).
- Jika ada overflow terdeteksi, wajib diperbaiki sebelum melanjutkan.
- Rujuk ke `docs/standard/CODING_STANDARD.md` untuk panduan pencegahan overflow.

---

## Verifikasi Ketat (Zero Error Policy)

- **Jalan Langsung Jadi:** Dilarang melakukan *commit* atau menganggap tugas selesai jika masih ada *error*, sekecil apa pun.
- Agent **WAJIB** melakukan verifikasi menyeluruh (*build*, *compile*, *test*) secara internal.
- Dilarang memberikan kode yang membutuhkan proses *debugging* panjang dari pihak User.

---

## Checklist Sebelum Commit

- [ ] Routing endpoint baru sudah divalidasi (`php artisan route:list`)
- [ ] Migrasi sudah diverifikasi (`php artisan migrate --pretend`)
- [ ] Unit Test untuk logika finansial sudah dibuat dan berstatus *PASS*
- [ ] `dart analyze` tidak mengembalikan error (Zero Error Policy ditaati)
- [ ] Tidak ada overflow linting warnings pada UI yang diubah
- [ ] Kode baru sudah mengikuti `CODING_STANDARD.md` (termasuk batas 300 baris)
