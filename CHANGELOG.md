# Changelog

## v2026.4.13

Fix stale/repeated stories appearing in the daily digest.

**Root cause:** Brave Search was returning homepage and section-page URLs (e.g. `apnews.com/`, `axios.com/`, `foxnews.com/`). The script fetched these pages, extracted text containing multiple stories -- including old ones still featured on the homepage -- and Claude picked from that text. Pre-Claude dedup couldn't catch these because it compared Brave's generic page titles, not the headlines Claude would ultimately generate.

**Changes:**

- Add `is_article_url()` + `filter_homepage_urls()` -- filters out homepage and section-page URLs before they become candidates. Only URLs with article slugs, date patterns, or long hash IDs pass through. This eliminates the #1 source of stale stories.
- Add `post_claude_dedup()` -- a final dedup pass that runs AFTER Claude generates headlines, comparing them against the previous 3 days. Stale repeats are removed; stories with genuine new developments get an "Update: " prefix.
- Tighten cross-day dedup thresholds -- similarity threshold lowered from 0.55 to 0.45, keyword overlap from 4 to 3 shared words.
- Trim `UPDATE_INDICATOR_WORDS` -- removed overly common words (confirmed, closed, response, fallout, killed, etc.) that were false-positive-ing nearly every headline through the update bypass.
- Strengthen Claude's prompt -- added explicit freshness check instruction, stricter previous-days rules, and "Update: " headline prefix requirement for follow-up stories.

## 1.0.0 (2026-04-12)

Initial versioned release. The site has been running since April 4, 2026. This release captures the current state as v1.0.0.

- Automated 6x/day national and world news digest
- Brave Search API + Google News RSS for news gathering
- Claude Sonnet 4.6 for curation and multi-paragraph summarization
- Balanced left/center/right source strategy
- Multi-pass deduplication with update-detection keywords
- Overnight catch-up mode on first daily run
- Cookie-based read tracking (iOS home screen persistent)
- Deep linking support for individual stories
- 14-day rolling JSON archive
- DST-aware dual cron schedules
- Custom domain: 101news.org
- README with setup docs, cost estimate, and architecture
