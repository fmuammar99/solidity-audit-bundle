# OPUS_TASK_SOL_01 — Solidity compiler: codegen storage / SSTORE

Konteks: ini task pertama audit Solidity compiler. Kamu mulai dari KOSONG
(tidak ada repo lokal). Clone dulu, baca kode, jawab dalam Bahasa Indonesia.
Target: https://github.com/ethereum/solidity — commit `41d9e65`
("Merge pull request #17224 from walnuthq/fix/json-integer-range-checks").

## Langkah 0 — clone dan baca daftar bug yang sudah diketahui

```
git clone --depth 1 https://github.com/ethereum/solidity.git solidity
```

BACA DULU `solidity/docs/bugs.json` — itu daftar bug compiler yang sudah
diketahui dan TIDAK eligible untuk bounty. Setiap kandidatmu harus dicek
terhadap daftar itu (dan sebutkan hasil ceknya).

## Yang dicek (8 pemeriksaan)

1. **Storage layout.** `solidity/libsolidity/codegen/ContractCompiler.cpp`
   dan `CompilerContext.*`: penentuan slot storage, packing variabel,
   offset inheritance. Cari off-by-one, overlap slot, atau slot yang
   terlewat saat ada inheritance / variabel immutable.
2. **SSTORE codegen.** Cari jalur emit penyimpanan ke storage
   (mis. fungsi penyimpanan storage di ContractCompiler / CompilerUtils):
   apakah semua write ke storage memakai slot yang benar? Apakah ada
   jalur yang menulis ke slot salah (mis. setelah optimasi / inlining)?
   Cek juga transient storage (tstore/tload, EIP-1153) bila ada.
3. **`delete` storage.** `delete` pada array storage, struct storage,
   dan mapping: apakah semua slot benar-benar dibersihkan? Cari jalur
   `delete` yang melewatkan sebagian slot (sisa data lama = bug).
4. **ABI encoder: storage → memory/calldata.** `libsolidity/codegen/ir/`
   dan `ABIFunctions.*`: copy data storage ke memory/calldata untuk
   return / external call. Cari batas (bounds) yang salah, padding yang
   salah, atau copy yang kependekan/kepanjangan.
5. **IR codegen.** `libsolidity/codegen/ir/IRGeneratorForStatements.cpp`:
   akses storage di IR (sload/sstore hasil generate). Bandingkan dengan
   codegen lama (ContractCompiler) — adakah perilaku yang beda antara
   `--via-ir` dan codegen lama untuk pola storage yang sama?
6. **Yul optimizer & storage.** `solidity/libyul/optimiser/`: step
   optimizer yang menyentuh storage (mis. penghapus load/store,
   constant folding). Cari optimasi yang menghilangkan sstore yang
   seharusnya ada, atau menggabungkan dua sstore yang berbeda.
7. **ConstantEvaluator.** `libsolidity/codegen/ConstantEvaluator.*`:
   evaluasi compile-time atas nilai yang melibatkan storage / keccak.
   Apakah ada nilai yang di-fold secara salah?
8. **Cross-check bugs.json.** Untuk tiap kandidat di atas, cek apakah
   sudah ada di `docs/bugs.json` atau pola yang sama pernah dilaporkan.
   Tulis hasil ceknya per kandidat.

## Aturan

- JANGAN menjalankan build penuh / test suite di sesimu (kamu di Android,
  build solc butuh CMake + Boost dan berat). Kalau perlu bukti compile,
  tulis kontrak test kecil + perintah `solc` persisnya — Muse yang
  menjalankan di VM (solc 0.8.28 sudah terinstall di sana).
- Tandai tiap temuan "checked" (kamu baca kodenya langsung) atau
  "inferred" (penalaran dari kode tanpa menjalankan).
- JANGAN klaim "belum pernah dilaporkan" — Muse yang cek duplikat di GitHub.

## Format jawaban (Bahasa Indonesia)

- (A) Kandidat bug compiler — codegen salah untuk source valid.
  Tiap kandidat: file+baris, kontrak contoh minimal yang memicu,
  bytecode yang dihasilkan vs yang seharusnya, dampak (dana bisa dicuri
  / terkunci / perilaku salah).
- (B) Kandidat divergensi — `--via-ir` vs codegen lama berperilaku beda,
  atau solc vs dokumentasi.
- (C) Temuan lemah / perlu cek lanjutan.
- 3 kandidat terkuat.
- 3 hal yang sudah dicek dan ternyata aman (dengan file+baris).
