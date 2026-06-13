# 🧪 Testing Standard — Konveksio

> **Versi:** 2.0 (Coverage & Pattern Update)
> **Dibuat:** 11 Juni 2026
> **Diperbarui:** 13 Juni 2026

---

## 1. Aturan Testing Backend (Laravel)

### 1.1 Validasi Routing Endpoint
- Setiap selesai membuat endpoint Laravel baru, agent **WAJIB** memvalidasi bahwa routing tidak error dengan menjalankan:
  ```bash
  php artisan route:list --name=[nama_route]
  ```
- Pastikan tidak ada konflik route atau route yang `Missing` sebelum melanjutkan ke implementasi berikutnya.
- Jika endpoint memerlukan middleware (auth, role), wajib diuji bahwa middleware teraplikasi dengan benar.

### 1.2 Validasi Migrasi Database
- Setelah membuat atau mengubah file migrasi, wajib menjalankan:
  ```bash
  php artisan migrate --pretend
  ```
  untuk memverifikasi SQL yang dihasilkan sebelum menjalankan migrasi sesungguhnya.

---

## 2. Automated Testing (PHPUnit)

### 2.1 Kewajiban Test

| Kategori | Wajib Test? | Minimum Coverage |
|---|---|---|
| **Service layer** (semua logic finansial) | ✅ **WAJIB** | 80% line coverage |
| **Controller** (API endpoint) | ✅ **WAJIB** | 60% (happy path + 1 error path) |
| **Model** (scope, accessor, relationship) | ⚠️ Recommended | Scope yang kompleks |
| **Helper / Utility** | ⚠️ Recommended | Pure function |
| **Migration & Seeder** | ❌ Tidak perlu | Validasi via `migrate --pretend` |
| **View / Blade** | ❌ Tidak perlu | Manual check |

### 2.2 Fitur yang WAJIB Punya Test

Setiap fitur yang menyangkut uang **WAJIB** memiliki Automated Unit Test:

- ✅ Kalkulasi Gaji Borongan (`PayrollService`)
- ✅ Kasbon & limit checking (`KasbonService`)
- ✅ Potongan gaji (`PayrollService`)
- ✅ Kalkulator HPP (`ProduksiService`)
- ✅ Invoice total calculation (`InvoiceService`)
- ✅ Payment & sisa tagihan (`PaymentService`)
- ✅ Laporan Laba Rugi (`FinanceService`)

### 2.3 Test Structure

```
tests/
├── Unit/                    ← Pure logic, tanpa database
│   ├── Helpers/
│   └── Calculations/
├── Feature/                 ← Butuh database & Laravel app
│   ├── Services/            ← Service layer tests
│   ├── Api/                 ← HTTP endpoint tests
│   └── Commands/            ← Artisan command tests (jika ada)
└── TestCase.php             ← Base test class
```

### 2.4 Test Naming Convention

Format: `test_{method}_{scenario}_{expected_result}`

```php
// ✅ BENAR — Nama deskriptif
public function test_ajukanKasbon_within_limit_returns_kasbon(): void
public function test_ajukanKasbon_exceeds_limit_throws_exception(): void
public function test_createOrder_with_bundle_expands_items(): void
public function test_calculateHpp_multiple_stages_returns_total(): void

// ❌ SALAH — Tidak jelas
public function test_kasbon(): void
public function test_order(): void
public function it_works(): void
```

### 2.5 AAA Pattern (Arrange-Act-Assert)

Setiap test method **WAJIB** mengikuti pola AAA:

```php
public function test_ajukanKasbon_within_limit_returns_kasbon(): void
{
    // ARRANGE — Setup data & state
    $user = User::factory()->create(['cabang_id' => 1, 'role' => 'staff_produksi']);
    $jumlah = 200000;

    // ACT — Jalankan logic yang di-test
    $kasbon = $this->service->ajukanKasbon($user, $jumlah);

    // ASSERT — Verifikasi hasil
    $this->assertNotNull($kasbon);
    $this->assertEquals($jumlah, $kasbon->jumlah);
    $this->assertEquals('pending', $kasbon->status);
}
```

### 2.6 Test Data Strategy

**Gunakan Model Factory, BUKAN hardcoded array:**

```php
// ✅ BENAR — Factory
$user = User::factory()->create([
    'cabang_id' => $cabang->id,
    'role' => 'staff_produksi',
]);

$order = Order::factory()->create([
    'customer_id' => $customer->id,
    'status' => 'pending',
    'total_harga' => 5000000,
]);

// ❌ SALAH — Hardcoded (rapuh, tidak realistis)
$user = User::create([
    'nama' => 'Test User',
    'email' => 'test@test.com',
    'password' => bcrypt('password'),
    'role' => 'staff_produksi',
    'cabang_id' => 1,
    'company_id' => 1,
]);
```

### 2.7 RefreshDatabase

Setiap test class yang menyentuh database **WAJIB** menggunakan `RefreshDatabase`:

```php
use Illuminate\Foundation\Testing\RefreshDatabase;

class KasbonServiceTest extends TestCase
{
    use RefreshDatabase;

    protected KasbonService $service;

    protected function setUp(): void
    {
        parent::setUp();
        $this->service = app(KasbonService::class);
    }
}
```

### 2.8 Skenario Test (Normal, Edge, Error)

Setiap service WAJIB di-test dengan 3 jenis skenario:

```php
// 1. Skenario Normal — happy path
public function test_createOrder_normal_single_item(): void { ... }

// 2. Skenario Batas (Edge Cases)
public function test_createOrder_zero_quantity_rejected(): void { ... }
public function test_createOrder_very_large_quantity(): void { ... }
public function test_ajukanKasbon_exact_limit_amount(): void { ... }

// 3. Skenario Error
public function test_createOrder_invalid_customer_throws_404(): void { ... }
public function test_ajukanKasbon_exceeds_limit_throws_exception(): void { ... }
```

---

## 3. API Integration Test

### 3.1 Kapan Dibuat

Test API endpoint dibuat saat:
- Endpoint memiliki logic kompleks (bukan sekadar CRUD)
- Endpoint melibatkan authorization check
- Endpoint memiliki validasi kompleks

### 3.2 Contoh

```php
class OrderApiTest extends TestCase
{
    use RefreshDatabase;

    public function test_staff_cannot_create_order(): void
    {
        $staff = User::factory()->create(['role' => 'staff_produksi']);
        $token = auth()->login($staff);

        $response = $this->withHeaders([
            'Authorization' => "Bearer $token",
        ])->postJson('/api/v1/orders', [
            'customer_id' => 1,
            'items' => [],
        ]);

        $response->assertStatus(403);
    }

    public function test_admin_can_create_order(): void
    {
        $admin = User::factory()->create(['role' => 'admin_cabang']);
        $customer = Customer::factory()->create();
        $token = auth()->login($admin);

        $response = $this->withHeaders([
            'Authorization' => "Bearer $token",
        ])->postJson('/api/v1/orders', [
            'customer_id' => $customer->id,
            'items' => [['product_id' => 1, 'qty_s' => 10]],
        ]);

        $response->assertStatus(201)
            ->assertJsonStructure(['success', 'message', 'data']);
    }
}
```

---

## 4. Aturan Testing Flutter UI

### 4.1 Bebas Syntax Error
- Setiap perubahan Flutter UI **WAJIB** bebas dari syntax error sebelum dianggap selesai.
- Validasi dilakukan dengan:
  ```bash
  dart analyze
  ```
  dan pastikan tidak ada error (hanya warning/info yang diizinkan).

### 4.2 Bebas Overflow Linting Warnings
- Setiap perubahan Flutter UI **WAJIB** bebas dari overflow linting warnings (`RenderFlex overflowed`).
- Jika ada overflow terdeteksi, wajib diperbaiki sebelum melanjutkan.
- Rujuk ke `docs/standard/CODING_STANDARD.md` untuk panduan pencegahan overflow.

### 4.3 Widget Test (Recommended)
- Widget yang memiliki logic (form validation, button action) sebaiknya punya widget test
- Widget murni UI (static text, layout) tidak perlu widget test

---

## 5. Verifikasi Ketat (Zero Error Policy)

- **Jalan Langsung Jadi:** Dilarang melakukan *commit* atau menganggap tugas selesai jika masih ada *error*, sekecil apa pun.
- Agent **WAJIB** melakukan verifikasi menyeluruh (*build*, *compile*, *test*) secara internal.
- Dilarang memberikan kode yang membutuhkan proses *debugging* panjang dari pihak User.

---

## 6. Menjalankan Test

### 6.1 Perintah Test

```bash
# Jalankan semua test
php artisan test

# Jalankan 1 file test
php artisan test tests/Feature/Services/KasbonServiceTest.php

# Jalankan 1 method test
php artisan test --filter=test_ajukanKasbon_exceeds_limit

# Jalankan dengan coverage (butuh Xdebug/PCOV)
php artisan test --coverage

# Jalankan test Flutter
flutter test

# Analyze Flutter
dart analyze
```

### 6.2 CI/CD (Opsional untuk Fase 1)

Jika sudah setup CI (GitHub Actions), test **WAJIB** pass sebelum merge:

```yaml
# .github/workflows/test.yml (opsional)
name: Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: composer install
      - run: php artisan test
```

---

## 7. Checklist Sebelum Commit

- [ ] Routing endpoint baru sudah divalidasi (`php artisan route:list`)
- [ ] Migrasi sudah diverifikasi (`php artisan migrate --pretend`)
- [ ] Unit Test untuk logika finansial sudah dibuat dan berstatus *PASS*
- [ ] API endpoint test sudah dibuat (jika ada logic authorization)
- [ ] Test menggunakan AAA pattern dan naming convention yang benar
- [ ] Test data menggunakan Factory, bukan hardcoded
- [ ] `dart analyze` tidak mengembalikan error (Zero Error Policy ditaati)
- [ ] Tidak ada overflow linting warnings pada UI yang diubah
- [ ] Kode baru sudah mengikuti `CODING_STANDARD.md` (termasuk batas 300 baris)
