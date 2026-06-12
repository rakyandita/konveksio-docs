# 🎨 Design System — Konveksio

> **Versi:** 2.0 (Modular Refactor)
> **Tanggal:** 5 Juni 2026
> **Reference:** `docs/blueprint/PRODUCT_BLUEPRINT.md`
> **Inspirasi:** Gojek Superapp — clean, minimalis, humanise, informatif

---

## 1. Brand Identity

### 1.1 Nama Brand
**Konveksio** — Aplikasi manajemen bisnis konveksi

### 1.2 Tagline
_"Kelola Konveksi Tanpa Ribet"_

### 1.3 Brand Personality
- **Profesional** — terpercaya untuk bisnis
- **Humanis** — ramah dan mudah digunakan oleh karyawan lapangan
- **Efisien** — mempercepat alur kerja
- **Modern** — tampilan bersih dan up-to-date

### 1.4 Logo
> ⚠️ Logo akan di-generate setelah dokumen SOT disetujui user.

**Konsep Logo:**
- Icon: Simbol jarum jahit / benang membentuk huruf "K" atau "V"
- Style: Geometric, clean, modern
- Warna: Emerald green gradient
- Tipe: Logomark + Logotype (bisa digunakan terpisah)

---

## 2. Color Palette

### 2.1 Warna Utama (Primary — Emerald Green)

| Token | Hex | Penggunaan |
|---|---|---|
| `--color-primary-50` | `#ECFDF5` | Background highlight, badge light |
| `--color-primary-100` | `#D1FAE5` | Background card aktif, chip |
| `--color-primary-200` | `#A7F3D0` | Border hover, progress bar background |
| `--color-primary-300` | `#6EE7B7` | Icon secondary, divider aktif |
| `--color-primary-400` | `#34D399` | Button hover, link hover |
| `--color-primary-500` | `#00AA13` | **Primary button, FAB, active nav (Gojek Emerald)** |
| `--color-primary-600` | `#00875A` | **Primary dark, pressed state (Emerald Gelap)** |
| `--color-primary-700` | `#006644` | **Header gradient end, deep state** |
| `--color-primary-800` | `#065F46` | Text on light, badge dark |
| `--color-primary-900` | `#064E3B` | Heading emphasis |

### 2.2 Gradien Utama

```css
/* Primary Gradient — untuk header, FAB, CTA */
background: linear-gradient(135deg, #059669 0%, #10B981 50%, #34D399 100%);

/* Shadow Gradient — untuk card elevation */
box-shadow: 0 4px 14px rgba(5, 150, 105, 0.25);

/* Subtle Gradient — untuk background section */
background: linear-gradient(180deg, #ECFDF5 0%, #FFFFFF 100%);
```

### 2.3 Warna Netral

| Token | Hex | Penggunaan |
|---|---|---|
| `--color-neutral-0` | `#FFFFFF` | Background utama |
| `--color-neutral-50` | `#F9FAFB` | Background sekunder, card |
| `--color-neutral-100` | `#F3F4F6` | Background input field, divider |
| `--color-neutral-200` | `#E5E7EB` | Border, separator |
| `--color-neutral-300` | `#D1D5DB` | Placeholder, disabled |
| `--color-neutral-400` | `#9CA3AF` | Caption text, icon inactive |
| `--color-neutral-500` | `#6B7280` | Body text secondary |
| `--color-neutral-600` | `#4B5563` | Body text primary |
| `--color-neutral-700` | `#374151` | Heading secondary |
| `--color-neutral-800` | `#1F2937` | **Heading primary, text utama** |
| `--color-neutral-900` | `#111827` | Title, emphasis |

### 2.4 Warna Semantik (Status & Feedback)

| Token | Hex | Penggunaan |
|---|---|---|
| `--color-success-light` | `#D1FAE5` | Background status sukses |
| `--color-success` | `#10B981` | Icon & text sukses, status "Selesai" |
| `--color-success-dark` | `#065F46` | Text on success background |
| `--color-warning-light` | `#FEF3C7` | Background warning |
| `--color-warning` | `#F59E0B` | Icon & text warning, status "Pending" |
| `--color-warning-dark` | `#92400E` | Text on warning background |
| `--color-error-light` | `#FEE2E2` | Background error |
| `--color-error` | `#EF4444` | Icon & text error, overdue |
| `--color-error-dark` | `#991B1B` | Text on error background |
| `--color-info-light` | `#DBEAFE` | Background info |
| `--color-info` | `#3B82F6` | Icon & text info, status "Diproduksi" |
| `--color-info-dark` | `#1E40AF` | Text on info background |

### 2.5 Warna Status Order

| Status | Background | Text/Icon | Hex Combo |
|---|---|---|---|
| Pending | `#FEF3C7` | `#92400E` | Kuning |
| DP Diterima | `#DBEAFE` | `#1E40AF` | Biru muda |
| Diproduksi | `#E0E7FF` | `#3730A3` | Indigo |
| QC & Packing | `#EDE9FE` | `#5B21B6` | Ungu |
| Siap Kirim | `#D1FAE5` | `#065F46` | Hijau muda |
| Pelunasan | `#FEF3C7` | `#92400E` | Kuning |
| Dikirim | `#CFFAFE` | `#155E75` | Cyan |
| Selesai | `#D1FAE5` | `#065F46` | Hijau (bold) |
| Dibatalkan | `#F3F4F6` | `#4B5563` | Abu-abu |
| Overdue | `#FEE2E2` | `#991B1B` | Merah |

---

## 3. Typography

### 3.1 Font Family

**Heading & UI:** `Plus Jakarta Sans` (Google Fonts)
- Alasan: Font modern, clean, memiliki akar Indonesia, mirip dengan feel Gojek. Memiliki banyak weight. Free & open source.

**Body & Caption:** `Plus Jakarta Sans`
- Satu font family untuk konsistensi dan performa app.

**Monospace (untuk angka/kode):** `JetBrains Mono` atau `Roboto Mono`
- Untuk nomor invoice, harga, qty agar digit sejajar.

### 3.2 Type Scale

| Token | Size | Weight | Line Height | Penggunaan |
|---|---|---|---|---|
| `display-lg` | 28px | Bold (700) | 36px | Dashboard angka besar |
| `display-md` | 24px | Bold (700) | 32px | Page title |
| `heading-lg` | 20px | SemiBold (600) | 28px | Section heading |
| `heading-md` | 18px | SemiBold (600) | 24px | Card title |
| `heading-sm` | 16px | SemiBold (600) | 22px | Sub-section title |
| `body-lg` | 16px | Regular (400) | 24px | Body text utama |
| `body-md` | 14px | Regular (400) | 20px | Body text standar, form label |
| `body-sm` | 13px | Regular (400) | 18px | Secondary info |
| `caption` | 12px | Regular (400) | 16px | Caption, timestamp, helper text |
| `overline` | 11px | SemiBold (600) | 16px | Label uppercase, tag |
| `button` | 14px | SemiBold (600) | 20px | Button text |
| `number` | 14-28px | Medium (500) | — | Angka, harga (monospace) |

### 3.3 Aturan Typography
- **Judul halaman:** `display-md` + `--color-neutral-900`
- **Judul kartu:** `heading-md` + `--color-neutral-800`
- **Body text:** `body-md` + `--color-neutral-600`
- **Harga:** `number` + `--color-neutral-900` + monospace font
- **Status label:** `overline` + warna sesuai status
- **Angka currency:** Selalu gunakan format `Rp 1.500.000` (titik sebagai pemisah ribuan)
- **Tanggal:** Format `DD/MM/YYYY` atau `DD MMM YYYY` (contoh: `05 Jun 2026`)

---

## 4. Spacing & Layout

### 4.1 Spacing Scale (base 4px)

| Token | Value | Penggunaan |
|---|---|---|
| `space-1` | 4px | Minimal gap, inline elements |
| `space-2` | 8px | Compact padding, icon-text gap |
| `space-3` | 12px | Default padding komponen kecil |
| `space-4` | 16px | Default padding kartu, section gap |
| `space-5` | 20px | Padding kartu, gap antar section |
| `space-6` | 24px | Section padding, margin between cards |
| `space-8` | 32px | Large section gap |
| `space-10` | 40px | Page padding top |
| `space-12` | 48px | Extra large gap |

### 4.2 Layout Grid
- **Screen margin:** 16px (kiri-kanan)
- **Card gap:** 12px (vertikal antar card)
- **Content max width:** 428px (mobile)
- **Bottom nav height:** 64px
- **App bar height:** 56px
- **FAB size:** 56px (standard) / 40px (mini)
- **FAB position:** Bottom-right, 16px dari edge, di atas bottom nav

### 4.3 Safe Areas
- Respect notch / dynamic island (iOS)
- Bottom padding: 64px + 16px (nav height + margin) = 80px

---

## 5. Komponen UI

### 5.1 Buttons

#### Primary Button
```
Background: linear-gradient(135deg, #059669, #10B981)
Text: #FFFFFF, 14px SemiBold
Border Radius: 12px
Height: 48px
Padding: 0 24px
Shadow: 0 4px 14px rgba(5, 150, 105, 0.25)
Pressed: darken 10%, shadow removed
Disabled: opacity 0.4
```

#### Secondary Button
```
Background: #FFFFFF
Border: 1.5px solid #10B981
Text: #059669, 14px SemiBold
Border Radius: 12px
Height: 48px
Pressed: background #ECFDF5
```

#### Ghost Button
```
Background: transparent
Text: #059669, 14px SemiBold
Height: 48px
Pressed: background #ECFDF5
```

#### Danger Button
```
Background: #EF4444
Text: #FFFFFF, 14px SemiBold
Border Radius: 12px
Height: 48px
Shadow: 0 4px 14px rgba(239, 68, 68, 0.25)
```

### 5.2 Cards

#### Default Card
```
Background: #FFFFFF
Border Radius: 16px
Padding: 16px
Shadow: 0 1px 3px rgba(0, 0, 0, 0.08), 0 1px 2px rgba(0, 0, 0, 0.04)
Margin Bottom: 12px
```

#### Elevated Card (untuk stats dashboard)
```
Background: #FFFFFF
Border Radius: 16px
Padding: 20px
Shadow: 0 4px 6px rgba(0, 0, 0, 0.05), 0 10px 15px rgba(0, 0, 0, 0.03)
```

#### SPK / Order Card (di Kanban Board)
```
Background: #FFFFFF
Border Radius: 12px
Padding: 16px
Border Left: 4px solid [warna status]
Shadow: 0 1px 3px rgba(0, 0, 0, 0.08)
Margin Bottom: 12px

Content Layout:
  Header:
    - [SPK-001-A] (overline, warna primary)
    - Badge Status (Selesai/Progress/Waiting)
  Title: Nama Produk (16px SemiBold)
  Subtitle: Nama Pelanggan (14px Regular)
  Details:
    - Size & Qty Breakdown (S:20, M:30, dll)
    - Bahan & Warna
  Footer:
    - Karyawan Assigned (Avatar Keroyokan jika > 1)
    - Progress Bar mini
    - Deadline
```

### 5.3 Input Fields

```
Background: #F3F4F6
Border: 1.5px solid #E5E7EB
Border Radius: 12px
Height: 48px
Padding: 0 16px
Font: 14px Regular, #1F2937
Placeholder: 14px Regular, #9CA3AF
Focus Border: 1.5px solid #10B981
Focus Shadow: 0 0 0 3px rgba(16, 185, 129, 0.15)
Error Border: 1.5px solid #EF4444
Label: 13px SemiBold, #4B5563, margin-bottom 6px
Helper Text: 12px Regular, #9CA3AF, margin-top 4px
Password/PIN Field: Wajib menyertakan icon mata (Eye visibility toggle) di sebelah kanan input.
```

### 5.4 Status Badge / Chip

```
Border Radius: 20px (pill shape)
Padding: 4px 12px
Font: 11px SemiBold, UPPERCASE
Background & Text: sesuai tabel warna status (section 2.5)
```

### 5.5 Bottom Navigation Bar

```
Background: #FFFFFF
Height: 64px (+ safe area)
Shadow: 0 -1px 3px rgba(0, 0, 0, 0.08)
Border Top: none

Item Active:
  Icon: #059669 (filled)
  Label: 11px SemiBold, #059669

Item Inactive:
  Icon: #9CA3AF (outlined)
  Label: 11px Regular, #9CA3AF

Tab Items (4 items):
  1. 🏠 Beranda (Dashboard)
  2. 📋 Order
  3. ⚙️ Produksi  (untuk Staff: "Tugas Saya")
  4. 👤 Profil
```

### 5.6 Floating Action Button (FAB)

```
Size: 56px × 56px
Border Radius: 16px
Background: linear-gradient(135deg, #059669, #10B981)
Icon: + (white, 24px)
Shadow: 0 6px 20px rgba(5, 150, 105, 0.35)
Position: bottom-right, 16px dari edge, 80px dari bottom
Pressed: scale(0.95), shadow reduced

Expanded FAB (on tap):
  - "+ Order Baru"
  - "+ Pembayaran"  
  - "+ Kasbon" (hanya Staff)
```

### 5.7 App Bar / Header

```
Background: #FFFFFF
Height: 56px
Shadow: 0 1px 3px rgba(0, 0, 0, 0.06)
Title: 18px SemiBold, #1F2937
Back Icon: #374151, 24px
Action Icons: #374151, 24px
```

Variasi — **Greeting Header (Dashboard):**
```
Background: linear-gradient(135deg, #059669 0%, #047857 100%)
Height: auto (content-based)
Text Greeting: "Selamat Pagi, [Nama]" — 16px Regular, #FFFFFF
Text Role: "Super Admin • Cabang Serang" — 13px Regular, #A7F3D0
```

### 5.8 Empty State

```
Illustration: Simple line art / lottie animation
Title: 16px SemiBold, #374151
Description: 14px Regular, #9CA3AF
CTA Button: Primary Button (jika ada aksi)
```

### 5.9 Progress Bar (Produksi)

```
Track:
  Background: #E5E7EB
  Height: 8px
  Border Radius: 4px

Fill:
  Background: linear-gradient(90deg, #059669, #34D399)
  Border Radius: 4px
  Animation: width transition 300ms ease

Label:
  "[X]%" — 13px SemiBold, #059669
  "[120/200 pcs]" — 12px Regular, #6B7280
```

### 5.10 Kanban Board Column

```
Column Container:
  Background: #F9FAFB (Neutral 50)
  Border Radius: 16px
  Padding: 12px
  Width: 320px (horizontal scroll on mobile)
  
Column Header:
  Title: Nama Tahap (14px SemiBold, #1F2937)
  Counter Badge: Jumlah antrean SPK di kolom (12px SemiBold, Background #E5E7EB)
  
List Content:
  - SPK Cards (Vertical scrollable)
```

### 5.11 Notifikasi Toast

```
Background: #1F2937
Text: #FFFFFF, 14px Regular
Border Radius: 12px
Padding: 12px 16px
Position: Top, 16px dari safe area
Shadow: 0 4px 12px rgba(0, 0, 0, 0.15)
Duration: 3 detik
Animation: slide down + fade in
```

---

## 6. Iconography

### 6.1 Icon Set
**Library:** Lucide Icons (konsisten, modern, open source)
- Alternatif: Phosphor Icons

### 6.2 Icon Sizes
| Context | Size |
|---|---|
| Navigation bar | 24px |
| In-card action | 20px |
| Inline with text | 16px |
| Status / badge | 14px |
| FAB | 24px |

### 6.3 Icon Style
- **Stroke weight:** 1.5px (sesuai library)
- **Active:** Filled variant, warna primary
- **Inactive:** Outline variant, warna neutral-400

### 6.4 Pemetaan Ikon Unik Menu Dashboard

| Menu | Ikon Lucide | Karakter Unik Ikon | Ref Representasi |
|---|---|---|---|
| **Order Baru** | `FilePlus2` / `file-plus-2` | Kertas dengan tanda tambah bersudut | Pembuatan order baru |
| **Laporan Keuangan** | `TrendingUp` / `trending-up` | Grafik tren naik beraksen anak panah | Pertumbuhan omset finansial |
| **Pelanggan** | `Users2` / `users-2` | Dua figur siluet manusia bertumpuk | Hubungan mitra pelanggan |
| **Vendor** | `Truck` / `truck` | Siluet mobil box kargo | Distribusi bahan baku / maklon |
| **Karyawan** | `Contact2` / `contact-2` | Kartu ID dengan foto figur | Manajemen SDM |
| **Pengaturan Tarif** | `Percent` / `percent` | Simbol persentase miring | Pembagian persentase tarif borongan |
| **Transaksi Cabang** | `ArrowLeftRight` / `arrow-left-right` | Dua anak panah berlawanan arah horizontal | Mutasi finansial antar cabang |
| **Pengaturan Aplikasi** | `Sliders` / `sliders` | Tiga tuas pengontrol vertikal | Kustomisasi preferensi sistem |
| **Tugas Saya** | `ClipboardCheck` / `clipboard-check` | Papan klip dengan checklist sukses | Tugas produksi aktif karyawan |
| **Input Handover** | `Layers` / `layers` | Tiga tumpukan bidang 3D | Estafet serah terima antar divisi |
| **Ajukan Kasbon** | `Coins` / `coins` | Dua koin bertumpuk bergradasi | Pinjaman uang muka karyawan |
| **Riwayat Handover** | `History` / `history` | Jam berputar melingkar | Riwayat pengerjaan borongan |
| **Slip Gaji Saya** | `Receipt` / `receipt` | Struk belanja bergerigi di bawah | Slip gaji digital bulanan |

---

## 7. Motion & Animation

### 7.1 Durasi

| Type | Duration | Easing |
|---|---|---|
| Micro (button press, toggle) | 100-150ms | ease-out |
| Standard (card expand, modal open) | 200-300ms | ease-in-out |
| Complex (page transition) | 300-400ms | cubic-bezier(0.4, 0, 0.2, 1) |
| Loading skeleton | 1500ms | ease-in-out (loop) |

### 7.2 Transisi Halaman
- **Push:** Slide dari kanan
- **Pop:** Slide ke kanan
- **Modal:** Slide dari bawah
- **Tab switch:** Fade crossdissolve

### 7.3 Skeleton Loading
```
Background: linear-gradient(90deg, #F3F4F6 25%, #E5E7EB 50%, #F3F4F6 75%)
Animation: shimmer 1.5s ease-in-out infinite
Border Radius: mengikuti komponen asli
```

---

## 8. Ilustrasi & Imagery

### 8.1 Style
- **Ilustrasi:** Flat illustration style, menggunakan warna primary palette
- **Empty state:** Ilustrasi sederhana (line art) + warna emerald
- **Avatar default:** Inisial nama dengan background gradient emerald
- **Foto produk:** Aspect ratio 1:1, border-radius 12px

### 8.2 Avatar
```
Size: 40px × 40px (list), 48px × 48px (detail), 80px × 80px (profil)
Border Radius: 50% (circle)
Background: linear-gradient(135deg, #059669, #34D399)
Text: Inisial, 16px Bold, #FFFFFF
```

---

## 9. Responsive Behavior

### 9.1 Breakpoints (Flutter)
- **Mobile (default):** 0 - 599px
- **Tablet:** 600 - 1023px (jika dibutuhkan di masa depan)

### 9.2 Adaptasi
- Single column layout di mobile
- Bottom navigation selalu tampil
- FAB selalu tampil (kecuali di halaman form)
- Pull-to-refresh di semua halaman list
- Infinite scroll untuk daftar panjang (order, pelanggan)

---

## 10. Accessibility

- **Touch target:** Minimum 48px × 48px
- **Kontras teks:** Minimum ratio 4.5:1 (WCAG AA)
- **Font minimum:** 11px (untuk overline/label saja)
- **Color tidak menjadi satu-satunya indikator:** Selalu sertakan icon/text untuk status
- **Loading state:** Selalu tunjukkan indicator saat proses async

---

## 11. Komponen Overlay (Dialog, Bottom Sheet, Modal)

### 11.1 Bottom Sheet

```
Background: #FFFFFF
Border Radius: 24px 24px 0 0 (top corners only)
Padding: 24px 16px
Max Height: 85% screen height
Handle Bar: 40px × 4px, #D1D5DB, center, margin-bottom 16px
Overlay: #000000 opacity 0.4
Animation: slide up 300ms cubic-bezier(0.4, 0, 0.2, 1)
Dismissible: swipe down atau tap overlay

Penggunaan:
  - Pilih metode pembayaran
  - Pilih proses tujuan handover
  - Filter & sort options
  - Form input cepat (qty handover, kasbon)
```

### 11.2 Dialog / Alert Dialog

```
Background: #FFFFFF
Border Radius: 20px
Padding: 24px
Width: min(calc(100% - 48px), 360px)
Overlay: #000000 opacity 0.4
Animation: scale(0.9→1.0) + fade in 200ms ease-out

Title: 18px SemiBold, #1F2937, center
Body: 14px Regular, #6B7280, center
Spacing Title→Body: 8px
Spacing Body→Buttons: 24px

Buttons Layout: horizontal (2 buttons max)
  Primary: full primary style
  Secondary: ghost style
  Spacing between buttons: 12px

Penggunaan:
  - Konfirmasi hapus order
  - Konfirmasi logout
  - Error message (session expired, dll.)
  - Info dialog (order berhasil dibuat)
```

### 11.3 Confirmation Modal (Aksi Kritis)

```
Background: #FFFFFF
Border Radius: 20px
Padding: 24px

Icon: 48px, center, margin-bottom 16px
  - Sukses: ✅ checkmark (#10B981) dengan circular background #D1FAE5
  - Warning: ⚠️ triangle (#F59E0B) dengan circular background #FEF3C7
  - Danger: 🗑️ trash (#EF4444) dengan circular background #FEE2E2

Title: 18px SemiBold, #1F2937, center
Description: 14px Regular, #6B7280, center

Primary Button: full width
  - Sukses: Primary gradient
  - Danger: Danger button (#EF4444)
Cancel: Ghost button, full width, margin-top 8px

Penggunaan:
  - Konfirmasi handover estafet ("Serahkan 100pcs ke proses Jahit?")
  - Konfirmasi approve/tolak kasbon
  - Konfirmasi proses gaji
  - Konfirmasi hapus data (pelanggan, vendor, karyawan)
```

### 11.4 Dialog Bagi Porsi (Keroyokan)

```
Background: #FFFFFF
Border Radius: 20px
Padding: 24px

Header: "Bagi Porsi Pekerjaan" (18px SemiBold)
Sub: SPK-001-A - Kaos Polo (Total Sisa: 100 pcs)

Content (List Karyawan & Input Qty):
  - Row Karyawan 1 [Avatar] [Nama] -> Input Qty [___]
  - Row Karyawan 2 [Avatar] [Nama] -> Input Qty [___]
  - [+ Tambah Karyawan]
  
Footer:
  - Porsi Tersisa: 0 pcs (Teks Merah jika < 0)
  - [Simpan] (Primary Button)
```

### 11.5 Snackbar (Feedback Ringan)

```
Position: bottom, 16px dari bottom nav, full width - 32px margin
Background: #1F2937
Text: #FFFFFF, 14px Regular
Action Text: #34D399, 14px SemiBold (opsional, misal "BATAL")
Border Radius: 12px
Padding: 12px 16px
Height: auto (min 48px)
Shadow: 0 4px 12px rgba(0, 0, 0, 0.15)
Duration: 4 detik (auto dismiss)
Animation: slide up + fade in

Penggunaan:
  - "Handover berhasil disimpan"
  - "Pembayaran berhasil diinput"
  - "Kasbon berhasil diajukan"
  - "Order berhasil dihapus" + [BATAL]
```

---
