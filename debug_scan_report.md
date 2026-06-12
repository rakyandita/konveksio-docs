# Laporan Debug Scan — Konveksio App

> **Tanggal Scan:** 9 Juni 2026
> **Scanner:** AI Agent (Automated)
> **Cakupan:** konveksio-api (Laravel) + konveksio-mobile (Flutter)

---

## Ringkasan Eksekutif

| Kategori | Status |
|----------|--------|
| **Kompilasi** | ✅ Berhasil (0 error dari `dart analyze`, semua route Laravel terdaftar) |
| **Bug Runtime** | 🔴 **4 BUG KRITIS** — dashboard, laporan, pembayaran, chart rusak |
| **Keamanan** | 🟡 PIN disimpan plain text di database |
| **UI Overflow** | 🟡 3 screen berisiko overflow pada layar kecil |
| **Kepatuhan SOT** | 🔴 ~5% (hanya GajiController yang mengikuti Service Pattern) |
| **Coding Standard** | 🟡 11 file Flutter melebihi batas 300 baris |

**Hasil `dart analyze`:** 0 error, 7 warning, 67 info — **aplikasi bisa di-build**, tetapi ada bug yang akan crash saat fitur tertentu digunakan.

---

## 🔴 BUG KRITIS (Crash saat Runtime)

### BUG-01: Dashboard Admin Crash — Kolom `deadline` Tidak Ada

- **Repo:** `konveksio-api`
- **File:** `app/Http/Controllers/Api/V1/Auth/DashboardController.php` (baris 98, 184)
- **Masalah:** Query menggunakan kolom `deadline` (`Order::where('deadline', ...)`) tetapi tabel `orders` hanya punya kolom `tanggal_estimasi`. Tidak ada migration yang menambahkan kolom `deadline`.
- **Dampak:** **Dashboard untuk role `super_admin` dan `admin_cabang` akan error SQL.** Fitur "Order Perlu Perhatian" dan "Tugas Aktif" tidak bisa tampil.
- **Perbaikan:** Ganti semua referensi `deadline` menjadi `tanggal_estimasi` di DashboardController.

### BUG-02: Semua Laporan Keuangan Crash — Kolom `total_bersih` Tidak Ada

- **Repo:** `konveksio-api`
- **File:** `app/Http/Controllers/Api/V1/Laporan/LaporanController.php` (baris 54, 142, 210)
- **Masalah:** Menggunakan `gaji.total_bersih` tetapi migration gaji mendefinisikan kolom `gaji_bersih`.
- **Dampak:** **Semua endpoint laporan crash** — `/laporan/ringkasan`, `/laporan/pengeluaran`, `/laporan/profit`, dan dashboard admin yang memanggil data gaji.
- **Perbaikan:** Ganti semua `gaji.total_bersih` menjadi `gaji.gaji_bersih`.

### BUG-03: Pembayaran DP Gagal — Status Enum Lama

- **Repo:** `konveksio-api`
- **File:** `app/Http/Controllers/Api/V1/Invoice/PembayaranController.php` (baris 68)
- **Masalah:** Setelah pembayaran, order status di-set ke `'produksi'` (enum lama). Enum baru hasil migration `2026_06_09_000004` hanya menerima: `pending, dp_diterima, diproduksi, siap_kirim, pelunasan, dikirim, selesai`.
- **Dampak:** **Setiap pembayaran pertama pada order berstatus `pending` akan gagal** — database constraint violation.
- **Perbaikan:** Ganti `'produksi'` menjadi `'dp_diterima'` (sesuai flow SOT: pembayaran pertama = DP Diterima).

### BUG-04: Chart Pendapatan Kosong — Key API Tidak Sesuai

- **Repo:** `konveksio-mobile`
- **File:** `lib/features/dashboard/presentation/widgets/revenue_chart_widget.dart` (baris 37)
- **Masalah:** Frontend membaca `data[i]['total_pendapatan']` tetapi API `/laporan/pendapatan` mengembalikan objek dengan key `total` (bukan `total_pendapatan`).
- **Dampak:** **Grafik pendapatan di dashboard selalu kosong** — semua nilai dibaca sebagai 0.
- **Perbaikan:** Ganti `data[i]['total_pendapatan']` menjadi `data[i]['total']`.

---

## 🟡 BUG TINGGI PRIORITAS

### BUG-05: PIN Disimpan Plain Text

- **Repo:** `konveksio-api`
- **File:** `app/Http/Controllers/Api/V1/Auth/AuthController.php` (baris 50)
- **Masalah:** PIN di-query langsung `where('pin', $request->pin)` tanpa hashing. PIN tersimpan readable di database.
- **Dampak:** **Vulnerabilitas keamanan** — siapa saja yang punya akses DB bisa baca PIN semua user.
- **Perbaikan:** Hash PIN saat simpan (`Hash::make`) dan gunakan `Hash::check` saat verifikasi.

### BUG-06: RoleController Method Kosong

- **Repo:** `konveksio-api`
- **File:** `app/Http/Controllers/Api/V1/User/RoleController.php` (baris 23-50)
- **Masalah:** Method `store()`, `show()`, `update()`, `destroy()` kosong — return `null` (HTTP 200 kosong).
- **Dampak:** CRUD role selain `index` tidak berfungsi, response kosong tanpa error.
- **Perbaikan:** Implementasi method atau hapus route yang tidak digunakan.

### BUG-07: Offline Sync Silent Failure

- **Repo:** `konveksio-mobile`
- **File:** `lib/core/services/offline_sync_service.dart` (baris 52, 69)
- **Masalah:** `catch` block kosong — error ditelan tanpa logging.
- **Dampak:** **Tidak mungkin debug masalah offline sync.** User tidak tahu data gagal tersimpan.
- **Perbaikan:** Tambahkan logging (`debugPrint` atau `logger`) di catch block.

### BUG-08: BuildContext Across Async Gap

- **Repo:** `konveksio-mobile`
- **File:** `lib/features/produksi/presentation/handover_screen.dart` (baris 95-100)
- **Masalah:** Menggunakan `ScaffoldMessenger.of(context)` setelah `await` dengan guard `mounted` yang tidak terkait.
- **Dampak:** Potensi crash jika widget di-dispose saat API call masih berjalan.
- **Perbaikan:** Gunakan pattern `if (!mounted) return;` sebelum akses context.

---

## 🟡 POTENSI UI OVERFLOW

### OVERFLOW-01: HandoverScreen — Row Tanpa Expanded

- **File:** `lib/features/produksi/presentation/handover_screen.dart` (baris 148-175)
- **Masalah:** `Row` berisi `Text` label dan `Text` value tanpa `Expanded`/`Flexible`. Nama produk atau nomor order panjang akan overflow.
- **Perbaikan:** Bungkus value `Text` dengan `Expanded` + `TextOverflow.ellipsis`.

### OVERFLOW-02: AttentionOrderList — Row Text Bersebelahan

- **File:** `lib/features/dashboard/presentation/widgets/attention_order_list.dart` (baris 138, 299)
- **Masalah:** `Row` berisi kode order `Text` dan deadline `Text` tanpa `Expanded`. Kode order panjang bisa overflow.
- **Perbaikan:** Bungkus kode order dengan `Expanded` atau `Flexible`.

### OVERFLOW-03: WalletCard — Angka Finansial Besar

- **File:** `lib/features/dashboard/presentation/widgets/wallet_card.dart` (baris 94-101)
- **Masalah:** `Text` menampilkan angka keuangan tanpa `TextOverflow.ellipsis` atau `maxLines`. Angka miliaran (13+ digit) bisa overflow di layar kecil.
- **Perbaikan:** Tambahkan `maxLines: 1, overflow: TextOverflow.ellipsis` atau format angka (Rp 1,2M).

---

## Pelanggaran SOT: Service Pattern

SOT `CODING_STANDARD.md` mewajibkan **Service Pattern** — controller hanya validasi, panggil service, return response.

| Controller | Status | Keterangan |
|------------|--------|------------|
| GajiController | ✅ Sesuai | Menggunakan `GajiService` |
| OrderController | ❌ Melanggar | Semua business logic di controller |
| ProduksiController | ❌ Melanggar | Semua business logic di controller |
| HandoverController | ❌ Melanggar | Semua business logic di controller |
| PembayaranController | ❌ Melanggar | Logic pembayaran + WA di controller |
| KasbonController | ❌ Melanggar | Logic limit mingguan di controller |
| LaporanController | ❌ Melanggar | Semua query aggregasi di controller |
| DashboardController | ❌ Melanggar | Semua query dashboard di controller |
| AuthController | ❌ Melanggar | Logic auth langsung di controller |
| UserController | ❌ Melanggar | Logic CRUD user di controller |
| PengaturanController | ❌ Melanggar | Logic settings di controller |
| Lainnya (11 controller) | ❌ Melanggar | Pola yang sama |

**Kepatuhan: 1/23 controller (~4%)**

---

## Pelanggaran SOT: Batas 300 Baris File Flutter

SOT `CODING_STANDARD.md` mewajibkan file Flutter **maksimal 300 baris**.

| File | Baris | Selisih |
|------|-------|---------|
| `order_form_screen.dart` | 402 | +102 |
| `tarif_borongan_screen.dart` | 381 | +81 |
| `transaksi_cabang_screen.dart` | 381 | +81 |
| `attention_order_list.dart` | 372 | +72 |
| `app_router.dart` | 371 | +71 |
| `pembayaran_form_screen.dart` | 359 | +59 |
| `laporan_screen.dart` | 349 | +49 |
| `order_item_card.dart` | 334 | +34 |
| `proses_gaji_bottom_sheet.dart` | 315 | +15 |
| `slip_gaji_screen.dart` | 302 | +2 |

**10 file melebihi batas. Refactoring dibutuhkan** — pisahkan ke widget terpisah.

---

## Peringatan `dart analyze` (7 Warning)

| Tipe | File | Keterangan |
|------|------|------------|
| `duplicate_import` | `core/router/app_router.dart:12` | Import duplikat |
| `unused_import` | `dashboard_providers.dart:2` | `dio_client.dart` tidak dipakai |
| `unused_import` | `revenue_chart_widget.dart:4` | `dio_client.dart` tidak dipakai |
| `unused_import` | `order_item_card.dart:8` | `auth_storage.dart` tidak dipakai |
| `unused_import` | `kelola_user_role_screen.dart:3` | `dio_client.dart` tidak dipakai |
| `unused_import` | `pengaturan_invoice_screen.dart:3` | `dio_client.dart` tidak dipakai |
| `unused_import` | `edit_profile_screen.dart:4` | `dio_client.dart` tidak dipakai |

---

## API Deprecated (20+ Instansi)

| API Lama | API Baru | Jumlah |
|----------|----------|--------|
| `Color.withOpacity(x)` | `Color.withValues(alpha: x)` | 5 instansi |
| `DropdownButtonFormField(value: x)` | `DropdownButtonFormField(initialValue: x)` | 10+ instansi |

---

## Prioritas Perbaikan

### Fase 1 — Critical Fix (Wajib sebelum deploy)
1. ✅ Fix BUG-01: Ganti `deadline` → `tanggal_estimasi` di DashboardController
2. ✅ Fix BUG-02: Ganti `gaji.total_bersih` → `gaji.gaji_bersih` di LaporanController
3. ✅ Fix BUG-03: Ganti `'produksi'` → `'dp_diterima'` di PembayaranController
4. ✅ Fix BUG-04: Ganti `total_pendapatan` → `total` di revenue_chart_widget.dart

### Fase 2 — Security & Stability
5. Hash PIN di AuthController
6. Implementasi atau hapus method kosong di RoleController
7. Tambahkan logging di OfflineSyncService
8. Fix BuildContext async gap di HandoverScreen

### Fase 3 — UI & Code Quality
9. Fix 3 potensi overflow (HandoverScreen, AttentionOrderList, WalletCard)
10. Bersihkan 7 unused/duplicate import
11. Ganti deprecated API (`withOpacity` → `withValues`, `value` → `initialValue`)

### Fase 4 — SOT Compliance (Refactor besar)
12. Ekstrak business logic ke Service classes (22 controller)
13. Refactor 10 file Flutter > 300 baris ke widget terpisah
