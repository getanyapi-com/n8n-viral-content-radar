# Daily Viral Content Radar (n8n + AnyAPI)

A free n8n workflow that runs every morning, pulls the top-performing TikToks and Instagram Reels in your niche, and uses an AI agent to explain why each one went viral and how to replicate it. It emails you the brief and logs every find to Google Sheets.

The whole thing runs on **one API key** for the data (TikTok + Instagram, and 200+ other sources if you want them), pay-per-request in USD, no subscription. A daily run costs about **$0.01** in data.

## What it does

1. **Trigger** - Schedule Trigger fires daily at 8AM. A Set node holds your niche (default: `ai automation`).
2. **Scrape** - two HTTP Request nodes call AnyAPI: `tiktok.search_top` and `instagram.reels_search`. Same key, same response shape.
3. **Rank** - a Code node merges both platforms, sorts by views, and keeps the top 8.
4. **Analyze** - an AI Agent (any model via OpenRouter) writes, for each video, why it likely went viral and a 3-step plan to replicate it, plus a "pattern of the day" takeaway.
5. **Deliver** - emails you the brief and appends every video to a Google Sheet so ideas stack up over time.

## Setup

1. **Import** `viral-content-radar.workflow.json` into n8n (Workflows > Import from File).
2. **AnyAPI key** - create one at [getanyapi.com](https://getanyapi.com). Both HTTP Request nodes use a single **Header Auth** credential:
   - Name: `Authorization`
   - Value: `Bearer YOUR_ANYAPI_KEY`
3. **OpenRouter** - add an OpenRouter credential on the model node, pick any model (default `openai/gpt-4.1-mini`).
4. **Gmail** and **Google Sheets** - connect your Google credentials, set the recipient email and target spreadsheet.
5. Set your **niche** in the Set node, then activate.

## Swap in anything

The two scrape nodes are plain HTTP calls to `https://api.getanyapi.com/v1/run/<sku>`. Change the SKU to pull from YouTube, Reddit, X, Google News, and more, all on the same key and the same `{ output, costUsd }` response. Browse the catalog at [getanyapi.com](https://getanyapi.com).

## Cost

Data is pay-per-request in USD. The default daily run (top TikToks + a week of Reels) is about $0.01, plus whatever your chosen model costs for one short brief. No subscription.

## License

MIT
