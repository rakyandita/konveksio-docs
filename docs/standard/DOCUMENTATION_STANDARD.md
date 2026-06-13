# 📝 Documentation Standard — Konveksio

> **Versi:** 1.0
> **Dibuat:** 11 Juni 2026

---

## Aturan Requirement Log (Wajib)

Setiap kali agent menyelesaikan satu iterasi tugas (satu sesi kerja yang menghasilkan perubahan kode/file), agent **WAJIB** mencetak tabel **Requirement Log** di bagian akhir responnya.

### Format Wajib

```markdown
| ID | Phase | Source Ref | Requirement Name | Status |
|---|---|---|---|---|
| R-XX | Phase N | [Dokumen SOT] | [Nama Requirement] | [TODO/IN_PROGRESS/DONE] |
```

### Kolom yang Wajib Diisi

| Kolom | Keterangan |
|---|---|
| **ID** | Nomor unik requirement, format `R-XX` (angka urut) |
| **Phase** | Fase kerja: Phase 0, Phase 1, Phase 2, atau Phase 3 |
| **Source Ref** | Dokumen SOT sumber: `PRODUCT_BLUEPRINT`, `USER_FLOWS`, `DESIGN_SYSTEM`, `INFO_ARCHITECTURE`, `TECH_STACK`, `SECURITY_PRIVACY_STANDARD`, `CODING_STANDARD`, atau `All SOTs` |
| **Requirement Name** | Nama singkat fitur/task yang dikerjakan |
| **Status** | `[TODO]`, `[IN_PROGRESS]`, atau `[DONE]` |

### Aturan Tambahan

1. **Jangan hapus entri lama.** Requirement Log bersifat kumulatif — entri lama dari sesi sebelumnya tetap dipertahankan.
2. **Gunakan ID yang konsisten.** Jika R-08 sudah ada di sesi sebelumnya, entry baru dimulai dari R-09.
3. **Minor Revision (Bugfixing):** Jika tugas adalah perbaikan *bug/error* dari Requirement sebelumnya, JANGAN buat ID baru yang meloncat. Gunakan ID lama dengan versi minor (contoh: `R-09.1`) atau status `[FIXING]`.
4. **Status harus akurat.** Jangan tandai `[DONE]` jika pekerjaan belum selesai sepenuhnya.

---

## Aturan Pembaruan Dokumen SOT

- Jika ada keputusan desain atau perubahan requirement yang disepakati dalam percakapan, agent **WAJIB** memperbarui dokumen SOT yang relevan sebelum mengimplementasikan kode.
- Urutan wajib: **Update SOT → Implementasi Kode → Requirement Log**.

---

## Struktur Direktori Dokumentasi

```
docs/
├── architecture/
│   ├── TECH_STACK.md
│   ├── SYSTEM_ARCHITECTURE.md
│   └── INFO_ARCHITECTURE.md
├── blueprint/
│   ├── PRODUCT_BLUEPRINT.md
│   └── USER_FLOWS.md
├── design/
│   └── DESIGN_SYSTEM.md
└── standard/
    ├── AI_AGENT_STANDARD.md
    ├── WORKFLOW_STANDARD.md
    ├── SECURITY_PRIVACY_STANDARD.md
    ├── CODING_STANDARD.md
    ├── TESTING_STANDARD.md
    ├── API_STANDARD.md
    ├── NAMING_CONVENTION.md
    ├── ERROR_HANDLING_STANDARD.md
    ├── GIT_STANDARD.md
    ├── DATA_MIGRATION_STANDARD.md
    └── DOCUMENTATION_STANDARD.md
```
