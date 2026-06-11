# steadyfetch n8n templates

Free n8n workflows for ad-creative data pipelines. No community nodes required — plain HTTP, importable into any n8n (cloud or self-hosted).

## Facebook Ad Transcripts — scrape competitor ads → hooks, CTAs & transcripts

**What it does:** searches the Facebook Ad Library for video ads → transcribes every creative → gives you one row per ad with the **first-3-seconds hook**, CTA, advertiser, and full transcript. The manual version of this pipeline is sold as a paid template elsewhere; this one is free.

**[Download the workflow JSON →](./facebook-ad-transcripts.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `facebook-ad-transcripts.workflow.json`.
2. Open the **Config** node and paste your [Apify API token](https://console.apify.com/settings/integrations) into `apifyToken`.
3. (Optional) Edit `adLibrarySearchUrl` — any [Facebook Ad Library](https://www.facebook.com/ads/library/) search URL works (set `media_type=video`), and `maxAds`.
4. Click **Test workflow**.

### What you get per ad

| field | example |
|---|---|
| `status` | `transcribed` (only these are charged) |
| `advertiser` | Native |
| `hook_first_3s` | "Look, women are always right. My girlfriend told me…" |
| `cta` | Shop Now |
| `transcript` | full speech-to-text, any video length |
| `seconds` | 32.8 |
| `charged` | `true` / `false` — expired links & music-only ads are **$0** |

From the last node, wire anything you like: Google Sheets, Airtable, Slack, a database.

### Cost

- Ad scraping: [curious_coder/facebook-ads-library-scraper](https://apify.com/curious_coder/facebook-ads-library-scraper) — $0.75 / 1,000 ads
- Transcription: [steadyfetch/facebook-ads-transcript-scraper](https://apify.com/steadyfetch/facebook-ads-transcript-scraper) — from $15.00 / 1,000 video ad transcripts, **charged only when a transcript is delivered**
- A 20-ad test run ≈ **$0.42**. Apify's free $5 credit covers it many times over.

### Swap-friendly

The transcription step accepts **any** ad-library scraper's output (it deep-scans rows for the video URL + metadata) — if you already use a different scraper, just point its results at the *Transcribe Ads* node body.

---

*More templates coming: TikTok ad transcripts on the same pipeline.*
