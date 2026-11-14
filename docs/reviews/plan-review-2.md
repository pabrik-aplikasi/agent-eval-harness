# Plan review — round 2 (terakhir)

14 Nov 2026. Subagent CRITIC dan DISCUSSANT baru dengan konteks segar, paralel, membaca PRD hasil ronde 1 dan [plan-review-1.md](plan-review-1.md). Model sama dengan penulis (tidak ada thread lain yang boleh dibuat). Klaim yang bisa diperiksa tanpa biaya model diperiksa sebelum diputuskan. Ini ronde terakhir; sisa item dibawa ke [progress.md](../progress.md).

Severity: B = BLOCKER, M = MAJOR, m = MINOR, D = usulan DISCUSSANT.

| # | Source | Severity | Finding | Decision | Reason | Applied in |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | CRITIC | B | Klaim "FPR ≤ 5%" salah untuk nol "rata-rata sama" dengan skor 0–1 dan sel tidak seimbang (simulasi reviewer: 22.5% pada 6 vs 2 slot dengan sebaran berbeda) | ACCEPT | Uji permutasi exact hanya di bawah nol tajam. Hipotesis nol dinyatakan sebagai "label arm tidak memengaruhi distribusi skor per task"; untuk skor biner setara pass rate sama; selftest menambah sel tidak seimbang dan skor 0–1 | G1, US-08 Metode, M-8 |
| 2 | CRITIC | B | AC-03.2 (`find /` tidak menemukan check) bertentangan dengan klaim keamanan yang dipersempit | ACCEPT | AC-03.2 hanya menguji yang dijamin; AC-03.3 negatif terdokumentasi; chmod 000 salinan check selama agent berjalan (pola Coder Eval) | AC-03.2–03.3, §6 |
| 3 | CRITIC + DISCUSSANT | B | Tool plugin/built-in akun (`slack_write`, `notify_user`, `trigger_pipeline`) lolos dari gerbang yang hanya memeriksa MCP | ACCEPT | Terlihat di daftar tool sesi ini. Allowlist tool per arm; attempt dihentikan saat init sebelum pesan assistant pertama; diuji dengan tool palsu; spike R11 untuk mematikan lewat settings; fallback service account | US-02, AC-05.3, §7, R11 |
| 4 | CRITIC | B | `--pilot` tanpa admission rule dan bertentangan dengan `repeats ≥ 2` | ACCEPT | Pilot berurutan dengan batas keras setelah setiap attempt (lebih paling banyak 1 attempt), dikecualikan dari AC-02.6 | AC-06.2, AC-02.6 |
| 5 | CRITIC | M | MDE tidak terdefinisi, terlalu lambat; aturan selftest ambigu | ACCEPT | Uji exact lewat konvolusi (tanpa Monte Carlo) membuat simulasi murah; MDE didefinisikan; Clopper-Pearson per sel, grid 27 sel + sel khusus, ≤ 10 menit | US-08, AC-06.7, AC-08.10, §6 |
| 6 | CRITIC | M | Kontrak p kurang: definisi dua sisi, ties, cabang Monte Carlo tanpa AC, toleransi bootstrap rapuh | ACCEPT | p = P(\|Δ*\| ≥ \|Δobs\|) dengan aritmetika rasional; cabang Monte Carlo dihapus; bootstrap direproduksi persis dengan RNG kontrak | US-08, AC-08.3 |
| 7 | CRITIC | M | `insufficient_data` vs `repeats ≥ 2` tidak konsisten; satu task rusak membatalkan suite | ACCEPT | `plan` menolak desain yang p terkecilnya ≥ α; task < 2 slot dikeluarkan dan didaftar, tidak membatalkan suite | AC-02.9, US-08, AC-08.4 |
| 8 | CRITIC | M | Semua error Amp jadi `infra_error` → eksklusi bergantung arm, bias | ACCEPT | Infra hanya pola provider/jaringan/rate-limit/auth; konteks penuh dsb. = `fail`; tanda `unbalanced_exclusions` | AC-04.3, US-08, AC-08.11 |
| 9 | CRITIC | M | Admission: statistik estimasi, skip vs tunggu, blok tidak seimbang, AC-06.4 trivial, biaya terlambat, float | ACCEPT | p90, admission per blok, berhenti pada penolakan pertama, micro-dollar, M-6 toleransi `max_parallel`, AC dengan biaya 2× estimasi, polling biaya | US-06, M-6, AC-04.1 |
| 10 | CRITIC | M | Resume: PID turunan tidak tercatat, PID basi, tanpa lock, aturan antre ulang | ACCEPT | `PAIRBENCH_ATTEMPT_ID` di environ + waktu mulai proses; lock; `cancelled`/`interrupted` diantrekan dan dihitung | US-07, §6 |
| 11 | CRITIC | M | Identitas hanya hash `experiment.yaml` | ACCEPT | Identitas mencakup isi task dan overlay | US-02, AC-02.5 |
| 12 | CRITIC | M | Environment check tidak dispesifikasi (conftest bisa membocorkan token) | ACCEPT | Check di environment ter-scrub tanpa kredensial agent; AC conftest | US-03, AC-03.11, AC-01.7 |
| 13 | CRITIC | M | Rahasia bocor lewat diff/HTML/export ke data-dir yang di-commit; masking berlebihan | ACCEPT | Masking di semua artefak, hanya rahasia yang dideklarasikan, peringatan data-dir di repo | Istilah, AC-03.12, AC-09.4 |
| 14 | CRITIC | M | "AGENTS.md global" tanpa AC; workspace di dalam repo memuat AGENTS.md repo | ACCEPT | Workspace di `$TMPDIR`, data-dir di `$XDG_STATE_HOME`; AGENTS.md dicatat dan yang asing ditolak; instruksi level akun di server = spike R11 | US-03, AC-05.6, R11 |
| 15 | CRITIC | M | Milestone salah urut (admission, gerbang tool, store sesudah attempt nyata M1) | ACCEPT | US-05 inti, US-06, US-09 pindah ke M1 | §9 |
| 16 | CRITIC | M | Service account tanpa baris §7; spike R11 gagal = adapter utama tak terpakai; biaya build UNKNOWN | ACCEPT | Baris §7 dengan aturan fallback; batas biaya build diajukan ke Thomas (Options) | §7, §10 |
| 17 | CRITIC | M | Batas waktu tidak dinyatakan; M0 tidak dihitung; US-08 terlalu kecil | ACCEPT | Batas ≤ 20 hari; M0 2 hari; US-08 dipecah; total 18.5 | §10 |
| 18 | CRITIC | m | Randomisasi berblok vs retry di akhir | ACCEPT | Retry di dalam blok; asumsi pertukaran dinyatakan | AC-04.4, US-08 |
| 19 | CRITIC | m | `stats --status` tanpa AC | ACCEPT | M-2 diukur dari tabel status `report` | M-2 |
| 20 | CRITIC | m | Window M-1/M-3 | ACCEPT | M-1 menyebut kebutuhan eksperimen keputusan; M-3 "mana yang lebih dulu" | §2, §10 |
| 21 | CRITIC | m | Interval [1,1] di samping `inconclusive` | ACCEPT | Catatan "interval is descriptive" | US-08 |
| 22 | CRITIC | m | `max_parallel` di orb 3 GB | ACCEPT | Peringatan > 2, peak RSS | §6 |
| 23 | CRITIC | m | CI push ke main memakai kredit tanpa izin | ACCEPT | Hanya `workflow_dispatch` | AC-12.2 |
| 24 | CRITIC | m | AC-05.2 di `plan` tanpa workspace | ACCEPT | `plan` membuat workspace sementara | AC-05.2 |
| 25 | DISCUSSANT | D | HTML statis ke Should | REJECT | Platform di ide menyebut web dashboard; total Must 18.5 ≤ 20 hari setelah item lain diturunkan. Dicatat di Options | — |
| 26 | DISCUSSANT | D | Rentang biaya p10–p90 dan `prune` ke Should | ACCEPT | Kosmetik; admission yang menjamin G3 | §4, US-06, US-09 |
| 27 | DISCUSSANT | D | Data-dir sebagai JSONL teks | ACCEPT | Export hampir gratis, diff terbaca; batasan untuk Phase 5 | US-09 |
| 28 | DISCUSSANT | D | Adapter generik tetap Must karena menjadi agent uji e2e | ACCEPT | Alasan ditulis | US-04 |
| 29 | DISCUSSANT | D | Faktual: Coder Eval sudah punya `paired_comparison` (task sebagai unit), `--resume`, budget USD, plugin agent, anti-cheat chmod | ACCEPT | **Diperiksa di source** commit `d8011f0` (`reports/helpers.py`, `cli/run_command.py`, `fs_permissions.py`); research.md dan §1 dikoreksi | research.md, §1 |
| 30 | DISCUSSANT | D | Rekomendasi build vs adopt harus berupa aturan keputusan | ACCEPT | Lihat Options 1 | Options |
| 31 | DISCUSSANT | D | Pairbench-lite sebagai kriteria exit spike Phase 5 (ukur biaya dan varians nyata, MDE > 40 pp = ubah desain suite) | ACCEPT | Butuh izin biaya ±US$3–10 | R1, Options 2 |
| 32 | DISCUSSANT | D | Option 4 jadi pertanyaan ya/tidak | ACCEPT | | Options 4 |
| 33 | DISCUSSANT | D | Kebijakan keputusan untuk `inconclusive` di skill retro | ACCEPT sebagai Options | Perubahan skill factory butuh persetujuan Thomas | Options 5 |
| 34 | DISCUSSANT | D | Suite contoh 3 task factory | ACCEPT | Lihat di bawah | US-01, §9 |
| 35 | DISCUSSANT | D | Kontaminasi lintas arm lewat file factory di orb | ACCEPT | R15 + workspace di luar repo + README | R15, US-03 |
| 36 | DISCUSSANT | D | Deteksi skill benar-benar dimuat | ACCEPT | AC-05.4 | US-05 |
| 37 | DISCUSSANT | D | Overlay `git:<repo>@<rev>` | ACCEPT | AC-02.8 | US-02 |
| 38 | DISCUSSANT | D | Tag `regression`/`targeted` | ACCEPT sebagai Should | R14 | §4, R14 |
| 39 | DISCUSSANT | D | M-1 nol bila skill retro tidak berubah | ACCEPT | Usulan perubahan skill retro di M3 | §9 M3 |
| 40 | DISCUSSANT | D | Thread Amp per attempt menumpuk di daftar thread | Carried over | Label thread lewat `amp label` setelah attempt, dicek di spike | Bawah |

## Suite contoh

Dari DISCUSSANT, disesuaikan. Biaya per attempt **ASUMSI** US$0.20–0.60 (diukur di spike Phase 5). 3 task × 3 ulangan × 2 arm = 18 attempt.

1. **`prd-stories-lint`** (skill `prd-writer`). Fixture: project fiktif kecil (antrean klinik 3 fitur) dengan `docs/brief.md`, `research.md`, `personas.md`, `docs/prd.md` §1–4 terisi (3 baris Must dengan story ID) dan §5 kosong, plus AGENTS.md project factory. Instruksi: tulis §5 untuk scope Must. Check (skor = rata-rata, 0 bila tidak ada story): heading `### US-xx` untuk setiap ID Must; ≥ 2 baris `AC-\d+\.\d+ Given .+ When .+ Then` per story; baris `States:` memuat kelima state; tanpa `TBD|TODO|nanti` di §4–5; kalimat story diawali "Sebagai". `protected_paths`: §1–4 dan file sumber. Oracle: §5 tulisan tangan.
2. **`readiness-planted-defects`** (skill `readiness-check`). Fixture: PRD kecil lengkap dengan 3 cacat ditanam (TBD di AC story Must, metrik tanpa "How measured", risiko High tanpa mitigasi) dan `docs/reviews/plan-review-1.md` tanpa BLOCKER. Check: `docs/reviews/readiness.md` ada; `**Verdict:** NO-GO`; item 2, 3, 4 gagal dan 1, 5–8 lulus (8 subcheck); "Fixes needed" menyebut ID lokasi cacat; PRD tidak berubah. Varian bersih yang harus GO ditambahkan kemudian.
3. **`personas-from-saved-sources`** (skill `idea-discovery`, tanpa web). Fixture: `sources/` berisi 6–8 halaman tersimpan dengan URL dan tanggal di header, template persona. Check: 2 persona + 1 anti-persona; semua field template ada; setiap `**Quote:**` muncul verbatim di salah satu file `sources/` dan URL-nya cocok; setiap Pain punya URL dari `sources/` atau `ASUMSI`.

## Options for Thomas

1. **Build vs adopt.** Fakta terbaru: Coder Eval (Apache-2.0, Python) sudah punya runner `tempdir`, repeats, resume, budget USD per run, perbandingan berpasangan task-unit, plugin agent, anti-cheat chmod; tidak punya Amp, uji exact bersyarat suite, admission budget per eksperimen, scrub env, gerbang tool/MCP, provenance skill Amp, subreaper. Perkiraan reviewer: adopsi menghemat ±5–7 dari ±18.5 hari. **Aturan keputusan yang diusulkan:** Phase 5 menghabiskan maksimal 1.5 hari agent untuk plugin Amp Coder Eval + script verdict exact pada suite contoh. Bila menghasilkan laporan A/A benar dan field provenance Amp, **adopt** dan kontribusikan upstream; bila tidak, **build** (stack Rust/Effect sesuai rencana, nilai belajar triage = 5). Alternatif: Thomas memilih build langsung karena nilai belajar.
2. **Spike Phase 5 dengan biaya nyata.** Spike menjalankan suite contoh di Amp (±12 attempt, perkiraan US$3–10, ASUMSI) untuk mengukur biaya, varians dan MDE. MDE > 40 pp = desain suite diubah sebelum M2. Butuh persetujuan biaya.
3. **Laporan HTML statis di Must** (keputusan sekarang) atau turun ke Should (hemat ±1–1.5 hari, MVP tanpa UI web sama sekali).
4. **Model biaya (ya/tidak):** apakah attempt eval memakai kuota langganan Megawatt tanpa biaya marjinal? Bila tidak dijawab, batas US$1 (M1) / US$10 (M3) tetap.
5. **Kebijakan `inconclusive` di skill retro** (perubahan skill factory, butuh persetujuan): push perubahan skill bila verdict `better`, atau `inconclusive` + perubahan memperbaiki kegagalan yang teramati di retro + tidak ada task dengan Δ negatif; revert bila `worse`. Trailer commit `pairbench-report: <hash>`.
6. **Batas kredit Amp untuk build (Phase 5–12).** Usulan: dicatat per fase di progress.md; berhenti dan bertanya bila melewati US$50 kredit di luar langganan.

## Carried over (not fixed this round)

- Spike Phase 5 (R11): apakah plugin, tool built-in akun, server MCP level akun, dan instruksi AGENTS.md level akun bisa dimatikan untuk `amp -x` lewat `--settings-file`/`--mcp-config`; bila tidak, service account (§7).
- Spike Phase 5 (R9): format dan keterlambatan `Cost:` di `amp threads usage` untuk thread yang baru selesai, termasuk penggunaan yang ditanggung langganan.
- Spike Phase 5: apakah stream Amp menampilkan event tool-use `skill` (AC-05.4); apakah `bwrap`/`unshare` bisa dipakai di orb (Could); executor orb `-ox` (Could).
- Thread Amp per attempt menumpuk di daftar thread Thomas: beri label dengan `amp label` setelah attempt (dicek di spike).
- Stack kandidat lama di [stack.md](../stack.md) (PostgreSQL, React SPA) tidak cocok dengan data-dir JSONL dan HTML statis; diputuskan ulang di Phase 5.
