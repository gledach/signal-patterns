# What is watchable

A catalogue of public sources a signal watcher can read, ranked by how early they move and
what they cost. Researched September 2026; sources listed at the bottom.

**The organising idea:** a company is only as visible as what it exposes publicly, and the
useful signals are the ones it exposes *before* it means to. Ranked by lead time, the
order is roughly: infrastructure → hiring → documentation → changelog → marketing → press.
Most tools start at the press end, which is the last place anything appears.

---

## Tier 1 — leading indicators (weeks to months ahead)

| Source | Lead time | Cost | Precision | Notes |
|---|---|---|---|---|
| **Certificate Transparency logs** | **20–30 days** ahead of hiring, longer ahead of launch | free | medium | Every publicly trusted CA must log to an append-only, auditable log under RFC 6962, so a follower sees *every* certificate issued anywhere within minutes. New subdomains reveal regional rollouts, new products and staging environments before anything is announced. |
| **Job postings** | months | free (scrape) / paid (API) | low-medium | The earliest signal and the least precise. ML hires suggest an AI feature; a burst in a new geography suggests expansion. Precision is low because hiring often reflects last quarter's plan. |
| **DNS / hosting / ASN changes** | weeks | free | low | Infra migrations leak ops shifts. Best as *enrichment* on a CT hit rather than a source of its own. |
| **Beta programmes, early-access forums, community Slack/Discord** | weeks | free–hard | high when it fires | Where features surface before launch. Hard to collect and easy to get wrong ethically — see the caution below. |

**The CT pipeline worth copying:** raw cert metadata → domain extraction → DNS/ASN/hosting
enrichment → vendor detection → classify and score. A CT entry alone proves only that
issuance was logged; validate ownership, DNS, hosting and page behaviour before drawing a
conclusion from it.

## Tier 2 — confirmation, fastest and most reliable (hours to days ahead)

| Source | Lead time | Cost | Precision | Notes |
|---|---|---|---|---|
| **Changelog / "what's new" pages** | hours–days | free | **high** | The single best confirmation signal. Updated *before* announcements reach social or email. One vendor's sample: a change detected on **73%** of changelog monitors within 90 days. |
| **API docs / integration guides / help articles** | hours–days | free | high | Technical documentation for a feature is usually published before the marketing post. New endpoints are new product surface. |
| **Pricing pages** | at launch | free | high | 59% of monitors saw a change within 90 days. Packaging changes are strategy, not copy. |
| **Product & feature pages** | at launch | free | high | 58%. A new section reliably means a launch is imminent or just happened. |
| **Careers pages** (as a diff, not a feed) | weeks | free | medium | 65% changed within 90 days. |
| **Sitemap + `robots.txt` diffs** | days | free | medium | Cheap proxy for all of the above — new paths appear before they are linked. `robots.txt` disallow rules leak unreleased sections by name. |
| **Status pages** | live | free | high | Outages route to support, not product — but a competitor's incident history is a durable quality argument. |

## Tier 3 — narrative and corroboration

| Source | Cost | Notes |
|---|---|---|
| **News / RSS / Google News** | free | High volume, low precision. Most useful as *corroboration* — several independent publishers on one event is a convergence; ten mentions of one press release is one event. |
| **Hacker News (Algolia API)** | free, no auth | Structured: points, comments, author, timestamps. Better than the RSS equivalent. |
| **Reddit** | free API | Where dissatisfaction lives. Search RSS and per-subreddit feeds are easy; thread bodies are where the value is and are harder. |
| **Review sites** (G2, Capterra, Trustpilot) | free to read, scraping-heavy | Strong SMB sentiment signal. Gated enough to be genuinely hard. |
| **GitHub releases / activity** | free API | Exact, timestamped, and for dev-tools competitors often *is* the launch. |
| **YouTube transcripts** | free | Founders say things on camera they would not write. Retain an excerpt and a link, not the whole work. |
| **Search trends** | free | `"<competitor> alternative"` is churn intent. Category volume is demand. |
| **SEC EDGAR** (10-Q, 10-K, 8-K) | free | Only useful when the competitor or its customers are public. Material events, legally required. |
| **arXiv** | free | For research-led competitors, affiliation changes catch researchers months before a job board does — at $0 instead of a paid hiring API. |
| **Answer-engine visibility** | LLM cost | "Which brands does a model name for this prompt" is a new category of share-of-voice. Moves on model-release timescales, so weekly at most. |

## Tier 4 — possible, and mostly not worth it

App-store review mining, package-registry download trends, podcast ad-buy monitoring,
Wayback diffing, prediction markets. Each is real; each has a worse ratio of effort to
signal than anything in Tier 2. Build these last.

---

## What to refuse

Worth naming explicitly, because these come up in every brainstorm and they are the line
between competitive analysis and surveillance:

- Satellite imagery of offices, EXIF on staff photos, reverse image search on employees
- Lurking in private communities under a false identity
- Anything requiring a fake account, a bypassed paywall, or a broken Terms of Service
- Call recording without checking single-party-consent jurisdiction first

A tool that collects these is not a better tool. It is a liability with a dashboard.

## Three practical cautions

**No single source wins.** Pricing, hiring, regulatory, news, financial and product signals
all move at different speeds. A layered approach — page diffs + jobs + news — beats any one
vendor, which is the argument for building rather than buying.

**Visibility is not uniform.** A competitor with no public job board, changelog or status
page produces few signals. Absence of signal is not absence of activity, which is why a
coverage block matters more than it sounds.

**Manual monitoring does not scale.** Checking five competitors daily is a full-time job.
That is the whole reason this is a cron and not a habit.

---

## Sources

- [Free Competitive Intelligence Tools (2026) — Visualping](https://visualping.io/blog/top-free-competitive-intelligence-tools) — change-detection rates per page type
- [Detect Competitor Feature Launches Before Your Customers Ask — CAM](https://www.getcam.io/blog/detect-competitor-feature-launches-before-customers-ask-about-them/) — changelog and docs as leading signals
- [Certificate Transparency for Brand Monitoring — isMalicious](https://ismalicious.com/posts/certificate-transparency-phishing-brand-monitoring) — RFC 6962, log-follower mechanics
- [Certificate Transparency Monitoring — Cloudflare](https://developers.cloudflare.com/ssl/edge-certificates/additional-options/certificate-transparency-monitoring/)
- [Forelight](https://forelight.net/) — infrastructure as a leading indicator of hiring
- [PredictLeads](https://predictleads.com/) — job-posting data at scale
- [Company Signals — Apify](https://apify.com/dev_web_col/company-signals) — hiring and multi-signal actors
- [Competitive Intelligence with Signal Data — Autobound](https://www.autobound.ai/blog/competitive-intelligence-signal-data)
