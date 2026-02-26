---
name: gentic-influencer
description: Guides the influencer creator discovery workflow using Gentic MCP tools. Activates when users want to find, score, or manage influencer/creator partnerships.
license: MIT
metadata:
  author: gentic
  version: "1.0.0"
---

# Gentic Influencer Discovery

Workflow guide for finding and scoring influencer/creator matches using Gentic's MCP tools. Steps build on each other — follow the order below.

## When to apply

- User wants to find influencers or creators for a brand campaign
- User asks to search Instagram or TikTok for creator partnerships
- User wants to score, review, or export creator match results
- User is setting up a brand profile for influencer matchmaking

## Tools reference

| Tool | Purpose | Cost |
|------|---------|------|
| `get_brand` | Read current brand profile | Free |
| `save_brand` | Create or update brand profile | Free |
| `fetch_page` | Scrape a web page for brand context | Free |
| `search_creators` | Vector search for creators on Instagram/TikTok | 15¢/result |
| `score_creators` | AI-score creators against brand profile | 35¢/result |
| `get_creator_results` | View scored results inline | Free |
| `export_creator_results` | Download results as CSV | Free |

## Workflow

### 1. Brand setup (required first)

Check if a brand profile exists with `get_brand`. If empty or missing, walk the user through `save_brand` before anything else. A good profile needs:
- `description`: brand identity, values, voice, target audience
- `matchmakingInstructions`: what makes a creator a good fit (content style, audience demographics, deal-breakers)
- `activeCampaignContext`: current campaign or product focus

If the user has a website, use `fetch_page` to pull context and draft the profile for them.

### 2. Search creators

Use `search_creators` to find creators. Key behavior:
- **Always search both platforms** (Instagram and TikTok) unless the user specifies one. Run both calls in parallel.
- Write descriptive `searchText` — "fitness micro-influencers who post workout routines and meal prep" beats "fitness influencers"
- Use filters: `minFollowers`, `maxFollowers`, `country`, `minEngagementRate` to narrow results
- `campaignContext` auto-reads from the brand profile — only override if the user gives campaign-specific instructions
- Results are stored automatically. Each search gets a unique `run_id`.

### 3. Score creators

After searching, run `score_creators` to AI-score the matches against the brand profile.
- **Scoring is async.** Tell the user results are processing and will be ready shortly.
- Score both platforms if both were searched.
- The `brandContext` auto-assembles from the brand profile — don't override unless asked.
- Scoring writes `final_match_score` (0-100) plus sub-scores back to the database.

### 4. View results

Use `get_creator_results` to review scored creators inline. When presenting results:

- Format as a **ranked table** with columns: rank, username, platform, followers, engagement rate, match score, one-line analysis
- **Bold** any creator with score >= 80
- Below the table, highlight the **top 3** with a short paragraph each: why they're a fit, any risk factors, and their personalized pitch angle
- If `total` exceeds `count`, mention how many more results are available
- If results have null scores, they haven't been scored yet — tell the user to run scoring first

### 5. Export

When the user wants to share or save results, use `export_creator_results` to generate a CSV download link. Mention the link expires in 1 hour.

## Important notes

- All tools are org-scoped. Users only see their own data.
- Mention costs when the user is doing a large search (50+ results).
- If a user asks to "find influencers" or "find creators" with no prior context, start from step 1.
