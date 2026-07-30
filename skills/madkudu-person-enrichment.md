---
name: MadKudu person discovery and enrichment
description: Discover and enrich people at a target account using MadKudu sourcing providers, returning contact info for outreach.
api: MadKudu API (MadAPI)
base_url: https://madapi.madkudu.com
auth: X-API-Key header
operations:
- GET /sourcing/providers
- POST /sourcing/persons/discover
- POST /sourcing/persons/enrich
- POST /search/persons
- GET /persons/{mk_id}
---

# MadKudu person discovery and enrichment

Find and enrich the right people at a target company.

## Auth
Send `X-API-Key: YOUR_API_KEY`. Base URL `https://madapi.madkudu.com`. Credit-metered;
handle HTTP 429 with backoff.

## Steps
1. **Check providers** — `GET /sourcing/providers` to see connected sourcing providers and
   their status.
2. **Discover persons** — `POST /sourcing/persons/discover` with filters to find candidate
   people via connected providers.
3. **Enrich contact info** — `POST /sourcing/persons/enrich` with the provider + provider-specific
   id to resolve contact details.
4. **Search existing** — alternatively `POST /search/persons` with advanced filters, sorting, and
   pagination to search known persons, then `GET /persons/{mk_id}` for full detail.

## Errors
JSON `{ "message": ... }`; 401/404/422/429/500 as in errors/madkudu-problem-types.yml.
Respect pagination on search/discover per conventions/madkudu-conventions.yml.
