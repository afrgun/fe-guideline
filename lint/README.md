# Lint — Architecture Boundaries

Aturan import antar layer **dipaksa oleh lint**, bukan hanya ditulis di skill. Skill `bp-architecture` adalah penjelasan; file di folder ini adalah penegaknya.

## Isi

| File | Fungsi |
|---|---|
| `eslint.boundaries.mjs` | Flat config (sintaks `eslint-plugin-boundaries` v7): elemen layer, kebijakan siapa boleh import siapa, `no-unknown-files`, larangan framework di domain/application/shared-lib, larangan HTTP di presentation |

## Elemen

| Elemen | Pola | Boleh di-import oleh |
|---|---|---|
| `app` | `src/app/**`, `src/*.{ts,tsx}` (file entry framework), `src/features/*/*.routes.ts` | tidak ada (kecuali `app`) — composition root |
| `di` | `src/core/di/**` | `presentation`, `app` — accessor DI (`usePorts`, `createServerPorts`) |
| `kernel` | `src/shared/kernel/**` | semua |
| `shared-lib` | `src/shared/lib/**` | semua layer kecuali `kernel` dan `di` — helper murni lintas fitur (mis. `formatOrDash`) |
| `shared-ui` | `src/shared/ui/**` | `presentation`, `app` |
| `core` | `src/core/**` | `infrastructure`, `di`, `app` |
| `domain` / `application` / `infrastructure` / `presentation` | `src/features/<feature>/<layer>/**` | sesuai `bp-architecture`; lintas fitur dilarang |

Urutan elemen penting: **elemen pertama yang cocok yang dipakai** — `app` dan `di` sengaja ditaruh sebelum `core`.

## Pemasangan

1. Install versi yang di-pin (catat juga di `stack-profile.md` § Tooling versions):
   ```
   eslint@^9  eslint-plugin-boundaries@^7  eslint-import-resolver-typescript@^4  typescript-eslint@^8
   ```
2. Di `eslint.config.mjs` repo target:
   ```js
   import boundaries from './guideline/lint/eslint.boundaries.mjs';
   export default [...stackConfigs, ...boundaries];
   ```
3. Sesuaikan `pattern` dengan **Layer map** di `stack-profile.md`.
4. Taruh file setup test (mis. `vitest.setup.ts`) **di luar `src/`**, atau tambahkan elemen untuknya — `no-unknown-files` akan menolak file `src/` yang tidak masuk layer mana pun.
5. Config ditulis untuk **v7; jangan downgrade**. Satu-satunya warning yang diharapkan adalah deprecation `mode: 'file'` untuk elemen file tunggal (alasannya di komentar config).

## Composition root per stack

| Stack | `app` | `di` |
|---|---|---|
| Next.js | `src/app/**` + `src/proxy.ts` (dulu `middleware.ts`) dsb. | `src/core/di/**` (`server-ports.ts`; `client-ports-context.tsx` hanya bila ada adapter di browser) |
| React | `src/app/**` + `src/main.tsx` | `src/core/di/**` (`ports.ts`, `ports-context.tsx`) |
| Angular | `src/app/**` + `src/main.ts` + `src/features/*/*.routes.ts` | — (DI bawaan Angular lewat providers di route; tidak ada `core/di`) |

## Config tambahan per stack

| Stack | Tambahkan |
|---|---|
| Next.js | `eslint-config-next`, `eslint-plugin-jsx-a11y`; **`no-restricted-imports` agar `src/features/*/presentation/client/**` tidak meng-import `**/server/**` maupun `@/core/di/server-ports`** |
| React | `eslint-plugin-react-hooks`, `eslint-plugin-jsx-a11y` |
| Angular | `angular-eslint` (TS + template, termasuk aksesibilitas template) |

## Uji sanity

Setelah memasang, tambahkan sementara import `presentation → infrastructure` di satu fitur; `lint` **harus gagal**. Bila lolos, boundaries tidak aktif.

## Adopsi bertahap di repo lama

Aktifkan dulu hanya untuk `src/features/**`, biarkan folder legacy di luar `boundaries/include`. Perluas cakupan seiring migrasi (`bp-refactor-legacy`).

## Promotion ladder

Pelajaran di second brain yang terulang 2–3 kali dijadikan rule di sini (atau di config stack), lalu catatannya ditandai `**Promoted**` (`bp-second-brain`).
