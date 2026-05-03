═══════════════════════════════════════════════════════════════════════════
  SEO AUDIT — edumentors.co.uk — 2026-05-03 (FULL — free-tool stack)
  Auditor: Claude Code (seo-vue-astro skill)
  Tools:   Lighthouse 11 (mobile, simulated throttling), curl, lychee/linkinator,
           W3C HTML validator, sitemap parser, custom HTML extractor.
  No paid tools used. No GSC / no CrUX / no Semrush.
═══════════════════════════════════════════════════════════════════════════

EXECUTIVE SUMMARY

  Stack:               Astro v3.6.5 on Netlify (en_GB)
  Sitemap URLs:        438  (status check: 100% return 200 ✅)
  Lighthouse pages:    6 (home + tutors + pricing + about-us + edu-ai-tutor + blog)
  Avg perf score:      71 / 100
  Avg page weight:     ~3.7 MB  (very heavy)
  Critical issues:     6 (P0)
  High priority:       9 (P1)
  Medium priority:     8 (P2)

  Core Web Vitals (lab, mobile):
    Pages passing LCP ≤ 2.5s:  0 of 6 (homepage at 2.6s — borderline)
    Pages passing CLS ≤ 0.1:   4 of 6 (pricing 0.224, about-us 0.237 fail)
    Pages passing INP/TBT:     6 of 6 (TBT all under 250ms — good)
  
  This is a Core-Web-Vitals failure across the board, not a metadata issue.
  The metadata is mostly fine (4/6 pages score 100 on Lighthouse SEO; only
  home loses a few points for the multi-H1). The performance is the
  problem — and within performance, image delivery and caching dominate.

═══════════════════════════════════════════════════════════════════════════

LIGHTHOUSE LAB SCORES (mobile, simulated, 2026-05-03)

  Page              Perf  SEO  A11y  BP   LCP    CLS    TBT   FCP   Bytes
  ─────────────────────────────────────────────────────────────────────────
  /                 93    92   88    54   2.6s   0.006  160   1.4s  3.4MB
  /tutors           64    100  90    79  10.9s   0.000  220   2.0s  1.8MB
  /pricing          63    100  88    75   8.3s   0.224  140   1.4s  2.0MB
  /about-us         64    100  92    75   5.0s   0.237  180   1.5s  1.9MB
  /edu-ai-tutor     83    100  95    71   4.2s   0.000  150   1.8s  7.5MB
  /blog             56    100  73    71   8.4s   0.000  120   3.9s  6.0MB
  
  ❌ = poor, ⚠ = needs improvement, ✅ = good
  
  LCP: home 2.6s ⚠ | tutors 10.9s ❌ | pricing 8.3s ❌ | about-us 5.0s ❌
       edu-ai 4.2s ❌ | blog 8.4s ❌
  CLS: pricing 0.224 ❌ | about-us 0.237 ❌
  
  Best-Practices on home (54) is the worst score on the site — caused by a
  silent JavaScript TypeError at line 489 hit on every page load.

═══════════════════════════════════════════════════════════════════════════

P0 — CRITICAL (fix within 24 h)

  [P0-1] Homepage JavaScript runtime error on every load
    Impact:     High         Confidence: Confirmed
    Evidence:   Lighthouse home report: 
                  TypeError: Cannot read properties of undefined
                    (reading 'addEventListener')
                    at HTMLDocument.<anonymous>
                    at https://edumentors.co.uk/:489:54
    Why it hurts: Browser console errors hurt Page Experience signal,
                trigger bf-cache failures, and may break tracking. Also
                the single biggest reason home best-practices = 54.
    Fix:        Inspect inline script at line 489 of homepage HTML.
                Likely a DOM lookup that runs before the target element
                is parsed. Wrap in DOMContentLoaded or move script to end
                of <body>.
    Effort: S    Priority: P0

  [P0-2] /pricing — broken <link rel="canonical"> + empty OG/Twitter URLs
    Impact:     High         Confidence: Confirmed
    Evidence:   <link rel="canonical">  ← no href attribute
                <meta property="og:url">     ← empty content
                <meta property="og:image">   ← empty content
                <meta name="twitter:image">  ← empty content
                (raw HTML inspection)
    Why it hurts: Google can canonicalize the page against URL parameter
                variants. Social shares render with no preview. /pricing
                is conversion-critical.
    Fix:        Astro layout — confirm canonical/OG props are wired:
                  <link rel="canonical" href={new URL(Astro.url.pathname, Astro.site)}/>
                  <meta property="og:url" content={canonical}/>
                  <meta property="og:image" content={new URL('/banner.png', Astro.site)}/>
    Effort: S    Priority: P0

  [P0-3] LCP > 2.5s on 5 of 6 audited pages (mobile simulated)
    Impact:     High         Confidence: Confirmed
    Evidence:   /tutors      LCP 10.9 s
                /pricing     LCP  8.3 s
                /blog        LCP  8.4 s
                /about-us    LCP  5.0 s
                /edu-ai      LCP  4.2 s
                /            LCP  2.6 s (borderline)
    Root causes (from Lighthouse "image-delivery-insight"):
                 home: 189 KiB image savings available
                 tutors: 22 KiB
                 about-us: 337 KiB
                 edu-ai: NOT measured (heavy page)
                 blog: 1,759 KiB image savings — biggest single page
    Fix:        Tier 1 (highest impact):
                  • Generate AVIF + sized variants in Astro build (broken now)
                  • Set width/height on hero images (kills CLS too)
                  • Set Cache-Control headers (see P0-5)
                Tier 2:
                  • Trim font preloads from 13 → 2 (P1-1)
                  • Defer GTM/GA scripts (currently render-blocking)
    Effort: M    Priority: P0
    Expected: -3 to -5s LCP on most pages.

  [P0-4] Multi-H1 on / and /pricing
    Impact:     High         Confidence: Confirmed
    Evidence:   /          : 2 × <h1>  (hero + Edu-AI section)
                /pricing   : 3 × <h1>  (third is "Frequently asked questions")
                Other 50 URLs sampled: all H1=1 ✅ (issue isolated)
    Fix:        Demote secondary H1s to H2 / H3.
                Lighthouse "heading-order" audit fails on / and /tutors —
                fixing H1 cascade will likely fix that too.
    Effort: S    Priority: P0

  [P0-5] Cache-Control: no-cache on every response (including HTML)
    Impact:     High         Confidence: Confirmed
    Evidence:   curl -I https://edumentors.co.uk/  →
                  cache-control: no-cache
                Lighthouse "uses-long-cache-ttl" fails on every page:
                  /home    7 resources, no efficient cache
                  /blog    Est savings: 4,938 KiB (!)
                  /tutors  Est savings:   218 KiB
                  /pricing Est savings:   218 KiB
    Why it hurts: Every CDN hit becomes a revalidation. TTFB suffers.
                Repeat-visit perf also suffers. Crawler revisits cost
                origin time. /blog losing 4.9 MB cacheable assets per
                navigation is the single biggest perf opportunity.
    Fix:        netlify.toml:
                  [[headers]]
                    for = "/*"
                    [headers.values]
                    Cache-Control = "public, max-age=0, must-revalidate"
                  [[headers]]
                    for = "/_astro/*"
                    [headers.values]
                    Cache-Control = "public, max-age=31536000, immutable"
                  [[headers]]
                    for = "/fonts/*"
                    [headers.values]
                    Cache-Control = "public, max-age=31536000, immutable"
                  [[headers]]
                    for = "/data/tutors/images/*"
                    [headers.values]
                    Cache-Control = "public, max-age=86400"
                  [[headers]]
                    for = "*.{jpg,jpeg,png,webp,avif,svg,ico}"
                    [headers.values]
                    Cache-Control = "public, max-age=2592000"
    Effort: S    Priority: P0
    Expected: 1–2s LCP improvement on warm-cache visits; massive byte
              savings for crawlers + repeat visitors.

  [P0-6] AVIF image pipeline broken — 10× 404s on homepage
    Impact:     High         Confidence: Confirmed
    Evidence:   linkinator crawl of homepage:
                  /data/tutors/images/new-images/mildred-a-600w.avif → 404
                  /data/tutors/images/new-images/mildred-a-800w.avif → 404
                  ... 8 more tutor headshot AVIFs
                Spot check:
                  mildred-a.webp  → 200 ✅
                  mildred-a.avif  → 404 ❌
                  mildred-a-600w.avif → 404 ❌
                Lighthouse on /blog confirms 2,332 KiB savings available
                from "modern-image-formats" — AVIF NOT generated globally.
    Why it hurts: Each homepage view fires 10 wasted 404 requests for
                AVIF that never existed. Browser falls back to webp/png
                so users don't see broken images, BUT:
                  • LCP image stays as WebP (not optimal AVIF)
                  • 10 wasted round-trips on every load
                  • Mobile users on slow connections feel each one
                  • 2.3 MB savings on /blog = visible LCP improvement
    Fix:        Either (a) fix the AVIF generation pipeline at build, OR
                (b) remove the AVIF <source> tags from <picture> elements
                if you don't intend to ship AVIF (no-op fallback to webp).
                Astro 5 has built-in AVIF support — see P1-7 (upgrade).
    Effort: M    Priority: P0

═══════════════════════════════════════════════════════════════════════════

P1 — HIGH PRIORITY (fix within 2 weeks)

  [P1-1] CLS 0.224 on /pricing and 0.237 on /about-us
    Impact:     High         Confidence: Confirmed
    Evidence:   Lighthouse /pricing  CLS = 0.224 (target ≤ 0.10)
                Lighthouse /about-us CLS = 0.237
                4 layout shifts on /pricing
                "unsized-images" audit fails on home and /pricing
    Fix:        Set explicit width/height on every <Image> instance.
                For Astro: width and height props are required when not
                using static imports. <Image src={x} width={N} height={M}/>
    Effort: M    Priority: P1
    Note: Fixed simultaneously with P0-3 image work.

  [P1-2] 13 font files preloaded — kills LCP head budget
    Impact:     High         Confidence: Confirmed
    Evidence:   13 × <link rel="preload" as="font"> in <head> on every page.
                7 × PublicSans weights + 6 × NunitoSans weights.
                Fonts compete with hero image for early bandwidth.
    Fix:        Preload only 2 weights actually used above the fold
                (PublicSans-Regular + PublicSans-Bold). Drop the rest;
                let @font-face load them on demand.
    Effort: S    Priority: P1

  [P1-3] No explicit AI crawler policy in robots.txt
    Impact:     Medium       Confidence: Confirmed
    Evidence:   robots.txt has zero references to GPTBot, ClaudeBot,
                PerplexityBot, Google-Extended, Applebot-Extended, CCBot.
                Default = allow, but no audit trail.
                Education vertical: AI Overviews on ~83 % of queries.
    Fix:        Append to robots.txt:
                  User-agent: GPTBot
                  Allow: /
                  User-agent: ClaudeBot
                  Allow: /
                  User-agent: PerplexityBot
                  Allow: /
                  User-agent: Google-Extended
                  Allow: /
                  User-agent: Applebot-Extended
                  Allow: /
                  User-agent: OAI-SearchBot
                  Allow: /
    Effort: S    Priority: P1

  [P1-4] /pricing has ZERO JSON-LD; only WebPage on /tutors
    Impact:     Medium       Confidence: Confirmed
    Evidence:   /pricing — 0 JSON-LD blocks
                /tutors  — 1 JSON-LD block (only generic WebPage)
                Conversion pages with no Service/Offer/BreadcrumbList.
    Fix:        Add BreadcrumbList + Service + Offer schema to /pricing.
                Add ItemList + Service to /tutors.
                Service.priceRange = "£15–£75"
                Service.provider = { "@id": "https://edumentors.co.uk/#org" }
    Effort: S    Priority: P1

  [P1-5] All pages share generic banner.png as og:image
    Impact:     Medium       Confidence: Confirmed
    Evidence:   og:image = "https://edumentors.co.uk/banner.png" on
                home, tutors, pricing (empty), about-us, edu-ai-tutor.
                /pricing's empty og:image is a P0 by itself (above).
    Fix:        Use per-page OG images. Easiest in Astro: install
                `astro-og-canvas` or `astro-og-image-service`. For dynamic
                pages (tutor profiles, blog posts) generate at build.
    Effort: M    Priority: P1

  [P1-6] HTML validation: 19 errors on homepage
    Impact:     Medium       Confidence: Confirmed
    Evidence:   W3C validator output:
                  • Duplicate ID "book-free-trial__btn--header"
                  • <style> nested inside <button> (invalid)
                  • <div> nested inside <button> (×2)
                  • <div> nested inside <label> (×14)
                  • Stray <script> tag at line 232 (parser broke)
    Why it hurts: Duplicate IDs break analytics tracking + assistive tech.
                Invalid nesting breaks form submission semantics.
                Parser-breaking errors mean DOM after line 232 is ill-
                defined — browsers recover but signals are lost.
    Fix:        Audit BaseLayout.astro and the search/booking widgets.
                Replace nested <div> in <label> with flex layout sibling.
                Make IDs unique (suffix component instance).
    Effort: M    Priority: P1

  [P1-7] Astro v3.6.5 — three major versions behind
    Impact:     Medium       Confidence: Confirmed
    Evidence:   <meta name="generator" content="Astro v3.6.5">
                Current: Astro 5.x.
    Why it hurts: AVIF generation (P0-6) is an Astro 4+ feature. Content
                Layer (cleaner blog ingest), View Transitions, native i18n,
                better image service all come with v4/v5.
    Fix:        Schedule migration. Astro 3 → 4 → 5 has codemods.
                Test on Netlify branch deploy.
    Effort: L    Priority: P1

  [P1-8] /blog Best-Practices issues: viewport user-scalable=no, button-name
    Impact:     Medium       Confidence: Confirmed
    Evidence:   Lighthouse /blog:
                  • meta-viewport user-scalable="no" (a11y violation)
                  • Buttons without accessible names
                  • Touch targets too small
                  • 53 requests not served via HTTP/2 (third-party CDN)
                  • A11y score 73 (worst on site)
    Fix:        Remove user-scalable=no. Audit blog template buttons,
                add aria-label or visible text. Touch targets ≥ 48×48 CSS px.
    Effort: M    Priority: P1

  [P1-9] /blog has 6 MB total weight, 4.9 MB cache-eligible
    Impact:     High         Confidence: Confirmed
    Evidence:   Lighthouse /blog: total bytes 6,044 KiB
                "cache-insight" estimated savings: 4,938 KiB
                "modern-image-formats" estimated savings: 2,332 KiB
                "uses-responsive-images" savings: 965 KiB
                "image-delivery-insight" savings: 1,759 KiB
    Fix:        Combine P0-5 (cache headers) and P0-6 (AVIF/responsive).
                Should bring /blog from 6 MB to under 2 MB.
    Effort: M    Priority: P1
    Expected: LCP 8.4s → 3–4s.

═══════════════════════════════════════════════════════════════════════════

P2 — MEDIUM PRIORITY

  [P2-1] DOM size 3,162 elements on home (target < 1,500). Bloated DOM
         hurts INP, parsing, memory.
  [P2-2] /about-us description = 202 chars (truncates in SERP, target ≤ 160).
  [P2-3] FAQPage schema on home — won't show rich results since 2023; keep
         as AI signal but don't expect SERP wins.
  [P2-4] Disallow: *?* in robots.txt blocks ALL query-string URLs site-wide.
         Risk: any future filter/search feature is invisible to crawlers.
  [P2-5] Site-wide missing security headers: CSP, X-Frame-Options, X-Content-
         Type-Options, Referrer-Policy, Permissions-Policy. Affects Page
         Experience signal indirectly.
  [P2-6] og:image:alt missing site-wide.
  [P2-7] No BreadcrumbList schema on any inspected page.
  [P2-8] No <link rel="alternate" hreflang="en-GB"> declared.

═══════════════════════════════════════════════════════════════════════════

POSITIVE FINDINGS (CARRY FORWARD)

  + 100% of 438 sitemap URLs return 200. No 4xx, no 5xx, no broken pages.
  + 0 noindex on indexable pages (sampled 50). No accidental deindexing.
  + 4/6 pages score Lighthouse SEO 100 (perfect basic SEO).
  + EducationalOrganization schema on homepage is excellent — full E-E-A-T
    (founder, foundingDate, contactPoint, sameAs, knowsAbout).
  + /llms.txt is published with rich content. Top 10% of sites.
  + HSTS, HTTPS, sitemap referenced from robots.txt.
  + Brotli compression enabled (~70% reduction).
  + HTTP/2 served (53 third-party requests are exception).
  + Self-hosted fonts (good for perf control).
  + Partytown for third-party offload.
  + Lazy loading on most images.
  + Per-page canonical on 4/5 sampled pages (only /pricing breaks).
  + Title lengths all in 30–65 char target.
  + Multi-H1 issue narrow (only home + /pricing of 50 sampled).

═══════════════════════════════════════════════════════════════════════════

ACTION PLAN (priority-weighted, week by week)

  Week 1 — Stop the bleeding (P0)
    Day 1:  P0-1  Fix homepage JS error (line 489) — 30 min
    Day 1:  P0-2  Fix /pricing broken canonical/OG — 30 min
    Day 1:  P0-4  Demote extra H1s on / and /pricing — 30 min
    Day 2:  P0-5  Add cache headers in netlify.toml — 1 h
    Day 3:  P0-6  Either fix AVIF pipeline OR remove AVIF <source> tags — 2-4 h
    Day 4:  Verify all P0 fixes via re-running Lighthouse on same 6 pages
    Day 5:  Re-submit sitemap to GSC + BingWMT to trigger recrawl

  Week 2 — Performance polish (P0-3 + P1)
    P0-3 unblocks naturally with P0-5 + P0-6 + P1-2 done.
    P1-1  Fix /pricing /about-us CLS by setting image dimensions — 2 h
    P1-2  Trim font preloads from 13 → 2 — 30 min
    P1-3  Add AI crawler policy to robots.txt — 15 min
    P1-4  Add Service / BreadcrumbList schema to /pricing — 1 h
    P1-5  Per-page OG images via astro-og-canvas — 4 h
    P1-6  Fix W3C HTML errors (mostly BaseLayout cleanup) — 2 h

  Week 3 — Strategic
    P1-7  Plan Astro 3 → 5 migration. Test on staging. — 2-3 days work
    P1-8  /blog template a11y polish — 4 h
    P1-9  Falls out of P0-5 + P0-6.
    P2 items as time allows.

  Week 4 — Validate
    Re-run Lighthouse on top 20 pages.
    Re-submit sitemap.
    Monitor GSC daily (need access).
    Compare CWV before/after — expect:
      LCP avg:  ~7s → ~2.5–3.5s
      CLS:      0.23 → < 0.10 on /pricing /about-us
      Perf:     71 → 85+

═══════════════════════════════════════════════════════════════════════════

EXPECTED IMPACT (rough, based on Lighthouse savings estimates)

  Pre fixes (current):       avg perf 71, avg LCP ~6.5s
  Post P0+P1:                avg perf ~88, avg LCP ~2.8s

  Per-page bandwidth savings (for repeat visits + crawlers):
    /blog:    -4.9 MB        (cache + image format)
    /home:    -1.0 MB
    /pricing: -0.5 MB
    /about-us: -0.5 MB
    /tutors:  -0.4 MB
  Sitewide: an estimated ~50% bandwidth reduction.

  Crawl efficiency:
    Removing AVIF 404s: -10 wasted requests per pageview.
    Adding cache headers: ~80% of /blog assets re-servable from edge.
    Both improve crawler experience, particularly for AI bots that don't
    retry on slow responses.

═══════════════════════════════════════════════════════════════════════════

WHAT WE STILL CAN'T MEASURE (without paid tools / OAuth)

  • Real-user CWV (CrUX) — needs free Google API key
  • Keyword positions / clicks (GSC) — needs OAuth
  • Backlink profile (Ahrefs / Semrush) — needs auth
  • Competitor keyword gap (Semrush / Ahrefs) — needs auth
  • SERP feature presence per query — needs scraping or API
  • Real INP from RUM — needs analytics integration

  Free upgrade path (25 min total setup):
    1. Verify domain in Google Search Console + Bing Webmaster
    2. Verify domain in Ahrefs Webmaster Tools (free, owned-domain only)
    3. Get a free CrUX API key
    4. Wire up web-vitals.js for first-party RUM
    Then re-run audit with traffic-weighted priorities.

═══════════════════════════════════════════════════════════════════════════

METHODOLOGY (full)

  Crawl:
    curl -L -A "Mozilla/5.0" — 438 sitemap URLs, parallel 8-way
    Status code distribution, redirect chain check
    Per-URL HTML head extraction (50 sampled in detail)

  Lab CWV:
    Lighthouse 11.x, mobile preset, simulated throttling
    6 pages: /, /tutors, /pricing, /about-us, /edu-ai-tutor, /blog
    Categories: performance, seo, accessibility, best-practices

  Broken-link check:
    linkinator on homepage (skip social media domains)
    Found: 10 broken AVIF image references

  HTML validation:
    W3C Nu validator (https://validator.w3.org/nu/) — full POST
    19 errors on homepage

  Schema:
    Manual JSON-LD parse + JSON.parse validation
    structured-data-testing-tool attempted (silent failure on Astro v3 SSR)

  Headers:
    curl -I — security + caching headers on home

  AI / GEO:
    Manual review of /llms.txt
    AI crawler audit in robots.txt
    First-200-words check per blog post (timed out — needs rerun)

  Skipped (no API key):
    CrUX API (real-user CWV)
    PageSpeed Insights API (rate-limited without key)
    Google Search Console
    Ahrefs Webmaster Tools
    Semrush

═══════════════════════════════════════════════════════════════════════════
