# Deployment Checklist

Use this checklist before uploading `chain-monitor` to the target server / production environment.

## Runtime model

This project is primarily a Cloudflare Worker deployment, not a traditional always-on Node server.

Production pieces:
- Cloudflare Worker runtime
- Cloudflare D1 database
- Static assets from `public/`
- Secrets:
  - `RPC_URL`
  - `L1_RPC_URL`

## Required accounts / access

- Cloudflare account with Workers + D1 enabled
- Wrangler authenticated (`npx wrangler login`)
- Alchemy or equivalent RPC endpoints for:
  - Gensyn mainnet
  - Ethereum mainnet

## Pre-deploy checks

1. Install dependencies

```bash
npm install
```

2. Verify Wrangler config

- Check `wrangler.toml`
- Confirm the correct D1 `database_id`
- Confirm cron schedule is intended:

```toml
[triggers]
crons = ["*/2 * * * *"]
```

3. Create / verify D1 database

```bash
npx wrangler d1 create chain-monitor
```

4. Apply schema

```bash
npx wrangler d1 execute chain-monitor --remote --file=schema.sql
```

5. Set secrets

```bash
npx wrangler secret put RPC_URL
npx wrangler secret put L1_RPC_URL
```

6. Local verification

```bash
npm run dev
npm run dev:scheduled
curl "http://localhost:8787/api/data"
curl "http://localhost:8787/__scheduled"
```

Note: `__scheduled` only works when using `npm run dev:scheduled`.

## Production deploy

```bash
npx wrangler deploy
```

## Post-deploy validation

- Open the Worker URL in browser
- Confirm dashboard loads
- Confirm `/api/data` returns JSON
- Confirm Funding panel does not show negative Delphi TVL
- Confirm cron-backed Delphi data is filling D1
- Confirm `RPC_URL` and `L1_RPC_URL` secrets are set correctly

## If deploying to a traditional VPS

If you want to run this outside Cloudflare Workers, additional adaptation is needed because the current web app targets the Cloudflare Worker runtime and D1 bindings.

In that case prepare:
- a Node-compatible HTTP server wrapper, or
- a separate static frontend + API backend deployment model

At the moment, the repo is production-ready for Cloudflare Worker deployment first.