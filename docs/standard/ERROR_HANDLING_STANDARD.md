# 🛡️ Error Handling Standard — Konveksio

> **Versi:** 1.0
> **Dibuat:** 13 Juni 2026
> **Status:** MANDATORY — Semua Service dan Controller WAJIB mengikuti standar ini

---

## 1. Prinsip Dasar

1. **Jangan pernah expose stack trace ke client** — log di server, return pesan generik
2. **Selalu return response envelope** — bahkan saat error (lihat `API_STANDARD.md`)
3. **Error message dalam Bahasa Indonesia** — user-friendly, bukan teknis
4. **Setiap catch block WAJIB ada log** — tidak boleh silent failure
5. **Fail fast** — validasi di awal, jangan biarkan error merambat

---

## 2. Exception Classes

### 2.1 Gunakan Bawaan Laravel

| Exception | HTTP Code | Kapan Digunakan |
|---|---|---|
| `ValidationException` | 422 | Otomatis dari FormRequest |
| `ModelNotFoundException` | 404 | Resource tidak ditemukan |
| `AuthenticationException` | 401 | Token tidak valid / expired |
| `AuthorizationException` | 403 | User tidak punya hak akses |
| `HttpException` | varies | Error HTTP generik |

### 2.2 Custom Exception (Jika Perlu)

Buat custom exception **HANYA** jika exception bawaan tidak cukup:

```php
// app/Exceptions/BusinessLogicException.php
namespace App\Exceptions;

use Exception;

class BusinessLogicException extends Exception
{
    protected $errorCode;

    public function __construct(string $message, string $errorCode = 'BUSINESS_ERROR', int $httpCode = 400)
    {
        parent::__construct($message);
        $this->errorCode = $errorCode;
        $this->httpCode = $httpCode;
    }

    public function getErrorCode(): string { return $this->errorCode; }
    public function getHttpCode(): int { return $this->httpCode; }
}
```

Contoh penggunaan:
```php
// Di Service
if ($totalKasbonMingguan + $jumlah > $limitKasbon) {
    throw new BusinessLogicException(
        'Limit kasbon mingguan sudah tercapai. Sisa limit: Rp ' . number_format($sisaLimit),
        'KASBON_LIMIT_EXCEEDED',
        400
    );
}
```

### 2.3 Daftar Custom Exception yang Diizinkan

| Class | Error Code | HTTP Code | Penggunaan |
|---|---|---|---|
| `BusinessLogicException` | `BUSINESS_ERROR` / custom | 400 | Logika bisnis gagal (limit, stok, dsb.) |
| `BranchMismatchException` | `BRANCH_MISMATCH` | 403 | Operasi lintas cabang yang tidak diizinkan |

JANGAN buat lebih dari 5 custom exception. Jika perlu lebih, arsitektur perlu dipertanyakan.

---

## 3. Global Exception Handler

### 3.1 Struktur Handler

Semua exception ditangani di `app/Exceptions/Handler.php` (atau `bootstrap/app.php` untuk Laravel 11+):

```php
use Illuminate\Http\JsonResponse;
use Illuminate\Validation\ValidationException;
use Illuminate\Auth\AuthenticationException;
use Illuminate\Database\Eloquent\ModelNotFoundException;
use App\Exceptions\BusinessLogicException;

// Di bootstrap/app.php -> withExceptions()
$exceptions->render(function (Throwable $e, Request $request) {
    // Hanya handle API request
    if ($request->is('api/*')) {
        return match (true) {
            $e instanceof ValidationException => new JsonResponse([
                'success' => false,
                'message' => 'Validasi gagal',
                'errors' => $e->errors(),
            ], 422),

            $e instanceof ModelNotFoundException => new JsonResponse([
                'success' => false,
                'message' => 'Data tidak ditemukan',
            ], 404),

            $e instanceof AuthenticationException => new JsonResponse([
                'success' => false,
                'message' => 'Token tidak valid atau sudah expired',
            ], 401),

            $e instanceof BusinessLogicException => new JsonResponse([
                'success' => false,
                'message' => $e->getMessage(),
                'errors' => ['code' => $e->getErrorCode()],
            ], $e->getHttpCode()),

            default => $this->handleGenericException($e),
        };
    }
});
```

### 3.2 Generic Exception Handler

```php
private function handleGenericException(Throwable $e): JsonResponse
{
    // Log detail untuk developer
    Log::error('Unhandled Exception', [
        'message' => $e->getMessage(),
        'file' => $e->getFile(),
        'line' => $e->getLine(),
        'trace' => $e->getTraceAsString(),
    ]);

    // Response generik untuk client
    return new JsonResponse([
        'success' => false,
        'message' => app()->environment('production')
            ? 'Terjadi kesalahan pada server. Silakan coba lagi.'
            : $e->getMessage(), // Di dev, tampilkan pesan asli untuk debugging
    ], 500);
}
```

---

## 4. Logging Strategy

### 4.1 Level Log

| Level | Kapan Digunakan | Contoh |
|---|---|---|
| `error` | Exception, crash, bug tak terduga | Unhandled exception, DB connection failed |
| `warning` | Kondisi abnormal tapi tidak crash | Login gagal 5x, file upload terlalu besar |
| `info` | Event bisnis penting | Order dibuat, pembayaran diterima, gaji diproses |
| `debug` | Informasi untuk development | Query detail, request payload (HANYA di local) |

### 4.2 Aturan Log

```php
// ✅ BENAR — Context lengkap
Log::error('Gagal membuat order', [
    'user_id' => auth()->id(),
    'data' => $validatedData,
    'error' => $e->getMessage(),
]);

// ❌ SALAH — Tidak ada context
Log::error('Error terjadi');

// ❌ SALAH — Log di production untuk debug
Log::debug('Request data: ' . json_encode($request->all()));
```

### 4.3 Channel Log

| Channel | File | Untuk Apa |
|---|---|---|
| `single` (default) | `storage/logs/laravel.log` | Semua log umum |
| `daily` | `storage/logs/laravel-YYYY-MM-DD.log` | Production (rotasi harian) |
| `stderr` | stdout | Docker / containerized deployment |

Konfigurasi di `.env`:
```
LOG_CHANNEL=daily
LOG_LEVEL=error
```

### 4.4 Log untuk Transaksi Finansial

Setiap operasi yang melibatkan uang **WAJIB** di-log:

```php
Log::info('Pembayaran dibuat', [
    'order_id' => $order->id,
    'user_id' => auth()->id(),
    'jenis' => $data['jenis'],
    'jumlah' => $data['jumlah'],
    'metode' => $data['metode'],
]);

Log::info('Gaji diproses', [
    'karyawan_id' => $karyawan->id,
    'periode' => $periode,
    'gaji_kotor' => $gajiKotor,
    'total_kasbon' => $totalKasbon,
    'gaji_bersih' => $gajiBersih,
]);
```

---

## 5. Retry Strategy

### 5.1 Operasi Eksternal

Untuk service yang memanggil API eksternal (WhatsApp, FCM, payment gateway):

```php
use Illuminate\Support\Facades\Http;

// Dengan retry
$response = Http::retry(3, 1000) // 3x retry, delay 1 detik
    ->timeout(10) // timeout 10 detik
    ->post('https://api.fonnte.com/send', [
        'target' => $phone,
        'message' => $message,
    ]);

if ($response->failed()) {
    Log::warning('Gagal kirim WA notification', [
        'phone' => $phone,
        'response' => $response->body(),
    ]);
    // Jangan throw exception — notif gagal bukan error kritis
}
```

### 5.2 Aturan Retry

| Service | Max Retry | Delay | Fallback |
|---|---|---|---|
| WhatsApp API | 3x | 1s, 2s, 4s (exponential) | Log warning, jangan block flow |
| FCM Push | 3x | 1s | Log warning, notif tidak terkirim bukan error |
| File upload | 1x | — | Return error ke user, minta retry manual |
| Database | 0x | — | Throw exception, rollback transaction |

---

## 6. Transaction Handling

### 6.1 Kewajiban DB::transaction

Setiap operasi yang mengubah **lebih dari 1 tabel** WAJIB dibungkus `DB::transaction`:

```php
use Illuminate\Support\Facades\DB;

public function createOrder(array $data): Order
{
    return DB::transaction(function () use ($data) {
        $order = Order::create($data['order']);

        foreach ($data['items'] as $item) {
            OrderItem::create(array_merge($item, ['order_id' => $order->id]));
        }

        Invoice::create([
            'order_id' => $order->id,
            'nomor_invoice' => $this->generateInvoiceNumber(),
        ]);

        return $order->load('orderItems', 'invoice');
    });
}
```

### 6.2 Aturan Transaction

- JANGAN tangkap exception di dalam transaction — biarkan rollback otomatis
- JANGAN ada operasi non-DB (kirim email, WA, FCM) di dalam transaction
- Kirim notifikasi **setelah** transaction berhasil commit

```php
// ✅ BENAR
$order = DB::transaction(function () use ($data) {
    // ... semua operasi DB
});

// Notifikasi DI LUAR transaction
$this->sendNotification($order);

// ❌ SALAH
DB::transaction(function () use ($data) {
    $order = Order::create($data);
    $this->sendWhatsApp($order); // ← JANGAN! Jika WA gagal, order ikut rollback
});
```

---

## 7. Error di Controller

### 7.1 Jangan Try-Catch di Controller

Controller TIDAK BOLEH try-catch. Biarkan Exception Handler yang menangani:

```php
// ✅ BENAR — Clean controller
public function store(StoreOrderRequest $request, OrderService $service)
{
    $order = $service->createOrder($request->validated());
    return response()->json([
        'success' => true,
        'message' => 'Order berhasil dibuat',
        'data' => new OrderResource($order),
    ], 201);
}

// ❌ SALAH — Try-catch di controller (redundant)
public function store(Request $request)
{
    try {
        $order = Order::create($request->all());
        return response()->json(['data' => $order]);
    } catch (\Exception $e) {
        return response()->json(['error' => $e->getMessage()], 500);
    }
}
```

### 7.2 Pengecualian

Try-catch di controller **DIPERBOLEHKAN** hanya jika:
- Ada logika fallback yang spesifik (misal: coba cache, fallback ke DB)
- Operasi yang memang expected gagal (misal: external API call)

---

## 8. Error di Service

### 8.1 Throw, Jangan Return Error

```php
// ✅ BENAR — Throw exception
public function ajukanKasbon(User $user, float $jumlah): Kasbon
{
    if ($this->cekLimitMingguan($user) < $jumlah) {
        throw new BusinessLogicException('Limit kasbon mingguan sudah tercapai');
    }

    return Kasbon::create([...]);
}

// ❌ SALAH — Return error array
public function ajukanKasbon(User $user, float $jumlah): array
{
    if ($this->cekLimitMingguan($user) < $jumlah) {
        return ['success' => false, 'message' => 'Limit tercapai'];
    }

    return ['success' => true, 'data' => Kasbon::create([...])];
}
```

### 8.2 Custom Error Code

Untuk error yang perlu ditangani berbeda di frontend, gunakan error code:

```php
throw new BusinessLogicException(
    'Limit kasbon mingguan sudah tercapai. Sisa limit: Rp 0',
    'KASBON_LIMIT_EXCEEDED',  // Frontend bisa cek code ini
    400
);
```

Frontend bisa cek `errors.code` untuk menampilkan UI yang berbeda:
```dart
if (response.data['errors']?['code'] == 'KASBON_LIMIT_EXCEEDED') {
    showLimitReachedDialog();
}
```
