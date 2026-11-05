# Public strategy performance site (read-only)

Static site that shows **accounts ①–⑭ / #1–#14** strategy PnL since onboard.

- **No API keys, UIDs, account names, MCP ids, or exchange credentials**
- **Read-only**: the page cannot place orders
- Auto-update = overwrite `data.json` (HTML stays put)

## What’s on the page

| Block | Content |
| --- | --- |
| KPI | Total equity, principal, net PnL, return %, strategy count, strategy net U |
| Per-account | Ordinal only: onboard day, equity, principal, PnL, return %, strategy W/L |
| History | Open/close time (Asia/Shanghai), symbol, side, open/close price, PnL |
| Notes | Public methodology + disclaimer |

## Local preview

```bash
cd /workspace/public-strategy-site
python3 -m http.server 8765
# open http://127.0.0.1:8765/
```

`file://` will not load `data.json` (browser CORS); use any static HTTP server.

## Refresh data (nightly / after new snapshot)

```bash
/workspace/public-strategy-site/refresh-public-data.sh
```

Optional overrides:

```bash
SRC_JSON=/path/to/bitget-1-14-strategies-since-onboard-YYYY-MM-DD.json \
PNL_JSON=/path/to/account-since-onboard-pnl-latest.json \
/workspace/public-strategy-site/refresh-public-data.sh
```

The script **only reads** existing onboard JSON snapshots and writes scrubbed `data.json`.
It does **not** call trading write APIs. Parent agent / ops should produce the onboard JSON separately, then run this script, then publish the folder (or just the updated `data.json`).

## Deploy for a free public URL (no custom domain)

### Option A — Cloudflare Pages (recommended)

1. Create a GitHub/GitLab repo containing **only** this folder’s contents (`index.html`, `data.json`, `README.md`).
2. Cloudflare Dashboard → **Workers & Pages** → **Create** → **Pages** → connect the repo.
3. Build settings: **Framework preset = None**, build command empty, **output directory = `/`** (repo root = site root).
4. Deploy → you get `https://<project>.pages.dev`.

One-liner after `wrangler` login (if using CLI):

```bash
cd /workspace/public-strategy-site && npx --yes wrangler pages deploy . --project-name=strategy-perf-public
```

### Option B — GitHub Pages

```bash
# from a fresh repo whose root is this site
git init && git add index.html data.json README.md && git commit -m "public strategy site"
# push to GitHub, then Settings → Pages → Deploy from branch main / root
```

URL shape: `https://<user>.github.io/<repo>/`

### Option C — Vercel

```bash
cd /workspace/public-strategy-site && npx --yes vercel --yes
```

URL shape: `https://<project>.vercel.app`

## Nightly auto-update pattern

1. Ops/routine rebuilds onboard snapshot JSON under `/workspace/` (read-only equity OK).
2. Run `refresh-public-data.sh` → updates `data.json`.
3. Re-publish **only** `data.json` (or the whole static folder) to Pages/Vercel.
4. Visitors always fetch fresh `data.json` (`cache: no-store` in the page).

Never commit `.env`, API keys, MCP tokens, or internal account maps into the public repo.

## Out of scope

- Account ⑮
- Live order placement
- Discord posting (handled elsewhere)
- Creating Cursor routines (parent agent does that)
