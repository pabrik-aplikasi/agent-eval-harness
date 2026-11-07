# Pembahasan dan keputusan — Coding-agent eval harness

## Status keputusan

Masuk ranking triage; belum dipilih untuk wave implementasi. Skor dan klaim pasar pada triage adalah **ASUMSI**, bukan riset pasar. Repo dibuat atas permintaan dokumentasi semua project pada 7 November 2026; bukan instruksi menjalankan wave atau membatalkan usulan merge/kill.

Prioritas sumber: keputusan Thomas terbaru → tabel stack revisi dan pendukung → catatan review historis → brainstorm. Catatan historis di bawah tidak boleh membatalkan koreksi terbaru.

## Keputusan dan batas draf

- **Arahan Thomas:** terapkan rekomendasi review pada dokumen, pertahankan diversitas, jangan coding. Kecocokan stack lebih penting daripada kuota; sedikit variasi yang bermakna dapat diterima. Batas lama dua penggunaan dan beda 4/8 bukan alasan otomatis mengganti stack. Bahasa/framework utama tetap bervariasi; tidak menambah komponen tanpa kebutuhan. Perubahan ini terbatas pada artifact, bukan sinkronisasi aturan repository atau registry stack final.
- Thomas membolehkan **Go core + TS interface** maupun **Rust core + TS interface**, dengan minat lebih besar pada Rust. Interface TS tidak harus berarti backend TS tambahan.
- Semua **135 ide tetap terlihat**, termasuk 7 usulan merge dan 11 usulan kill. Status itu belum keputusan final; kandidat standalone untuk mereka bersifat kondisional.
- Kategori 2/3/6 tetap memakai aplikasi utama TS/Go; Rust boleh menjadi core komputasi terbatas yang disebutkan eksplisit. Kategori 7/8 selalu mempunyai bahasa eksplorasi dengan tugas nyata. Ini interpretasi desain, bukan izin mengganti seluruh backend bisnis menjadi Rust.
- Pilihan utama = rekomendasi awal berdasarkan brief enam baris, belum berdasarkan PRD lengkap. Versi, lisensi, maintenance, biaya, kompatibilitas semua kombinasi, dan kelulusan e2e belum dibuktikan.
- Tidak ada keputusan wave, merge/kill, produksi, atau pengeluaran uang yang dibuat melalui dokumen ini. Registry final tidak diubah.

## Keputusan Thomas setelah pembahasan stack pendukung

Thomas menyetujui rekomendasi pembahasan pada 7 November 2026 dan meminta seluruh stack pendukung dilengkapi, tetap tanpa coding. Persetujuan mencakup:

- Palette merupakan daftar pilihan, bukan kewajiban memakai semua teknologi. Diversitas utama dipertahankan; auth, query tooling, monorepo, dan observability boleh berulang ketika cocok.
- Phoenix, Reitit, Giraffe, Wisp, Kemal dan SQLite standalone diizinkan sebagai kandidat. Ini bukan approval otomatis semua versi, dependency turunan, instalasi, atau biaya.
- 815: web memakai template tepercaya dan data tervalidasi; CLI boleh menjalankan kode milik operator. Hosted arbitrary user code tidak masuk baseline.
- 713: draft boleh offline; penerimaan final diputuskan server melalui finalisasi idempotent dan receipt. Submit setelah deadline menunggu keputusan pengawas, bukan otomatis diterima; jam perangkat bukan bukti waktu penyerahan. Keputusan pengawas harus tercatat, tanpa mengganti waktu penerimaan asli.
- 716/813/809: offline mencakup tutup dan buka ulang setelah data/aset siap tersimpan; storage browser tetap bisa hilang dan perlu pesan pemulihan.
- 212: PostgreSQL menjadi baseline dengan asumsi belum ada integrasi MySQL wajib; MySQL tetap alternatif dengan inventaris sync lengkap.
- Lengkapi pilihan auth/otorisasi, data/migrasi, runtime/kontrak, komponen khusus dan operasional seluruh project sebagai rencana, bukan kelulusan pengujian.

Wave, merge/kill, pembelian, aktivasi layanan dan deployment tetap tidak disetujui melalui keputusan ini. Aturan lama di repository/skills belum disinkronkan; registry final tetap tidak diubah.

## Kutipan triage khusus project

## Ranking

| Rank | Idea | Cat | Pain | User | Diff | Feas | Money | Learn | Total | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | `115-agent-eval-harness` Coding-agent eval harness | 1 | 4 | 5 | 4 | 4 | 1 | 5 | 23 | Langsung memperbaiki factory; user = Thomas |
### Usulan wave 1 (belum disetujui)

| Idea | Cat | Total | Why in this wave |
| --- | --- | --- | --- |
| `115-agent-eval-harness` Coding-agent eval harness | 1 | 23 | Langsung memperbaiki factory; user = Thomas |

## Pertimbangan percakapan awal

Tidak ada PRD final atau keputusan desain per project yang ditemukan.

## Ringkasan brainstorm awal

Task berulang lintas agent/model, skor hasil/biaya/waktu; Thomas; evaluasi skill/reviewer factory.

Sumber: artifact brainstorm revisi 1 (6 November 2026), ditautkan di sources.md. Ini bahan ide, bukan scope MVP atau riset terverifikasi; klaim pasar, kompetitor, tarif dan regulasi adalah **ASUMSI**. Koreksi keputusan terbaru tetap berlaku.

## Hasil review yang menyebut project ini

Tidak ada baris koreksi khusus ber-ID project ini; baseline dan batasnya tetap mengikuti tabel stack terkini.

## Riwayat review (bukan instruksi mengganti keputusan terbaru)

Tidak ada temuan R01–R15 yang secara eksplisit menyebut ID project ini.
