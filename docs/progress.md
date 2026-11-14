# Progress — Coding-agent eval harness

## Status

- Phase: 1 Discovery selesai → 2 Plan
- Status: active (wave 1, disetujui Thomas 14 Nov 2026)
- Thread: https://ampcode.com/threads/T-01a1d88d-c3dd-7437-9eab-5db84d3a0445
- Cost so far: UNKNOWN USD (kredit Amp thread ini belum dibaca; dicek di gate), ~0.2 h

## Decisions

| Date | Phase | Decision | Why | Decided by |
| --- | --- | --- | --- | --- |
| 14 Nov 2026 | 0 | Docs dalam bahasa Indonesia; produk, CLI dan code dalam bahasa Inggris | AGENTS.md project + Batasan ide "bahasa EN" | agent |
| 14 Nov 2026 | 1 | Variasi terpilih V2: paired eval CLI untuk factory, Amp first + adapter command generik | Satu-satunya variasi yang menjawab pertanyaan retro "sebelum vs sesudah"; harness lain tanpa Amp dan butuh Docker | agent |
| 14 Nov 2026 | 1 | Delivery OSS (Apache-2.0), dashboard lokal, tanpa hosting | Kategori 1 public, budget < US$20/bulan, anti-persona skala lab | agent |
| 14 Nov 2026 | 1 | Nama kerja `pairbench` (.dev kemungkinan tersedia, .com terdaftar) | RDAP 14 Nov 2026; tidak dibeli | agent |
| 14 Nov 2026 | 1 | MVP tidak bergantung pada Docker | Daemon Docker tidak aktif di orb factory | agent |
| 14 Nov 2026 | 1 | Ikuti aturan diversitas stack yang dilonggarkan 7 Nov (fit > kuota) | Arahan Thomas, docs/tech-stack-plan.md factory | Thomas |

## Gates

| Gate | Date | Answer from Thomas |
| --- | --- | --- |
| Wave 1 selection | 14 Nov 2026 | Disetujui: 115, 403, 402, 601, 816 |

## Blockers

None.

## Riwayat sebelum wave 1

- 7 Nov 2026: repo dibuat sebagai dokumentasi perencanaan. Stack kandidat di [stack.md](stack.md) dan [stack-support.md](stack-support.md) adalah baseline perencanaan, bukan keputusan Phase 6. Keputusan lama di [decisions.md](decisions.md).

## Handoff

<Ditulis hanya saat pindah thread.>
