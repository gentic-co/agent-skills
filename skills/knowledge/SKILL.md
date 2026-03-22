---
name: gentic-knowledge
description: Guides the knowledge base workflow using Gentic MCP tools. Activates when users want to vectorize documents, web pages, or text content for semantic search across their organization's knowledge.
license: MIT
metadata:
  author: gentic
  version: "1.0.0"
---

# Gentic Knowledge Base

Workflow guide for building and searching a semantic knowledge base using Gentic's MCP tools. Vectorize documents, web pages, emails, and other content — then search across everything with natural language.

## When to apply

- User wants to add documents, web pages, or text content to their knowledge base
- User wants to search their knowledge base
- User wants to see what content has been indexed
- User wants to delete content from their knowledge base
- User mentions "knowledge base", "vectorize", "index", or "semantic search"
- User wants to store and search emails, Slack messages, reviews, notes, or other text

## Tools reference

| Tool | Purpose | Cost |
|------|---------|------|
| `vectorize_document` | Vectorize a PDF, TXT, RTF, or DOCX file | 5¢/chunk |
| `vectorize_content` | Vectorize raw text (emails, notes, reviews, etc.) | 5¢/chunk |
| `vectorize_web_content` | Scrape and vectorize a web page | 5¢/chunk + 5¢ scraping fee |
| `search_knowledge` | Semantic search across all indexed content | Free |
| `delete_content` | Soft-delete content by document ID or chunk IDs | Free |
| `list_kb_sources` | List all indexed sources with chunk counts | Free |

A chunk is approximately 1,000 characters. A short email is typically 1 chunk. A 10-page document might produce 20-40 chunks.

## Workflow

### 1. Index content

Choose the right tool based on the content type:

**Documents** — use `vectorize_document`:
- Supports PDF, TXT, RTF, and DOCX files
- Accepts direct HTTP URLs and Google Drive links
- Provide an optional `title` and `category` for better organization

**Web pages** — use `vectorize_web_content`:
- Provide the URL — the page is scraped, cleaned, chunked, and embedded automatically
- Thin pages are retried with JavaScript rendering
- Provide an optional `category` for filtering

**Text content** — use `vectorize_content`:
- For emails, Slack messages, reviews, social posts, notes, and other raw text
- Requires `source_type`: email, slack, review, social, creator_feedback, note, or other
- Provide optional `title`, `source_id`, `category`, and `created_at` for metadata
- Content is deduplicated by `source_id` (if provided) or content hash

All vectorize tools are **async** — content is processed in the background and becomes searchable once complete. Use `list_kb_sources` to confirm indexing finished.

### 2. Search the knowledge base

Use `search_knowledge` to find relevant content with natural language queries.

Key parameters:
- `query` (required): Natural language search query — be descriptive for better results
- `source_types` (optional): Filter by type — e.g. `["document", "web"]` or `["email", "slack"]`
- `category` (optional): Filter by category
- `document_id` (optional): Search within a specific document
- `limit` (optional): Number of results (default 10, max 50)

When presenting search results:
- Show the most relevant text snippets with their source info (title, type, category)
- Include similarity scores to indicate relevance
- If searching across many sources, group results by document or source type
- Mention the total result count

### 3. Manage content

**List sources** — use `list_kb_sources` to see everything indexed:
- Shows each document/content with its source type, title, chunk count, category, and date
- Use this to verify indexing completed or to find document IDs for deletion

**Delete content** — use `delete_content` to remove content:
- Delete by `document_id` to remove all chunks for a source
- Delete by `content_ids` to remove specific chunks
- Content is soft-deleted (excluded from search but retained in storage)

## Typical workflows

### "Add this document to my knowledge base"
1. `vectorize_document` with the document URL
2. Tell the user it's processing and will be searchable shortly
3. `list_kb_sources` to confirm indexing completed

### "Index this web page"
1. `vectorize_web_content` with the page URL
2. Tell the user it's processing (mention the 5¢ scraping fee + per-chunk cost)

### "Store these customer emails for search"
1. `vectorize_content` for each email with `source_type: "email"` and a `source_id` (e.g. email thread ID) to prevent duplicates
2. Use `category` to organize (e.g. "support", "feedback", "complaints")

### "Search my knowledge base for..."
1. `list_kb_sources` first if you're unsure what's been indexed
2. `search_knowledge` with a descriptive query
3. Use `source_types` or `category` filters to narrow results if needed

### "What's in my knowledge base?"
1. `list_kb_sources` to show all indexed content
2. Present as a table: title, source type, chunk count, category, date

### "Remove this document from my knowledge base"
1. `list_kb_sources` to find the `document_id`
2. `delete_content` with the `document_id`
3. Confirm deletion count

## Important notes

- All tools are org-scoped. Users only see their own knowledge base.
- Vectorization is async — content won't appear in search results immediately. Processing typically takes 10-60 seconds depending on content size.
- Each chunk is ~1,000 characters. Mention estimated costs for large documents (e.g. "This 50-page PDF will likely produce ~150 chunks at 5¢ each = ~$7.50").
- Searching is always free — encourage users to search often.
- Content is deduplicated by document ID. Re-vectorizing the same URL or source_id updates rather than duplicates.
- The `category` field is useful for filtering searches later — encourage users to set it when indexing.
