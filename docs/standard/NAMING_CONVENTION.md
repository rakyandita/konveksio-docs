# 🏷️ Naming Convention — Konveksio

> **Versi:** 1.0
> **Dibuat:** 13 Juni 2026
> **Status:** MANDATORY — Semua file, variabel, kolom, dan route WAJIB mengikuti konvensi ini

---

## 1. Bahasa

### 1.1 Aturan Umum
- **Kode (variable, function, class):** Bahasa **Inggris** (`getOrder`, `calculateTotal`)
- **Database kolom:** Bahasa **Indonesia** (`nama`, `cabang_id`, `tanggal`, `total_harga`)
- **UI label & pesan error:** Bahasa **Indonesia** ("Order berhasil dibuat", "Nama wajib diisi")
- **Komentar kode:** Bahasa **Indonesia** (untuk bisnis), Inggris (untuk teknis)

### 1.2 Pengecualian
- Nama library/framework bawaan mengikuti konvensi masing-masing (Laravel, Flutter)
- Kata yang sudah umum di industri tidak diterjemahkan: `order`, `invoice`, `status`, `token`, `filter`, `sort`

---

## 2. Database

### 2.1 Tabel
- **Plural, snake_case, Bahasa Indonesia**
- Contoh: `orders`, `order_items`, `tahap_produksis`, `log_handovers`, `porsi_pekerjaans`
- Tabel pivot (jika ada): alphabetical order — `branch_user` bukan `user_branch`

### 2.2 Kolom
- **snake_case, Bahasa Indonesia**
- Contoh: `nama`, `email`, `no_whatsapp`, `tanggal_lahir`, `total_harga`

### 2.3 Foreign Key
- Format: `{nama_entitas_singular}_id`
- Contoh: `order_id`, `customer_id`, `cabang_id`, `company_id`, `vendor_id`
- WAJIB ada foreign key constraint di migration (kecuali alasan performa yang terdokumentasi)

### 2.4 Boolean
- Prefix `is_` + snake_case
- Contoh: `is_active`, `is_paid_rework`, `is_default`
- Type: `boolean` di migration, bukan integer 0/1

### 2.5 Timestamp
- Ikuti Laravel default: `created_at`, `updated_at`, `deleted_at`
- Untuk timestamp custom bisnis: `tanggal_order`, `tanggal_kirim`, `tanggal_approve`
- Format di database: `datetime` (bukan unix timestamp)

### 2.6 Soft Delete
- **WAJIB** untuk entitas transaksional: `Order`, `Pembayaran`, `Kasbon`, `Invoice`, `Pengeluaran`, `MutasiCabang`
- **OPSIONAL** untuk entitas master: `Customer`, `Vendor`, `Product`, `User` (lebih baik pakai `is_active = false`)
- Migration: gunakan `softDeletes()` helper Laravel

### 2.7 Enum / Status
- Simpan sebagai `string` (bukan MySQL ENUM), agar mudah ditambah nilainya
- Contoh kolom: `status` dengan nilai `pending`, `dp_diterima`, `diproduksi`, `selesai`
- Definisikan konstanta di Model untuk validasi:
  ```php
  const STATUS_SELESAI = 'selesai';
  const VALID_STATUSES = ['pending', 'dp_diterima', 'diproduksi', 'selesai'];
  ```

---

## 3. Model (Laravel)

### 3.1 Nama Model
- **Singular, PascalCase, Bahasa Indonesia/Inggris sesuai domain**
- Contoh: `Order`, `OrderItem`, `TahapProduksi`, `LogHandover`, `PorsiPekerjaan`, `MutasiCabang`

### 3.2 Properti Model
```php
// snake_case mengikuti kolom database
protected $fillable = ['nama', 'email', 'cabang_id', 'total_harga'];
protected $casts = [
    'is_active' => 'boolean',
    'tanggal_order' => 'date',
    'total_harga' => 'decimal:2',
];
```

### 3.3 Relationship Method
- **camelCase, Bahasa Inggris deskriptif**
- HasMany/BelongsToMany: plural (`orderItems`, `pembayarans`)
- BelongsTo: singular (`customer`, `cabang`, `order`)
- Contoh:
  ```php
  public function orderItems() { return $this->hasMany(OrderItem::class); }
  public function customer() { return $this->belongsTo(Customer::class); }
  public function cabang() { return $this->belongsTo(Branch::class, 'cabang_id'); }
  ```

### 3.4 Scope Method
- Prefix `scope` + PascalCase
- Contoh: `scopeActive()`, `scopeByCabang()`, `scopeSelesai()`

### 3.5 Accessor / Mutator
- camelCase mengikuti Laravel convention
- Contoh: `getFullNameAttribute()`, `setPasswordAttribute()`

---

## 4. Controller

### 4.1 Nama Controller
- Format: `{Model}Controller` — `OrderController`, `CustomerController`, `ProduksiController`
- Namespace: `App\Http\Controllers\Api\V1\`

### 4.2 Method Name
| Method | Nama | Keterangan |
|---|---|---|
| GET (list) | `index` | Daftar resource (paginated) |
| GET (detail) | `show` | Detail 1 resource |
| POST (create) | `store` | Buat resource baru |
| PUT/PATCH (update) | `update` | Update resource |
| DELETE | `destroy` | Hapus resource |
| Custom action | `camelCase` deskriptif | `approve`, `reject`, `getSummary` |

### 4.3 Controller Structure
```php
public function store(StoreOrderRequest $request, OrderService $service)
{
    $order = $service->createOrder($request->validated());

    return response()->json([
        'success' => true,
        'message' => 'Order berhasil dibuat',
        'data' => new OrderResource($order),
    ], 201);
}
```

---

## 5. Service

### 5.1 Nama Service
- Format: `{Domain}Service` — `OrderService`, `KasbonService`, `ProduksiService`
- Namespace: `App\Services\`

### 5.2 Method Name
- camelCase, verb-first deskriptif
- Contoh:
  ```php
  createOrder(array $data): Order
  updateOrderStatus(Order $order, string $status): Order
  calculateHpp(OrderItem $item): float
  getMonthlySummary(int $cabangId, string $bulan): array
  ajukanKasbon(User $user, float $jumlah): Kasbon
  ```

---

## 6. FormRequest (Validation)

### 6.1 Nama
- Format: `Store{Model}Request` untuk create, `Update{Model}Request` untuk update
- Contoh: `StoreOrderRequest`, `UpdateCustomerRequest`
- Namespace: `App\Http\Requests\`

### 6.2 Rules Method
```php
public function rules(): array
{
    return [
        'nama_pelanggan' => 'required|string|max:255',
        'no_whatsapp' => 'required|string|max:20',
        'total_harga' => 'required|numeric|min:0',
        'cabang_id' => 'required|exists:cabangs,id',
    ];
}

public function messages(): array
{
    return [
        'nama_pelanggan.required' => 'Nama pelanggan wajib diisi',
        'total_harga.required' => 'Total harga wajib diisi',
        'total_harga.min' => 'Total harga tidak boleh negatif',
    ];
}
```

---

## 7. Migration

### 7.1 Nama File
- Format: `YYYY_MM_DD_HHMMSS_{action}_{table}_table.php`
- Action: `create`, `add_*_to`, `rename_*_in`, `drop_*_from`
- Contoh:
  - `2026_01_01_000006_create_orders_table.php`
  - `2026_06_13_105831_add_fcm_token_to_users_table.php`

### 7.2 Urutan Migration
- Tabel master (tanpa FK) dulu: `companies`, `branches`, `users`
- Tabel referensi: `customers`, `products`, `vendors`
- Tabel transaksional (punya FK): `orders`, `order_items`, `pembayarans`

---

## 8. Route

### 8.1 URL Path
- **kebab-case, plural** untuk resource
- Contoh: `/api/v1/orders`, `/api/v1/order-items`, `/api/v1/tahap-produksi`

### 8.2 Route Name
- **dot-notation, plural**
- Contoh: `orders.index`, `orders.store`, `orders.show`

### 8.3 Grouping
```php
Route::prefix('v1')->middleware('auth:api')->group(function () {
    Route::apiResource('orders', OrderController::class);
    Route::apiResource('customers', CustomerController::class);
});
```

---

## 9. Flutter / Mobile

### 9.1 File Name
- **snake_case** untuk file Dart
- Contoh: `order_list_screen.dart`, `order_card_widget.dart`, `auth_service.dart`

### 9.2 Class Name
- **PascalCase** untuk class dan widget
- Screen: `{Name}Screen` — `OrderListScreen`, `DashboardScreen`
- Widget: `{Name}Card`, `{Name}Button` — `OrderCard`, `PrimaryButton`
- Service: `{Name}Service` — `OrderService`, `AuthService`
- Provider (Riverpod): `{name}Provider` — `orderListProvider`, `authProvider`

### 9.3 Variable Name
- **camelCase**
- Contoh: `orderList`, `isLoading`, `selectedCustomer`

### 9.4 Constant
- **SCREAMING_SNAKE_CASE**
- Contoh: `API_BASE_URL`, `MAX_FILE_SIZE`

---

## 10. Konsistensi Lintas Platform

### 10.1 JSON Field Name
- Backend return **snake_case** (mengikuti kolom database)
- Flutter menerima snake_case dan convert ke camelCase di model layer
- Contoh flow:
  ```
  DB: total_harga → API JSON: "total_harga" → Flutter Model: totalHarga
  ```

### 10.2 Status Values
- Sama persis di semua layer (backend, mobile, database)
- Contoh: `pending`, `dp_diterima`, `diproduksi`, `siap_kirim`, `selesai`
- JANGAN translate status ke bahasa Inggris di layer manapun

### 10.3 Currency
- Backend simpan: `decimal(15,2)` di database, `float` di PHP
- Flutter simpan: `double` atau `int` (satuan rupiah, tanpa desimal)
- Display: `Rp 1.500.000` (titik pemisah ribuan, tanpa desimal untuk rupiah)

### 10.4 Date Format
- API kirim: ISO 8601 (`2026-06-13T14:30:00+07:00`)
- Flutter parse: `DateTime` object
- Display: `13 Jun 2026` atau `13/06/2026`
