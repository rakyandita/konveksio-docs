# 📐 Coding Standard — Konveksio

> **Versi:** 2.0 (Modular Refactor)
> **Sumber:** `docs/design_system.md` Bagian 12 + Penambahan Aturan Teknis

---

## 12. Standar Pencegahan Overflow & Adaptabilitas UI (Flutter)

Untuk memastikan UI responsif dan bebas overflow pada berbagai ukuran layar perangkat mobile, panduan berikut wajib dipatuhi:

### 12.1 Penggunaan Text (Label/Value) dalam Row
- Teks dinamis yang berada dalam baris mendatar (Row) wajib dibungkus dengan `Expanded` atau `Flexible`.
- Gunakan `maxLines: 1` (atau batasan lain yang sesuai) dan `overflow: TextOverflow.ellipsis` agar teks yang terlalu panjang dipotong dan ditandai dengan elipsis (...).
- **Contoh Benar**:
  ```dart
  Expanded(
    child: Text(
      'Teks sangat panjang yang rentan memicu overflow',
      maxLines: 1,
      overflow: TextOverflow.ellipsis,
    ),
  )
  ```

### 12.2 Form Inputs & Dropdowns
- Widget Dropdown, Text Field, atau Input Form lainnya di dalam baris yang sejajar dengan teks/ikon lain harus berada di dalam `Expanded` untuk mengisi ruang sisa secara dinamis tanpa melampaui lebar layar.

### 12.3 Action Buttons & Quick Menu
- Sekumpulan Action Buttons (ikon dan teks berjajar horizontal) disarankan menggunakan `Wrap` daripada `Row` jika jumlahnya dinamis, atau menggunakan rasio pembagian (misal: grid atau flex layout yang setara) bila posisinya tetap.
- Jika menggunakan `Row`, pastikan ada penggunaan `Expanded` pada tiap elemen atau membungkus container dengan `FittedBox`.

### 12.4 Ruang Kritis (Layar Sempit < 360dp)
- Hindari hardcoding lebar kontainer (misal: `width: 300`) secara mutlak jika kontainer tersebut adalah turunan langsung layar. Gunakan `constraints` atau biarkan elemen beradaptasi terhadap *parent constraints*.

---

## Aturan Backend Laravel

### Service Pattern (Wajib)
- Wajib menggunakan pattern `Service` untuk memisahkan logika bisnis dari `Controller`.
- Setiap `Controller` hanya boleh berisi:
  1. Validasi request
  2. Pemanggilan `Service`
  3. Return response
- **Contoh Benar:**
  ```php
  // OrderController.php
  public function store(OrderRequest $request, OrderService $service)
  {
      $order = $service->createOrder($request->validated());
      return response()->json(['order' => $order], 201);
  }
  ```
- **Contoh Salah:** Menempatkan logika kalkulasi HPP, perubahan status, atau query kompleks langsung di dalam Controller.

### Clean Code & Strict Validation
- **No Fat Files:** Batas maksimum file di backend Laravel adalah **300 baris**. Jika lebih, pisahkan ke Trait, Service, atau Helper terpisah.
- **Efisiensi:** Dilarang ada perulangan (loops) yang tidak berguna. Hindari N+1 query problem menggunakan *Eager Loading*.
- **Zero Issue:** Kode harus benar-benar bersih (*clean*) tanpa *linting issues* atau potensi *unhandled exceptions*. Pastikan sistem jalan langsung jadi tanpa butuh *debugging* panjang.
- **Maintainability & Logging:** Harus mudah di-*maintenance* dengan penulisan *error log* yang sangat jelas agar jika terjadi *crash*, penyebabnya mudah dilacak.

---

## Aturan Flutter UI

### Batas Panjang File (Hard Limit 300 Baris)
- File UI Flutter maupun file logika apa pun yang melebihi **300 baris** **WAJIB** dipecah (refactor) menjadi bagian-bagian terpisah.
- Strategi pemecahan:
  1. Ekstrak bagian besar menjadi `StatelessWidget` atau `StatefulWidget` tersendiri.
  2. Simpan widget yang dapat digunakan ulang di folder `widgets/` dalam feature folder yang sama.
  3. Hindari file UI monolitik yang menangani terlalu banyak state sekaligus.

### State Management (Riverpod)
- Gunakan **Riverpod** sebagai arsitektur State Management standar aplikasi Flutter Konveksio.

### UI/UX Copywriting (Bahasa Konveksi)
- Pastikan semua label UI, placeholder, dan *error message* menggunakan bahasa konveksi yang mudah dipahami *end-user* (misal: "Barang Cacat", "Kuitansi", "Potong Gaji", dsb), BUKAN bahasa teknis *developer*.
