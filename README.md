# SQUILL Signal Cycle — GitHub Actions Cron

Repository publik ini bertugas sebagai **scheduler** untuk menjalankan signal cycle SQUILL setiap jam, menggantikan keterbatasan Vercel Free (hanya 1x/hari).

## Cara Kerja

```
GitHub Actions (cron tiap jam)
        ↓
  POST /api/write2earn
  { action: "run_signal_cycle", secret: "..." }
        ↓
  Vercel (SQUILL backend)
        ↓
  Proses semua wallet aktif
  → Telegram signal
  → Binance Square
```

**Tidak ada kode sensitif di repo ini.** Hanya satu file YAML yang memanggil API Vercel.

---

## Setup (sekali saja)

### 1. Fork / Create repo ini sebagai Public

Pastikan repo ini berstatus **Public** agar mendapat GitHub Actions gratis unlimited.

### 2. Tambahkan Secrets

Buka: `Settings → Secrets and variables → Actions → New repository secret`

| Secret Name | Value |
|---|---|
| `SQUILL_API_URL` | URL Vercel kamu, misal `https://squill.vercel.app` |
| `SIGNAL_SECRET` | Nilai dari env var `SIGNAL_SECRET` di Vercel |

> **Penting:** Secrets tidak akan terlihat di log meskipun repo ini publik.

### 3. Enable GitHub Actions

Buka tab `Actions` → klik **"I understand my workflows, go ahead and enable them"**

### 4. Test manual

Buka `Actions → SQUILL Signal Cycle → Run workflow` untuk test sebelum menunggu cron otomatis.

---

## Jadwal Cron

Default: **setiap jam** (`0 * * * *` UTC)

Ubah di `.github/workflows/signal-cycle.yml` sesuai kebutuhan:

```yaml
# Setiap jam
- cron: '0 * * * *'

# Setiap 2 jam
- cron: '0 */2 * * *'

# 4x sehari (jam 0, 6, 12, 18 UTC)
- cron: '0 0,6,12,18 * * *'

# Setiap 30 menit (GitHub mungkin delay)
- cron: '*/30 * * * *'
```

> ⚠️ GitHub Actions gratis dapat mengalami delay 10–30 menit pada jam sibuk.

---

## Keamanan

- `SIGNAL_SECRET` divalidasi di server — request tanpa secret yang benar akan ditolak (HTTP 401)
- Log Actions bisa dilihat publik, tapi **tidak ada data sensitif yang diprint**
- Kode SQUILL tetap di repo privat, tidak terekspos di sini
