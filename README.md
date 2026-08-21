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

## Competitor ad teardown — one competitor page/keyword → hooks, CTAs & transcripts, one row per ad

**What it does:** point it at one competitor (their Facebook Page URL or an Ad Library search URL) → scrapes their active video ads → chains the transcript actor **by dataset ID** (no ad rows pass through n8n, so big batches stay light) → gives you a spreadsheet-shaped teardown: one row per ad with the **first-3-seconds hook**, CTA, transcript length in words, language, seconds, and the full transcript. Transcribed ads first, longest first.

**[Download the workflow JSON →](./competitor-ad-teardown.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `competitor-ad-teardown.workflow.json`.
2. Open the **Config** node and paste your [Apify API token](https://console.apify.com/settings/integrations) into `apifyToken`.
3. Set `competitorUrl` — a competitor's Facebook Page URL (e.g. `https://www.facebook.com/nike`) or any [Ad Library](https://www.facebook.com/ads/library/) search URL (set `media_type=video`) — and `maxAds`.
4. Click **Test workflow**. The flow polls the scraper run until it finishes, then transcribes.

### What you get per ad

| field | example |
|---|---|
| `advertiser` | Nike |
| `ad_id` | 1318059422847100 |
| `status` | `transcribed` (only these are charged) |
| `hook_first_3s` | "Your sister's on the phone." |
| `cta` | Shop Now |
| `seconds` | 36.7 |
| `transcript_words` | 58 |
| `language` | en |
| `charged` | `true` / `false` — expired links & music-only ads are **$0** |
| `transcript` | full speech-to-text, any video length |

The last node is a plain Set node (no credentials needed) — wire Google Sheets, Airtable, Slack, or a database after it.

### Swap the scraper

The scrape step is a plain HTTP call to `curious_coder/facebook-ads-library-scraper` (`{ "urls": [{ "url": ... }], "count": N }`). Any other Ad Library scraper works too: change the actor slug and JSON body in the *Start Ad Library scrape* node — the transcript step reads the run's dataset ID and deep-scans rows for the video URL, whatever the field names.

### Validated

Both workflows import cleanly into n8n (last checked against n8n 2.35.6 via `n8n import:workflow`).

---

*More templates coming: TikTok ad transcripts on the same pipeline.*
