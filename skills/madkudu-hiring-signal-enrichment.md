---
name: MadKudu hiring-signal enrichment
description: Read a company's open job postings through MadKudu to detect growth, expansion and buying intent, then attach that signal to the account and the people who matter.
api: MadKudu API (MadAPI)
base_url: https://madapi.madkudu.com
auth: x-api-key header
generated: '2026-08-14'
method: generated
source: openapi/madkudu-madapi-openapi.yml
operations:
- Enrichment_searchJobPostings
- Lookup_lookupAccounts
- Accounts_getAccountTopPersons
- AI_webSearch
---

# MadKudu hiring-signal enrichment

Hiring is the cheapest public proxy for budget and initiative. This flow reads a company's job
openings, then grounds them in the account you already know.

## Auth

`x-api-key: YOUR_API_KEY`, HTTPS only, base `https://madapi.madkudu.com`.

## Steps

1. **Pull the postings** — `Enrichment_searchJobPostings` →
   `POST /enrichment/job-postings`
   Body `JobPostingSearchRequestBody`:

   ```json
   {
     "domain": "example.com",
     "limit": 20,
     "cursor": 0,
     "search": "platform engineer",
     "filters": [{ "property": "...", "operator": "...", "value": "..." }],
     "filter_logic": "and",
     "sort": { "sort_by": "event_on", "sort_order": "desc" }
   }
   ```

   `domain` is the only required field. Note the snake_case `filter_logic` here against
   `filterLogic` on the account/person search bodies — the same concept is spelled two ways on
   the same API.

   Each `JobPosting` carries `company_domain`, `job_opening_title`, `job_opening_url`,
   `keywords`, `categories`, `location`, `event`, `event_on` and `first_event_on`.
   Response envelope is `{ data: [...], meta: { limit, total, has_next_page, next_cursor } }`.

   **Cost: 3 credits per 20 postings.** Set `limit` deliberately.

2. **Attach it to the account** — `Lookup_lookupAccounts` → `GET /lookup/accounts?domain=...`
   to get the `mk_id`, so the hiring signal is filed against the same account record your CRM
   sync uses (`source_system` tells you which system that record came from).

3. **Find who to tell** — `Accounts_getAccountTopPersons` →
   `GET /accounts/{mk_id}/top-persons?limit=5`. Hiring in a function is only actionable next to
   the people in that function who already engage with you.

4. *(optional)* **Corroborate** — `AI_webSearch` → `POST /ai/web-search` with
   `{ "query": "<company> funding OR expansion", "searchDepth": "advanced", "includeAnswer": true }`.
   Tavily-backed; returns `results[]` with `title`, `url`, `content`, `score`, `publishedDate`
   and an optional LLM `answer`. **Cost: 5 credits** — the most expensive call on the API, so
   gate it behind a real hiring hit rather than running it per account.

## Paging

Feed `meta.next_cursor` back as `cursor` while `meta.has_next_page` is true. There is no
idempotency key: if a page request times out and you retry it, you are billed for both.

## Errors

`Enrichment_searchJobPostings` declares 422 with `Common.Errors.ValidationError`
(`{"detail":[{"loc":[...],"msg":"...","type":"..."}]}`); `AI_webSearch` declares no error
responses at all. 429 means credits are exhausted and carries no `Retry-After`.
See `errors/madkudu-problem-types.yml`.

## Not available as an MCP tool

Job-posting enrichment is REST-only — there is no `madkudu-*` tool for it in the published
MadMCP tool list (see the `rest_only` block in `mcp/madkudu-tool-crosswalk.yml`). An agent has to
call MadAPI directly for this signal.
