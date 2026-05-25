# SQUILL Signal Cycle — GitHub Actions Cron

Repository publik ini bertugas sebagai **scheduler** untuk menjalankan signal cycle SQUILL,
menggantikan keterbatasan Vercel Free Plan (max 60s per request, 1x/hari cron).

---

## Cara Kerja

```
GitHub Actions (cron tiap 30 menit)
        │
        ▼
Job 1 — get-symbols
  POST /api/write2earn
  { action: "get_active_symbols" }
  ← Supabase: aggregate symbols dari semua wallet aktif
  → output: ["BTCUSDT", "ETHUSDT", "SOLUSDT", ...]
        │
        ▼
Job 2 — run-signal (matrix, max 5 paralel)
  Untuk setiap symbol secara paralel:
  POST /api/write2earn
  { action: "run_signal_symbol", symbol: "BTCUSDT" }
  ← Vercel runs: TA engine + AI validate (~15-25s per symbol)
  → Telegram signal / Binance Square / WunderTrading auto exec
  → Retry otomatis hingga 2x jika gagal (timeout / cold start)
        │
        ▼
Job 3 — summary
  Tulis hasil ke GitHub Step Summary
```

**Kenapa per-symbol dan bukan satu request?**

`run_signal_symbol` memproses tepat 1 symbol per request (~15-25 detik),
jauh di bawah limit 60 detik Vercel Hobby. Dengan matrix paralel,
10 symbols selesai dalam ~25 detik, bukan 250 detik.

**Guard duplikat:** Meski symbols diproses paralel, SQUILL punya
`_hasOpenPosition()` dan `_isCooldown()` di level database —
sinyal yang sama tidak akan terkirim dua kali. Ditambah `concurrency`
di workflow yang mencegah dua cycle berjalan bersamaan.

---

## Setup (sekali saja)

### 1. Fork / Create repo ini sebagai Public

Repo harus **Public** agar mendapat GitHub Actions gratis unlimited.

### 2. Tambahkan Secrets

Buka: `Settings → Secrets and variables → Actions → New repository secret`

| Secret Name | Value |
|---|---|
| `SQUILL_API_URL` | URL Vercel kamu, misal `https://squill.vercel.app` |
| `SIGNAL_SECRET` | Nilai dari env var `SIGNAL_SECRET` di Vercel |

> Secrets tidak akan terlihat di log meskipun repo ini publik.

### 3. Enable GitHub Actions

Buka tab `Actions` → klik **"I understand my workflows, go ahead and enable them"**

### 4. Test manual

Buka `Actions → SQUILL Signal Cycle → Run workflow`

Atau dengan symbol tertentu: isi input `symbol` dengan `BTCUSDT`.

---

## Jadwal Cron

Default: **setiap 30 menit** (`*/30 * * * *` UTC)

Ubah di `.github/workflows/signal-cycle.yml` sesuai kebutuhan:

```yaml
# Setiap 30 menit
- cron: '*/30 * * * *'

# Setiap jam
- cron: '0 * * * *'

# Setiap 2 jam
- cron: '0 */2 * * *'

# 4x sehari (jam 0, 6, 12, 18 UTC)
- cron: '0 0,6,12,18 * * *'
```

> ⚠️ GitHub Actions bisa mengalami delay 10–30 menit pada jam sibuk.

---

## Fitur Workflow

| Fitur | Detail |
|---|---|
| **Matrix paralel** | Max 5 symbols sekaligus, `fail-fast: false` |
| **Retry otomatis** | Hingga 2x per symbol, jeda 10 detik antar percobaan |
| **Concurrency guard** | Satu cycle aktif sekaligus — run baru diqueue, tidak cancel |
| **Manual trigger** | Bisa dispatch dari GitHub UI dengan symbol tertentu |
| **Step Summary** | Hasil tiap run tampil di tab Summary Actions |

---

## Keamanan

- `SIGNAL_SECRET` divalidasi server-side — request tanpa secret yang benar ditolak (HTTP 401)
- Log Actions bisa dilihat publik, tapi **tidak ada data sensitif yang diprint**
- Kode SQUILL tetap di repo privat
- Secrets di-mask otomatis oleh GitHub bahkan jika tidak sengaja diprint

---

## Troubleshooting

**Job `get-symbols` mengembalikan fallback `["BTCUSDT","ETHUSDT","BNBUSDT"]`**
→ SQUILL API tidak bisa diakses atau `SQUILL_API_URL` salah. Cek di tab Actions → log step "Fetch active symbols".

**Job `run-signal` gagal setelah 2 attempts**
→ Cek response body di log — bisa timeout Binance, Vercel cold start, atau symbol tidak valid.

**Cycle baru ditampilkan sebagai "queued" bukan langsung running**
→ Normal — concurrency guard aktif karena run sebelumnya masih berjalan.
