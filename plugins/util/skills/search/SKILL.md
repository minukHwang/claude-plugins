---
name: search
description: |
  Use when performing any web search (WebSearch, WebFetch).
  Trigger phrases: "검색해봐", "알아봐", "찾아봐", "분석해봐", "search", "look up", "find"
  Ensures Claude uses the current year instead of defaulting to outdated years (e.g., 2024).
---

# Current Date Awareness for Web Search

## MANDATORY: Check Current Date Before Search

Before using `WebSearch` or `WebFetch`, ALWAYS run:

```bash
date +%Y-%m-%d
```

Use the returned year (e.g., 2025) in your search queries.

## Include Current Year in Queries

For time-sensitive topics (tech docs, versions, news, trends), append the current year:

| Bad Query | Good Query |
|-----------|------------|
| "React 19 features" | "React 19 features 2025" |
| "Next.js latest version" | "Next.js latest version 2025" |
| "AI trends" | "AI trends 2025" |

**NEVER assume or hardcode the year. ALWAYS check first.**
