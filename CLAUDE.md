# demo-workflow – project context

Single GitHub Actions workflow: `.github/workflows/desco_balance_notifier.yml`
(daily 00:00 UTC = 6 AM BDT, plus manual `workflow_dispatch`). It fetches the DESCO prepaid
meter balance and notifies via Gmail (SMTP) and Discord webhook. The SMS step is commented out
on purpose; do not re-enable unless asked.

## Auth + balance (new portal, since 2026-11-08)
DESCO replaced the public balance endpoint. The workflow now:
1. `GET /api/auth/csrf` → `XSRF-TOKEN` cookie
2. `POST /api/auth/login` with `{"username","password"}` + `X-XSRF-TOKEN`
3. `GET /api/accounts` → pick linked account (filter by `DESCO_METER_NO` / `DESCO_ACCOUNT_NO`)
4. `GET /api/accounts/<accountId>/balance` → daily snapshot (`.balance`)

Prefer the daily snapshot over `POST .../realtime-balance` (rate-limited). Fail loudly if
balance is not numeric. Never commit or paste session cookies/passwords.

## Secrets used
DESCO_USER, DESCO_PASS, EMAIL_USER, EMAIL_PASS, DISCORD_WEBHOOK
(SMS_API_KEY only if SMS is re-enabled).

## Keeping context in sync
Cursor reads `.cursor/rules/project-context.mdc`, which mirrors this file. When one changes, update the other.
