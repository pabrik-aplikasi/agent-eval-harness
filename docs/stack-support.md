# Stack pendukung — Coding-agent eval harness

Rencana, bukan hasil pengujian. Kolom delta mengganti komponen utama bila alternatif dipilih. Tidak menginstal atau mengaktifkan kandidat † tanpa persetujuan yang diperlukan. Catatan “belum commit/provisioning” dalam kutipan sumber merujuk tahap perencanaan; repository dokumentasi kini dibuat terpisah dari implementasi.

| ID | Auth / batas akses | Data & migrasi | Runtime / API / validasi | Komponen khusus | Operasional / biaya | Delta alternatif |
| --- | --- | --- | --- | --- | --- | --- |
| 115 | Lokal tanpa akun; Better Auth hanya dashboard bersama. | PostgreSQL Drizzle/pg/Kit milik orchestrator; artefak S3 bila berbagi. | Effect Node+Effect Schema; Rust runner lewat subprocess JSON berversi. | Manifest task/seed/model; sandbox disposable, egress terbatas, timeout/cancellation; proses anak bukan isolasi. | L/S; sandbox, token model, artefak. | Go Connect+pgx†/goose†; Keycloak jika bersama; runner Go native mengganti Rust/Effect; client Connect TS. |

## Cara membaca pilihan dan approval

- Nama tanpa versi adalah keluarga teknologi yang dipilih untuk rencana. Versi compiler/runtime/SDK/model harus dikunci bersama lockfile dan checksum pada fase validasi nanti, bukan menebak versi terbaru sekarang.
- Tanda **†** berarti tambahan dependency/tool/provider konkret yang belum memperoleh persetujuan khusus di palette. Ia boleh ditelaah sebagai kandidat, tetapi tidak boleh dipasang atau diaktifkan hanya karena tercantum di sini. Persetujuan umum merencanakan bukan persetujuan membeli atau menginstal. SDK/driver pendamping tetap perlu audit meskipun teknologi induknya disetujui.
- `none` / tanpa akun / tanpa DB merupakan pilihan yang disengaja. Library lokal, CLI dan browser-only tidak diberi server hanya agar terlihat lengkap.
- `S3` berarti AWS S3 sebagai kandidat cloud yang konkret, bukan resource yang sudah ada. Filesystem lokal hanya jalur demo; tidak membuktikan signed URL, multipart, lifecycle atau permission S3 bekerja. Data sensitif memakai object privat dengan akses terbatas, bukan bucket publik.
- Kolom operasional mencatat target dan penggerak biaya, bukan penawaran harga. Volume pengguna/event, retensi, ukuran objek, durasi komputasi dan harga wilayah belum cukup untuk mengeluarkan estimasi bulanan yang dapat dipercaya.
- Hal yang belum dipilih karena izin data, kontrak eksternal atau ketersediaan hardware ditulis sebagai gate. Tidak ada fallback diam-diam menjadi scraping tanpa izin, API berbayar, model cloud, PWA pengganti native, atau implementasi kriptografi sendiri.

## Kontrak pendukung bersama

### Auth adalah identitas; izin domain tetap milik aplikasi

Untuk aplikasi TS dengan runtime dan adapter sesuai, Better Auth adalah kandidat library utama; tidak menambah service TS kedua untuk aplikasi yang tidak memerlukannya. Schema auth yang dihasilkan masuk satu jalur migrasi yang ditunjuk, bukan dua migrator yang saling mengubah tabel. Session browser memerlukan kebijakan cookie/CSRF; token native memerlukan penyimpanan aman dan revocation yang sesuai. Pilihan per baris dapat memakai auth bawaan PocketBase, Clerk untuk Convex, GitHub App, atau tanpa akun bila lebih cocok.

Untuk aplikasi non-TS multiuser, Keycloak OIDC adalah kandidat IdP agar core domain tidak perlu menulis sistem password sendiri. Browser/native menggunakan authorization code dengan PKCE bila sesuai jenis client; tidak memakai password grant atau menyimpan client secret dalam browser/aplikasi native. Server memvalidasi issuer, audience, signature, expiry dan izin domain. Keycloak **menambah service dan database milik IdP, default PostgreSQL**; jika DB domain bukan PostgreSQL atau tidak ada DB domain, tambahan ini tetap harus dihitung. Pengecualian eksplisit alternatif610 memakai database identity MySQL terpisah pada instance MySQL, yang juga didukung Keycloak. Ia bukan dependency tersembunyi dan bukan default wajib bagi CLI, demo lokal atau situs anonim.

Tenant membership, owner/resource access, hubungan wali–murid, role pengawas, entitlement pembayaran dan policy medis tetap diperiksa di core aplikasi. Role IdP bukan pengganti pemeriksaan baris/data. Capability link hanya memberi akses sempit, dapat dicabut/kedaluwarsa; bukan token admin atau pengganti autentikasi semua pengguna. Integrasi GitHub App menggunakan installation permission yang dibatasi, bukan PAT bersama.

### Satu pemilik migrasi per schema

Driver/query generator mengikuti bahasa yang menulis transaksi. Tool migrasi dipilih per baris; tidak menjalankan Drizzle, goose dan SQLx bergantian pada tabel yang sama. Auth, domain, queue/workflow, dan sync bucket memiliki ownership schema yang terpisah secara eksplisit, walaupun dapat berbagi mesin DB. Framework-managed schema seperti Convex/PocketBase tidak ditulis langsung melalui driver SQL eksternal.

Alternatif mengganti driver, serializer, validator dan migrator milik aplikasinya; ia mewarisi kontrak bisnis, platform, privacy dan durability kecuali delta mengatakan lain. Pergantian pilihan stack bukan migrasi data yang sudah ada, karena belum ada aplikasi yang dibangun. Frontend TS tidak memiliki kredensial DB server.

### Durable jobs, sync dan realtime tidak saling menggantikan

Job/outbox dicatat atomik bersama transaksi domain bila diperlukan. Worker mengklaim pekerjaan, retry dengan batas, menyimpan hasil dan menangani duplikasi. JetStream untuk pekerjaan memakai file-backed storage, durable consumer dan acknowledgement setelah hasil durable; Core NATS hanya boleh untuk data ephemeral yang dapat dipulihkan. Tidak ada klaim exactly-once efek eksternal.

Semua PowerSync mencakup service, bucket storage PostgreSQL terpisah secara logis, auth, **Sync Streams** dan upload API tervalidasi. Sync Rules lama berstatus deprecated menurut dokumentasi saat dibaca; baseline baru memilih rilis yang mendukung Sync Streams, bukan mencampur YAML/contoh lama. Source MySQL memerlukan bucket database tambahan; status source/adapter perlu dipastikan kembali saat memilih versi. PowerSync tidak mengubah draft ujian menjadi submission accepted atau stok global menjadi pasti tersedia saat perangkat offline. SDK native Expo memakai adapter OP-SQLite, sementara web/Electron memakai adapter masing-masing yang dijelaskan pada713; tidak dianggap saling tertukar.

Jika Encore Pub/Sub dipilih, provider dan resource harus benar-benar ada dalam inventaris. Provider tambahan yang disebut dalam tabel merupakan kandidat, bukan kemampuan broker otomatis dari image Encore. Tidak ada broker baru pada baseline312 dan tidak ada Trigger.dev pada baseline512.

### Packaging dan protokol

Semua project tetap memiliki repository sendiri, monorepo di dalamnya. Pilihan bersama untuk bagian JS/TS adalah **pnpm workspaces**; Bun tetap boleh menjadi runtime/test runner ketika tabel utama memilihnya, bukan alasan mengganti bahasa/runtime. Cargo, Go modules, Gradle, Mix, dotnet, Gleam, Crystal, Nim, OCaml dan Zig tetap memiliki dependency/build native masing-masing. Tidak menambahkan Nx/Turborepo atau task runner lintas bahasa untuk semua project.

Pemanggilan build lintas bahasa dapat memakai workspace scripts sederhana pada fase implementasi, tetapi dependency graph native tidak dipindah ke package manager JS. CI yang direncanakan adalah GitHub Actions dengan checks unit, integrasi service nyata dan e2e mengikuti profil pada peta utama; belum ada workflow atau runner yang dibuat. Runtime/image dan dependency harus dipin saat validasi, bukan memakai tag latest di produksi.

API eksternal memiliki satu kontrak versioned: OpenAPI, Protobuf/Connect atau schema framework sesuai baris. Typed client tidak menggantikan validasi server. Rust–TS memakai salah satu boundary nyata: WASM, napi-rs pada Node, subprocess berversi, atau API/queue. Go alternatif tidak memuat addon Node. Generated code tidak dianggap aman hanya karena lolos parser.

### Navigasi dan state interface

Profil berikut berlaku untuk seluruh baris dengan interface terkait, termasuk alternatif; tidak mengubah pilihan React/Svelte/Solid/Astro atau native pada peta utama. Layar tunggal tanpa navigasi tidak perlu router tambahan.

| Interface | Pilihan pendukung | Batas |
| --- | --- | --- |
| React SPA | TanStack Router untuk multi-route; TanStack Query untuk remote server state; state React untuk interaksi lokal | Cache query bukan sumber saldo/stok/entitlement; tidak menambah Zustand/XState tanpa kebutuhan |
| Svelte SPA | svelte-spa-router† untuk multi-route; state bawaan Svelte dan fetch untuk akses API sederhana | Kandidat router hash, bukan migrasi ke SvelteKit; halaman yang memakai fragment untuk kunci/capability tidak membagi fragment dengan router |
| Solid SPA | Solid Router† untuk multi-route; state bawaan Solid dan fetch untuk akses API sederhana | Tidak mengubahnya menjadi SolidStart atau menyalin algoritma core ke state UI |
| TanStack Start / SvelteKit / SolidStart / Astro | Router dan data-loading bawaan framework; state island mengikuti React/Svelte/Solid yang memang dipilih | Tidak memasang router SPA kedua ke seluruh halaman SSR/SSG |
| Expo | Expo Router†; TanStack Query untuk remote API bila bukan data milik sync engine; state React lokal | Native screen dan deep link; bukan pengganti offline storage atau bukti plugin bekerja di Expo Go |
| Kotlin native | Toolkit dan persistence Android yang disebut pada baris501/509; tidak memakai library DOM | Build/test memerlukan toolchain dan runner/perangkat native |
| CLI/API-only | Tidak ada router/UI state | Tidak menambah dashboard hanya untuk menyeragamkan |

Data Convex tetap melalui client reactive Convex, draft PowerSync melalui SQLite/sync, dan dokumen Yjs melalui CRDT; jangan membuat salinan otoritatif kedua di query cache. Form sederhana memakai kontrol platform, schema validasi yang dipilih pada baris dan validasi server; belum menambah library form global. Browser-only tanpa data remote tidak memerlukan query library. Styling/komponen tetap mengikuti R1–R4, S1–S2, O1, A1, N1 dan K1 pada peta utama.

### Profil operasional dan biaya

| Profil | Target rencana dan layanan dasar | Batas biaya dan bukti berikutnya |
| --- | --- | --- |
| L | Binary/CLI, browser lokal, build mobile atau desktop; log lokal teredaksi; tanpa telemetry keluar secara default | Distribusi binary/aplikasi, ukuran model/aset, perangkat uji dan signing mungkin berbiaya; lokal tidak berarti semua distribusi gratis |
| S | Docker Compose lokal untuk server/DB/worker yang memang dipilih; JSON logs teredaksi, health/readiness, backup/restore, restart recovery | Belum menyewa VPS. Jika di-host, hitung API, worker, IdP, DB, broker, sync, storage, backup, traffic dan vendor; jangan hanya menghitung web process |
| P | Layanan publik kategori3, target Railway/VPS sesuai peta utama; static build bila tidak ada backend, rate limiting/abuse control bila ada API | Tidak otomatis memenuhi <US$20/bulan. Tambahan Keycloak/Baserow/Convex, bandwidth dan API model harus masuk estimasi |
| M | Managed service yang sudah dipilih, terutama Convex; export/restore dan batas penggunaan tetap perlu dirancang | Tidak membuat akun/berlangganan. Kuota, metering, lokasi data dan biaya auth/storage diperiksa sebelum aktivasi |

Profil dapat digabung, misalnya L+S untuk aplikasi native dengan server, atau P+M untuk web publik dengan Convex. Profil tidak menghapus target hosting pada tabel utama.

Observability bersama: JSON logs dengan request/job ID dan redaksi data sensitif sebagai baseline. OpenTelemetry dipakai pada sinyal/runtime yang didukung; ekspor hanya diaktifkan setelah tujuan dan privacy disepakati. Kandidat lingkungan observability lokal adalah **Grafana LGTM**, opsional untuk demo/test terpisah, bukan rangkaian layanan yang wajib dipasang per project kecil atau konfigurasi produksi yang sudah siap. Jangan memaksa SDK JavaScript ke runtime browser/WASM atau menganggap semua bahasa eksplorasi memiliki kematangan SDK yang sama. App lokal tidak mengirim dokumen, gaji, token atau prompt ke telemetry otomatis.

Untuk estimasi nanti, setiap project memerlukan: pengguna aktif dan puncak concurrency; volume/ukuran event dan objek; retensi+backup; durasi dan frekuensi job; egress; panggilan AI/WA/mail/payment; serta biaya runner/perangkat. Jumlah resource idle juga dihitung. Tidak ada tarif atau jaminan budget yang dikarang pada tahap ini.

### Provider dan model yang dipilih untuk rencana

Pilihan ini melengkapi baris yang hanya menulis WA, email atau LLM; tidak menambahkan integrasi tersebut ke project yang tidak membutuhkannya. Provider yang disebut khusus pada suatu baris mengungguli default ini. Semua komunikasi nyata, pengiriman data pengguna dan biaya tetap memerlukan approval.

| Kebutuhan | Kandidat konkret | Batas keputusan |
| --- | --- | --- |
| WA dan email | Meta WhatsApp Cloud API†; Resend† untuk email transaksional | Consent, template, domain, sandbox dan biaya belum disetujui; demo hanya menangkap intent lokal |
| Pembayaran | Midtrans† QRIS untuk pembayaran satu kali; Subscription API kartu/GoPay pada604 | Bukan QRIS recurring; onboarding, callback, refund dan rekonsiliasi masih gate |
| Ringkasan/label/penulisan cloud pada110/111/310/408/607/608/814 | OpenAI† `gpt-5.6-terra` sebagai kandidat awal, AI SDK pada TS; HTTP provider pada core Go/Rust | Pilihan untuk evaluasi, bukan klaim paling murah/akurat; schema output, token budget, consent, PII dan availability harus diuji; tidak mengubah115 menjadi benchmark satu model |
| Transkrip teks608/814 | OpenAI† `gpt-transcribe` melalui Audio Transcriptions API | Tidak mengasumsikan word timestamps, speaker diarization atau SRT dari endpoint teks; consent dan kualitas Bahasa harus dibuktikan |
| Subtitle bertiming615 | whisper.cpp† CLI dengan model `base` multilingual†, output SRT/VTT; FFmpeg† untuk media | Proses C/C++ eksternal tidak mengganti Hono/Node sebagai aplikasi utama; CPU/RAM, timing dan kualitas model perlu diuji, bukan API hosted Whisper |
| AI lokal809 | Transformers.js + ONNX Runtime Web, distribusi ONNX Qwen2.5-0.5B-Instruct† | Tidak memakai provider cloud; versi/hash artefak, operator/model compatibility, kualitas ID/EN dan restart offline tetap gate |

Pemilihan model814 dikoreksi saat konsolidasi: halaman resmi menandai `gpt-4.1-mini` dan hosted `whisper-1` deprecated. Karena itu keduanya tidak dijadikan baseline baru. Model dan provider tetap terpisah dari aturan bisnis, agar lifecycle model tidak memaksa pergantian bahasa/framework. Library/SDK yang belum menerima identifier/model baru harus di-upgrade setelah audit atau memakai API HTTP yang terdokumentasi, bukan menganggap dukungan otomatis. Tidak ada API model dipanggil pada pekerjaan dokumen ini.

### Konvensi kategori 1 dan 4

- **† = kandidat baru di luar palette, belum approved.** Reitit, Giraffe, Wisp, Kemal dan SQLite standalone sudah diizinkan sebagai kandidat; tetap bukan pengesahan versi. Dependency JS/TS mengikuti izin ekosistem modern dalam palette, tetapi maintenance, lisensi, dan kompatibilitasnya tetap harus diaudit. Tidak ada versi paket dipatok atau klaim tested.
- **L:** lokal, tanpa telemetry remote. **S:** server Compose, JSON logs tersensor, health/readiness, OTel hanya pada runtime yang mendukung; ekspor opt-in. Tidak ada profil P atau M di bagian ini: tidak menambahkan hosting publik/managed pada kategori 1/4. Kolom biaya hanya menyebut pendorong biaya dan layanan tambahan, bukan estimasi.
- **Akun:** Better Auth di runtime TS yang cocok; Keycloak OIDC untuk server non-TS multiuser. Authz tenant/resource selalu di aplikasi. CLI/komputasi lokal tidak mendapat auth server. GitHub App memakai installation token dan verifikasi webhook; bukan pengganti login pengguna lintas platform. Capability link bertoken acak, tersimpan hash, terbatas scope/expiry dan dapat dicabut; bukan akun.
- **Data:** setiap schema domain hanya mempunyai satu pemilik migrasi. `Drizzle/pg/Kit` berarti Drizzle + node-postgres + migrasi SQL Drizzle Kit yang direview; `Drizzle/libSQL/Kit` memakai `@libsql/client/node`. `pgx†/goose†` berarti pool/query Go pgx dan CLI migrasi SQL goose; sqlc hanya ditambahkan pada query domain terstruktur. `SQLx†` mencakup driver PostgreSQL dan migrator SQLx. Migrator queue River/pg-boss memiliki schema internal terpisah, diurutkan oleh satu pemilik deployment, tidak ikut membuat ulang tabel domain.
- **ClickHouse:** HTTP parameterized queries; goose† khusus DDL idempotent non-transaksional pada instance standalone. Bukan migrator cluster yang sudah terbukti. PostgreSQL tambahan pada 111/117 hanya konfigurasi/otorisasi, bukan pengganti DB analitik utama.
- **Object storage:** setiap penyebutan S3 berarti **AWS S3 sebagai kandidat cloud dalam kelas palette S3-compatible**, memerlukan approval biaya. Filesystem hanya untuk demo lokal, tidak membuktikan kompatibilitas S3. Signing/upload memakai SDK sesuai runtime yang kelak diaudit, bukan mewarisi binding bahasa lain. Retensi, lifecycle, backup/restore dan penghapusan berlaku juga pada salinan/cadangan.
- **Packaging:** pnpm workspace untuk interface/worker TS, ditambah toolchain native bahasa masing-masing; tidak menambah task runner. Subprocess memakai kontrak JSON berversi, batas ukuran/waktu dan cancellation. Subprocess bukan sandbox keamanan. Binding/driver alternatif disebut eksplisit; semantik akses dan storage eksternal boleh diwarisi.
- **Ingest kategori 4:** HTTP/API/feed berizin dahulu; Crawlee/Playwright Node hanya bila rendering diperlukan, terisolasi dari core dan tanpa akses DB domain. Core menerima envelope tervalidasi berisi URL, waktu ambil, versi parser, hash dan lisensi. NATS pada tabel berarti JetStream untuk pekerjaan durable; ack sesudah commit, event-id unik, retry terbatas, karantina kegagalan. Outbox disebut eksplisit jika publish harus atomik terhadap perubahan DB; bukan klaim transaksi lintas DB/broker.

### Gate kategori 1 dan 4

1. **Izin mendahului data:** kategori4 memerlukan matriks sumber, robots/ToS, lisensi penggunaan ulang, retensi dan hak ekspor. Khusus417, stop bila izin menyimpan/mengekspor lead tidak eksplisit; API berbayar atau place ID tidak menyelesaikannya. 405/414 tetap kondisional;404 tidak menduplikasi712 jika merge disetujui.
2. **Boundary yang harus dibuktikan:** alt101 memakai DB outbox; primary402 serta alt401/414 memakai outbox+JetStream. Buktikan crash sesudah commit/sebelum publish, redelivery, dan idempotensi notifikasi. Migrator domain tetap satu owner; queue internal dipisahkan. TimescaleDB+River dan migrasi ekstensi perlu pemeriksaan kompatibilitas, bukan asumsi karena sama-sama PostgreSQL.
3. **Tidak ada pewarisan runtime palsu:** alt116 benar-benar Go graph/CLI/WASM; alt411 menjalankan Rust lewat subprocess, bukan napi-rs Go. Gate utama: cancellation, batas buffer, error propagation, target binary/ABI dan kontrak angka. Driver dan migrator alternatif menggantikan yang lama, bukan keduanya menulis schema domain.
4. **Keamanan tool developer:**106 harus menolak provider tanpa endpoint nama-key-only;102/107 tidak mengeksekusi perubahan ke produksi;103/108 menolak ref arbitrer;114/115 butuh isolasi snippet/agent nyata. Scope awal114 pada snippet OCaml dan107 pada PostgreSQL adalah **ASUMSI scope**, harus dikonfirmasi dalam PRD, bukan pembatasan brief yang sudah disetujui.
5. **Model dan dokumen:** PDF scan/OCR, XBRL taxonomy/context, korpus regulasi dan benchmark valuasi belum tervalidasi. PDF extraction bukan OCR; parsing XML bukan pembuktian angka akuntansi; regresi baseline bukan klaim harga pasar. Bila komponen† ditolak, revisi desain sebelum coding, jangan mengganti diam-diam.
6. **Auth dan data sensitif:** audit adapter Better Auth per-runtime; Keycloak memakai code+PKCE dan validator token yang dirawat sesuai bahasa, bukan crypto buatan. Binding validator OIDC belum dipatok versinya. Tenant isolation, rotasi credential, capability expiry/revocation, redaksi telemetry dan restore backup wajib punya skenario nyata.
7. **117 harus membenarkan build:** diferensiasi terhadap produk yang ada, provider HTTP streaming, cancellation, usage hilang/duplikat dan rekonsiliasi biaya. Connect hanya control API; jangan memperlakukan protokol provider sebagai RPC Connect. PostgreSQL konfigurasi+ClickHouse+NATS menambah biaya/operasi meski tidak ada harga yang dikarang.
8. **Biaya dan operasi:** batas brief bukan bukti seluruh layanan muat di bawah budget. Ukur volume/retensi, node ISP112, crawl/rendering, broker, Keycloak, LLM, S3/egress dan provider sebelum meminta approval pengeluaran. Tidak ada akun, install, test aplikasi, crawling, commit, provisioning atau deployment dilakukan dalam pekerjaan dokumen ini.

## Sumber kontrak bersama

- [pnpm workspaces](https://pnpm.io/workspaces): dukungan workspace/package linking di satu repository; bukan pengganti toolchain native.
- [Keycloak OIDC](https://www.keycloak.org/securing-apps/oidc-layers): authorization, token, discovery, client types dan grant; password grant tidak dipilih.
- [Keycloak database](https://www.keycloak.org/server/db): penyimpanan user/client/realm di database relasional; PostgreSQL adalah pilihan, database dev bukan target produksi.
- [Better Auth database](https://www.better-auth.com/docs/concepts/database): adapter, schema generation, dan batas migrasi built-in dibanding adapter ORM.
- [OpenTelemetry language SDKs](https://opentelemetry.io/docs/languages/): kematangan berbeda per bahasa dan sinyal; bukan bukti instrumentation semua app sudah berjalan.
- [Grafana Docker LGTM](https://grafana.com/docs/opentelemetry/docker-lgtm/): lingkungan observability development/demo/testing, bukan produksi siap pakai.
- [Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html): object/bucket, akses, lifecycle, region dan biaya berbasis penggunaan.
- [TanStack Router](https://tanstack.com/router/latest/docs/framework/react/overview) dan [Query](https://tanstack.com/query/latest/docs/framework/react/overview): navigasi React dan remote server state; cache bukan transaksi bisnis.
- [Solid Router](https://docs.solidjs.com/solid-router) dan [svelte-spa-router](https://github.com/ItalyPaleAle/svelte-spa-router): pilihan router masing-masing framework; yang terakhir berbasis hash dan mendokumentasikan Svelte5.
- [Expo Router](https://docs.expo.dev/router/introduction/): file-based navigation dan deep linking untuk Expo native; tidak memverifikasi integrasi perangkat.
- [PowerSync Sync Rules legacy](https://docs.powersync.com/sync/rules/overview): status deprecated dan arah Sync Streams untuk konfigurasi baru.
- [GPT-5.6 Terra](https://developers.openai.com/api/docs/models/gpt-5.6-terra) dan [GPT-Transcribe](https://developers.openai.com/api/docs/models/gpt-transcribe): kandidat API/model yang disebut; status dan endpoint dibaca ulang secara live, tanpa memanggil inference.
- [GPT-4.1 mini](https://developers.openai.com/api/docs/models/gpt-4.1-mini), [Whisper hosted](https://developers.openai.com/api/docs/models/whisper-1) dan [panduan transkripsi](https://developers.openai.com/api/docs/guides/speech-to-text): kandidat lama berstatus deprecated; contoh timestamp/subtitle lama bukan janji untuk model pengganti.
- [whisper.cpp CLI](https://raw.githubusercontent.com/ggml-org/whisper.cpp/master/examples/cli/README.md) dan [model files](https://github.com/ggml-org/whisper.cpp/blob/master/models/README.md): opsi output SRT/VTT/JSON dan model `base` multilingual. Ini runtime/model lokal, bukan endpoint deprecated `whisper-1`.

Sumber di setiap kelompok di bawah memperinci pilihan driver, framework, media, native dan model. Membaca dokumentasi memverifikasi kemampuan tertentu, bukan kompatibilitas kombinasi versi, akurasi model, keamanan, performa atau biaya nyata.

### Sumber kategori 1 dan 4

Daftar ini memverifikasi kapabilitas komponennya, **bukan kompatibilitas gabungan atau versi produksi**. Tautan `latest`/`CURRENT`/branch dokumentasi bergerak dan perlu dipatok ulang pada audit versi.

- [pgx](https://github.com/jackc/pgx): driver/pool PostgreSQL Go, COPY dan transaksi; [goose](https://github.com/pressly/goose): CLI migrasi SQL, PostgreSQL/ClickHouse, anotasi non-transaksional; [batas ClickHouse goose](https://github.com/pressly/goose/blob/main/doc/dialect-clickhouse.md): DDL harus idempotent, standalone dipilih, cluster memiliki batas tambahan.
- [SQLx](https://docs.rs/sqlx/latest/sqlx/): driver/pool PostgreSQL, query dan modul migrasi Rust; [Drizzle PostgreSQL](https://orm.drizzle.team/docs/get-started-postgresql): node-postgres/postgres.js; [Drizzle Turso](https://orm.drizzle.team/docs/connect-turso): libSQL Node mendukung file dan koneksi remote; [Drizzle Kit](https://orm.drizzle.team/docs/kit-overview): generate dan apply migrasi SQL, bukan wajib schema push.
- [next.jdbc](https://cljdoc.org/d/com.github.seancorfield/next.jdbc/CURRENT/doc/getting-started): Clojure datasource/transaksi dengan dependency JDBC eksplisit; [pgJDBC](https://jdbc.postgresql.org/documentation/use/): koneksi JVM PostgreSQL; [Npgsql](https://www.npgsql.org/doc/index.html): driver PostgreSQL ADO.NET dan query async; [crystal-pg](https://github.com/will/crystal-pg): driver melalui crystal-db; [Pog](https://hexdocs.pm/pog/): client PostgreSQL Gleam berbasis PGO dan supervised pool.
- [MongoDB Java](https://www.mongodb.com/docs/drivers/java/sync/current/) dan [MongoDB Rust](https://www.mongodb.com/docs/drivers/rust/current/): driver resmi masing-masing runtime; bukan migrator domain turnkey. Migrasi dokumen/checkpoint aplikasi adalah keputusan desain, masih perlu spike.
- [ClickHouse HTTP](https://clickhouse.com/docs/interfaces/http): SELECT/INSERT, query parameters dan autentikasi HTTP; [reqwest](https://docs.rs/reqwest/latest/reqwest/): client HTTP Rust async/blocking dan redirect policy. Tidak memakai SQLx untuk ClickHouse.
- [Better Auth/Hono](https://www.better-auth.com/docs/integrations/hono): handler dan session pada server yang ada; [Keycloak OIDC](https://www.keycloak.org/securing-apps/oidc-layers): discovery, code flow, JWKS/introspection; [GitHub App auth](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/about-authentication-with-a-github-app): membedakan app, installation, dan user token. Adapter Better Auth selain Hono masih wajib diperiksa pada spike runtime.
- [Encore SQLDatabase](https://encore.dev/docs/ts/primitives/databases): query API dan file migrasi yang dimiliki Encore; [raw endpoints](https://encore.dev/docs/ts/primitives/raw-endpoints): request/response Node tingkat rendah. Ini **belum membuktikan adapter Better Auth lengkap**; karena itu408 memilih Keycloak, bukan menyatakan integrasi auth otomatis.
- [River transactional enqueue](https://riverqueue.com/docs/transactional-enqueueing): job dan perubahan PostgreSQL dapat berada dalam satu transaksi; [NATS JetStream](https://docs.nats.io/nats-concepts/jetstream): persistence/replay, ack/redelivery, batas retensi; bukan transaksi atomik PostgreSQL+NATS.
- [Connect](https://connectrpc.com/docs/introduction/): Go/TS generated clients, browser dan protokol RPC; [Go WASM](https://go.dev/wiki/WebAssembly): target js/wasm dan kecocokan compiler/wasm_exec; [wasm-bindgen](https://wasm-bindgen.github.io/wasm-bindgen/): interop Rust/JS dan binding TS; [napi-rs](https://napi.rs/docs/introduction/getting-started): addon Node-API, bukan bridge Go.
- [pg_query Rust](https://docs.rs/pg_query/latest/pg_query/) dan [pg_query_go](https://github.com/pganalyze/pg_query_go): parser dari source PostgreSQL, dengan build C/CGO; [Swagger Parser](https://apidevtools.com/swagger-parser/): validate/dereference; [kin-openapi](https://github.com/getkin/kin-openapi): load/validate OpenAPI Go. Tidak menganggap seluruh dialect/version identik.
- [OpenFeature provider](https://openfeature.dev/docs/reference/concepts/provider/): kontrak provider evaluasi dan SDK, bukan server flag siap pakai; [pdf-extract](https://docs.rs/pdf-extract/latest/pdf_extract/): ekstraksi teks PDF; [quick-xml](https://docs.rs/quick-xml/latest/quick_xml/): parser XML pull-based, bukan validator akuntansi XBRL atau OCR.
- [linfa-linear](https://docs.rs/linfa-linear/latest/linfa_linear/): regresi linear native Rust; [Gonum mat](https://pkg.go.dev/gonum.org/v1/gonum/mat): operasi matriks/QR native Go; tidak menjamin model valuasi akurat atau serialisasi lintas library.
- [S3 presigned URL](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ShareObjectPreSignedURL.html): akses object berbatas waktu sesuai izin signer, bukan account system; [Google Places policies](https://developers.google.com/maps/documentation/places/web-service/policies): pembatasan ekstraksi/cache/redistribusi; pengecualian place ID bukan izin mengekspor nama/alamat bisnis sebagai lead.
