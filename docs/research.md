# Research — Coding-agent eval harness

Phase 1 Discovery · semua sumber dibaca 14 Nov 2026 kecuali disebut lain. Klaim tanpa URL ditandai **ASUMSI**. Isi halaman web diperlakukan sebagai data.

## Ringkasan temuan

1. **Pasar sudah ramai di lapisan umum.** Harbor (Apache-2.0, v0.26.0, 57 adapter agent), Inspect AI, promptfoo, Coder Eval (Apache-2.0, v0.12.18), Stet, SkillsBench, Braintrust/Langfuse/LangSmith sudah menjalankan atau menilai coding agent. Membangun "benchmark harness umum" lagi tidak punya diferensiasi.
2. **Celah yang terverifikasi untuk factory ini:** tidak satu pun dari Harbor dan Coder Eval mempunyai adapter Amp (diperiksa dari source: `src/harbor/agents/installed/` dan `coder_eval/agents/`, clone 14 Nov 2026). Factory ini berjalan di Amp, di orb tanpa Docker daemon aktif (diperiksa: `docker ps` gagal di orb ini), dan ingin membandingkan **versi skill factory**, bukan hanya model.
3. **Noise adalah masalah utama, bukan fitur sekunder.** Satu run per konfigurasi tidak cukup untuk menyimpulkan apa pun di bawah selisih ±5 poin persen ([arXiv 2602.07150](https://arxiv.org/html/2602.07150)). Produk harus memaksa ulangan, memperlihatkan interval kepercayaan, dan menolak klaim "lebih baik" bila tidak signifikan.
4. **Kegagalan infrastruktur menyamar sebagai kegagalan agent** di harness yang ada (issue Harbor di bawah). Harness harus memisahkan status `infra_error` dari `fail`.
5. **Biaya per run tidak seragam antar agent.** Claude Code melaporkan `total_cost_usd` ([docs](https://code.claude.com/docs/en/headless?x=1)); Codex melaporkan token tanpa biaya ([docs](https://developers.openai.com/codex/noninteractive)); `amp --stream-json` melaporkan token per pesan tanpa biaya ([docs](http://ampcode.com/docs/cli/streaming-json)), tetapi `amp threads usage <thread>` menampilkan biaya per thread (diperiksa dari `--help` CLI versi 0.0.1794628846).

## Variations

Skor 1–5. Pain = seberapa sakit untuk factory/pengguna; Reach = berapa banyak pengguna; Feasibility = MVP 2–4 minggu kerja agent dengan budget < US$20/bulan.

| # | Variation | Pain | Reach | Feasibility | Note |
| --- | --- | --- | --- | --- | --- |
| V1 | Benchmark harness umum (task suite publik, leaderboard, banyak agent) | 2 | 4 | 2 | Bersaing langsung dengan Harbor/Terminal-Bench; tidak ada diferensiasi |
| V2 | **Paired eval CLI untuk factory: baseline vs kandidat (agent/model/versi skill) pada suite task kecil dari repo factory, N ulangan, statistik + dashboard lokal; Amp first, adapter command generik** | 5 | 2 | 4 | Menjawab langsung "apakah perubahan skill ini membantu?" di retro |
| V3 | Adapter Amp untuk Harbor + dashboard factory di atas job directory Harbor | 4 | 3 | 3 | Memakai ulang Harbor; butuh Docker/cloud sandbox (Daytona/Modal berbayar) dan Python; Docker daemon tidak aktif di orb |
| V4 | SaaS hosted "replay PR Anda" (seperti Stet) | 3 | 3 | 1 | Butuh eksekusi kode pihak ketiga di server; melanggar budget dan batas keamanan |
| V5 | GitHub Action: gate CI bila skill/AGENTS.md berubah | 4 | 3 | 3 | Bagus sebagai Should; tiap run memakan kredit model di CI |
| V6 | Linter statis untuk SKILL.md/AGENTS.md (tanpa menjalankan agent) | 2 | 4 | 5 | Murah tapi tidak mengukur perilaku; penelitian menunjukkan efek instruksi baru terlihat saat dijalankan |
| V7 | Pengukur "skill dipanggil atau tidak" (activation test) saja | 3 | 3 | 4 | Coder Eval sudah punya `skill_triggered` |
| V8 | Analitik pasif atas thread Amp nyata (biaya, durasi, jumlah intervensi) tanpa task suite | 3 | 3 | 4 | Tidak bisa membandingkan pada task yang sama; berguna sebagai Could |
| V9 | LLM-as-judge review kualitas diff tanpa test | 2 | 3 | 4 | Tidak objektif; tidak memenuhi "cara objektif" di ide |
| V10 | Plugin Amp yang menjalankan eval dari thread | 3 | 2 | 3 | API plugin belum diperiksa untuk eksekusi panjang; ASUMSI |
| V11 | Template repo "eval suite" untuk dijual ke tim (OSS + konsultasi) | 2 | 2 | 4 | Tidak menyelesaikan masalah factory sekarang |
| V12 | Eval berbasis acceptance criteria PRD factory (task = AC + e2e test project) | 4 | 1 | 3 | Task sangat relevan tetapi butuh project yang sudah punya e2e; masuk V2 sebagai sumber task fase berikut |

**Chosen: V2** — paired eval CLI untuk factory, dengan V12 sebagai sumber task setelah project wave 1 punya e2e dan V5 sebagai Should. Alasannya: (1) masalah inti di ide adalah membandingkan konfigurasi pada task yang sama, dan pertanyaan retro factory selalu berbentuk pasangan "sebelum vs sesudah perubahan skill"; (2) penelitian noise menunjukkan evaluasi berpasangan dengan ulangan dan interval kepercayaan adalah cara minimum yang jujur; (3) harness yang ada tidak mendukung Amp; Harbor mengasumsikan Docker atau sandbox cloud berbayar, sedangkan factory berjalan di orb Amp yang sendiri sudah disposable; (4) V3 tetap dicatat sebagai alternatif build-vs-adopt untuk Phase 4. **Koreksi plan review ronde 1:** kandidat adopsi yang realistis adalah **Coder Eval**, bukan Harbor, karena driver default Coder Eval adalah `tempdir` di host tanpa Docker (diperiksa di source `src/coder_eval/models/sandbox.py`, `default="tempdir"`, 14 Nov 2026). Bila Thomas memilih adopsi, sebagian besar M1 diganti oleh plugin agent Amp untuk Coder Eval.

## Jobs-to-be-Done

- **Main job:** "Saat aku mengubah skill, prompt, model atau agent di factory, aku ingin tahu dengan bukti apakah hasilnya lebih baik, sama, atau lebih buruk pada pekerjaan factory yang nyata, supaya aku hanya mem-push perubahan yang membantu."
- **Supporting jobs:**
  - Memilih model/agent termurah yang masih lulus task kelas tertentu (biaya per task lulus).
  - Mendeteksi regresi saat vendor merilis model baru atau CLI agent ter-update (CLI Amp di orb ini di-release 1 jam sebelum dibaca).
  - Memberi retro Phase 13 tabel angka yang bisa dikutip, bukan kesan.
  - Menyimpan riwayat agar perbandingan minggu ini dan bulan lalu bisa dibuat tanpa menyimpan folder manual (keluhan Harbor job directory, [Braintrust blog](https://www.braintrust.dev/blog/harbor-agent-evals)).
- **Solusi yang dipakai hari ini:**
  - Tidak melakukan apa-apa: perubahan skill factory di `skills/` di-push berdasarkan review baca saja (repo factory, commit `59ac144`).
  - Script buatan sendiri dan spreadsheet: contoh 386 eksekusi dan US$55.40 untuk satu pertanyaan ([Reddit r/ClaudeCode](https://www.reddit.com/r/ClaudeCode/comments/1sisi74/claude_code_degradation_is_real_but_it_is_not)).
  - Harbor/Terminal-Bench untuk benchmark publik ([GitHub](https://github.com/harbor-framework/harbor)).
  - Membaca leaderboard publik yang tidak mewakili codebase sendiri ([HN 48810964](https://news.ycombinator.com/item?id=48810964)).

## Journey map

| Tahap | Aksi | Pikiran | Pain | Di mana app masuk |
| --- | --- | --- | --- | --- |
| 1 Sadar | Retro project menunjukkan agent mengulang kesalahan; Thomas mengedit skill | "Apakah edit ini benar-benar membantu?" | Tidak ada cara mengukur; review baca saja | — |
| 2 Coba manual | Menjalankan `amp -x` dua kali dengan skill lama dan baru | "Hasilnya beda, tapi apakah karena skill atau acak?" | Satu run per sisi = noise ([arXiv 2602.07150](https://arxiv.org/html/2602.07150)) | `pairbench run` dengan N ulangan per sisi |
| 3 Siapkan task | Menulis instruksi + repo awal + command pemeriksa | "Berapa lama menulis satu task?" | Harness lain butuh Dockerfile/verifier per task ([Harbor #2549](https://github.com/harbor-framework/harbor/issues/2549)) | Format task 1 file + fixture repo; `pairbench task check` menjalankan pemeriksa tanpa agent |
| 4 Jalankan | Menjalankan suite untuk 2 konfigurasi | "Berapa biayanya? Kapan selesai?" | Biaya tidak terlihat sampai tagihan | Estimasi dan batas biaya sebelum run; progres live |
| 5 Baca hasil | Membuka dashboard | "Mana yang menang, dan seberapa yakin?" | Angka tanpa interval kepercayaan menyesatkan | Tabel berpasangan + interval + verdict "belum cukup bukti" |
| 6 Putuskan | Push skill atau revert | "Bisakah aku kutip ini di retro?" | Bukti tersebar di thread | Laporan Markdown untuk docs/retro.md |
| 7 Rutin | Menjalankan ulang tiap perubahan skill atau rilis model | "Apakah masih benar minggu ini?" | Lupa menjalankan | Should: GitHub Action pada PR yang mengubah `skills/` |

## Competitors

| Product | Type | Price | Target | Strengths | Weaknesses | Source |
| --- | --- | --- | --- | --- | --- | --- |
| Harbor | OSS framework (Python, Apache-2.0) | Gratis; sandbox cloud dibayar terpisah | Lab, pembuat benchmark, RL | Harness resmi Terminal-Bench 2.0; 57 adapter agent; skala ribuan environment | Tidak ada adapter Amp; butuh Docker/cloud; issue setup dan verifier di bawah | [GitHub](https://github.com/harbor-framework/harbor), source clone 14 Nov 2026 |
| Inspect AI (UK AISI) | OSS framework (Python) | Gratis | Peneliti evaluasi frontier | 200+ eval siap pakai, Inspect View, sandbox Docker/K8s/Modal, bisa menjalankan Claude Code/Codex/Gemini CLI | Berorientasi riset model; tidak menyebut Amp | [inspect.aisi.org.uk](https://inspect.aisi.org.uk/) |
| promptfoo (coding agents) | OSS CLI + web viewer (TS) | Gratis OSS; versi enterprise | Tim aplikasi LLM | Provider Codex SDK, Claude Agent SDK, OpenCode; YAML; viewer | Fokus SDK; tidak menyebut Amp; issue UI tidak menampilkan prompt dari config kustom ([#2471](https://github.com/promptfoo/promptfoo/issues/2471)) | [Guide](https://www.promptfoo.dev/docs/guides/evaluate-coding-agents/) |
| Coder Eval (UiPath) | OSS CLI (Python, Apache-2.0) | Gratis | Pembuat CLI dan skill | YAML task, skor 0–1 berbobot, A/B, `skill_triggered`, biaya per tool call, evalboard, CI gate; driver default `tempdir` tanpa Docker (source `models/sandbox.py`); `paired_comparison` dua varian dengan task sebagai unit (`reports/helpers.py`), `--resume`, jendela anti-cheat chmod 000 (`fs_permissions.py`), commit `d8011f0` | Agent: Claude Code, Codex, Antigravity, OpenCode, pi; tidak ada Amp; uji berpasangan per task punya power sangat rendah pada 3–5 task; tanpa provenance skill Amp, scrub env, atau penolakan tool/MCP | [Docs](https://coder-eval.com/docs), [perbandingan](https://coder-eval.com/docs/comparison) |
| Stet | SaaS/produk komersial | Tidak dipublikasikan di halaman utama (ASUMSI: berbayar) | Tim yang mengubah AGENTS.md/model | Task dari PR yang sudah merge, skor kualitas selain test, keputusan PROMOTE/HOLD | Kode repo diproses produk pihak ketiga; harga tidak transparan | [stet.sh](https://www.stet.sh/), [blog](https://www.stet.sh/blog/how-i-used-codex-to-improve-its-own-agents-md) |
| SWE-bench harness | OSS benchmark | Gratis | Peneliti | Standar industri, Docker, pass/fail test | Dataset tetap; tidak menjalankan agent sendiri; rawan harness khusus benchmark ([HN 44874831](https://news.ycombinator.com/item?id=44874831)) | [Guide](https://www.swebench.com/SWE-bench/guides/evaluation) |
| SkillsBench (BenchFlow) | Benchmark + leaderboard | Gratis | Peneliti skill | Mengukur efek skill berpasangan; 87 task | Task tetap di 8 domain, bukan skill milik pengguna | [Paper](https://arxiv.org/html/2602.12670v4), [blog](https://www.skillsbench.ai/blogs/skillsbench-1-1) |
| Braintrust | SaaS eval/observability | Starter US$0 (1 GB, 10k skor, 14 hari); Pro US$249/bln | Tim AI | Eksperimen, plugin Harbor, trace | Tidak menjalankan agent; berbayar saat skala | [Pricing](https://www.braintrust.dev/pricing), [blog](https://www.braintrust.dev/blog/harbor-agent-evals) |
| Langfuse | OSS + cloud | Hobby gratis; Core US$29/bln | Tim AI | MIT self-host, eksperimen vs baseline | Self-host butuh Postgres+ClickHouse+Redis+S3; tidak menjalankan agent | [Pricing](https://langfuse.com/pricing), [Index Agentica](https://indexagentica.com/compare/agent-observability-platforms) |
| LangSmith | SaaS | Developer gratis 1 seat; Plus US$39/seat/bln | Tim LangChain | Eval online/offline | Per seat dan per trace; tidak menjalankan coding agent | [Pricing](https://www.langchain.com/pricing) |
| Script buatan sendiri + spreadsheet | Non-app | Waktu + token | Developer individual | Bebas, pas dengan repo sendiri | Tidak reproducible, statistik manual, mahal waktu | [Reddit r/ClaudeCode](https://www.reddit.com/r/ClaudeCode/comments/1sisi74/claude_code_degradation_is_real_but_it_is_not), [Reddit r/ClaudeAI](https://www.reddit.com/r/ClaudeAI/comments/1tbkpa8/a_practical_claude_code_vs_codex_experiment_6/) |
| Tidak melakukan apa-apa (review baca) | Non-app | 0 | Factory saat ini | Murah | Perubahan instruksi yang terdengar benar bisa memperburuk hasil ([Stet blog](https://www.stet.sh/blog/how-i-used-codex-to-improve-its-own-agents-md)) | Repo factory `skills/` |

## Complaint themes

| Theme | Count | Example quote | Source |
| --- | --- | --- | --- |
| Benchmark publik tidak mewakili pekerjaan sendiri | 3 | "I think coding benchmark results don't represent our messy reality." | [HN 48810964](https://news.ycombinator.com/item?id=48810964), [HN 44874831](https://news.ycombinator.com/item?id=44874831), [HN 49676820](https://news.ycombinator.com/item?id=49676820) |
| Noise antar run menenggelamkan selisih | 5 | "reported improvements of 2–3 percentage points may reflect evaluation noise rather than genuine algorithmic progress." | [arXiv 2602.07150](https://arxiv.org/html/2602.07150), [arXiv 2609.33812](https://arxiv.org/html/2609.33812v1), [Reddit degradation](https://www.reddit.com/r/ClaudeCode/comments/1sisi74/claude_code_degradation_is_real_but_it_is_not), [Reddit CLAUDE.md holdout](https://www.reddit.com/r/ClaudeCode/comments/1tpgczf/i_had_my_agent_use_autoresearch_over_8_iterations/), [Reddit LLMDevs](https://www.reddit.com/r/LLMDevs/comments/1vgdwfz/i_benchmarked_5_codebase_tools_across_claude_code/) |
| Kegagalan infrastruktur tercatat sebagai kegagalan agent | 4 | "Thus the missing scores were setup failures, not evaluations of either skill's behavior." | [Harbor #3450](https://github.com/harbor-framework/harbor/issues/3450), [#2549](https://github.com/harbor-framework/harbor/issues/2549), [#583](https://github.com/harbor-framework/harbor/issues/583), [#2250](https://github.com/harbor-framework/harbor/issues/2250) |
| Integritas penilaian bocor (agent membaca grader, tetap jalan saat verifikasi, curang) | 3 | "an agent can cat /tests/test.sh to read the previous step's grading script" | [Harbor #1960](https://github.com/harbor-framework/harbor/issues/1960), [#3866](https://github.com/harbor-framework/harbor/issues/3866), [arXiv 2609.33812](https://arxiv.org/html/2609.33812v1) |
| Test lulus bukan berarti kode bagus | 2 | "Passing tests is a weak signal and proving a fix is truly safe to merge is much harder" | [HN 47295537](https://news.ycombinator.com/item?id=47295537), [stet.sh](https://www.stet.sh/) |
| Perubahan instruksi/skill diputuskan berdasarkan kesan | 3 | "I have a confession: I vibe-coded my CLAUDE.md , and I'm pretty sure it's slop." | [Reddit CLAUDE.md](https://www.reddit.com/r/ClaudeCode/comments/1tpgczf/i_had_my_agent_use_autoresearch_over_8_iterations/), [HN 47034087](https://news.ycombinator.com/item?id=47034087), [SkillsBench](https://www.skillsbench.ai/blogs/skillsbench-1-1) |
| Biaya tidak seragam atau tidak terlihat | 3 | "the same job cost $0.08 a run through pi and $1.86 through Claude Code on one endpoint" | [arXiv 2609.33812](https://arxiv.org/html/2609.33812v1), [Claude Code docs](https://code.claude.com/docs/en/headless?x=1) ("client-side estimates"), [Amp streaming JSON](http://ampcode.com/docs/cli/streaming-json) (tanpa field biaya) |

Implikasi produk dari tema (masuk PRD):

- Ulangan wajib dan interval kepercayaan; verdict "inconclusive" adalah hasil yang sah.
- Status run dibedakan: `pass`, `fail`, `infra_error`, `timeout`, `cancelled`; hanya `pass`/`fail` masuk statistik, sisanya dilaporkan terpisah.
- Pemeriksa task disalin ke workspace hanya setelah agent selesai dan prosesnya dihentikan; agent tidak bisa membaca pemeriksa.
- Biaya dicatat dengan sumbernya (`reported` dari agent, `computed` dari token × tarif, `unknown`).

## Bukti tambahan yang memengaruhi desain

- Jumlah run yang dibutuhkan: untuk mendeteksi selisih 5 pp dengan σ 1.5 pp butuh 2 run per agent (p<0.05, power 80%); selisih 2 pp butuh 9 run ([arXiv 2602.07150, Tabel 2](https://arxiv.org/html/2602.07150)). Itu di tingkat benchmark 500 task; untuk suite factory kecil (≤10 task) variansnya lebih besar. **ASUMSI:** default 3 ulangan per task per konfigurasi, dan dashboard menyatakan ukuran efek minimum yang bisa dideteksi.
- Skill kurasi menaikkan rata-rata pass rate 33.9% → 50.5%, tetapi skill buatan agent sendiri menurunkannya (−8.1 sampai −11.5 pp) dan bundel 4+ skill memberi lift lebih kecil ([SkillsBench 1.1](https://www.skillsbench.ai/blogs/skillsbench-1-1)). Ini mendukung pertanyaan factory: perubahan skill memang bisa membantu atau merugikan, dan harus diukur.
- Agent CLI non-interaktif tersedia untuk ketiga agent target: `amp -x --stream-json` ([docs](http://ampcode.com/docs/cli/streaming-json)), `claude -p --output-format json` ([docs](https://code.claude.com/docs/en/headless?x=1)), `codex exec --json` ([docs](https://developers.openai.com/codex/noninteractive)). Amp di skrip butuh `AMP_API_KEY` access token ([execute mode](https://ampcode.com/docs/cli/execute-mode)).
- Amp `-ox` menjalankan prompt di orb baru; orb adalah lingkungan disposable yang bisa menjadi sandbox untuk run Amp ([execute mode](https://ampcode.com/docs/cli/execute-mode)). Mengambil diff dari orb tersebut belum diperiksa: **ASUMSI**, divalidasi di spike Phase 5.
- Diperiksa di orb 14 Nov 2026 (plan review ronde 1): `amp skill list --json` mengembalikan `name`, `source`, `baseDir`; skill di `.agents/skills/` workspace (`source: workspace-agents`) mengalahkan skill `global-user` dengan nama sama; HOME sementara **tidak** menyembunyikan skill `global-user` dan plugin (diunduh ulang dari akun). `amp threads usage <thread>` hanya mencetak teks (`Cost: $0.04`, tanpa `--json`).

## Positioning

Untuk operator factory agent yang mengubah skill, prompt atau model setiap minggu, **pairbench** adalah CLI eval berpasangan yang menjalankan konfigurasi lama dan baru pada task dari repo sendiri, dengan ulangan dan interval kepercayaan, tidak seperti Harbor atau leaderboard publik yang mengukur agent di benchmark tetap tanpa dukungan Amp.

## Pricing

- **Delivery: OSS** (kategori 1, repo public). Tidak ada hosting wajib; dashboard berjalan lokal.
- Harga: gratis, lisensi Apache-2.0 (sama dengan Harbor dan Coder Eval). Pembanding: Braintrust Starter US$0, Langfuse Core US$29/bln, LangSmith Plus US$39/seat/bln, Braintrust Pro US$249/bln (sumber di tabel kompetitor).
- Biaya nyata pengguna = token/kredit model per run. Contoh dari pihak ketiga: US$0.21–0.32 per run Claude Code pada task kecil ([Reddit](https://www.reddit.com/r/ClaudeCode/comments/1sisi74/claude_code_degradation_is_real_but_it_is_not)); US$3.56–4.55 per task pada PR nyata ([stet.sh](https://www.stet.sh/)). **ASUMSI** untuk factory: 5 task × 3 ulangan × 2 konfigurasi = 30 run; pada US$0.30–4.00 per run, satu perbandingan berbiaya US$9–120. Karena itu batas biaya per suite wajib ada di MVP.
- Monetisasi di luar factory (Could, bukan MVP): template suite dan konsultasi. Skor triage Money = 1 tetap berlaku.

## Names and domains

Dicek dengan RDAP (`https://rdap.org/domain/<name>`) pada 14 Nov 2026; 404 biasanya berarti belum terdaftar, 200 berarti terdaftar. Hanya dicek, tidak dibeli.

| Name | .com | .dev | Note |
| --- | --- | --- | --- |
| **pairbench** | terdaftar (200) | kemungkinan tersedia (404) | Pilihan utama: menggambarkan perbandingan berpasangan; nama CLI `pairbench` |
| twinrun | terdaftar (200) | kemungkinan tersedia (404) | Alternatif pendek |
| trialpair | terdaftar (200) | kemungkinan tersedia (404) | Alternatif |
| retrobench | terdaftar | terdaftar | Gugur |
| skilldiff | terdaftar | terdaftar | Gugur |
| ablate | terdaftar | terdaftar | Gugur |
| sparring | — | terdaftar (.dev) | Gugur |

Kolom .id tidak relevan: pasar global berbahasa Inggris. Domain tidak dibutuhkan untuk MVP (OSS di GitHub); pembelian tetap butuh persetujuan Thomas.

## Sources

Semua dibaca 14 Nov 2026.

- Harbor: https://github.com/harbor-framework/harbor ; issue #583, #1960, #2250, #2549, #3450, #3866 di https://github.com/harbor-framework/harbor/issues/ ; Braintrust plugin: https://www.braintrust.dev/blog/harbor-agent-evals
- Terminal-Bench: https://github.com/harbor-framework/terminal-bench ; harness-bench: https://github.com/ajrrac/harness-bench
- Inspect AI: https://inspect.aisi.org.uk/
- promptfoo: https://www.promptfoo.dev/docs/guides/evaluate-coding-agents/ ; issue https://github.com/promptfoo/promptfoo/issues/2471
- Coder Eval: https://coder-eval.com/docs ; https://coder-eval.com/docs/comparison ; source https://github.com/UiPath/coder_eval
- Stet: https://www.stet.sh/ ; https://www.stet.sh/blog/how-i-used-codex-to-improve-its-own-agents-md
- SWE-bench: https://www.swebench.com/SWE-bench/guides/evaluation
- SkillsBench: https://arxiv.org/html/2602.12670v4 ; https://www.skillsbench.ai/blogs/skillsbench-1-1
- Noise: https://arxiv.org/html/2602.07150 ; https://arxiv.org/html/2609.33812v1 ; https://arxiv.org/html/2608.14711
- Pricing: https://www.braintrust.dev/pricing ; https://langfuse.com/pricing ; https://www.langchain.com/pricing ; https://laminar.sh/article/llm-observability-pricing
- Agent CLI: http://ampcode.com/docs/cli/streaming-json ; https://ampcode.com/docs/cli/execute-mode ; https://code.claude.com/docs/en/headless?x=1 ; https://developers.openai.com/codex/noninteractive
- Diskusi: https://news.ycombinator.com/item?id=48810964 ; https://news.ycombinator.com/item?id=49676820 ; https://news.ycombinator.com/item?id=47034087 ; https://news.ycombinator.com/item?id=44874831 ; https://news.ycombinator.com/item?id=47295537 ; https://www.reddit.com/r/ClaudeCode/comments/1sisi74/claude_code_degradation_is_real_but_it_is_not ; https://www.reddit.com/r/ClaudeCode/comments/1tpgczf/i_had_my_agent_use_autoresearch_over_8_iterations/ ; https://www.reddit.com/r/ClaudeAI/comments/1tbkpa8/a_practical_claude_code_vs_codex_experiment_6/ ; https://www.reddit.com/r/LLMDevs/comments/1vgdwfz/i_benchmarked_5_codebase_tools_across_claude_code/
- Pemeriksaan lokal di orb: `amp --help`, `amp threads usage --help`, `docker ps`, clone source Harbor dan Coder Eval.
