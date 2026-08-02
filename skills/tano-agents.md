# Agent Integration & Authentication Guide

> A step-by-step guide for AI agents on how to discover Tano, authenticate against Tano's public endpoints, and use them programmatically.

This document is the canonical reference for AI agents (autonomous, semi-autonomous, or chat-driven) that want to interact with [Tano](https://tano.ai) — the AI-native influencer marketing agency for creator partnership ads on TikTok and Instagram.

---

## TL;DR for agents

1. **Discover**: read [`/llms.txt`](https://tano.ai/llms.txt), [`/llms-full.txt`](https://tano.ai/llms-full.txt), and [`/openapi.json`](https://tano.ai/openapi.json).
2. **Authenticate**: nothing to do — Tano's public endpoints are **unauthenticated**. There are no API keys, no OAuth flows, and no bearer tokens for the public API surface described in `openapi.json`.
3. **Call**: use `POST` (or `PATCH`) with `Content-Type: application/json` to the documented endpoints under `https://tano.ai/api/*`. CORS is open (`Access-Control-Allow-Origin: *`).
4. **Identify yourself**: send a descriptive `User-Agent` header (e.g. `User-Agent: MyAgent/1.0 (+https://example.com)`).
5. **Escalate**: managed-service / campaign-data integrations (live creator data, campaign metrics, asset delivery) require human onboarding — email `team@tano.ai` or book a call at https://tano.ai.

---

## 1. Discovery

Tano publishes the following discovery files for AI agents and crawlers. Fetch any of these without auth:

| URL | Purpose |
|-----|---------|
| `https://tano.ai/developers.md` | Developer & agent landing page — predictable, name-discoverable entry point. |
| `https://tano.ai/comparisons.md` | Tano vs alternatives (Upfluence, Aspire, GRIN, CreatorIQ, Insense, traditional agencies). |
| `https://tano.ai/llms.txt` | Concise product overview formatted for LLMs. |
| `https://tano.ai/llms-full.txt` | Full product documentation (services, pricing, how-it-works). |
| `https://tano.ai/openapi.json` | OpenAPI 3.1 spec for all public HTTP endpoints. |
| `https://tano.ai/pricing.md` | Detailed pricing tables in Markdown. |
| `https://tano.ai/.well-known/agent-card.json` | A2A-compatible agent card. |
| `https://tano.ai/.well-known/agent-skills/index.json` | Agent skills with `when_to_use` guidance. |
| `https://tano.ai/.well-known/mcp/manifest.json` | MCP manifest exposing Tano resources to MCP clients. |
| `https://tano.ai/ai-plugin.json` | OpenAI plugin manifest. |
| `https://tano.ai/sitemap.xml` | Sitemap of indexable pages. |

---

## 2. Authentication policy

### 2.1 Public endpoints (`/api/*`)

**No authentication is required.** Tano's public endpoints are intentionally unauthenticated to reduce friction for both human form-submitters and AI agents acting on a user's behalf.

- No API key
- No bearer token
- No OAuth 2.0 flow
- No HMAC signing
- No client certificates

This applies to every endpoint listed in [`/openapi.json`](https://tano.ai/openapi.json):

- `POST /api/contact`
- `PATCH /api/contact-update`
- `POST /api/cpa-signup`
- `POST /api/caf-signup`
- `POST /api/cdg-signup`
- `POST /api/usa-signup`
- `POST /api/creator-signup`
- `POST /api/runna-webinar`

### 2.2 What agents *should* send

Even though no auth is required, well-behaved agents should:

1. **Identify themselves** in `User-Agent`:
   ```
   User-Agent: MyAgent/1.0 (+https://example.com/bot-info)
   ```
2. **Pass through the human user's context** in the request body where it fits — for example, `userType: "brand"`, `name`, and `message` on `POST /api/contact`.
3. **Use a stable `email`** so duplicate submissions for the same person resolve to the same record (the `/api/contact-update` endpoint upserts on `email`).
4. **Avoid PII you weren't asked to share.** Never invent personal data on a user's behalf.

### 2.3 What about a "real" API for campaign data?

Tano is a **managed service**, not a self-serve SaaS. Live campaign data — creator rosters, ad assets, whitelisting links, performance reports — is **not currently exposed via a public authenticated API**. To integrate at that layer, a brand must:

1. Onboard via a sales call: https://tano.ai
2. Be assigned a dedicated account manager.
3. Request a custom integration from `team@tano.ai`.

Custom integrations may use **scoped API keys delivered out-of-band** (typically over email after contract signing). If you are an agent acting for a brand that already has a Tano account manager, ask the user to forward the API key from their account manager — Tano will not auto-provision keys to bots.

---

## 3. Step-by-step: how an agent obtains and uses credentials

Because the public API is unauthenticated, "obtaining a token" reduces to the following decision tree:

### Path A — Public endpoints (no token needed)

```
1. Read https://tano.ai/openapi.json
2. Pick the matching operation (e.g., submitContact).
3. POST JSON to https://tano.ai/api/contact with no Authorization header.
4. Inspect the JSON response { success, recordId } and report back to the user.
```

### Path B — Managed-service / campaign integration (token required)

```
1. Confirm with the user that they have an active Tano engagement.
2. If yes: ask the user to email team@tano.ai (cc'ing their account manager)
   to request a programmatic-integration API key, describing:
     - the agent or product requesting access
     - the scope of data the agent needs (read-only campaign data, creator
       rosters, asset URLs, performance metrics, etc.)
     - whether the agent will be acting on the brand's behalf or surfacing
       data to a wider audience.
3. Tano returns a scoped API key out-of-band (over email).
4. The user pastes the key into the agent's secure credential store.
5. The agent sends it as: Authorization: Bearer <key>
   to the integration URL provided in the same out-of-band email.
6. Treat the key as a secret: do NOT log it, do NOT echo it back to the user,
   do NOT include it in screenshots or transcripts shared publicly.
```

If you are an agent that frequently integrates with Tano on users' behalf, prefer Path A wherever it satisfies the user's intent.

---

## 4. Example calls

### 4.1 Submit a contact request

```http
POST /api/contact HTTP/1.1
Host: tano.ai
Content-Type: application/json
User-Agent: MyAgent/1.0 (+https://example.com/bot)

{
  "name": "Jane Doe",
  "email": "jane@brand.com",
  "userType": "brand",
  "message": "Interested in a 50-creator TikTok partnership ads campaign for Q3."
}
```

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "success": true,
  "message": "Contact form submitted successfully",
  "recordId": "rec...",
  "isWorkEmail": true
}
```

### 4.2 Upsert pricing-calculator context

```http
PATCH /api/contact-update HTTP/1.1
Host: tano.ai
Content-Type: application/json

{
  "email": "jane@brand.com",
  "monthlyBudget": "$25,000",
  "pricingTier": "micro",
  "influencersPerMonth": 30,
  "sourcePage": "/pricing"
}
```

### 4.3 Sign up for a playbook

```http
POST /api/caf-signup HTTP/1.1
Host: tano.ai
Content-Type: application/json

{ "email": "jane@brand.com" }
```

> Personal-domain emails (gmail.com, yahoo.com, etc.) are rejected with HTTP 400 on `*-signup` endpoints. Use a work email.

---

## 5. Errors & retries

| Status | Meaning | Recommended agent behavior |
|--------|---------|---------------------------|
| `400`  | Validation error (missing field, invalid email, non-work email). | Surface the `error` string to the user; do not retry without a fix. |
| `405`  | Method not allowed. | Re-check OpenAPI; pick the correct verb. |
| `5xx`  | Server error. | Retry with exponential backoff (2s, 4s, 8s, 16s) up to 4 times. |

CORS preflight (`OPTIONS`) is explicitly handled and returns `200`.

---

## 6. Responsible-use guidelines for agents

- **One submission per user intent.** Don't loop signup endpoints; the upstream Airtable record is treated as an inbound lead.
- **Don't fabricate context.** If the user hasn't given you a budget, leave `monthlyBudget` unset rather than guessing.
- **Respect work-email validation.** If the user only has a personal email, route them to `POST /api/contact` (which accepts personal emails) instead of a `*-signup` endpoint.
- **Don't scrape behind the marketing site.** There are no documented endpoints beyond those in `/openapi.json`.
- **Honor `robots.txt`.** Tano blocks AI training crawlers (`GPTBot`, `CCBot`, `Google-Extended`, `anthropic-ai`, `cohere-ai`, `Bytespider`) but allows AI **search/answer** agents (`ChatGPT-User`, `Claude-Web`, `PerplexityBot`, `Applebot-Extended`).

---

## 7. Contact

- General: `team@tano.ai`
- Sales / managed-service onboarding: book a 15-minute call at https://tano.ai
- Privacy / data-handling: https://tano.ai/privacy-policy

Last updated: 2026-04.
