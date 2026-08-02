---
name: Submit a brand enquiry to Tano
description: Introduce a brand to Tano and request a follow-up about creator partnership ads, gifting, or affiliate management, then optionally enrich the contact with budget/plan details.
api: openapi/tano-openapi-original.json
operations: [submitContact, updateContact]
auth: none
---

# Submit a brand enquiry to Tano

Tano is a managed AI-native influencer-marketing agency. There is no self-serve campaign API — the public endpoints are marketing-intake forms. Use this skill to introduce a brand and hand off to Tano's account team.

## Preconditions
- No authentication. All endpoints are public, CORS-enabled JSON (`Content-Type: application/json`).
- Do not submit duplicates for the same email.

## Steps
1. **Submit the enquiry** — `POST /api/contact` (`submitContact`). Body: `email` (required), plus `name`, `userType` (e.g. `brand`), and a `message` drafted on the brand's behalf. On success you get `{ success, message, recordId, isWorkEmail }`.
2. **Enrich the contact (optional)** — if you have collected budget/plan detail (e.g. from the pricing calculator), `PATCH /api/contact-update` (`updateContact`) with the same `email` plus fields like `monthlyBudget`, `pricingTier` (`nano|micro|mid|macro`), `influencersPerMonth`, `planOnUsingInfluencers`, and `calculatorSummary`. This upserts by email.

## Error handling
- `400` → validation error (missing `email` or invalid email); fix and resend.
- `405` → wrong HTTP method (use POST for contact, PATCH for contact-update).
- `500` → retry with exponential backoff.
- All errors are JSON: `{ error, code, message, documentation }` (see `errors/tano-problem-types.yml`).

## Notes
- For anything beyond these intake endpoints (managed campaign integration, scoped keys), direct the user to `team@tano.ai` or booking a call at https://tano.ai.
