---
name: gentic-creative-assets
description: Guides the creative asset workflow using Gentic MCP tools. Activates when users want to generate ad images, vectorize product photos, search their product catalog, or find competitor ad inspiration.
license: MIT
metadata:
  author: gentic
  version: "1.0.0"
---

# Gentic Creative Assets

Workflow guide for generating ad images and managing a searchable product image catalog using Gentic's MCP tools. Steps build on each other — follow the order below.

## When to apply

- User wants to generate ad images or creative assets
- User wants to create ads inspired by another brand's style
- User wants to upload or vectorize product photos
- User wants to search their product image catalog
- User wants to check the status of image generation jobs
- User wants to browse or search competitor brand ads for inspiration

## Tools reference

| Tool | Purpose | Cost |
|------|---------|------|
| `fetch_page` | Scrape a web page for brand/product context | 10¢/page |
| `vectorize_product_images` | Batch vectorize product photos for searchable catalog | 5¢/image |
| `search_product_images` | Search product catalog by text description | Free |
| `search_inspiration_ads` | Search competitor brand ads from Meta Ad Library | Free |
| `generate_ad_asset` | Generate ad images (with optional style inspiration) | $1/image |
| `list_asset_jobs` | Check status and results of generation jobs | Free |

## Workflow

### 1. Build the product catalog (recommended)

Use `vectorize_product_images` to upload and vectorize product photos. This creates a searchable catalog that can be used when generating ads.

- Accepts 1-50 images per call
- Each image needs an `image_url` (public URL)
- `product_name` is optional — AI vision auto-generates one if omitted
- **Processing is async** — images become searchable in ~1-2 minutes
- Upserts on `image_url` — re-vectorizing the same image updates rather than duplicates

### 2. Find competitor ads for inspiration

Use `search_inspiration_ads` to browse a curated library of brand ads scraped from Meta Ad Library.

- Describe the style you want in natural language (e.g. "minimalist skincare ad with white background", "bold fitness ad with athlete")
- Filter by `brand_name` to see a specific brand's ads
- Filter by `category` (e.g. "Health/beauty", "Apparel")
- Filter by `platform` (FACEBOOK or INSTAGRAM)
- Returns ad images ranked by visual similarity with ad copy and metadata
- Use the returned `image_url` values as `inspiration_image_urls` when generating ads

### 3. Find product images for ads

Use `search_product_images` to find the right product photos from the catalog.

- Describe what you're looking for in natural language (e.g. "blue athletic sneakers", "summer beach accessories")
- Returns results ranked by relevance with similarity scores
- Use the returned `image_url` values as `brand_image_urls` when generating ads

### 4. Generate ad images

Use `generate_ad_asset` to create ad images. Two modes:

**Standard generation** — generate from a text prompt:
```
prompt: "A vibrant summer campaign ad featuring our beach sandals on a tropical background"
brand_image_urls: ["https://...product-photo.jpg"]
aspect_ratio: "9:16"
count: 3
```

**Inspiration-based generation** — emulate another brand's ad style:
```
prompt: "Create an ad for our protein bars in this clean, minimalist style"
inspiration_image_urls: ["https://...competitor-ad.jpg"]
brand_image_urls: ["https://...our-product.jpg"]
aspect_ratio: "1:1"
```

Key parameters:
- `prompt` (required): Detailed description of the ad to generate
- `inspiration_image_urls` (optional): 0-5 reference/competitor ad URLs for style guidance — the AI emulates the visual aesthetic but NOT the content
- `brand_image_urls` (optional): 0-5 brand asset URLs (product photos, logos) to incorporate into the ad
- `aspect_ratio`: 1:1, 2:3, 3:2, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9 (default 1:1)
- `image_size`: 1K, 2K, 4K (default 4K)
- `count`: 1-10 images in parallel (default 1) — use this instead of calling the tool multiple times

**Generation is async.** Tell the user images are processing and will be ready shortly.

### 5. Check results

Use `list_asset_jobs` to check the status of generation jobs.

- Filter by `status`: "processing", "completed", or "failed"
- Shows job IDs, prompts, image URLs, and timestamps
- Default returns the 20 most recent jobs

When presenting completed results:
- Show the generated image URLs
- If multiple images were generated, present them as a numbered list
- If jobs are still processing, tell the user to check back in 30-60 seconds

## Typical workflows

### "Generate ads for my product"
1. `search_product_images` → find relevant product photos
2. `generate_ad_asset` → generate ads with product photos as `brand_image_urls`
3. `list_asset_jobs` → retrieve generated image URLs

### "Make an ad like this competitor's ad"
1. `search_inspiration_ads` → find competitor ads by style or brand name
2. `search_product_images` → find relevant product photos
3. `generate_ad_asset` → pass competitor ad as `inspiration_image_urls`, product photos as `brand_image_urls`
4. `list_asset_jobs` → retrieve generated image URLs

### "Show me what skincare brands are running on Meta"
1. `search_inspiration_ads` → search by category or style description
2. Present results with image URLs, ad copy, and brand names

### "Upload my product photos"
1. `vectorize_product_images` → batch upload and vectorize
2. Wait ~1-2 minutes
3. `search_product_images` → verify images are searchable

## Important notes

- Most tools are org-scoped (users only see their own data). Exception: `search_inspiration_ads` searches a shared library of brand ads available to all customers.
- Mention costs when the user is generating many images ($1 each).
- Write detailed, descriptive prompts for best ad generation results.
- When the user provides inspiration images, emphasize that the AI copies the *style* (layout, colors, typography, vibe) but creates original content.
- If a user asks to "create an ad" with no prior context, ask what product/brand they want to advertise and gather the needed image URLs.
