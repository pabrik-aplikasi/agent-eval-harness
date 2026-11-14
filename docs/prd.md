# PRD — pairbench (Coding-agent eval harness)

Status: draft
Delivery: OSS (Apache-2.0, repo public `pabrik-aplikasi/agent-eval-harness`); tanpa hosting wajib
Project `115-agent-eval-harness` · kategori 1 · Phase 2, 14 Nov 2026
Bahasa: dokumen untuk Thomas dalam bahasa Indonesia; CLI, dashboard, pesan error dan laporan dalam bahasa Inggris.

## 1. Problem and goals

- **Problem:** factory mengubah skill, prompt dan model tanpa bukti apakah perubahan membantu ([brief.md](brief.md), [research.md](research.md#ringkasan-temuan)). Satu run per sisi tidak bisa membedakan efek dari noise: pass@1 satu run bervariasi 2.2–6.0 pp di SWE-bench Verified ([arXiv 2602.07150](https://arxiv.org/html/2602.07150)). Harness yang ada (Harbor, Coder Eval) tidak punya adapter Amp dan mengandalkan Docker atau sandbox cloud ([research.md](research.md#competitors)); daemon Docker tidak aktif di orb factory.
- **Goals:**
  - G1 Menjawab "apakah konfigurasi kandidat lebih baik dari baseline pada task factory?" dengan verdict `better` / `worse` / `inconclusive` dan interval kepercayaan. ← pain Persona 1 "satu run per sisi tidak bisa membedakan efek dari noise"; tema complaint "noise" ([research.md](research.md#complaint-themes)).
  - G2 Menjalankan Amp secara native, plus agent lain lewat adapter command generik, tanpa Docker. ← pain Persona 1 "harness yang ada tidak mendukung Amp".
  - G3 Membuat biaya terlihat dan dibatasi sebelum dan selama run. ← pain Persona 1 kuota, Persona 2 biaya tidak seragam; tema "biaya tidak terlihat".
  - G4 Menulis task dari pekerjaan factory sendiri dalam ≤ 20 menit per task, dengan pemeriksa yang dibuktikan valid sebelum dipakai. ← tema "benchmark publik tidak mewakili pekerjaan sendiri"; issue Harbor tentang verifier rapuh.
  - G5 Membedakan kegagalan infrastruktur dari kegagalan agent dan membuktikan versi skill yang benar-benar aktif. ← tema "kegagalan infrastruktur tercatat sebagai kegagalan agent" dan "integritas penilaian bocor".
- **Non-goals:**
  - Leaderboard publik, benchmark tetap, atau impor dataset publik (SWE-bench, Terminal-Bench). Harbor sudah melayani ini (anti-persona, [personas.md](personas.md#anti-persona-dr-lin-peneliti-lab-yang-membuat-benchmark-publik)).
  - Sandbox keamanan untuk kode tak tepercaya. Task dan fixture berasal dari operator sendiri; workspace sementara bukan batas keamanan.
  - SaaS hosted multiuser, akun, login.
  - Menilai kualitas kode dengan LLM-judge di MVP (Could).
  - Mengoptimalkan skill secara otomatis (agent menulis skill sendiri). SkillsBench menunjukkan skill buatan agent sendiri menurunkan pass rate ([SkillsBench 1.1](https://www.skillsbench.ai/blogs/skillsbench-1-1)).

## 2. Success metrics

| Metric | Target | Window | How measured |
| --- | --- | --- | --- |
| M-1 Dipakai di retro factory | ≥ 2 file `docs/retro.md` project factory mengutip laporan pairbench | 60 hari setelah M3 | `grep -l "pairbench" docs/retro.md` di repo project `pabrik-aplikasi/*` |
| M-2 Infra error rate | < 5% dari run yang dimulai | 200 run pertama setelah M3 | `SELECT status, count(*) FROM runs` (status `infra_error` / total) |
| M-3 Task valid saat pertama dipakai | ≥ 90% task yang lulus `task check` tidak pernah menghasilkan check error pada run sungguhan | 20 task pertama | Event `task_check_passed` + run dengan `check_status = error` per task |
| M-4 Waktu menulis task | median ≤ 20 menit dari `task new` sampai `task check` lulus pertama kali | 10 task pertama | Selisih timestamp event `task_created` dan `task_check_passed` |
| M-5 Overhead harness | p95 ≤ 5 s per run (di luar waktu agent dan waktu check) | 200 run pertama | Field `overhead_ms` = `total_ms − agent_ms − check_ms` per run |
| M-6 Biaya tidak melewati batas | 0 eksperimen dengan biaya tercatat > batas + biaya 1 run termahal yang sedang berjalan | Semua eksperimen 60 hari pertama | `SUM(cost_usd)` per eksperimen vs `budget_usd` |
| M-7 Laporan reproducible | 100% `pairbench report` diulang pada data sama menghasilkan byte yang identik | Setiap rilis | e2e test membandingkan dua output |

## 3. Personas

Primary: **Thomas, operator AI software factory**. Secondary: Ben, developer yang merawat AGENTS.md dan skill tim kecil. Anti-persona: peneliti lab benchmark publik. Lihat [personas.md](personas.md).

## 4. MVP scope (MoSCoW)

| Priority | Feature | Story IDs |
| --- | --- | --- |
| Must | Format task (instruksi, fixture, setup, check tersembunyi, oracle) + `task check` | US-01 |
| Must | File eksperimen: baseline vs kandidat (agent, model/mode, skill overlay, env), ulangan, batas biaya | US-02 |
| Must | Runner: workspace sekali pakai, timeout, hentikan seluruh process tree, check setelah agent berhenti, status run terpisah | US-03 |
| Must | Adapter Amp (preset) + adapter command generik (JSON berversi lewat stdout) | US-04 |
| Must | Bukti skill aktif per run (provenance) | US-05 |
| Must | Estimasi biaya, dry-run, batas biaya dihormati | US-06 |
| Must | Resume eksperimen setelah proses mati tanpa kehilangan hasil | US-07 |
| Must | Perbandingan berpasangan: verdict + interval kepercayaan + laporan Markdown | US-08 |
| Must | Dashboard lokal: daftar eksperimen, perbandingan, detail run (diff, log, output check) | US-09 |
| Should | Preset Claude Code (`claude -p --output-format json`) dan Codex (`codex exec --json`) | US-10 |
| Should | Mode CI: exit code non-zero bila verdict `worse`; contoh GitHub Action untuk PR yang mengubah `skills/` | US-11 |
| Should | `task from-commit`: buat task dari commit yang sudah merge (fixture = parent, oracle = diff, check = test yang diubah) | US-12 |
| Could | Menjalankan run Amp di orb terpisah (`amp -ox`) sebagai isolasi lebih kuat | — |
| Could | Skor kualitas LLM-judge sebagai metrik sekunder, tidak memengaruhi verdict | — |
| Could | Grafik tren lintas eksperimen; dashboard bersama dengan auth | — |
| Won't (this version) | Leaderboard publik, impor benchmark publik, sandbox kode tak tepercaya, eksekusi di server hosted, ratusan run paralel, optimasi skill otomatis | — |

## 5. User stories

Istilah: **run** = satu eksekusi agent pada satu task dengan satu konfigurasi dan satu nomor ulangan. **Arm** = baseline atau kandidat. **Workspace** = direktori sementara tempat agent bekerja. Status run: `pass`, `fail`, `timeout`, `infra_error`, `cancelled`, `skipped_budget`. Hanya `pass`, `fail` dan `timeout` dihitung sebagai hasil agent; `infra_error`, `cancelled`, `skipped_budget` dilaporkan terpisah dan tidak masuk statistik.

### US-01 Menulis dan memvalidasi task

Sebagai Thomas, aku ingin menulis task dari pekerjaan factory dan membuktikan pemeriksanya valid sebelum dipakai, supaya hasil eval tidak rusak oleh task yang salah.

Format task: satu direktori berisi `task.yaml` (`id`, `instruction`, `setup` command opsional, `check` command, `timeout_sec`, `tags`), `fixture/` (isi awal workspace), `checks/` (file pemeriksa, tidak terlihat oleh agent), `oracle/` (patch `solution.patch` atau script `solve.sh`).

Acceptance criteria:
- AC-01.1 Given task valid yang check-nya gagal pada fixture dan lulus setelah oracle diterapkan, When `pairbench task check <dir>` dijalankan, Then exit code 0 dan output memuat `fixture: fail (expected)` dan `oracle: pass (expected)`.
- AC-01.2 Given task yang check-nya sudah lulus pada fixture tanpa perubahan, When `task check` dijalankan, Then exit code non-zero dan pesan `check passes on the untouched fixture; the task tests nothing`.
- AC-01.3 Given task yang check-nya gagal setelah oracle diterapkan, When `task check` dijalankan, Then exit code non-zero dan output memuat stdout/stderr check dari percobaan oracle.
- AC-01.4 Given `task.yaml` tanpa field `check` atau dengan `timeout_sec` ≤ 0, When `task check` dijalankan, Then exit code non-zero dan pesan menyebut nama field dan baris yang salah.
- AC-01.5 Given `pairbench task new <id>`, When dijalankan di direktori kosong, Then dibuat kerangka task yang langsung gagal `task check` dengan pesan yang menunjuk file mana yang harus diisi.

States: empty (belum ada task: `pairbench task list` mencetak cara membuat task pertama) / loading (progres langkah fixture → oracle) / error (AC-01.2–01.4) / offline (tidak relevan: tanpa jaringan) / success (AC-01.1).

### US-02 Mendefinisikan eksperimen baseline vs kandidat

Sebagai Thomas, aku ingin satu file yang menyebut dua konfigurasi, daftar task, jumlah ulangan dan batas biaya, supaya perbandingan bisa diulang persis.

Experiment file `experiment.yaml`: `name`, `tasks` (path atau glob), `repeats` (default 3, minimum 1), `budget_usd` (wajib), `max_parallel` (default 1), `arms.baseline` dan `arms.candidate`, masing-masing berisi `agent` (preset atau `command`), `model`/`mode` opsional, `skills_overlay` (direktori yang disalin ke `.agents/skills/` di workspace) opsional, `env` (daftar nama variabel yang diteruskan; nilai tidak disimpan), `timeout_sec` opsional.

Acceptance criteria:
- AC-02.1 Given experiment file valid dengan 2 task, `repeats: 3`, When `pairbench plan experiment.yaml` dijalankan, Then output mencantumkan 12 run (2 task × 2 arm × 3 ulangan) dan urutan run yang sudah diacak dengan seed yang tercetak.
- AC-02.2 Given experiment file tanpa `budget_usd`, When `plan` atau `run` dijalankan, Then exit code non-zero dan pesan `budget_usd is required`.
- AC-02.3 Given kedua arm identik di semua field, When `plan` dijalankan, Then output memuat peringatan `arms are identical; this measures noise only` dan exit code 0 (berguna sebagai kalibrasi noise A/A).
- AC-02.4 Given `env` menyebut variabel yang tidak ada di environment, When `plan` dijalankan, Then exit code non-zero dan pesan menyebut nama variabel tanpa mencetak nilai variabel lain.
- AC-02.5 Given experiment file berubah setelah eksperimen dimulai, When `run` dilanjutkan, Then pairbench menolak dengan pesan `experiment file changed (hash mismatch); start a new experiment` kecuali `--new` diberikan.

States: empty (tidak ada file: pesan menunjuk `pairbench init`) / loading (tidak relevan) / error (AC-02.2, 02.4, 02.5) / offline (tidak relevan) / success (AC-02.1).

### US-03 Menjalankan run dengan aman dan jujur

Sebagai Thomas, aku ingin setiap run berjalan di workspace bersih dan dinilai setelah agent benar-benar berhenti, supaya hasil tidak tercemar oleh run lain, oleh agent yang membaca pemeriksa, atau oleh masalah infrastruktur.

Acceptance criteria:
- AC-03.1 Given eksperimen 1 task × 2 arm × 1 ulangan dengan agent uji yang menulis jawaban benar, When `pairbench run` selesai, Then dua run tercatat `pass`, masing-masing di workspace berbeda, dan workspace awal setiap run identik dengan `fixture/` (hash isi sama).
- AC-03.2 Given agent uji yang mencoba membaca `checks/` (mencari file check di filesystem di bawah direktori task dan workspace), When run selesai, Then file check tidak ditemukan oleh agent, dan check baru disalin ke workspace setelah process tree agent dihentikan.
- AC-03.3 Given agent uji yang berjalan lebih lama dari `timeout_sec` dan memulai proses anak di background, When timeout tercapai, Then status `timeout`, seluruh process group agent mati sebelum check dimulai (tidak ada PID anak yang masih hidup), dan check tetap dijalankan dan dicatat hanya sebagai informasi.
- AC-03.4 Given perintah agent tidak ditemukan atau `setup` task gagal, When run dijalankan, Then status `infra_error` dengan pesan penyebab, dan run itu tidak dihitung di statistik.
- AC-03.5 Given agent keluar dengan kode non-zero tetapi check lulus, When run dicatat, Then status mengikuti check (`pass`) dan exit code agent disimpan terpisah.
- AC-03.6 Given Ctrl+C saat run berjalan, When pairbench berhenti, Then run aktif dicatat `cancelled`, process tree agent mati, dan workspace dibiarkan untuk diperiksa bila `--keep-workspaces` aktif, atau dihapus.
- AC-03.7 Given run selesai, When detail run dibuka, Then tersedia: diff workspace terhadap fixture, stdout/stderr agent (dengan nilai variabel `env` disamarkan), stdout/stderr check, durasi agent/check/total, exit code.

States: empty (eksperimen tanpa task: error dari US-02) / loading (progres `run 3/12 · task t1 · candidate · rep 2 · 01:12`) / error (AC-03.4) / offline (provider agent tidak bisa dihubungi: run `infra_error` bila agent melaporkan error koneksi; ditandai `retryable`) / success (AC-03.1).

### US-04 Adapter agent: Amp preset dan command generik

Sebagai Thomas, aku ingin menjalankan Amp tanpa menulis glue code, dan agent lain lewat template command, supaya pairbench tidak terkunci pada satu vendor.

Kontrak adapter generik: command template dengan placeholder `{instruction_file}`, `{workspace}`, `{model}`; adapter boleh menulis satu baris JSON terakhir ke stdout berformat `{"pairbench_adapter": 1, "tokens_in": n, "tokens_out": n, "cost_usd": x | null, "session_id": "..." | null}`. Bila baris itu tidak ada, token dan biaya `unknown`.

Acceptance criteria:
- AC-04.1 Given arm dengan `agent: amp` dan `AMP_API_KEY` tersedia, When run dijalankan, Then pairbench memanggil `amp -x --stream-json` di workspace, menyimpan thread ID dari `session_id`, menjumlah token dari field `usage` pesan assistant, dan mencatat biaya dengan `cost_source` (`reported`, `computed`, atau `unknown`).
- AC-04.2 Given arm Amp tanpa `AMP_API_KEY`, When `plan` dijalankan, Then exit code non-zero dengan pesan `AMP_API_KEY is not set; create an access token in Amp settings`.
- AC-04.3 Given arm `command` yang mencetak baris JSON adapter valid, When run selesai, Then token dan biaya tercatat dengan `cost_source: reported`.
- AC-04.4 Given arm `command` yang mencetak JSON adapter rusak, When run selesai, Then status run tetap ditentukan check, biaya `unknown`, dan peringatan `adapter output ignored: <reason>` tersimpan di run.
- AC-04.5 Given output `--stream-json` Amp memuat tipe pesan yang tidak dikenal, When diproses, Then pesan itu diabaikan tanpa membuat run gagal, dan jumlahnya tercatat di `adapter_warnings`.

States: empty (tidak relevan) / loading (tidak relevan) / error (AC-04.2, 04.4) / offline (Amp tidak bisa dihubungi: `infra_error retryable`) / success (AC-04.1, 04.3).

### US-05 Bukti skill yang aktif

Sebagai Thomas, aku ingin tahu versi skill mana yang benar-benar dimuat agent di setiap run, supaya perbandingan "skill v1 vs v2" tidak diam-diam membandingkan skill yang sama.

Latar: Amp memakai skill pertama dengan `name` tertentu menurut urutan sumber; `~/.config/agents/skills/` mengalahkan `.agents/skills/` di project, dan project mengalahkan personal skills repository ([Amp Skills](https://ampcode.com/docs/customize/skills)).

Acceptance criteria:
- AC-05.1 Given arm dengan `skills_overlay` berisi skill `prd-writer`, When run Amp dijalankan, Then pairbench menyimpan hasil `amp skills list --json` dari workspace sebelum agent mulai, dan detail run menampilkan path asal `prd-writer` di dalam workspace serta hash SHA-256 isi `SKILL.md`.
- AC-05.2 Given skill dengan nama sama ada di direktori lokal yang mengalahkan overlay (misalnya `~/.config/agents/skills/prd-writer`), When `plan` atau run dijalankan, Then run tidak dimulai, status `infra_error`, dengan pesan `skill prd-writer resolves to <path>, not to the overlay`.
- AC-05.3 Given baseline dan kandidat memakai overlay dengan hash `SKILL.md` identik, When `plan` dijalankan, Then muncul peringatan yang sama dengan AC-02.3.
- AC-05.4 Given agent `command` generik, When run dijalankan, Then provenance berisi hash isi overlay yang disalin dan catatan `skill resolution not verified for generic agents`.

States: empty (arm tanpa overlay: provenance mencatat `no overlay`) / loading (tidak relevan) / error (AC-05.2) / offline (tidak relevan) / success (AC-05.1).

### US-06 Biaya terlihat dan dibatasi

Sebagai Thomas, aku ingin estimasi biaya sebelum mulai dan jaminan run baru berhenti dimulai saat batas tercapai, supaya eval tidak menghabiskan kuota tanpa izin.

Acceptance criteria:
- AC-06.1 Given eksperimen baru tanpa riwayat run, When `plan` dijalankan, Then output memuat jumlah run dan estimasi `unknown (no history)`, dan menyarankan `--pilot` (1 ulangan per task per arm) dulu.
- AC-06.2 Given riwayat run sebelumnya untuk task dan agent yang sama, When `plan` dijalankan, Then estimasi = median biaya historis per (task, agent, model) × jumlah run, dicetak bersama rentang p10–p90.
- AC-06.3 Given `budget_usd: 1.00` dan setiap run uji melaporkan `cost_usd: 0.40`, When `run` dijalankan untuk 6 run dengan `max_parallel: 1`, Then tepat 3 run dijalankan (0.40, 0.80, 1.20 — run ke-3 dimulai karena total sebelum mulai 0.80 < 1.00), 3 run sisanya berstatus `skipped_budget`, dan ringkasan mencetak `budget reached: spent $1.20 of $1.00`.
- AC-06.4 Given biaya run `unknown`, When batas dihitung, Then pairbench memakai estimasi median historis untuk run itu bila ada; bila tidak ada, eksperimen berhenti setelah run pertama dengan biaya `unknown` kecuali `--allow-unknown-cost` diberikan.
- AC-06.5 Given `--dry-run`, When `run` dijalankan, Then tidak ada agent yang dipanggil, dan output sama dengan `plan`.

States: empty (tanpa riwayat: AC-06.1) / loading (biaya berjalan tampil di progres) / error (AC-06.4) / offline (tidak relevan) / success (AC-06.2, 06.3).

### US-07 Resume tanpa kehilangan hasil

Sebagai Thomas, aku ingin eksperimen yang terhenti (orb pause, proses mati) bisa dilanjutkan, supaya run yang sudah dibayar tidak hilang atau terulang.

Acceptance criteria:
- AC-07.1 Given eksperimen 6 run, When proses pairbench dibunuh dengan SIGKILL setelah run ke-3 tercatat dan saat run ke-4 berjalan, Then `pairbench run --resume <experiment>` menjalankan tepat run ke-4 sampai ke-6, run 1–3 tidak diulang, dan run ke-4 lama tercatat `infra_error` (`interrupted`).
- AC-07.2 Given eksperimen yang sudah selesai, When `--resume` dijalankan, Then tidak ada run baru dan pesan `nothing to resume`.
- AC-07.3 Given hasil run hanya dianggap tercatat setelah ditulis durable, When proses dibunuh tepat setelah agent selesai tetapi sebelum check selesai, Then run itu diulang saat resume, bukan dicatat setengah.

States: empty (AC-07.2) / loading (progres resume) / error (eksperimen tidak ditemukan: pesan dengan daftar ID) / offline (tidak relevan) / success (AC-07.1).

### US-08 Perbandingan berpasangan dan verdict

Sebagai Thomas, aku ingin satu tabel yang menyatakan kandidat lebih baik, lebih buruk, atau belum cukup bukti, dengan alasan statistiknya, supaya aku bisa mengutipnya di retro.

Metode (dikunci di M0 sebagai kontrak): per task t, pass rate p_arm,t = jumlah `pass` / jumlah run terhitung. Selisih berpasangan d_t = p_cand,t − p_base,t. Estimasi utama = rata-rata d_t atas task. Interval kepercayaan 95% dengan bootstrap berpasangan dua tingkat (resample task, lalu resample run dalam setiap task-arm), 10 000 resample, seed tetap dicetak. Verdict `better` bila batas bawah > 0, `worse` bila batas atas < 0, selain itu `inconclusive`. Laporan juga mencetak biaya rata-rata per run, biaya per `pass`, durasi median per arm, dan jumlah run per status.

Acceptance criteria:
- AC-08.1 Given data uji dengan 3 task dan 4 ulangan di mana kandidat lulus 12/12 dan baseline 0/12, When `pairbench report <exp>` dijalankan, Then verdict `better`, estimasi +1.00, dan batas bawah interval > 0.
- AC-08.2 Given data uji A/A dengan pola lulus identik di kedua arm, When `report` dijalankan, Then estimasi 0.00 dan verdict `inconclusive`.
- AC-08.3 Given data uji dengan `infra_error` pada 2 run kandidat, When `report` dijalankan, Then run itu tidak masuk penyebut pass rate, tercantum di tabel status, dan laporan memuat peringatan `2 runs excluded (infra_error)`.
- AC-08.4 Given task yang hanya punya run terhitung di satu arm, When `report` dijalankan, Then task itu dikeluarkan dari selisih berpasangan dan disebut di bagian `unpaired tasks`.
- AC-08.5 Given `report` dijalankan dua kali pada data sama, When kedua output dibandingkan, Then byte-nya identik (M-7).
- AC-08.6 Given `report --format md`, When dijalankan, Then output Markdown memuat tabel per task, verdict, interval, seed, versi pairbench, hash experiment file dan hash overlay skill setiap arm.
- AC-08.7 Nilai harapan untuk AC-08.1–08.4 dihitung di luar kode pairbench (script referensi terpisah di `e2e/fixtures/`), bukan dari output pairbench itu sendiri.

States: empty (eksperimen tanpa run terhitung: `no countable runs yet`) / loading (tidak relevan; < 2 s) / error (eksperimen tidak ditemukan) / offline (tidak relevan) / success (AC-08.1).

### US-09 Dashboard lokal

Sebagai Thomas, aku ingin membuka hasil di browser dan menelusuri run yang gagal sampai ke diff dan log, supaya aku mengerti kenapa kandidat menang atau kalah.

Acceptance criteria:
- AC-09.1 Given 2 eksperimen tercatat, When `pairbench serve` dijalankan dan halaman awal dibuka, Then daftar menampilkan kedua eksperimen dengan nama, tanggal, verdict, jumlah run dan total biaya.
- AC-09.2 Given eksperimen dengan verdict, When halaman eksperimen dibuka, Then tabel per task menampilkan pass rate tiap arm, selisih, dan baris ringkasan dengan interval yang sama persis dengan `report`.
- AC-09.3 Given run berstatus `fail`, When baris run diklik, Then halaman detail menampilkan diff workspace, log agent (nilai rahasia tersamar), output check, provenance skill, durasi dan biaya dengan sumbernya.
- AC-09.4 Given eksperimen sedang berjalan, When halaman eksperimen terbuka, Then progres run bertambah tanpa reload manual dalam ≤ 5 s setelah run tercatat.
- AC-09.5 Given `serve` dijalankan, When port diperiksa, Then server hanya mendengarkan di `127.0.0.1` kecuali `--host` diberikan, dan saat `--host` non-loopback diberikan muncul peringatan `dashboard has no authentication`.
- AC-09.6 Given database tanpa eksperimen, When halaman awal dibuka, Then empty state menampilkan perintah `pairbench init` dan `pairbench task new`.
- AC-09.7 Given halaman awal, eksperimen dan detail run, When diperiksa dengan axe-core, Then tidak ada pelanggaran WCAG 2.2 AA tingkat serious atau critical, dan seluruh navigasi bisa dilakukan dengan keyboard.

States: empty (AC-09.6) / loading (skeleton tabel; detail run memuat log bertahap) / error (database tidak bisa dibuka: halaman error dengan path dan perintah perbaikan) / offline (dashboard lokal; bila server mati, halaman menampilkan `server stopped`) / success (AC-09.1–09.3).

### US-10 Preset Claude Code dan Codex (Should)

Sebagai Ben, aku ingin memakai preset `claude-code` dan `codex`, supaya aku bisa membandingkan agent tanpa menulis adapter.

Acceptance criteria:
- AC-10.1 Given arm `agent: claude-code`, When run dijalankan, Then pairbench memanggil `claude -p --output-format json` dan mencatat `total_cost_usd` dengan `cost_source: reported` ([docs](https://code.claude.com/docs/en/headless?x=1)).
- AC-10.2 Given arm `agent: codex`, When run dijalankan, Then pairbench memanggil `codex exec --json` dengan sandbox `workspace-write`, menjumlah token dari `turn.completed.usage`, dan biaya `computed` bila tarif model dikonfigurasi, selain itu `unknown` ([docs](https://developers.openai.com/codex/noninteractive)).
- AC-10.3 Given CLI agent tidak terpasang, When `plan` dijalankan, Then exit code non-zero dengan nama binary yang tidak ditemukan.

States: sama dengan US-04.

### US-11 Mode CI (Should)

Sebagai Thomas, aku ingin pairbench gagal di CI bila perubahan skill membuat hasil lebih buruk, supaya regresi tidak di-push ke User Skills.

Acceptance criteria:
- AC-11.1 Given `pairbench run --ci`, When verdict `worse`, Then exit code 2; `better` atau `inconclusive` → exit code 0; error konfigurasi → exit code 1.
- AC-11.2 Given contoh workflow GitHub Actions di `examples/ci/`, When PR mengubah file di `skills/`, Then workflow menjalankan eksperimen dengan `budget_usd` dari input workflow dan mengunggah laporan Markdown sebagai artifact.

States: error (AC-11.1 exit 1) / success (AC-11.1).

### US-12 Task dari commit (Should)

Sebagai Ben, aku ingin membuat task dari commit yang sudah merge, supaya task mewakili pekerjaan nyata tanpa menulis dari nol.

Acceptance criteria:
- AC-12.1 Given commit C yang mengubah kode dan test, When `pairbench task from-commit <repo> <C> --check "<cmd>"` dijalankan, Then terbentuk task dengan fixture = parent C tanpa perubahan test, `checks/` = file test dari C, oracle = diff non-test C, dan `instruction` berisi pesan commit yang bisa diedit.
- AC-12.2 Given task hasil from-commit yang check-nya lulus pada fixture, When `task check` dijalankan, Then gagal sesuai AC-01.2.

States: error (commit tidak ditemukan; AC-12.2) / success (AC-12.1).

## 6. Non-functional requirements

| Area | Requirement |
| --- | --- |
| Performance | CLI `plan`/`report` < 2 s untuk 1 000 run di orb 2 vCPU. Overhead harness p95 ≤ 5 s per run (M-5). Dashboard: LCP < 1.5 s di localhost untuk eksperimen 1 000 run (Chrome desktop, Lighthouse); JS awal ≤ 200 KB gzip. Pembaruan progres ≤ 5 s (AC-09.4). |
| Reliability | Setiap run ditulis durable sebelum run berikutnya dimulai; resume tanpa kehilangan atau duplikasi (US-07). Process tree agent selalu dihentikan sebelum check (AC-03.3). Status `infra_error` dibedakan dari `fail`. Tidak ada uptime target (tanpa hosting). |
| Security | Task dan fixture dianggap tepercaya; workspace sementara **bukan** batas keamanan, dinyatakan di README dan output `plan`. Check tidak bisa dibaca agent sampai agent berhenti (AC-03.2). Nilai variabel `env` tidak pernah disimpan; log menyamarkan nilai variabel yang diteruskan. Dashboard bind `127.0.0.1` secara default, tanpa auth (AC-09.5). Dashboard menampilkan log dan diff sebagai teks yang di-escape (tidak ada HTML mentah dari output agent; OWASP A03 injection/XSS). Dependency dipindai di CI (`pnpm audit` atau setara stack). |
| Accessibility | Dashboard WCAG 2.2 AA (AC-09.7); output CLI tidak bergantung pada warna saja (status juga ditulis sebagai kata). |
| Privacy | Inventaris data di bawah. Tidak ada telemetry ke luar. Semua data lokal. |
| Localization | English saja. Waktu ISO 8601 UTC. Biaya dalam USD dengan 4 desimal. |
| Portability | Linux x64 (orb) wajib; macOS arm64 Should. Tanpa Docker. |

Inventaris data (tidak ada data pribadi pihak ketiga yang dikumpulkan dengan sengaja; pasar global, UU PDP tidak berlaku karena tidak ada pengguna atau server di Indonesia yang memproses data orang lain):

| Data | Kenapa | Disimpan di | Retensi |
| --- | --- | --- | --- |
| Instruksi task, fixture, check, oracle | Mendefinisikan task | File di repo pengguna | Dikelola pengguna |
| Diff workspace, stdout/stderr agent dan check | Menjelaskan hasil run | Database lokal + direktori artefak lokal | Sampai `pairbench prune --older-than <d>` atau hapus manual |
| Thread/session ID agent, token, biaya | Biaya dan penelusuran | Database lokal | Sama |
| Nama variabel env (tanpa nilai) | Reproducibility | Database lokal | Sama |
| Nama git author dari `task from-commit` | Ikut dari commit | Tidak disimpan; hanya pesan commit | — |

## 7. External dependencies

| Dependency | Purpose | Access needed | Cost | Fallback |
| --- | --- | --- | --- | --- |
| Amp CLI (`amp -x --stream-json`, `amp skills list --json`, `amp threads usage`) | Agent utama dan provenance skill | Akun Amp, access token `AMP_API_KEY` ([docs](https://ampcode.com/docs/cli/execute-mode)) | Kredit Amp per run; **setiap eksperimen nyata butuh batas biaya yang disetujui Thomas** | Adapter command generik |
| Claude Code CLI (Should) | Pembanding | Akun Anthropic/langganan | Per token | Tidak dipakai |
| Codex CLI (Should) | Pembanding | Akun OpenAI/ChatGPT | Per token | Tidak dipakai |
| Git | Fixture, diff | Lokal | Gratis | — |
| Node/Rust/Go toolchain sesuai stack | Build | Lokal | Gratis | — |
| GitHub Actions (Should, US-11) | Mode CI | Repo public | Gratis untuk repo public (ASUMSI, dicek di Phase 5) | Jalankan manual |
| Hosting | — | — | US$0 | — |

## 8. Risks and assumptions

| Risk or assumption | Impact | Likelihood | Mitigation or validation |
| --- | --- | --- | --- |
| R1 Suite kecil membuat hampir semua verdict `inconclusive` | High | High | Laporan mencetak ukuran efek minimum yang bisa dideteksi dengan data sekarang; dokumentasi menyarankan task dengan pass rate baseline 20–80%; A/A kalibrasi (AC-02.3). Diterima: `inconclusive` adalah jawaban jujur |
| R2 Biaya eval besar (US$9–120 per perbandingan 30 run, [research.md](research.md#pricing)) | High | Medium | `budget_usd` wajib, `--pilot`, estimasi historis, `skipped_budget` (US-06); eksperimen nyata pertama dibatasi US$10 kecuali Thomas setuju lebih |
| R3 Format `--stream-json` Amp atau CLI lain berubah | Medium | High | Parser toleran tipe tidak dikenal (AC-04.5); contract test dengan rekaman output nyata; versi CLI agent disimpan per run |
| R4 Skill overlay tertutup skill lokal dengan nama sama | High | Medium | Provenance `amp skills list --json` dan penolakan run (AC-05.1–05.2). **Validasi di spike Phase 5:** apakah `amp skills list --json` menampilkan path sumber |
| R5 Baseline "tanpa skill" tidak bisa dibuat bila skill ada di personal skills repository | Medium | Medium | MVP mendukung "skill v1 vs v2" (overlay dua-duanya), bukan "tanpa skill"; opsi menonaktifkan skill dicek di spike |
| R6 Agent masih bisa melihat check lewat jalur lain (misalnya direktori task di path absolut, riwayat git) | High | Low | Workspace di direktori temp di luar repo task; fixture disalin tanpa `.git` task; check disalin setelah agent berhenti; e2e AC-03.2 dengan agent penyerang |
| R7 Test lulus ≠ kode bagus ([HN 47295537](https://news.ycombinator.com/item?id=47295537)) | Medium | High | Detail run menampilkan diff untuk dibaca; skor kualitas = Could; dinyatakan di laporan |
| R8 Proses anak agent lolos dari kill (daemon, `setsid`) | Medium | Medium | Process group + kill pohon PID, cek tidak ada PID tersisa (AC-03.3); di Linux cgroup bila tersedia (diputuskan di ADR) |
| R9 Biaya Amp tidak tersedia di stream-json | Medium | High | `cost_source`; ambil dari `amp threads usage <thread>` setelah run; spike Phase 5 memeriksa format output-nya |
| R10 Build ulang harness yang sudah ada (Harbor) daripada adopsi | Medium | Medium | Opsi build vs adopt dibawa ke gate Phase 4; MVP fokus di celah (Amp, tanpa Docker, provenance skill, statistik berpasangan) |
| R11 Run paralel saling mengganggu (port, cache, rate limit) | Low | Medium | `max_parallel` default 1 |
| A1 Agent bisa dijalankan non-interaktif dengan output mesin | — | — | Terverifikasi dari docs ([research.md](research.md#bukti-tambahan-yang-memengaruhi-desain)) |
| A2 Operator menulis task tepercaya | — | — | Dinyatakan di README dan output `plan` |

## 9. Milestones

| Milestone | Contents | Exit criteria |
| --- | --- | --- |
| M0 Foundations and contracts | Schema `task.yaml` dan `experiment.yaml` (JSON Schema), schema database run, kontrak JSON adapter v1, spesifikasi metode statistik + script referensi, CI lint/typecheck/test, design tokens dashboard | Schema dan kontrak ter-commit di `contracts/`; CI hijau; script referensi menghasilkan nilai harapan AC-08 |
| M1 Core loop | US-01, US-02, US-03, US-04 (Amp + generik), US-07, US-08 (CLI + Markdown) | e2e AC-01–04, 07, 08 hijau dengan agent uji deterministik; satu run Amp sungguhan berbiaya ≤ US$1 lulus (dengan persetujuan Thomas) |
| M2 Supporting features | US-05, US-06, US-09 | e2e AC-05, 06, 09 hijau; axe-core bersih |
| M3 Polish and launch | US-10, US-11, US-12 bila waktu cukup; README, contoh suite 3 task dari repo factory; eksperimen nyata pertama (misalnya skill `prd-writer` saat ini vs varian) dengan batas US$10 | Semua Must hijau; laporan eksperimen nyata pertama di `docs/`; rilis GitHub v0.1.0 |

## 10. Budget

- Time: ~3 minggu kerja agent. Estimasi kasar per story (hari agent): US-01 1.5, US-02 1, US-03 2.5, US-04 2, US-05 1, US-06 1, US-07 1.5, US-08 2, US-09 3 → Must ≈ 15.5 hari. Should US-10 1, US-11 0.5, US-12 1.5 → 3 hari. M0 + polish ≈ 2 hari.
- Money: hosting US$0/bulan (lokal). Biaya pengembangan: kredit Amp thread build (dicatat di progress.md). Biaya eval nyata: hanya dengan persetujuan Thomas; usulan awal US$1 untuk M1 dan US$10 untuk eksperimen M3.
