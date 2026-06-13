# 🔀 Git Standard — Konveksio

> **Versi:** 1.0
> **Dibuat:** 13 Juni 2026
> **Status:** MANDATORY — Semua developer dan AI Agent WAJIB mengikuti standar ini

---

## 1. Branch Strategy

### 1.1 Branch Utama

| Branch | Fungsi | Protected? |
|---|---|---|
| `main` | Production-ready code. Selalu deployable. | ✅ Ya — tidak boleh direct push |
| `develop` | Integration branch. Semua feature branch merge ke sini. | ✅ Ya — minimal 1 review |

### 1.2 Branch Fitur

Format: `{type}/{module-id}-{deskripsi-singkat}`

| Type | Prefix | Contoh |
|---|---|---|
| Feature baru | `feature/` | `feature/M-02-crud-order` |
| Bug fix | `fix/` | `fix/nv-01-hpp-controller` |
| Hotfix (urgent prod) | `hotfix/` | `hotfix/login-token-expired` |
| Refactor | `refactor/` | `refactor/extract-payment-service` |
| Documentation | `docs/` | `docs/add-api-standard` |
| Test | `test/` | `test/kasbon-service-coverage` |

### 1.3 Aturan Branch

- **Satu branch = satu task/feature.** Jangan campur beberapa perubahan tidak terkait dalam 1 branch.
- **Branch fitur selalu dibuat dari `develop`**, bukan dari `main`.
- **Hotfix dibuat dari `main`**, lalu di-merge ke `main` DAN `develop`.
- **Maksimal umur branch: 7 hari.** Jika lebih, segera merge atau rebase.
- **Branch naming WAJIB lowercase**, kebab-case.

---

## 2. Commit Message

### 2.1 Format

```
[type] scope: deskripsi singkat

[body opsional — penjelasan detail jika perlu]
```

### 2.2 Type

| Type | Kapan Digunakan |
|---|---|
| `feat` | Fitur baru |
| `fix` | Perbaikan bug |
| `refactor` | Refactor tanpa perubahan fungsi |
| `test` | Menambah/memperbaiki test |
| `docs` | Perubahan dokumentasi |
| `style` | Formatting, whitespace, semicolon (tanpa perubahan logika) |
| `chore` | Build process, dependencies, tooling |
| `perf` | Perubahan untuk performa |

### 2.3 Scope

Gunakan module ID atau nama komponen:
- `M-02` — Manajemen Order
- `M-03` — Tracking Produksi
- `M-05` — Invoice & Pembayaran
- `M-09` — Karyawan & Penggajian
- `auth` — Autentikasi
- `infra` — Infrastructure / config

### 2.4 Contoh Commit Message

```
[feat] M-02: add StoreOrderRequest validation rules
[fix] M-05: fix invoice total calculation excluding delivery cost
[refactor] M-03: move kalkulasiHpp from controller to ProduksiService
[test] M-09: add PayrollServiceTest for borongan calculation
[docs] standard: add API_STANDARD.md with response envelope
[chore] config: update composer.lock after adding jwt-auth
[perf] M-02: add eager loading for customer relationship in order list
```

### 2.5 Aturan Commit

- **Commit message dalam Bahasa Inggris** (untuk konsistensi git log)
- **Imperative mood:** "add feature" bukan "added feature" atau "adds feature"
- **Maksimal 72 karakter** untuk baris pertama (subject)
- **Jangan commit file sensitif:** `.env`, `storage/`, `vendor/`, `node_modules/`
- **Setiap commit harus buildable** — jangan commit kode yang tidak bisa jalan

---

## 3. Pull Request

### 3.1 Aturan PR

- **Target branch:** `develop` (kecuali hotfix → `main`)
- **Minimal 1 approval** sebelum merge (jika ada reviewer)
- **PR title:** sama dengan commit message format
- **PR description:** wajib berisi ringkasan perubahan

### 3.2 Template PR Description

```markdown
## Ringkasan
[Deskripsi singkat perubahan]

## Module
[M-XX: Nama Module]

## Tipe Perubahan
- [ ] Feature baru
- [ ] Bug fix
- [ ] Refactor
- [ ] Test
- [ ] Documentation

## Checklist
- [ ] Kode mengikuti CODING_STANDARD.md
- [ ] Response mengikuti API_STANDARD.md
- [ ] Test sudah pass (jika applicable)
- [ ] Tidak ada linting error
- [ ] SOT sudah diupdate jika ada perubahan requirement
```

### 3.3 Merge Strategy

- **Feature branch → develop:** Squash merge (1 commit per feature)
- **develop → main:** Merge commit (pertahankan history)
- **Hotfix → main:** Merge commit

---

## 4. Release & Tagging

### 4.1 Versioning (Semantic Versioning)

Format: `v{major}.{minor}.{patch}`

| Bagian | Kapan Naik | Contoh |
|---|---|---|
| Major | Breaking change (redesign API, ubah DB schema besar) | `v2.0.0` |
| Minor | Fitur baru backward-compatible | `v1.1.0` |
| Patch | Bug fix, hotfix | `v1.0.1` |

### 4.2 Pre-release Tags

| Tag | Keterangan |
|---|---|
| `v1.0.0-alpha` | Development preview, belum lengkap |
| `v1.0.0-beta` | Feature complete, masih ada bug |
| `v1.0.0-rc.1` | Release candidate, final testing |
| `v1.0.0` | Production release |

### 4.3 Tagging Process

```bash
# Dari branch main, setelah merge dari develop
git tag -a v1.0.0-alpha -m "Alpha release: MVP backend complete"
git push origin v1.0.0-alpha
```

### 4.4 Changelog

Setiap release WAJIB memiliki changelog di `CHANGELOG.md`:

```markdown
## [v1.0.0-alpha] - 2026-06-13

### Added
- M-02: CRUD Order dengan bundle expansion
- M-05: Invoice generation dengan public link
- M-09: Payroll borongan calculation

### Fixed
- Fix branch isolation bypass untuk super admin
- Fix HPP calculation missing biaya_vendor

### Changed
- Migrate from SQLite to PostgreSQL
```

---

## 5. .gitignore Rules

### 5.1 Wajib di-gitignore

```
# Dependencies
/vendor/
/node_modules/

# Environment
.env
.env.local
.env.production

# IDE
.idea/
.vscode/
*.swp

# OS
.DS_Store
Thumbs.db

# Build
/public/build/
/public/hot/

# Storage
/storage/*.key
/storage/logs/*.log
/storage/framework/cache/*
/storage/framework/sessions/*
/storage/framework/views/*

# Testing
.phpunit.result.cache
coverage/

# Flutter
.dart_tool/
.flutter-plugins
.flutter-plugins-dependencies
build/
*.iml
```

### 5.2 WAJIB di-commit

```
# Selalu commit
composer.json
composer.lock
package.json
package-lock.json
pubspec.yaml
pubspec.lock
database/migrations/*.php
database/seeders/*.php
config/*.php
routes/*.php
app/**/*.php
resources/views/**/*.php
docs/**/*.md
```

---

## 6. AI Agent Git Rules

### 6.1 Aturan Khusus Agent

- Agent **TIDAK BOLEH** push langsung ke `main` tanpa approval
- Agent **BOLEH** push ke feature branch
- Agent **WAJIB** membuat commit message mengikuti format di atas
- Agent **WAJIB** mention module ID (`M-XX`) di commit message jika perubahan terkait fitur bisnis
- Agent **DILARANG** force push ke branch yang sudah di-push sebelumnya
- Agent **DILARANG** mengubah git config (`user.name`, `user.email`, hooks)

### 6.2 Commit Frequency

- **Satu logical change = satu commit**
- Jangan commit per-file (terlalu granular)
- Jangan commit semua sekaligus (terlalu kasar)
- Contoh ideal: 1 feature = 2-5 commit (migration, model, service, controller, test)

---

## 7. Conflict Resolution

### 7.1 Rebase vs Merge

- **Feature branch dari develop:** Rebase (`git rebase develop`)
- **develop dari main:** Merge (`git merge main`)
- **Hotfix:** Merge saja

### 7.2 Aturan Conflict

- Jika conflict di migration file: **JANGAN** ubah migration orang lain. Diskusikan dulu.
- Jika conflict di `composer.lock`: Jalankan `composer install` ulang setelah merge.
- Jika conflict di SOT docs (`docs/`): Prioritaskan versi terbaru yang sudah disetujui user.
