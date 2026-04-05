---
name: gentic-meta
description: Guides the Meta advertising workflow for managing campaigns, ad sets, creatives, performance insights, and report exports through the Meta Graph API.
license: MIT
metadata:
  author: gentic
  version: "1.0.0"
---

# Gentic Meta

Workflow guide for managing Meta (Facebook/Instagram) advertising through Gentic's MCP tools. Navigate ad accounts, analyze performance, create campaigns, upload creatives, and export reports — all through natural language.

## When to apply

- User wants to view or analyze Meta ad performance
- User wants to create Meta campaigns, ad sets, or ads
- User wants to upload images or videos for Meta ads
- User wants to export ad performance reports as CSV
- User wants to browse their Meta ad account hierarchy
- User asks about their Facebook or Instagram ads
- User wants to find their Facebook Page ID for ad creation

## Tools reference

| Tool | Purpose | Cost |
|------|---------|------|
| `get_meta_ids` | Browse ad account hierarchy: accounts → campaigns → ad sets → ads → creatives | Free |
| `get_meta_pages` | List Facebook Pages for the authenticated user | Free |
| `fetch_meta_insights` | Get performance metrics (spend, clicks, CTR, conversions) with date filtering and breakdowns | 10¢/call |
| `fetch_meta_ad_creatives` | View ad creative details: copy, images, videos, CTA | 5¢/call |
| `create_meta_campaign` | Create a new campaign with objective, budget, and bid strategy | Free |
| `create_meta_adset` | Create an ad set with targeting, scheduling, and optimization | Free |
| `upload_meta_video` | Upload a video to an ad account from a public URL | Free |
| `upload_meta_image` | Upload an image to an ad account from a public URL | Free |
| `create_meta_ad` | Create a video ad with copy, CTA, and destination URL | $1.00/call |
| `create_meta_image_ad` | Create a static image ad with copy, CTA, and destination URL | $1.00/call |
| `export_meta_report` | Export performance data as a downloadable CSV (expires in 1 hour) | $1.00/call |

## Workflow

### 1. Browse ad accounts

Use `get_meta_ids` to navigate the ad account hierarchy. Start at the top and drill down:

1. `level: "adaccounts"` — discover all ad accounts
2. `level: "campaigns", parent_id: "act_XXXXXX"` — list campaigns in an account
3. `level: "adsets", parent_id: "<campaign_id>"` — list ad sets in a campaign
4. `level: "ads", parent_id: "<adset_id>"` — list ads in an ad set
5. `level: "adcreatives", parent_id: "<ad_id>"` — get creative IDs for an ad

Use `date_preset` (e.g. `"last_30d"`) or `time_range` (`{"since":"2026-01-01","until":"2026-01-31"}`) to include spend/performance alongside the hierarchy.

**Critical:** All IDs must be passed as **quoted strings** (e.g. `"120239005302490769"`), never as bare numbers. JavaScript rounds large integers, causing silent failures.

### 2. Get Facebook Pages

Use `get_meta_pages` to list Pages associated with the user. Page IDs are required when creating ads — the ad is published on behalf of the Page.

This tool takes no parameters.

### 3. Fetch insights

Use `fetch_meta_insights` to analyze ad performance. Key parameters:

- `level`: `"campaign"`, `"adset"`, or `"ad"`
- `ids`: comma-separated IDs as a quoted string (e.g. `"120239005302490769,120239005302490770"`)
- `date_preset`: `"today"`, `"yesterday"`, `"last_7d"`, `"last_14d"`, `"last_30d"`, `"this_month"`, `"last_month"`, `"maximum"`
- `time_range`: custom date range as JSON string (`{"since":"2026-01-01","until":"2026-01-31"}`)
- `breakdowns`: e.g. `"age,gender"`, `"platform_position"`, `"country"`
- `fields`: defaults to impressions, spend, clicks, CTR, actions, conversions, action_values

Use `date_preset` OR `time_range`, not both.

### 4. View ad creatives

Use `fetch_meta_ad_creatives` to see creative details (copy, images, videos, CTA) for specific ad creatives. Pass creative IDs from `get_meta_ids` with `level: "adcreatives"`.

### 5. Create campaigns

Use `create_meta_campaign` to create a new campaign. Key behavior:

- Defaults to `PAUSED` status and `OUTCOME_SALES` objective
- Objectives: `OUTCOME_SALES`, `OUTCOME_TRAFFIC`, `OUTCOME_AWARENESS`, `OUTCOME_ENGAGEMENT`, `OUTCOME_LEADS`, `OUTCOME_APP_PROMOTION`
- Enable CBO with `is_campaign_budget_optimization: true` and set `daily_budget` in cents (e.g. `"270000"` for $2,700)
- Bid strategies: `LOWEST_COST_WITHOUT_CAP`, `LOWEST_COST_WITH_BID_CAP`, `COST_CAP`
- Set `special_ad_categories` if required (e.g. `["CREDIT", "EMPLOYMENT", "HOUSING"]`)

### 6. Create ad sets

Use `create_meta_adset` to define targeting, budget, and optimization within a campaign.

- `targeting`: JSON object for audience definition (e.g. `{"geo_locations": {"countries": ["US"]}, "age_min": 25, "age_max": 55}`)
- `optimization_goal`: `REACH`, `LINK_CLICKS`, `OFFSITE_CONVERSIONS`, `IMPRESSIONS`, etc.
- `daily_budget`: in cents as a string (required unless the parent campaign has CBO enabled)
- `billing_event`: defaults to `IMPRESSIONS`
- `start_time` / `end_time`: ISO format
- `promoted_object`: e.g. `{"pixel_id": "...", "custom_event_type": "PURCHASE"}`
- Defaults to `PAUSED` status

### 7. Upload media

**Videos:** Use `upload_meta_video` with `ad_account_id`, a public `file_url`, and a `name`.

**Images:** Use `upload_meta_image` with `ad_account_id`, a public `image_url`, and a `name`. Returns an `image_hash` used when creating image ads.

Both tools require a publicly accessible URL (e.g. S3 pre-signed URL, public CDN link).

### 8. Create ads

**Video ads:** Use `create_meta_ad` with the `video_id` from `upload_meta_video`.

**Image ads:** Use `create_meta_image_ad` with `image_hash` from `upload_meta_image` (preferred) or a direct `image_url`.

Both require:
- `adset_id`, `page_id` (from `get_meta_pages`), `ad_account_id`
- `message` (ad copy), `title`, `link_description`
- `call_to_action_type`: `SHOP_NOW`, `LEARN_MORE`, `SIGN_UP`, `DOWNLOAD`, `BOOK_NOW`, `CONTACT_US`, `GET_QUOTE`, `APPLY_NOW`
- `call_to_action_link`: destination URL
- Optional `url_tags` for UTM tracking parameters

All Advantage+ creative enhancements are automatically disabled on ad creation. Ads default to `PAUSED` status.

### 9. Export reports

Use `export_meta_report` to generate a CSV download of performance data.

- `campaign_id`: required — auto-resolves the ad account
- `level`: `"campaign"`, `"adset"`, or `"ad"`
- `date_preset` or `time_range` for date filtering
- `time_increment`: `"1"` for daily, `"7"` for weekly, `"monthly"`, `"all_days"` (default)
- `breakdowns`: e.g. `"age,gender,country"`

Returns a download link that expires in 1 hour.

## Typical workflows

### "How are my Meta ads performing?"
1. `get_meta_ids` with `level: "adaccounts"` to find the ad account
2. `get_meta_ids` with `level: "campaigns"` to list campaigns
3. `fetch_meta_insights` on the campaigns with `date_preset: "last_30d"`

### "Create a new campaign with ads"
1. `get_meta_ids` with `level: "adaccounts"` to get the ad account ID
2. `get_meta_pages` to get the Facebook Page ID
3. `create_meta_campaign` with objective and budget
4. `create_meta_adset` with targeting and optimization
5. `upload_meta_image` or `upload_meta_video` with creative assets
6. `create_meta_image_ad` or `create_meta_ad` with copy and CTA

### "Export a report for last month"
1. `get_meta_ids` with `level: "campaigns"` to find the campaign ID
2. `export_meta_report` with `date_preset: "last_month"` and `level: "ad"`

## Important notes

- All IDs must be **quoted strings** — JavaScript rounds large integers, causing silent failures with wrong IDs.
- Advantage+ creative enhancements are automatically disabled when creating ads.
- Meta Graph API v23.0 is used.
- Batch API supports up to 50 requests per call.
- Export report download links expire in 1 hour.
- All ads and campaigns are created in `PAUSED` status by default — the user must manually activate them.
- Budgets are specified in **cents** (e.g. `"500000"` = $5,000).
- All tools are org-scoped. Users only see their own ad accounts.
