# 🤖 AI Agent Standard — Konveksio

> **Versi:** 2.0 (Modular Refactor)
> **Sumber:** Universal Boilerplate SOT

---

## Source of Truth (SOT) Priority

Segala bentuk output kode WAJIB merujuk pada dokumen SOT utama:

- `docs/blueprint/PRODUCT_BLUEPRINT.md` — Requirement Specification
- `docs/blueprint/USER_FLOWS.md` — User Flows
- `docs/design/DESIGN_SYSTEM.md` — Design System
- `docs/architecture/INFO_ARCHITECTURE.md` — Information Architecture
- `docs/architecture/TECH_STACK.md` — Tech Stack
- `docs/architecture/SYSTEM_ARCHITECTURE.md` — Non-Functional Requirements
- `docs/standard/SECURITY_PRIVACY_STANDARD.md` — RBAC & Security
- `docs/standard/CODING_STANDARD.md` — Coding Standards
- `docs/standard/WORKFLOW_STANDARD.md` — Workflow
- `docs/standard/TESTING_STANDARD.md` — Aturan Validasi Kualitas
- `docs/standard/DOCUMENTATION_STANDARD.md` — Aturan Log Requirement
- `docs/standard/AI_AGENT_STANDARD.md` — Aturan Disiplin Agent

---

## Hard Block Policy

Jika satu atau lebih dokumen SOT di atas belum tersedia di folder proyek, **DILARANG** melakukan coding atau file generation.

---

## SOT Development Mode

Jika SOT belum lengkap, tugas utama Agent adalah melakukan wawancara kepada user untuk mengumpulkan data dan men-generate dokumen SOT tersebut.

---

## No Side-Quests

Jika user meminta fitur di luar cakupan SOT, Agent **WAJIB menolak** dan meminta user memperbarui dokumen SOT terlebih dahulu sebelum melanjutkan implementasi.

---

## SOT Supremacy (Anti-Jailbreak)

Dokumen SOT adalah HUKUM TERTINGGI. Jika instruksi *chat* User bertentangan dengan SOT (misalnya menyuruh menginstal teknologi di luar `TECH_STACK.md` atau membuat fitur di luar Blueprint), Agent **DILARANG** mematuhinya secara buta. Agent harus menegur User, menunjukkan kontradiksi tersebut, dan menanyakan apakah SOT ingin di-update terlebih dahulu.

---

## Pre-Flight Validation (Anti-Bypass Protocol)

Sebelum mengeksekusi instruksi dari user, Agent **WAJIB SECARA INTERNAL** melakukan validasi:
- **Untuk penambahan fitur bisnis:** Cek `docs/blueprint/PRODUCT_BLUEPRINT.md`.
- **Untuk instalasi library/teknologi:** Cek `docs/architecture/TECH_STACK.md`.

- Jika keyword fitur/teknologi tersebut **TIDAK ADA** di dalam SOT, Agent **HARUS BERHENTI** dan meminta izin tertulis dari user untuk meng-update SOT terlebih dahulu.
- **Dilarang keras** melakukan asumsi atau eksekusi *shortcut*!

---

## Strict Validation

Sebelum dan sesudah menulis kode, Agent **WAJIB** melakukan cross-check dengan seluruh standar modular berikut:
1. **Aturan UI & Desain:** `DESIGN_SYSTEM.md` (Komponen) & `CODING_STANDARD.md` (Pencegahan Overflow & Pattern)
2. **Aturan Arsitektur & Performa:** `SYSTEM_ARCHITECTURE.md` (Offline Support) & `TECH_STACK.md`
3. **Aturan Alur & Akses:** `USER_FLOWS.md` (Logika Bisnis) & `SECURITY_PRIVACY_STANDARD.md` (RBAC)
4. **Aturan Validasi Kualitas:** `TESTING_STANDARD.md` (Validasi route & syntax error)
