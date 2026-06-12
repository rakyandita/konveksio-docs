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

## Checklist Sebelum Commit

- [ ] Routing endpoint baru sudah divalidasi (`php artisan route:list`)
- [ ] Migrasi sudah diverifikasi (`php artisan migrate --pretend`)
- [ ] `dart analyze` tidak mengembalikan error
- [ ] Tidak ada overflow linting warnings pada UI yang diubah
- [ ] Kode baru sudah mengikuti `CODING_STANDARD.md`
