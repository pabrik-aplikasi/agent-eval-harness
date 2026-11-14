# PRD — pairbench (Coding-agent eval harness)

Status: in review (round 1 applied, 14 Nov 2026) — perubahan dicatat di [reviews/plan-review-1.md](reviews/plan-review-1.md)
Delivery: OSS (Apache-2.0, repo public `pabrik-aplikasi/agent-eval-harness`, org dan repo sudah ada: diperiksa dengan `gh repo view` 14 Nov 2026); tanpa hosting wajib
Project `115-agent-eval-harness` · kategori 1 · Phase 2–3
Bahasa: dokumen untuk Thomas dalam bahasa Indonesia; CLI, laporan dan pesan error dalam bahasa Inggris.

## 1. Problem and goals

- **Problem:** factory mengubah skill, prompt dan model tanpa bukti apakah perubahan membantu ([brief.md](brief.md), [research.md](research.md#ringkasan-temuan)). Satu run per sisi tidak bisa membedakan efek dari noise: pass@1 satu run bervariasi 2.2–6.0 pp di SWE-bench Verified ([arXiv 2602.07150](https://arxiv.org/html/2602.07150)). Harness yang ada tidak mempunyai adapter Amp; Harbor mengandalkan Docker atau sandbox cloud, Coder Eval berjalan di `tempdir` tetapi tanpa verdict berpasangan dan tanpa provenance skill Amp ([research.md](research.md#competitors)).
- **Goals:**
  - G1 Menjawab "apakah kandidat lebih baik dari baseline **pada suite task ini**?" dengan verdict `better` / `worse` / `inconclusive` / `insufficient_data`, uji statistik yang false-positive rate-nya terbukti ≤ 5%, dan ukuran efek minimum yang bisa dideteksi dicetak **sebelum** biaya dikeluarkan. ← pain Persona 1 "satu run per sisi tidak bisa membedakan efek dari noise".
  - G2 Menjalankan Amp secara native, plus agent lain lewat adapter command generik, tanpa Docker. ← pain Persona 1 "harness yang ada tidak mendukung Amp".
  - G3 Biaya terlihat sebelum eksperimen dan dibatasi **sebelum setiap run dimulai**. ← pain Persona 1 kuota, Persona 2 biaya tidak seragam.
  - G4 Task dari pekerjaan factory sendiri (kode **dan dokumen**) bisa ditulis dalam ≤ 20 menit, dengan pemeriksa yang dibuktikan valid dan stabil sebelum dipakai. ← tema "benchmark publik tidak mewakili pekerjaan sendiri"; issue Harbor tentang verifier rapuh.
  - G5 Membedakan kegagalan infrastruktur dari kegagalan agent, dan mencatat lingkungan agent yang benar-benar aktif (skill, tool, MCP, mode, versi). ← tema "kegagalan infrastruktur tercatat sebagai kegagalan agent" dan "integritas penilaian bocor".
- **Non-goals:**
  - Leaderboard publik, benchmark tetap, impor dataset publik (SWE-bench, Terminal-Bench). Harbor sudah melayani ini (anti-persona, [personas.md](personas.md#anti-persona-dr-lin-peneliti-lab-yang-membuat-benchmark-publik)).
  - Klaim generalisasi ke "task pada umumnya". Inferensi bersyarat pada suite yang dipilih operator.
  - Sandbox keamanan untuk kode tak tepercaya. Task dan fixture berasal dari operator sendiri; workspace sementara **bukan** batas keamanan. Penyembunyian check adalah perlindungan terhadap kebetulan dan pembacaan langsung, bukan terhadap agent yang sengaja menyerang (§6 Security).
  - SaaS hosted, akun, login, server dashboard live di MVP (Should).
  - LLM-judge di verdict. Optimasi skill otomatis (skill buatan agent sendiri menurunkan pass rate, [SkillsBench 1.1](https://www.skillsbench.ai/blogs/skillsbench-1-1)).

## 2. Success metrics

| Metric | Target | Window | How measured |
| --- | --- | --- | --- |
| M-1 Mengubah keputusan factory | ≥ 2 perubahan skill factory yang di-push atau di-revert dengan tautan laporan pairbench | 60 hari setelah M3 | `git log --grep "pairbench-report:"` di repo factory + footer laporan berisi hash eksperimen |
| M-2 Infra error rate | < 5% dari attempt | 30 attempt nyata pertama setelah M3 | `pairbench stats --status`: `infra_error` / total attempt |
| M-3 Task valid saat dipakai | ≥ 90% task yang lulus `task check` tidak pernah menghasilkan `check_status = error` | 20 task pertama, 60 hari setelah M3 | Tabel `events` (`task_check_passed`) + `runs.check_status` |
| M-4 Waktu menulis task | median ≤ 20 menit dari `task new` sampai `task check` lulus pertama | 10 task pertama yang dibuat dengan `task new` | `events`: selisih `task_created` dan `task_check_passed` per task |
| M-5 Overhead harness (executor lokal) | p95 ≤ 5 s per attempt | 30 attempt nyata pertama | `overhead_ms = total_ms − setup_ms − agent_ms − check_ms − provenance_ms` |
| M-6 Batas biaya | 0 eksperimen dengan `spent_total` > `budget_usd` + biaya satu attempt termahal | Semua eksperimen 60 hari pertama | `spent_total = SUM(cost_usd)` + estimasi untuk attempt `unknown`; `unknown_count` dilaporkan terpisah |
| M-7 Laporan reproducible | 100% `report` diulang pada data dan mesin yang sama menghasilkan byte identik | Setiap rilis | e2e test membandingkan dua output |
| M-8 Kalibrasi statistik | A/A false-positive rate ≤ 5% (batas atas 95% CI simulasi ≤ 6%) | Setiap rilis | Studi simulasi `pairbench selftest stats` di CI (§5 US-08) |

## 3. Personas

Primary: **Thomas, operator AI software factory**. Secondary: Ben, developer yang merawat AGENTS.md dan skill tim kecil. Anti-persona: peneliti lab benchmark publik. Lihat [personas.md](personas.md).

## 4. MVP scope (MoSCoW)

| Priority | Feature | Story IDs |
| --- | --- | --- |
| Must | Format task (instruksi, fixture, setup, check tersembunyi yang memberi skor 0–1, oracle) + `task check --repeat` | US-01 |
| Must | File eksperimen: baseline vs kandidat, ulangan, batas biaya, seed | US-02 |
| Must | Runner: workspace sekali pakai, lingkungan agent terkontrol, timeout, hentikan seluruh turunan proses, check dari salinan milik pairbench, status attempt | US-03 |
| Must | Adapter Amp (preset) + adapter command generik (JSON v1) dengan klasifikasi error | US-04 |
| Must | Provenance lingkungan: skill, tool, MCP, mode, versi CLI, AGENTS.md global | US-05 |
| Must | Estimasi biaya, pilot, admission rule biaya | US-06 |
| Must | Resume tanpa kehilangan atau duplikasi | US-07 |
| Must | Uji berpasangan bersyarat suite + interval + MDE + laporan Markdown dan HTML statis | US-08 |
| Must | Penyimpanan hasil di direktori yang bisa di-commit + export/import JSONL | US-09 |
| Should | Server dashboard live (`pairbench serve`) dengan progres | US-10 |
| Should | Preset Claude Code dan Codex | US-11 |
| Should | Mode CI dan contoh workflow | US-12 |
| Should | `task from-commit` | US-13 |
| Could | Executor orb (`amp -ox`) untuk isolasi dan paralel | — |
| Could | Skor kualitas LLM-judge (blinded A vs B), terpisah dari verdict | — |
| Could | `pairbench usage`: ringkasan biaya thread nyata per project/phase untuk retro | — |
| Won't (this version) | Leaderboard publik, impor benchmark publik, sandbox kode tak tepercaya, server hosted, ratusan run paralel, optimasi skill otomatis | — |

Perubahan dari draft: dashboard live turun dari Must ke Should dan diganti laporan HTML statis (budget waktu, [plan-review-1.md](reviews/plan-review-1.md)). Thomas bisa membaliknya di gate Phase 4.

## 5. User stories

Istilah:

- **slot** = (task, arm, nomor ulangan). **attempt** = satu percobaan menjalankan slot; satu slot bisa punya beberapa attempt bila ada `infra_error` yang di-retry. **arm** = `baseline` atau `candidate`.
- **Status attempt:** `pass`, `fail`, `timeout`, `infra_error`, `cancelled`, `interrupted`, `skipped_budget`.
- **Attempt terhitung** untuk statistik: status `pass`, `fail`, `timeout` (timeout dihitung sebagai skor 0). Slot dihitung memakai attempt terhitung terakhirnya. `infra_error`, `cancelled`, `interrupted`, `skipped_budget` dilaporkan terpisah dan tidak masuk statistik. **Semua** attempt, termasuk yang tidak terhitung, dihitung biayanya.
- **Skor** run = angka 0–1 dari check. Check yang hanya memberi exit code: 0 → skor 1 (`pass`), lainnya → skor 0 (`fail`). Check berskor menulis `{"score": x, "subchecks": {...}}` ke file `$PAIRBENCH_SCORE_FILE`; status `pass` bila skor ≥ `pass_threshold` task (default 1.0).
- **check_status:** `ok` (check selesai dan memberi skor), `error` (check tidak ditemukan, crash, keluaran skor rusak, atau melewati `check_timeout_sec`). `check_status = error` membuat attempt `infra_error`.

### US-01 Menulis dan memvalidasi task

Sebagai Thomas, aku ingin menulis task dari pekerjaan factory (kode maupun dokumen) dan membuktikan pemeriksanya valid dan stabil sebelum dipakai, supaya hasil eval tidak rusak oleh task yang salah.

Format task: direktori berisi `task.yaml` (`id`, `instruction`, `setup` opsional, `check`, `timeout_sec`, `check_timeout_sec` default 300, `pass_threshold` default 1.0, `protected_paths` opsional, `tags`), `fixture/`, `checks/`, `oracle/` (`solution.patch` atau `solve.sh`). Contoh resmi mencakup satu task kode (test unit) dan satu task dokumen (lint aturan PRD berskor).

Acceptance criteria:
- AC-01.1 Given task valid, When `pairbench task check <dir>` dijalankan (default `--repeat 3`), Then exit code 0, output memuat `fixture: score 0.00 < threshold (3/3)` dan `oracle: score 1.00 >= threshold (3/3)`, dan event `task_check_passed` tercatat.
- AC-01.2 Given task yang skor fixture-nya sudah ≥ `pass_threshold`, When `task check` dijalankan, Then exit code 1 dan pesan `check passes on the untouched fixture; the task tests nothing`.
- AC-01.3 Given task yang skor oracle-nya < `pass_threshold` pada salah satu ulangan, When `task check` dijalankan, Then exit code 1, output memuat nomor ulangan yang gagal dan stdout/stderr check-nya, dan pesan `check is not stable` bila ulangan lain lulus.
- AC-01.4 Given `task.yaml` tanpa field `check`, When `task check` dijalankan, Then exit code 1 dan pesan `<path>/task.yaml: missing required field "check"`. Given `timeout_sec: 0`, Then pesan menyebut path, field, nomor baris dan `must be > 0`.
- AC-01.5 Given `pairbench task new my-task` di direktori D, When dijalankan, Then dibuat `D/my-task/` berisi kerangka lengkap, event `task_created` tercatat, dan `task check D/my-task` langsung gagal dengan pesan `fill in task.yaml: instruction, check`.
- AC-01.6 Given check yang melewati `check_timeout_sec`, When `task check` dijalankan, Then exit code 1 dan pesan `check timed out after <n>s`.

States: empty (`task list` tanpa task mencetak `no tasks; run pairbench task new <id>`) / loading (progres `fixture 2/3`, `oracle 1/3`) / error (AC-01.2–01.4, 01.6) / offline (tidak relevan) / success (AC-01.1).

### US-02 Mendefinisikan eksperimen

Sebagai Thomas, aku ingin satu file yang menyebut dua konfigurasi, daftar task, ulangan, batas biaya dan seed, supaya perbandingan bisa diulang persis.

`experiment.yaml`: `name`, `tasks` (path/glob), `repeats` (default 3, minimum 2), `budget_usd` (wajib), `max_parallel` (default 1), `seed` (opsional; bila kosong dibuat dan dicatat), `max_attempts_per_slot` (default 2), `arms.baseline`, `arms.candidate`: `agent` (preset atau `command`), `mode` (untuk Amp: nilai `-m`), `model` (untuk preset lain), `skills_overlay`, `env` (daftar nama variabel yang diteruskan), `timeout_sec`. `budget_usd` tidak masuk hash identitas eksperimen.

Acceptance criteria:
- AC-02.1 Given 2 task, `repeats: 3`, `seed: 7`, When `pairbench plan experiment.yaml` dijalankan, Then output mencantumkan 12 slot dan urutan blok teracak: setiap pasangan (task, ulangan) menjalankan kedua arm berurutan dengan urutan arm acak; dua kali `plan` dengan seed sama menghasilkan urutan identik.
- AC-02.2 Given tanpa `budget_usd`, When `plan` atau `run`, Then exit code 1 dan `budget_usd is required`.
- AC-02.3 Given kedua arm identik (termasuk hash overlay), When `plan`, Then peringatan `arms are identical; this measures noise only (A/A)` dan exit code 0.
- AC-02.4 Given `env` menyebut variabel yang tidak ada, When `plan`, Then exit code 1 dan pesan menyebut nama variabel itu saja.
- AC-02.5 Given field selain `budget_usd` berubah setelah eksperimen dimulai, When `run` dilanjutkan, Then ditolak dengan `experiment file changed (hash mismatch); use --new to start a new experiment`. Given hanya `budget_usd` berubah, Then `run` melanjutkan dengan batas baru dan mencatat perubahan batas.
- AC-02.6 Given `repeats: 1`, When `plan`, Then exit code 1 dan `repeats must be >= 2 for a paired comparison`.
- AC-02.7 Given `pairbench run experiment.yaml` untuk eksperimen yang sudah punya attempt (hash sama), When dijalankan tanpa `--resume` atau `--new`, Then exit code 1 dan `experiment already started; use --resume or --new`.

States: empty (tanpa file: `pairbench init` membuat contoh experiment + 1 task) / loading (tidak relevan) / error (AC-02.2, 02.4–02.7) / offline (tidak relevan) / success (AC-02.1).

### US-03 Menjalankan attempt dengan aman dan jujur

Sebagai Thomas, aku ingin setiap attempt berjalan di workspace bersih dengan lingkungan agent terkontrol, dan dinilai setelah agent benar-benar berhenti, supaya hasil tidak tercemar dan agent yang dievaluasi tidak bisa melakukan aksi ke luar tanpa izin.

Lingkungan agent: environment hanya berisi `PATH`, `LANG`, variabel di `env` arm, kredensial agent yang diperlukan preset (`AMP_API_KEY`), `HOME`/`XDG_*` sementara per attempt, `GIT_CONFIG_GLOBAL=/dev/null`; tanpa `GH_TOKEN`/`GITHUB_TOKEN` kecuali disebut di `env`. Untuk Amp: `--settings-file` buatan pairbench dan `--mcp-config '{}'`.

Acceptance criteria:
- AC-03.1 Given eksperimen 1 task × 2 arm × 2 ulangan dengan agent uji yang menulis jawaban benar, When `run` selesai, Then 4 attempt `pass` di 4 workspace berbeda, dan hash isi setiap workspace sebelum `setup` = hash(fixture + overlay arm) yang dihitung terpisah.
- AC-03.2 Given agent uji yang mencari file check dengan `find /` untuk nama file di `checks/` dan membaca `/proc/<ppid>/cmdline`, When attempt selesai, Then nama file check tidak muncul di output agent uji dan path direktori task tidak muncul di environment, argumen, atau cwd agent.
- AC-03.3 Given agent uji yang mengubah file di `protected_paths` (misalnya `package.json`, `conftest.py`), When check dijalankan, Then file itu dipulihkan dari fixture sebelum check, dan attempt menyimpan `integrity_warning: protected path modified: <path>`.
- AC-03.4 Given agent uji yang menulis ke direktori data pairbench atau direktori task, When attempt selesai, Then hash direktori task dan data sebelum/sesudah dibandingkan dan perubahan menghasilkan `integrity_warning`; attempt tetap dinilai.
- AC-03.5 Given agent uji yang melewati `timeout_sec` dan memulai turunan dengan `setsid` + double-fork, When timeout tercapai, Then status `timeout`, tidak ada proses turunan yang masih hidup sebelum check dimulai (diverifikasi dari daftar PID turunan yang dicatat), dan check tetap dijalankan sebagai informasi.
- AC-03.6 Given `setup` gagal atau perintah agent tidak ditemukan, When attempt dijalankan, Then status `infra_error` dengan penyebab.
- AC-03.7 Given agent keluar non-zero tanpa klasifikasi error infrastruktur dan check memberi skor 1, When dicatat, Then status `pass` dan exit code agent disimpan terpisah.
- AC-03.8 Given Ctrl+C, When pairbench berhenti, Then attempt aktif `cancelled`, turunan proses mati, workspace dihapus kecuali `--keep-workspaces`.
- AC-03.9 Given attempt selesai, When `pairbench show <attempt-id>` dijalankan, Then tersedia diff workspace terhadap fixture+overlay, stdout/stderr agent dengan nilai semua variabel yang diteruskan (termasuk `AMP_API_KEY`) diganti `***`, output check, skor dan subchecks, durasi per fase, exit code, warnings.
- AC-03.10 Given agent uji yang mencetak `GH_TOKEN` dan `HOME` dari environment-nya, When attempt selesai, Then `GH_TOKEN` kosong dan `HOME` adalah direktori sementara attempt.

States: empty (lihat US-02) / loading (progres `slot 3/12 · t1 · candidate · rep 2 · 01:12 · $0.84 spent`) / error (AC-03.6) / offline (provider tidak bisa dihubungi: `infra_error` dengan `error_kind: infra`, lihat US-04) / success (AC-03.1).

### US-04 Adapter agent: Amp preset dan command generik

Sebagai Thomas, aku ingin menjalankan Amp tanpa glue code, dan agent lain lewat template command, supaya pairbench tidak terkunci pada satu vendor.

Kontrak adapter generik v1 (dikunci di M0): command template dengan placeholder `{instruction_file}`, `{workspace}`, `{model}`; baris terakhir stdout boleh berisi `{"pairbench_adapter": 1, "error_kind": "infra"|"agent"|null, "usage": {"input": n, "cache_creation": n, "cache_read": n, "output": n} | null, "cost_usd": x | null, "session_id": s | null}`.

Acceptance criteria:
- AC-04.1 Given arm `agent: amp, mode: <m>` dan `AMP_API_KEY` tersedia, When attempt dijalankan, Then pairbench memanggil `amp -x --stream-json -m <m>` di workspace, menyimpan `session_id` dari pesan init, menjumlah `usage` dari pesan `assistant` saja (input, cache creation, cache read, output terpisah), dan setelah agent selesai mengambil biaya dari `amp threads usage <session_id>` (baris `Cost: $x`) dengan `cost_source: reported`.
- AC-04.2 Given output `amp threads usage` tidak memuat baris `Cost:` yang bisa di-parse, When biaya dicatat, Then `cost_source: unknown` dan peringatan `amp cost unavailable`.
- AC-04.3 Given pesan `result` Amp dengan `is_error: true` atau `subtype` error, atau pesan error yang cocok dengan pola error provider (rate limit, quota, network) di daftar terkontrol, When attempt dicatat, Then status `infra_error` dengan `error_kind: infra`, terlepas dari skor check.
- AC-04.4 Given attempt `infra_error`, When slot itu belum mencapai `max_attempts_per_slot`, Then slot dijadwalkan ulang di akhir urutan; semua attempt tercatat dan biayanya dihitung.
- AC-04.5 Given arm Amp tanpa `AMP_API_KEY`, When `plan`, Then exit code 1 dan `AMP_API_KEY is not set; create an access token in Amp settings`.
- AC-04.6 Given adapter generik yang mencetak JSON v1 valid, When attempt selesai, Then usage, biaya (`cost_source: reported`) dan `error_kind` tercatat sesuai isi JSON.
- AC-04.7 Given adapter generik yang mencetak JSON rusak, When attempt selesai, Then status ditentukan check, biaya `unknown`, dan `adapter output ignored: <reason>` tersimpan.
- AC-04.8 Given stream Amp memuat tipe pesan tidak dikenal, When diproses, Then pesan diabaikan dan jumlahnya tercatat di `adapter_warnings`. Contract test memakai rekaman output nyata CLI Amp yang versinya dicatat.

States: error (AC-04.2, 04.5, 04.7) / offline (AC-04.3) / success (AC-04.1, 04.6).

### US-05 Provenance lingkungan agent

Sebagai Thomas, aku ingin tahu skill, tool, server MCP, mode dan versi yang benar-benar aktif di setiap attempt, supaya perbandingan "skill v1 vs v2" tidak diam-diam membandingkan hal lain, dan agent yang dievaluasi tidak memegang tool yang bisa mengirim sesuatu ke orang lain.

Fakta yang sudah diperiksa (14 Nov 2026, [research.md](research.md#bukti-tambahan-yang-memengaruhi-desain)): `amp skill list --json` memberi `name`, `source`, `baseDir`; overlay di `.agents/skills/` (`source: workspace-agents`) mengalahkan skill `global-user` bernama sama; HOME sementara tidak menyembunyikan skill `global-user` dan plugin.

Acceptance criteria:
- AC-05.1 Given arm dengan overlay berisi skill `prd-writer`, When attempt Amp dijalankan, Then sebelum agent mulai pairbench menyimpan output `amp skill list --json` dari workspace, dan `show` menampilkan `prd-writer` dengan `source: workspace-agents`, `baseDir` di dalam workspace, dan SHA-256 isi direktori skill.
- AC-05.2 Given skill overlay bernama sama yang ter-resolve ke `baseDir` di luar workspace, When `plan` dijalankan, Then exit code 1 dan `skill prd-writer resolves to <baseDir>, not to the overlay`. Bila terdeteksi saat attempt, attempt `infra_error`.
- AC-05.3 Given attempt Amp, When pesan init diterima, Then `tools`, `mcp_servers`, `agent_mode` (bila ada) dan versi `amp --version` disimpan; bila `mcp_servers` tidak kosong atau `agent_mode` berbeda dari `mode` arm, attempt dihentikan sebelum turn pertama selesai dan dicatat `infra_error` dengan alasan `unexpected MCP servers: <names>` atau `mode mismatch`.
- AC-05.4 Given arm tanpa overlay, When attempt dijalankan, Then provenance mencatat `no overlay` dan daftar lengkap skill non-official yang aktif (`global-user`, `plugin`), sehingga laporan bisa menyebut skill akun yang ikut aktif di kedua arm.
- AC-05.5 Given adapter generik, When attempt dijalankan, Then provenance berisi hash overlay dan catatan `skill resolution not verified for generic agents`.

States: empty (AC-05.4) / error (AC-05.2, 05.3) / success (AC-05.1).

### US-06 Biaya terlihat dan dibatasi

Sebagai Thomas, aku ingin estimasi biaya sebelum mulai dan jaminan tidak ada attempt baru yang dimulai bila batas akan terlewati, supaya eval tidak menghabiskan kuota tanpa izin.

Admission rule: attempt baru dimulai hanya bila `spent + reserved_in_flight + estimate_next ≤ budget_usd`, dengan `estimate_next` = estimasi per attempt (di bawah). `spent` memakai biaya tercatat; attempt `unknown` dihitung dengan estimasinya. Kunci estimasi: (task, agent, mode/model, hash overlay), fallback (task, agent), fallback `--assume-cost-per-run`.

Acceptance criteria:
- AC-06.1 Given eksperimen tanpa riwayat dan tanpa `--assume-cost-per-run`, When `plan`, Then estimasi `unknown (no history)` dan saran `run --pilot` (1 ulangan per task per arm, dicatat sebagai eksperimen terpisah berlabel `pilot`, riwayatnya dipakai untuk estimasi).
- AC-06.2 Given riwayat untuk sebagian kunci, When `plan`, Then output memuat estimasi total, rentang dari resample biaya per attempt historis (p10–p90 dari jumlah, seed tetap), dan `estimated from history for 4/6 keys; 2 keys use --assume-cost-per-run`.
- AC-06.3 Given `budget_usd: 1.00`, estimasi per attempt US$0.30, setiap attempt uji melaporkan US$0.30, 6 slot, `max_parallel: 1`, When `run`, Then 3 attempt dijalankan (sebelum ke-4: 0.90 + 0.30 > 1.00), 3 slot `skipped_budget`, ringkasan `budget reached: spent $0.90 of $1.00`.
- AC-06.4 Given `max_parallel: 3` dengan nilai yang sama, When `run`, Then jumlah `spent + reserved` tidak pernah melebihi US$1.00 pada saat attempt mana pun dimulai (diverifikasi dari log admission).
- AC-06.5 Given tanpa riwayat, tanpa `--assume-cost-per-run`, dan bukan `--pilot`, When `run`, Then exit code 1 dan `cannot enforce budget without a cost estimate; run --pilot or pass --assume-cost-per-run`.
- AC-06.6 Given `--dry-run`, When `run`, Then tidak ada agent dipanggil dan output sama dengan `plan`.
- AC-06.7 Given `plan`, When biaya dan desain diketahui, Then output memuat ukuran efek minimum yang bisa dideteksi (MDE) dengan power 80% untuk desain itu (US-08).

States: empty (AC-06.1) / loading (biaya berjalan di progres) / error (AC-06.5) / success (AC-06.2, 06.3).

### US-07 Resume tanpa kehilangan atau duplikasi

Sebagai Thomas, aku ingin eksperimen yang terhenti (orb pause, update CLI, proses mati) bisa dilanjutkan, supaya attempt yang sudah dibayar tidak hilang atau terulang.

Acceptance criteria:
- AC-07.1 Given 6 slot, When pairbench dibunuh dengan SIGKILL setelah slot ke-3 tercatat dan saat slot ke-4 berjalan, Then `pairbench run --resume <exp>` pertama-tama menghentikan turunan proses slot ke-4 yang masih hidup (dari PGID dan daftar PID yang dicatat durable sebelum agent dimulai), mencatat attempt itu `interrupted` dengan biayanya bila `session_id` tersedia, lalu menjalankan ulang slot ke-4 sampai ke-6; slot 1–3 tidak diulang.
- AC-07.2 Given eksperimen selesai, When `--resume`, Then tidak ada attempt baru dan pesan `nothing to resume`.
- AC-07.3 Given proses dibunuh setelah agent selesai tetapi sebelum skor check tercatat, When resume, Then attempt itu `interrupted` dan slot dijalankan ulang; tidak ada attempt dengan skor setengah tercatat.
- AC-07.4 Given resume dengan `--budget 2.00`, When dijalankan, Then batas baru dipakai tanpa mengubah identitas eksperimen (AC-02.5).

States: empty (AC-07.2) / loading (progres) / error (eksperimen tidak ditemukan: pesan dengan daftar ID) / success (AC-07.1).

### US-08 Perbandingan berpasangan, verdict, laporan

Sebagai Thomas, aku ingin satu laporan yang menyatakan kandidat lebih baik, lebih buruk, belum cukup bukti, atau datanya kurang, dengan metode yang terbukti tidak memberi false positive lebih dari 5%, supaya aku bisa mengutipnya di retro.

Metode (kontrak M0, inferensi **bersyarat pada suite task ini**):

- Per task t dan arm a: skor rata-rata s̄_a,t dari slot terhitung. Selisih d_t = s̄_cand,t − s̄_base,t. Estimasi Δ = rata-rata d_t atas task berpasangan.
- **Uji:** permutation test berstrata: label arm diacak di dalam setiap task, statistik Δ, p dua sisi. Exact bila jumlah permutasi ≤ 1 000 000, selain itu Monte Carlo 100 000 dengan seed tetap dan p = (k+1)/(N+1). α = 0.05.
- **Interval:** bootstrap berstrata 95% (resample slot di dalam setiap sel task-arm, task tetap), percentile, 10 000 resample, seed tetap. Dilaporkan sebagai deskripsi, bukan dasar verdict.
- **Verdict:** `insufficient_data` bila ada task berpasangan dengan < 2 slot terhitung di salah satu arm, atau total task berpasangan < 1; selain itu `better` bila p < 0.05 dan Δ > 0, `worse` bila p < 0.05 dan Δ < 0, `inconclusive` selainnya. Laporan dari eksperimen yang belum selesai atau terpotong budget diberi label `interim`.
- **MDE:** simulasi power (2 000 simulasi, seed tetap) dengan pass rate baseline teramati per task (atau 0.5 bila belum ada data), α 0.05, power 80%; dicetak di `plan` dan laporan.
- **Kalibrasi M-8:** `pairbench selftest stats` mensimulasikan A/A pada grid 3–10 task × 2–5 ulangan × pass rate 0.1–0.9 dan gagal bila false-positive rate > 5% (batas atas CI simulasi > 6%).

Acceptance criteria (nilai harapan dihitung oleh script referensi terpisah di `e2e/fixtures/` dengan enumerasi exact, bukan oleh pairbench):
- AC-08.1 Given 3 task × 4 ulangan, kandidat 12/12 dan baseline 0/12, When `report`, Then Δ = +1.00, p exact sama dengan script referensi (70³ = 343 000 permutasi; p = 2 × (1/70)³ ≈ 5.8e-6), verdict `better`.
- AC-08.2 Given data A/A dengan pola identik, When `report`, Then Δ = 0.00, p = 1.00, verdict `inconclusive`.
- AC-08.3 Given fixture campuran tidak degeneratif (3 task × 3 ulangan, skor bervariasi per sel), When `report`, Then Δ, p exact, dan batas interval sama dengan script referensi dalam toleransi ±0.005 untuk interval dan exact untuk p.
- AC-08.4 Given satu task dengan hanya 1 slot terhitung di kandidat, When `report`, Then verdict `insufficient_data` dan pesan menyebut task itu.
- AC-08.5 Given attempt `infra_error`, `cancelled`, `interrupted`, `skipped_budget`, When `report`, Then tabel status mencantumkan jumlah per status per arm dan peringatan `N attempts excluded (<status>)` untuk setiap status yang ada.
- AC-08.6 Given `report` dua kali pada data sama di mesin sama, Then byte identik (tanpa waktu jam dinding di isi; angka diformat tetap 4 desimal; RNG ditentukan di kontrak).
- AC-08.7 Given `report --format md`, Then output memuat tabel per task, Δ, p, interval, MDE, verdict, seed, versi pairbench, hash eksperimen, hash overlay setiap arm, daftar skill non-official yang aktif, dan footer `pairbench-report: <experiment-hash>`.
- AC-08.8 Given `report --format html --out <dir>`, Then `<dir>/index.html` (daftar eksperimen), halaman eksperimen dan halaman per attempt (diff, log tersamar, output check, provenance) terbentuk sebagai file statis tanpa server; semua output agent di-escape; axe-core tidak menemukan pelanggaran WCAG 2.2 AA serious/critical; semua link bisa diakses keyboard.
- AC-08.9 Given biaya per pass dengan 0 pass di satu arm, Then ditulis `n/a`; bila ada attempt biaya `unknown`, jumlahnya dicetak di samping.
- AC-08.10 Given `pairbench selftest stats`, When dijalankan di CI, Then lulus (M-8).

States: empty (`no countable runs yet`) / loading (< 2 s, tidak ada) / error (eksperimen tidak ditemukan) / success (AC-08.1, 08.7, 08.8).

### US-09 Penyimpanan hasil yang bertahan

Sebagai Thomas, aku ingin hasil eval bertahan setelah orb atau thread selesai, supaya riwayat biaya dan metrik 60 hari tidak hilang.

Acceptance criteria:
- AC-09.1 Given `PAIRBENCH_HOME` atau `--data-dir`, When eksperimen dijalankan, Then semua data (database, artefak, event) ada di direktori itu; default `./.pairbench/` di direktori kerja.
- AC-09.2 Given eksperimen selesai, When `pairbench export <exp> --out f.jsonl`, lalu `pairbench import f.jsonl` di data-dir kosong, Then `report` di kedua data-dir menghasilkan byte identik.
- AC-09.3 Given artefak attempt > 1 MB per log, When disimpan, Then log dipotong dengan penanda `[truncated N bytes]` dan ukuran asli dicatat.
- AC-09.4 Given `pairbench prune --older-than 30d`, Then artefak attempt lebih tua dari 30 hari dihapus, ringkasan hasil dan biaya tetap, dan jumlah yang dihapus dicetak.

States: empty (data-dir baru dibuat otomatis) / error (data-dir tidak bisa ditulis: path dan errno) / success (AC-09.2).

### US-10 Dashboard live (Should)

Sebagai Thomas, aku ingin membuka hasil yang sedang berjalan di browser lewat portal orb.

- AC-10.1 Given `pairbench serve`, Then server hanya mendengarkan di `127.0.0.1` kecuali `--host`; dengan `--host` non-loopback muncul `dashboard has no authentication`.
- AC-10.2 Given eksperimen berjalan, When halaman eksperimen terbuka, Then progres bertambah ≤ 5 s setelah attempt tercatat, dan verdict berlabel `interim`.
- AC-10.3 README menjelaskan cara menjalankan `run` dan `serve` sebagai `amp orb service` dengan `--portal` di orb.

### US-11 Preset Claude Code dan Codex (Should)

- AC-11.1 Given `agent: claude-code`, Then pairbench memanggil `claude -p --output-format json` dan mencatat `total_cost_usd` sebagai `cost_source: reported` ([docs](https://code.claude.com/docs/en/headless?x=1)).
- AC-11.2 Given `agent: codex`, Then pairbench memanggil `codex exec --json --sandbox workspace-write`, menjumlah `turn.completed.usage`, dan biaya `computed` dari file tarif yang disediakan operator (`rates.yaml`, versi dicatat) atau `unknown` ([docs](https://developers.openai.com/codex/noninteractive)).
- AC-11.3 Given binary tidak terpasang, When `plan`, Then exit code 1 dengan nama binary.

### US-12 Mode CI (Should)

- AC-12.1 Given `run --ci`, Then exit code 2 untuk verdict `worse`, 0 untuk `better`/`inconclusive`, 3 untuk `insufficient_data` atau eksperimen terpotong budget, 1 untuk error konfigurasi.
- AC-12.2 Contoh workflow di `examples/ci/` hanya berjalan pada `workflow_dispatch` dan push ke branch utama (tidak pada `pull_request` dari fork), membaca `AMP_API_KEY` dari secret, dan mengunggah laporan Markdown sebagai artifact. Dibuktikan dengan `actionlint` dan satu run manual yang disetujui Thomas.

### US-13 Task dari commit (Should)

- AC-13.1 Given commit C dan `--test-glob "<glob>"`, When `task from-commit <repo> <C> --check "<cmd>"`, Then fixture = parent C, `checks/` = file C yang cocok glob, oracle = diff non-test C, `instruction` = judul commit saja (body dan trailer `Co-authored-by` tidak disalin), siap diedit.
- AC-13.2 Given hasil from-commit yang lulus pada fixture, Then `task check` gagal sesuai AC-01.2.

## 6. Non-functional requirements

| Area | Requirement |
| --- | --- |
| Performance | `plan`/`report` < 2 s untuk 1 000 attempt di orb 2 vCPU/3 GB (permutation test exact dibatasi 1 000 000 permutasi). Overhead harness lokal p95 ≤ 5 s (M-5). Laporan HTML: halaman eksperimen ≤ 500 KB untuk 1 000 attempt, tanpa JS wajib; LCP < 1.5 s dibuka dari disk (Lighthouse desktop). |
| Reliability | Setiap attempt ditulis durable (PGID/PID sebelum agent mulai; hasil sebelum slot berikutnya). Resume tanpa kehilangan/duplikasi (US-07). Turunan proses dihentikan sebelum check: process group + subreaper (`PR_SET_CHILD_SUBREAPER`) + sapuan PID turunan; diuji dengan `setsid` double-fork (AC-03.5). `infra_error` dipisah dari `fail` (AC-04.3). |
| Security | **Model ancaman:** operator tepercaya, agent yang dievaluasi semi-tepercaya (bisa salah atau mencari jalan pintas, tidak diasumsikan jahat). Jaminan yang diberikan: check disimpan di direktori data pairbench di luar workspace dan leluhurnya, tidak muncul di instruksi, environment, argumen atau cwd agent (AC-03.2); check dijalankan dari salinan milik pairbench setelah turunan agent mati; `protected_paths` dipulihkan; perubahan direktori task/data dideteksi (AC-03.3–03.4). **Tidak dijamin:** agent yang sengaja membaca seluruh filesystem pengguna yang sama. Dinyatakan di README dan output `plan`. Lingkungan agent di-scrub (US-03), Amp dengan settings minimal dan `--mcp-config '{}'`, attempt ditolak bila server MCP muncul (AC-05.3); disarankan memakai API key service account workspace tanpa koneksi MCP. Nilai rahasia disamarkan di log; rahasia yang dibaca agent dari file tidak disamarkan (dinyatakan). Laporan HTML meng-escape semua output agent (OWASP A03). Dependency audit di CI. |
| Accessibility | Laporan HTML (dan dashboard Should) WCAG 2.2 AA (AC-08.8); output CLI tidak bergantung warna saja. |
| Privacy | Inventaris di bawah. Tidak ada telemetry. Data tinggal di mesin dan kendali pengguna; pairbench tidak mengirim data ke server mana pun selain yang dipanggil agent yang dipilih pengguna. Karena itu kewajiban pengendali data UU PDP tidak muncul dari pairbench sendiri; data pribadi yang mungkin ada di repo pengguna tetap tanggung jawab pengguna. |
| Localization | English. Waktu ISO 8601 UTC (hanya di metadata, tidak di isi laporan reproducible). Biaya USD: 4 desimal di data dan laporan, 2 desimal di pesan progres CLI. |
| Portability | Linux x64 wajib (orb); macOS arm64 Should. Tanpa Docker. Operasi panjang di orb dijalankan sebagai `amp orb service` (README). |

Inventaris data:

| Data | Kenapa | Disimpan di | Retensi |
| --- | --- | --- | --- |
| Instruksi task, fixture, check, oracle | Mendefinisikan task | Repo pengguna; salinan check di data-dir | Dikelola pengguna |
| Diff, stdout/stderr agent dan check (nilai rahasia tersamar) | Menjelaskan hasil | Data-dir | Sampai `prune` |
| Session/thread ID, usage token, biaya | Biaya dan penelusuran | Data-dir | Sampai dihapus; tidak terkena `prune` |
| Nama variabel env (tanpa nilai), provenance lingkungan | Reproducibility | Data-dir | Sama |
| Event `task_created`, `task_check_passed` | Metrik M-3, M-4 | Data-dir | Sama |

## 7. External dependencies

| Dependency | Purpose | Access needed | Cost | Fallback |
| --- | --- | --- | --- | --- |
| Amp CLI (`amp -x --stream-json`, `amp skill list --json`, `amp threads usage`, `--settings-file`, `--mcp-config`) | Agent utama, provenance, biaya | Access token `AMP_API_KEY` ([docs](https://ampcode.com/docs/cli/execute-mode)); disarankan service account | Kredit Amp per attempt; thread ini tercatat "Covered by your Amp Megawatt subscription" (`amp threads usage`, 14 Nov 2026). **Setiap eksperimen nyata butuh batas biaya yang disetujui Thomas** | Adapter generik |
| GitHub org `pabrik-aplikasi` + repo public | Kode dan rilis | Sudah ada (gh) | Gratis | — |
| Claude Code / Codex CLI (Should) | Pembanding | Akun masing-masing | Per token | Tidak dipakai |
| Git, toolchain stack | Fixture, diff, build | Lokal | Gratis | — |
| GitHub Actions (Should) | Mode CI | Repo public | Gratis untuk repo public (ASUMSI, dicek di Phase 5) | Manual |
| Hosting | — | — | US$0 | — |

## 8. Risks and assumptions

| Risk or assumption | Impact | Likelihood | Mitigation or validation |
| --- | --- | --- | --- |
| R1 Desain default hanya mendeteksi efek besar (perkiraan reviewer: 5 task × 3 ulangan → efek ~50 pp) | High | High | MDE dicetak di `plan` sebelum biaya (AC-06.7); skor 0–1 menurunkan varians; panduan memilih task dengan baseline 20–80%; eksperimen pertama adalah kalibrasi, bukan keputusan (M3) |
| R2 Biaya eval besar | High | Medium | Admission rule + pilot (US-06); batas awal US$1 (M1) dan US$10 (M3) kecuali Thomas setuju lebih |
| R3 Format output CLI agent berubah | Medium | High | Parser toleran (AC-04.8); contract test rekaman; versi CLI per attempt |
| R4 Overlay skill kalah oleh skill lokal | High | Low | **Diperiksa:** overlay workspace mengalahkan `global-user`; tetap diverifikasi per attempt (AC-05.1–05.2) |
| R5 Baseline "tanpa skill" untuk skill yang ter-deploy di akun | Medium | Medium | Skill factory belum ter-deploy di akun (diperiksa di orb ini); bila nanti ter-deploy: overlay versi lama vs baru, atau service account tanpa skill tersebut |
| R6 Agent membaca check atau mengubah grader | High | Medium | Jaminan terbatas yang dinyatakan + `protected_paths` + deteksi integritas (§6); isolasi kuat lewat executor orb = Could, di-spike Phase 5 |
| R7 Test lulus ≠ hasil bagus | Medium | High | Skor berbutir (subchecks); diff dan log di laporan; LLM-judge = Could |
| R8 Turunan proses lolos | Medium | Medium | Subreaper + sapuan PID (§6); service orb yang dimulai agent tercatat bila terdeteksi (`amp orb service list` sebelum/sesudah); diuji AC-03.5 |
| R9 Biaya Amp hanya tersedia sebagai teks | Medium | High | Parse `Cost: $x` dengan contract test; `unknown` + estimasi bila gagal (AC-04.2, US-06) |
| R10 Membangun ulang yang bisa diadopsi (Coder Eval, `tempdir`, Apache-2.0) | Medium | Medium | Keputusan build vs adopt di gate Phase 4; bila build, Phase 5 tetap men-spike plugin Amp untuk Coder Eval sebagai pembanding di ADR |
| R11 Agent yang dievaluasi memakai kredensial/MCP operator untuk aksi ke luar | High | Medium | Environment di-scrub, settings minimal, `--mcp-config '{}'`, tolak attempt bila MCP muncul (AC-03.10, 05.3); service account disarankan. **Spike Phase 5:** apakah server MCP level akun muncul di `amp -x` dengan settings minimal |
| R12 Task dokumen tidak bisa dicek deterministik | Medium | Medium | Check berskor (lint aturan, fixture cacat yang ditanam untuk skill review); contoh resmi task dokumen di M0 |
| R13 Statistik kecil disalahgunakan (mengintip hasil sementara) | Medium | Medium | Label `interim`; mode CI hanya pada eksperimen selesai (AC-12.1) |
| A1 Agent bisa dijalankan non-interaktif dengan output mesin | — | — | Terverifikasi dari docs |
| A2 Operator menulis task tepercaya | — | — | Dinyatakan di README dan `plan` |

## 9. Milestones

| Milestone | Contents | Exit criteria |
| --- | --- | --- |
| M0 Foundations and contracts | JSON Schema `task.yaml`/`experiment.yaml`, schema data-dir dan event, adapter JSON v1, spesifikasi statistik + script referensi, `selftest stats`, contoh 1 task kode + 1 task dokumen, CI | Kontrak di `contracts/`; script referensi menghasilkan nilai AC-08; `selftest stats` lulus (M-8); CI hijau |
| M1 Core loop | US-01, US-02, US-03, US-04, US-07, US-08 (Markdown) | e2e AC terkait hijau dengan agent uji deterministik; 1 attempt Amp nyata ≤ US$1 lulus (izin Thomas) |
| M2 Supporting features | US-05, US-06, US-08 (HTML), US-09 | e2e hijau; axe-core bersih |
| M3 Polish and launch | README (model ancaman, cara orb service), Should bila waktu cukup, **eksperimen kalibrasi** ≤ US$10: A/A pada 3 task murah, lalu kontrol positif (skill yang memuat aturan factory yang bisa dicek regex vs tanpa skill) | Semua Must hijau; laporan kalibrasi di `docs/`; rilis GitHub v0.1.0 |

## 10. Budget

- Time (hari kerja agent, perkiraan kasar): US-01 1.5, US-02 1, US-03 3, US-04 2, US-05 1, US-06 1.5, US-07 1.5, US-08 2.5 (termasuk HTML statis dan selftest), US-09 1 → Must 15. M0 kontrak + spike Phase 5 sudah terpisah di Phase 5 (stack). Polish 1.5. Total MVP ≈ 16.5 hari agent ≈ 3.5 minggu kalender; Should (US-10–13, ≈ 5 hari) hanya bila waktu tersisa.
- Money: hosting US$0/bulan. Kredit Amp untuk build: UNKNOWN sampai fase build; thread ini mencatat US$0.04 per `amp threads usage` (tidak termasuk subagent bila ditagih terpisah). Eval nyata: US$1 (M1) + US$10 (M3), masing-masing dengan persetujuan Thomas.
