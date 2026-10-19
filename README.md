# fe-guideline

Paket panduan untuk AI coding agent (Claude Code, Codex, Gemini, OpenCode, dsb.) dan tim frontend. Isinya **stack-agnostic**: aturan & proses sama untuk Next.js, React, dan Angular; detail per stack ada di `references/<stack>.md`. Bisa dipakai untuk kerja standar maupun kerja ber-timebox (lomba, hackathon, POC) lewat `mode` di stack profile.

> **Status: v0.2 — divalidasi dengan template Next.js dan Angular.** `fe-starter-nextjs` (Next.js 16) dan `fe-starter-angular` (Angular 21), masing-masing dengan fitur contoh `topup`, lolos lint + boundaries, typecheck, test, dan build (mock & HTTP); `references/nextjs.md` dan `references/angular.md` ditulis ulang dari kode itu. `references/react.md` **masih draf** sampai template React dibuat.

---

## Struktur

```
fe-guideline/                    # repo ini = sumber paket; di repo target disalin sebagai folder guideline/
├── README.md                    # file ini
├── PROMPTS.md                   # cheat sheet prompt untuk tim (manusia)
├── AGENTS.md, CLAUDE.md         # instruksi untuk MERAWAT paket ini (tidak ikut disalin)
├── AGENTS.template.md           # → jadi AGENTS.md di root repo target
├── CLAUDE.template.md           # → jadi CLAUDE.md di root repo target (pointer @AGENTS.md)
├── stack-profile.template.md    # diisi per project: stack, command, layer map, DI, state, test
├── references/                  # SATU file per stack: metode yang direkomendasikan + template kode
│   ├── nextjs.md
│   ├── react.md
│   └── angular.md
├── skills/                      # SUMBER skill (diedit di sini saja)
│   ├── tier-1/                  # wajib — dipakai hampir setiap tugas
│   ├── tier-2/                  # penting — sesuai jenis tugas
│   └── tier-3/                  # pelengkap — saat tim makin matang
├── agents/                      # subagent reviewer (architecture, test, ui)
├── lint/                        # ESLint boundaries: aturan import antar layer dipaksa mesin
├── hooks/                       # hook agent: blokir edit salinan skill, lint file yang baru diedit
├── scaffold/                    # format awal file milik project: page-map, second-brain, api-contracts, ASSUMPTIONS,
│                                #   layer-agents/ (AGENTS.md per folder src/)
└── scripts/
    └── sync-skills.mjs          # salin skills/tier-*/* → .claude/skills, .agents/skills, ... (+ --check untuk CI)
```

## Cara kerja agnostic

```
Skill (agnostic: aturan & proses)
   └─ baca <guideline>/stack-profile.md      → stack apa, command, folder, DI, state, test runner
        └─ baca <guideline>/references/<stack>.md → metode & template kode untuk stack itu
```

Cukup perintahkan agent: **"baca `references/<stack>.md` sesuai stack yang dipakai"** — semua skill sudah menunjuk ke sana. Bila `stack-profile.md` belum ada atau kosong, skill **berhenti dan bertanya**, tidak menebak — kecuali di `mode: competition`, di mana agent memakai default dan mencatatnya di `ASSUMPTIONS.md` (lihat `bp-competition-mode`).

`<guideline>` = folder paket ini di repo target (default: `guideline/`; ubah di `AGENTS.md` target bila dipindah).

## Katalog skill

| Tier | Skill | Ringkas |
|---|---|---|
| 1 | `bp-architecture` | Layer, slicing per fitur, port/adapter, mapper wajib, `Result`, error bertipe, kapan usecase dibuat |
| 1 | `bp-feature-lifecycle` | Orkestrator fitur: discover → spec → domain → ports → application → infra → DI → UI → verify |
| 1 | `bp-ui-components` | Atomic design, atom wajib dari shared UI, tanpa native HTML, fallback `'-'`, a11y dasar |
| 1 | `bp-conventions` | Naming, batas ukuran file, done criteria |
| 1 | `bp-git-workflow` | Commit, staging, branch, deskripsi MR/PR |
| 1 | `bp-competition-mode` | Override untuk kerja ber-timebox: fase lifecycle diringkas, test minimum, asumsi dicatat alih-alih bertanya |
| 2 | `bp-api-integration` | Kontrak API → endpoint, raw type, mapper, kode BE → `AppError`, mock adapter |
| 2 | `bp-testing` | Strategi test per layer (test-first wajib untuk logika murni) |
| 2 | `bp-figma-to-code` | Klasifikasi surface, source of truth, kegagalan akses Figma |
| 2 | `bp-security` | OWASP frontend |
| 2 | `bp-debugging` | Reproduksi → root cause → fix + regression test |
| 3 | `bp-second-brain` | Kurasi pelajaran teknis + promotion ladder ke lint rule |
| 3 | `bp-release` | Versioning, changelog, tag |
| 3 | `bp-performance` | Prinsip performa agnostic + rujukan per stack |
| 3 | `bp-refactor-legacy` | Migrasi bertahap ke pola baru |

Prinsip kurasi: **satu topik = satu skill pemilik**; skill lain hanya merujuk (mencegah kontradiksi antar-skill).

## Cara mengadopsi ke project baru

1. Salin isi repo ini (tanpa `.git`, `AGENTS.md`, `CLAUDE.md` milik repo ini) ke folder `guideline/` di root repo target. Nama folder boleh diganti; sesuaikan `<guideline>` di langkah 2.
2. Salin `guideline/AGENTS.template.md` → `AGENTS.md` dan `guideline/CLAUDE.template.md` → `CLAUDE.md` di root repo target (hapus komentar TEMPLATE). Bila repo target sudah punya file sejenis, gabungkan — jangan ada dua sumber instruksi.
3. Salin `stack-profile.template.md` → `stack-profile.md`, isi sesuai stack terpilih (termasuk `mode`, versi tooling, dan skill vendor bila dipakai — kandidatnya di `references/<stack>.md` § 1).
   Opsional sejak awal: salin `scaffold/page-map.md` dan `scaffold/api-contracts/` ke `<guideline>/` (skill juga akan membuatnya saat pertama dibutuhkan).
   Salin `scaffold/layer-agents/src/` ke `src/` repo target (instruksi singkat per folder layer; lewati `core/di/` untuk Angular).
4. Jalankan `node <guideline>/scripts/sync-skills.mjs` → skill tersalin flat ke folder agent (`.claude/skills/`, `.agents/skills/`, ...). **Jangan edit hasil salinan**; edit di `skills/tier-*/`.
5. Pasang `lint/` ke konfigurasi ESLint repo (lihat `lint/README.md`), sesuaikan path dengan layer map di `stack-profile.md`, lalu jalankan uji sanity: import `presentation → infrastructure` harus membuat lint gagal.
6. Bangun **satu fitur contoh** lewat `bp-feature-lifecycle`. Bandingkan hasilnya dengan `references/<stack>.md`; perbaiki reference dari kode yang sudah direview (bukan sebaliknya).
7. Tambahkan `node <guideline>/scripts/sync-skills.mjs --check` ke CI agar salinan skill tidak drift.
8. (Claude Code) Gabungkan `hooks/settings.example.json` ke `.claude/settings.json` agar edit ke salinan skill diblokir dan file yang diedit langsung di-lint (lihat `hooks/README.md`).
9. Bagikan `PROMPTS.md` ke tim sebagai panduan memulai pekerjaan dengan agent.

## Upgrade paket di project yang sudah memakai

File di `<guideline>/` terbagi dua:

| Milik paket (boleh ditimpa versi baru) | Milik project (**jangan ditimpa**) |
|---|---|
| `skills/`, `references/`, `agents/`, `lint/`, `hooks/`, `scripts/`, `scaffold/`, `*.template.md`, `README.md`, `PROMPTS.md` | `stack-profile.md`, `page-map.md`, `second-brain/`, `api-contracts/`, salinan `src/**/AGENTS.md` dari `layer-agents` |

Langkah: salin versi baru hanya untuk kolom kiri → bandingkan `AGENTS.template.md` baru dengan `AGENTS.md` root project dan gabungkan perubahannya → jalankan `sync-skills.mjs` → cek `stack-profile.template.md` untuk field baru dan tambahkan ke `stack-profile.md`.

## Aturan pemeliharaan paket

1. Satu sumber: skill hanya diedit di `skills/tier-*/`; file agent lain hanya pointer/salinan hasil script.
2. Skill baru wajib didaftarkan di tabel routing `AGENTS.template.md` dan di katalog README ini.
3. Tidak ada path mesin, username, drive letter, kredensial, atau token di file mana pun.
4. Perubahan pola di kode acuan wajib memperbarui skill/reference terkait di MR yang sama.
5. Skill inti tidak menyebut framework. Detail framework hanya di `references/<stack>.md` dan `stack-profile.md`.
