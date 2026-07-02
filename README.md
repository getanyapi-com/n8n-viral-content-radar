# Daily Viral Content Radar (n8n + AnyAPI)

Drop in your website URL once. Every morning at 8am, the workflow searches this week's top
TikTok, Instagram Reels, YouTube and X for the search terms your audience actually uses,
filters out the junk with an LLM, reads the top videos' actual transcripts, and emails you a
brief: why each one went viral, the angle for YOUR business, and a ready-to-shoot script.
Every winner is also logged to a Google Sheet so a swipe file builds itself.

Two workflows in this repo:

| File | What it is |
|---|---|
| `viral-content-radar-pro.workflow.json` | **The full radar** (recommended): website onboarding, 4 platforms, LLM relevance filter, transcripts, script writer, email + Sheets |
| `viral-content-radar.workflow.json` | The lite version: fixed niche, TikTok + Reels, rank by views, one AI brief, email + Sheets |

## How the Pro workflow works

**Setup - run once**

- Fill in a form with your website URL. An AnyAPI node scrapes the site and an LLM writes a
  profile of your actual business: what you sell, who it's for, and the 5-7 search phrases
  your audience really uses (for a couples budgeting app it came back with "money date",
  "sinking funds", "cancel subscriptions" - not "personal finance"). Saved to an n8n Data
  Table so it steers every morning's run.

**Every morning**

1. **Your niche** - reads the saved profile (safe default if empty) and expands it into the
   exact search terms a creator in your space would use.
2. **Find what's hot** - pulls this week's top TikToks, Reels, YouTube and X for every term.
   One AnyAPI key, same `{ output, costUsd }` response shape across all of them.
3. **Cut the noise** - pools ~30 candidates and an LLM filter drops everything that only
   hashtag-matched: course funnels, "things I bought" hauls, big-brand ads, mindset fluff.
4. **Read why it won** - an agent pulls the top videos' actual spoken transcripts (it picks
   the right platform transcript API itself), then writes what each video is, why it spread,
   the specific angle for your business, and a 3-step shoot plan.
5. **Your morning brief** - one 8am email: the landscape read, the picks with reach numbers,
   and a ready-to-shoot 30-45s script for the best idea.
6. **Archive to Sheets** - every selected winner is appended to a Google Sheet (date,
   platform, author, link, why it worked).

### Why the LLM filter step exists (measured)

We benchmarked the "what's worth surfacing" step against a hand-researched ground-truth
top-8 across three niches. A views+engagement+keyword heuristic got **46% overlap** with
ground truth and let **10 junk videos** through (funnels, non-English ads, hauls). Swapping
that one step for an LLM filter with hard exclude rules got **79% overlap and zero junk**.
Engagement math cannot tell craft from grift; a model reading the captions can.

## Setup (Pro)

1. **Import** `viral-content-radar-pro.workflow.json` into n8n (Workflows > Import from File).
2. **Install the AnyAPI community node**: Settings > Community Nodes > install
   `n8n-nodes-anyapi`. Create an **AnyAPI API** credential with a key from
   [getanyapi.com](https://getanyapi.com) and select it on every AnyAPI node. That one key
   covers the site scrape, all 4 platform searches, and the transcript fetches.
3. **OpenRouter** - add an OpenRouter credential on the model nodes. The relevance filter and
   analyst need a smart model (we use `deepseek/deepseek-v4-pro`); the writing steps run fine
   on a fast one (`deepseek/deepseek-v4-flash`).
4. **Gmail + Google Sheets** - connect your Google credentials, set the recipient email and
   target spreadsheet.
5. **Data Table** - create an n8n Data Table named `niche_store` with columns: `key`,
   `niche`, `keywords`, `business`, `audience`.
6. Run the "Onboard Your Website" form once with your URL, then activate the schedule.

## Setup (lite)

1. Import `viral-content-radar.workflow.json`.
2. Install `n8n-nodes-anyapi` and select an AnyAPI API credential on the two AnyAPI nodes
   (`tiktok.search_top` and `instagram.reels_search`).
3. Add an OpenRouter credential on the model node, connect Gmail + Sheets, set your niche in
   the Set node, and activate.

## Swap in anything

The scrape nodes are AnyAPI nodes: change the API in the node's dropdown to pull from
YouTube, Reddit, X, Google News, and 1,200+ other sources - same key, same
`{ output, costUsd }` response shape. Browse the catalog at
[getanyapi.com](https://getanyapi.com).

## Cost

Data is pay-per-request in USD, no subscription. A full Pro morning run is a few cents
across the 4 platforms plus the two transcripts; the lite run is about $0.01. Model costs
depend on your OpenRouter choice.

## Honest caveats

- The Pro daily run takes a few minutes (sequential platform searches + transcript fetches).
- The platform searches occasionally 502; the AnyAPI nodes are set to retry.
- The filter is only as good as the business profile - if your website is thin, edit the
  saved `niche_store` row by hand.

## License

MIT
