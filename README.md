# FLOP Network Bot — `flop-bot` v1.0.0

Bot fleet untuk **FLOP Network** (`@flop_labs`) — L1 proof-of-inference milik Arthur Hayes
(BitMEX). Fokus pada satu-satunya lane yang tidak butuh GPU dan tidak butuh modal:
**agent** (24% genesis pool = 596.030.400 FLOP di yellow paper v0.5).

> Repo ini showcase. Source code lengkap ada di repo private, akses lewat lisensi.

## Yang dikerjakan bot ini

| Modul | Isi |
|---|---|
| Wallet fleet | sr25519, SS58 prefix 42, mnemonic dienkripsi AES-256-GCM (scrypt) |
| Faucet | `POST faucet.flop.network/fund` → 25 test FLOP, retry mengikuti `Retry-After` |
| Gateway enroll | challenge → signature sr25519 → bearer key model-scoped |
| Agent inference | `chat/completions` berbayar dengan pool prompt riset, bukan filler |
| Technocore DID | `did:key` Ed25519 per wallet, `say-signed` terverifikasi server + registry `kv/did` |
| Testnet watcher | deteksi cutover devnet → testnet, exit code 10 untuk notifikasi cron |
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
- Testnet: target 24 November 2026 (bisa geser), ±90 hari. Mainnet + TGE Q1 2027.
- Enrollment gateway devnet sedang menutup allowance (`503 wallet_enrollment_cap_reached`);
  bot mencoba ulang tiap hari dan menyimpan key otomatis saat terbuka.
- Satu IP tetap satu cluster di mata reviewer. Bot menjaga jumlah wallet kecil, bukan
  menambah volume.

## Harga

**Rp 75.000** — satu kali bayar, source lengkap, lisensi per user, update versi minor gratis.

Order & katalog: [@LYIDLicenseBot](https://t.me/LYIDLicenseBot) · Channel [@layerairdrop](https://t.me/layerairdrop)

---

Tidak ada afiliasi dengan Flop Labs. NFA. DYOR.
