# 🗄️ Data Migration Standard — Konveksio

> **Versi:** 1.0
> **Dibuat:** 13 Juni 2026
> **Status:** MANDATORY — Semua perubahan database WAJIB mengikuti standar ini

---

## 1. Migration File

### 1.1 Penamaan File

Format: `YYYY_MM_DD_HHMMSS_{action}_{table}_table.php`

| Action | Prefix | Contoh |
|---|---|---|
| Buat tabel baru | `create_` | `2026_01_01_000006_create_orders_table.php` |
| Tambah kolom | `add_{column}_to_` | `2026_06_13_105831_add_fcm_token_to_users_table.php` |
| Hapus kolom | `drop_{column}_from_` | `2026_07_01_120000_drop_old_field_from_orders_table.php` |
| Rename kolom | `rename_{column}_in_` | `2026_07_01_120000_rename_status_in_orders_table.php` |
| Rename tabel | `rename_{old}_to_{new}_` | `2026_07_01_120000_rename_pelanggans_to_customers_table.php` |

### 1.2 Urutan Migration

Migration WAJIB dijalankan dalam urutan ini (berdasarkan dependency):

1. **Tabel tenant:** `companies`
2. **Tabel cabang:** `branches` (FK → companies)
3. **Tabel user:** `users` (FK → companies, branches)
4. **Tabel master:** `customers`, `products`, `vendors`, `rekenings` (FK → companies, branches)
5. **Tabel referensi:** `model_pakaians`, `master_tarifs` (FK → companies, branches)
6. **Tabel transaksional:** `orders` (FK → customers, branches)
7. **Tabel detail:** `order_items`, `tahap_produksis`, `pembayarans` (FK → orders)
8. **Tabel operasional:** `kasbons`, `pengeluarans`, `mutasi_cabangs` (FK → users, branches)
9. **Tabel log:** `log_handovers`, `porsi_pekerjaans` (FK → tahap_produksis)

### 1.3 Struktur Migration

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('orders', function (Blueprint $table) {
            // Primary key
            $table->id();

            // Tenant & branch isolation (WAJIB)
            $table->foreignId('company_id')->constrained()->cascadeOnDelete();
            $table->foreignId('cabang_id')->constrained('branches')->cascadeOnDelete();

            // Foreign keys
            $table->foreignId('customer_id')->constrained()->restrictOnDelete();

            // Data fields
            $table->string('nomor_order')->unique();
            $table->string('status')->default('pending');
            $table->decimal('total_harga', 15, 2)->default(0);
            $table->date('deadline');
            $table->text('catatan')->nullable();

            // Soft delete (untuk entitas transaksional)
            $table->softDeletes();

            // Timestamps
            $table->timestamps();

            // Indexes
            $table->index(['company_id', 'cabang_id', 'status']);
            $table->index('customer_id');
            $table->index('deadline');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('orders');
    }
};
```

---

## 2. Aturan Wajib

### 2.1 Foreign Key Constraint

- **WAJIB** ada foreign key constraint di setiap relasi
- Gunakan `constrained()` helper Laravel
- Tentukan cascade behavior:
  - `cascadeOnDelete()` — hapus child saat parent dihapus (untuk: order_items, tahap_produksis)
  - `restrictOnDelete()` — tolak hapus jika punya child (untuk: orders → customers)
  - `nullOnDelete()` — set null jika parent dihapus (jarang digunakan)

### 2.2 Index

- **WAJIB** buat index untuk kolom yang sering di-query:
  - Foreign key columns
  - Kolom yang dipakai di `WHERE` clause (status, is_active)
  - Kolom yang dipakai di `ORDER BY` (created_at, deadline)
- **JANGAN** buat index berlebihan — index memperlambat write

### 2.3 Nullable

- Kolom opsional: `->nullable()`
- Kolom wajib: tanpa nullable, beri default jika masuk akal
- **JANGAN** buat semua kolom nullable tanpa alasan

### 2.4 Default Value

```php
// ✅ BENAR — default yang masuk akal
$table->string('status')->default('pending');
$table->decimal('total_harga', 15, 2)->default(0);
$table->boolean('is_active')->default(true);

// ❌ SALAH — default untuk kolom yang harus diisi user
$table->string('nama')->default(''); // ← nama tidak boleh kosong
$table->decimal('harga_satuan', 15, 2)->default(0); // ← harga harus diisi
```

### 2.5 Tipe Data

| Data | Tipe | Keterangan |
|---|---|---|
| Nama, email, string pendek | `string('nama')` (varchar 255) | Default length |
| Nomor telepon, WA | `string('no_whatsapp', 20)` | Batasi panjang |
| Alamat, catatan | `text('alamat')` | Unlimited length |
| Harga, nominal uang | `decimal('total', 15, 2)` | 15 digit, 2 desimal |
| Qty, jumlah integer | `unsignedInteger('qty')` atau `integer('qty')` | |
| Boolean | `boolean('is_active')` | |
| Tanggal (tanpa waktu) | `date('deadline')` | |
| Tanggal + waktu | `timestamp('tanggal_approve')` atau `timestamps()` | |
| JSON (array, config) | `json('ukuran')` | PostgreSQL native JSONb |
| File path | `string('desain_file', 500)->nullable()` | |

---

## 3. Rollback

### 3.1 Kewajiban down()

- Setiap `up()` **WAJIB** punya `down()` yang valid
- `down()` harus bisa mengembalikan database ke kondisi sebelum `up()` dijalankan
- Untuk `create table` → `down()` = `dropIfExists`
- Untuk `add column` → `down()` = `dropColumn`

### 3.2 Data Migration Rollback

Untuk migration yang mengubah data (bukan schema), rollback lebih kompleks:

```php
public function up(): void
{
    // Tambah kolom baru
    Schema::table('orders', function (Blueprint $table) {
        $table->decimal('biaya_kirim', 15, 2)->default(0)->after('total_harga');
    });

    // Backfill data existing
    DB::table('orders')->whereNull('biaya_kirim')->update(['biaya_kirim' => 0]);
}

public function down(): void
{
    Schema::table('orders', function (Blueprint $table) {
        $table->dropColumn('biaya_kirim');
    });
}
```

---

## 4. Seeder

### 4.1 Kewajiban Seeder

- **WAJIB** ada seeder untuk data development
- Seeder harus idempotent (bisa dijalankan berulang tanpa error)
- Gunakan `updateOrCreate` atau `firstOrCreate`, bukan `create`

### 4.2 Seeder yang Dibutuhkan

| Seeder | Isi |
|---|---|
| `CompanySeeder` | 1 company dummy (Konveksio) |
| `BranchSeeder` | 2 cabang: Serang, Solo |
| `UserSeeder` | Super admin, admin cabang, staff produksi (min 5 user) |
| `CustomerSeeder` | 5-10 pelanggan dummy |
| `ProductSeeder` | Produk standar: Kaos, Kemeja, Seragam, dll. |
| `ModelPakaianSeeder` | Model pakaian default |
| `MasterTarifSeeder` | Tarif default per tahap × produk |
| `RekeningSeeder` | Rekening bank dummy per cabang |

### 4.3 Struktur Seeder

```php
<?php

namespace Database\Seeders;

use App\Models\Company;
use Illuminate\Database\Seeder;

class CompanySeeder extends Seeder
{
    public function run(): void
    {
        Company::updateOrCreate(
            ['nama' => 'Konveksio'],
            [
                'nama' => 'Konveksio',
                'domain_saas' => 'konveksio.com',
                'status_langganan' => 'active',
            ]
        );
    }
}
```

### 4.4 DatabaseSeeder

```php
public function run(): void
{
    $this->call([
        CompanySeeder::class,
        BranchSeeder::class,
        UserSeeder::class,
        CustomerSeeder::class,
        ProductSeeder::class,
        ModelPakaianSeeder::class,
        MasterTarifSeeder::class,
        RekeningSeeder::class,
    ]);
}
```

---

## 5. Data Migration (Backfill)

### 5.1 Aturan Backfill

Jika menambah kolom baru ke tabel yang sudah ada data:

1. **Schema change dulu** — tambah kolom dengan `nullable()` atau `default`
2. **Backfill data** — isi kolom untuk existing records
3. **Constraint (opsional)** — jika kolom tidak boleh null, ubah setelah backfill

```php
public function up(): void
{
    // Step 1: Tambah kolom nullable
    Schema::table('orders', function (Blueprint $table) {
        $table->string('nomor_invoice')->nullable()->after('nomor_order');
    });

    // Step 2: Backfill
    $orders = DB::table('orders')->whereNull('nomor_invoice')->get();
    foreach ($orders as $order) {
        DB::table('orders')
            ->where('id', $order->id)
            ->update(['nomor_invoice' => 'INV-' . date('Ym', strtotime($order->created_at)) . '-' . str_pad($order->id, 4, '0', STR_PAD_LEFT)]);
    }

    // Step 3: Buat NOT NULL (jika perlu)
    Schema::table('orders', function (Blueprint $table) {
        $table->string('nomor_invoice')->nullable(false)->change();
    });
}
```

### 5.2 Performance

- Untuk tabel besar (> 10.000 rows), gunakan chunking:
  ```php
  DB::table('orders')->whereNull('nomor_invoice')->chunkById(500, function ($orders) {
      foreach ($orders as $order) {
          // process...
      }
  });
  ```
- **JANGAN** jalankan backfill tanpa chunking di tabel besar — bisa timeout

---

## 6. Validasi Migration

### 6.1 Sebelum Commit

Setiap migration baru **WAJIB** divalidasi:

```bash
# 1. Cek SQL yang dihasilkan
php artisan migrate --pretend

# 2. Jalankan migration
php artisan migrate

# 3. Cek rollback bisa jalan
php artisan migrate:rollback --step=1

# 4. Jalankan ulang
php artisan migrate

# 5. Jalankan seeder (jika ada seeder baru)
php artisan db:seed --class=NamaSeeder
```

### 6.2 Checklist

- [ ] Nama file mengikuti konvensi
- [ ] Foreign key constraint sudah ada
- [ ] Index untuk kolom yang sering di-query
- [ ] `down()` sudah benar (bisa rollback)
- [ ] `migrate --pretend` menghasilkan SQL yang sesuai
- [ ] Seeder diupdate jika perlu
- [ ] Model `$fillable` sudah sinkron dengan kolom baru
