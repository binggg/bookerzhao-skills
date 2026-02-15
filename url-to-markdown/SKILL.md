---
name: url-to-markdown
description: Convert any webpage URL to clean Markdown using markdown.new (Cloudflare). Use when the user or agent needs to get a URL as Markdown (e.g. for summarization, RAG, context, or saving as .md). Works via URL prefix or fetch with Accept text/markdown.
---

# URL to Markdown (markdown.new)

Convert any URL to clean Markdown — ~80% fewer tokens than raw HTML, no parsing. Powered by Cloudflare (Markdown for Agents).

## When to Use

- User asks to "turn a webpage into markdown", "convert URL to markdown", "get page content as markdown"
- RAG / context: need clean document text from a URL
- Saving or archiving a page in readable Markdown

## How to Use

### 1. URL prefix (simplest)

Prepend `https://markdown.new/` to any URL:

```
https://markdown.new/https://blog.cloudflare.com/markdown-for-agents/
```

Or in browser: go to https://markdown.new and paste the target URL.

### 2. Fetch as Markdown (for agents/scripts)

Request the URL through markdown.new so the response is already Markdown:

```bash
# Any URL → Markdown (markdown.new proxies and returns text/markdown)
curl -sL -H "Accept: text/markdown" "https://markdown.new/https://example.com/page"
```

Or in code:

```javascript
const url = "https://example.com/article";
const res = await fetch(`https://markdown.new/${url}`, {
  headers: { Accept: "text/markdown" },
});
const markdown = await res.text();
const tokenEstimate = res.headers.get("x-markdown-tokens"); // optional
```

### 3. Direct content negotiation (if target is Cloudflare-enabled)

If the target site supports it, you can skip markdown.new and request Markdown directly:

```javascript
fetch(targetUrl, { headers: { Accept: "text/markdown" } });
```

Otherwise use markdown.new as above.

## What You Get

- **Body**: Clean Markdown (often with YAML-like metadata block at top).
- **Header**: `x-markdown-tokens` — estimated token count (for context-window planning).
- **Content-Type**: `text/markdown; charset=utf-8`.

## Pipeline (reference)

markdown.new uses a three-tier pipeline: (1) Markdown for Agents / content negotiation, (2) Workers AI `toMarkdown()` on HTML, (3) Browser Rendering for JS-heavy pages. No need to implement this yourself — just use the URL or fetch pattern above.

## Built For

AI agents, RAG pipelines, documentation migration, knowledge bases, content archival. Prefer this over feeding raw HTML to models when you need structured page content.
