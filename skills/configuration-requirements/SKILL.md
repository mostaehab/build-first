---
name: configuration-requirements
description: >
  Write or refresh CONFIGURATION_REQUIREMENTS.md — every API key, secret, token and
  account the built code needs, with the third-party service it comes from and the steps
  to get it. Use when a credential comes up mid-build, when the build is finished, or for
  "what keys does this need", "list the secrets", "configuration requirements",
  /configuration-requirements.
---

`CONFIGURATION_REQUIREMENTS.md` lives at the repo root. It is the single place a missing
credential gets recorded, so building never stops to ask for one. It holds instructions
for getting values — never the values themselves.

## Two ways in

**Mid-build, one new requirement:** append its entry. Do not rewrite the file, do not
reconcile the whole codebase, do not break stride — this is a few seconds of work.

**End of build (or when asked):** reconcile the whole file against the code, then hand it
over.

## Reconciling

1. Find every environment read in the code, including ones you did not write:

   ```
   grep -rniE "process\.env\.|import\.meta\.env\.|os\.environ|getenv|ENV\[|Deno\.env" \
     --include="*.{ts,tsx,js,jsx,mjs,py,go,rb,rs,java,kt,php,cs,sh,yml,yaml}" .
   ```

   Add `.env.example`, `.env.sample`, docker-compose `environment:` blocks, CI workflow
   `secrets.*`, and any deploy config (`vercel.json`, `fly.toml`, k8s manifests).
2. Every variable found gets an entry. Delete entries for variables no longer read.
3. Non-secret config (port, log level, feature flags) goes in a short second section — do
   not pad the credential list with it.
4. Check `.env.example` lists all of them with empty values, and that `.env` / `.env.local`
   is gitignored.
5. Do not guess a console's navigation path or a URL. If you are not sure where the key
   lives in that provider's dashboard today, look it up (find-docs / `ctx7`, or a web
   search) and write what you confirmed.

## The format

````markdown
# Configuration Requirements

The app is built. These are the credentials it needs before each feature works.
Put the values in `.env.local` (already gitignored) — never in the code, never in this file.

| # | Variable | Service | Needed for | Required |
|---|----------|---------|------------|----------|
| 1 | `STRIPE_SECRET_KEY` | Stripe | Checkout, subscriptions | Yes |
| 2 | `RESEND_API_KEY` | Resend | Receipt + password-reset email | Yes |
| 3 | `SENTRY_DSN` | Sentry | Error reporting | No |

---

## 1. `STRIPE_SECRET_KEY` — Stripe

- **Needed for:** checkout session + webhook verification — `src/billing/stripe.ts:14`,
  `src/api/webhooks/stripe.ts:8`
- **Without it:** `POST /api/checkout` throws on startup of the first request.
- **Value looks like:** `sk_test_…` while developing, `sk_live_…` in production (~107 chars)
- **How to get it:**
  1. Create an account or sign in at https://dashboard.stripe.com
  2. Leave test mode on for development.
  3. **Developers → API keys → Secret key → Reveal**
  4. Copy it into `.env.local` as `STRIPE_SECRET_KEY=`
- **Also needed:** `STRIPE_WEBHOOK_SECRET` (entry 4) — a separate value from
  **Developers → Webhooks → your endpoint → Signing secret**
- **Account requirements:** business details must be verified before live keys charge money;
  test keys work immediately.
- **Cost:** free account, 2.9% + $0.30 per successful charge.

## 2. `RESEND_API_KEY` — Resend
…
````

Per entry, in that order: what breaks without it, the shape of the value, numbered steps
through the real dashboard, anything else the same signup produces, account/verification
prerequisites, and cost. Skip a line only when it genuinely does not apply.

## Stubs and shortcuts

If a feature was stubbed while unconfigured, say so in its entry on one line: what the stub
returns, and that it disappears once the variable is set. Keep a short **Stubbed until
configured** list at the top of the file if there is more than one.

## Handing it over

End the build with the file path, the count of required vs optional credentials, which
features are dark until they are set, and an offer to wire the values in and run it. Never
paste a credential the user gives you into this file — put it in `.env.local` and tick the
checklist row.
