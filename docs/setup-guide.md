# Setup Guide — SIGNAL_SECRET di Vercel

## 1. Generate SIGNAL_SECRET

Gunakan nilai random yang kuat. Bisa generate dengan salah satu cara:

```bash
# macOS / Linux
openssl rand -hex 32

# Online (jika tidak ada terminal)
# https://generate-secret.vercel.app/32
```

Contoh hasil: `a3f8c2e1d4b7091f6e5a2c8d3b1f7e4a9c6d2e8f1b3a5c7d9e0f2b4a6c8d0e2`

---

## 2. Tambahkan ke Vercel Environment Variables

1. Buka [vercel.com/dashboard](https://vercel.com/dashboard)
2. Pilih project SQUILL
3. Masuk ke **Settings → Environment Variables**
4. Tambahkan:

| Key | Value | Environment |
|---|---|---|
| `SIGNAL_SECRET` | (nilai yang kamu generate) | Production, Preview, Development |

5. Klik **Save**
6. **Redeploy** project agar env var aktif

---

## 3. Tambahkan ke GitHub Secrets

1. Buka repo publik cron ini
2. **Settings → Secrets and variables → Actions → New repository secret**
3. Tambahkan:

| Name | Value |
|---|---|
| `SQUILL_API_URL` | `https://nama-project-kamu.vercel.app` |
| `SIGNAL_SECRET` | (nilai yang sama seperti di Vercel) |

---

## 4. Verifikasi

Test manual dari GitHub Actions:
- Tab `Actions` → `SQUILL Signal Cycle` → `Run workflow`

Atau via curl:
```bash
curl -X POST "https://nama-project-kamu.vercel.app/api/write2earn" \
  -H "Content-Type: application/json" \
  -d '{"action":"run_signal_cycle","secret":"ISI_SECRET_KAMU"}'
```

Response yang diharapkan:
```json
{ "ok": true, "processed": 1, "results": [...] }
```
