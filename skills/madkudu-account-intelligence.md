---
name: MadKudu account intelligence lookup
description: Turn a company domain into a sales-ready account brief with MadKudu — resolve the account, pull firmographics and fit scoring, read the engagement timeline, and identify the most engaged people.
api: MadKudu API (MadAPI)
base_url: https://madapi.madkudu.com
auth: x-api-key header
generated: '2026-08-14'
method: generated
source: openapi/madkudu-madapi-openapi.yml
operations:
- Lookup_lookupAccounts
- Accounts_getAccount
- Accounts_getAccountActivities
- Accounts_getAccountTopPersons
---

# MadKudu account intelligence lookup

Use MadAPI to turn a company domain into an account brief. Every operationId below is taken
from the OpenAPI 3.1.0 MadKudu publishes in its API reference.

## Auth

Send `x-api-key: YOUR_API_KEY` on **every** request. HTTPS only — plain HTTP fails.
Base URL `https://madapi.madkudu.com` (staging `https://madapi.wisekudu.com`).
Keys are personal and carry the issuing user's permissions; create them at
`https://admin.madkudu.com` → Personal Settings → My API Keys.

## Cost before you call

Calls are credit-metered, and MCP tool calls count against the same allowance:

| Step | Operation | Credits |
|---|---|---|
| 1 | `Lookup_lookupAccounts` | 1 |
| 2 | `Accounts_getAccount` | 3 |
| 3 | `Accounts_getAccountActivities` | — (activity retrieval, not in the published table) |
| 4 | `Accounts_getAccountTopPersons` | — (detail-class call) |

There is **no idempotency key** and **no rate-limit header**. A retried POST is billed again,
and nothing in the response tells you how much allowance is left — only the Admin console does.

## Steps

1. **Resolve the account** — `Lookup_lookupAccounts` → `GET /lookup/accounts`
   Query by any one of `domain`, `linkedin` (e.g. `company/madkudu`), `twitter`,
   `crunchbase` (e.g. `organization/madkudu`), or `external_id` (your CRM account id).
   Returns an **array** of `AccountDetailsMini`. Take `mk_id` — every later step needs it.
   Invalid input returns **422** with `{"detail":[{"loc":[...],"msg":"...","type":"..."}]}`.

2. **Get full details** — `Accounts_getAccount` → `GET /accounts/{mk_id}`
   Firmographics plus `scores.customer_fit` (score, segment, signals) and
   `scores.likelihood_to_buy`. Note: this operation declares **no error responses** in the
   spec, so handle 401/404/429 defensively.

3. **Read the engagement timeline** — `Accounts_getAccountActivities` →
   `POST /accounts/{mk_id}/activities`
   Body is `Common.Models.AccountActivitySearchRequest`:
   `{ "limit": 50, "cursor": 0, "search": "", "filters": [{"property":"...","operator":"...","value":"..."}], "sort": {...} }`
   Response is `{ data: [...], meta: { limit, total, has_next_page, next_cursor } }`.
   Page by feeding `meta.next_cursor` back as `cursor` while `meta.has_next_page` is true.

4. **Find the buyers** — `Accounts_getAccountTopPersons` →
   `GET /accounts/{mk_id}/top-persons?limit=N`
   Most engaged contacts, ranked by activity. Use these `mk_id`s with
   `Persons_getPerson` for detail.

5. *(optional)* **Ground the brief in your own positioning** —
   `Organisation_getValueProposition` → `GET /organisation/value-prop` returns
   persona-specific value props (0 credits) so a generated brief argues in your language.

## Errors

Plain JSON, never `application/problem+json`. Two envelopes exist: `{"message": "..."}` for auth,
rate-limit and server errors, and `{"detail": [...]}` for the 422s the spec declares.
`429` means the credit allowance is exhausted (`message` + `details`) — back off blind, there is
no `Retry-After`. See `errors/madkudu-problem-types.yml` and
`rate-limits/madkudu-rate-limits.yml`.

## Doing this from an agent instead

The same flow is exposed as MCP tools on `https://mcp.madkudu.com/YOUR_API_KEY/mcp` —
`madkudu-account-details`, `madkudu-account-activities`, `madkudu-account-top-users`,
`madkudu-account-brief-instructions`. The tool→operation bindings are in
`mcp/madkudu-tool-crosswalk.yml`.
