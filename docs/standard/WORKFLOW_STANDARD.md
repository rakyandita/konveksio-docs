# 🔄 Workflow Standard — Konveksio

> **Versi:** 2.0 (Modular Refactor)
> **Sumber:** `.agents/agent_rules.md`

---

## Alur Kerja (Fase 0–3)

> **PENTING (Workflow Looping):** Jika di tengah Fase 1, 2, atau 3 User meminta penambahan fitur/teknologi baru yang belum terdaftar di SOT, workflow WAJIB di-pause. Agent harus kembali ke **Fase 0** (Update SOT) sebelum melanjutkan implementasi kode.

### Fase 0: Validasi SOT

> **Input:** Chat User / Scan Folder

1. Scan file sistem untuk seluruh **17 dokumen SOT** di `docs/` berdasarkan struktur di `DOCUMENTATION_STANDARD.md`.
2. Jika 17 dokumen SOT eksis DAN memiliki konten spesifikasi yang substantif (bukan sekadar file kosong): Lanjut ke Fase 1.
3. Jika struktur dokumen SOT belum lengkap: Agent WAJIB mewawancarai user untuk mendapatkan data yang hilang dan men-generate file SOT yang kurang.

---

### Fase 1: Project Scaffolding (Arsitektur)

> **Reference:** `docs/blueprint/PRODUCT_BLUEPRINT.md`, `docs/architecture/TECH_STACK.md`

1. Generate struktur folder utama.
2. Buat file konfigurasi (linting, environment, dependencies).
3. Buat file entry point aplikasi.
4. **Constraint:** Jangan buat UI atau logika bisnis di fase ini.

---

### Fase 2: Layout & Navigation (Shell)

> **Reference:** `docs/architecture/INFO_ARCHITECTURE.md`

1. Implementasi kerangka utama (Navbar, Sidebar, Footer).
2. Setup sistem routing berdasarkan IA.
3. Buat halaman kosong (skeleton) untuk semua rute yang terdaftar.
4. **Constraint:** Gunakan hanya warna/komponen dasar dari Design System.

---

### Fase 3: Detailed Implementation (Functional)

> **Reference:** `docs/blueprint/USER_FLOWS.md` & `docs/design/DESIGN_SYSTEM.md`

1. Implementasi komponen UI mendetail di setiap halaman.
2. Hubungkan logika bisnis sesuai alur user flow.
3. Pastikan semua aksi (button, form) memiliki feedback sesuai dokumentasi.

---

🔗 Untuk aturan logging, lihat: `docs/standard/DOCUMENTATION_STANDARD.md`
