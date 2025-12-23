# Util Plugin

Utility tools for Claude Code.

## Version

1.0.0

## Skills

| Skill | Description |
|-------|-------------|
| `search` | Current date awareness for web searches |

## search

Ensures Claude uses the **current year** when performing web searches, preventing outdated results (e.g., defaulting to 2024 instead of 2025).

**Trigger phrases:**
- Korean: "검색해봐", "알아봐", "찾아봐", "분석해봐"
- English: "search", "look up", "find"

**How it works:**
1. Before any `WebSearch` or `WebFetch`, runs `date +%Y-%m-%d`
2. Uses the current year in search queries

**Example:**

| User Request | Search Query |
|--------------|--------------|
| "React 19 알아봐" | "React 19 features 2025" |
| "Next.js 최신 버전 찾아봐" | "Next.js latest version 2025" |
| "AI 트렌드 검색해봐" | "AI trends 2025" |

## Installation

```bash
# Add marketplace
/plugin marketplace add minukHwang/claude-plugins

# Install plugin
/plugin install util@minukHwang-plugins
```

## License

MIT
