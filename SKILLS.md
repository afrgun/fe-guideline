# Panduan Skill

Penjelasan untuk tim: setiap skill di paket ini **untuk apa**, **isinya apa**, dan **contoh prompt** yang memicunya. Aturan lengkapnya ada di `skills/tier-*/<skill>/SKILL.md`, contoh kasusnya di `examples.md` di folder yang sama (memakai fitur fiktif `topup`).

Ada 15 skill dalam tiga tingkat:

| Tier | Kapan dipakai |
|---|---|
| **1** | Dasar — hampir setiap tugas |
| **2** | Sesuai jenis pekerjaan |
| **3** | Pelengkap saat tim makin matang — **tidak dipakai** di `mode: competition` |

Tidak perlu menyebut nama skill di prompt: agent memilihnya lewat tabel routing di `AGENTS.md`. Contoh kalimat lengkap per kebutuhan: [`PROMPTS.md`](./PROMPTS.md).

---

## Tier 1 — dasar

### 1. `bp-architecture` — peta rumah kodenya
- **Untuk:** menentukan di mana sebuah kode harus ditaruh.
- **Isi:** daftar layer (`domain`, `application`, `infrastructure`, `presentation`, `core`, `shared`, `app`) beserta apa yang boleh dan dilarang di tiap layer, plus 11 aturan inti:
  - setiap panggilan ke luar lewat port;
  - port mengembalikan `Result` dan tidak pernah melempar error;
  - mapper wajib;
  - aturan bisnis berupa fungsi murni di `domain/`;
  - use case hanya dibuat kalau ada logikanya;
  - endpoint di satu file.
- **Contoh:** saat ragu "aturan minimal top up ditaruh di komponen atau di domain?"

### 2. `bp-feature-lifecycle` — urutan kerja membangun fitur
- **Untuk:** membangun fitur dari nol sampai selesai, tanpa ada yang terlewat.
- **Isi:** fase 0–8: tanya kebutuhan → spec test → domain → port → use case → infrastructure → wiring DI → UI → verifikasi. Ada juga mode *plan only* (dry run) dan ringkasan di akhir.
- **Contoh prompt:** "Bangun fitur top up dari tiket ABC-482, mulai fase 0."

### 3. `bp-ui-components` — aturan membuat tampilan
- **Untuk:** UI konsisten, tidak dibuat ulang di setiap halaman.
- **Isi:**
  - atomic design (atom → molekul → organisme → halaman);
  - wajib memakai atom shared (tidak boleh `<button>` mentah; atom boleh berupa komponen atau directive pada elemen native);
  - nilai kosong ditampilkan lewat `formatOrDash`, sehingga muncul `'-'`;
  - pesan error dari message map pusat;
  - state loading, kosong, dan error wajib ada;
  - aksesibilitas dasar.
- **Contoh prompt:** "Buat atom SegmentedControl untuk pilihan nominal."

### 4. `bp-conventions` — standar penamaan dan definisi "selesai"
- **Untuk:** keseragaman nama file, fungsi, port, adapter, dan mapper, serta batas ukuran file.
- **Isi:** **done criteria**. Pekerjaan baru dianggap selesai kalau lint, typecheck, test, dan build sudah dijalankan dan hasilnya dilaporkan, bukan sekadar "seharusnya jalan".
- **Contoh prompt:** "Jalankan done criteria dan laporkan hasilnya."

### 5. `bp-git-workflow` — commit, branch, dan MR
- **Untuk:** commit yang rapi dan mudah dilacak.
- **Isi:**
  - format pesan commit (dari stack profile) dan nomor tiket dari URL;
  - staging eksplisit, tanpa `git add -A` sembarangan;
  - bertanya dulu kalau ada perubahan yang tidak terkait;
  - template deskripsi MR/PR.
- **Contoh prompt:** "Commit perubahan top up, tiket ABC-482" atau "tuliskan commit message saja".

### 6. `bp-competition-mode` — mode cepat untuk lomba, hackathon, atau POC
- **Untuk:** kerja ber-timebox tanpa merusak struktur.
- **Isi:** tabel aturan yang dilonggarkan (test cukup untuk domain dan mapper, spec dilewati) dan aturan yang **tetap wajib** (port, lint, keamanan). Agent tidak bertanya, tapi memakai default dan mencatatnya di `ASSUMPTIONS.md`. Pengecualiannya: aksi destruktif, secret, dan ganti stack tetap harus ditanyakan.
- **Aktif kalau:** `mode: competition` di `stack-profile.md`.

---

## Tier 2 — sesuai jenis tugas

### 7. `bp-api-integration` — menyambungkan endpoint backend
- **Untuk:** dari kontrak API sampai data siap dipakai.
- **Isi:**
  - baca atau simpan kontrak di `guideline/api-contracts/`;
  - konstanta endpoint dan tipe data mentah;
  - mapper (normalisasi enum dan tanggal, tidak mengarang field wajib);
  - cek kode sukses HTTP **dan** kode bisnis, lalu kode error bisnis dipetakan ke `AppError`;
  - adapter mock yang melewati mapper asli dan tidak boleh ikut ke bundle production.
- **Contoh prompt:** "Integrasikan `GET /topup/v1/banks`, ini contoh responsnya…"

### 8. `bp-testing` — apa yang wajib dites dan caranya
- **Untuk:** strategi test per layer.
- **Isi:**
  - **test-first wajib** (red → green → refactor) untuk domain, mapper, use case, dan logika bercabang di store; kode produksi tidak boleh ditulis sebelum test-nya dijalankan dan gagal;
  - **bukti red → green wajib** di ringkasan (cuplikan output gagal lalu lolos); tanpa bukti dianggap belum selesai;
  - use case dites dengan fake port;
  - adapter dites dengan HTTP palsu;
  - komponen dites lewat perilaku yang terlihat user;
  - format file spec, dan daftar anti-pattern (snapshot besar, memalsukan logika sendiri).
- **Contoh prompt:** "Tulis test untuk `validateAmount` dan mapper bank, test-first."

### 9. `bp-figma-to-code` — dari desain ke kode
- **Untuk:** mengimplementasikan desain Figma atau screenshot dengan benar.
- **Isi:**
  - klasifikasi jenis perubahan (fitur baru, redesign, ekstrak komponen, dll.);
  - urutan sumber kebenaran (Figma > screenshot > repo);
  - apa yang dilakukan kalau Figma tidak bisa dibuka;
  - tabel pemetaan elemen desain ke komponen;
  - token warna dan spacing, bukan hex langsung di komponen.
- **Contoh prompt:** "Implementasikan frame ini `<link Figma>`, klasifikasikan dulu."

### 10. `bp-security` — keamanan frontend (OWASP)
- **Untuk:** mencegah celah umum.
- **Isi:** HTML dari API wajib disanitasi, redirect harus dari allow-list, token tidak boleh di `localStorage`, tidak ada secret di kode client, pesan error backend tidak ditampilkan mentah, guard route (menyembunyikan menu bukan otorisasi), tanpa PII di analytics, dan tidak ada kredensial di file agent. **Berlaku di semua mode.**
- **Contoh prompt:** "Cek perubahan ini dengan checklist keamanan."

### 11. `bp-debugging` — mencari akar masalah sebelum memperbaiki
- **Untuk:** memperbaiki bug tanpa sekadar menambal gejalanya.
- **Isi:**
  - reproduksi dulu, lalu telusuri layer (page-map → network → mapper → domain) sampai ketemu layer pertama yang salah;
  - satu hipotesis, satu perubahan;
  - wajib ada regression test;
  - format laporan (gejala, akar masalah, perbaikan, test).
- **Contoh prompt:** "Daftar bank kosong untuk sebagian user, cari root cause dulu."

---

## Tier 3 — pelengkap

### 12. `bp-second-brain` — catatan pelajaran teknis tim
- **Untuk:** menyimpan pelajaran dari commit dan MR (bug teknis, performa, keputusan arsitektur, tooling) supaya kesalahan yang sama tidak terulang.
- **Isi:**
  - 5 kategori catatan dan aturan filter (fitur dan styling tidak dicatat);
  - **promotion ladder:** pelajaran yang terulang 2–3 kali dinaikkan jadi lint rule atau aturan skill.
- **Contoh prompt:** "Update second brain dari commit sejak marker terakhir."

### 13. `bp-release` — rilis versi
- **Untuk:** menentukan nomor versi (MAJOR/MINOR/PATCH) dari daftar commit, menulis changelog yang bisa dibaca user, dan membuat tag.
- **Isi:** wajib minta konfirmasi sebelum tag production; tag tidak boleh dipindah.
- **Contoh prompt:** "Siapkan rilis berikutnya, minta konfirmasi sebelum tag."

### 14. `bp-performance` — memperbaiki aplikasi yang lambat
- **Untuk:** optimasi yang terukur.
- **Isi:**
  - **ukur dulu** (waterfall request, request ganda, ukuran bundle, re-render);
  - perbaiki sesuai urutan biaya terbesar;
  - laporkan angka sebelum dan sesudah;
  - jangan memoize semuanya tanpa bukti.
- **Contoh prompt:** "Halaman top up lambat, ukur dulu baru usulkan perbaikan."

### 15. `bp-refactor-legacy` — memindahkan kode lama ke pola baru
- **Untuk:** migrasi bertahap tanpa merusak perilaku.
- **Isi:**
  - characterization test dulu, lalu satu aspek per langkah dan setiap langkah tetap hijau;
  - perubahan perilaku yang ditemukan dilaporkan, bukan diam-diam diperbaiki;
  - compatibility shim sementara yang punya tiket penghapusan.
- **Contoh prompt:** "Pindahkan logika top up dari `TopupPage` lama ke pola baru, bertahap."

---

## Subagent reviewer (`agents/`)

| Subagent | Fungsi |
|---|---|
| `architecture-reviewer` | Memeriksa perubahan terhadap aturan layer |
| `ui-reviewer` | Memeriksa atom, state loading/kosong/error, dan aksesibilitas |
| `test-writer` | Menulis test sesuai `bp-testing` |

---

Saat menambah atau mengubah skill, perbarui juga file ini, tabel routing di `AGENTS.template.md`, dan katalog di `README.md`.
