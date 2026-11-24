# FLOP Network Bot — `flop-bot` v1.0.2

Bot fleet untuk **FLOP Network** (`@flop_labs`) — L1 proof-of-inference milik Arthur Hayes
(BitMEX). Fokus pada satu-satunya lane yang tidak butuh GPU dan tidak butuh modal:
**agent** (24% genesis pool = 596.030.400 FLOP di yellow paper v0.5).

> Repo ini showcase. Source code lengkap ada di repo private, akses lewat lisensi.

> v1.0.2 (24 Nov 2026): enrollment gateway balik `200` — bearer key lane agent terbukti
> end-to-end untuk 3/3 wallet; crash WASM saat signing dan inference `400 unsupported_capability`
> sudah diperbaiki.

## Bukti jalan (bukan klaim)

Output apa adanya dari mesin sendiri, 24 Nov 2026:

```
$ node cli.js status
label   FLOP   key   did   infer/day
flop01  25     yes   yes   1
flop02  25     yes   yes   1
flop03  25     yes   yes   2

$ node cli.js daily --wallet flop03 --force
   enroll       ok=true status=200 model=qwen/qwen3-14b quota=100
   infer        ok=true status=200 chars=251 usage={... cost_flop:0.0011825}
   infer        ok=true status=200 chars=62  usage={... cost_flop:0.0007385}
   did-presence ok=true status=200 did=did:key:z6Mkw46c9...
```

Yang belum terbukti saya tulis apa adanya: **testnet belum rilis** (domain
`faucet.testnet.flop.network` / `gateway.testnet.flop.network` masih NXDOMAIN saat dicek 24 Nov),
dan apakah inference berbayar dihitung untuk cohort agent masih **UNVERIFIED** — kriterianya belum
dipublikasikan Flop Labs.

## Yang dikerjakan bot ini

| Modul | Isi |
|---|---|
| Wallet fleet | sr25519, SS58 prefix 42, mnemonic dienkripsi AES-256-GCM (scrypt) |
| Faucet | `POST faucet.flop.network/fund` → 25 test FLOP, retry mengikuti `Retry-After` |
| Gateway enroll | challenge → signature sr25519 → bearer key model-scoped |
| Agent inference | `chat/completions` berbayar dengan pool prompt riset, bukan filler |
| Technocore DID | `did:key` Ed25519 per wallet, `say-signed` terverifikasi server + registry `kv/did` |
| Testnet watcher | deteksi cutover devnet → testnet dari CLI resmi, tulis `network-state.json`, run berikutnya otomatis pakai RPC/faucet/gateway testnet, exit code 10 untuk notifikasi cron |
| Status | tabel fleet: saldo, key, DID, aktivitas hari ini |

## Humanisasi (biar tidak kelihatan satu skrip)

- slot harian **deterministik per wallet** dari `sha256(label|tanggal)` — retry jatuh di slot
  yang sama, bukan kunjungan kedua
- persona tetap per wallet: jam aktif, tempo 25–95 detik, selera jumlah aksi
- skip-today 5–13%; ada hari wallet memang diam
- cap harian: 1–2 inference, 1 baris presence
- teks tidak pernah identik antar wallet (room Technocore menolak duplikat)

## Fakta proyek yang perlu diketahui sebelum beli

- **Devnet tidak bernilai dan tidak dihitung airdrop** — ini tertulis di dokumentasi resmi.
  Bot ini menyiapkan fleet, bukan mencetak token.
- Testnet: target awal 24 November 2026, **belum rilis** per 24 Nov (domain testnet masih NXDOMAIN).
  Mainnet + TGE Q1 2027.
- **Cutover otomatis.** Watcher membaca profil testnet dari `@flop-labs/cli`; begitu live,
  `daily` run berikutnya langsung di endpoint testnet tanpa konfigurasi manual.
- **Enrollment gateway terbuka sejak 24 Nov 2026**: `POST /v1/enroll/wallet` → `200` dengan
  bearer key (`qwen/qwen3-14b`, quota 100 FLOP, kedaluwarsa ~24 jam). Sebelumnya sempat
  `503 wallet_enrollment_cap_reached`; bot mencoba ulang tiap tick dan menyimpan key otomatis.
- Satu IP tetap satu cluster di mata reviewer. Bot menjaga jumlah wallet kecil, bukan
  menambah volume.

## Harga

**Rp 75.000** — satu kali bayar, source lengkap, lisensi per user, update versi minor gratis.

Order & katalog: [@OG_LYID_bot](https://t.me/OG_LYID_bot) · Channel [@layerairdrop](https://t.me/layerairdrop)

---

Tidak ada afiliasi dengan Flop Labs. NFA. DYOR.
