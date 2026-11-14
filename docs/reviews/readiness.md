# Readiness — 14 Nov 2026

**Verdict:** GO (dengan 3 syarat yang menjadi kriteria exit spike Phase 5, di bawah)

Diperiksa terhadap [prd.md](../prd.md) status "ready for readiness check" setelah [plan review ronde 1](plan-review-1.md) dan [ronde 2](plan-review-2.md).

| # | Item | Pass | Evidence |
| --- | --- | --- | --- |
| 1 | Every story has testable acceptance criteria | Yes | 82 AC; setiap AC Must menyebut hasil yang bisa di-assert e2e: exit code, string pesan, isi file/laporan, status attempt, isi `/proc` (US-01–US-09). Nilai statistik dibandingkan dengan script referensi independen (US-08). Setiap story Must punya ≥ 1 AC error/edge dan baris States |
| 2 | No TBD in MVP scope | Yes | `grep -i "TBD\|TODO\|nanti\|?"` pada prd.md: 4 kecocokan, semuanya bukan item terbuka (pertanyaan yang dijawab G1, pola regex check suite contoh, URL `?x=1`, regex AC-01) |
| 3 | Success metrics have numbers and a measurement method | Yes | §2 M-1–M-8: target, window, cara ukur (event/tabel/grep) |
| 4 | Major risks have mitigations | Yes | §8: R1, R2, R4, R6, R11, R15 (High) masing-masing punya mitigasi; R11 punya spike + fallback service account + aturan bila gagal (§7) |
| 5 | External dependencies identified | Yes | §7: Amp CLI (access token, kredit, ditanggung langganan menurut `amp threads usage`), service account (pemilik, ASUMSI biaya, fallback), GitHub org (sudah ada), repo factory, Claude/Codex (Should), GitHub Actions (Should, ASUMSI gratis), hosting US$0 |
| 6 | MVP fits the time and money budget | Yes | §10: Must ≈ 18.5 hari agent ≤ batas 20 hari; hosting US$0 < US$20/bulan (idea.md); eval US$1 + US$10 dengan persetujuan. Batas kredit build diajukan di Options 6 |
| 7 | Delivery mode stated | Yes | Header PRD: OSS Apache-2.0, tanpa hosting wajib (kategori 1) |
| 8 | Plan review finished | Yes | plan-review-1.md (4 BLOCKER, semua ACCEPT dan diterapkan) dan plan-review-2.md (4 BLOCKER, semua ACCEPT dan diterapkan); tidak ada BLOCKER terbuka; maksimal 2 ronde tercapai |

## Syarat GO (kriteria exit spike Phase 5, bukan perbaikan PRD)

1. **R11 tool akun:** spike membuktikan attempt `amp -x` bisa berjalan dengan `tools` ⊆ allowlist dan tanpa MCP (lewat `--settings-file`/`--mcp-config` atau service account). Bila tidak bisa, berhenti dan tanya Thomas sebelum build: Amp unsupported berarti tujuan G2 gagal.
2. **R9 biaya Amp:** spike mengukur keterlambatan dan kelengkapan `Cost:` di `amp threads usage`. Bukti awal: thread ini menunjukkan `Cost: $0.04` pada 06:01 UTC dan masih `$0.04` pada 06:20 UTC setelah kerja tambahan dan 4 subagent, jadi biaya yang dilaporkan terlambat atau tidak memasukkan subagent. Bila biaya tidak bisa dipercaya, budget Amp memakai `--assume-cost-per-run` (AC-06.5) dan Thomas diberi tahu.
3. **R1 power:** spike menjalankan suite contoh dengan biaya yang disetujui dan melaporkan MDE terukur; MDE > 40 pp → desain suite diubah sebelum M2.

## Options for Thomas (dari plan review)

Lengkap di [plan-review-2.md](plan-review-2.md#options-for-thomas). Ringkas:

1. Build vs adopt: aturan keputusan spike 1.5 hari Coder Eval + Amp → adopt bila lulus, build bila tidak; atau build langsung.
2. Spike Phase 5 dengan biaya nyata ±US$3–10 (ASUMSI) untuk ±12 attempt.
3. Laporan HTML statis di Must (sekarang) atau Should.
4. Attempt eval memakai kuota langganan tanpa biaya marjinal? (ya/tidak)
5. Kebijakan `inconclusive` + trailer `pairbench-report:` di skill retro factory.
6. Batas kredit build: berhenti dan bertanya di atas US$50 di luar langganan.

## Cost so far

- Amp: `amp threads usage` thread ini = **US$0.04, "Covered by your Amp Megawatt subscription"** (06:20 UTC). Angka ini kemungkinan belum lengkap (lihat syarat 2); detail di https://ampcode.com/threads/T-01a1d88d-c3dd-7437-9eab-5db84d3a0445/usage.
- Uang lain: US$0 (tanpa domain, hosting, Firecrawl atau layanan berbayar).
- Waktu: ±50 menit (05:36–06:25 UTC), Phase 0–4.
