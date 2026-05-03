# SEO Platforms — Registration & Setup Reference

Every external platform / service the SEO skills can use. Grouped by access tier and purpose. **All require some form of registration unless marked otherwise.**

> Tier definitions
> - **🟢 Free + verification**: free tier, requires owning the domain you audit
> - **🟢 Free + signup**: free tier, just create an account
> - **🟡 Free trial**: limited time or limited features without payment
> - **🔴 Paid**: subscription or pay-per-use
> - **⚫ Enterprise**: sales contact required, no public pricing

---

## 1. Google ecosystem (free)

| # | Platform | Tier | URL | Verifies | Unlocks |
|---|----------|------|-----|----------|---------|
| 1 | **Google Search Console** | 🟢 Free + verification | https://search.google.com/search-console | DNS TXT or HTML file | Real rankings, clicks, impressions, coverage errors, mobile usability, real-user CWV |
| 2 | **Google Analytics 4** | 🟢 Free + signup | https://analytics.google.com | Tag installation | Traffic, conversions, attribution, user paths (free up to 10M events/mo) |
| 3 | **Google Cloud Console** | 🟢 Free + signup | https://console.cloud.google.com | Account only | API keys for CrUX / PageSpeed / Search Console / Indexing APIs |
| 4 | **CrUX API** | 🟢 Free + API key | (via #3) | API key | Real-user p75 LCP/INP/CLS for any public URL |
| 5 | **PageSpeed Insights API** | 🟢 Free + API key | (via #3) | API key | Lighthouse + CrUX bundled, 25k req/day |
| 6 | **Search Console API** | 🟢 Free + API key | (via #3) | OAuth | Programmatic GSC access |
| 7 | **Indexing API** | 🟢 Free + API key | (via #3) | API key | Push indexing (only for JobPosting + BroadcastEvent schema types) |
| 8 | **Google Tag Manager** | 🟢 Free + signup | https://tagmanager.google.com | None | Tag deployment, also auto-verifies GSC + GA4 |
| 9 | **Google Trends** | 🟢 No signup | https://trends.google.com | None | Relative keyword interest over time |
| 10 | **Google Rich Results Test** | 🟢 No signup | https://search.google.com/test/rich-results | None | Schema rich-result eligibility (web only) |

---

## 2. Bing / Microsoft ecosystem (free)

| # | Platform | Tier | URL | Verifies | Unlocks |
|---|----------|------|-----|----------|---------|
| 11 | **Bing Webmaster Tools** | 🟢 Free + verification | https://www.bing.com/webmasters | Import from GSC OR DNS / HTML file | Bing rankings, free keyword research, IndexNow integration |
| 12 | **Bing Webmaster API** | 🟢 Free + API key | (via #11) | Token from BWT | Programmatic submissions, indexing status |
| 13 | **IndexNow** | 🟢 No signup, just key file | https://www.indexnow.org | UUID key file at site root | Push indexing to Bing + Yandex + Seznam (NOT Google) |

---

## 3. Backlink + keyword data (free with verification)

| # | Platform | Tier | URL | Verifies | Unlocks |
|---|----------|------|-----|----------|---------|
| 14 | **Ahrefs Webmaster Tools (AWT)** | 🟢 Free + verification | https://ahrefs.com/webmaster-tools | DNS / HTML file | Backlink profile + keyword data for owned domains (paid-Ahrefs equivalent, owned-only) |

---

## 4. Performance / CWV monitoring (free tier)

| # | Platform | Tier | URL | Limits | Unlocks |
|---|----------|------|-----|--------|---------|
| 15 | **WebPageTest** | 🟢 Free + signup | https://www.webpagetest.org | ~200 tests/day shared quota | Best lab CWV testing in industry, full waterfalls |
| 16 | **GTmetrix** | 🟢 Free + signup | https://gtmetrix.com | 5 tests/day, history retained | Lighthouse + extra waterfall views |
| 17 | **DebugBear** | 🟢 Free tier | https://www.debugbear.com | 1 site | Continuous Lighthouse + RUM with alerts |
| 18 | **Treo** | 🟢 Free tier | https://treo.sh | 10 URLs | Continuous CWV monitoring |
| 19 | **Lighthouse CI server** | 🟢 Self-hosted free | https://github.com/GoogleChrome/lighthouse-ci | Unlimited | Continuous Lighthouse history |
| 20 | **Calibre** | 🟡 Trial | https://calibreapp.com | 14-day trial → $25/mo+ | Lighthouse trends + perf budgets |
| 21 | **SpeedCurve** | 🔴 Paid | https://www.speedcurve.com | $29/mo+ | Marketing perf metrics |

---

## 5. Crawl & site audit (free tier or local)

| # | Platform | Tier | URL | Limits |
|---|----------|------|-----|--------|
| 22 | **Screaming Frog SEO Spider** | 🟢 Free up to 500 URLs | https://www.screamingfrog.co.uk/seo-spider | License email signup; local desktop app |
| 23 | **Sitebulb Lite** | 🟡 Trial | https://sitebulb.com | 14-day trial |
| 24 | **Lumar** (formerly Deepcrawl) | ⚫ Enterprise | https://www.lumar.io | Sales |
| 25 | **Oncrawl** | 🔴 Paid | https://www.oncrawl.com | $69/mo+ |

---

## 6. Keyword research (free tier)

| # | Platform | Tier | URL | Limits |
|---|----------|------|-----|--------|
| 26 | **Ubersuggest** | 🟢 Free + signup | https://neilpatel.com/ubersuggest | 3 searches/day free |
| 27 | **AnswerThePublic** | 🟢 Free + signup | https://answerthepublic.com | 1 search/day free |
| 28 | **Moz Free** | 🟢 Free + signup | https://moz.com | 10 queries/mo |
| 29 | **Keyword Surfer** (Chrome ext) | 🟢 Free + install | https://chromewebstore.google.com (search "Keyword Surfer") | Inline volume in Google SERP |
| 30 | **Keywords Everywhere** | 🟡 Tiny credit pack | https://keywordseverywhere.com | $1.25/10k credits |

---

## 7. Comprehensive SEO suites (paid)

| # | Platform | Tier | URL | MCP? |
|---|----------|------|-----|------|
| 31 | **Semrush Pro** | 🔴 $140+/mo, 10 free searches/day | https://www.semrush.com | ✅ Official MCP |
| 32 | **Ahrefs Standard** | 🔴 $229+/mo (AWT free for owned) | https://ahrefs.com | ❌ Community wrappers only |
| 33 | **Moz Pro** | 🔴 $99+/mo | https://moz.com/products/pro | ❌ |
| 34 | **DataForSEO** | 🔴 Pay-as-you-go (~$0.0001/req) | https://dataforseo.com | ✅ Community MCP |
| 35 | **Sistrix** | 🔴 €129+/mo | https://www.sistrix.com | ❌ |
| 36 | **SE Ranking** | 🔴 $55+/mo, 14-day trial | https://seranking.com | ❌ |
| 37 | **SEranking SEO API** | 🔴 Per-request | (via #36) | ❌ |

---

## 8. Position tracking (paid; GSC covers your own free)

| # | Platform | Tier | URL | Notes |
|---|----------|------|-----|-------|
| 38 | **AccuRanker** | 🔴 $129+/mo, 14-day trial | https://www.accuranker.com | Daily-rank refresh |
| 39 | **Nightwatch** | 🔴 $39+/mo, 14-day trial | https://nightwatch.io | Cheaper serious tracker |
| 40 | **SerpRobot** | 🔴 $9+/mo, 25 keywords free | https://www.serprobot.com | Budget option |

---

## 9. AI / generative search optimization

| # | Platform | Tier | URL | Notes |
|---|----------|------|-----|-------|
| 41 | **Otterly.AI** | 🟡 Trial → $19+/mo | https://otterly.ai | AI mention tracking |
| 42 | **Peec.AI** | 🟡 Trial → $49+/mo | https://peec.ai | LLM citation tracking |
| 43 | **Profound** | ⚫ Enterprise | https://www.tryprofound.com | AI-search rank tracking |
| 44 | **Cloudflare Radar** | 🟢 Free + signup | https://radar.cloudflare.com | Industry trend + search trend data |

---

## 10. Validation tools (free, no signup)

| # | Platform | Tier | URL |
|---|----------|------|-----|
| 45 | **Schema.org validator** | 🟢 No signup | https://validator.schema.org |
| 46 | **W3C HTML Nu validator** | 🟢 No signup | https://validator.w3.org/nu/ |
| 47 | **SSL Labs** | 🟢 No signup | https://www.ssllabs.com/ssltest/ |
| 48 | **Mobile-Friendly Test** | 🟢 No signup | https://search.google.com/test/mobile-friendly |
| 49 | **AMP Test** (deprecated AMP) | 🟢 No signup | https://search.google.com/test/amp |

---

## 11. Change detection / competitor monitoring

| # | Platform | Tier | URL | Limits |
|---|----------|------|-----|--------|
| 50 | **Visualping** | 🟢 Free tier | https://visualping.io | 5 monitors |
| 51 | **Hexowatch** | 🟢 Free tier | https://hexowatch.com | 3 monitors |
| 52 | **Crawly** | 🟢 Free tier | https://crawly.diffbot.com | Limited |

---

## 12. Infrastructure platforms (often relevant for SEO)

| # | Platform | Tier | URL | SEO relevance |
|---|----------|------|-----|---------------|
| 53 | **Cloudflare** | 🟢 Free + signup | https://www.cloudflare.com | DNS, CDN, page rules, security headers, Workers (rewrites/redirects) |
| 54 | **Netlify** | 🟢 Free tier | https://www.netlify.com | Edge, headers config, redirects, sitemap auto-handling |
| 55 | **Vercel** | 🟢 Free tier | https://vercel.com | Same as Netlify; better Next.js integration |
| 56 | **GitHub Actions** | 🟢 Free for public repos | https://github.com | Lighthouse CI, schema validation, broken link checks in CI |

---

## 13. Enterprise SEO platforms (sales-required)

| # | Platform | Tier | URL |
|---|----------|------|-----|
| 57 | **Conductor** | ⚫ Enterprise | https://www.conductor.com |
| 58 | **BrightEdge** | ⚫ Enterprise | https://www.brightedge.com |
| 59 | **Botify** | ⚫ Enterprise | https://www.botify.com |
| 60 | **seoClarity** | ⚫ Enterprise | https://www.seoclarity.net |
| 61 | **Searchmetrics** | ⚫ Enterprise | https://www.searchmetrics.com |

---

## 📊 Verification methods cheat sheet

| Method | Where you do it | Time |
|---|---|---|
| **DNS TXT record** | Domain registrar (Cloudflare, Namecheap, etc.) | 5-10 min (DNS propagation) |
| **HTML file upload** | Server / repo public dir | 2 min |
| **HTML meta tag** | Site `<head>` | 2 min |
| **Google Tag Manager** | Already installed = instant verify (GSC, GA4) | 1 min |
| **Email magic link** | Inbox | 1 min |
| **OAuth handshake** | Browser flow | 1 min |

---

## 🟢 Recommended priority order (Edumentors-style small/mid site)

| Order | Platform | Why first | Setup time |
|---|---|---|---|
| 1 | **Google Search Console** (#1) | Most valuable single signup; all other Google tools chain off this | 5–10 min |
| 2 | **Bing Webmaster Tools** (#11) | Auto-imports from GSC | 2 min |
| 3 | **Ahrefs Webmaster Tools** (#14) | Free competitor-level data for owned site | 5 min |
| 4 | **Google Cloud key for CrUX + PSI** (#3-5) | Real-user data in audits | 3 min |
| 5 | **IndexNow** (#13) | Push instant Bing indexing | 15 min |
| 6 | **GA4** (#2) | If not already (probably is via GTM) | 5–10 min |
| 7 | **Lighthouse CI in GitHub Actions** (#56) | PR-level perf gate | 30 min |
| 8 | **DebugBear / Treo free tier** (#17 or #18) | Continuous CWV alerts | 5 min |
| 9 | **Screaming Frog free** (#22) | Full sitewide crawl | 10 min install |

**Total setup time**: ~90 minutes. **Total cost**: £0/month. **Coverage**: ~85% of paid Semrush feature set.

---

## 🔴 Skip on a small budget

- Moz Pro (Semrush is more comprehensive at similar price)
- Standalone position trackers (GSC covers your own positions)
- Enterprise tools (sales required, not relevant at this scale)
- AMP-related tools (AMP effectively dead since 2021)
- "AI SEO content writers" (HCS demotes thin AI content)
- "100 directory submission" services (Penguin made these harmful)
- Otterly / Peec / Profound (AI-search tracking) until you've got Tier 1+2 covered

---

## API key environment cheat sheet

```bash
# Google APIs (CrUX, PSI, Search Console, Indexing) — same project key
export GOOGLE_API_KEY="..."

# Or specific:
export CRUX_API_KEY="$GOOGLE_API_KEY"
export PSI_API_KEY="$GOOGLE_API_KEY"

# Search Console — OAuth (use gcloud)
gcloud auth application-default login

# Bing Webmaster Tools
export BING_WMT_API_KEY="..."

# IndexNow (Bing-compatible)
export INDEXNOW_KEY="$(uuidgen | tr -d -)"
# host at: https://yoursite.com/$INDEXNOW_KEY.txt

# Ahrefs Webmaster Tools
export AHREFS_WMT_TOKEN="..."

# Semrush (Tier 3, optional)
export SEMRUSH_API_KEY="..."  # The Semrush MCP picks this up automatically

# DataForSEO (Tier 3, alternative to Semrush)
export DATAFORSEO_LOGIN="..."
export DATAFORSEO_PASSWORD="..."
```

Add these to `~/.zshrc` or `.envrc` (with `direnv`) so they're available across audits.

---

## Cross-reference

- For **what each tool gives you in audit findings**, see `tools.md`
- For **the optional Semrush MCP audit module**, see `semrush-mcp.md`
- For **the audit workflow**, see the parent `SKILL.md` (`seo-nextjs/` or `seo-vue-astro/`)
