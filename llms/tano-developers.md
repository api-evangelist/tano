# Tano for Developers & AI Agents

> Tano (https://tano.ai) is the AI-native influencer marketing partner. This page is the canonical entry point for developers and AI agents building against Tano.

If you are an AI agent, an LLM-powered application, or a developer looking for "Tano API", "Tano MCP server", "Tano developer documentation", or "Tano integration guide" — start here.

---

## At a glance

| What | Where |
|------|-------|
| Public API spec (OpenAPI 3.1) | https://tano.ai/openapi.json |
| MCP server (Streamable HTTP, JSON-RPC 2.0) | https://tano.ai/api/mcp |
| MCP manifest (with MCP Apps `ui://` resources) | https://tano.ai/.well-known/mcp/manifest.json |
| Agent authentication & integration guide | https://tano.ai/agents.md |
| A2A agent card | https://tano.ai/.well-known/agent-card.json |
| Agent skills index (with when-to-use guidance) | https://tano.ai/.well-known/agent-skills/index.json |
| OpenAI plugin manifest | https://tano.ai/ai-plugin.json |
| llms.txt (product summary for LLMs) | https://tano.ai/llms.txt |
| llms-full.txt (full product documentation for LLMs) | https://tano.ai/llms-full.txt |
| Pricing details (Markdown) | https://tano.ai/pricing.md |
| Comparisons (Tano vs alternatives) | https://tano.ai/comparisons.md |
| Sitemap | https://tano.ai/sitemap.xml |
| Robots policy (AI crawlers) | https://tano.ai/robots.txt |

All resources are unauthenticated and CORS-open.

---

## Tano API

Tano's public HTTP API is documented as an OpenAPI 3.1 specification at [`https://tano.ai/openapi.json`](https://tano.ai/openapi.json). It covers the contact and signup endpoints under `https://tano.ai/api/*`.

### Conventions

- **Transport**: HTTPS only.
- **Methods**: `POST` for submissions, `PATCH` for `contact-update`, `OPTIONS` for CORS preflight, `GET` only on `/api/mcp` (returns a JSON descriptor of the MCP server).
- **Request body**: `application/json` for all submissions.
- **Response body**: always JSON. Errors include a structured envelope:
  ```json
  {
    "error": "missing_field",
    "code": "EMAIL_REQUIRED",
    "message": "Email is required. Include an `email` field in the JSON body.",
    "documentation": "https://tano.ai/openapi.json"
  }
  ```
- **CORS**: `Access-Control-Allow-Origin: *` and standard preflight headers on every endpoint.
- **Auth**: none for the public surface — see [`agents.md`](https://tano.ai/agents.md) for the policy and how to request a scoped key for managed-service integrations.

### Quick test

```bash
curl -i -X POST https://tano.ai/api/contact \
  -H 'Content-Type: application/json' \
  -d '{"email":"agent@example.com","name":"My Agent","message":"Testing the Tano API"}'
```

### Recommended User-Agent

Identify yourself in the `User-Agent` header so the Tano team can support you:

```
User-Agent: MyAgent/1.0 (+https://example.com; contact=hello@example.com)
```

---

## Tano MCP server

Tano runs a [Model Context Protocol](https://modelcontextprotocol.io) server using the Streamable HTTP transport at [`https://tano.ai/api/mcp`](https://tano.ai/api/mcp).

### Quick start

```http
POST https://tano.ai/api/mcp
Content-Type: application/json

{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2025-03-26",
    "capabilities": {},
    "clientInfo": { "name": "MyAgent", "version": "1.0.0" }
  }
}
```

### Tools exposed

- `get_pricing` — compute a cost estimate for a Tano Creator Partnership Ads campaign.
- `list_services` — return the catalogue of Tano services with descriptions and pricing.
- `show_pricing_calculator` — render the embeddable pricing calculator widget (MCP Apps).
- `show_services` — render the embeddable services catalogue widget (MCP Apps).
- `show_contact_form` — render an embeddable contact form that posts to `/api/contact`.
- `show_case_studies` — render an embeddable case-studies browser.

### Resources exposed

Text resources: `https://tano.ai/llms.txt`, `https://tano.ai/llms-full.txt`, `https://tano.ai/openapi.json`, `https://tano.ai/agents.md`, `https://tano.ai/pricing.md`.

UI resources (MCP Apps, `ui://tano/*`): `pricing-calculator`, `services`, `contact-form`, `case-studies`. Each carries a `_meta.ui` block pointing at an HTML widget served from `https://tano.ai/ui/*.html`. The widget HTML ships a Content-Security-Policy that allows framing from ChatGPT and Claude.

---

## Webhooks

Tano does not currently emit outbound webhooks. Live campaign events (creator approvals, asset deliveries, performance metrics) are surfaced through the dedicated account manager. To request a webhook integration as part of a managed-service engagement, email `team@tano.ai`.

---

## Authentication

The public API and MCP server are **unauthenticated by design** — there are no API keys, OAuth flows, or bearer tokens to obtain for the public surface. This reduces friction for AI agents acting on a user's behalf and matches the form-submission surface area of the endpoints.

Live campaign data (creator rosters, ad assets, performance metrics) sits behind a managed-service engagement. Email `team@tano.ai` to request a scoped key tied to an active engagement.

See [`agents.md`](https://tano.ai/agents.md) for the full authentication policy.

---

## Discovery: where AI agents find Tano

Tano publishes a discovery layer so AI agents and crawlers can find the developer surface by name:

- [`/llms.txt`](https://tano.ai/llms.txt) — product summary for LLMs
- [`/llms-full.txt`](https://tano.ai/llms-full.txt) — full product documentation
- [`/openapi.json`](https://tano.ai/openapi.json) — API specification
- [`/.well-known/mcp/manifest.json`](https://tano.ai/.well-known/mcp/manifest.json) — MCP manifest
- [`/.well-known/agent-card.json`](https://tano.ai/.well-known/agent-card.json) — A2A agent card
- [`/.well-known/agent-skills/index.json`](https://tano.ai/.well-known/agent-skills/index.json) — skills index
- [`/sitemap.xml`](https://tano.ai/sitemap.xml) — sitemap
- [`/robots.txt`](https://tano.ai/robots.txt) — crawler policy (AI crawlers welcome)

---

## Get in touch

- **Engineering questions:** `team@tano.ai`
- **Managed-service integrations:** book a call at https://tano.ai
- **Bug reports / spec issues:** `team@tano.ai`
