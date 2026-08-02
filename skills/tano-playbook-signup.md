---
name: Sign up for a Tano playbook or product
description: Register a brand's work email for one of Tano's downloadable playbooks / product offerings (Creator Partnership Ads, Content Analysis Framework, Creator Discovery Guide) or the USA waitlist.
api: openapi/tano-openapi-original.json
operations: [cpaSignup, cafSignup, cdgSignup, usaSignup]
auth: none
---

# Sign up for a Tano playbook or product

Use this skill when a brand wants a Tano playbook/download or to join the USA waitlist. These endpoints are work-email gated.

## Preconditions
- No authentication; JSON in/out, CORS open.
- `cpaSignup`, `cafSignup`, `cdgSignup` **require a work email** — personal domains (gmail.com, yahoo.com, …) are rejected with `400`.

## Steps (choose the offering)
1. **Creator Partnership Ads** — `POST /api/cpa-signup` (`cpaSignup`), body: `email` (work), optional `phoneNumber`.
2. **Content Analysis Framework** — `POST /api/caf-signup` (`cafSignup`), body: `email` (work).
3. **Creator Discovery Guide** — `POST /api/cdg-signup` (`cdgSignup`), body: `email` (work).
4. **USA waitlist** — `POST /api/usa-signup` (`usaSignup`), body: `email` plus optional `name`, `gifted_or_allowlisted`, `already_working_with_influencers`.

Each returns `{ success, message, recordId }`.

## Error handling
- `400` → missing/invalid email or a personal-domain email where a work email is required.
- `405` → use POST.
- `500` → retry with exponential backoff.
- JSON error envelope: `{ error, code, message, documentation }` (see `errors/tano-problem-types.yml`).

## Related
- Creators (not brands) use `POST /api/creator-signup` (`creatorSignup`).
- Webinar registration uses `POST /api/runna-webinar` (`runnaWebinarSignup`).
