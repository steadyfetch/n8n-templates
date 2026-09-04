# steadyfetch n8n templates

Free n8n workflows for ad-creative, Instagram and keyword data pipelines. No community nodes required — plain HTTP, importable into any n8n (cloud or self-hosted).

**Library-ready (Sep 2026):** every workflow uses an n8n **Header Auth** credential for Apify (no token field), writes to **Google Sheets**, and opens with an overview sticky note plus section notes. Five of them are scheduled monitors: they loop over your targets, remember what they already reported with the native **Remove Duplicates** node, label each new row with an **AI Information Extractor** step (Groq by default, a drop-in swap for OpenAI or Anthropic), and send a digest to Slack, Gmail or Telegram only when there is something new.

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

## LinkedIn competitor ad monitor — weekly hooks, AI analysis & a Slack digest

**What it does:** every Monday it walks your list of competitor [LinkedIn Ad Library](https://www.linkedin.com/ad-library/home) pages, scrapes each one with [silva95gustavo/linkedin-ad-library-scraper](https://apify.com/silva95gustavo/linkedin-ad-library-scraper), chains [steadyfetch/linkedin-ads-transcript-scraper](https://apify.com/steadyfetch/linkedin-ads-transcript-scraper) **by dataset ID** (no ad rows pass through n8n), drops every ad it has already reported with the native **Remove Duplicates** node, then has an AI classify the new ones — hook type, offer, CTA type, target persona, angle and why it works. New rows go to Google Sheets and the five longest new hooks go to Slack. A quiet week writes nothing and sends nothing.

**Built for:** B2B demand-generation and ABM managers watching a fixed competitor set.

**[Download the workflow JSON →](./linkedin-ad-transcripts.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `linkedin-ad-transcripts.workflow.json`.
2. Create an **HTTP Header Auth** credential in n8n (name `Authorization`, value `Bearer YOUR_APIFY_TOKEN`, token from [Apify → Settings → Integrations](https://console.apify.com/settings/integrations)) and select it on the three HTTP Request nodes.
3. Add a **Groq** credential on the *Groq chat model* node — or drop in an OpenAI / Anthropic chat model node in its place.
4. Connect **Google Sheets** and **Slack**.
5. Open **Config**: paste one Ad Library URL per line into `competitors` (`https://www.linkedin.com/ad-library/search?accountOwner=hubspot&countries=US&dateOption=last-30-days`), set `maxAdsPerCompetitor` and `slackChannel`.
6. Click **Test workflow**, then leave the weekly Schedule Trigger on.

### What you get per ad

| field | example |
|---|---|
| `advertiser` · `ad_id` · `headline` | HubSpot · 1508450294 · "We are HubSpot Elite Partner" |
| `hook_type` *(AI)* | question / statistic / pain-point / story / bold-claim / other |
| `hook_first_3s` | "If your revenue team is working harder than ever, but closing less," |
| `offer` · `cta_type` *(AI)* | "free CRM audit" · demo |
| `target_persona` *(AI)* | RevOps leader |
| `angle_summary` · `why_it_works` *(AI)* | ≤20 words · ≤25 words |
| `seconds` · `transcript_words` · `language` | 43.9 · 112 · English |
| `status` | `transcribed` or `image_text_extracted` (the charged rows) — document, carousel, article and text-only ads show `non_video_skipped` |
| `charged` | `true` / `false` — text-free creatives, silent videos and blocked pages carry no result fee |
| `transcript` · `detail_url` | full speech-to-text · https://www.linkedin.com/ad-library/detail/1508450294 |

Rows are appended to **Google Sheets**; the digest goes to **Slack**. Swap either for Airtable, Teams or email.

### Cost

- Ad Library scraping: [silva95gustavo/linkedin-ad-library-scraper](https://apify.com/silva95gustavo/linkedin-ad-library-scraper) — $2.00–4.00 / 1,000 ads at the time of writing (check its Pricing tab)
- Transcription: [steadyfetch/linkedin-ads-transcript-scraper](https://apify.com/steadyfetch/linkedin-ads-transcript-scraper) — from $8.00 / 1,000 ad creative transcripts, **charged only when text is delivered**
- A 25-ad test run ≈ **$0.60 at most** (video and image ads both deliver; text-free creatives are not charged). Apify's free $5 credit covers it many times over.

### Swap the scraper

The scrape step is a plain HTTP call to `silva95gustavo/linkedin-ad-library-scraper` (`{ "startUrls": [{ "url": ... }], "resultsLimit": N }`). Any Ad Library scraper works — `dz_omar/linkedin-ads-scraper`, `memo23/linkedin-ads-scraper`, `ivanvs/linkedin-ad-library-scraper`, `automation-lab/linkedin-ad-library-scraper` and others: change the actor slug and JSON body in the *Scrape this competitor's Ad Library* node. The transcript step reads the run's dataset ID and deep-scans rows for the LinkedIn video link or ad detail link, whatever the field names. You can also skip the scraper entirely — the transcript actor searches the Ad Library itself when given advertiser names or keywords.

---

## Google video ad tracker — weekly script breakdowns by email

**What it does:** every Monday it walks your list of advertiser domains in the [Google Ads Transparency Center](https://adstransparency.google.com/), scrapes each one with [silva95gustavo/google-ads-scraper](https://apify.com/silva95gustavo/google-ads-scraper), chains [steadyfetch/google-ads-video-transcript-scraper](https://apify.com/steadyfetch/google-ads-video-transcript-scraper) **by dataset ID** (no ad rows pass through n8n), drops every creative it has already reported with the native **Remove Duplicates** node, then has an AI break the new scripts down — hook type, promise, proof element, CTA type and script structure. New rows go to Google Sheets and Gmail sends the week's breakdown, counting transcribed and silent ads separately.

**Built for:** YouTube and Google Ads performance marketers tracking a set of brands.

**[Download the workflow JSON →](./google-ads-video-transcripts.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `google-ads-video-transcripts.workflow.json`.
2. Create an **HTTP Header Auth** credential in n8n (name `Authorization`, value `Bearer YOUR_APIFY_TOKEN`, token from [Apify → Settings → Integrations](https://console.apify.com/settings/integrations)) and select it on the three HTTP Request nodes.
3. Add a **Groq** credential on the *Groq chat model* node — or drop in an OpenAI / Anthropic chat model node in its place.
4. Connect **Google Sheets** and **Gmail**.
5. Open **Config**: one advertiser domain per line in `advertiserDomains` (`hellofresh.com`), plus `region`, `maxAdsPerDomain` and `digestRecipient`.
6. Click **Test workflow**, then leave the weekly Schedule Trigger on.

### What you get per ad

| field | example |
|---|---|
| `advertiser` · `creative_id` | HelloFresh SE · CR17857019577632817153 |
| `hook_type` *(AI)* | question / statistic / pain-point / story / bold-claim / demo / other |
| `hook_first_3s` | "What if your food was smarter?" |
| `promise` *(AI)* | "dinner solved in 15 minutes" |
| `proof_element` *(AI)* | testimonial / demo / stat / none |
| `cta_type` · `script_structure` *(AI)* | shop now · problem-solution |
| `summary` *(AI)* | ≤25 words |
| `seconds` · `transcript_words` · `language` | 15 · 34 · en |
| `status` | `transcribed` (only these are charged) |
| `charged` | `true` / `false` — removed videos, music-only ads and rows with no video carry no result fee |
| `transcript` · `video_url` | full speech-to-text · https://www.youtube.com/watch?v=ONzIwHyQGJs |

Silent ads still reach the sheet with their status, so nothing goes missing. Rows are appended to **Google Sheets**; the digest goes out by **Gmail** — swap it for Slack or Teams.

### Cost

- Transparency Center scraping: [silva95gustavo/google-ads-scraper](https://apify.com/silva95gustavo/google-ads-scraper) — $1.90 / 1,000 ads at the time of writing (check its Pricing tab)
- Transcription: [steadyfetch/google-ads-video-transcript-scraper](https://apify.com/steadyfetch/google-ads-video-transcript-scraper) — from $8.00 / 1,000 video ad transcripts, **charged only when a transcript is delivered**
- A 25-ad test run ≈ **$0.55 at most** (text and image ads are not charged). Apify's free $5 credit covers it many times over.

### Swap the scraper

The scrape step is a plain HTTP call to `silva95gustavo/google-ads-scraper` (`{ "startUrls": [{ "url": ... }], "resultsLimit": N }`). Any Transparency Center scraper whose rows carry the ad's YouTube link works — `lexis-solutions/google-ads-scraper`, `solidcode/ads-transparency-scraper` and others: change the actor slug and JSON body in the *Scrape the Transparency Center* node. The transcript step reads the run's dataset ID and deep-scans rows for the YouTube link or video ID, whatever the field names. You can also skip the scraper entirely — the transcript actor finds an advertiser's video ads itself when given advertiser names or domains.

Want the **image and text ads'** copy too? Pair it with [steadyfetch/google-ads-creative-text-scraper](https://apify.com/steadyfetch/google-ads-creative-text-scraper) — same advertiser input, extracts headlines, body copy and CTAs from the non-video creatives.

---


---

## Reels swipe file — weekly hooks, AI rewrites & a Telegram digest

**What it does:** every Monday it loops over the creators you follow, runs [steadyfetch/instagram-reel-transcript-scraper](https://apify.com/steadyfetch/instagram-reel-transcript-scraper) on each handle (the actor lists the newest reels itself — no scraper to chain, no login), drops every reel it has already filed with the native **Remove Duplicates** node, then has an AI label the hook and **rewrite it for your own brand** in 20 words. New rows go to Google Sheets and the five most-liked new hooks go to Telegram. A quiet week writes nothing and sends nothing.

**Built for:** UGC creators and social media managers keeping a swipe file they can film from.

**[Download the workflow JSON →](./instagram-reel-transcripts.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `instagram-reel-transcripts.workflow.json`.
2. Create an **HTTP Header Auth** credential (name `Authorization`, value `Bearer YOUR_APIFY_TOKEN` — token from [Apify → Settings → Integrations](https://console.apify.com/settings/integrations)) and select it on the HTTP Request node.
3. Add a **Groq** credential on the *Groq chat model* node — or drop in an OpenAI / Anthropic chat model node in its place.
4. Connect **Google Sheets** and **Telegram**.
5. Open **Config**: one handle per line in `creatorHandles` (`natgeo`, `@natgeo` or a profile URL), plus `reelsPerProfile`, `yourBrand` (one line the AI rewrites hooks for) and `telegramChatId`. Optional: `includeOnScreenText` → `true` to read text off silent reels (those become charged rows).
6. Click **Test workflow**, then leave the weekly Schedule Trigger on.

### What you get per reel

| field | example |
|---|---|
| `creator` · `reel_url` · `posted_at` | natgeo · https://www.instagram.com/reel/… · 2026-09-04T17:31Z |
| `hook_type` *(AI)* | question / statistic / pain-point / story / bold-claim / listicle / other |
| `content_pillar` *(AI)* | education / entertainment / behind-the-scenes / promo / story / other |
| `emotional_trigger` *(AI)* | curiosity |
| `cta_present` *(AI)* | `true` / `false` |
| `rewrite_for_my_brand` *(AI)* | your hook, rewritten in ≤20 words |
| `likes` · `plays` · `seconds` · `transcript_words` | 18,985 · 299,322 · 69.6 · 146 |
| `language` · `status` · `caption` | English · `transcribed` (only these are charged) · the post caption |
| `charged` | `true` / `false` — silent reels and dead links are **$0** |
| `transcript` | full speech-to-text, any length |

### Cost

[steadyfetch/instagram-reel-transcript-scraper](https://apify.com/steadyfetch/instagram-reel-transcript-scraper) — from $5.00 / 1,000 reels on Apify's Business plan ($15.00 on the free plan), plus $0.005 per minute beyond the first 3 minutes of a long video. A 10-reel test ≈ **$0.15**.

---

## Keyword opportunity scorer — a form in, scored and clustered keywords out

**What it does:** submit a keyword list through the built-in **n8n Form Trigger** (or run it manually with the demo list) → [steadyfetch/keyword-search-volume-scraper](https://apify.com/steadyfetch/keyword-search-volume-scraper) returns real Google Ads Keyword Planner figures through a licensed provider — **average monthly searches**, competition, top-of-page bid range, 12-month trend and **CPC wherever Google publishes one** — then an AI labels each keyword's **search intent**, a topic **cluster** and the **page type** worth building. A Code node scores every keyword `volume × (1 − competition/100) ÷ CPC`, sorts by it, and **Remove Duplicates** skips anything already scored for that country in an earlier run. New rows go to Google Sheets; the ten best opportunities go to Slack.

No modelled numbers, no invented difficulty score; keywords Google has no data for come back uncharged. Switch `mode` to `ideas` to expand each keyword into new keyword ideas with the same metrics.

**Built for:** SEO and SEM specialists who want the list triaged before they read it.

**[Download the workflow JSON →](./keyword-search-volume.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `keyword-search-volume.workflow.json`.
2. Create an **HTTP Header Auth** credential (name `Authorization`, value `Bearer YOUR_APIFY_TOKEN` — token from [Apify → Settings → Integrations](https://console.apify.com/settings/integrations)) and select it on the HTTP Request node.
3. Add a **Groq** credential on the *Groq chat model* node — or drop in an OpenAI / Anthropic chat model node in its place.
4. Connect **Google Sheets** and **Slack**.
5. Open the **On form submission** node and copy its form URL — that is the plug-and-play front door (keywords, country, language). The **Config** node holds the demo list, `mode` and `slackChannel` for manual runs.
6. Click **Test workflow**, or open the form and paste a list.

### What you get per keyword

| field | notes |
|---|---|
| `rank` · `opportunity_score` | position in the sorted list and the score behind it |
| `intent` *(AI)* | informational / commercial / transactional / navigational |
| `cluster` *(AI)* | a two or three word topic label |
| `suggested_page_type` *(AI)* | blog / comparison / product / landing / tool |
| `avg_monthly_searches` | Google's average; `0` means fewer than Google reports |
| `cpc_usd` | average CPC, `null` where Google publishes none |
| `competition` · `competition_index` · `low_top_of_page_bid_usd` · `high_top_of_page_bid_usd` | LOW / MEDIUM / HIGH, 0–100, and the bid range |
| `trend` · `data_as_of` | `rising` / `flat` / `falling` and the last month Google reported |
| `charged` | `true` / `false` — no-data keywords are **$0** |

The score favours high volume, low competition and cheap clicks. Reweight it in the *Score the opportunity* Code node.

### Cost

[steadyfetch/keyword-search-volume-scraper](https://apify.com/steadyfetch/keyword-search-volume-scraper) — from $2.00 / 1,000 keywords on Apify's Business plan ($8.00 on the free plan), no minimum batch. From 16 September 2026 a run that buys fresh data also pays one $0.19 fresh-lookup fee (runs answered from the actor's 30-day cache pay none). 50 fresh keywords ≈ **$0.59** on the free plan ($0.40 before that date); set the run's Maximum cost per run to at least $0.25.

---

## Instagram competitor watch — daily posts, a viral flag & a Slack alert

**What it does:** every morning it loops over the competitor accounts you name, runs [steadyfetch/instagram-profile-posts](https://apify.com/steadyfetch/instagram-profile-posts) on each (no login, no cookies; your limit is exact, 30 means 30), drops every post it has already reported with the native **Remove Duplicates** node, then works out which of the new ones **broke out**: engagement (likes + comments) is compared with the median for that same account, so a big account and a small one are judged fairly. An AI reads each caption for theme, tone, format and CTA. Every new post is appended to Google Sheets — but Slack is pinged **only** when something went viral.

**Built for:** social media managers who want to hear about the breakouts, not about every post.

**[Download the workflow JSON →](./instagram-profile-posts.workflow.json)**

### 3-minute setup

1. In n8n: **Workflows → Import from File** → pick `instagram-profile-posts.workflow.json`.
2. Create an **HTTP Header Auth** credential (name `Authorization`, value `Bearer YOUR_APIFY_TOKEN` — token from [Apify → Settings → Integrations](https://console.apify.com/settings/integrations)) and select it on the HTTP Request node.
3. Add a **Groq** credential on the *Groq chat model* node — or drop in an OpenAI / Anthropic chat model node in its place.
4. Connect **Google Sheets** and **Slack**.
5. Open **Config**: one handle per line in `competitorHandles`, plus `postsPerProfile`, `viralMultiplier` (2 = twice that account's median) and `slackChannel`.
6. Click **Test workflow**, then leave the daily Schedule Trigger on.

### What you get per post

| field | example |
|---|---|
| `account` · `posted_by` · `post_url` | natgeo · natgeo · https://www.instagram.com/p/DchLnq8E21N/ |
| `posted_at` · `type` | 2026-08-26T21:30:18Z · carousel |
| `theme` · `tone` *(AI)* | wildlife conservation · factual |
| `cta_present` · `format_guess` *(AI)* | `false` · carousel |
| `summary` *(AI)* | ≤20 words |
| `likes` · `comments` · `plays` | 19,990 · 140 · — |
| `engagement` · `account_median_engagement` · `is_viral` | 20,130 · 8,400 · `true` |
| `caption` | "How it started 👉 how it's going…" |
| `charged` | `true` / `false` — posts we could not deliver are **$0** |

Raise `viralMultiplier` to 3 for a quieter alert.

### Cost

[steadyfetch/instagram-profile-posts](https://apify.com/steadyfetch/instagram-profile-posts) — from $0.60 / 1,000 posts on Apify's Business plan ($2.40 on the free plan). Two profiles × 30 posts ≈ **$0.15** on the free plan. From 16 September 2026 each profile that delivers posts adds one $0.003 profile lookup (two profiles ≈ $0.006 more); a profile that delivers nothing pays none.


---

### Validated

All eight workflows are gated on the latest n8n in Docker — **2.37.7** — not just parsed. Every sticky note is measured on a rendered canvas at 100% zoom (clipped text is silent: the sticky wrapper is `overflow: hidden`), and every workflow is executed end to end against the live Apify API from the editor's own run endpoint, with one HTTP Header Auth credential and the Google Sheets / Slack / Gmail / Telegram nodes disabled.

The five scheduled monitors are run **twice**: pass 1 appends the rows and sends the digest, pass 2 finds nothing new — the native Remove Duplicates node drops every row it already reported and the workflow ends on its "nothing new" branch. That is the whole point of a scheduled template, so it is proven rather than assumed.

Every workflow uses an HTTP Header Auth credential for the Apify token (no token field anywhere) and writes to a Google Sheets append node.
