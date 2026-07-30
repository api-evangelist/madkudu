---
name: MadKudu account intelligence lookup
description: Look up a company account in MadKudu, pull its full details, activities, and top engaged people so an agent can brief a sales rep.
api: MadKudu API (MadAPI)
base_url: https://madapi.madkudu.com
auth: X-API-Key header
operations:
- GET /lookup/accounts
- GET /accounts/{mk_id}
- POST /accounts/{mk_id}/activities
- GET /accounts/{mk_id}/top-persons
---

# MadKudu account intelligence lookup

Use the MadKudu API (MadAPI) to turn a company domain into an account brief.

## Auth
Send `X-API-Key: YOUR_API_KEY` on every request. Base URL `https://madapi.madkudu.com`
(staging `https://madapi.wisekudu.com`). Requests are credit-metered; on HTTP 429
("credits exhausted") back off and retry.

## Steps
1. **Resolve the account** — `GET /lookup/accounts` with the company domain to get the
   MadKudu account id (`mk_id`) and mini account details.
2. **Get full details** — `GET /accounts/{mk_id}` for fit/intent/engagement scores and
   firmographics.
3. **Pull activities** — `POST /accounts/{mk_id}/activities` (filters in the body) for the
   engagement timeline.
4. **Find the buyers** — `GET /accounts/{mk_id}/top-persons` for the most engaged people.

## Errors
JSON body `{ "message": ... }`. 401 = bad/expired key, 404 = unknown mk_id, 422 = validation,
429 = rate/credit limit (adds `details`), 500 = server. See errors/madkudu-problem-types.yml.
