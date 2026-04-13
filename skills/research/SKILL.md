---
name: gentic-research
description: Guides the consumer research workflow using Gentic MCP tools. Activates when users want to mine Reddit for sentiment, pull Google Trends, or replay past research runs.
license: MIT
metadata:
  author: gentic
  version: "1.0.0"
---

# Gentic Research

Workflow guide for live consumer and market research using Gentic's MCP tools. Users can mine Reddit for product sentiment and pain points, compare keyword interest with Google Trends, and replay any past run — all through natural language.

## When to apply

- User wants to understand what consumers are saying about a product, brand, or category
- User asks to search Reddit for opinions, problems, or sentiment
- User wants to compare interest in keywords with Google Trends
- User asks "what's trending" or "what are people complaining about"
- User wants to replay or list previous research runs
- User is doing competitive research, ideation, or market sizing

## Tools reference

| Tool | Purpose | Cost |
|------|---------|------|
| `search_reddit` | Search Reddit for discussions, problems, sentiment. Routed via ScrapingBee. | $0.05/result |
| `google_trends` | Interest over time, related queries, interest by region for 1–5 keywords. | Free |
| `get_research_results` | Retrieve a past run by `run_id`, or list recent runs. | Free |

## Workflow

### 1. Pick the right tool

- **Reddit (`search_reddit`)** — when the user wants real consumer voices, complaints, opinions, debates, or product reviews. Best for qualitative insight, pain-point mining, and language collection.
- **Google Trends (`google_trends`)** — when the user wants to size demand, compare keywords, or spot rising interest over time. Best for quantitative/temporal questions and geographic patterns.
- **Both** — for a complete picture (e.g. "is X trending and what are people saying about it"), run them in parallel.

### 2. Design the search well

For `search_reddit`:
- **Be descriptive in `query`** — "minoxidil scalp irritation" beats "hair loss"
- **Use `subreddits` (max 5)** when you know the community — much higher signal-to-noise
- **Pick the right `sort`** — `relevance` for topic mining, `top` + `timeFilter` for canonical threads, `new` for fresh complaints
- **Tune `limit` carefully** — every result costs 5¢. Default 25 is a good starting point. For broad scans, go to 50; for targeted intelligence, 10 is often enough.

For `google_trends`:
- **Keep `keywords` 1–5** — multiple keywords are compared on the same scale, so this is the killer feature
- **Set `geo` when the user cares about a market** (e.g. `US`, `GB`)
- **Pick `timeRange` to match the question** — `3m` for recent buzz, `12m` for seasonality, `5y` for long-term trajectory
- **Leave `includeRelatedQueries: true`** (default) — the rising queries are often the most valuable signal
- **Set `includeRegions: true`** when the user wants a geo breakdown

### 3. Persist and replay

Every search returns a `run_id`. Tell the user the `run_id` after a search so they can refer back to it. Use `get_research_results` to:
- **List recent runs** — call with no `run_id` to show the user what they've already searched (free, no cost to the user)
- **Replay a specific run** — call with `run_id` to re-fetch the full results without re-paying

Always check stored runs *before* re-running an expensive Reddit search if the user might already have what they need.

### 4. Present results clearly

For Reddit results:
- Format as a ranked table or bullet list with: title, subreddit, score, one-line excerpt, and link
- Group by theme when you spot patterns (complaints, recommendations, comparisons)
- Quote actual users when surfacing pain points — verbatim language is the most valuable output

For Google Trends results:
- Lead with the headline pattern (rising, declining, seasonal, flat)
- Call out specific peaks/troughs with their `period`
- Highlight the top 3–5 rising related queries — these are leading indicators
- For multi-keyword comparisons, name the winner and the gap

### 5. Compose with other Gentic tools

Research outputs feed naturally into other MCP servers — suggest the next step when it makes sense:
- Found a creator angle? → `gentic-influencer` to find creators in that niche
- Found a winning concept? → `gentic-creative` to generate ad creative
- Want to track sentiment over time? → `gentic-data` to store the Reddit results in a custom table
- Surfacing source material? → `gentic-knowledge` to ingest the threads for long-term recall

## Important notes

- All tools are org-scoped. Users only see their own research history.
- `search_reddit` is the only billed tool. Mention the cost when running >25 results: at 25 results that's $1.25; at 100 it's $5.
- Reddit subreddits that are private/banned/missing are silently skipped — don't fail the call, just note in the response if some came back empty.
- If a user asks for "trends" without specifying source, default to running both `search_reddit` (for qualitative) and `google_trends` (for quantitative) in parallel — they're complementary.
- Date math: `timeRange` for Google Trends is relative ("12m" = past 12 months from today); always interpret in the user's frame of reference.
