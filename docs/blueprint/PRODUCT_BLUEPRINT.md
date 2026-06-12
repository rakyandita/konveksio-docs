# 📋 Product Blueprint — Konveksio

> **Versi:** 2.0 (Modular Refactor)
> **Sumber:** `docs/requirement_spec.md` (Bagian 1, 2, 5, 7, 8)
> **Tanggal:** 5 Juni 2026

---

## 1. Ringkasan Proyek

**Konveksio** adalah aplikasi manajemen bisnis konveksi yang dirancang untuk mengelola seluruh alur operasional — mulai dari penerimaan order, tracking produksi dengan sistem estafet handover antar karyawan, pengelolaan keuangan (invoice, pembayaran, kasbon, gaji borongan), hingga komunikasi status pesanan ke pelanggan melalui public link dan notifikasi WhatsApp.

### 1.1 Latar Belakang

Bisnis konveksi milik user saat ini menghadapi kendala operasional:

- Pelacakan order & status produksi masih via WhatsApp/catatan manual
- Kalkulasi HPP & harga jual tidak akurat
- Stok bahan baku tidak terkontrol
- Progress produksi per tahap sulit dipantau
- Laporan keuangan tidak real-time
- Komunikasi antar divisi tidak efisien
- Penjadwalan & kapasitas produksi sulit diatur
- Manajemen pelanggan & riwayat order tidak rapi

### 1.2 Tujuan

1. Digitalisasi seluruh alur order — dari input pesanan hingga pengiriman
2. Menyediakan sistem estafet produksi agar karyawan bisa langsung handover tanpa birokrasi admin
3. Otomatisasi kalkulasi HPP dan penggajian borongan
4. Menyediakan public link invoice agar pelanggan bisa cek status pesanan & pembayaran secara mandiri
5. Menyediakan dashboard real-time untuk owner & admin

---

## 2. Profil Bisnis

| Aspek | Detail |
|---|---|
| Nama Bisnis | Konveksio |
| Jenis Layanan | Produksi massal, sablon (DTF/DTG/screen print), bordir, full package (desain → produksi → packing) |
| Skala | ~20 karyawan, <50 order/bulan |
| Cabang | 2 cabang: **Serang** (produksi utama) & **Solo** (vendor/CMT partner) |
| Mata Uang | Rupiah (Rp) |
| Bahasa Antarmuka | Bahasa Indonesia |

### 2.1 Perbedaan Operasional Antar Cabang

#### Cabang Serang (Produksi Utama)
- Karyawan digaji **borongan per pcs** yang dikerjakan
- Gajian fleksibel: per minggu atau per 2 minggu (tiap Sabtu sore)
- Setiap tahap produksi punya tarif berbeda per jenis produk
- Contoh tarif: potong kaos Rp1.000/pcs, potong kemeja Rp2.000/pcs, potong jas Rp3.000/pcs, jahit kaos Rp2.500/pcs, lubang kancing Rp500/pcs, pasang kancing Rp300/pcs

#### Cabang Solo
- Beroperasi sebagai cabang utuh (memiliki pelanggan sendiri, transaksi mandiri, dsb.)
- Juga bertindak sebagai **suplier/vendor** untuk Serang (seragam batik jadi, bahan batik, topi, sabuk)
- Sistem bayar produksi tetap borongan / CMT (Cutting Modelling Trimming) per pcs × qty
- Terdapat SPV dan ~5 karyawan yang mengurus produksi.
- Karyawan Solo dikelola *di luar sistem Konveksio* (gaji dibagi manual oleh SPV). Di dalam sistem, SPV Solo diberikan akses setara "Staff Produksi".

### 2.2 Transaksi Antar Cabang
- Cabang Serang sering beli dari Solo: seragam batik jadi, bahan batik, topi, sabuk
- Perlu dicatat sebagai transaksi antar cabang di sistem

---

## 5. Modul & Fitur

### 5.1 Fase 1 — MVP (Prioritas Utama)

#### M-01: Dashboard
- Ringkasan total order bulan ini vs bulan lalu
- Grafik pendapatan & pengeluaran
- Order per status (berapa pending, produksi, selesai)
- Top 5 pelanggan (order terbanyak/terbesar)
- Performa produksi (rata-rata waktu penyelesaian)
- Rekap gaji karyawan periode berjalan
- Filter per cabang (untuk Super Admin)

#### M-02: Manajemen Order
- **CRUD Order** dengan data:
  - Tanggal pembuatan nota (auto-generate + editable)
  - Nama pelanggan, nama instansi/organisasi (opsional) & kontak (WA/telepon) — pilih dari database atau input baru
  - Jenis produk (pilih dari katalog, input custom, atau **Produk Bundle**)
  - Qty per size (S, M, L, XL, XXL, 3XL, 4XL, Custom). *Catatan: Qty pesanan bisa di-edit bertambah di tengah proses produksi.*
  - Bahan dan warna yang digunakan
  - Deadline / tanggal pengiriman
  - Harga satuan & total
  - Catatan khusus / instruksi tambahan
  - Upload file desain (gambar/PDF)
- **Produk Bundle (Setelan):** Jika memilih produk bertipe bundle (misal: "Seragam Pramuka"), sistem otomatis men-*expand* menjadi beberapa sub-item pesanan (misal: "Kemeja" dan "Celana"), masing-masing memiliki SPK dan jalur produksi terpisah, namun di invoice pelanggan tetap tampil sebagai 1 item "Seragam Pramuka".
- **Status Order:** Pending → DP Diterima → Diproduksi → Siap Kirim → Pelunasan → Dikirim → Selesai → Dibatalkan
  - Catatan: Pembayaran sangat fleksibel (bisa mulai produksi tanpa DP, bisa lunas di awal, atau kirim dulu baru pelunasan).
  - Jika Dibatalkan: Uang DP hangus, karyawan yang telanjur bekerja tetap dibayarkan gajinya.
- **Pencarian & Filter:** berdasarkan status, pelanggan, tanggal, cabang
- **Persiapan Produksi:** Status pesanan "Pending" tidak otomatis masuk produksi. Admin harus menekan tombol "Persiapan Produksi" yang terdapat pada **setiap kartu item pesanan** (di halaman Detail Order) untuk mengatur tahapan produksi, tarif jasa, dan men-*generate* SPK per item pesanan. Jika pesanan bertipe Bundle/Setelan, maka setiap komponen anak harus dilakukan Persiapan Produksi secara independen.

#### M-03: Tracking Produksi (Kanban) & Estafet Handover
- **Daftar Tahap Produksi & Tipe Tarif:**
  1. Potong (Tarif: Per Jenis Produk)
  2. Sablon (Tarif: Per Desain — Internal/Vendor)
  3. Bordir (Tarif: Per Desain — Internal/Vendor)
  4. Jahit (Tarif: Per Jenis Produk)
  5. Lubang Kancing (Tarif: Flat)
  6. Pasang Kancing (Tarif: Flat)
  7. Setrika (Tarif: Flat)
  8. QC & Packing (Tarif: Flat)
- **Dynamic Routing:** Saat "Persiapan Produksi", admin dapat mencentang tahap mana saja yang berlaku untuk item pesanan tersebut (dinamis per order item).
- **SPK (Surat Perintah Kerja):** Dibuat per *order item*. Bisa dilihat di aplikasi atau dicetak PDF. Mencakup bahan, warna, breakdown size, alur produksi, tarif, dan catatan tambahan.
- **Papan Kanban (Visual Progress):** Admin dapat melihat pergerakan produksi via Kanban Board. Kolom Kanban dinamis mengikuti tahapan yang aktif.
- **Role-Based Task View:** Karyawan hanya melihat tugas (antrean produksi) yang sesuai dengan *divisi* mereka (misal: Tukang Potong hanya melihat antrean Potong).
- **Sistem Keroyokan (Multi-worker):** Tugas dalam satu tahap (misal jahit 100 pcs) bisa dibagi porsinya ke beberapa karyawan (misal 30 pcs untuk A, 70 pcs untuk B) oleh Boss Cabang / Tukang Potong.
- **Handover Parsial per Size:** Saat menyerahkan hasil kerja ke tahap berikutnya, form akan memecah data berdasarkan jumlah *size* (S, M, L, XL) yang sudah selesai, beserta siapa yang mengerjakannya.
- **Log Rework/Reject:** Saat ada barang cacat (QC), dikembalikan untuk diperbaiki (Rework). Ada dua tipe Rework: (1) Paid Rework (reject karena kain cacat, gaji ditambah), (2) Unpaid Rework (karena human error, wajib perbaiki tanpa tambahan gaji).
- **Kirim/Terima Vendor:** Untuk tahap Sablon/Bordir, atau jika CMT sedang *overload*, order bisa dilempar ke vendor luar. Admin mencatat "Kirim ke Vendor" dan "Terima dari Vendor" di aplikasi.
- **Pause/Switch Order:** Karyawan dapat menunda (pause) pekerjaannya untuk mengerjakan order prioritas/VIP lain.
- **Edit Tahap Berjalan:** Admin bisa menambah atau menghapus tahapan (misal: kelupaan tahap Setrika) meskipun proses produksi sudah berjalan.

#### M-04: Master Tarif Jasa & Kalkulasi HPP
- **Master Tarif Jasa Produksi:** Berbentuk matriks antara [Tahap] x [Jenis Produk].
  - Tarif Jahit/Potong berbeda antara Kaos vs Kemeja.
  - Tarif Sablon/Bordir bersifat input manual per desain (karena tergantung kerumitan/warna).
  - Tarif Finishing (setrika, kancing, packing) bersifat harga *flat*.
- **Override Harga Aktual:** Saat rilis produksi, admin bisa meng-edit tarif dari master data khusus untuk order tersebut.
- **Kalkulator Biaya Bahan (MVP):** Di form Persiapan Produksi, admin meng-input Biaya Bahan Baku dibantu dengan kalkulator sederhana (Kebutuhan bahan per pcs × harga satuan × qty order) karena modul stok belum tersedia di Fase 1.
- **Kalkulasi HPP otomatis:** Total HPP = Σ (tarif per tahap × qty aktual yang dikerjakan) + biaya bahan + biaya vendor.
- **Margin & harga jual:** Tampilkan selisih antara HPP vs harga jual.

#### M-05: Invoice & Pembayaran
- **Nomor invoice auto-generate:** Format `INV-[TAHUN][BULAN]-[NOMOR_URUT]`
  - Contoh: `INV-202606-0001`
- **Konten invoice:**
  - Detail pelanggan: nama pelanggan, nama instansi/organisasi (jika ada), kontak
  - Detail item pesanan (produk, qty per size, harga satuan, total)
  - Riwayat pembayaran (DP → Cicilan → Pelunasan) dengan tanggal
  - Status: Lunas / Belum Lunas / Sebagian
  - Info rekening pembayaran (BCA, Mandiri, BRI, dll.)
  - Tanda tangan digital / cap perusahaan
  - QR code → link ke public link invoice
- **Download PDF**
- **Sistem pembayaran bertahap:** DP → Cicilan (opsional, bisa lebih dari 1) → Pelunasan
  - Wajib melampirkan kuitansi / bukti terima uang dari admin di setiap histori pembayaran.
- **Pengaturan Rekening:** Admin dapat mengatur (tambah/hapus) rekening tujuan transfer (BCA, Mandiri, Cash, dll.) yang bersifat spesifik **per cabang**.
- **Public Link Invoice:**
  - Bisa diakses pelanggan tanpa login
  - Menampilkan: detail pesanan, status pembayaran, status produksi (Pending → Diproduksi → Siap Kirim → Selesai), riwayat pembayaran
  - URL: `konveksio.com/invoice/[KODE_UNIK]`

#### M-06: Manajemen Pelanggan
- **Data pelanggan:** Nama, instansi/organisasi (opsional), kontak (WA/telepon), alamat, email (opsional)
- **Riwayat order** per pelanggan
- **Riwayat pembayaran** per pelanggan
- **Total transaksi** kumulatif
- **Pencarian & filter**

#### M-07: Manajemen Vendor
- **Data vendor:** Nama, kontak, jenis layanan (CMT, sablon, bordir, aksesoris, bahan kain, dll.), alamat
- **Tarif vendor** per layanan
- **Riwayat transaksi** ke vendor
- **Kategori vendor:** Batik, Bordir, CMT, Aksesoris (topi, dasi, sabuk), Baju Jadi, Bahan, dll.

#### M-08: Laporan Keuangan
- **Laporan pendapatan:** Per periode (harian, mingguan, bulanan)
- **Laporan pengeluaran:** Gaji karyawan, operasional, dan **pembayaran vendor** (termasuk tagihan per vendor)
- **Laporan profit:** Pendapatan - Pengeluaran
- **Filter per cabang** (untuk Super Admin)
- **Export laporan** (PDF/Excel)

#### M-09: Manajemen Karyawan & Penggajian
- **Data karyawan:** Nama, kontak, posisi/keahlian, cabang, role di app
- **Rekap gaji borongan:**
  - Otomatis dihitung dari log handover (tahap × qty × tarif per jenis produk)
  - Periode gajian fleksibel per cabang (mingguan / 2 mingguan)
  - Hari gajian: Sabtu sore (default, bisa diubah)
  - **Rekap gaji mingguan:** Ringkasan gaji per karyawan per minggu
  - **Rekap gaji bulanan:** Akumulasi gaji per karyawan per bulan
- **Slip gaji digital:**
  - Setiap karyawan bisa melihat slip gaji sendiri di aplikasi
  - Detail: breakdown per order per tahap, total pcs, gaji kotor, potongan kasbon, gaji bersih
  - Tersedia per periode (mingguan & bulanan)
- **Kasbon karyawan:**
  - Input harian oleh karyawan
  - Limit kasbon mingguan: Rp500.000 (configurable)
  - Push notification ke boss cabang / super admin untuk approval
  - Notifikasi ke karyawan jika disetujui/ditolak
  - Riwayat kasbon & saldo kasbon
  - Potong dari gaji otomatis. **Catatan:** Jika kasbon melebihi gaji mingguan, sisa utang otomatis di-*carry over* ke minggu berikutnya.

#### M-10: Notifikasi
- **Push Notification (in-app via FCM):**
  - Handover estafet: "Ada [X]pcs [produk] masuk ke proses [tahap] Anda"
  - Deadline mendekat: H-3 dan H-1
  - Kasbon diajukan (ke approver)
  - Kasbon di-approve/ditolak (ke karyawan)

#### M-11: Katalog Produk & Bundle
- **Produk standar yang tersedia:**
  - Kaos, Kemeja, Seragam, Celana, Batik, Topi, Jas, Rok, Kerudung, dll.
- **Produk Bundle (Setelan/Paket):**
  - Pembuatan produk paket (misal: "Seragam Pramuka") yang terdiri dari beberapa sub-produk ("Kemeja" dan "Rok").
- **Custom product:** Bisa input jenis produk baru per order.

#### M-12: Transaksi Antar Cabang
- Pencatatan pembelian/penjualan antar cabang
- Contoh: Cabang Serang beli seragam batik jadi dari Cabang Solo
- Tercatat di laporan keuangan masing-masing cabang

#### M-13: Pengiriman
- **Metode pengiriman fleksibel:**
  - Kirim sendiri
  - Ekspedisi (JNE, J&T, SiCepat, dll.)
  - Pickup oleh pelanggan
- **Data pengiriman:** Nomor resi (jika ekspedisi), tanggal kirim, penerima
- **Update status** ke pelanggan via public link & WA

### 5.2 Fase 2 — Pengembangan Lanjutan (Post-MVP)

| ID | Fitur | Keterangan |
|---|---|---|
| F2-01 | Manajemen Bahan Baku & Inventory | Stok masuk/keluar, alert stok minimum, jenis: kain, benang, kancing/resleting/aksesori, label/tag/packaging |
| F2-02 | Gaji Campuran | Gaji tetap + bonus produksi untuk karyawan tertentu |
| F2-03 | Absensi Karyawan | Absensi harian dengan lokasi (GPS) |
| F2-04 | Customer Portal | Pelanggan bisa login untuk lihat semua ordernya |
| F2-05 | Integrasi Ekspedisi | Auto cek ongkir & tracking via API (RajaOngkir) |
| F2-06 | Auto-Update Aplikasi | Update OTA menggunakan Shorebird agar app update otomatis tanpa reinstall APK |
| F2-07 | Notifikasi WhatsApp | Integrasi otomatisasi WA API ke pelanggan (ditunda dari MVP untuk memitigasi blokir) |

---

## 8. Glossary

| Istilah | Definisi |
|---|---|
| **HPP** | Harga Pokok Produksi — total biaya untuk memproduksi satu item |
| **CMT** | Cutting, Modelling, Trimming — jasa konveksi dari potong sampai finishing |
| **Borongan** | Sistem upah berdasarkan jumlah item yang dikerjakan |
| **Estafet Handover** | Proses serah terima pekerjaan dari satu tahap produksi ke tahap berikutnya |
| **Public Link** | URL yang bisa diakses pelanggan tanpa login untuk cek status pesanan & invoice |
| **Kasbon** | Pinjaman/uang muka karyawan yang dipotong dari gaji |
| **Finishing** | Tahap akhir produksi: lubang kancing, pasang kancing, setrika, buang benang sisa |
| **QC** | Quality Control — pemeriksaan kualitas produk sebelum packing |

---
