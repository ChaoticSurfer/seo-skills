# SEO Tools Catalogue (2026)

Reference for what's available, what each tool gives you, and when to reach for it. Three tiers based on access cost — not capability.

> **Stack recommendation at the bottom.** Read sections in order; the bottom synthesizes.

---

## Tier 1 — Truly free (no registration, no API key)

Run these immediately, every audit. Zero friction.

### Crawling & extraction

| Tool | What it gives you | How |
|---|---|---|
| **`curl`** | Raw HTML, status codes, redirect chains, response headers, timings | `curl -sIL -A "Mozilla/5.0"` |
| **`curl --resolve` / Googlebot UA** | Bot-vs-browser parity check, soft-404 detection, slow-for-bots issues | `-A "Googlebot/2.1"` |
| **`xmllint`** | Sitemap parsing + XML validation | `xmllint --noout sitemap.xml` |
| **Plain shell (grep/awk/sort)** | Per-URL meta extraction, H1 counts, canonical checks across hundreds of URLs in seconds | scripts in `examples/` |

### Performance (lab)

| Tool | What it gives you | How | Cost |
|---|---|---|---|
| **Lighthouse CLI** | Lab CWV (LCP/INP/CLS/TBT) + a11y + best-practices + SEO score per URL | `npx lighthouse <url> --form-factor=mobile` | Free, local |
| **WebPageTest free** | Best lab CWV testing in the industry; full waterfalls | webpagetest.org/test (no key) | ~200 tests/day shared quota |
| **GTmetrix free** | Lighthouse + extra waterfall views | gtmetrix.com (no key) | ~5 tests/day |

### Validation

| Tool | What it gives you | How |
|---|---|---|
| **W3C HTML validator (Nu)** | HTML errors, duplicate IDs, invalid nesting | `curl -X POST -H "Content-Type: text/html" --data-binary @page.html https://validator.w3.org/nu/?out=json` |
| **Schema.org validator** | JSON-LD structural validation | https://validator.schema.org (web only) |
| **Google Rich Results Test** | Schema rich-result eligibility check | https://search.google.com/test/rich-results (web only) |
| **structured-data-testing-tool** (CLI) | Programmatic schema validation in CI | `npx structured-data-testing-tool <url>` |
| **HTTPS / SSL Labs** | TLS configuration + cipher audit | https://www.ssllabs.com/ssltest/ |

### Link / accessibility

| Tool | What it gives you | How |
|---|---|---|
| **lychee** (Rust) | Broken-link checker, fast, runs in CI | `lychee https://site.com` |
| **linkinator** (Node) | Same, easier on macOS | `npx linkinator <url>` |
| **axe-core CLI** | Accessibility audit (WCAG 2.1) | `npx @axe-core/cli <url>` |

### AI / search

| Tool | What it gives you | Cost |
|---|---|---|
| **Google Trends** | Relative keyword interest over time | Free, no signup |
| **Bing IndexNow** | Push instant indexing notifications to Bing | Free, no auth — anonymous POST |
| **Perplexity / ChatGPT** (manual) | AI citation test for target queries | Free tier sufficient for spot checks |

### Crawl coverage (limited but free)

| Tool | What it gives you | Limit |
|---|---|---|
| **Screaming Frog SEO Spider** (free tier) | Full site crawl, broken links, duplicates, redirects, schema, hreflang | 500 URLs |
| **Sitebulb Lite** (free trial) | Visual site crawl audit | Trial period |

### Testing in browser

| Tool | What it gives you |
|---|---|
| **Chrome DevTools** | CWV measurement live, source attribution, render-blocking detection |
| **PageSpeed Insights web UI** | Lab Lighthouse + CrUX field data combined per URL (NOT API — see Tier 2) |

### MCP servers (truly free, no key)

None at this tier yet — Tier 1 tools are CLI-based.

---

## Tier 2 — Free tier, needs registration + API key

5–25 minutes of setup each. Massive coverage upgrade. **Verify domain ownership for 1 and 2.**

### 🟢 The "must verify" four (do all four — total ~25 min setup, $0/mo)

| # | Tool | What it adds | Setup |
|---|------|---|---|
| 1 | **Google Search Console** | Real rankings, clicks, impressions, coverage errors, mobile usability, real-user CWV from your owned domains. Single most-valuable free signal. | DNS or HTML-file domain verification (5 min). [search.google.com/search-console](https://search.google.com/search-console) |
| 2 | **Bing Webmaster Tools** | Bing rankings, IndexNow integration, free keyword research tool, impression data | Domain verification (5 min). [bing.com/webmasters](https://www.bing.com/webmasters) |
| 3 | **Ahrefs Webmaster Tools (AWT)** | Backlink profile + keyword data **for your owned sites only** — what Ahrefs paid users get, free for owned domains | Domain verification (5 min). [ahrefs.com/webmaster-tools](https://ahrefs.com/webmaster-tools) |
| 4 | **CrUX API key** | Real-user (p75) LCP/INP/CLS for **any public URL** (your own + competitors') | Free Google Cloud key (3 min). [developers.google.com/web/tools/chrome-user-experience-report](https://developers.google.com/web/tools/chrome-user-experience-report/api) |

### Other Tier-2 free APIs

| Tool | What it adds | Limit | Setup |
|---|---|---|---|
| **PageSpeed Insights API** | Lighthouse + CrUX bundled per URL | 25k queries/day | Free Google API key (3 min) |
| **Google Analytics 4** | Site traffic, conversions, attribution | Free up to 10M events/mo | Tag installation |
| **Bing Webmaster API** | Programmatic submissions, indexing status | Generous | Token from BWT account |
| **DebugBear free tier** | Continuous CWV monitoring | 1 site | Free signup |
| **Lighthouse CI server** (self-hosted) | Continuous Lighthouse history | Unlimited (self-hosted) | Docker / npm install |
| **Treo free tier** | Continuous CWV monitoring | 10 URLs | Free signup |
| **Cloudflare Radar** | Industry-trend data, search trends | Free | None |
| **Bing IndexNow with key** | Higher rate-limit IndexNow submissions | Free tier generous | Free, simple TXT-file token |

### Continuous monitoring (free tier)

| Tool | Free tier | What you give up vs paid |
|---|---|---|
| **Visualping** | 5 monitors | Detection cadence, premium triggers |
| **Hexowatch** | 3 monitors | Cadence, AI features |
| **Crawly** | Limited | Fewer projects, fewer alerts |

### MCP servers (Tier 2 — registration required)

| MCP | Wraps | Status (early 2026) |
|---|---|---|
| **`gsc-mcp`** community wrappers | Google Search Console | Multiple on GitHub. Quality varies. Auth via OAuth. |
| **`pagespeed-mcp`** community | PageSpeed Insights API | Simple wrapper, easy to write yourself |
| **`bing-mcp`** community | Bing Webmaster API | Less common |
| **`crux-mcp`** community | CrUX API | Tiny wrapper around the API |

When connected, these expose tools like `mcp__gsc__top_queries`, `mcp__crux__page_metrics`, etc. — same opt-in pattern as the Semrush MCP integration.

---

## Tier 3 — Paid (with free trial / limited free tier)

Skip on a small budget. Useful for agencies, larger sites, or when competitor data is critical.

### Keyword + competitor research

| Tool | Pricing | Best for | MCP? | Free tier |
|---|---|---|---|---|
| **Semrush Pro** | ~$140/mo | Comprehensive — keywords + backlinks + site audit + position tracking | ✅ Official MCP | 10 searches/day |
| **Ahrefs Standard** | ~$229/mo | Industry-leading backlink index | ❌ No official, community wrappers exist | Webmaster Tools is free for owned domains (see Tier 2) |
| **Moz Pro** | ~$99/mo | Simpler UX, less data depth | ❌ | 10 queries/mo |
| **DataForSEO** | Pay-as-you-go (~$0.0001/req) | API-first, cheaper than Semrush per-call | ✅ Community MCP | None |
| **Sistrix** | ~$129/mo | German market, visibility index | ❌ | None |
| **SEranking** | ~$55/mo | Cheapest of the comprehensives | ❌ | 14-day trial |
| **Conductor / BrightEdge / Botify** | Enterprise (talk to sales) | Large agencies, e-commerce | Some | None |

### Performance / RUM

| Tool | Pricing | Best for | Free tier |
|---|---|---|---|
| **DebugBear** | $24/mo+ | Continuous CWV, Lighthouse history | 1 site free |
| **SpeedCurve** | $29/mo+ | Marketing perf metrics | None |
| **Calibre** | $25/mo+ | Lighthouse trends + budgets | 14-day trial |
| **Treo** | $29/mo+ | Continuous CWV | Free 10 URLs |

### Crawl / audit

| Tool | Pricing | Best for | Free |
|---|---|---|---|
| **Screaming Frog (paid)** | £199/yr | Unlimited crawls + JS rendering | 500 URLs free |
| **Sitebulb** | $35/mo | Visualized site auditor | Trial only |
| **Lumar (formerly Deepcrawl)** | Enterprise | Big-site crawl, integrations | None |
| **Oncrawl** | $69/mo+ | Log-file analysis, big sites | None |

### Position tracking

| Tool | Pricing | Notes |
|---|---|---|
| **AccuRanker** | $129/mo+ | Daily-rank refresh, no other features | None free |
| **Nightwatch** | $39/mo+ | Cheapest serious tracker | 14-day trial |
| **SerpRobot** | $9/mo+ | Budget option | 25 keywords free |
| **GSC** | Free (Tier 2) | Already covers your own positions — usually enough |

### Generative / AI search optimization

| Tool | Pricing | Best for | Free |
|---|---|---|---|
| **Profound** | Enterprise | AI-search rank tracking (ChatGPT/Perplexity/Gemini) | None |
| **Otterly.AI** | $19/mo+ | AI mention tracking | 14-day trial |
| **Peec.AI** | $49/mo+ | LLM citation tracking | Trial |

---

## Stack recommendations

### 🪙 Solo / personal site (~£0/mo)

Just **Tier 1** + the four "must verify" Tier-2 tools:

```
Tier 1:  Lighthouse, curl, lychee, W3C validator, axe-core, Schema validator,
         Rich Results Test, Screaming Frog (free 500 URLs)
Tier 2:  GSC, Bing WMT, Ahrefs Webmaster Tools, CrUX API key
```

Coverage: ~85% of what you'd get from a Semrush Pro subscription. Limit: no
competitor keyword data, no historical tracking beyond what GSC stores.

### 💼 Small agency / startup (~£35/mo)

Add **Screaming Frog full** (£199/yr ≈ £17/mo) + one **continuous monitoring**
tool (Treo free tier or DebugBear $24/mo).

```
Adds:   Unlimited site crawls, JS rendering, custom extraction
        Continuous CWV monitoring with alerting
```

Still no Semrush — keyword research via GSC + Ahrefs Webmaster Tools.

### 🏢 Mid-market (~£200/mo)

Add **Semrush Pro** ($140/mo). Now you have competitor data + position tracking
+ keyword volume + content gap.

```
Adds:   Competitor backlink + keyword analysis
        Position tracking for non-owned keyword sets
        Content gap analysis
        Keyword Magic Tool for content briefs
        Site Audit at scale
```

Optional: connect the **Semrush MCP** to this plugin (see
`semrush-mcp.md`) — automates competitor evidence into audit findings.

### 🏛️ Enterprise (£500+/mo)

Add Conductor/BrightEdge/Lumar for log-file analysis, content optimization at
scale, governance. Out of scope for this plugin.

---

## When to skip a tool

- **Skip Moz Pro** — Semrush Pro is more comprehensive at similar price.
- **Skip standalone position trackers** if you have GSC + Semrush.
- **Skip AMP-related tools** — AMP is effectively dead since 2021.
- **Skip "AI SEO content writers"** — they hurt more than help (HCS demotes thin AI content).
- **Skip "100 directory submission" tools** — Google Penguin made these harmful.

---

## Tool-to-finding mapping

When you need evidence for a finding, here's where to look:

| Finding | Where evidence comes from |
|---|---|
| Page returns 4xx/5xx | curl + `xargs` (Tier 1) |
| LCP / INP / CLS lab | Lighthouse CLI (Tier 1) |
| LCP / INP / CLS real-user | CrUX API (Tier 2) — best — or PSI |
| Title / desc / H1 issues | curl + grep (Tier 1) |
| Broken internal links | linkinator / lychee (Tier 1) |
| Schema validity | structured-data-testing-tool (Tier 1) + Rich Results Test |
| HTML errors | W3C Nu validator (Tier 1) |
| Crawled pages count | Screaming Frog free (Tier 1, ≤500 URLs) |
| Keyword rankings (your own) | GSC (Tier 2) |
| Keyword rankings (competitor) | Semrush / Ahrefs (Tier 3) |
| Backlinks (your own) | AWT (Tier 2) |
| Backlinks (competitor) | Ahrefs / Semrush (Tier 3) |
| Search volume (rough) | Bing WMT keyword research (Tier 2) or Ubersuggest free |
| Search volume (precise) | Semrush / Ahrefs (Tier 3) |
| Position tracking history | GSC (Tier 2) — last 16 months |
| Content gap | Semrush / Ahrefs (Tier 3) |
| AI Overview presence | Manual SERP testing (Tier 1) or Otterly.AI / Profound (Tier 3) |
| Continuous CWV monitoring | DebugBear / Treo / SpeedCurve (Tier 2 free tier or Tier 3) |
| Log file analysis | Oncrawl / Lumar (Tier 3) — or roll your own with `goaccess` |

---

## API keys cheat sheet (copy-paste setup)

```bash
# CrUX API
export CRUX_API_KEY="..."  # https://console.cloud.google.com/apis/credentials

# PageSpeed Insights (same Google project as CrUX)
export PSI_API_KEY="$CRUX_API_KEY"

# Bing Webmaster Tools
export BING_WMT_API_KEY="..."  # https://www.bing.com/webmasters/about

# IndexNow (Bing-compatible, anonymous-or-token)
export INDEXNOW_KEY="$(uuidgen | tr -d -)"
# Then host the key at:  https://yoursite.com/$INDEXNOW_KEY.txt
# Submit:
curl -X POST "https://api.indexnow.org/indexnow" -H "Content-Type: application/json" -d '{
  "host": "yoursite.com",
  "key": "'$INDEXNOW_KEY'",
  "urlList": ["https://yoursite.com/changed-page"]
}'

# GSC — needs OAuth2, easiest via gcloud:
gcloud auth application-default login
# then python: from googleapiclient.discovery import build
#              service = build('searchconsole', 'v1')

# Ahrefs Webmaster Tools — token from AWT settings page
export AHREFS_WMT_TOKEN="..."

# Semrush API (Tier 3)
export SEMRUSH_API_KEY="..."  # MCP picks this up automatically
```

Add these to `~/.zshrc` or a `.envrc` (with `direnv`) so they're available every audit.
