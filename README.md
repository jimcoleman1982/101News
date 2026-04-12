# 101 News

Automated national and world news digest. Updates 6 times daily, entirely on GitHub infrastructure.

Every few hours from 5 AM to 9 PM Denver time, a GitHub Actions workflow gathers the top US and world news stories, summarizes them via the Anthropic API (Claude Sonnet 4.6), and publishes them to a static site hosted on GitHub Pages.

**Live site:** [101news.org](https://101news.org)

## How It Works

1. **GitHub Actions** fires 6x/day on a cron schedule (DST-aware dual schedules for MDT/MST)
2. **Python script** searches for national/world news via Brave Search API + Google News RSS
3. Fetches article content from AP, Reuters, NPR, BBC, WSJ, and other major sources
4. Sends candidates to **Claude Sonnet 4.6** for curation, categorization, and multi-paragraph summarization
5. Writes/updates a dated JSON file in `data/`
6. Commits and pushes -- **GitHub Pages** auto-deploys the updated site

## Features

- **Rolling updates** -- stories accumulate through the day (2-5 new stories per run)
- **Balanced sourcing** -- intentionally draws from left, center, and right-leaning outlets
- **Smart deduplication** -- multi-pass similarity + keyword matching, with update detection (arrests, rulings, etc. override cross-day dedup)
- **Overnight catch-up** -- first run of the day extends the freshness window to 12 hours
- **Read tracking** -- cookie-based headline tracking that persists on iOS home screen
- **14-day archive** -- older data auto-pruned on each run
- **Deep linking** -- shareable URLs for individual stories

## Categories

- Politics and Government
- World News
- Business and Economy
- Technology
- Science and Health
- National News (general)

## Setup

### Prerequisites

1. **GitHub account** with a private repo
2. **Anthropic API key** -- [console.anthropic.com](https://console.anthropic.com/)
3. **Brave Search API key** -- [brave.com/search/api](https://brave.com/search/api/) (free tier: 2,000 queries/month)

### Step-by-Step

1. **Clone or push all project files** to the repo

2. **Add API key secrets:**
   - Go to repo **Settings** > **Secrets and variables** > **Actions**
   - Add `ANTHROPIC_API_KEY` with your Anthropic key
   - Add `BRAVE_SEARCH_API_KEY` with your Brave Search key

3. **Enable GitHub Pages:**
   - Go to repo **Settings** > **Pages**
   - Source: **Deploy from a branch**
   - Branch: `main`
   - Folder: `/ (root)`
   - Click **Save**

4. **Configure custom domain** (optional):
   - Add a `CNAME` file to the repo root with your domain
   - Point your domain's DNS to GitHub Pages (see GitHub docs)

5. **Test it:**
   - Go to the **Actions** tab
   - Select **The Daily Briefing**
   - Click **Run workflow** > **Run workflow**
   - After it completes, check `data/` for the new JSON file
   - Visit the site to see the results

## Cost Estimate

| Service | Cost |
|---------|------|
| GitHub Actions | Free (well within 2,000 min/month free tier) |
| GitHub Pages | Free |
| Anthropic API | ~$0.50-$1.50/day (Sonnet 4.6, 6 runs/day) |
| Brave Search API | Free tier (2,000 queries/month, uses ~1,200-1,400/month) |
| **Monthly total** | **~$15-$45/month** |

## Architecture

```
101News/
├── .github/workflows/daily-digest.yml   (6x/day cron, DST-aware)
├── scripts/
│   └── generate_digest.py               (news search + curation + summarization)
├── data/                                (daily JSON files, 14-day rolling)
├── index.html                           (single-page static site)
├── CNAME                                (custom domain: 101news.org)
├── requirements.txt
├── CHANGELOG.md
└── README.md
```

## Budget Safety

- `MAX_BRAVE_QUERIES_PER_RUN = 20` hard cap per run
- `MAX_ANTHROPIC_CALLS_PER_RUN = 5` hard cap per run
- `ANTHROPIC_MAX_TOKENS = 8000` output token limit
- Retry logic with 30-second backoff
- Token usage and cost logged in every run

## Update Schedule

Runs 6 times daily at Denver local time:
- 5:07 AM, 8:07 AM, 11:07 AM, 2:07 PM, 6:07 PM, 9:07 PM

Dual cron entries handle MST/MDT transitions automatically. The script detects which timezone is active and skips the wrong cron trigger.
