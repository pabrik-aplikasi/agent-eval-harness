# Stack — Coding-agent eval harness

Salinan rencana per project dari sumber factory pada commit `910a1523f397a1db94548c27b138c1d0998657a4`. Utama adalah baseline perencanaan; alternatif bukan stack kedua yang dijalankan bersamaan. Detail pendukung berada di [stack-support.md](stack-support.md).

## 1 — Dev tools · 17 project



| ID | Project / triage | Bahasa | Backend | Interface | DB | Hosting | Async | UI | Test | Alternatif | Alasan dan batas core |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 115 | Agent eval harness | Rust | Effect | React SPA + CLI TS | PostgreSQL | none | none | R1 | RW+CLI | L=Go; B=Connect RPC; T=GW+CLI | Rust runner, cancellation dan hasil; Effect TS orchestrator/provider. Subprocess bukan sandbox keamanan; desain isolasi terpisah. |

## Cara membaca delapan dimensi dan alternatif

Delapan kolom **Bahasa, Backend, Interface, DB, Hosting, Async, UI, Test** adalah dimensi kandidat utama. Bahasa adalah bahasa application/domain utama; TS pada interface dicatat dalam Interface. Detail Rust/Go tambahan dan batasnya ada di alasan.

Alternatif ditulis sebagai perubahan terhadap kandidat utama: `L` bahasa, `B` backend, `F` interface, `D` database, `H` hosting, `A` async, `U` UI, `T` testing. **Kolom yang tidak disebut tetap sama**; jadi setiap alternatif mendefinisikan delapan dimensi juga, bukan hanya nama backend. Alternatif belum menjadi keputusan final; perubahan harus ditinjau ulang agar database, worker, runtime, platform, dan kontraknya tetap cocok, bukan sekadar memenuhi angka diversitas.

`none` berarti tidak ada komponen tersebut, bukan belum dipikirkan. Hosting `none` berarti target pertama adalah artefak lokal/Compose/binary/build mobile; bukan janji service notifikasi bisa beroperasi publik tanpa server. Ketika layanan diaktifkan, hosting dan budget harus dipilih lagi. `VPS` berarti Docker Compose di VPS; `Railway` berarti target hosting usulan, **bukan resource yang sudah dibuat**. Semua kategori 3 mempunyai target hosting, termasuk website statis yang tidak mempunyai backend.

**Asumsi platform:** brief web+mobile dipertahankan sebagai web dan aplikasi native, bukan dipersempit menjadi PWA. `Expo` berarti native mobile; `Electron` hanya dipakai pada brief desktop. Offline, kamera, notifikasi background dan perangkat kasir tidak dianggap otomatis didukung. Tidak menambah dashboard untuk CLI/API-only.

### Kontrak komponen yang berlaku pada utama dan alternatif

- **Pekerjaan durable vs realtime:** setiap pekerjaan yang sudah diterima harus memiliki sumber durable. `DB outbox` berarti job/event dicatat dalam transaksi domain, lalu worker dengan claim/retry/idempotensi; bukan jaminan exactly-once efek eksternal. JetStream yang dipakai untuk pekerjaan wajib file-backed dengan volume persisten, durable consumer dan acknowledgement setelah hasil durable; outbox/rekonsiliasi menutup celah commit DB–publish. Label `NATS` yang masih singkat pada baris lain tidak mengizinkan Core NATS menjadi satu-satunya penyimpan pekerjaan: tentukan semantik durable sebelum build. Core NATS/Channels/presence hanya untuk update ephemeral bila kehilangan update dapat diterima dan state dapat dipulihkan.
- **PowerSync:** semua kandidat yang memakainya, termasuk alternatif509 dan kedua kandidat713, mencakup service, bucket storage PostgreSQL terpisah secara logis dari source, autentikasi klien, Sync Streams untuk konfigurasi baru (Sync Rules legacy), serta API upload yang memvalidasi dan menerapkan perubahan. Source PostgreSQL ≥14 dapat berbagi server dengan bucket storage; source MySQL tetap butuh PostgreSQL/MongoDB untuk bucket. Sync draft/cache tidak mengambil alih keputusan transaksi server.
- **Encore Pub/Sub yang tetap dipilih pada project lain:** dokumen pendukung memilih NSQ† self-host sebagai kandidat untuk408/606 utama dan211/218/608 alternatif, dengan outbox/rekonsiliasi serta batas durabilitas dan tidak adanya replikasi NSQ. Image Encore bukan broker otomatis; konfigurasi, resource, approval dependency dan biaya tetap gate. Pada312 baseline sengaja tidak memakai Pub/Sub.
- **Offline:** 716/813 menyimpan update lokal di IndexedDB dan menyediakan cache app shell, network provider serta persistence server. 809 menyimpan shell/WASM/model berversi dan menampilkan kesiapan offline setelah lengkap. Test mencakup tutup/buka ulang tanpa jaringan, reconnect bila relevan, cache hilang/eviction, serta pesan pemulihan; tidak menjanjikan storage browser tak pernah dihapus. Kandidat provider/cache/model dirinci di dokumen pendukung: Yjs + y-indexeddb + y-websocket pada716/813, serta Cache API/IndexedDB dan Transformers.js/ONNX Runtime Web pada809. Belum dipasang atau diuji.
- Middleware async `none`: tidak memakai queue/broker terpisah, bukan larangan batch atau pekerjaan terjadwal. Pada612 event disimpan sebelum ack; job agregasi/retensi mempunyai state/checkpoint DB yang dapat dipulihkan. Performa dan ukuran batch belum dibuktikan.
- **Alternatif hybrid:** bahasa pada kolom L tidak menghapus komponen lain yang disebut eksplisit. Pada814 TS tetap memiliki SSR/workflow DBOS; pada411 Rust tetap memiliki kalkulasi lewat subprocess. Test lintas runtime dan kontrak menjadi bagian kandidat, bukan pengecualian nanti.

### Legenda UI dan test

| UI | Isi |
| --- | --- |
| R1 | Tailwind + Base UI, React |
| R2 | Tailwind + shadcn/ui, React |
| R3 | Tailwind + HeroUI, React |
| R4 | StyleX + komponen React sendiri |
| S1 | Tailwind + Bits UI, Svelte |
| S2 | Tailwind + Skeleton, Svelte |
| O1 | Tailwind + komponen Solid sendiri |
| A1 | Tailwind + komponen Astro sendiri; islands mengikuti renderer yang ditulis |
| N1 | React Native StyleSheet + token komponen native |
| K1 | Komponen Kotlin native; toolkit spesifik perlu review |
| none | Tanpa GUI |

Semua UI tetap harus di-theme ulang pada fase desain. Gabungan seperti `R1+N1` berarti web dan mobile memiliki implementasi sesuai platform, bukan library DOM dipasang di native.

| Test | Rencana verifikasi, belum dijalankan |
| --- | --- |
| TW / BW | Vitest / Bun test + integrasi DB nyata jika ada + Playwright web |
| JW | Japa + integrasi DB nyata + Playwright untuk AdonisJS |
| GW / RW | go test / cargo test + Vitest interface TS + integrasi nyata + Playwright |
| KW / EW / FW / CW / LW / XW | Kotlin test / ExUnit / dotnet test / clojure.test / gleeunit / Crystal spec + integrasi nyata + Vitest interface + Playwright |
| ZW | zig test + test binding WASM dan browser Playwright |
| TM / GM / RM / KM | Unit TS / Go / Rust / Kotlin + integrasi nyata + Maestro di perangkat/runner native; tambah Playwright bila ada web |
| TC / GC / RC / NC / OC | Unit TS / Go / Rust / Nim / OCaml + e2e executable CLI dengan input/output nyata |
| TA / GA / RA / KA | Unit TS / Go / Rust / Kotlin + integrasi nyata + e2e API melalui proses server |

Test tambahan wajib mengikuti interface yang disebut: `+CLI`, `+API`, `+extension`, `+Electron`, `+Rust`, atau `+TS`. Semua binding lintas bahasa punya contract test; semua job durable punya uji retry/idempotensi. Maestro bukan bukti hardware/peripheral bekerja. Orb ini tidak menjalankan emulator Android; diperlukan device/runner native. Test profil bukan alasan untuk mengklaim pasangan arsitektur sudah berbeda secara bermakna.

## Arsitektur hybrid dan monorepo yang direncanakan

**Go core + TS interface:** pilihan utama REST/OpenAPI atau Connect RPC; browser memakai client TS, bukan Go FFI. Go CLI dapat memakai stdin/stdout bila perlu wrapper TS. Go WASM mungkin, tetapi tidak dipilih hanya untuk menyamai Rust; compiler dan wasm\_exec harus seversi dan ukuran/runtime perlu diuji.

**Rust core + TS interface:** WASM untuk browser-local compute; napi-rs hanya pada Node target yang diuji; subprocess untuk tool/batch; HTTP/queue untuk server/worker yang benar-benar perlu proses terpisah. Rust native addon tidak dianggap otomatis bekerja di workerd atau seluruh runtime Bun.

Struktur rencana mengikuti monorepo factory: apps/web, apps/api atau apps/worker jika perlu; packages/core untuk core lintas interface; packages/bindings untuk satu binding terpilih; contracts untuk schema; e2e untuk scenario nyata. Tidak semua folder perlu dibuat. DB transaction dimiliki satu application/domain boundary, bukan ditulis sembarang dari dua bahasa.

Rencana monorepo memakai pnpm workspaces untuk bagian JS/TS, dengan Cargo/Go/dotnet/Mix/Gradle/Gleam/Zig dan toolchain native lain tetap memiliki build masing-masing. Baseline tidak menambah task runner lintas bahasa; tambahkan hanya bila kebutuhan build membenarkannya. Ini pilihan dokumentasi, bukan instalasi atau konfigurasi CI yang sudah berjalan.

## Gate teknis dan tambahan palette
Tanda `*` pada tabel merupakan penanda asal tambahan palette. Thomas kini mengizinkan Phoenix, Reitit, Giraffe, Wisp, Kemal dan SQLite standalone sebagai kandidat; tambahan lain belum otomatis disetujui. Bahasa induknya sudah berada dalam palette. Semua framework/driver, binding WASM/napi-rs, codec, inference runtime, dan widget native tetap memerlukan audit versi/lisensi/maintenance sebelum implementasi; kandidat baru dalam dokumen pendukung diberi tanda †.

SvelteKit/SolidStart dan framework JS/TS tambahan ditinjau berdasarkan izin ekosistem JS/TS modern dalam palette; belum berarti setiap dependency telah lolos. Effect HttpApi v4 yang diperiksa berlabel unstable; jangan mencampur tutorial v3 dan v4. S3, LLM, WA, layanan auth, provider data, dan hosting tidak diasumsikan gratis. Seluruh biaya perlu approval.

Auth dan driver/migrasi kini mempunyai kandidat per project beserta delta alternatif di dokumen pendukung. Better Auth dipakai pada runtime TS yang cocok; Keycloak menjadi default lintas bahasa untuk aplikasi multiuser, dengan pengecualian eksplisit PocketBase native auth, Clerk/Convex dan aplikasi tanpa akun. Otorisasi domain tetap di core; Keycloak memiliki identity store dan biaya resource sendiri. Driver/migrator mengikuti runtime dan satu pemilik per schema; kompatibilitas, versi, tenancy/SSO, lisensi dan approval tambahan masih gate. Ini bukan klaim production-ready.

Maksimal dua overengineering choice per project tetap berlaku. Hybrid, sync engine, broker terpisah, distributed DB dan layanan ML masing-masing harus punya alasan; kombinasi yang tidak bisa dibenarkan akan disederhanakan saat PRD/ADR, bukan diam-diam diloloskan.

## Sumber dan verifikasi yang tersedia

Sumber kebutuhan adalah seluruh135 brief dan triage repo, bukan asumsi pasar baru. Docs resmi yang telah dibaca memverifikasi kapabilitas tertentu, bukan semua kombinasi:

- [Hono](https://hono.dev/docs/), [H3](https://h3.dev/), [Nitro](https://nitro.build/docs), [Elysia/Eden](https://elysiajs.com/at-glance), [Fastify](https://fastify.dev/docs/latest/Reference/Encapsulation/), [NestJS adapter](https://docs.nestjs.com/http/performance), [Encore.ts](https://encore.dev/docs/ts), [Effect HttpApi](https://effect.website/docs/v4/api/effect/http-api/HttpApi).
- [TanStack Start](https://tanstack.com/start/latest/docs/framework/react/overview), [SvelteKit](https://svelte.dev/docs/kit/introduction), [SolidStart](https://docs.solidjs.com/solid-start), [Astro](https://docs.astro.build/en/basics/rendering-modes/), [Convex](https://docs.convex.dev/).
- [Connect Go/TS](https://connectrpc.com/docs/introduction/), [Go WASM](https://go.dev/wiki/WebAssembly), [Huma](https://huma.rocks/), [River](https://riverqueue.com/docs).
- [Axum](https://docs.rs/axum/latest/axum/), [Actix Web](https://actix.rs/docs/), [Salvo](https://salvo.rs/), [Poem/OpenAPI](https://docs.rs/poem-openapi/latest/poem_openapi/), [Rocket](https://rocket.rs/guide/v0.5/overview/), [wasm-bindgen](https://rustwasm.github.io/docs/wasm-bindgen/), [napi-rs](https://napi.rs/docs/introduction/getting-started).
- [Ktor](https://ktor.io/docs/server-create-a-new-project.html), [Phoenix](https://www.phoenixframework.org/), [Wisp](https://hexdocs.pm/wisp/), [Reitit](https://metosin.github.io/reitit/), [Giraffe](https://github.com/giraffe-fsharp/Giraffe), [Kemal](https://kemalcr.com/), [Zig WASM](https://ziglang.org/documentation/master/#WebAssembly), [Nim tests](https://nim-lang.org/docs/unittest.html).
- [PowerSync Expo](https://docs.powersync.com/client-sdk-references/react-native-and-expo), [Crawlee](https://crawlee.dev/js/docs/introduction).
- [PowerSync source database — termasuk status MySQL](https://docs.powersync.com/configuration/source-db/setup), [PocketBase](https://pocketbase.io/docs/), [Android SQLite](https://developer.android.com/training/data-storage/sqlite), [AdonisJS](https://docs.adonisjs.com/guides/introduction).

Riset sumber tidak menggantikan spike. Testing pada tabel adalah rencana, **bukan hasil test aplikasi**. Audit data matriks hanya memeriksa cakupan, isi kolom dan konflik aturan.
