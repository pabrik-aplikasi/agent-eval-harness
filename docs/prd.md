# PRD — pairbench (Coding-agent eval harness)

Status: ready for readiness check (plan review round 2 applied, 14 Nov 2026) — keputusan di [reviews/plan-review-1.md](reviews/plan-review-1.md) dan [reviews/plan-review-2.md](reviews/plan-review-2.md)
Delivery: OSS (Apache-2.0, repo public `pabrik-aplikasi/agent-eval-harness`; org dan repo sudah ada, diperiksa dengan `gh repo view` 14 Nov 2026); tanpa hosting wajib
Project `115-agent-eval-harness` · kategori 1
Bahasa: dokumen untuk Thomas dalam bahasa Indonesia; CLI, laporan dan pesan error dalam bahasa Inggris.

## 1. Problem and goals

- **Problem:** factory mengubah skill, prompt dan model tanpa bukti apakah perubahan membantu ([brief.md](brief.md), [research.md](research.md#ringkasan-temuan)). Satu run per sisi tidak bisa membedakan efek dari noise: pass@1 satu run bervariasi 2.2–6.0 pp di SWE-bench Verified ([arXiv 2602.07150](https://arxiv.org/html/2602.07150)). Harness yang ada tidak mempunyai adapter Amp; Harbor mengandalkan Docker atau sandbox cloud; Coder Eval berjalan di `tempdir` dan punya perbandingan berpasangan dengan task sebagai unit, tetapi tanpa Amp, tanpa provenance skill Amp, dan dengan power sangat rendah pada suite 3–5 task ([research.md](research.md#competitors)).
- **Goals:**
  - G1 Menjawab "apakah kandidat lebih baik dari baseline **pada suite task ini**?" dengan verdict `better` / `worse` / `inconclusive` / `insufficient_data`, uji exact yang false-positive rate-nya ≤ 5% di bawah hipotesis nol yang dinyatakan (US-08), dan ukuran efek minimum yang bisa dideteksi dicetak **sebelum** biaya dikeluarkan. ← pain Persona 1 "satu run per sisi tidak bisa membedakan efek dari noise".
  - G2 Menjalankan Amp secara native, plus agent lain lewat adapter command generik, tanpa Docker. ← pain Persona 1 "harness yang ada tidak mendukung Amp".
  - G3 Biaya terlihat sebelum eksperimen dan dibatasi **sebelum setiap attempt dimulai**. ← pain Persona 1 kuota, Persona 2 biaya tidak seragam.
  - G4 Task dari pekerjaan factory sendiri (kode **dan dokumen**) bisa ditulis dalam ≤ 20 menit, dengan pemeriksa yang dibuktikan valid dan stabil sebelum dipakai. ← tema "benchmark publik tidak mewakili pekerjaan sendiri"; issue Harbor tentang verifier rapuh.
  - G5 Membedakan kegagalan infrastruktur dari kegagalan agent, mencatat lingkungan agent yang benar-benar aktif, dan mencegah agent yang dievaluasi memegang tool yang bisa bertindak ke luar. ← tema "kegagalan infrastruktur tercatat sebagai kegagalan agent", "integritas penilaian bocor"; guardrail factory "always ask before sending anything to another person".
- **Non-goals:**
  - Leaderboard publik, benchmark tetap, impor dataset publik. Harbor sudah melayani ini (anti-persona, [personas.md](personas.md#anti-persona-dr-lin-peneliti-lab-yang-membuat-benchmark-publik)).
  - Klaim generalisasi ke "task pada umumnya". Inferensi bersyarat pada suite yang dipilih operator.
  - Sandbox keamanan. Task dan fixture berasal dari operator; workspace sementara **bukan** batas keamanan; penyembunyian check menahan pembacaan tidak sengaja dan langsung, bukan agent yang sengaja menyerang (§6 Security).
  - SaaS hosted, akun, login, dashboard live di MVP (Should).
  - LLM-judge di verdict. Optimasi skill otomatis (skill buatan agent sendiri menurunkan pass rate, [SkillsBench 1.1](https://www.skillsbench.ai/blogs/skillsbench-1-1)).

## 2. Success metrics

| Metric | Target | Window | How measured |
| --- | --- | --- | --- |
| M-1 Mengubah keputusan factory | ≥ 2 perubahan skill factory yang di-push atau di-revert dengan trailer `pairbench-report: <hash>` | 60 hari setelah M3; butuh ≥ 2 eksperimen keputusan dengan budget yang disetujui Thomas | `git log --grep "pairbench-report:"` di repo factory; hash cocok dengan footer laporan |
| M-2 Infra error rate | < 5% dari attempt | 30 attempt nyata pertama setelah M3 | Tabel status `pairbench report` (jumlah `infra_error` / total attempt) |
| M-3 Task valid saat dipakai | ≥ 90% task yang lulus `task check` tidak pernah menghasilkan `check_status = error` | 20 task pertama, atau semua task pada hari ke-60 setelah M3, mana yang lebih dulu | Event `task_check_passed` + `check_status` per attempt di data-dir |
| M-4 Waktu menulis task | median ≤ 20 menit dari `task new` sampai `task check` lulus pertama | 10 task pertama yang dibuat dengan `task new` | Event `task_created` dan `task_check_passed` per task |
| M-5 Overhead harness | p95 ≤ 5 s per attempt | 30 attempt nyata pertama | `overhead_ms = total_ms − setup_ms − agent_ms − check_ms − provenance_ms − cost_poll_ms` |
| M-6 Batas biaya | 0 eksperimen dengan `spent_total` > `budget_usd` + `max_parallel` × biaya attempt termahal | Semua eksperimen 60 hari pertama | `spent_total` = jumlah biaya tercatat + estimasi untuk attempt `unknown`; `unknown_count` dilaporkan terpisah |
| M-7 Laporan reproducible | 100% `report` diulang pada data dan mesin sama menghasilkan byte identik | Setiap rilis | e2e test |
| M-8 Kalibrasi statistik | Batas atas Clopper-Pearson 95% dari FPR A/A ≤ 6% di setiap sel grid | Setiap rilis | `pairbench selftest stats` di CI (US-08) |

## 3. Personas

Primary: **Thomas, operator AI software factory**. Secondary: Ben, developer yang merawat AGENTS.md dan skill tim kecil. Anti-persona: peneliti lab benchmark publik. Lihat [personas.md](personas.md).

## 4. MVP scope (MoSCoW)

| Priority | Feature | Story IDs |
| --- | --- | --- |
| Must | Format task (check tersembunyi dengan skor 0–1, oracle) + `task check --repeat` + suite contoh 3 task factory | US-01 |
| Must | File eksperimen: dua arm, overlay skill dari direktori atau `git:<repo>@<rev>`, ulangan, batas biaya, seed; identitas = hash isi | US-02 |
| Must | Runner: workspace di luar repo mana pun, environment di-scrub untuk agent dan check, timeout, hentikan seluruh turunan, check dari salinan milik pairbench | US-03 |
| Must | Adapter Amp + adapter command generik v1 (juga menjadi agent uji deterministik untuk e2e), klasifikasi error | US-04 |
| Must | Provenance dan gerbang lingkungan: skill tersedia dan **dimuat**, allowlist tool, MCP, mode, versi, AGENTS.md | US-05 |
| Must | Estimasi biaya, pilot berbatas, admission rule | US-06 |
| Must | Resume tanpa kehilangan atau duplikasi | US-07 |
| Must | Uji berpasangan exact, interval, MDE, laporan Markdown dan HTML statis | US-08 |
| Must | Data-dir teks (JSONL + artefak) yang bisa di-commit, export/import | US-09 |
| Should | Dashboard live (`pairbench serve`) | US-10 |
| Should | Preset Claude Code dan Codex | US-11 |
| Should | Mode CI dan contoh workflow | US-12 |
| Should | `task from-commit` | US-13 |
| Should | Rentang biaya p10–p90 di `plan`, `prune`, verdict per tag (`regression` / `targeted`) | US-06, US-09, US-08 |
| Could | Executor orb (`amp -ox`); isolasi filesystem dengan namespace (`bwrap`/`unshare`) bila tersedia di orb | — |
| Could | Skor kualitas LLM-judge (blinded A vs B), terpisah dari verdict | — |
| Could | `pairbench usage`: ringkasan biaya thread nyata per project/phase untuk retro | — |
| Won't (this version) | Leaderboard publik, impor benchmark publik, sandbox kode tak tepercaya, server hosted, ratusan run paralel, optimasi skill otomatis | — |

## 5. User stories

Istilah:

- **slot** = (task, arm, nomor ulangan). **blok** = (task, nomor ulangan): berisi slot kedua arm, dijalankan berurutan dengan urutan arm acak. **attempt** = satu percobaan menjalankan slot. **arm** = `baseline` atau `candidate`.
- **Status attempt:** `pass`, `fail`, `timeout`, `infra_error`, `cancelled`, `interrupted`, `skipped_budget`.
- **Attempt terhitung:** `pass`, `fail`, `timeout` (skor 0). Slot memakai attempt terhitung terakhirnya. Status lain dilaporkan per arm dan tidak masuk statistik. **Semua** attempt dihitung biayanya.
- **Skor** = angka 0–1 dari check. Check exit-code saja: 0 → skor 1, lainnya → 0. Check berskor menulis `{"score": x, "subchecks": {...}}` ke `$PAIRBENCH_SCORE_FILE`; status `pass` bila skor ≥ `pass_threshold` (default 1.0), selain itu `fail`.
- **check_status:** `ok` atau `error` (tidak ditemukan, crash, keluaran skor rusak, melewati `check_timeout_sec`). `error` membuat attempt `infra_error`.
- **Rahasia** = `AMP_API_KEY` dan entri `env` yang ditandai `secret: true` dengan nilai ≥ 8 karakter. Hanya nilai rahasia yang disamarkan (`***`).

### US-01 Menulis dan memvalidasi task

Sebagai Thomas, aku ingin menulis task dari pekerjaan factory (kode maupun dokumen) dan membuktikan pemeriksanya valid dan stabil sebelum dipakai, supaya hasil eval tidak rusak oleh task yang salah.

Format task: direktori berisi `task.yaml` (`id`, `instruction`, `setup` opsional, `check`, `timeout_sec`, `check_timeout_sec` default 300, `pass_threshold` default 1.0, `protected_paths`, `tags`), `fixture/`, `checks/`, `oracle/` (`solution.patch` atau `solve.sh`).

Suite contoh resmi (M0, juga suite kalibrasi M3; detail di [plan-review-2.md](reviews/plan-review-2.md#suite-contoh)):
1. `prd-stories-lint` (skill `prd-writer`): fixture PRD kecil fiktif dengan §1–4 terisi dan §5 kosong; check berskor 5 subcheck (setiap story ID Must punya heading; ≥ 2 AC Given/When/Then per story; baris `States:` lengkap; tanpa TBD/TODO/nanti; kalimat "Sebagai"); §1–4 dan file sumber di `protected_paths`.
2. `readiness-planted-defects` (skill `readiness-check`): PRD lengkap dengan 3 cacat yang ditanam; check berskor per item tabel readiness, verdict `NO-GO`, lokasi cacat disebut; PRD di `protected_paths`.
3. `personas-from-saved-sources` (skill `idea-discovery`, tanpa web): 6–8 halaman tersimpan dengan URL di header; check: struktur 2 persona + 1 anti-persona, setiap `Quote` muncul verbatim di salah satu sumber dengan URL yang cocok, setiap Pain punya URL atau `ASUMSI`.

Acceptance criteria:
- AC-01.1 Given task valid, When `pairbench task check <dir>` (default `--repeat 3`), Then exit code 0, output memuat `fixture: score <s> < threshold (3/3)` dan `oracle: score <s> >= threshold (3/3)`, dan event `task_check_passed` tercatat.
- AC-01.2 Given skor fixture ≥ `pass_threshold`, When `task check`, Then exit code 1 dan `check passes on the untouched fixture; the task tests nothing`.
- AC-01.3 Given skor oracle < `pass_threshold` di salah satu ulangan, When `task check`, Then exit code 1, nomor ulangan dan stdout/stderr check tercetak, dan `check is not stable` bila ulangan lain lulus.
- AC-01.4 Given `task.yaml` tanpa `check`, Then exit code 1 dan `<path>/task.yaml: missing required field "check"`. Given `timeout_sec: 0`, Then pesan menyebut path, field, nomor baris dan `must be > 0`.
- AC-01.5 Given `pairbench task new my-task` di direktori D, Then `D/my-task/` berisi kerangka lengkap, event `task_created` tercatat, dan `task check D/my-task` gagal dengan `fill in task.yaml: instruction, check`.
- AC-01.6 Given check melewati `check_timeout_sec`, Then exit code 1 dan `check timed out after <n>s`.
- AC-01.7 Given fixture berisi file konfigurasi test yang dikenal (`conftest.py`, `pytest.ini`, `package.json`, `vitest.config.*`, `jest.config.*`) yang tidak ada di `protected_paths`, When `task check`, Then peringatan `unprotected test config: <path>`.
- AC-01.8 Given tiga task suite contoh, When `task check` dijalankan pada masing-masing, Then ketiganya lulus.

States: empty (`task list` tanpa task: `no tasks; run pairbench task new <id>`) / loading (`fixture 2/3`, `oracle 1/3`) / error (AC-01.2–01.4, 01.6) / offline (tidak relevan) / success (AC-01.1).

### US-02 Mendefinisikan eksperimen

Sebagai Thomas, aku ingin satu file yang menyebut dua konfigurasi, daftar task, ulangan, batas biaya dan seed, supaya perbandingan bisa diulang persis.

`experiment.yaml`: `name`, `tasks`, `repeats` (default 3, minimum 2), `budget_usd` (wajib), `max_parallel` (default 1), `seed`, `max_attempts_per_slot` (default 2), `arms.<arm>`: `agent`, `mode`, `model`, `skills_overlay` (direktori lokal atau `git:<repo>@<rev>:<path>`), `env` (nama variabel; opsi `secret: true`), `tool_allowlist` (default: baca/edit file, shell, `skill`), `timeout_sec`.

**Identitas eksperimen** = hash isi `experiment.yaml` tanpa `budget_usd`, isi setiap direktori task, dan isi setiap overlay (rev git di-resolve ke SHA commit).

Acceptance criteria:
- AC-02.1 Given 2 task, `repeats: 3`, `seed: 7`, When `pairbench plan`, Then 12 slot dalam 6 blok; urutan blok dan urutan arm di dalam blok acak; `plan` dua kali dengan seed sama → urutan identik.
- AC-02.2 Given tanpa `budget_usd`, Then exit code 1 dan `budget_usd is required`.
- AC-02.3 Given kedua arm identik (termasuk hash overlay), Then peringatan `arms are identical; this measures noise only (A/A)` dan exit code 0.
- AC-02.4 Given `env` menyebut variabel yang tidak ada, Then exit code 1 dan pesan menyebut nama variabel itu saja.
- AC-02.5 Given isi task, overlay, atau field selain `budget_usd` berubah setelah eksperimen dimulai, When `run --resume`, Then ditolak dengan `experiment identity changed (<what>); use --new`.
- AC-02.6 Given `repeats: 1` tanpa `--pilot`, Then exit code 1 dan `repeats must be >= 2 for a paired comparison`.
- AC-02.7 Given eksperimen dengan identitas sama sudah punya attempt, When `run` tanpa `--resume`/`--new`, Then exit code 1 dan `experiment already started; use --resume or --new`.
- AC-02.8 Given `skills_overlay: git:<repo>@<rev>:skills/prd-writer`, When `plan`, Then rev di-resolve ke SHA commit yang tercetak dan masuk identitas; rev yang tidak ada → exit code 1.
- AC-02.9 Given desain yang p terkecil yang mungkin dicapainya ≥ 0.05 (misalnya 1 task × 2 ulangan), When `plan`, Then exit code 1 dan `this design cannot reach p < 0.05 (min p = <p>); add tasks or repeats, or pass --allow-underpowered`.

States: empty (`pairbench init` membuat contoh experiment + suite contoh) / error (AC-02.2, 02.4–02.9) / success (AC-02.1).

### US-03 Menjalankan attempt dengan aman dan jujur

Sebagai Thomas, aku ingin setiap attempt berjalan di workspace bersih dengan lingkungan terkontrol, dan dinilai setelah agent benar-benar berhenti, supaya hasil tidak tercemar dan agent tidak bisa bertindak ke luar tanpa izin.

Lokasi: workspace di `$TMPDIR/pairbench-<exp>/<attempt>/` (di luar repo mana pun); data-dir default `$XDG_STATE_HOME/pairbench` (US-09). Environment agent **dan** check: hanya `PATH`, `LANG`, `env` arm, kredensial yang diperlukan preset (hanya untuk agent, tidak untuk check), `HOME`/`XDG_*` sementara, `GIT_CONFIG_GLOBAL=/dev/null`, `PAIRBENCH_ATTEMPT_ID`; tanpa `GH_TOKEN`/`GITHUB_TOKEN` kecuali disebut di `env`. Selama agent berjalan, salinan check di data-dir di-chmod 000. Amp: `--settings-file` buatan pairbench dan `--mcp-config '{}'`.

Acceptance criteria:
- AC-03.1 Given 1 task × 2 arm × 2 ulangan dengan agent uji yang menulis jawaban benar, When `run` selesai, Then 4 attempt `pass` di 4 workspace berbeda, dan hash setiap workspace sebelum `setup` = hash(fixture + overlay arm) yang dihitung terpisah.
- AC-03.2 Given agent uji yang mencetak instruksinya, environment-nya, argv, cwd, isi workspace dan semua leluhur workspace, When attempt selesai, Then tidak ada path direktori task, path data-dir, atau nama file di `checks/` dalam output itu.
- AC-03.3 (negatif, terdokumentasi) Given agent uji yang menjalankan `find /` di bawah user yang sama, When attempt selesai, Then hasilnya dicatat di test e2e `e2e/threat-model/` dan dikutip di README sebagai batas yang tidak dijamin; salinan check di data-dir tidak terbaca selama agent berjalan (chmod 000), sedangkan direktori task asli operator terbaca.
- AC-03.4 Given agent uji yang mengubah file di `protected_paths`, When check dijalankan, Then file dipulihkan dari fixture sebelum check dan attempt menyimpan `integrity_warning: protected path modified: <path>`.
- AC-03.5 Given agent uji yang menulis ke data-dir atau direktori task, Then perubahan hash sebelum/sesudah menghasilkan `integrity_warning`; attempt tetap dinilai.
- AC-03.6 Given agent uji yang melewati `timeout_sec` dan memulai turunan dengan `setsid` + double-fork, When timeout, Then status `timeout` dan sebelum check tidak ada proses dengan `PAIRBENCH_ATTEMPT_ID` attempt itu di `/proc/*/environ`; check tetap dijalankan sebagai informasi.
- AC-03.7 Given `setup` gagal atau perintah agent tidak ditemukan, Then `infra_error` dengan penyebab.
- AC-03.8 Given agent keluar non-zero tanpa klasifikasi error infrastruktur dan skor 1, Then `pass` dan exit code agent disimpan terpisah.
- AC-03.9 Given Ctrl+C, Then attempt aktif `cancelled`, turunan mati, workspace dihapus kecuali `--keep-workspaces`.
- AC-03.10 Given attempt selesai, When `pairbench show <attempt-id>`, Then tersedia diff terhadap fixture+overlay, stdout/stderr agent, output check, skor dan subchecks, durasi per fase, exit code, warnings, provenance.
- AC-03.11 Given agent uji yang mencetak `GH_TOKEN` dan `HOME`, dan menanam `conftest.py` yang mencetak `GH_TOKEN` dan `AMP_API_KEY` saat check, Then `GH_TOKEN` kosong, `HOME` adalah direktori sementara attempt, dan output check menunjukkan kedua variabel kosong.
- AC-03.12 Given agent uji yang menulis nilai `AMP_API_KEY` ke file di workspace dan ke stdout, Then diff, log, output check, laporan HTML dan export JSONL menampilkan `***`, dan variabel non-rahasia (misalnya `NODE_ENV=test`) tidak disamarkan.

States: loading (`block 3/6 · t1 · candidate · rep 2 · 01:12 · $0.84 spent`) / error (AC-03.7) / offline (US-04 AC-04.3) / success (AC-03.1).

### US-04 Adapter agent

Sebagai Thomas, aku ingin menjalankan Amp tanpa glue code, dan agent lain lewat template command, supaya pairbench tidak terkunci pada satu vendor. Adapter generik juga menjadi agent uji deterministik untuk semua e2e.

Kontrak adapter generik v1 (M0): placeholder `{instruction_file}`, `{workspace}`, `{model}`; baris terakhir stdout boleh `{"pairbench_adapter": 1, "error_kind": "infra"|"agent"|null, "usage": {"input": n, "cache_creation": n, "cache_read": n, "output": n} | null, "cost_usd": x | null, "session_id": s | null, "tools": [..] | null, "skills_loaded": [..] | null}`.

Acceptance criteria:
- AC-04.1 Given arm `agent: amp, mode: <m>`, When attempt, Then pairbench memanggil `amp -x --stream-json -m <m>` di workspace, menyimpan `session_id` dari init, menjumlah `usage` dari pesan `assistant` saja (empat field terpisah), lalu membaca `amp threads usage <session_id>` sampai baris `Cost: $x` sama pada dua pembacaan berturut-turut berjarak 5 s (maks 60 s) dan mencatat `cost_source: reported`.
- AC-04.2 Given output `amp threads usage` tanpa baris `Cost:` yang bisa di-parse setelah 60 s, Then `cost_source: unknown`, peringatan `amp cost unavailable`, dan estimasi dipakai untuk budget (US-06).
- AC-04.3 Given error yang cocok dengan daftar pola infrastruktur terkontrol (rate limit, quota habis, autentikasi, jaringan, provider 5xx) di pesan `result` atau stderr, Then `infra_error` dengan `error_kind: infra`. Error lain (konteks penuh, batas turn, refusal, `is_error` tanpa pola infrastruktur) → `fail` skor 0 dengan alasan tercatat.
- AC-04.4 Given attempt `infra_error`, `interrupted` atau `cancelled` dan slot belum mencapai `max_attempts_per_slot`, Then slot dijalankan ulang **di dalam bloknya** sebelum blok berikutnya; semua attempt tercatat dan biayanya dihitung.
- AC-04.5 Given arm Amp tanpa `AMP_API_KEY`, When `plan`, Then exit code 1 dan `AMP_API_KEY is not set; create an access token in Amp settings`.
- AC-04.6 Given adapter generik dengan JSON v1 valid, Then usage, biaya (`reported`), `error_kind`, `tools`, `skills_loaded` tercatat sesuai isi.
- AC-04.7 Given JSON adapter rusak, Then status dari check, biaya `unknown`, `adapter output ignored: <reason>`.
- AC-04.8 Given tipe pesan stream Amp tidak dikenal, Then diabaikan dan dihitung di `adapter_warnings`. Contract test memakai rekaman output CLI Amp nyata yang versinya dicatat.

States: error (AC-04.2, 04.5, 04.7) / offline (AC-04.3) / success (AC-04.1, 04.6).

### US-05 Provenance dan gerbang lingkungan

Sebagai Thomas, aku ingin tahu skill, tool, server MCP, mode, versi dan instruksi yang benar-benar aktif, dan menolak attempt yang memegang tool di luar allowlist, supaya perbandingan tidak diam-diam membandingkan hal lain dan agent yang dievaluasi tidak bisa mengirim sesuatu ke orang lain.

Fakta yang sudah diperiksa (14 Nov 2026, [research.md](research.md#bukti-tambahan-yang-memengaruhi-desain)): `amp skill list --json` memberi `name`, `source`, `baseDir`; overlay `.agents/skills/` (`source: workspace-agents`) mengalahkan skill `global-user` bernama sama; HOME sementara tidak menyembunyikan skill `global-user` dan plugin.

Acceptance criteria:
- AC-05.1 Given overlay berisi `prd-writer`, When attempt Amp, Then sebelum agent mulai pairbench menyimpan `amp skill list --json` dari workspace, dan `show` menampilkan `prd-writer` dengan `source: workspace-agents`, `baseDir` di dalam workspace dan SHA-256 isi direktori skill.
- AC-05.2 Given skill overlay ter-resolve ke `baseDir` di luar workspace, When `plan` (yang membuat workspace sementara untuk pemeriksaan ini), Then exit code 1 dan `skill prd-writer resolves to <baseDir>, not to the overlay`; bila baru terdeteksi saat attempt, attempt `infra_error`.
- AC-05.3 Given pesan init Amp, When `tools` bukan subset `tool_allowlist` arm, atau `mcp_servers` tidak kosong, atau `agent_mode` berbeda dari `mode`, Then pairbench menghentikan agent saat membaca init (sebelum memproses pesan assistant pertama) dan attempt `infra_error` dengan `tool not allowed: <names>`, `unexpected MCP servers: <names>` atau `mode mismatch`. Diuji dengan agent uji yang mengumumkan tool palsu `slack_write`.
- AC-05.4 Given attempt selesai, Then `skills_loaded` (dari event tool-use `skill` di stream Amp, atau JSON adapter) tercatat, dan laporan menampilkan tingkat pemuatan skill overlay per arm dengan peringatan `candidate skill loaded in <k>/<n> attempts` bila < 100%.
- AC-05.5 Given arm tanpa overlay, Then provenance mencatat `no overlay` dan semua skill non-official yang aktif (`global-user`, `plugin`), dan laporan mencantumkannya.
- AC-05.6 Given attempt Amp, Then versi `amp --version` dan daftar file AGENTS.md di workspace, leluhurnya dan HOME sementara (path + hash) tercatat; AGENTS.md di luar workspace dan HOME sementara → `infra_error` `unexpected AGENTS.md: <path>`.
- AC-05.7 Given adapter generik, Then provenance berisi hash overlay dan `skill resolution not verified for generic agents`.

States: empty (AC-05.5) / error (AC-05.2, 05.3, 05.6) / success (AC-05.1).

### US-06 Biaya terlihat dan dibatasi

Sebagai Thomas, aku ingin estimasi sebelum mulai dan jaminan tidak ada attempt baru yang dimulai bila batas akan terlewati, supaya eval tidak menghabiskan kuota tanpa izin.

Aturan: biaya disimpan sebagai integer micro-dollar. **Estimasi per attempt** = p90 biaya historis untuk kunci (task, agent, mode/model, hash overlay), fallback (task, agent), fallback `--assume-cost-per-run`. **Admission** per blok: blok dimulai hanya bila `spent + reserved_in_flight + estimasi(blok) ≤ budget_usd`, dengan `estimasi(blok)` = jumlah estimasi kedua slot. Pada penolakan pertama, tidak ada blok baru yang dimulai; blok yang sedang berjalan diselesaikan; sisa slot `skipped_budget`. Attempt `unknown` dihitung dengan estimasinya.

Acceptance criteria:
- AC-06.1 Given tanpa riwayat dan tanpa `--assume-cost-per-run`, When `plan`, Then estimasi `unknown (no history)` dan saran `run --pilot`.
- AC-06.2 Given `--pilot` (1 ulangan per task per arm, dicatat sebagai eksperimen berlabel `pilot`, dikecualikan dari AC-02.6), `budget_usd: 1.00`, attempt uji berbiaya 0.40, 3 task, When `run --pilot`, Then pilot berjalan berurutan, berhenti memulai attempt baru begitu `spent ≥ budget_usd`: tepat 3 attempt (0.40, 0.80, 1.20), sisanya `skipped_budget`, dan riwayatnya dipakai sebagai estimasi eksperimen berikutnya.
- AC-06.3 Given estimasi 0.30 per attempt, budget 1.00, attempt uji melaporkan 0.30, 3 blok (6 slot), `max_parallel: 1`, When `run`, Then blok 1 (0.60) berjalan, blok 2 ditolak (0.60 + 0.60 > 1.00), 4 slot `skipped_budget`, ringkasan `budget reached: spent $0.60 of $1.00`.
- AC-06.4 Given estimasi 0.30, attempt uji melaporkan 0.60 (2× estimasi), budget 2.00, `max_parallel: 2`, Then log admission menunjukkan setiap keputusan admission memenuhi aturan pada saat itu, dan `spent_total` akhir ≤ `budget_usd` + `max_parallel` × 0.60 (M-6).
- AC-06.5 Given tanpa riwayat, tanpa `--assume-cost-per-run`, bukan `--pilot`, When `run`, Then exit code 1 dan `cannot enforce budget without a cost estimate; run --pilot or pass --assume-cost-per-run`.
- AC-06.6 Given `--dry-run`, Then tidak ada agent dipanggil dan output sama dengan `plan`.
- AC-06.7 Given `plan` dengan riwayat pilot, Then output memuat MDE dan nilainya sama dengan script referensi (US-08).

States: empty (AC-06.1) / loading (biaya berjalan di progres) / error (AC-06.5) / success (AC-06.3).

### US-07 Resume tanpa kehilangan atau duplikasi

Sebagai Thomas, aku ingin eksperimen yang terhenti (orb pause, update CLI, proses mati) bisa dilanjutkan, supaya attempt yang sudah dibayar tidak hilang atau terulang.

Acceptance criteria:
- AC-07.1 Given 3 blok, When pairbench dibunuh SIGKILL setelah blok 1 tercatat dan saat blok 2 berjalan, Then `run --resume <exp>` pertama-tama menghentikan proses yang `environ`-nya memuat `PAIRBENCH_ATTEMPT_ID` attempt aktif dan waktu mulai prosesnya (`/proc/<pid>/stat`) cocok dengan catatan, mencatat attempt itu `interrupted` dengan biayanya bila `session_id` tersedia, lalu menjalankan ulang slot itu dan blok 3; blok 1 tidak diulang.
- AC-07.2 Given eksperimen selesai, When `--resume`, Then tidak ada attempt baru dan `nothing to resume`.
- AC-07.3 Given proses dibunuh setelah agent selesai tetapi sebelum skor tercatat, When resume, Then attempt `interrupted` dan slot diulang; tidak ada skor setengah tercatat.
- AC-07.4 Given `--resume --budget 2.00`, Then batas baru dipakai, dicatat, dan berlaku untuk resume berikutnya; identitas tidak berubah.
- AC-07.5 Given dua proses `run --resume` pada eksperimen yang sama, Then proses kedua keluar dengan exit code 1 dan `experiment is locked by pid <p> (started <t>)`; lock dari proses yang sudah mati (pid + waktu mulai tidak cocok) diambil alih.
- AC-07.6 Given slot `cancelled` atau `interrupted`, When resume, Then slot diantrekan ulang selama belum mencapai `max_attempts_per_slot` (attempt `interrupted` dan `cancelled` ikut dihitung).

States: empty (AC-07.2) / loading (progres) / error (eksperimen tidak ditemukan: daftar ID; AC-07.5) / success (AC-07.1).

### US-08 Perbandingan berpasangan, verdict, laporan

Sebagai Thomas, aku ingin satu laporan yang menyatakan kandidat lebih baik, lebih buruk, belum cukup bukti, atau datanya kurang, dengan metode yang false-positive rate-nya terbukti terkontrol, supaya aku bisa mengutipnya di retro.

Metode (kontrak M0; semua angka di `contracts/stats.md`; inferensi **bersyarat pada suite ini**):

- **Hipotesis nol:** di setiap task, label arm tidak memengaruhi distribusi skor (slot dalam satu task dapat dipertukarkan antar arm). Untuk skor biner ini sama dengan pass rate yang sama. Untuk skor 0–1 uji ini juga menolak bila arm berbeda sebaran walau rata-rata sama; laporan menyatakannya.
- Per task t: d_t = s̄_cand,t − s̄_base,t dari slot terhitung. Statistik Δ = rata-rata d_t atas task berpasangan. **Task berpasangan** = task dengan ≥ 2 slot terhitung di kedua arm; task lain dikeluarkan dan didaftar dengan peringatan.
- **Uji exact:** distribusi nol Δ dihitung exact dengan konvolusi distribusi permutasi per task (independen antar task), aritmetika rasional. p dua sisi = P(|Δ*| ≥ |Δ_obs|). Tanpa cabang Monte Carlo. α = 0.05.
- **Interval:** bootstrap berstrata 95% (resample slot di dalam setiap sel task-arm), percentile, 10 000 resample, RNG dan algoritma resampling ditetapkan di kontrak. Dicetak dengan catatan `interval is descriptive; the verdict uses p`.
- **Verdict:** `insufficient_data` bila tidak ada task berpasangan atau p terkecil yang mungkin untuk sel teramati ≥ α; selain itu `better` bila p < α dan Δ > 0, `worse` bila p < α dan Δ < 0, `inconclusive` selainnya. Eksperimen belum selesai atau terpotong budget → label `interim`. Bila tingkat eksklusi per arm berbeda > 10 pp, verdict diberi tanda `unbalanced_exclusions`.
- **MDE:** pergeseran aditif terkecil pada grid 0.05 (skor di-clamp ke [0,1]) yang mencapai power ≥ 0.8 dengan uji exact di atas, 2 000 simulasi, seed tetap, pass rate baseline per task di-shrink (k+1)/(n+2) (0.5 bila tanpa data).
- **Kalibrasi M-8:** `pairbench selftest stats` mensimulasikan A/A (2 000 per sel, seed tetap) pada grid {3, 5, 10} task × {2, 3, 5} ulangan × pass rate {0.1, 0.5, 0.9}, plus sel tidak seimbang (2 vs 4 slot) dan skor 0–1 dengan distribusi identik; gagal bila batas atas Clopper-Pearson 95% FPR > 6% di sel mana pun.

Acceptance criteria (nilai harapan dari script referensi independen di `e2e/fixtures/`, bukan dari pairbench):
- AC-08.1 Given 3 task × 4 ulangan, kandidat 12/12 dan baseline 0/12, When `report`, Then Δ = +1.0000, p = 2/343 000 (5.8309e-06) persis, verdict `better`.
- AC-08.2 Given data A/A dengan pola identik, Then Δ = 0.0000, p = 1.0000, verdict `inconclusive`.
- AC-08.3 Given fixture campuran tidak degeneratif (3 task × 3 ulangan, skor berbeda per sel, satu sel tidak seimbang), Then Δ dan p sama persis dengan script referensi, dan batas interval sama persis (RNG kontrak).
- AC-08.4 Given satu dari tiga task hanya punya 1 slot terhitung di kandidat, Then task itu dikeluarkan dan didaftar, verdict dihitung dari dua task lain atau `insufficient_data` bila p terkecil yang mungkin ≥ α.
- AC-08.5 Given attempt berstatus `infra_error`, `cancelled`, `interrupted`, `skipped_budget`, Then tabel status per arm dan peringatan `N attempts excluded (<status>)` per status.
- AC-08.6 Given `report` dua kali pada data sama di mesin sama, Then byte identik (tanpa waktu jam dinding di isi; angka 4 desimal; p dalam notasi ilmiah 4 digit).
- AC-08.7 Given `report --format md`, Then memuat tabel per task, Δ, p, interval dengan catatan, MDE, verdict dan label, seed, versi pairbench, identitas eksperimen, SHA overlay per arm, tingkat pemuatan skill, skill non-official yang aktif, dan footer `pairbench-report: <identity>`.
- AC-08.8 Given `report --format html --out <dir>`, Then file statis `index.html`, halaman eksperimen dan halaman per attempt terbentuk tanpa server; output agent di-escape; axe-core tidak menemukan pelanggaran WCAG 2.2 AA serious/critical; semua link bisa dicapai dengan keyboard.
- AC-08.9 Given 0 pass di satu arm, Then biaya per pass `n/a`; jumlah attempt biaya `unknown` dicetak di sampingnya.
- AC-08.10 Given `pairbench selftest stats` di CI, Then lulus (M-8) dalam ≤ 10 menit di runner 2 vCPU.
- AC-08.11 Given tingkat eksklusi kandidat 3/9 dan baseline 0/9, Then verdict bertanda `unbalanced_exclusions`.

States: empty (`no countable runs yet`) / error (eksperimen tidak ditemukan) / success (AC-08.1, 08.7, 08.8).

### US-09 Data yang bertahan

Sebagai Thomas, aku ingin hasil eval bertahan setelah orb atau thread selesai, supaya riwayat biaya dan metrik tidak hilang.

Acceptance criteria:
- AC-09.1 Given `PAIRBENCH_HOME` atau `--data-dir`, Then semua data (JSONL append-only per eksperimen, artefak, event) ada di direktori itu; default `$XDG_STATE_HOME/pairbench` (di luar workspace dan repo).
- AC-09.2 Given eksperimen selesai, When `export <exp> --out f.jsonl` lalu `import f.jsonl` di data-dir kosong, Then `report` di kedua data-dir byte identik.
- AC-09.3 Given log attempt > 1 MB, Then dipotong dengan `[truncated N bytes]` dan ukuran asli dicatat.
- AC-09.4 Given data-dir berada di dalam repo git yang punya remote, When `run`, Then peringatan `data-dir is inside a git repository; logs and diffs may be published if committed`.

States: empty (data-dir dibuat otomatis) / error (tidak bisa ditulis: path dan errno) / success (AC-09.2).

### US-10 Dashboard live (Should)

- AC-10.1 Given `pairbench serve`, Then hanya `127.0.0.1` kecuali `--host`; `--host` non-loopback → `dashboard has no authentication`.
- AC-10.2 Given eksperimen berjalan, Then progres bertambah ≤ 5 s setelah attempt tercatat, verdict berlabel `interim`.
- AC-10.3 README menjelaskan `run` dan `serve` sebagai `amp orb service` dengan `--portal`.

### US-11 Preset Claude Code dan Codex (Should)

- AC-11.1 Given `agent: claude-code`, Then `claude -p --output-format json`, `total_cost_usd` → `cost_source: reported` ([docs](https://code.claude.com/docs/en/headless?x=1)).
- AC-11.2 Given `agent: codex`, Then `codex exec --json --sandbox workspace-write`, jumlah `turn.completed.usage`, biaya `computed` dari `rates.yaml` operator (versi dicatat) atau `unknown` ([docs](https://developers.openai.com/codex/noninteractive)).
- AC-11.3 Given binary tidak terpasang, When `plan`, Then exit code 1 dengan nama binary.

### US-12 Mode CI (Should)

- AC-12.1 Given `run --ci`, Then exit code 2 untuk `worse`, 0 untuk `better`/`inconclusive`, 3 untuk `insufficient_data` atau `interim`, 1 untuk error konfigurasi.
- AC-12.2 Contoh workflow `examples/ci/` hanya `workflow_dispatch` (tidak ada trigger otomatis yang memakai kredit), `AMP_API_KEY` dari secret, laporan Markdown sebagai artifact; dibuktikan dengan `actionlint` dan satu run manual yang disetujui Thomas.

### US-13 Task dari commit (Should)

- AC-13.1 Given commit C dan `--test-glob`, When `task from-commit <repo> <C> --check "<cmd>"`, Then fixture = parent C, `checks/` = file C yang cocok glob, oracle = diff non-test C, `instruction` = judul commit saja (tanpa body dan trailer).
- AC-13.2 Given hasil from-commit yang lulus pada fixture, Then `task check` gagal sesuai AC-01.2.

## 6. Non-functional requirements

| Area | Requirement |
| --- | --- |
| Performance | `report` < 2 s untuk 1 000 attempt di orb 2 vCPU/3 GB (uji exact via konvolusi). `plan` dengan MDE < 10 s. Overhead harness p95 ≤ 5 s (M-5). Laporan HTML: halaman eksperimen ≤ 500 KB untuk 1 000 attempt, tanpa JS wajib, LCP < 1.5 s dibuka dari disk (Lighthouse desktop). `max_parallel` > 2 → peringatan memori; peak RSS per attempt dicatat. |
| Reliability | PGID, `PAIRBENCH_ATTEMPT_ID` dan waktu mulai ditulis durable sebelum agent mulai; hasil sebelum blok berikutnya. Lock eksperimen (AC-07.5). Turunan dihentikan sebelum check: process group + subreaper (`PR_SET_CHILD_SUBREAPER`) + sapuan `/proc/*/environ` (AC-03.6). `infra_error` hanya untuk pola infrastruktur (AC-04.3). |
| Security | **Model ancaman:** operator tepercaya; agent yang dievaluasi semi-tepercaya (bisa salah atau mencari jalan pintas, tidak diasumsikan jahat). **Dijamin:** check tidak muncul di instruksi, environment, argv, cwd, workspace atau leluhurnya (AC-03.2); salinan check chmod 000 selama agent berjalan; check dari salinan milik pairbench setelah turunan agent mati; `protected_paths` dipulihkan; perubahan task/data-dir terdeteksi; environment agent dan check di-scrub; tool di luar allowlist, MCP, mode tidak sesuai, AGENTS.md asing → attempt ditolak saat init (AC-05.3, 05.6). **Tidak dijamin:** agent yang sengaja membaca seluruh filesystem user yang sama (AC-03.3, README). Rahasia disamarkan di semua artefak dan export (AC-03.12). HTML meng-escape output agent (OWASP A03). Dependency audit di CI. Disarankan API key service account workspace tanpa plugin/MCP (§7). |
| Accessibility | Laporan HTML (dan dashboard Should) WCAG 2.2 AA (AC-08.8); output CLI tidak bergantung warna saja. |
| Privacy | Inventaris di bawah. Tanpa telemetry. Data tinggal di mesin dan kendali pengguna; pairbench tidak mengirim data ke server selain yang dipanggil agent pilihan pengguna, jadi kewajiban pengendali data UU PDP tidak muncul dari pairbench sendiri; data pribadi di repo pengguna tetap tanggung jawab pengguna. Peringatan saat data-dir di repo dengan remote (AC-09.4). |
| Localization | English. Waktu ISO 8601 UTC di metadata saja. USD: integer micro-dollar di data, 4 desimal di laporan, 2 desimal di progres. |
| Portability | Linux x64 wajib (orb); macOS arm64 Should (subreaper Linux-only; macOS memakai process group + sapuan `ps`). Tanpa Docker. Operasi panjang di orb sebagai `amp orb service` (README). |

Inventaris data:

| Data | Kenapa | Disimpan di | Retensi |
| --- | --- | --- | --- |
| Instruksi task, fixture, check, oracle | Mendefinisikan task | Repo pengguna; salinan check di data-dir | Dikelola pengguna |
| Diff, stdout/stderr agent dan check (rahasia tersamar) | Menjelaskan hasil | Data-dir | Sampai dihapus (`prune` = Should) |
| Session/thread ID, usage, biaya | Biaya dan penelusuran | Data-dir | Sampai dihapus |
| Nama variabel env (tanpa nilai), provenance | Reproducibility | Data-dir | Sampai dihapus |
| Event `task_created`, `task_check_passed` | M-3, M-4 | Data-dir | Sampai dihapus |

## 7. External dependencies

| Dependency | Purpose | Access needed | Cost | Fallback |
| --- | --- | --- | --- | --- |
| Amp CLI (`-x --stream-json`, `-m`, `skill list --json`, `threads usage`, `--settings-file`, `--mcp-config`) | Agent utama, provenance, biaya | Access token `AMP_API_KEY` ([docs](https://ampcode.com/docs/cli/execute-mode)) | Kredit Amp per attempt; thread ini tercatat "Covered by your Amp Megawatt subscription" (`amp threads usage`, 14 Nov 2026). Setiap eksperimen nyata butuh batas yang disetujui Thomas | Adapter generik |
| Service account workspace Amp tanpa plugin/MCP | Menghindari tool akun pribadi di attempt | Dibuat oleh Thomas sebagai admin workspace bila spike R11 menunjukkan plugin/tool akun tidak bisa dimatikan lewat settings ([docs](https://ampcode.com/docs/cli/execute-mode)) | ASUMSI: tanpa biaya tambahan di luar kredit; dicek di Phase 5 | Bila tidak tersedia: attempt Amp hanya berjalan bila allowlist lolos (AC-05.3); bila tidak pernah lolos, Amp ditandai unsupported dan adapter generik menjadi jalur utama (blocker dicatat) |
| GitHub org `pabrik-aplikasi` + repo public | Kode dan rilis | Sudah ada | Gratis | — |
| Repo factory (`git:` overlay) | Overlay skill per rev | Clone `https://ampcode.com/@thomasbudiarjo/software-factory` (bisa dari orb factory) | Gratis | Overlay direktori lokal |
| Claude Code / Codex CLI (Should) | Pembanding | Akun masing-masing | Per token | Tidak dipakai |
| GitHub Actions (Should) | Mode CI | Repo public | Gratis untuk repo public (ASUMSI, dicek di Phase 5) | Manual |
| Hosting | — | — | US$0 | — |

## 8. Risks and assumptions

| Risk or assumption | Impact | Likelihood | Mitigation or validation |
| --- | --- | --- | --- |
| R1 Desain default hanya mendeteksi efek besar (perkiraan reviewer: 5 task × 3 ulangan ≈ 50 pp) | High | High | MDE di `plan` (AC-06.7); desain yang tidak mungkin signifikan ditolak (AC-02.9); skor 0–1; suite contoh; Phase 5 mengukur varians dan biaya nyata dan memakai MDE > 40 pp sebagai sinyal desain suite harus diubah sebelum M2 |
| R2 Biaya eval besar | High | Medium | Admission per blok, pilot berbatas (US-06); batas awal US$1 (M1) dan US$10 (M3) kecuali Thomas setuju lain |
| R3 Format output CLI agent berubah | Medium | High | Parser toleran (AC-04.8); contract test rekaman; versi CLI per attempt |
| R4 Overlay kalah oleh skill lokal | High | Low | Diperiksa: overlay menang atas `global-user`; tetap diverifikasi per attempt (AC-05.1–05.2) |
| R5 Baseline bergeser karena skill akun berubah | Medium | Medium | Overlay `git:<repo>@<rev>` untuk kedua arm (AC-02.8); skill non-official aktif dicatat (AC-05.5) |
| R6 Agent membaca check atau mengubah grader | High | Medium | Jaminan terbatas + chmod 000 + `protected_paths` + deteksi integritas + peringatan config test (§6, AC-01.7); isolasi namespace/orb = Could, di-spike Phase 5 |
| R7 Test lulus ≠ hasil bagus | Medium | High | Subchecks; diff dan log di laporan; LLM-judge = Could |
| R8 Turunan proses lolos | Medium | Medium | Subreaper + sapuan environ (AC-03.6); service orb yang dimulai agent dicatat (`amp orb service list` sebelum/sesudah) |
| R9 Biaya Amp hanya teks dan bisa terlambat | Medium | High | Polling sampai stabil (AC-04.1); `unknown` + estimasi (AC-04.2) |
| R10 Membangun ulang yang bisa diadopsi (Coder Eval) | Medium | Medium | Aturan keputusan build vs adopt di gate Phase 4 ([plan-review-2.md](reviews/plan-review-2.md#options-for-thomas)) |
| R11 Agent memakai tool/kredensial operator untuk aksi ke luar | High | Medium | Allowlist tool + tolak MCP saat init (AC-05.3), env scrub (AC-03.11), service account (§7). Spike Phase 5: apakah plugin/tool akun bisa dimatikan lewat `--settings-file`; apakah instruksi akun (AGENTS.md level user di server) ikut dimuat |
| R12 Task dokumen tidak bisa dicek deterministik | Medium | Medium | Check berskor; suite contoh 3 task dokumen (US-01) |
| R13 Mengintip hasil sementara | Medium | Medium | Label `interim`; CI hanya eksperimen selesai (AC-12.1) |
| R14 Check ditulis untuk perubahan yang dievaluasinya (teaching to the test) | Medium | Medium | Tag `regression`/`targeted` dengan verdict per tag (Should); kebijakan keputusan di retro (Options for Thomas) |
| R15 Kontaminasi lintas arm lewat file factory di mesin yang sama (skill, PRD nyata) | High | Medium | Workspace di luar repo; README: jalankan eksperimen di orb tanpa checkout factory/project; AC-03.3 mencatat apa yang terlihat |
| A1 Agent bisa dijalankan non-interaktif dengan output mesin | — | — | Terverifikasi dari docs |
| A2 Operator menulis task tepercaya | — | — | Dinyatakan di README dan `plan` |

## 9. Milestones

| Milestone | Contents | Exit criteria |
| --- | --- | --- |
| M0 Foundations and contracts | JSON Schema `task.yaml`/`experiment.yaml`, format JSONL data-dir dan event, adapter JSON v1, `contracts/stats.md` + script referensi, `selftest stats`, suite contoh 3 task, CI | Kontrak di `contracts/`; script referensi menghasilkan nilai AC-08; `selftest stats` lulus (M-8); 3 task contoh lulus `task check`; CI hijau |
| M1 Core loop | US-01, US-02, US-03, US-04, US-05 (AC-05.1–05.3), US-06, US-07, US-08 (Markdown), US-09 | e2e AC terkait hijau dengan agent uji deterministik; 1 attempt Amp nyata ≤ US$1 lulus dengan izin Thomas |
| M2 Supporting features | US-05 (AC-05.4–05.7), US-08 (HTML) | e2e hijau; axe-core bersih |
| M3 Polish and launch | README (model ancaman, orb service), eksperimen kalibrasi ≤ US$10 (A/A pada suite contoh, lalu kontrol positif skill vs tanpa skill pada `prd-stories-lint`), usulan perubahan skill `retro` factory (trailer `pairbench-report:`) untuk disetujui Thomas, Should bila waktu tersisa | Semua Must hijau; laporan kalibrasi di `docs/`; rilis GitHub v0.1.0 |

## 10. Budget

- **Batas waktu:** ≤ 20 hari kerja agent untuk Must (workflow factory: MVP 2–4 minggu).
- **Estimasi (hari agent):** M0 2 (schema, kontrak statistik, script referensi, selftest, suite contoh). US-01 1.5, US-02 1, US-03 3, US-04 2, US-05 1.5, US-06 1.5, US-07 1.5, US-08 mesin statistik 1.5 + laporan Markdown/HTML 1.5, US-09 0.5 → 15.5. Polish 1. **Total Must ≈ 18.5 hari.** Should (US-10–13 + item Should lain) ≈ 5.5 hari, hanya bila tersisa.
- **Money:** hosting US$0/bulan. Eval nyata: US$1 (M1) + US$10 (M3) + eksperimen keputusan M-1 (diusulkan US$10 masing-masing), semuanya dengan persetujuan Thomas. Kredit Amp untuk build: thread ini US$0.04 per `amp threads usage` sampai Phase 3 (tidak termasuk subagent bila ditagih terpisah); usulan batas build untuk disetujui di gate: lihat [plan-review-2.md](reviews/plan-review-2.md#options-for-thomas).
