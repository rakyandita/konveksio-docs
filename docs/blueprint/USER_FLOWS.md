# 🔀 User Flows — Konveksio

> **Versi:** 1.0
> **Tanggal:** 5 Juni 2026
> **Reference:** requirement_spec.md

---

## UF-01: Autentikasi (Login)

```
[Buka App]
    │
    ├── Sudah pernah login? ──(Ya)──→ [Cek Biometrik (Fingerprint/FaceID)]
    │                                      │
    │                                      ├── Berhasil → [Masuk ke Dashboard]
    │                                      └── Gagal → [Tampilkan Form Login]
    │
    └── Belum pernah login ──→ [Tampilkan Form Login]
                                    │
                                    ├── Input Email + Password/PIN
                                    │
                                    ├── [Validasi Server]
                                    │      │
                                    │      ├── Valid → [Tawarkan Setup Biometrik]
                                    │      │              │
                                    │      │              ├── Ya → [Simpan, Masuk ke Dashboard]
                                    │      │              └── Nanti → [Masuk ke Dashboard]
                                    │      │
                                    │      └── Invalid → [Tampilkan Error, Ulangi]
                                    │
                                    └── Lupa Password → [Reset via Email]
```

### Catatan:
- Session persistent di mobile (tidak perlu login ulang kecuali logout manual)
- Biometrik opsional, bisa diaktifkan/nonaktifkan di Settings
- PIN karyawan: 6 digit numerik (untuk karyawan yang tidak punya email)

---

## UF-02: Membuat Order Baru (Admin/Admin Cabang)

```
[Dashboard] → [Tap FAB "+"] → [Pilih "Order Baru"]
    │
    ├── Step 1: Data Pelanggan
    │   ├── Cari pelanggan existing (autocomplete)
    │   │   └── Ditemukan → Auto-fill data pelanggan
    │   └── Pelanggan baru → Input: Nama, Instansi/Organisasi (opsional), No. WA, Alamat
    │
    ├── Step 2: Detail Pesanan (bisa multi-item / bundle)
    │   ├── Item 1:
    │   │   ├── Pilih jenis produk (katalog / custom / Bundle)
    │   │   │   └── Jika Bundle (misal: Seragam Pramuka), sistem otomatis memecah jadi Sub-Item (Kemeja & Celana)
    │   │   ├── Input qty per size: S[ ] M[ ] L[ ] XL[ ] Custom[ ]
    │   │   ├── Pilih bahan dan warna
    │   │   ├── Harga satuan (auto dari template / override)
    │   │   └── Upload desain (gambar/PDF) [opsional]
    │   ├── [+ Tambah Item] → Item 2, 3, ...
    │   └── Catatan khusus / instruksi tambahan
    │
    ├── Step 3: Jadwal & Pengiriman
    │   ├── Tanggal deadline
    │   ├── Metode pengiriman: [Kirim Sendiri] [Ekspedisi] [Pickup]
    │   └── Alamat pengiriman (auto dari data pelanggan / input baru)
    │
    ├── Step 5: Ringkasan & HPP
    │   ├── Tampilkan kalkulasi HPP otomatis
    │   ├── Total harga jual
    │   ├── Margin profit
    │   └── [Edit jika perlu]
    │
    └── [Simpan Order]
        │
        ├── Order tersimpan dengan status "Pending"
        ├── Invoice otomatis ter-generate (INV-YYYYMM-XXXX)
        ├── Public link ter-generate
        └── [Kirim invoice ke pelanggan via WA?]
            ├── Ya → Kirim WA dengan link invoice
            └── Nanti → Kembali ke daftar order
```

---

## UF-03: Alur Status Order

```
[PENDING]
    │
    │ ← Admin input pembayaran DP
    ▼
[DP DITERIMA]
    │
    │ ← Admin melakukan "Persiapan Produksi"
    │   - Set tahapan produksi aktif per Order Item
    │   - Set tarif jasa per tahap
    │   - Generate SPK
    ▼
[DIPRODUKSI] ←── Kanban Board & Estafet (UF-04)
    │
    │ ← Semua tahap produksi (termasuk QC) selesai
    ▼
[QC & PACKING]
    │
    │ ← QC passed, barang sudah dipacking
    ▼
[SIAP KIRIM]
    │
    │ ← Ada 2 kemungkinan:
    │
    ├── Case A (Normal): Pelunasan dulu → Kirim
    │   │
    │   ▼
    │   [PELUNASAN]
    │   │
    │   │ ← Admin input pembayaran pelunasan
    │   ▼
    │   [DIKIRIM]
    │   │
    │   ▼
    │   [SELESAI]
    │
    └── Case B (Pelanggan Tertentu): Kirim dulu → Pelunasan
        │
        ▼
        [DIKIRIM]
        │
        │ ← Admin input pembayaran pelunasan
        ▼
        [PELUNASAN]
        │
        ▼
        [SELESAI]
```

### Status di Public Link (Simplified untuk Pelanggan):
```
Pending → Diproduksi → Finishing / Siap Kirim → Selesai
```

---

## UF-04: Kanban Produksi & Estafet (Admin & Staff)

### A. Persiapan Produksi (Admin)
```
[Detail Order - PENDING] → [Persiapan Produksi]
    │
    ├── Untuk setiap Order Item (misal: Kemeja PDL):
    │   ├── Checklist tahapan aktif: ☑ Potong  ☑ Jahit  ☐ Sablon  ☑ Packing
    │   └── Konfirmasi tarif per tahap (ambil dari master data matriks)
    │
    └── [Generate SPK & Mulai Produksi]
        ├── SPK-001-A terbentuk
        └── SPK masuk ke Papan Kanban (Kolom "Potong")
```

### B. Mengerjakan & Handover Estafet (Staff)
```
[Staff Produksi] → [Tab "Tugas Saya"]
    │
    ├── Lihat SPK di kolom antrean divisinya (misal: Tukang Potong lihat SPK masuk)
    │
    ├── [Opsi Keroyokan / Bagi Porsi] — Khusus Tukang Potong / Boss
    │   └── Bagikan 100 pcs tugas Potong ke 3 karyawan: A (30), B (40), C (30)
    │
    ├── [Kerjakan] → Update progress secara parsial:
    │   ├── "Saya sudah potong size S: 20pcs, size M: 10pcs"
    │   └── [Simpan Progress]
    │
    └── [Handover ke Proses Berikutnya]
        │
        ├── Input qty size yang di-handover (S:20, M:10)
        ├── Penerima (otomatis masuk antrean divisi selanjutnya di Kanban)
        └── [Konfirmasi Handover]
            │
            ├── ✅ Log tercatat: [Budi] handover [30pcs] ke [Jahit]
            ├── 🔔 Push notification ke Staff Jahit
            └── Qty SPK bergeser parsial di Papan Kanban
```

### C. Alur Vendor (Sablon/Bordir Eksternal)
```
[Tukang Potong / Admin] → [Handover SPK]
    │
    ├── Pilih proses tujuan: SABLON (Vendor)
    │   ├── Pilih nama vendor: "Vendor Sablon AA"
    │   └── Catat tanggal kirim
    │
    └── Beberapa hari kemudian...
        └── [Terima dari Vendor]
            ├── Update jumlah yang selesai
            └── SPK lanjut ke tahap Jahit di antrean internal
```

---

## UF-05: Input Pembayaran (Admin)

```
[Detail Order] → [Tab Pembayaran] → [+ Input Pembayaran]
    │
    ├── Jenis Pembayaran:
    │   ○ DP (Down Payment)
    │   ○ Cicilan
    │   ○ Pelunasan
    │
    ├── Jumlah: Rp [________]
    ├── Tanggal bayar: [__/__/____]
    ├── Metode: [Transfer BCA] [Transfer Mandiri] [Transfer BRI] [Cash]
    ├── Bukti transfer (upload foto) [opsional]
    ├── Catatan [opsional]
    │
    └── [Simpan]
        │
        ├── Riwayat pembayaran terupdate
        ├── Sisa tagihan otomatis terhitung
        ├── Status invoice terupdate (Belum Lunas / Sebagian / Lunas)
        ├── Public link terupdate
        └── [Kirim notifikasi WA ke pelanggan?]
            ├── Ya → "Pembayaran [jenis] sebesar Rp[jumlah] telah kami terima. Sisa: Rp[sisa]. Cek detail: [public_link]"
            └── Tidak → Selesai
```

---

## UF-06: Kasbon Karyawan

### A. Pengajuan (oleh Staff Produksi)

```
[Profil Saya] → [Kasbon] → [Ajukan Kasbon]
    │
    ├── Cek limit mingguan: Rp500.000
    │   ├── Sisa limit: Rp[sisa]
    │   └── Jika sudah melebihi limit → [Tampilkan pesan "Limit kasbon minggu ini sudah tercapai"]
    │
    ├── Input jumlah kasbon: Rp [________]
    ├── Keperluan [opsional]
    │
    └── [Ajukan]
        │
        ├── Status: MENUNGGU APPROVAL
        └── 🔔 Push notification ke Boss Cabang / Super Admin:
            "Kasbon Rp[jumlah] diajukan oleh [Nama Karyawan]"
```

### B. Approval (oleh Boss Cabang / Super Admin)

```
[Notifikasi masuk] → [Lihat Detail Kasbon]
    │
    ├── Info: Nama karyawan, jumlah, sisa limit minggu ini, riwayat kasbon
    │
    ├── [Setujui]
    │   ├── Status → DISETUJUI
    │   ├── Saldo kasbon karyawan bertambah
    │   └── 🔔 Notifikasi ke karyawan: "Kasbon Rp[jumlah] Anda telah disetujui"
    │
    └── [Tolak]
        ├── Alasan penolakan [opsional]
        ├── Status → DITOLAK
        └── 🔔 Notifikasi ke karyawan: "Kasbon Rp[jumlah] Anda ditolak. Alasan: [alasan]"
```

### C. Pemotongan Otomatis di Gajian

```
[Hari Gajian (Sabtu Sore)]
    │
    ├── Sistem hitung gaji borongan: Σ (qty handover × tarif per tahap per produk)
    ├── Kurangi total kasbon periode ini
    ├── Gaji bersih = Gaji Borongan - Total Kasbon
    │
    └── Tampilkan slip gaji digital ke karyawan
```

---

## UF-07: Rekap Gaji Borongan (Admin)

```
[Menu Karyawan] → [Rekap Gaji]
    │
    ├── Pilih periode: [Minggu ini] [2 minggu terakhir] [Custom range]
    ├── Pilih cabang: [Serang] [Solo] [Semua]
    │
    ├── Tampilkan tabel:
    │   ┌──────────┬────────────┬────────────┬──────────┬───────────┬──────────┐
    │   │ Karyawan │ Total Pcs  │ Gaji Kotor │ Kasbon   │ Potongan  │ Gaji Net │
    │   ├──────────┼────────────┼────────────┼──────────┼───────────┼──────────┤
    │   │ Budi     │ 450 pcs    │ Rp 875.000 │ Rp50.000 │ Rp 50.000 │ Rp825.000│
    │   │ Ani      │ 380 pcs    │ Rp 720.000 │ Rp 0     │ Rp 0      │ Rp720.000│
    │   └──────────┴────────────┴────────────┴──────────┴───────────┴──────────┘
    │
    ├── [Lihat Detail] → Breakdown per order per tahap:
    │   Budi - Detail Gaji:
    │   Order #123: Kaos Polo  × 200pcs × Rp1.000 (potong) = Rp200.000
    │   Order #124: Kemeja     × 100pcs × Rp2.000 (potong) = Rp200.000
    │   Order #123: Kaos Polo  × 150pcs × Rp500  (lubang kancing) = Rp75.000
    │   ...
    │   Total: Rp 875.000
    │
    └── [Approve & Proses Gaji] → Tandai gaji sudah dibayar
```

---

## UF-08: Public Link Invoice (Pelanggan / Tanpa Login)

```
[Pelanggan klik link / scan QR dari WA]
    │
    ▼
[Halaman Public Invoice]
    │
    ├── Header: Logo Konveksio + Nomor Invoice
    │   └── Detail Pelanggan: Nama Pelanggan, Instansi (jika ada), No. WA
    │
    ├── Detail Pesanan:
    │   ├── Jenis produk, qty, harga
    │   ├── Total tagihan
    │   └── Tanggal deadline
    │
    ├── Status Produksi (simplified):
    │   ○ Pending → ● Diproduksi → ○ Siap Kirim → ○ Selesai
    │
    ├── Riwayat Pembayaran:
    │   ├── DP: Rp2.000.000 — 01/06/2026 ✅
    │   ├── Cicilan 1: Rp1.000.000 — 05/06/2026 ✅
    │   └── Sisa: Rp1.500.000 — Belum Lunas
    │
    ├── Info Rekening Pembayaran:
    │   ├── BCA: 123-456-789 a.n. Konveksio
    │   ├── Mandiri: 987-654-321 a.n. Konveksio
    │   └── BRI: 111-222-333 a.n. Konveksio
    │
    └── Download Invoice PDF
```

---

## UF-09: Transaksi Antar Cabang

```
[Menu Transaksi] → [Transaksi Antar Cabang] → [+ Buat Transaksi]
    │
    ├── Cabang Asal: [Serang ▼]
    ├── Cabang Tujuan: [Solo ▼]
    ├── Jenis: [Pembelian ▼]
    │
    ├── Item:
    │   ├── Deskripsi: "Seragam batik jadi" 
    │   ├── Qty: 50
    │   ├── Harga satuan: Rp 85.000
    │   └── Total: Rp 4.250.000
    │
    ├── [+ Tambah Item]
    │
    └── [Simpan]
        │
        ├── Tercatat sebagai pengeluaran di cabang Serang
        ├── Tercatat sebagai pemasukan di cabang Solo
        └── Muncul di laporan keuangan masing-masing cabang
```

---

## UF-10: Manajemen Vendor

```
[Menu Vendor] → [Daftar Vendor]
    │
    ├── [+ Tambah Vendor]
    │   ├── Nama vendor
    │   ├── Kontak (WA/telepon)
    │   ├── Alamat
    │   ├── Kategori: ☑Batik ☐Bordir ☑CMT ☐Aksesoris ☐Bahan ☐Lainnya
    │   ├── Tarif / harga per layanan
    │   └── [Simpan]
    │
    ├── [Detail Vendor]
    │   ├── Info vendor
    │   ├── Riwayat transaksi
    │   ├── Total transaksi kumulatif
    │   └── [Edit] [Nonaktifkan]
    │
    └── [Cari & Filter] by kategori, nama
```

---

## UF-11: Pengiriman

```
[Detail Order] → Status: [SIAP KIRIM]
    │
    ├── [Proses Pengiriman]
    │   ├── Metode: ○ Kirim Sendiri  ○ Ekspedisi  ○ Pickup
    │   │
    │   ├── Jika Ekspedisi:
    │   │   ├── Nama ekspedisi: [JNE ▼] / input manual
    │   │   ├── Nomor resi: [________________]
    │   │   └── Ongkir: Rp [________]
    │   │
    │   ├── Jika Kirim Sendiri:
    │   │   ├── Pengirim: [Nama driver]
    │   │   └── Estimasi sampai: [tanggal]
    │   │
    │   ├── Jika Pickup:
    │   │   └── Tanggal pickup: [__/__/____]
    │   │
    │   └── [Konfirmasi Pengiriman]
    │       │
    │       ├── Status order → DIKIRIM
    │       ├── Public link terupdate
    │       └── 🔔 WA ke pelanggan: "Pesanan Anda telah dikirim! [detail tracking]. Cek: [public_link]"
    │
    └── [Konfirmasi Selesai] (setelah pelanggan terima)
        │
        ├── Status order → SELESAI
        └── Public link terupdate
```

---

## UF-12: Notifikasi Deadline (Otomatis oleh Sistem)

```
[Cron Job harian - pagi hari]
    │
    ├── Scan semua order aktif (belum SELESAI)
    │
    ├── Deadline H-3:
    │   └── 🔔 Push ke Admin Cabang:
    │       "⚠️ Order #[id] - [produk] deadline 3 hari lagi ([tanggal]). Progress: [X]%"
    │
    ├── Deadline H-1:
    │   └── 🔔 Push ke Admin Cabang + Super Admin:
    │       "🚨 Order #[id] - [produk] deadline BESOK ([tanggal])! Progress: [X]%"
    │
    └── Deadline Lewat:
        └── 🔔 Push ke Super Admin:
            "❌ Order #[id] - [produk] sudah MELEWATI deadline sejak [tanggal]!"
```

---
