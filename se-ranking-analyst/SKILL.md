---
name: se-ranking-analyst
description: >-
  Pull, analyze, and report on SE Ranking data — keyword rankings, backlinks,
  AI/LLM search visibility, SERPs, and site audits — via the SE Ranking MCP
  tools or API. Use this skill whenever the task touches SE Ranking data or a
  tracked project: weekly or monthly client SEO reports, "what moved this week",
  rank-tracking changes, backlinks gained or lost, AI visibility and
  ChatGPT/Perplexity brand mentions, competitor SERP comparison, or computing
  period-over-period deltas. Trigger even when SE Ranking is not named
  explicitly but the user clearly wants analysis of rankings, backlinks, AI
  search visibility, or audit data for a site they track. It encodes which tool
  to reach for per data category, how to resolve a project and comparison dates,
  how to compute period-over-period deltas accurately, and the
  report format.
compatibility: >-
  Requires access to SE Ranking data, either through the SE Ranking MCP
  connector (recommended) or the SE Ranking API in a pipeline. Tool names
  may vary slightly by connector version — confirm exact tool/parameter
  names at runtime rather than assuming them.
---

# SE Ranking Analyst

Turn raw SE Ranking data into a defensible analysis: pull the right metrics, compute period-over-period change, explain what moved and why, and propose actions. The whole point of this skill is that the tool selection, the safe-delta rules, and the report shape are settled in advance, so each run is fast and consistent instead of re-derived from scratch.

## Mental model: two tool families, one API

SE Ranking exposes a large set of tools, and the safe default is to load a tool's exact parameters before calling it (e.g. via tool search) rather than guessing parameter names. They split into two families, and it's worth picking the right one up front:

- **PROJECT_\*** — operate on the user's own tracked projects (identified by a `site_id`). These reflect the real tracking setup: the keywords being monitored, the search engine and location configured, the actual check dates. Use these for client reporting and anything about "our site / our tracked keywords".
- **DATA_\*** — research tools that work on *any* domain or keyword without a project. They consume SE Ranking data-API credits. Use these for competitor research, prospecting, or pulling a metric that isn't set up as a tracked project.

When the user means "my client's monthly report", that is almost always PROJECT_\*. When they mean "how does competitor X look" or "research this keyword", that is DATA_\*. The full categorized tool map is in `references/endpoints.md` — read it before pulling data so you reach for the correct tool the first time.

The same logic applies if the user is building an n8n / raw-API pipeline rather than calling MCP tools live: the endpoint families and the safe-delta rules below are identical; only the call mechanism differs.

## Step 1 — Resolve the project and the comparison window

Do this before pulling anything.

1. Identify the exact project. If you don't have a `site_id`, call `PROJECT_listProjects` and match by domain. **Confirm the project name/domain back to the user before pulling** — this makes sure the whole report is built on the project the user actually meant, and it is cheap to confirm now.
2. Establish two dates: the current period and the comparison baseline. Use `PROJECT_getCheckDates` (dates positions were actually checked) or `PROJECT_getHistoricalDates` (standard comparison points like yesterday / last month) so you compare against a date that genuinely exists, not an interpolated one.
3. Confirm scope: which data categories are in this report (rankings, backlinks, AI visibility, audit) and which search engine / location, if the project has several.

## Step 2 — Pull the data

Consult `references/endpoints.md` for the right tool per category. Pull the current period and the baseline period for each category in the report. Keep each category's raw result separate and labelled — do not merge pulls into one blob before analysis, or the downstream reasoning gets a tangle of unattributed numbers.

## Step 3 — Compute deltas SAFELY (read this every time)

These keep each report built only from its own project's real data. Treat the following as hard rules, not preferences:

- **One project per report.** Never merge data from multiple `site_id`s into a single client's analysis. Run each client as an isolated pass.
- **Missing data is missing — never substitute.** If a metric has no record for a date (SE Ranking writes no record when nothing changed, so gaps are normal and meaningful), report it as "no data for this period". Don't fill a gap with the previous period's value or with an unrelated figure. Absence is signal, not a hole to fill.
- **No silent reuse.** If you reuse a prior result for any reason (e.g. a genuinely unchanged metric), say so explicitly in the output. Do not present reused or carried-over numbers as freshly pulled.
- **Compute change as current minus baseline, per metric, with both raw and percent where it helps.** For positions, lower is better — a move from 8 to 3 is an improvement of 5; state direction in words so the sign is never ambiguous.

## Step 4 — Analyze

Lead with what changed, not with raw tables. For each category, surface the movements that matter and skip the noise.

Default significance thresholds (state them in the report; treat as tunable, and adjust if the user has their own):
- **Rankings:** a position change of 3+ spots, or any keyword crossing the top-10 / top-3 boundary in either direction.
- **AI / LLM visibility:** a change in visibility share or in the count of prompts where the brand is mentioned; new prompts where the brand appeared or dropped out. This is volatile — be explicit about the period and prefer share-of-mentions over single-prompt anecdotes.
- **Backlinks:** net new vs lost referring domains (weight domains over raw link count); any lost link from a high-authority domain.
- **Audit:** new critical errors vs the previous crawl; issues resolved.

For each significant movement give: what moved (metric + magnitude + direction), the most plausible *why* (tie to a known cause when one exists — an algo update, a new/lost link, a content change — and clearly mark it as a hypothesis when it is one), and a recommended next action. Do not invent a cause to fill the slot; "cause unclear, worth investigating" is a valid and honest line.

## Step 5 — Format the report

Default structure (adapt to the user's house style if they have one):

```
# [Client / domain] — [period] vs [baseline period]
## TL;DR
2-4 bullets: the headline movements only.
## Rankings
What moved past threshold, with direction and likely why.
## AI / LLM visibility
Share + prompt-level changes, with the period stated.
## Backlinks
Net referring-domain change; notable gains/losses.
## Site audit (if in scope)
New critical issues; resolved issues.
## Recommended actions
Prioritised, tied to the movements above.
## Data notes
Dates compared, any "no data" gaps, anything reused or carried over.
```

The **Data notes** section is not optional — it is where the safe-delta discipline becomes visible and auditable.

## Client-facing delivery

Anything client-facing (an email, a posted report) is an explicit-permission action: draft it, show the user, and wait for a clear yes before sending. Do not auto-send. The human approval gate is the last line of defence, not a step to optimise away.

## Credit and token discipline

DATA_\* calls cost data-API credits; live MCP analysis costs tokens — two separate budgets. Pull only the categories and date pairs the report needs, batch where a bulk endpoint exists (e.g. `DATA_getKeywordsMetrics` for many keywords at once), and don't re-pull a period you already have in context. Keep the Step 3 data rules in place even when optimising for credits — accuracy comes first.

## Language

Match the language of the request. If a deliverable is for a specific audience and the language isn't obvious, ask which language it should be in.
