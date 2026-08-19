---
name: MadKudu person discovery and enrichment
description: Find the right people at a target company with MadKudu — search people you already know, discover net-new prospects through a connected data provider, then enrich them with email and phone.
api: MadKudu API (MadAPI)
base_url: https://madapi.madkudu.com
auth: x-api-key header
generated: '2026-08-14'
method: generated
source: openapi/madkudu-madapi-openapi.yml
operations:
- Sourcing_getProviders
- Sourcing_discoverPersons
- Sourcing_enrichPerson
- Search_searchPersons
- Lookup_lookupPersons
- Persons_getPerson
---

# MadKudu person discovery and enrichment

Two different jobs, two different paths: **search** finds people MadKudu already knows about
from your connected systems; **sourcing** goes out to a third-party provider for people who are
not in your CRM at all.

## Auth

`x-api-key: YOUR_API_KEY` on every request, HTTPS only, base `https://madapi.madkudu.com`.

## Prerequisite for sourcing

Discovery and enrichment require a connected **Apollo**, **Cognism** or **ZoomInfo** account on
your MadKudu workspace. Sourcing calls cost **0 MadKudu credits** — they consume *your provider's*
credits instead.

## Path A — people MadKudu already knows

1. **Resolve one person** — `Lookup_lookupPersons` → `GET /lookup/persons`
   Query by `email`, `linkedin` (e.g. `in/francisbrero`), `twitter`, or `external_id`.
   Returns an array of `PersonDetailsMini`; take `mk_id`. 422 on invalid input.

2. **Or search many** — `Search_searchPersons` → `POST /search/persons`
   Body `PersonSearchRequest`: `limit`, `cursor`, `search`, `filters[]`, `filterLogic`, `sort`.
   `filters[].property` is a **closed enum** — `email`, `first_name`, `last_name`, `name`,
   `title`, `persona`, `company_domain`, `country`, `state`, `city`, `activities`,
   `has_left_company` — so validate before spending the credit.
   Response `{ data: [PersonDetailSearch], meta: { limit, total, has_next_page, next_cursor } }`.

3. **Full detail** — `Persons_getPerson` → `GET /persons/{mk_id}` for title, persona, company
   reference, `scores.customer_fit` and `scores.likelihood_to_buy`.

## Path B — net-new prospects

1. **Check what is connected** — `Sourcing_getProviders` → `GET /sourcing/providers`
   Returns `[{provider, isConnected}]`. Do not attempt discovery against a provider that reports
   `isConnected: false`.

2. **Discover** — `Sourcing_discoverPersons` → `POST /sourcing/persons/discover`
   The request body is a **provider-shaped union** — `ApolloDiscoverRequest`,
   `CognismDiscoverRequest` or `ZoomInfoDiscoverRequest`. The filter vocabulary is entirely
   different per provider (`person_titles`/`person_seniorities` for Apollo,
   `jobTitles`/`seniority`/`managementLevel` for Cognism, `jobTitle`/`rpp` for ZoomInfo), and so
   is pagination: **page/size** for Apollo and ZoomInfo (size max 100), an opaque **cursor**
   string for Cognism. Branch on the provider; there is no common shape.
   Returns `PersonDiscoverResponse` — `data[]` of `SourcingPersonResult` plus `pagination`.

3. **Enrich** — `Sourcing_enrichPerson` → `POST /sourcing/persons/enrich`
   Body `PersonEnrichRequest` `{ provider, provider_id }` — the `provider_id` is the `id` from the
   discovery result. Returns email and phone. Enrich only the people you actually intend to
   contact; each enrichment spends provider credits.

## Errors and paging

`{"message": "..."}` for auth/limit/server errors, `{"detail":[...]}` for the declared 422s.
Note that the sourcing and detail operations declare **no** error responses in the published
spec — handle 401/404/429 anyway. Page with `meta.next_cursor` until `meta.has_next_page` is
false. See `errors/madkudu-problem-types.yml` and `conventions/madkudu-conventions.yml`.

## Doing this from an agent instead

MCP equivalents on `https://mcp.madkudu.com/YOUR_API_KEY/mcp`: `madkudu-search-persons`,
`madkudu-person-details`, `madkudu-discover-persons`, `madkudu-enrich-persons`.
Bindings in `mcp/madkudu-tool-crosswalk.yml`.
