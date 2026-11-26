# FLOP Network Bot — `flop-bot` v1.0.3

Bot fleet untuk **FLOP Network** (`@flop_labs`) — L1 proof-of-inference milik Arthur Hayes
(BitMEX). Fokus pada satu-satunya lane yang tidak butuh GPU dan tidak butuh modal:
**agent** (24% genesis pool = 596.030.400 FLOP di yellow paper v0.5).

> Repo ini showcase. Source code lengkap ada di repo private, akses lewat lisensi.

> **v1.0.3 (26 Nov 2026):** gateway testnet sudah terbit, jadi bot sekarang punya **dua lane
> faucet** (devnet `POST /fund` 25 FLOP tanpa signature, testnet D10 signed claim 10 FLOP/24 jam)
> dan gate cutover yang menuntut katalog model tidak kosong — bukan sekadar host yang menjawab.

## Bukti jalan (bukan klaim)

Output apa adanya dari mesin sendiri:

```
# 26 Nov 2026 — watcher, gate bekerja apa adanya
$ node cli.js testnet
testnet profile: phase=testnet ready=false rpc=FLOP Testnet faucet=200 gateway=200 models=0
BLOCKED: testnet gateway is live but has no registered model yet
TESTNET infra reachable but inference not usable yet. Still on devnet.

# 26 Nov 2026 — satu wallet, jalur nyata (bukan dry-run)
$ node cli.js daily --wallet flop02 --force
flop02 5EeEV61U…KkkE balance 25
   faucet       skipped=true reason=balance 25 >= 20 (grant 25)
   enroll       skipped=true reason=gateway not enrollable: models=2 enroll=false
   infer        skipped=true reason=bearer dead and enrollment closed (enroll 503) — needs issuer renewal
   did-presence ok=true status=200 did=did:key:z6MkvmYBCcV9rbzbXnbLhkzMxn8KB6XeCobZEXsuVshnYysx registry=200

# 24 Nov 2026 — bukti lane agent pernah penuh end-to-end (3/3 wallet)
$ node cli.js daily --wallet flop03 --force
   enroll       ok=true status=200 model=qwen/qwen3-14b quota=100
   infer        ok=true status=200 chars=251 usage={... cost_flop:0.0011825}
   infer        ok=true status=200 chars=62  usage={... cost_flop:0.0007385}
   did-presence ok=true status=200 did=did:key:z6Mkw46c9...
```

Tabel status yang jujur per 26 Nov 2026 — **jangan dibaca sebagai "bot rusak"**, ini kondisi platform:

| Bagian | Status 26 Nov | Sebab |
|---|---|---|
| Faucet devnet `POST /fund` | **hidup** | 25 FLOP/address/24 jam |
| Faucet testnet signed claim | **hidup**, signature kita **diterima** | ditahan `ip_cooldown` (429), bukan `claim_signature_invalid` |
| RPC testnet | **hidup** | genesis `0x918abd58…cfd800` cocok dengan pin resmi |
| Gateway testnet | **hidup** (`FLOP REST Gateway v2.0.0`) | `/v1/models` masih `[]`, `/v1/enroll/*` 404 |
| Enrollment gateway devnet | **503** `wallet_enrollment_unavailable` | issuer menutup lane; 43 issue terbuka di `flop-labs/network` |
| Inference berbayar | **401** (bearer mati 25 Nov 02:08Z) | tidak bisa diperpanjang sendiri dari sisi klien |
| Status testnet | **unscored** | halaman resmi: aktivitas sebelum blok aktivasi diumumkan tidak bernilai |

Artinya: **tidak ada satu pun jalur yang menghasilkan alokasi hari ini**, dan itu tertulis di
dokumentasi FLOP — bukan kesimpulan bot. Yang bot lakukan sekarang adalah menjaga fleet siap
(faucet jalan, DID presence jalan, watcher menyalak begitu model testnet terdaftar).

## Yang dikerjakan bot ini

| Modul | Isi |
|---|---|
| Wallet fleet | sr25519, SS58 prefix 42, mnemonic dienkripsi AES-256-GCM (scrypt) |
| Faucet **dua lane** | devnet `POST /fund` (25 FLOP, tanpa signature) · testnet D10 signed claim (`flop-faucet-agent/v1\|<genesis>\|<ss58>\|<nonce>`, 10 FLOP/24 jam). Lane dipilih dari `/chain`, bukan dari nama host |
| Gateway enroll | challenge → signature sr25519 → bearer key model-scoped |
| Agent inference | `chat/completions` berbayar dengan pool prompt riset, bukan filler |
| Technocore DID | `did:key` Ed25519 per wallet, `say-signed` terverifikasi server + registry `kv/did` |
| Testnet watcher | flip hanya kalau RPC testnet + genesis pin cocok + faucet menjawab + **katalog model tidak kosong**; tulis `network-state.json`, exit code 10 untuk notifikasi cron |
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
- **Testnet sudah buka 24 Nov 2026 tapi masih unscored.** Infrastrukturnya hidup (RPC, faucet,
  gateway), model inference-nya **belum terdaftar**. Mainnet + TGE Q1 2027.
- **Cutover otomatis dan konservatif.** Watcher tidak flip hanya karena host menjawab; ia menunggu
  katalog model tidak kosong. Tanpa syarat itu bot akan pindah ke testnet lalu setiap `infer`
  menabrak 404 model.
- **Dua protokol faucet itu beda, dan ini jebakan paling mahal.** Lane `anonymous` 25 FLOP milik
  **devnet**; testnet `agent` cuma 10 FLOP/24 jam dengan `rolloverRequiresSpend: true`.
  `POST /fund` di host testnet **404**. Bot memilih lane dari jawaban `/chain`.
- **Enrollment testnet belum ada route-nya** (`/v1/enroll/*` → 404). Kalau nanti cutover, auth-nya
  bukan enroll lama — kandidatnya broker key / `flop agent credential`.
- Satu IP tetap satu cluster di mata reviewer. Bot menjaga jumlah wallet kecil, bukan
  menambah volume.

## Harga

**Rp 75.000** — satu kali bayar, source lengkap, lisensi per user, update versi minor gratis.

Order & katalog: [@OG_LYID_bot](https://t.me/OG_LYID_bot) · Channel [@layerairdrop](https://t.me/layerairdrop)

---

Tidak ada afiliasi dengan Flop Labs. NFA. DYOR.
