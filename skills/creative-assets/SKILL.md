---
name: gentic-creative-assets
description: Guides the creative asset workflow using Gentic MCP tools. Activates when users want to generate ad images, video clips, vectorize product photos, search their product catalog, or find competitor ad inspiration.
license: MIT
metadata:
  author: gentic
  version: "2.0.0"
---

# Gentic Creative Assets

Workflow guide for generating ad images, video clips, and managing a searchable product image catalog using Gentic's MCP tools. Steps build on each other — follow the order below.

## When to apply

- User wants to generate ad images or creative assets
- User wants to generate short video clips
- User wants to create ads inspired by another brand's style
- User wants to upload or vectorize product photos
- User wants to search their product image catalog
- User wants to check the status of image or video generation jobs
- User wants to browse or search competitor brand ads for inspiration

## Tools reference

| Tool | Purpose | Cost |
|------|---------|------|
| `fetch_page` | Scrape a web page for brand/product context | 10¢/page |
| `vectorize_product_images` | Batch vectorize product photos for searchable catalog | 5¢/image |
| `search_product_images` | Search product catalog by text description | Free |
| `search_inspiration_ads` | Search competitor brand ads from Meta Ad Library | Free |
| `generate_ad_asset` | Generate ad images (with optional style inspiration) | $1/image |
| `generate_video_clip` | Generate short AI video clips (4-8 seconds) | ~$1.50-$10/clip |
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
- Use the returned `image_url` values as `brand_image_urls` when generating ads or as `reference_image_urls` when generating video clips

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
- `webhook_url` (optional): URL to receive a POST with job results on completion (useful for n8n/automation workflows)

**Generation is async.** Tell the user images are processing. They will receive an email notification when the job completes.

### 5. Generate video clips

Use `generate_video_clip` to create short AI-generated video clips (4-8 seconds) using Google Veo 3.1.

**Simple prompt (auto-enhanced):**
```
prompt: "Our protein bar being unwrapped on a kitchen counter, warm morning light"
aspect_ratio: "9:16"
duration: "8"
tone: "warm and cozy"
platform: "instagram_reels"
```

Key parameters:
- `prompt` (required): Text description of the video — can be simple, the AI cinematographer enhances it automatically
- `aspect_ratio`: 16:9 (landscape) or 9:16 (portrait/vertical) — default 16:9
- `duration`: 4, 6, or 8 seconds — default 8
- `resolution`: 720p, 1080p, or 4k — default 720p (1080p/4k require 8s duration)
- `quality`: "fast" (~$1.50-3.00, faster) or "full" (~$5-10, highest quality) — default fast
- `reference_image_urls` (optional): 0-3 reference image URLs for style/character consistency
  - 1 image: uses image-to-video mode (animates the image directly, supports fast model + any duration)
  - 2-3 images: uses reference mode (requires full model + 8s duration, auto-upgraded)
- `enhance_prompt` (default true): Automatically transforms simple prompts into cinematic, Veo-optimized prompts using Gemini. Set to false for full manual control.
- `tone` (optional): Creative mood — e.g. "luxurious", "playful", "raw/authentic", "cinematic", "energetic"
- `platform` (optional): Target platform — instagram_reels, tiktok, youtube_preroll, youtube_shorts. Influences pacing and framing conventions.
- `webhook_url` (optional): URL to receive a POST with job results on completion

**Generation is async and takes 1-6 minutes.** Tell the user the video is processing. They will receive an email notification with a preview when the job completes.

**Prompt enhancement:** By default, even a simple prompt like "product on a table" gets transformed into a detailed cinematic prompt with camera movements, lighting, timeline breakdown, and Veo-specific optimizations. The enhanced prompt is included in the completion email so users can learn from it.

### 6. Check results

Use `list_asset_jobs` to check the status of image and video generation jobs.

- Filter by `status`: "processing", "completed", or "failed"
- Filter by `type`: "image" or "video"
- Shows job IDs, prompts, asset URLs, and timestamps
- Default returns the 20 most recent jobs

When presenting completed results:
- Show the generated image/video URLs
- If multiple assets were generated, present them as a numbered list
- If jobs are still processing, tell the user they'll get an email when it's done

## Notifications

Both `generate_ad_asset` and `generate_video_clip` automatically send **email notifications** when jobs complete. The email includes:
- Inline image preview or video link
- The prompt used
- Aspect ratio, duration, resolution metadata
- The enhanced prompt (for video clips) so users can learn what good prompting looks like
- Error details if the job failed

For **automation workflows** (n8n, etc.), pass a `webhook_url` parameter to receive a POST with the full job payload:
```json
{
  "job_id": "uuid",
  "status": "completed",
  "type": "image | video",
  "asset_url": "https://...",
  "mime_type": "image/png",
  "prompt": "the original prompt",
  "enhanced_prompt": { ... },
  "aspect_ratio": "16:9",
  "duration_seconds": 8,
  "resolution": "720p",
  "error": null
}
```

## Typical workflows

### "Generate ads for my product"
1. `search_product_images` → find relevant product photos
2. `generate_ad_asset` → generate ads with product photos as `brand_image_urls`
3. `list_asset_jobs` → retrieve generated image URLs (or wait for email)

### "Make an ad like this competitor's ad"
1. `search_inspiration_ads` → find competitor ads by style or brand name
2. `search_product_images` → find relevant product photos
3. `generate_ad_asset` → pass competitor ad as `inspiration_image_urls`, product photos as `brand_image_urls`
4. `list_asset_jobs` → retrieve generated image URLs (or wait for email)

### "Create a product video for Instagram"
1. `search_product_images` → find the product photo
2. `generate_video_clip` → pass product photo as `reference_image_urls`, set platform to "instagram_reels", aspect_ratio to "9:16"
3. Wait for email notification with video preview

### "Create a cinematic brand video"
1. `generate_video_clip` → describe the scene, set quality to "full", tone to "cinematic"
2. Wait for email notification with video preview and enhanced prompt details

### "Show me what skincare brands are running on Meta"
1. `search_inspiration_ads` → search by category or style description
2. Present results with image URLs, ad copy, and brand names

### "Upload my product photos"
1. `vectorize_product_images` → batch upload and vectorize
2. Wait ~1-2 minutes
3. `search_product_images` → verify images are searchable

## Important notes

- Most tools are org-scoped (users only see their own data). Exception: `search_inspiration_ads` searches a shared library of brand ads available to all customers.
- Mention costs when the user is generating many images ($1 each) or videos ($1.50-$10 each depending on quality and duration).
- Users don't need to write sophisticated prompts for video — the AI cinematographer handles prompt enhancement automatically. Simple descriptions work great.
- When the user provides inspiration images for ads, emphasize that the AI copies the *style* (layout, colors, typography, vibe) but creates original content.
- If a user asks to "create an ad" or "make a video" with no prior context, ask what product/brand they want to feature and gather the needed image URLs.
