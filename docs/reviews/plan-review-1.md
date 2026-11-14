# Plan review — round 1

14 Nov 2026. Dua subagent dengan konteks segar berjalan paralel: CRITIC (staff PM skeptis) dan DISCUSSANT (strategist konstruktif). Keduanya model yang sama dengan penulis PRD; tidak ada model berbeda yang bisa dipakai karena thread ini tidak boleh membuat thread lain (instruksi wave 1). Konsolidasi oleh agent penulis. Klaim faktual reviewer yang bisa diperiksa tanpa biaya model diperiksa di orb sebelum diputuskan (kolom Reason).

Severity: B = BLOCKER, M = MAJOR, m = MINOR, D = usulan DISCUSSANT.

| # | Source | Severity | Finding | Decision | Reason | Applied in |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | CRITIC | B | Bootstrap dua tingkat bisa memberi `better` dari 2 run (interval [1,1]); 3 task tidak cukup untuk generalisasi; coverage tidak diketahui | ACCEPT | Benar: sel semua-lulus/semua-gagal membuat interval nol-lebar. Diganti permutation test berstrata (exact, FPR ≤ α), inferensi bersyarat suite, verdict `insufficient_data`, `repeats ≥ 2`, studi kalibrasi A/A di CI | §1 G1, §2 M-8, US-02 AC-02.6, US-08 |
| 2 | CRITIC | B | "Agent tidak bisa membaca check" tidak bisa dijamin tanpa sandbox; `checks/` ada di direktori task; tampering lewat `conftest.py`/`package.json` tidak ditangani | ACCEPT | Klaim dipersempit dan model ancaman ditulis; check di data-dir, dijalankan dari salinan pairbench, `protected_paths` dipulihkan, deteksi integritas; R6 naik ke Medium/High; isolasi orb = Could + spike | §6 Security, US-03 AC-03.2–03.4, R6 |
| 3 | CRITIC | B | Biaya Amp tidak ada di stream-json; `amp threads usage` tanpa `--json` | ACCEPT | **Diperiksa:** output teks `Cost: $0.04`. Parse dengan contract test, fallback `unknown` + estimasi + `--assume-cost-per-run`; run tanpa estimasi ditolak | US-04 AC-04.1–04.2, US-06, R9 |
| 4 | CRITIC + DISCUSSANT | B | Agent yang dievaluasi mewarisi kredensial, MCP akun (Resend, Linear, dll.), AGENTS.md global; bisa mengirim sesuatu ke orang lain | ACCEPT | **Diperiksa:** HOME sementara tidak menyembunyikan skill `global-user`/plugin. Environment di-scrub, `--settings-file` minimal, `--mcp-config '{}'`, tolak attempt bila MCP muncul, provenance tool/MCP/mode, service account disarankan; apakah MCP akun muncul di `-x` = spike Phase 5 | US-03, US-05 AC-05.3–05.4, §6, R11 |
| 5 | CRITIC | M | AC-08.1/08.2 lolos untuk implementasi yang salah; tidak ada fixture tidak degeneratif | ACCEPT | Fixture campuran + script referensi exact | AC-08.3 |
| 6 | CRITIC + DISCUSSANT | M | MDE tidak didefinisikan dan tidak diuji; harus dicetak sebelum biaya | ACCEPT | Simulasi power didefinisikan, dicetak di `plan` | US-08, AC-06.7, R1 |
| 7 | CRITIC | M | Budget tidak konsisten untuk paralel, biaya `unknown`, attempt gagal | ACCEPT | Admission rule dengan reservasi; semua attempt dihitung; M-6 memasukkan estimasi; G3 "sebelum setiap run" | G3, US-06, M-6 |
| 8 | CRITIC | M | Kunci estimasi biaya mengabaikan overlay/mode; `--pilot` tidak didefinisikan | ACCEPT | Kunci baru + fallback + AC pilot | AC-06.1–06.2 |
| 9 | CRITIC | M | Resume: kontradiksi AC-07.1/07.3, agent yatim tetap jalan, budget di hash | ACCEPT | Status `interrupted`, PGID durable, bunuh yatim, budget di luar hash, `run` tanpa flag ditolak | US-07, AC-02.5, AC-02.7 |
| 10 | CRITIC | M | Error provider tetap dihitung `fail` | ACCEPT | `error_kind` di adapter v1, mapping `result` error Amp, retry `max_attempts_per_slot` | AC-04.3–04.4 |
| 11 | CRITIC | M | Process group tidak menangkap `setsid` double-fork; keputusan cgroup TBD | ACCEPT | Subreaper + sapuan PID sebagai mekanisme Must; test agent double-fork | AC-03.5, §6 Reliability, R8 |
| 12 | CRITIC | M | AC-03.1 hash workspace = fixture mustahil karena overlay | ACCEPT | Hash = fixture + overlay sebelum setup | AC-03.1 |
| 13 | CRITIC | M | Preseden skill `global-user` vs overlay belum diketahui | ACCEPT | **Diperiksa:** overlay `workspace-agents` menang. Tetap diverifikasi per attempt | US-05, R4 |
| 14 | CRITIC | M | Provenance hanya skill; mode tidak diverifikasi | ACCEPT | Tool, MCP, `agent_mode`, versi CLI dicatat; mismatch → `infra_error` | AC-05.3 |
| 15 | CRITIC | M | Event M-3/M-4 tidak didefinisikan; `check_status` tidak ada | ACCEPT | Event `task_created`/`task_check_passed`, `check_status`, `check_timeout_sec` | Istilah US, US-01, M-3, M-4 |
| 16 | CRITIC | M | Window 200 run tidak sesuai budget US$11 | ACCEPT | Window jadi 30 attempt nyata | M-2, M-5 |
| 17 | CRITIC | M | Data hilang bersama orb | ACCEPT | Data-dir yang bisa di-commit + export/import | US-09 |
| 18 | CRITIC | M | Run panjang dan `serve` mati saat CLI Amp restart; dashboard butuh portal | ACCEPT | README `amp orb service` + `--portal` | AC-10.3, §6 Portability |
| 19 | CRITIC | M | `task check` satu kali tidak membuktikan stabilitas | ACCEPT | `--repeat 3` default, `check_timeout_sec` | AC-01.1, 01.3, 01.6 |
| 20 | CRITIC + DISCUSSANT | M | Must 17.5 hari > 3 minggu; dashboard 3 hari | ACCEPT | Dashboard live → Should; Must = laporan HTML statis. Perubahan scope, **dicatat di Options for Thomas** | §4, US-08 AC-08.8, US-10, §10 |
| 21 | CRITIC | M | Must masih punya keputusan terbuka (R8 ADR, R9, spike, build vs adopt); biaya build UNKNOWN; org GitHub | ACCEPT sebagian | R8 dan R9 diputuskan (#3, #11); R4 terjawab; org diperiksa ada (`gh repo view`); spike tersisa (MCP akun, orb) bukan Must dan punya fallback. Build vs adopt ke gate. Biaya build tetap UNKNOWN sampai Phase 10 dengan alasan | §7, §10, R10, R11 |
| 22 | CRITIC | M | Akuntansi token: cache split hilang, sumber penjumlahan tidak jelas, tarif tidak disebut | ACCEPT | Sum dari pesan `assistant` saja, empat field usage, tarif hanya untuk Codex dari `rates.yaml` | AC-04.1, adapter v1, AC-11.2 |
| 23 | CRITIC | m | `timeout` vs research.md | ACCEPT | Timeout = skor 0, terhitung | Istilah US |
| 24 | CRITIC | m | Format USD | ACCEPT | 4 desimal data, 2 di progres | §6 Localization, AC-06.3 |
| 25 | CRITIC | m | AC-01.4 baris untuk field hilang; AC-01.5 kabur | ACCEPT | Pesan dan path dispesifikasi | AC-01.4, 01.5 |
| 26 | CRITIC | m | Perintah tanpa AC (`init`, `task list`, `prune`, `--seed`) | ACCEPT | AC/state ditambahkan | US-01 states, US-02, US-09 AC-09.4 |
| 27 | CRITIC | m | Urutan acak murni bisa mengelompokkan arm (drift provider) | ACCEPT | Randomisasi berblok | AC-02.1 |
| 28 | CRITIC | m | Mengintip hasil sementara | ACCEPT | Label `interim`; CI hanya eksperimen selesai | US-08, AC-12.1, R13 |
| 29 | CRITIC | m | Biaya per pass dengan 0 pass | ACCEPT | `n/a` + jumlah unknown | AC-08.9 |
| 30 | CRITIC | m | Peringatan status lain | ACCEPT | Semua status | AC-08.5 |
| 31 | CRITIC | m | Byte identik butuh format tetap | ACCEPT | Tanpa waktu jam dinding, 4 desimal, per mesin | AC-08.6 |
| 32 | CRITIC | m | M-1 grep lemah; M-5 memasukkan setup | ACCEPT | M-1 berbasis keputusan dengan footer hash; M-5 mengeluarkan setup/provenance | M-1, M-5 |
| 33 | CRITIC | m | Penyamaran `AMP_API_KEY`; environment diwarisi | ACCEPT | Scrub env, semua nilai yang diteruskan disamarkan | US-03, AC-03.9 |
| 34 | CRITIC | m | US-11/12 lama: secret di PR fork; test glob; trailer | ACCEPT | `workflow_dispatch`/push saja; `--test-glob`; hanya judul commit | AC-12.2, AC-13.1 |
| 35 | CRITIC | m | Alasan UU PDP berbasis geografi | ACCEPT | Berbasis "data lokal di kendali pengguna" | §6 Privacy |
| 36 | DISCUSSANT | D | Coder Eval berjalan `tempdir` tanpa Docker; research.md salah | ACCEPT | **Diperiksa di source** (`models/sandbox.py`, `default="tempdir"`); research.md dan §1 dikoreksi | research.md, §1, R10 |
| 37 | DISCUSSANT | D | Harbor bukan kandidat adopsi realistis | ACCEPT | Docker default, task berbasis container | research.md |
| 38 | DISCUSSANT | D | Check berskor 0–1 untuk task dokumen | ACCEPT | Task factory sebagian besar dokumen; skor menurunkan varians. Verdict tetap dari uji berpasangan pada skor | Istilah US, US-01, R12 |
| 39 | DISCUSSANT | D | Laporan HTML statis menggantikan dashboard | ACCEPT | Sama dengan #20 | §4 |
| 40 | DISCUSSANT | D | Executor orb `amp -ox` | ACCEPT sebagai Could | Biaya orb, boot time, hanya Amp; spike di Phase 5 | §4 Could, R6 |
| 41 | DISCUSSANT | D | Pairbench-lite 2–3 hari sebelum build penuh | Options for Thomas | Mengubah urutan rencana secara signifikan | Bawah |
| 42 | DISCUSSANT | D | Eksperimen pertama = kalibrasi A/A + kontrol positif | ACCEPT | Eksperimen `prd-writer` mahal dan hampir pasti `inconclusive` | §9 M3 |
| 43 | DISCUSSANT | D | MCP akun berbahaya | ACCEPT | Sama dengan #4 | — |
| 44 | DISCUSSANT | D | Ringkasan biaya thread nyata untuk retro (`pairbench usage`) | ACCEPT sebagai Could | Korelasi, bukan kausal | §4 Could |
| 45 | DISCUSSANT | D | M-1 berbasis keputusan | ACCEPT | Sama dengan #32 | M-1 |
| 46 | DISCUSSANT | D | Budget sebagai "sisa kuota yang reset" | Options for Thomas | Butuh informasi model tagihan Thomas | Bawah |

## Options for Thomas

1. **Build vs adopt (paling penting).** (a) Bangun pairbench sesuai PRD (~16.5 hari agent; nilai belajar stack Rust/Effect sesuai rencana). (b) Adopsi Coder Eval: tulis plugin agent Amp + script verdict berpasangan + provenance (perkiraan reviewer 1–2 hari spike, belum terbukti; dependensi Python milik UiPath; kontribusi upstream menjangkau lebih banyak orang). (c) Hybrid: bangun pairbench, tetapi Phase 5 men-spike (b) sebagai pembanding di ADR. **Rekomendasi agent: (c).**
2. **Pairbench-lite dulu.** Script 2–3 hari (JSONL, Markdown, Amp saja) untuk mengukur biaya nyata, varians dan apakah task dokumen bisa dicek, sebelum MVP penuh. Trade-off: kode mungkin dibuang; menunda rilis OSS. Rekomendasi agent: tidak sebagai fase terpisah; spike vertical slice Phase 5 sudah mencakup sebagian besar pertanyaan ini.
3. **Dashboard live di Must atau Should.** Sudah diturunkan ke Should dan diganti HTML statis karena budget waktu. Thomas bisa mengembalikannya (+~3 hari).
4. **Model biaya eval.** Thread ini tercatat "Covered by your Amp Megawatt subscription". Bila eval memakai kuota langganan yang reset dan sisa kuota tidak bernilai, batas bisa berupa "maksimal X kredit per eksperimen" dan batas US$1/US$10 bisa dinaikkan. Butuh jawaban Thomas.

## Carried over (not fixed this round)

- Spike Phase 5: apakah server MCP level akun muncul di `amp -x` dengan `--settings-file` minimal dan `--mcp-config '{}'` (R11).
- Spike Phase 5: format dan stabilitas `amp threads usage` per thread yang baru selesai, termasuk keterlambatan pencatatan biaya (R9).
- Spike Phase 5: executor orb `-ox` (Could).
- Stack kandidat di [stack.md](../stack.md) memakai PostgreSQL dan React SPA; reviewer menilai SQLite/JSONL dan HTML statis lebih cocok untuk CLI lokal di orb 3 GB. Diputuskan di Phase 5, bukan di PRD.
