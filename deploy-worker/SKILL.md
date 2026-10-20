---
name: deploy-worker
description: Use when user says "deploy forms", "deploy admin-api", "deploy WORKER", "ship it", or wants to push a Cloudflare worker to production.
---

# Deploy Cloudflare Worker

## Deploy a Specific Worker
```bash
cd workers/WORKER_NAME && bws run -- npx wrangler deploy
```

## Active Workers
- `altruist` — Custodian OAuth + account sync
- `data-api` — Public API for website (ethicic.com)
- `phone` — Vonage voicemail + call routing
- `webhooks` — Inbound webhook router (GitHub, Zoho, Resend)
- `website` — Astro SSR site (ethicic.com)

## Pre-Deploy Checklist
1. Run tests: `cd workers/WORKER_NAME && npm test`
2. Check for TypeScript errors: `npx tsc --noEmit`
3. Review wrangler.toml for correct bindings
4. Deploy: `bws run -- npx wrangler deploy`
5. Check logs: `bws run -- npx wrangler tail`

## Tail Logs After Deploy
```bash
cd workers/WORKER_NAME && bws run -- npx wrangler tail
```

## Rollback
```bash
# List recent deployments
cd workers/WORKER_NAME && bws run -- npx wrangler deployments list
# Rollback to previous
cd workers/WORKER_NAME && bws run -- npx wrangler rollback
```

## Website Deploy (Special Case)
Website uses environment-specific deploys:
```bash
# Staging (auto-deploys on push to main via CI)
cd website && bws run -- npx wrangler deploy

# Production (manual)
cd website && bws run -- bash -c 'npx astro build && npx wrangler deploy --env production'
```

## For observability, binding inspection, metrics, and debugging
Use the `workers-observability` skill.

## Deprecated (do not use)

- `doppler run -- ...` deploy/tail commands — this repo uses `bws run --`
- Hardcoded `/Users/...` paths — every writing session runs in its own worktree,
  so absolute paths are wrong exactly where this skill is used. Paths here are
  relative to the repo root.
