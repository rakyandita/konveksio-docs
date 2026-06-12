# 🤖 Project Konteks & AI Rules

Selamat datang Agent. Project ini menggunakan arsitektur dokumentasi modular (SOT Microservices) untuk efisiensi context window. 

**MANDATORY INSTRUCTION:**
Setiap kali Anda ditugaskan, Agent WAJIB memuat (`view_file`) file Universal Boilerplate ini terlebih dahulu, lalu mengidentifikasi domain tugas dan membaca HANYA dokumen dinamis yang secara spesifik relevan dengan tugas SEBELUM eksekusi kode.

## 🗺️ Peta Dokumentasi (SOT Index)

### 1. Aturan Dasar & Workflow (BACA PERTAMA KALI)
- `docs/standard/AI_AGENT_STANDARD.md` -> (Aturan absolut: No side quests, pre-flight checks)
- `docs/standard/WORKFLOW_STANDARD.md` -> (Fase kerja 0-3)
- `docs/standard/DOCUMENTATION_STANDARD.md` -> (Aturan cetak Requirement Log)

### 2. Produk & Arsitektur (BACA UNTUK KONTEKS FITUR)
- `docs/blueprint/PRODUCT_BLUEPRINT.md` -> (Ringkasan bisnis & daftar fitur/modul)
- `docs/blueprint/USER_FLOWS.md` -> (Alur langkah pengguna)
- `docs/architecture/SYSTEM_ARCHITECTURE.md` -> (Batasan sistem & infrastruktur)
- `docs/architecture/TECH_STACK.md` -> (Teknologi backend, frontend, db)

### 3. Implementasi Teknis (BACA SAAT CODING)
- `docs/design/DESIGN_SYSTEM.md` -> (Aturan UI, warna, komponen)
- `docs/standard/CODING_STANDARD.md` -> (Aturan anti-overflow, arsitektur Service Laravel)
- `docs/standard/TESTING_STANDARD.md` -> (Checklist sebelum commit/selesai tugas)
- `docs/standard/SECURITY_PRIVACY_STANDARD.md` -> (Matriks Hak Akses / RBAC)

---
*Kegagalan membaca dokumen SOT yang relevan sebelum menulis kode akan dianggap sebagai pelanggaran Workflow kritis.*
