# 🌐 API Standard — Konveksio

> **Versi:** 1.0
> **Dibuat:** 13 Juni 2026
> **Status:** MANDATORY — Semua endpoint WAJIB mengikuti standar ini

---

## 1. Response Envelope (Wajib)

Setiap response dari API **WAJIB** menggunakan format envelope berikut. Tidak ada pengecualian.

### 1.1 Success Response

```json
{
  "success": true,
  "message": "Order berhasil dibuat",
  "data": { ... }
}
```

### 1.2 Error Response

```json
{
  "success": false,
  "message": "Validasi gagal",
  "errors": {
    "nama_pelanggan": ["Nama pelanggan wajib diisi"],
    "total_harga": ["Total harga harus berupa angka"]
  }
}
```

### 1.3 Aturan Envelope

| Field | Tipe | Wajib | Keterangan |
|---|---|---|---|
| `success` | boolean | ✅ | `true` jika berhasil, `false` jika gagal |
| `message` | string | ✅ | Pesan singkat dalam **Bahasa Indonesia** yang user-friendly |
| `data` | object/array/null | ✅ | Payload utama. `null` jika tidak ada data (misal: delete) |
| `errors` | object/null | ⚠️ | Hanya ada saat `success: false`. Key = nama field, Value = array pesan error |
| `meta` | object/null | ❌ | Opsional, digunakan untuk pagination |

---

## 2. HTTP Status Codes

| Code | Kapan Digunakan | Contoh |
|---|---|---|
| **200** | Success — Read, Update, Delete | GET list, GET detail, PUT, PATCH, DELETE |
| **201** | Success — Create (resource baru dibuat) | POST store |
| **400** | Bad Request — Error logika bisnis | Stok tidak cukup, limit kasbon tercapai |
| **401** | Unauthorized — Token tidak valid/expired | Token JWT expired, token tidak ada |
| **403** | Forbidden — User tidak punya hak akses | Staff mencoba akses CRUD order |
| **404** | Not Found — Resource tidak ditemukan | Order ID tidak exist |
| **422** | Unprocessable Entity — Validasi gagal | FormRequest validation errors |
| **500** | Internal Server Error — Bug di server | Exception tidak tertangani (harusnya tidak terjadi) |

### Aturan Tambahan
- **DELETE** tetap return `200` dengan `data: null` dan message konfirmasi
- **Login** return `200` dengan `data: { token, user }`
- **401** return body: `{ "success": false, "message": "Token tidak valid atau sudah expired" }`
- **500** JANGAN pernah expose stack trace ke client. Log ke server, return message generik

---

## 3. Pagination

### 3.1 Request

```
GET /api/v1/orders?page=1&per_page=15
```

| Parameter | Tipe | Default | Keterangan |
|---|---|---|---|
| `page` | integer | 1 | Halaman yang diminta |
| `per_page` | integer | 15 | Jumlah item per halaman (max: 100) |

### 3.2 Response

```json
{
  "success": true,
  "message": "Daftar order",
  "data": [ ... ],
  "meta": {
    "current_page": 1,
    "per_page": 15,
    "total": 142,
    "last_page": 10,
    "from": 1,
    "to": 15
  }
}
```

### 3.3 Implementasi

Gunakan method `paginate()` bawaan Laravel. Response `meta` otomatis tersedia dari Laravel paginator.

```php
$orders = Order::latest()->paginate($request->get('per_page', 15));
return response()->json([
    'success' => true,
    'message' => 'Daftar order',
    'data' => OrderResource::collection($orders),
    'meta' => [
        'current_page' => $orders->currentPage(),
        'per_page' => $orders->perPage(),
        'total' => $orders->total(),
        'last_page' => $orders->lastPage(),
        'from' => $orders->firstItem(),
        'to' => $orders->lastItem(),
    ],
]);
```

---

## 4. Filter & Sort

### 4.1 Filter Convention

Gunakan query parameter langsung (bukan nested):

```
GET /api/v1/orders?status=pending&customer_id=5&cabang_id=2
GET /api/v1/kasbon?status=approved&karyawan_id=10
GET /api/v1/orders?date_from=2026-06-01&date_to=2026-06-30
```

| Parameter | Keterangan |
|---|---|
| `status` | Filter by status field |
| `customer_id` | Filter by foreign key |
| `cabang_id` | Filter by cabang (untuk Super Admin) |
| `date_from` | Filter tanggal mulai (format: YYYY-MM-DD) |
| `date_to` | Filter tanggal sampai (format: YYYY-MM-DD) |
| `search` | Pencarian teks bebas (nama, nomor, dll.) |

### 4.2 Sort Convention

```
GET /api/v1/orders?sort=-created_at
GET /api/v1/orders?sort=total_harga
GET /api/v1/orders?sort=-deadline,nama_pelanggan
```

| Prefix | Arti | Contoh |
|---|---|---|
| `-` | Descending (terbesar/terbaru dulu) | `-created_at` = terbaru dulu |
| (tanpa prefix) | Ascending | `nama` = A-Z |
| Comma-separated | Multi-sort | `-deadline,nama` |

### 4.3 Default Sort

Jika parameter `sort` tidak diberikan, gunakan default per endpoint:
- Order: `-created_at` (terbaru)
- Pembayaran: `-tanggal` (terbaru)
- Kasbon: `-tanggal` (terbaru)
- Pelanggan: `nama` (A-Z)
- Vendor: `nama` (A-Z)
- Karyawan: `nama` (A-Z)

---

## 5. API Versioning

### 5.1 Strategi

- **URL Prefix:** `/api/v1/`, `/api/v2/`, dst.
- Saat ini: semua endpoint di bawah `/api/v1/`
- Namespace controller: `App\Http\Controllers\Api\V1\`

### 5.2 Aturan Breaking Change

Buat versi baru (`v2`) **HANYA** jika:
- Mengubah struktur response envelope
- Menghapus atau rename field yang sudah dipakai client
- Mengubah authentication mechanism

**TIDAK PERLU** versi baru untuk:
- Menambah field baru di response (backward compatible)
- Menambah endpoint baru
- Menambah filter parameter

### 5.3 Deprecation

Jika versi lama akan dihentikan:
1. Return header `X-API-Deprecated: true` di response lama
2. Berikan waktu minimal **3 bulan** sebelum versi lama dimatikan
3. Dokumentasikan di changelog

---

## 6. Authentication Endpoints

### 6.1 Format Request

Semua endpoint yang dilindungi (`auth:api` middleware) **WAJIB** menerima token JWT di header:

```
Authorization: Bearer <token>
```

### 6.2 Login Response

```json
{
  "success": true,
  "message": "Login berhasil",
  "data": {
    "token": "eyJ0eXAiOiJKV1QiLCJhbGci...",
    "token_type": "bearer",
    "expires_in": 3600,
    "user": {
      "id": 1,
      "nama": "Budi Santoso",
      "email": "budi@konveksio.com",
      "role": "super_admin",
      "cabang_id": 1,
      "company_id": 1
    }
  }
}
```

### 6.3 Token Management

| Aspek | Nilai |
|---|---|
| Token TTL | 60 menit |
| Refresh Token TTL | 7 hari |
| Refresh endpoint | `POST /api/v1/auth/refresh` |
| Logout | `POST /api/v1/auth/logout` (blacklist token) |

---

## 7. Public Endpoints (Tanpa Auth)

Endpoint yang bisa diakses tanpa login:

| Method | Path | Keterangan |
|---|---|---|
| `GET` | `/invoice/{code}` | Public link invoice (blade view, bukan JSON) |

Rate limit untuk public endpoint: **30 request / menit per IP**.

---

## 8. File Upload

### 8.1 Aturan Upload

| Aspek | Nilai |
|---|---|
| Max file size | **5 MB** |
| Allowed MIME types | `jpg`, `jpeg`, `png`, `pdf` |
| Storage | `storage/app/public/` (accessible via symlink) |
| Filename | UUID + extension (`a1b2c3d4-e5f6.jpg`) — JANGAN pakai nama asli |
| Response | Return path relatif: `"data": { "file_path": "bukti/2026/06/uuid.jpg" }` |

### 8.2 Folder Struktur Upload

```
storage/app/public/
├── bukti/          ← Bukti pembayaran (kuitansi)
│   └── 2026/06/
├── desain/         ← File desain order
│   └── 2026/06/
├── foto/           ← Foto profil karyawan
└── dokumen/        ← Dokumen lainnya
```

---

## 9. API Resource (Response Transform)

### 9.1 Kewajiban

- Untuk response **collection** (list), WAJIB menggunakan `JsonResource` collection
- Untuk response **single item**, boleh return array langsung jika sederhana

### 9.2 Contoh

```php
// Controller
public function index(Request $request, OrderService $service)
{
    $orders = $service->getOrders($request);
    return response()->json([
        'success' => true,
        'message' => 'Daftar order',
        'data' => OrderResource::collection($orders),
        'meta' => [
            'current_page' => $orders->currentPage(),
            'per_page' => $orders->perPage(),
            'total' => $orders->total(),
            'last_page' => $orders->lastPage(),
        ],
    ]);
}
```

```php
// OrderResource.php
public function toArray($request)
{
    return [
        'id' => $this->id,
        'nomor_order' => $this->nomor_order,
        'status' => $this->status,
        'total_harga' => $this->total_harga,
        'deadline' => $this->deadline,
        'customer' => [
            'id' => $this->customer->id,
            'nama' => $this->customer->nama,
        ],
        'created_at' => $this->created_at->toIso8601String(),
    ];
}
```

---

## 10. Date & Time Format

| Konteks | Format | Contoh |
|---|---|---|
| API Response (datetime) | ISO 8601 | `2026-06-13T14:30:00+07:00` |
| API Response (date only) | YYYY-MM-DD | `2026-06-13` |
| API Request (date filter) | YYYY-MM-DD | `?date_from=2026-06-01` |
| Database | Laravel default | `2026-06-13 14:30:00` |
| Display (frontend) | DD MMM YYYY | `13 Jun 2026` |
| Currency | Rp X.XXX | `Rp 1.500.000` |

---

## 11. CORS Policy

- **Allowed Origins:** `http://localhost:*` (dev), `https://*.konveksio.com` (prod)
- **Allowed Methods:** `GET, POST, PUT, PATCH, DELETE, OPTIONS`
- **Allowed Headers:** `Authorization, Content-Type, Accept, X-Requested-With`
- **Credentials:** `true`

Konfigurasi di `config/cors.php`.
