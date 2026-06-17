# SE Ranking tool map

Which tool to reach for, by data category. Tools are deferred — `tool_search` to load a tool's exact parameters before calling; do not guess parameter names. PROJECT_\* tools act on the user's tracked projects (need a `site_id`); DATA_\* tools research any domain/keyword and cost data-API credits.

## Contents
- Project resolution & dates
- Rankings / positions
- AI / LLM visibility
- Backlinks
- SERP
- Keyword research
- Site audit
- Competitors

## Project resolution & dates
- `PROJECT_listProjects` — all the user's projects; match by domain to get `site_id`.
- `PROJECT_getSummary` — summary stats for one project.
- `PROJECT_getSearchEngines` — which engine/location a project tracks (pick the right one when several).
- `PROJECT_getCheckDates` — dates positions were actually checked (use these as comparison anchors).
- `PROJECT_getHistoricalDates` — standard comparison points (yesterday, last month, etc.).
- `PROJECT_listKeywords` — tracked keywords with target pages.

## Rankings / positions
Tracked project:
- `PROJECT_getPositionHistory` — time-series position history (the core delta source).
- `PROJECT_getKeywordStats` — per-keyword stats.
- `PROJECT_runPositionCheck` — force a fresh check (costs; use sparingly).
- `PROJECT_setKeywordPosition` — manual override (rare).

Any domain (research):
- `DATA_getDomainKeywords` — keywords a domain ranks for (organic/paid).
- `DATA_getDomainOverviewByCountry` / `DATA_getDomainOverviewHistory` / `DATA_getDomainOverviewWorldwide` — traffic & ranking overview.
- `DATA_getDomainPages` — top ranking pages of a domain.

## AI / LLM visibility
Tracked project (AI Result Tracker):
- `PROJECT_getPromptsRankings` — AI-search ranking data over a date range (core delta source).
- `PROJECT_getPromptAnswer` — cached AI answer for a tracked prompt on a date.
- `PROJECT_listPrompts` — prompts tracked per LLM engine.
- `PROJECT_listLlmEngines` / `PROJECT_getLlmStatistics` / `PROJECT_getLlmStatus` — per-engine config, stats, tracking status.
- `PROJECT_getSiteBrand` — brand configured for the site.

Any target (research):
- `DATA_getAiSearchOverview` — AI search visibility metrics for a target.
- `DATA_getAiSearchLeaderboard` — compare domains/brands across ChatGPT, Perplexity, etc.
- `DATA_getAiSearchPromptsByBrand` / `DATA_getAiSearchPromptsByTarget` — prompts mentioning a brand / target.
- `DATA_getAiSearchBrand` — the brand name SE Ranking attributes to a target.

## Backlinks
Any target (research):
- `DATA_getBacklinksSummary` — profile fields + counts for one or many targets (good first call).
- `DATA_getBacklinksMetrics` — core counts.
- `DATA_listNewLostBacklinks` / `DATA_getNewLostBacklinksCount` — new vs lost within a period (core delta source).
- `DATA_getBacklinksRefDomains` / `DATA_listNewLostReferringDomains` / `DATA_getNewLostRefDomainsCount` — referring domains (weight these over raw links).
- `DATA_getBacklinksAnchors`, `DATA_getBacklinksAuthority`, `DATA_getDomainAuthority`, `DATA_getAllBacklinks` — anchors, authority, full list.

Monitored project backlinks:
- `PROJECT_getBacklinkStats`, `PROJECT_listProjectBacklinks`, `PROJECT_recheckProjectBacklinks`.
- `PROJECT_listDisavowedBacklinks` — disavow list.

## SERP
- `DATA_getSerpResults` — run a SERP query and get results.
- `DATA_getSerpTaskResults` / `DATA_getSerpTaskAdvancedResults` — retrieve queued SERP tasks.
- `DATA_getSerpLocations` — resolve city/region `location_id`.

## Keyword research
- `DATA_getKeywordsMetrics` — bulk metrics for many keywords in one call (prefer for batches).
- `DATA_getRelatedKeywords` / `DATA_getSimilarKeywords` / `DATA_getLongTailKeywords` / `DATA_getKeywordQuestions` — expansion.

## Site audit
Tracked project:
- `PROJECT_createAudit` / `PROJECT_recheckAudit` — launch / re-crawl.
- `PROJECT_getAuditReport`, `PROJECT_getAuditStatus`, `PROJECT_getAuditHistory` — report, status, prior runs.
- `PROJECT_getAuditPagesByIssue`, `PROJECT_getIssuesByUrl`, `PROJECT_getCrawledPages`.

Ad hoc:
- `DATA_createStandardAudit` / `DATA_createAdvancedAudit` (advanced renders JS), `DATA_getAuditReport`, `DATA_getAuditStatus`, `DATA_getAuditPagesByIssue`.

## Competitors
- `PROJECT_listCompetitors` / `PROJECT_getCompetitorPositions` — tracked competitors.
- `PROJECT_getCompetitorSerp10` / `PROJECT_getCompetitorSerp100` — top-N SERP for a tracked keyword.
- `DATA_getDomainCompetitors` — discover competing domains for any target.
- `DATA_getDomainKeywordsComparison` — compare two domains' keyword rankings.
