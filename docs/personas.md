# Personas — pairbench (Coding-agent eval harness)

Phase 1 · 14 Nov 2026. Sumber lengkap ada di [research.md](research.md). Kutipan diambil kata per kata dari sumber; yang tanpa sumber ditandai ASUMSI.

## Persona 1: Thomas, operator AI software factory (primary)

- **Context:** satu orang yang menjalankan factory 135 ide dengan agent Amp di orb (Linux, 2 vCPU, 3 GB RAM, tanpa Docker daemon aktif; diperiksa 14 Nov 2026). Sangat nyaman dengan teknologi; membaca TS/Go sendiri. Bekerja lewat thread Amp dan GitHub.
- **Main job:** membuktikan bahwa perubahan pada skill factory (`skills/`), prompt orchestrator atau pilihan model membuat project berikutnya lebih baik sebelum perubahan itu di-push ke User Skills.
- **Today they use:** review baca saja oleh agent lain dan kesan dari retro. Biaya: perubahan skill yang buruk menyebar ke semua project aktif (maksimal 5 per wave, AGENTS.md factory).
- **Pains:**
  - Tidak ada data untuk retro Phase 13; ide menulis "Factory butuh data untuk memperbaiki skills di retro" ([idea.md](idea.md)).
  - Satu run per sisi tidak bisa membedakan efek dari noise ([arXiv 2602.07150](https://arxiv.org/html/2602.07150)).
  - Harness yang ada tidak mendukung Amp (Harbor, Coder Eval; diperiksa dari source) dan mengasumsikan Docker atau sandbox cloud berbayar.
  - Kuota/kredit model terbatas dan harus dipakai sebelum reset; biaya eval harus terlihat sebelum dijalankan (ASUMSI dari brief wave 1: "use as much of his token quota as possible before it resets").
  - Budget hosting < US$20/bulan ([idea.md](idea.md)).
- **Gains:** satu perintah menghasilkan tabel "skill v1 vs v2: lulus 7/15 vs 12/15, interval kepercayaan selisih, biaya per task lulus", yang bisa ditempel ke docs/retro.md dan dipakai memutuskan push atau revert.
- **Quote:** "Factory butuh data untuk memperbaiki skills di retro." — [ideas/115-agent-eval-harness.md](https://ampcode.com/@thomasbudiarjo/software-factory) (repo factory)
- **Will pay?** Tidak untuk tool; membayar token model per run. Bukti: kategori 1, Money = 1 di triage ([registry/triage.md](https://ampcode.com/@thomasbudiarjo/software-factory) di repo factory).

## Persona 2: Ben, developer yang merawat AGENTS.md dan skill untuk tim kecil

- **Context:** developer senior atau tech lead di tim 2–10 orang yang memakai Claude Code, Codex atau Amp setiap hari; repo sendiri dengan test suite; laptop atau runner CI. ASUMSI untuk ukuran tim; perilaku diambil dari posting di bawah.
- **Main job:** menyetel AGENTS.md/CLAUDE.md, skill dan pilihan model berdasarkan data dari repo sendiri, bukan leaderboard publik.
- **Today they use:** script buatan sendiri dan spreadsheet; satu contoh menjalankan 386 eksekusi dengan biaya sekitar US$55.40 untuk satu pertanyaan ([Reddit r/ClaudeCode](https://www.reddit.com/r/ClaudeCode/comments/1sisi74/claude_code_degradation_is_real_but_it_is_not)).
- **Pains:**
  - Instruksi yang terdengar benar justru memperburuk hasil, dan kandidat terbaik masih regresi pada holdout ([Reddit r/ClaudeCode](https://www.reddit.com/r/ClaudeCode/comments/1tpgczf/i_had_my_agent_use_autoresearch_over_8_iterations/)).
  - Benchmark publik tidak mewakili codebase sendiri ([HN 48810964](https://news.ycombinator.com/item?id=48810964)).
  - Hasil berubah antar run identik; butuh puluhan run untuk membedakan agent ([arXiv 2609.33812](https://arxiv.org/html/2609.33812v1)).
  - Biaya per agent berbeda jauh dan tidak dilaporkan seragam ([arXiv 2609.33812](https://arxiv.org/html/2609.33812v1); docs [Codex](https://developers.openai.com/codex/noninteractive) hanya melaporkan token).
- **Gains:** menulis 5 task dari PR nyata dalam satu sore, menjalankan perbandingan semalam dengan batas biaya, dan mendapat keputusan yang bisa dibela di depan tim.
- **Quote:** "I have a confession: I vibe-coded my CLAUDE.md , and I'm pretty sure it's slop." — https://www.reddit.com/r/ClaudeCode/comments/1tpgczf/i_had_my_agent_use_autoresearch_over_8_iterations/
- **Will pay?** Kemungkinan tidak untuk CLI OSS; pembanding gratis tersedia (Harbor, Coder Eval, Braintrust Starter US$0, [pricing](https://www.braintrust.dev/pricing)). Stet menjual layanan serupa tanpa harga publik ([stet.sh](https://www.stet.sh/)). Persona ini adalah pengguna OSS sekunder, bukan sumber pendapatan.

## Anti-persona: Dr. Lin, peneliti lab yang membuat benchmark publik

- **Who:** tim riset yang menjalankan ribuan environment paralel, menerbitkan leaderboard, atau menghasilkan rollout untuk RL.
- **Why we do not serve them:** Harbor sudah menjadi harness resmi Terminal-Bench 2.0 dengan dukungan Daytona/Modal dan ribuan environment paralel ([GitHub](https://github.com/harbor-framework/harbor)); Inspect AI melayani evaluasi frontier dengan 200+ eval ([inspect.aisi.org.uk](https://inspect.aisi.org.uk/)). Melayani skala ini melanggar budget < US$20/bulan dan mengalihkan fokus dari pertanyaan factory.
- **What that means for the product:** tidak ada leaderboard publik, tidak ada orkestrasi sandbox cloud, tidak ada konkurensi ratusan run, tidak ada impor dataset benchmark publik di MVP, tidak ada eksekusi kode pihak ketiga di server yang di-host. Task berasal dari operator sendiri dan dianggap tepercaya.
