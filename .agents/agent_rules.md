# 🤖 Project Konteks & AI Rules

Selamat datang Agent. Project ini menggunakan arsitektur dokumentasi modular (**17 SOT Documents**) untuk efisiensi context window.

**MANDATORY INSTRUCTION:**
Setiap kali Anda ditugaskan, Agent WAJIB memuat (`view_file`) file Universal Boilerplate ini terlebih dahulu, lalu mengidentifikasi domain tugas dan membaca HANYA dokumen dinamis yang secara spesifik relevan dengan tugas SEBELUM eksekusi kode.

## 🗺️ Peta Dokumentasi (17 SOT Index)

### 1. Aturan Dasar & Workflow (BACA PERTAMA KALI — WAJIB)
| Dokumen | Fungsi |
|---|---|
| `docs/standard/AI_AGENT_STANDARD.md` | Aturan absolut: SOT Supremacy, No side-quests, Pre-flight validation |
| `docs/standard/WORKFLOW_STANDARD.md` | Fase kerja 0-3, Workflow Looping |
| `docs/standard/DOCUMENTATION_STANDARD.md` | Aturan cetak Requirement Log, struktur direktori SOT |
| `docs/standard/GIT_STANDARD.md` | Branch naming, commit message format, PR rules, release tagging |

### 2. Produk & Arsitektur (BACA UNTUK KONTEKS FITUR)
| Dokumen | Fungsi |
|---|---|
| `docs/blueprint/PRODUCT_BLUEPRINT.md` | 13 modul MVP (M-01 s/d M-13), business rules, kasbon limit, HPP timing |
| `docs/blueprint/USER_FLOWS.md` | 12 user flows (UF-01 s/d UF-12) |
| `docs/architecture/SYSTEM_ARCHITECTURE.md` | Non-functional requirements, offline support, constraints |
| `docs/architecture/TECH_STACK.md` | Library wajib/terlarang, PHP extensions, Flutter packages |
| `docs/architecture/INFO_ARCHITECTURE.md` | 44 screens, navigasi, data model overview |

### 3. Implementasi Backend (BACA SAAT CODING LARAVEL)
| Dokumen | Fungsi |
|---|---|
| `docs/standard/CODING_STANDARD.md` | Service Pattern, 300-line limit, response format, soft delete, eager loading |
| `docs/standard/API_STANDARD.md` | Response envelope, pagination, filter/sort, versioning, file upload |
| `docs/standard/NAMING_CONVENTION.md` | Database column naming, model naming, route naming, Flutter naming |
| `docs/standard/ERROR_HANDLING_STANDARD.md` | Exception classes, logging strategy, DB::transaction, retry strategy |
| `docs/standard/DATA_MIGRATION_STANDARD.md` | Migration naming, foreign key, seeder, backfill |
| `docs/standard/TESTING_STANDARD.md` | Coverage target (80% service), AAA pattern, factory data |
| `docs/standard/SECURITY_PRIVACY_STANDARD.md` | RBAC matrix, branch isolation, JWT token, rate limiting, audit trail |

### 4. Implementasi Frontend (BACA SAAT CODING FLUTTER)
| Dokumen | Fungsi |
|---|---|
| `docs/design/DESIGN_SYSTEM.md` | Color tokens, typography, spacing, komponen UI, iconography |
| `docs/standard/NAMING_CONVENTION.md` | Flutter file/class/variable naming (Section 9) |

---

## ⚠️ Verification Script

Sebelum dan sesudah implementasi, Agent **WAJIB** menjalankan:

```bash
bash verify_sot.sh
```

Script ini mengecek **18 kategori** compliance terhadap SOT:
1. Model isolation traits
2. Service pattern (no Eloquent in controller)
3. File length ≤ 300 baris
4. Response envelope format
5. FormRequest tidak kosong
6. Soft delete untuk entitas transaksional
7. Library terlarang
8. N+1 query detection
9. Naming convention (boolean prefix)
10. Migration down() method
11. Foreign key constraints
12. No try-catch in controller
13. DB::transaction untuk multi-table write
14. Rate limiter configuration
15. Audit trail (LogsActivity)
16. Database seeders
17. Unit test existence
18. SOT document count (17)

**Jika script return ERROR, Agent WAJIB memperbaiki SEBELUM melanjutkan.**

### 📱 Mobile (Flutter) Verification
Untuk implementasi Flutter, Agent **WAJIB** menjalankan:

```bash
cd konveksio-mobile
bash verify_sot_mobile.sh
```

Script ini mengecek **27 kategori** compliance SOT untuk mobile:
1. File length ≤ 300 baris
2. File naming snake_case
3. Class naming PascalCase
4. Required packages (15 packages)
5. Forbidden packages
6. Dev dependencies
7. Folder structure (Clean Architecture)
8. API envelope handling
9. Token storage (secure storage)
10. API base URL (10.0.2.2 bukan localhost)
11. Offline support (connectivity_plus — CRITICAL)
12. Hive initialization (sebelum runApp)
13. Riverpod ProviderScope
14. Design System color tokens
15. Typography (Plus Jakarta Sans)
16. Screen naming ({Name}Screen)
17. UI overflow prevention (Expanded)
18. No hardcoded width
19. Currency formatter (Rp 1.500.000)
20. Date formatter
21. Feature structure (domain/data/presentation)
22. Shared widgets
23. dart analyze (zero errors)
24. Flutter SDK check
25. Status values consistency (Bahasa Indonesia)
26. Generated files (freezed/json_serializable)
27. SOT document count

**Jika script return ERROR, Agent WAJIB memperbaiki SEBELUM melanjutkan.**

---

## 🚫 Hard Rules

1. **DILARANG** menulis kode tanpa membaca SOT yang relevan
2. **DILARANG** skip `verify_sot.sh` (backend) atau `verify_sot_mobile.sh` (mobile) setelah implementasi
3. **DILARANG** menambah library di luar `TECH_STACK.md` tanpa update SOT
4. **DILARANG** membuat endpoint tanpa response envelope (`API_STANDARD.md`)
5. **DILARANG** commit tanpa mengikuti commit message format (`GIT_STANDARD.md`)
6. **DILARANG** pakai `localhost`/`127.0.0.1` di Flutter — WAJIB `10.0.2.2` untuk Android emulator

---
*Kegagalan membaca dokumen SOT yang relevan sebelum menulis kode akan dianggap sebagai pelanggaran Workflow kritis.*
