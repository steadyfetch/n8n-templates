# steadyfetch n8n templates

Free n8n workflows for ad-creative, Instagram and keyword data pipelines. No community nodes required — plain HTTP, importable into any n8n (cloud or self-hosted).

**Library-ready (Sep 2026):** the five ad-transcript workflows use an n8n **Header Auth** credential for Apify (no token field), end in a **Google Sheets** node, and open with an overview sticky note plus section notes. They are being published to n8n's template library one at a time.

## Facebook Ad Transcripts — scrape competitor ads → hooks, CTAs & transcripts

**What it does:** searches the Facebook Ad Library for video ads → transcribes every creative → gives you one row per ad with the **first-3-seconds hook**, CTA, advertiser, and full transcript. The manual version of this pipeline is sold as a paid template elsewhere; this one is free.

**[Download the workflow JSON →](./facebook-ad-transcripts.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `facebook-ad-transcripts.workflow.json`.
2. Create an **HTTP Header Auth** credential in n8n (name `Authorization`, value `Bearer YOUR_APIFY_TOKEN`, token from [Apify → Settings → Integrations](https://console.apify.com/settings/integrations)) and select it on the HTTP Request nodes.
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

The last node appends one row per item to **Google Sheets** (connect your Google account and pick a sheet), or swap it for Airtable, Slack, or a database.

### Cost

- Ad scraping: [curious_coder/facebook-ads-library-scraper](https://apify.com/curious_coder/facebook-ads-library-scraper) — $0.75 / 1,000 ads
- Transcription: [steadyfetch/facebook-ads-transcript-scraper](https://apify.com/steadyfetch/facebook-ads-transcript-scraper) — from $10.00 / 1,000 ad creative transcripts, **charged only when a transcript is delivered**
- A 20-ad test run ≈ **$0.42**. Apify's free $5 credit covers it many times over.

### Swap-friendly

The transcription step accepts **any** ad-library scraper's output (it deep-scans rows for the video URL + metadata) — if you already use a different scraper, just point its results at the *Transcribe Ads* node body.

---

## Competitor ad teardown — one competitor page/keyword → hooks, CTAs & transcripts, one row per ad

**What it does:** point it at one competitor (their Facebook Page URL or an Ad Library search URL) → scrapes their active video ads → chains the transcript actor **by dataset ID** (no ad rows pass through n8n, so big batches stay light) → gives you a spreadsheet-shaped teardown: one row per ad with the **first-3-seconds hook**, CTA, transcript length in words, language, seconds, and the full transcript. Transcribed ads first, longest first.

**[Download the workflow JSON →](./competitor-ad-teardown.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `competitor-ad-teardown.workflow.json`.
2. Create an **HTTP Header Auth** credential in n8n (name `Authorization`, value `Bearer YOUR_APIFY_TOKEN`, token from [Apify → Settings → Integrations](https://console.apify.com/settings/integrations)) and select it on the HTTP Request nodes.
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

The last node appends one row per item to **Google Sheets** (connect your Google account and pick a sheet), or swap it for Airtable, Slack, or a database.

### Swap the scraper

The scrape step is a plain HTTP call to `curious_coder/facebook-ads-library-scraper` (`{ "urls": [{ "url": ... }], "count": N }`). Any other Ad Library scraper works too: change the actor slug and JSON body in the *Start Ad Library scrape* node — the transcript step reads the run's dataset ID and deep-scans rows for the video URL, whatever the field names.

---

## TikTok Top Ads transcripts — Creative Center industry/region → hooks & transcripts

**What it does:** pick a Creative Center market, lookback window and industry → scrapes the **Top Ads** with [azzouzana/tiktok-creative-center-top-ads-scraper](https://apify.com/azzouzana/tiktok-creative-center-top-ads-scraper) → chains [steadyfetch/tiktok-ads-transcript-scraper](https://apify.com/steadyfetch/tiktok-ads-transcript-scraper) **by dataset ID** right away (no ad rows pass through n8n, and TikTok's ~6-hour video links are still fresh) → gives you a spreadsheet-shaped table: one row per ad with the **first-3-seconds hook**, CTR, likes, brand, ad title, transcript length in words, language, seconds and the full transcript. Transcribed ads first, highest CTR first.

**[Download the workflow JSON →](./tiktok-ad-transcripts.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `tiktok-ad-transcripts.workflow.json`.
2. Create an **HTTP Header Auth** credential in n8n (name `Authorization`, value `Bearer YOUR_APIFY_TOKEN`, token from [Apify → Settings → Integrations](https://console.apify.com/settings/integrations)) and select it on the HTTP Request nodes.
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

The last node appends one row per item to **Google Sheets** (connect your Google account and pick a sheet), or swap it for Airtable, Slack, or a database. Expect a fair share of uncharged `no_audio` rows — many Top Ads are music plus on-screen text.

### Cost

- Top Ads scraping: [azzouzana/tiktok-creative-center-top-ads-scraper](https://apify.com/azzouzana/tiktok-creative-center-top-ads-scraper) — $1.00 / 1,000 ads at the time of writing (check its Pricing tab)
- Transcription: [steadyfetch/tiktok-ads-transcript-scraper](https://apify.com/steadyfetch/tiktok-ads-transcript-scraper) — from $8.00 / 1,000 ad video transcripts, **charged only when a transcript is delivered**
- A 25-ad test run ≈ **$0.53 at most** (less when some ads are music-only). Apify's free $5 credit covers it many times over.

### Swap the scraper

The scrape step is a plain HTTP call to `azzouzana/tiktok-creative-center-top-ads-scraper` (`{ "countryCode", "period", "industry", "maxItems" }`). Any Creative Center scraper whose rows carry the TikTok CDN video link works — `doliz/tiktok-creative-center-scraper`, `datapeak/tiktok-creative-center`, `beyondops/tiktok-ad-library-scraper`, `lexis-solutions/tiktok-top-ads-scraper`, `dltik/tiktok-creative-center` and others: change the actor slug and JSON body in the *Scrape Creative Center Top Ads* node. The transcript step reads the run's dataset ID and deep-scans rows for the video link and material ID, whatever the field names.

### Freshness

Creative Center video links expire about **6 hours** after the scraper minted them — this flow transcribes immediately after the scrape. Expired rows come back as uncharged `unavailable_expired`; if the row carries the ad's material ID, the transcript actor fetches a fresh link first.

---

## LinkedIn Ad Library transcripts — one advertiser → hooks & transcripts

**What it does:** point it at one advertiser's [LinkedIn Ad Library](https://www.linkedin.com/ad-library/home) search URL (or a company URL) → scrapes their ads with [silva95gustavo/linkedin-ad-library-scraper](https://apify.com/silva95gustavo/linkedin-ad-library-scraper) → chains [steadyfetch/linkedin-ads-transcript-scraper](https://apify.com/steadyfetch/linkedin-ads-transcript-scraper) **by dataset ID** (no ad rows pass through n8n) → gives you a spreadsheet-shaped table: one row per ad with the **first-3-seconds hook**, headline, format, transcript length in words, language, seconds and the full transcript. Video ads are transcribed and image ads have their on-image copy read (both delivered rows, both charged); document, carousel, article and text-only creatives pass through uncharged with their metadata, sorted to the bottom.

**[Download the workflow JSON →](./linkedin-ad-transcripts.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `linkedin-ad-transcripts.workflow.json`.
2. Create an **HTTP Header Auth** credential in n8n (name `Authorization`, value `Bearer YOUR_APIFY_TOKEN`, token from [Apify → Settings → Integrations](https://console.apify.com/settings/integrations)) and select it on the HTTP Request nodes.
3. Set `adLibraryUrl` — an Ad Library search URL (`https://www.linkedin.com/ad-library/search?accountOwner=hubspot&countries=US&dateOption=last-30-days`) or a company URL (`https://www.linkedin.com/company/hubspot/`) — and `maxAds`.
4. Click **Test workflow**. The flow polls the scraper run until it finishes, then transcribes by dataset ID.

### What you get per ad

| field | example |
|---|---|
| `advertiser` | HubSpot |
| `headline` | "We are HubSpot Elite Partner" |
| `ad_id` | 1508450294 |
| `format` | SPONSORED_VIDEO |
| `status` | `transcribed` or `image_text_extracted` (these are the charged rows) — document, carousel, article and text-only ads show `non_video_skipped` |
| `hook_first_3s` | "If your revenue team is working harder than ever, but closing less," |
| `seconds` | 43.9 |
| `transcript_words` | 112 |
| `language` | English |
| `charged` | `true` / `false` — text-free creatives, silent videos and blocked pages carry no result fee |
| `transcript` | full speech-to-text, any video length |
| `detail_url` | https://www.linkedin.com/ad-library/detail/1508450294 |

The last node appends one row per item to **Google Sheets** (connect your Google account and pick a sheet), or swap it for Airtable, Slack, or a database.

### Cost

- Ad Library scraping: [silva95gustavo/linkedin-ad-library-scraper](https://apify.com/silva95gustavo/linkedin-ad-library-scraper) — $2.00–4.00 / 1,000 ads at the time of writing (check its Pricing tab)
- Transcription: [steadyfetch/linkedin-ads-transcript-scraper](https://apify.com/steadyfetch/linkedin-ads-transcript-scraper) — from $8.00 / 1,000 ad creative transcripts, **charged only when text is delivered**
- A 25-ad test run ≈ **$0.60 at most** (video and image ads both deliver; text-free creatives are not charged). Apify's free $5 credit covers it many times over.

### Swap the scraper

The scrape step is a plain HTTP call to `silva95gustavo/linkedin-ad-library-scraper` (`{ "startUrls": [{ "url": ... }], "resultsLimit": N }`). Any Ad Library scraper works — `dz_omar/linkedin-ads-scraper`, `memo23/linkedin-ads-scraper`, `ivanvs/linkedin-ad-library-scraper`, `automation-lab/linkedin-ad-library-scraper` and others: change the actor slug and JSON body in the *Scrape the Ad Library (advertiser)* node. The transcript step reads the run's dataset ID and deep-scans rows for the LinkedIn video link or ad detail link, whatever the field names. You can also skip the scraper entirely — the transcript actor searches the Ad Library itself when given advertiser names or keywords.

---

## Google Ads video transcripts — one advertiser/domain → hooks & transcripts

**What it does:** point it at one advertiser or domain in the [Google Ads Transparency Center](https://adstransparency.google.com/) → scrapes their ads with [silva95gustavo/google-ads-scraper](https://apify.com/silva95gustavo/google-ads-scraper) → chains [steadyfetch/google-ads-video-transcript-scraper](https://apify.com/steadyfetch/google-ads-video-transcript-scraper) **by dataset ID** (no ad rows pass through n8n) → gives you a spreadsheet-shaped table: one row per video ad with the **first-3-seconds hook**, creative ID, YouTube video ID, transcript length in words, language, seconds and the full transcript. Transcribed ads first, longest first.

**[Download the workflow JSON →](./google-ads-video-transcripts.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `google-ads-video-transcripts.workflow.json`.
2. Create an **HTTP Header Auth** credential in n8n (name `Authorization`, value `Bearer YOUR_APIFY_TOKEN`, token from [Apify → Settings → Integrations](https://console.apify.com/settings/integrations)) and select it on the HTTP Request nodes.
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

The last node appends one row per item to **Google Sheets** (connect your Google account and pick a sheet), or swap it for Airtable, Slack, or a database.

### Cost

- Transparency Center scraping: [silva95gustavo/google-ads-scraper](https://apify.com/silva95gustavo/google-ads-scraper) — $1.90 / 1,000 ads at the time of writing (check its Pricing tab)
- Transcription: [steadyfetch/google-ads-video-transcript-scraper](https://apify.com/steadyfetch/google-ads-video-transcript-scraper) — from $8.00 / 1,000 video ad transcripts, **charged only when a transcript is delivered**
- A 25-ad test run ≈ **$0.55 at most** (text and image ads are not charged). Apify's free $5 credit covers it many times over.

### Swap the scraper

The scrape step is a plain HTTP call to `silva95gustavo/google-ads-scraper` (`{ "startUrls": [{ "url": ... }], "resultsLimit": N }`). Any Transparency Center scraper whose rows carry the ad's YouTube link works — `lexis-solutions/google-ads-scraper`, `solidcode/ads-transparency-scraper` and others: change the actor slug and JSON body in the *Scrape the Transparency Center (advertiser)* node. The transcript step reads the run's dataset ID and deep-scans rows for the YouTube link or video ID, whatever the field names. You can also skip the scraper entirely — the transcript actor finds an advertiser's video ads itself when given advertiser names or domains.

Want the **image and text ads'** copy too? Pair it with [steadyfetch/google-ads-creative-text-scraper](https://apify.com/steadyfetch/google-ads-creative-text-scraper) — same advertiser input, extracts headlines, body copy and CTAs from the non-video creatives.

---


---

## Transcribe Instagram Reels from a creator's profile to Google Sheets with Apify

**What it does:** give it an Instagram handle → the actor lists that creator's most recent reels itself and transcribes them → one row per reel with the **first-3-seconds hook**, the full spoken transcript, language, duration, caption, plays and likes. No scraper to chain, no login. Reels with no speech are not charged unless you switch on on-screen text reading.

**[Download the workflow JSON →](./instagram-reel-transcripts.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `instagram-reel-transcripts.workflow.json`.
2. Create an **HTTP Header Auth** credential (name `Authorization`, value `Bearer YOUR_APIFY_TOKEN` — token from [Apify → Settings → Integrations](https://console.apify.com/settings/integrations)) and select it on the HTTP Request node.
3. Connect **Google Sheets** on the last node and pick the spreadsheet and sheet.
4. Open the **Config** node: set `instagramHandle` (`nasa`, `@nasa` or a profile URL) and `maxReels`. Optional: `includeOnScreenText` → `true` to read text off silent reels (those become charged rows).
5. Click **Test workflow**.

### What you get per reel

| field | example |
|---|---|
| `status` | `transcribed` (only these are charged) |
| `creator` | nasa |
| `hook_first_3s` | "I think we got my mom the perfect Mother's Day gift this year" |
| `transcript` | full speech-to-text, any length |
| `language` · `seconds` · `caption` | English · 61.1 · the post caption |
| `charged` | `true` / `false` — silent reels and dead links are **$0** |

### Cost

[steadyfetch/instagram-reel-transcript-scraper](https://apify.com/steadyfetch/instagram-reel-transcript-scraper) — from $5.00 / 1,000 reels on Apify's Business plan ($15.00 on the free plan), plus $0.005 per minute beyond the first 3 minutes of a long video. A 10-reel test ≈ **$0.15**.

---

## Get Google search volume and CPC for a keyword list to Google Sheets with Apify

**What it does:** paste keywords one per line → one row per keyword with Google Ads Keyword Planner figures through a licensed provider: **average monthly searches**, competition, top-of-page bid range, 12-month trend direction, and **CPC wherever Google publishes one**. No modelled numbers, no invented difficulty score; keywords Google has no data for come back uncharged. Switch `mode` to `ideas` to expand each keyword into new keyword ideas with the same metrics.

**[Download the workflow JSON →](./keyword-search-volume.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `keyword-search-volume.workflow.json`.
2. Create an **HTTP Header Auth** credential (name `Authorization`, value `Bearer YOUR_APIFY_TOKEN` — token from [Apify → Settings → Integrations](https://console.apify.com/settings/integrations)) and select it on the HTTP Request node.
3. Connect **Google Sheets** on the last node and pick the spreadsheet and sheet.
4. Open the **Config** node: paste your list into `keywordsText` (one per line or comma-separated); set `country` (`US`, `GB`, `DE`…) and `language` (`en`, `de`…).
5. Click **Test workflow**. The IF node drops the run's summary row so only keyword rows reach your sheet.

### What you get per keyword

| field | notes |
|---|---|
| `avg_monthly_searches` | Google's average; `0` means fewer than Google reports |
| `cpc_usd` | average CPC, `null` where Google publishes none |
| `competition` · `low_top_of_page_bid_usd` · `high_top_of_page_bid_usd` | LOW / MEDIUM / HIGH and the bid range |
| `trend` · `data_as_of` | `rising` / `flat` / `falling` and the last month Google reported |
| `charged` | `true` / `false` — no-data keywords are **$0** |

### Cost

[steadyfetch/keyword-search-volume-scraper](https://apify.com/steadyfetch/keyword-search-volume-scraper) — from $2.00 / 1,000 keywords on Apify's Business plan ($8.00 on the free plan), no minimum batch. 50 keywords ≈ **$0.40** on the free plan.

---

## Pull the latest posts and reels from Instagram profiles to Google Sheets with Apify

**What it does:** handles or profile URLs in → one row per post or reel, **newest by date** (pinned posts are flagged, not promoted to the top), with caption, hashtags, like and comment counts, plays and duration for videos, and the media URL. No login, no cookies. Your limit is exact: 30 means 30.

**[Download the workflow JSON →](./instagram-profile-posts.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `instagram-profile-posts.workflow.json`.
2. Create an **HTTP Header Auth** credential (name `Authorization`, value `Bearer YOUR_APIFY_TOKEN` — token from [Apify → Settings → Integrations](https://console.apify.com/settings/integrations)) and select it on the HTTP Request node.
3. Connect **Google Sheets** on the last node and pick the spreadsheet and sheet.
4. Open the **Config** node: set `profiles` (comma-separated handles or URLs), `postsPerProfile`, and optionally `mediaType` (`any`, `image`, `video`, `carousel`).
5. Click **Test workflow**. The IF node keeps post rows only (profile and summary rows are dropped).

### What you get per post

| field | example |
|---|---|
| `profile` · `post_url` · `type` | nasa · https://www.instagram.com/p/DchLnq8E21N/ · carousel |
| `posted_at` · `caption` | 2026-08-26T21:30:18Z · "How it started 👉 how it's going…" |
| `likes` · `comments` · `plays` · `video_seconds` | 19,990 · 140 · — · — |
| `is_pinned` · `media_url` | false · the image or video URL |
| `charged` | `true` / `false` — posts we could not deliver are **$0** |

### Cost

[steadyfetch/instagram-profile-posts](https://apify.com/steadyfetch/instagram-profile-posts) — from $0.60 / 1,000 posts on Apify's Business plan ($2.40 on the free plan). Two profiles × 30 posts ≈ **$0.15** on the free plan.


---

### Validated

All eight workflows import cleanly into n8n via `n8n import:workflow` (the five ad-transcript workflows checked against n8n 2.36.9 on 2026-09-01; the three Instagram/keyword workflows re-checked after their library pass on 2026-09-02, see the commit for the exact image tag). Every workflow uses an HTTP Header Auth credential for the Apify token (no token field in the Config node) and ends in a Google Sheets append node.
