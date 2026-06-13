# 🚀 AI Agent Framework & Prompt Architecture

Panduan ini mengatur pola **"AI Agent Framework"** tingkat lanjut (*Enterprise-Grade AI Guardrails*) yang dirancang agar *reusable* (dapat didaur ulang) untuk semua proyek pengembangan perangkat lunak ke depannya.

Proteksi anti-halusinasi dan *anti-bypass* telah disuntikkan secara presisi ke dalam *Universal Boilerplate* ini, sehingga AI (LLM) tidak lagi bisa mencari alasan, mengambil jalan pintas, atau melenceng dari konteks.

Dalam arsitektur modular ini, dokumen *Source of Truth* (SOT) terbagi menjadi dua kategori besar:

---

## 1. 🛡️ Universal Boilerplate (Statis/Bawaan)

Isi dari file-file ini nyaris **tidak pernah berubah** apa pun jenis aplikasinya (baik itu aplikasi Konveksi, E-Commerce, maupun Sistem Rumah Sakit). File-file ini berfungsi sebagai "Sistem Operasi" atau *Trigger* bagi AI untuk bekerja sesuai standar kedisiplinan yang ketat.

**File yang WAJIB disalin (*copy-paste*) ke setiap project baru:**
- `.agents/agent_rules.md` — File *Index/Router* yang pertama kali dibaca oleh AI.
- `docs/standard/WORKFLOW_STANDARD.md` — Kerangka kerja sistematis (Fase 0-3) dan *Workflow Looping*.
- `docs/standard/DOCUMENTATION_STANDARD.md` — Aturan cetak *Requirement Log* dan struktur direktori.
- `docs/standard/AI_AGENT_STANDARD.md` — Aturan absolut kedisiplinan agent (Anti-Jailbreak, *Pre-Flight Validation*).
- `docs/standard/GIT_STANDARD.md` — Branch strategy, commit format, PR rules, release tagging.

---

## 2. 🧩 Project-Specific (Dinamis)

File-file ini akan di-*generate* dan berevolusi oleh AI berdasarkan instruksi spesifik di setiap proyek. Isinya 100% bergantung pada *scope* aplikasi yang sedang dibangun.

Struktur file dinamis yang akan dibentuk (di dalam folder `docs/`):
- **Blueprint:** `PRODUCT_BLUEPRINT.md` & `USER_FLOWS.md`
- **Arsitektur:** `SYSTEM_ARCHITECTURE.md`, `TECH_STACK.md`, `INFO_ARCHITECTURE.md`
- **Desain:** `DESIGN_SYSTEM.md`
- **Standar Khusus:**
  - `SECURITY_PRIVACY_STANDARD.md` — RBAC, branch isolation, token, rate limit, audit trail
  - `CODING_STANDARD.md` — Service Pattern, response format, soft delete
  - `API_STANDARD.md` — Response envelope, pagination, filter/sort
  - `NAMING_CONVENTION.md` — Database, model, controller, Flutter naming
  - `ERROR_HANDLING_STANDARD.md` — Exception, logging, transaction
  - `TESTING_STANDARD.md` — Coverage, AAA pattern, factory
  - `DATA_MIGRATION_STANDARD.md` — Migration, seeder, backfill

---

## 3. 🔍 Verification Script

File `verify_sot.sh` berfungsi sebagai **automated compliance checker** yang mengecek 18 kategori aturan SOT. Agent WAJIB menjalankan script ini sebelum dan sesudah implementasi.

```bash
bash verify_sot.sh
```

Exit code 0 = LULUS | Exit code 1 = ADA PELANGGARAN (wajib diperbaiki)

---

## 🪄 Panduan Memulai Proyek Baru ("The Magic Trigger")

Ketika Anda atau tim memulai proyek baru di masa depan (misal: membuat Aplikasi Kasir), Anda tidak perlu lagi mengajari AI cara bekerja dari nol. Cukup ikuti 2 langkah mudah ini:

### Langkah 1: Persiapan Folder
Salin (*copy-paste*) kelima file **Universal Boilerplate** di atas ke dalam folder proyek baru Anda yang masih kosong. Pastikan struktur foldernya dipertahankan (`.agents/` dan `docs/standard/`).

### Langkah 2: Eksekusi Prompt Pertama
Kirimkan *prompt* pertama Anda kepada Agent AI (misal: Antigravity/Cursor/Windsurf) menggunakan format berikut:

> *"Halo Agent. Kita akan membuat [NAMA_APLIKASI]. Tolong baca `.agents/agent_rules.md` dan `docs/standard/WORKFLOW_STANDARD.md`, lalu mulai **Fase 0** dengan men-generate kerangka file SOT yang dinamis (Blueprint, Arsitektur, dll) berdasarkan spesifikasi awal berikut: [MASUKKAN_IDE_ANDA_DI_SINI]."*

Dengan *trigger* di atas, AI akan otomatis mengadopsi standar kerja tingkat tinggi yang ketat, melakukan validasi sebelum eksekusi, dan bekerja secara terarah tanpa halusinasi.
