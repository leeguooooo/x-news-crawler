# X News Crawler

Local X (Twitter) news crawler for keyword-based signal collection.

This project uses `abs` (`agent-browser-stealth`) to crawl X search pages (`top`/`latest`), then outputs deduplicated JSON for downstream analysis and sync jobs.

## Features

- `top`, `latest`, and `hybrid` crawl modes
- Deduplicate by tweet status URL
- Sort by datetime and filter by recency window
- Partial fallback in `hybrid` mode (one source fails, one still returns)
- Structured output with `warnings` and `failed_sources`

## Required Setup

Run these before any crawl:

```bash
pnpm add -g agent-browser-stealth
pnpm approve-builds -g
```

Also required:

- `jq`
- `python3`

Enable CDP on your regular Chrome profile (no isolated profile):

```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9333
```

Do not pass `--user-data-dir` when launching Chrome for crawler usage.  
Using `--user-data-dir` creates a separate browser profile and loses your usual login/session state.

## Quick Start

From repo root:

```bash
./bin/x-news-crawler --query "openclaw" --mode hybrid --since-hours 72 --limit 30 --output .tmp/openclaw-news.json
```

Compatibility alias (old name):

```bash
./bin/x-news-crawl --query "openclaw" --mode latest --since-hours 24 --limit 20
```

## CLI Flags

- `--query <text>`: required keyword
- `--mode <hybrid|top|latest>`: default `hybrid`
- `--since-hours <int>`: keep rows newer than this window, default `72`
- `--limit <int>`: max output rows, default `30`
- `--scrolls <int>`: scroll rounds before extraction, default `4`
- `--cdp <port|url>`: abs CDP endpoint, default `9333`
- `--output <file>`: write JSON to file (stdout if omitted)

Debug:

- `X_NEWS_FORCE_FAIL_SOURCE=top|latest`: force source failure for fallback testing

## Output Shape

```json
{
  "fetched_at": "2026-03-04T02:00:00Z",
  "query": "openclaw",
  "mode": "hybrid",
  "since_hours": 72,
  "count": 12,
  "rows": [
    {
      "source": "latest",
      "user": "openclaw",
      "datetime": "2026-03-04T01:52:00.000Z",
      "status_url": "https://x.com/openclaw/status/123",
      "text": "....",
      "replies": "12",
      "reposts": "3",
      "likes": "56"
    }
  ],
  "warnings": [],
  "failed_sources": []
}
```

## Validate

```bash
python3 /Users/leo/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/x-news-crawler
./skills/x-news-crawler/scripts/smoke_test.sh
```
