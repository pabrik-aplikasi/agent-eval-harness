# Brief — Coding-agent eval harness (nama kerja: pairbench)

Project `115-agent-eval-harness` · kategori 1 (dev tools) · Phase 0 Intake 14 Nov 2026, diperbarui setelah Phase 1 Discovery.
Dokumen untuk Thomas ditulis dalam bahasa Indonesia (aturan AGENTS.md project); UI, CLI dan pesan produk dalam bahasa Inggris (Batasan ide: "bahasa EN").

## Ide terpilih dalam 5 baris

1. **Masalah:** factory mengubah skill, prompt dan model tanpa bukti apakah perubahan itu membantu; satu run per sisi tidak bisa membedakan efek dari noise ([research.md](research.md#ringkasan-temuan)).
2. **Untuk siapa:** Thomas sebagai operator factory (primary), lalu developer yang merawat AGENTS.md/skill tim kecil ([personas.md](personas.md)).
3. **Apa:** CLI `pairbench` yang menjalankan suite task kecil dari repo sendiri terhadap dua konfigurasi (baseline vs kandidat: agent, model, mode, versi skill) dengan N ulangan di workspace sekali pakai, menilai dengan command pemeriksa yang disembunyikan dari agent, mencatat status, biaya, waktu dan token, lalu menampilkan perbandingan berpasangan dengan interval kepercayaan di dashboard lokal dan laporan Markdown.
4. **Kenapa sekarang:** wave 1 factory baru mulai (14 Nov 2026); retro Phase 13 lima project pertama akan mengubah skill, dan Harbor/Coder Eval tidak mempunyai adapter Amp (diperiksa dari source).
5. **Batasan:** OSS Apache-2.0, tanpa hosting wajib (dashboard lokal), pasar global berbahasa Inggris, batas biaya model per suite wajib, task dianggap tepercaya (operator sendiri).

## Asumsi

| # | Asumsi | Status setelah Phase 1 |
| --- | --- | --- |
| A1 | Agent bisa dijalankan non-interaktif dengan output mesin | **Terverifikasi dari docs:** `amp -x --stream-json`, `claude -p --output-format json`, `codex exec --json` ([research.md](research.md#bukti-tambahan-yang-memengaruhi-desain)) |
| A2 | Pemeriksa paling objektif = command/test setelah agent selesai | **Didukung:** SWE-bench, Harbor, SkillsBench memakai verifier deterministik; keterbatasan "test lulus ≠ kode bagus" dicatat sebagai risiko |
| A3 | Biaya per run bisa diambil dari output agent | **Sebagian:** Claude Code melaporkan `total_cost_usd` (estimasi klien); Codex dan Amp stream-json hanya token; `amp threads usage` menampilkan biaya per thread. Biaya disimpan dengan sumbernya |
| A4 | Variansi tinggi, butuh ulangan | **Terverifikasi:** [arXiv 2602.07150](https://arxiv.org/html/2602.07150), [arXiv 2609.33812](https://arxiv.org/html/2609.33812v1) |
| A5 | Suite MVP kecil (≤10 task) supaya murah | Tetap ASUMSI; estimasi US$9–120 per perbandingan 30 run ([research.md](research.md#pricing)) |
| A6 | Subprocess di workspace sementara bukan sandbox keamanan | Tetap; task tepercaya, isolasi kuat = Won't di MVP, orb Amp (`-ox`) sebagai opsi isolasi dicek di spike |
| A7 | Docker tersedia untuk isolasi | **Salah untuk orb ini:** daemon Docker tidak aktif (`docker ps` gagal, 14 Nov 2026). MVP tidak boleh bergantung pada Docker |

## Pertanyaan terbuka

Jawaban sementara (diputuskan agent, bisa dikoreksi Thomas di gate Phase 4):

1. Agent di MVP: **Amp (preset) + adapter command generik**; preset Claude Code dan Codex = Should.
2. Sumber task: **task buatan sendiri dari repo factory/project** (format satu file + fixture); impor benchmark publik = Won't.
3. Dashboard: **lokal**, dibuka dari CLI (`pairbench serve`); tanpa hosting.
4. Biaya token: dibayar akun agent pengguna; batas biaya per suite wajib, run berhenti saat batas tercapai.
5. Ukuran "lebih baik": pass rate berpasangan (utama), biaya per task lulus, durasi; skor kualitas LLM-judge = Could.

Masih terbuka untuk Thomas: **build vs adopt** (bangun pairbench, atau tulis adapter Amp untuk Harbor/Coder Eval). Dibawa ke gate Phase 4.
