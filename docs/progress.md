# Progress — Coding-agent eval harness (pairbench)

## Status

- Phase: 4 Readiness — **menunggu gate GO / NO-GO dari Thomas**
- Status: active (wave 1, disetujui Thomas 14 Nov 2026)
- Thread: https://ampcode.com/threads/T-01a1d88d-c3dd-7437-9eab-5db84d3a0445
- Cost so far: US$0.04 per `amp threads usage` (ditanggung langganan Megawatt; kemungkinan belum lengkap), ±0.8 h

## Decisions

| Date | Phase | Decision | Why | Decided by |
| --- | --- | --- | --- | --- |
| 14 Nov 2026 | 0 | Docs dalam bahasa Indonesia; produk, CLI dan code dalam bahasa Inggris | AGENTS.md project + Batasan ide "bahasa EN" | agent |
| 14 Nov 2026 | 1 | Variasi terpilih V2: paired eval CLI untuk factory, Amp first + adapter command generik | Satu-satunya variasi yang menjawab pertanyaan retro "sebelum vs sesudah"; harness lain tanpa Amp | agent |
| 14 Nov 2026 | 1 | Delivery OSS (Apache-2.0), tanpa hosting | Kategori 1 public, budget < US$20/bulan, anti-persona skala lab | agent |
| 14 Nov 2026 | 1 | Nama kerja `pairbench` (.dev kemungkinan tersedia, .com terdaftar) | RDAP 14 Nov 2026; tidak dibeli | agent |
| 14 Nov 2026 | 1 | MVP tidak bergantung pada Docker | Daemon Docker tidak aktif di orb factory | agent |
| 14 Nov 2026 | 1 | Ikuti aturan diversitas stack yang dilonggarkan 7 Nov (fit > kuota) | Arahan Thomas, docs/tech-stack-plan.md factory | Thomas |
| 14 Nov 2026 | 3 | Inferensi bersyarat suite dengan uji permutasi berstrata exact (konvolusi), bukan bootstrap dua tingkat | Bootstrap dua tingkat bisa memberi `better` dari 2 run; plan review ronde 1–2 | agent |
| 14 Nov 2026 | 3 | Penyembunyian check dipersempit ke jaminan yang bisa diuji; model ancaman "semi-tepercaya" | Tanpa sandbox, `find /` tetap menemukan check | agent |
| 14 Nov 2026 | 3 | Allowlist tool + tolak MCP saat init; env agent dan check di-scrub | Tool akun (Slack, Bitbucket, notify) terlihat di sesi; guardrail "ask before sending" | agent |
| 14 Nov 2026 | 3 | Dashboard live turun ke Should; Must = laporan Markdown + HTML statis | Must harus ≤ 20 hari agent | agent (bisa dibalik Thomas) |
| 14 Nov 2026 | 3 | Check berskor 0–1 dan suite contoh 3 task dokumen dari skill factory | Sebagian besar skill factory menghasilkan dokumen | agent |
| 14 Nov 2026 | 3 | Riset dikoreksi: Coder Eval berjalan `tempdir` tanpa Docker dan punya paired comparison, resume, anti-cheat | Diperiksa di source commit `d8011f0` | agent |

## Gates

| Gate | Date | Answer from Thomas |
| --- | --- | --- |
| Wave 1 selection | 14 Nov 2026 | Disetujui: 115, 403, 402, 601, 816 |
| Phase 4 GO / NO-GO | — | Menunggu. Rekomendasi agent: GO dengan 3 syarat spike ([readiness.md](reviews/readiness.md)) |

## Blockers

None. Menunggu gate Phase 4.

## Carried over dari plan review

Lihat [plan-review-2.md](reviews/plan-review-2.md#carried-over-not-fixed-this-round): spike R11 (tool/MCP/AGENTS.md akun), R9 (keterlambatan biaya Amp), event tool-use `skill`, `bwrap`/`unshare`, executor orb, label thread, stack ulang di Phase 5.

## Riwayat sebelum wave 1

- 7 Nov 2026: repo dibuat sebagai dokumentasi perencanaan. Stack kandidat di [stack.md](stack.md) dan [stack-support.md](stack-support.md) adalah baseline perencanaan lama (PostgreSQL, React SPA), bukan keputusan Phase 6. Keputusan lama di [decisions.md](decisions.md).

## Handoff

<Ditulis hanya saat pindah thread.> Langkah berikut setelah GO: Phase 5 stack-selector (load skill `stack-selector`), dengan spike yang memenuhi 3 syarat di readiness.md dan aturan build vs adopt dari jawaban Thomas.
