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

---

## Instagram Reel Transcripts — scrape a profile's reels → hooks & transcripts

**What it does:** point it at an Instagram profile (or a list of reel URLs) → scrapes the profile's latest reels with [apify/instagram-reel-scraper](https://apify.com/apify/instagram-reel-scraper) → chains [steadyfetch/instagram-reel-transcript-scraper](https://apify.com/steadyfetch/instagram-reel-transcript-scraper) **by dataset ID** (no reel rows pass through n8n, so big batches stay light) → gives you a spreadsheet-shaped table: one row per reel with the **first-3-seconds hook**, the full audio transcript (what is *said*, not the caption), transcript length in words, language, seconds, creator, caption and post URL. Transcribed reels first, longest first.

**[Download the workflow JSON →](./instagram-reel-transcripts.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `instagram-reel-transcripts.workflow.json`.
2. Open the **Config** node and paste your [Apify API token](https://console.apify.com/settings/integrations) into `apifyToken`.
3. Set `instagramProfile` — a username (e.g. `nike`), a profile URL, or a direct reel URL — and `maxReels`.
4. Click **Test workflow**. The flow polls the scraper run until it finishes, then transcribes by dataset ID.

### What you get per reel

| field | example |
|---|---|
| `creator` | nike |
| `short_code` | DNr3dKxRq_s |
| `status` | `transcribed` (only these are charged) |
| `hook_first_3s` | "20 years in this jersey." |
| `seconds` | 34.9 |
| `transcript_words` | 58 |
| `language` | English |
| `charged` | `true` / `false` — expired links, music-only reels and image posts carry no result fee |
| `transcript` | full speech-to-text, any video length |
| `caption` | the post caption (kept separate — it is **not** the transcript) |
| `post_url` | https://www.instagram.com/reel/DNr3dKxRq_s/ |

The last node is a plain Set node (no credentials needed) — wire Google Sheets, Airtable, Slack, or a database after it.

### Cost

- Reel scraping: [apify/instagram-reel-scraper](https://apify.com/apify/instagram-reel-scraper) — pay-per-event (≈ $2.60 / 1,000 reels at the time of writing; check its Pricing tab)
- Transcription: [steadyfetch/instagram-reel-transcript-scraper](https://apify.com/steadyfetch/instagram-reel-transcript-scraper) — from $10.00 / 1,000 reel transcripts, **charged only when a transcript is delivered**
- A 10-reel test run ≈ **$0.18**. Apify's free $5 credit covers it many times over.

### Swap the scraper

The scrape step is a plain HTTP call to `apify/instagram-reel-scraper` (`{ "username": [...], "resultsLimit": N }`). Any Instagram scraper whose rows carry `videoUrl` / `audioUrl` works — `apify/instagram-scraper`, `apify/instagram-post-scraper`, `apify/instagram-api-scraper` and others: change the actor slug and JSON body in the *Scrape the profile's reels* node. The transcript step reads the run's dataset ID and deep-scans rows for the reel media URL, whatever the field names (reels nested inside profile results included).

### Freshness

Instagram's CDN links expire ~1–3 days after scraping — run the transcript step right after the scrape (this flow does). Expired rows come back as uncharged `unavailable_expired` with the exact expiry time.

---

## TikTok Top Ads transcripts — Creative Center industry/region → hooks & transcripts

**What it does:** pick a Creative Center market, lookback window and industry → scrapes the **Top Ads** with [azzouzana/tiktok-creative-center-top-ads-scraper](https://apify.com/azzouzana/tiktok-creative-center-top-ads-scraper) → chains [steadyfetch/tiktok-ads-transcript-scraper](https://apify.com/steadyfetch/tiktok-ads-transcript-scraper) **by dataset ID** right away (no ad rows pass through n8n, and TikTok's ~6-hour video links are still fresh) → gives you a spreadsheet-shaped table: one row per ad with the **first-3-seconds hook**, CTR, likes, brand, ad title, transcript length in words, language, seconds and the full transcript. Transcribed ads first, highest CTR first.

**[Download the workflow JSON →](./tiktok-ad-transcripts.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `tiktok-ad-transcripts.workflow.json`.
2. Open the **Config** node and paste your [Apify API token](https://console.apify.com/settings/integrations) into `apifyToken`.
3. Set `countryCode` (e.g. `US`, `GB`, `DE`, `SA`), `period` (`7`, `30` or `180` days), `industry` (`All industries` or one Creative Center industry name) and `maxAds`.
4. Click **Test workflow**. The flow polls the scraper run until it finishes, then transcribes by dataset ID.

### What you get per ad

| field | example |
|---|---|
| `brand` | Promote |
| `ad_title` | "Why Promote Stands Out…" |
| `material_id` | 7662340784365568017 |
| `status` | `transcribed` (only these are charged) |
| `hook_first_3s` | "Looking for more ways to support your growth on TikTok?" |
| `ctr` | 0.39 |
| `likes` | 19 |
| `seconds` | 44.2 |
| `transcript_words` | 61 |
| `language` | English |
| `charged` | `true` / `false` — expired links and music-only ads carry no result fee |
| `transcript` | full speech-to-text, any video length |

The last node is a plain Set node (no credentials needed) — wire Google Sheets, Airtable, Slack, or a database after it. Expect a fair share of uncharged `no_audio` rows — many Top Ads are music plus on-screen text.

### Cost

- Top Ads scraping: [azzouzana/tiktok-creative-center-top-ads-scraper](https://apify.com/azzouzana/tiktok-creative-center-top-ads-scraper) — $0.50 / 1,000 ads at the time of writing (check its Pricing tab)
- Transcription: [steadyfetch/tiktok-ads-transcript-scraper](https://apify.com/steadyfetch/tiktok-ads-transcript-scraper) — from $15.00 / 1,000 ad video transcripts, **charged only when a transcript is delivered**
- A 25-ad test run ≈ **$0.51 at most** (less when some ads are music-only). Apify's free $5 credit covers it many times over.

### Swap the scraper

The scrape step is a plain HTTP call to `azzouzana/tiktok-creative-center-top-ads-scraper` (`{ "countryCode", "period", "industry", "maxItems" }`). Any Creative Center scraper whose rows carry the TikTok CDN video link works — `doliz/tiktok-creative-center-scraper`, `datapeak/tiktok-creative-center`, `beyondops/tiktok-ad-library-scraper`, `lexis-solutions/tiktok-top-ads-scraper`, `dltik/tiktok-creative-center` and others: change the actor slug and JSON body in the *Scrape Creative Center Top Ads* node. The transcript step reads the run's dataset ID and deep-scans rows for the video link and material ID, whatever the field names.

### Freshness

Creative Center video links expire about **6 hours** after the scraper minted them — this flow transcribes immediately after the scrape. Expired rows come back as uncharged `unavailable_expired`; if the row carries the ad's material ID, the transcript actor fetches a fresh link first.

---

## LinkedIn Ad Library transcripts — one advertiser → hooks & transcripts

**What it does:** point it at one advertiser's [LinkedIn Ad Library](https://www.linkedin.com/ad-library/home) search URL (or a company URL) → scrapes their ads with [silva95gustavo/linkedin-ad-library-scraper](https://apify.com/silva95gustavo/linkedin-ad-library-scraper) → chains [steadyfetch/linkedin-ads-transcript-scraper](https://apify.com/steadyfetch/linkedin-ads-transcript-scraper) **by dataset ID** (no ad rows pass through n8n) → gives you a spreadsheet-shaped table: one row per ad with the **first-3-seconds hook**, headline, format, transcript length in words, language, seconds and the full transcript. Video ads are transcribed; image and text ads pass through uncharged with their metadata, sorted to the bottom.

**[Download the workflow JSON →](./linkedin-ad-transcripts.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `linkedin-ad-transcripts.workflow.json`.
2. Open the **Config** node and paste your [Apify API token](https://console.apify.com/settings/integrations) into `apifyToken`.
3. Set `adLibraryUrl` — an Ad Library search URL (`https://www.linkedin.com/ad-library/search?accountOwner=hubspot&countries=US&dateOption=last-30-days`) or a company URL (`https://www.linkedin.com/company/hubspot/`) — and `maxAds`.
4. Click **Test workflow**. The flow polls the scraper run until it finishes, then transcribes by dataset ID.

### What you get per ad

| field | example |
|---|---|
| `advertiser` | HubSpot |
| `headline` | "We are HubSpot Elite Partner" |
| `ad_id` | 1508450294 |
| `format` | SPONSORED_VIDEO |
| `status` | `transcribed` (only these are charged) — image/text ads show `non_video_skipped` |
| `hook_first_3s` | "If your revenue team is working harder than ever, but closing less," |
| `seconds` | 43.9 |
| `transcript_words` | 112 |
| `language` | English |
| `charged` | `true` / `false` — image ads, silent creatives and blocked pages carry no result fee |
| `transcript` | full speech-to-text, any video length |
| `detail_url` | https://www.linkedin.com/ad-library/detail/1508450294 |

The last node is a plain Set node (no credentials needed) — wire Google Sheets, Airtable, Slack, or a database after it.

### Cost

- Ad Library scraping: [silva95gustavo/linkedin-ad-library-scraper](https://apify.com/silva95gustavo/linkedin-ad-library-scraper) — $2.00–4.00 / 1,000 ads at the time of writing (check its Pricing tab)
- Transcription: [steadyfetch/linkedin-ads-transcript-scraper](https://apify.com/steadyfetch/linkedin-ads-transcript-scraper) — from $15.00 / 1,000 video ad transcripts, **charged only when a transcript is delivered**
- A 25-ad test run ≈ **$0.60 at most** (most LinkedIn ads are images, which are not charged). Apify's free $5 credit covers it many times over.

### Swap the scraper

The scrape step is a plain HTTP call to `silva95gustavo/linkedin-ad-library-scraper` (`{ "startUrls": [{ "url": ... }], "resultsLimit": N }`). Any Ad Library scraper works — `dz_omar/linkedin-ads-scraper`, `memo23/linkedin-ads-scraper`, `ivanvs/linkedin-ad-library-scraper`, `automation-lab/linkedin-ad-library-scraper` and others: change the actor slug and JSON body in the *Scrape the Ad Library (advertiser)* node. The transcript step reads the run's dataset ID and deep-scans rows for the LinkedIn video link or ad detail link, whatever the field names. You can also skip the scraper entirely — the transcript actor searches the Ad Library itself when given advertiser names or keywords.

---

## Google Ads video transcripts — one advertiser/domain → hooks & transcripts

**What it does:** point it at one advertiser or domain in the [Google Ads Transparency Center](https://adstransparency.google.com/) → scrapes their ads with [silva95gustavo/google-ads-scraper](https://apify.com/silva95gustavo/google-ads-scraper) → chains [steadyfetch/google-ads-video-transcript-scraper](https://apify.com/steadyfetch/google-ads-video-transcript-scraper) **by dataset ID** (no ad rows pass through n8n) → gives you a spreadsheet-shaped table: one row per video ad with the **first-3-seconds hook**, creative ID, YouTube video ID, transcript length in words, language, seconds and the full transcript. Transcribed ads first, longest first.

**[Download the workflow JSON →](./google-ads-video-transcripts.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `google-ads-video-transcripts.workflow.json`.
2. Open the **Config** node and paste your [Apify API token](https://console.apify.com/settings/integrations) into `apifyToken`.
3. Set `transparencyCenterUrl` — open the [Transparency Center](https://adstransparency.google.com/), pick an advertiser or type a domain, set region and date, and copy the address bar (e.g. `https://adstransparency.google.com/?region=US&domain=hellofresh.com&preset-date=Last+30+days`, or an `https://adstransparency.google.com/advertiser/AR…?region=US` URL) — and `maxAds`.
4. Click **Test workflow**. The flow polls the scraper run until it finishes, then transcribes by dataset ID.

### What you get per ad

| field | example |
|---|---|
| `advertiser` | HelloFresh SE |
| `creative_id` | CR17857019577632817153 |
| `video_id` | ONzIwHyQGJs |
| `status` | `transcribed` (only these are charged) |
| `hook_first_3s` | "What if your food was smarter?" |
| `seconds` | 15 |
| `transcript_words` | 34 |
| `language` | en |
| `charged` | `true` / `false` — removed videos, music-only ads and rows with no video carry no result fee |
| `transcript` | full speech-to-text, any video length |
| `video_url` | https://www.youtube.com/watch?v=ONzIwHyQGJs |

The last node is a plain Set node (no credentials needed) — wire Google Sheets, Airtable, Slack, or a database after it.

### Cost

- Transparency Center scraping: [silva95gustavo/google-ads-scraper](https://apify.com/silva95gustavo/google-ads-scraper) — $1.90 / 1,000 ads at the time of writing (check its Pricing tab)
- Transcription: [steadyfetch/google-ads-video-transcript-scraper](https://apify.com/steadyfetch/google-ads-video-transcript-scraper) — from $15.00 / 1,000 video ad transcripts, **charged only when a transcript is delivered**
- A 25-ad test run ≈ **$0.55 at most** (text and image ads are not charged). Apify's free $5 credit covers it many times over.

### Swap the scraper

The scrape step is a plain HTTP call to `silva95gustavo/google-ads-scraper` (`{ "startUrls": [{ "url": ... }], "resultsLimit": N }`). Any Transparency Center scraper whose rows carry the ad's YouTube link works — `lexis-solutions/google-ads-scraper`, `solidcode/ads-transparency-scraper` and others: change the actor slug and JSON body in the *Scrape the Transparency Center (advertiser)* node. The transcript step reads the run's dataset ID and deep-scans rows for the YouTube link or video ID, whatever the field names. You can also skip the scraper entirely — the transcript actor finds an advertiser's video ads itself when given advertiser names or domains.

Want the **image and text ads'** copy too? Pair it with [steadyfetch/google-ads-creative-text-scraper](https://apify.com/steadyfetch/google-ads-creative-text-scraper) — same advertiser input, extracts headlines, body copy and CTAs from the non-video creatives.

---

### Validated

All six workflows import cleanly into n8n (last checked against n8n 2.35.6 via `n8n import:workflow`).
