# OPUS_TASK_SOL_02 — Solidity compiler: type checker

Konteks: ini task kedua audit Solidity compiler (task pertama: codegen
storage/SSTORE — tertutup, 0 kandidat). Kamu mulai dari KOSONG (tidak ada
repo lokal). Target: https://github.com/ethereum/solidity — commit `41d9e65`
("Merge pull request #17224 from walnuthq/fix/json-integer-range-checks").
Repo bundle yang dihubungkan berisi kode yang dibutuhkan (lihat di bawah).

## Langkah 0 — clone dan baca daftar bug yang sudah diketahui

```
git clone --depth 1 https://github.com/ethereum/solidity.git solidity
```

BACA DULU `solidity/docs/bugs.json` — itu daftar bug compiler yang sudah
diketahui dan TIDAK eligible untuk bounty. Setiap kandidatmu harus dicek
terhadap daftar itu (dan sebutkan hasil ceknya).

## Sumber yang dipakai

Dari repo bundle (folder `sol-task02/`):
- `solidity/libsolidity/analysis/` — TypeChecker, DeclarationTypeChecker,
  OverrideChecker, ViewPureChecker, NameAndTypeResolver, Scoper,
  ConstantEvaluator, ImmutableValidator, StaticAnalyzer, ControlFlowAnalyzer
- `solidity/libsolidity/ast/` — Types, TypeProvider, AST, ASTUtils
- `solidity/libsolidity/interface/` — interface compiler
- `solidity/docs/bugs.json` — 67 entri bug yang sudah diketahui

Bila butuh file lain yang tidak ada di bundle, ambil dari GitHub pada
commit `41d9e65` dan catat file apa yang kamu ambil.

## Yang dicek (8 pemeriksaan)

1. **Konversi implisit.** `ast/Types.cpp` (fungsi konversi implisit
   antar tipe): integer / rational / literal, address vs address
   payable, bytes / string, tipe fungsi. Cari konversi yang seharusnya
   ditolak tapi diterima (mis. nilai yang overflow tipe tujuan, atau
   tipe yang tidak kompatibel lolos).
2. **Panggilan fungsi.** `analysis/TypeChecker.cpp` (visit FunctionCall):
   pencocokan argumen, overload resolution, named parameter, panggilan
   `abi.encode` / `abi.decode`. Cari panggilan yang seharusnya error
   tapi lolos, atau overload yang salah dipilih.
3. **Override.** `analysis/OverrideChecker.cpp`: override fungsi /
   variabel / modifier, linearisasi inheritance. Cari override yang
   seharusnya ditolak (signature tidak cocok, mutability berbeda,
   visibility berbeda) tapi diterima.
4. **View / pure.** `analysis/ViewPureChecker.cpp`: fungsi view/pure
   yang membaca / menulis state, termasuk lewat inline assembly dan
   panggilan internal. Cari jalur di mana fungsi pure bisa membaca
   state, atau view bisa menulis state, tanpa error.
5. **Data location.** `analysis/DeclarationTypeChecker.cpp`: aturan
   storage / memory / calldata untuk tipe referensi, termasuk transient
   storage dan array / struct bersarang. Cari deklarasi yang seharusnya
   ditolak (mis. storage di tempat yang tidak boleh) tapi lolos.
6. **Konstanta & immutable.** `analysis/ConstantEvaluator.cpp`,
   `analysis/ImmutableValidator.cpp`: evaluasi compile-time, penugasan
   immutable (hanya di constructor / saat deklarasi). Cari konstanta
   yang dihitung salah, atau immutable yang bisa diubah setelah
   constructor.
7. **Resolusi nama & scope.** `analysis/NameAndTypeResolver.cpp`,
   `analysis/Scoper.cpp`, `analysis/DeclarationContainer.cpp`:
   shadowing, `using for`, import, tabrakan nama. Cari nama yang
   seharusnya ambigu / tidak ditemukan tapi lolos, atau sebaliknya
   nama yang valid tapi ditolak.
8. **Control flow.** `analysis/ControlFlowAnalyzer.cpp`,
   `analysis/StaticAnalyzer.cpp`: return yang hilang, kode yang tidak
   terjangkau, urutan eksekusi modifier. Cari fungsi yang seharusnya
   error "missing return" tapi lolos (dan sebaliknya).

## Aturan

- JANGAN menjalankan build penuh / test suite di sesimu (kamu di Android).
  Kalau perlu bukti compile, tulis kontrak test kecil + perintah `solc`
  persisnya — Muse yang menjalankan di VM (solc hasil build dari commit
  `41d9e65` sudah ada di sana, versi 0.8.39-develop).
- Tandai tiap temuan "checked" (kamu baca kodenya langsung) atau
  "inferred" (penalaran dari kode tanpa menjalankan).
- JANGAN klaim "belum pernah dilaporkan" — Muse yang cek duplikat di GitHub.

## Format jawaban (Bahasa Indonesia)

- (A) Kandidat bug compiler — source tidak valid yang diterima, atau
  source valid yang ditolak, dengan dampak (kontrak yang seharusnya
  error tapi ter-deploy, atau sebaliknya).
  Tiap kandidat: file+baris, kontrak contoh minimal yang memicu,
  perilaku yang dihasilkan vs yang seharusnya, dampak.
- (B) Kandidat divergensi — type checker vs dokumentasi, atau
  perilaku yang berbeda antara `--via-ir` dan codegen lama untuk
  pola yang sama.
- (C) Temuan lemah / perlu cek lanjutan.
- 3 kandidat terkuat.
- 3 hal yang sudah dicek dan ternyata aman (dengan file+baris).
