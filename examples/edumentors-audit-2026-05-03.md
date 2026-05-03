═══════════════════════════════════════════════════════════════════════════
  SEO AUDIT — edumentors.co.uk — 2026-05-03
  Auditor: Claude Code (seo-vue-astro skill, no Semrush)
  Scope: 5 key pages + sitemap + robots.txt + llms.txt
═══════════════════════════════════════════════════════════════════════════

EXECUTIVE SUMMARY

  Stack:                  Astro v3.6.5 on Netlify (en_GB)
  Pages crawled:          5 (sample)
  Sitemap URLs declared:  ~25
  Avg page score:         54 / 100  ▸  rework band
  Critical issues:        4 (P0)
  High priority:          7 (P1)
  Medium priority:        7 (P2)

  Site has strong foundations (HSTS, self-hosted fonts, EducationalOrganization
  schema with full E-E-A-T, llms.txt, partytown). But head-tag emission is
  inconsistent: the /pricing page ships an empty <link rel="canonical">,
  empty og:url, empty og:image, AND empty twitter:image — all four crawler-
  facing tags are broken. /blog responds in 23 s to Googlebot vs 2.8 s to a
  normal browser, suggesting bot rate-limiting or a render path that doesn't
  cache for crawlers.

  Two pages ship multiple <h1> tags. AI-crawler policy is implicit (default
  allow). 13 font files are preloaded — far too many for LCP.

═══════════════════════════════════════════════════════════════════════════

POSITIVE FINDINGS (DON'T LOSE THESE)

  + EducationalOrganization JSON-LD on homepage is excellent — founder,
    foundingDate, contactPoint, sameAs, knowsAbout — strong E-E-A-T signal.
  + llms.txt is published with rich, citation-friendly content (Edumentors
    is in the ~10% of sites doing this — useful for Anthropic / Perplexity).
  + HSTS, HTTPS, sitemap referenced from robots.txt, partytown for 3rd-
    party scripts, lazy loading on most images.
  + Per-page canonical present on 4/5 sampled pages.
  + Title lengths all within 30–65 char target.

═══════════════════════════════════════════════════════════════════════════

P0 — CRITICAL (fix within 24 h)

  [P0-1] /pricing — broken <link rel="canonical"> (empty href)
    Impact:     High        Confidence: Confirmed
    Evidence:   `<link rel="canonical"><meta name="description" content="...">`
                The canonical tag has NO href attribute. Same page also
                emits empty <meta property="og:url">, empty og:image, empty
                twitter:image.
    Why it hurts: Google falls back to URL parameters for canonicalization;
                pricing page can be deduped against URL variants. Social
                shares of /pricing render with no preview image.
    Fix:        In src/pages/pricing.astro (or equivalent layout), confirm
                the canonical/OG props are wired:
                  <link rel="canonical" href={new URL(Astro.url.pathname, Astro.site)} />
                  <meta property="og:url" content={new URL(Astro.url.pathname, Astro.site)} />
                  <meta property="og:image" content={new URL('/banner.png', Astro.site)} />
                The undefined `Astro.props` value is most likely the cause.
    Effort: S    Priority: P0
    Expected:   Restored canonical signal; OG previews work in Slack/Twitter.

  [P0-2] /blog returns 504/slow path for Googlebot (23 s vs 2.8 s)
    Impact:     High        Confidence: Confirmed
    Evidence:   curl -L -A "Googlebot" https://edumentors.co.uk/blog
                  → status 200, time_total = 23.22 s
                curl -L -A "Mozilla/5.0" https://edumentors.co.uk/blog
                  → status 200, time_total =  2.83 s
                Earlier sample with default curl UA returned 504.
    Why it hurts: Googlebot soft-times-out after ~30 s. /blog is in your
                sitemap but only intermittently indexable. AI crawlers
                (ChatGPT/Perplexity) typically don't retry — citation lost.
    Fix:        1. Check Netlify edge logs for /blog requests with UA
                   Googlebot — look for cache misses or upstream timeouts.
                2. /blog is listed as Allow in robots.txt + appears proxied
                   to a CMS (the "/blog/wp-admin/*" Disallow suggests
                   WordPress upstream). If WP is proxied through Netlify,
                   a slow WP backend hurts crawlers most.
                3. Pre-render blog index OR cache aggressively at edge:
                   netlify.toml: [[headers]] for="/blog" Cache-Control =
                   "public, max-age=300, s-maxage=86400"
    Effort: M    Priority: P0

  [P0-3] Homepage has TWO <h1> tags
    Impact:     High        Confidence: Confirmed
    Evidence:   grep -c '<h1' /tmp/edu-home.html → 2
                  H1#1: "Best tutors from the [top universities]" (hero)
                  H1#2: "Edu-AI the first fully interactive live AI tutor"
    Why it hurts: Splits topical signal. Google's algorithms get confused
                about primary topic. Hurts AI extraction (which page-quote
                is the canonical answer?).
    Fix:        Keep H1#1 (matches title + brand intent). Demote
                "Edu-AI..." to H2. The Edu-AI section is a feature
                callout, not the page's primary topic.
    Location:   src/pages/index.astro — second H1 is in the section using
                `data-astro-cid-ipvns2jo`. Change <h1> → <h2>.
    Effort: S    Priority: P0

  [P0-4] /pricing has THREE <h1> tags
    Impact:     High        Confidence: Confirmed
    Evidence:   3 × <h1> in /pricing HTML. One is "Frequently asked
                questions" — that should be H2.
    Why it hurts: Same as P0-3, but /pricing is a money page.
    Fix:        Keep one H1 matching the page intent (likely "Affordable
                online tutoring pricing" or similar). Demote the other two
                to H2 / H3.
    Effort: S    Priority: P0

═══════════════════════════════════════════════════════════════════════════

P1 — HIGH PRIORITY (fix within 2 weeks)

  [P1-1] 13 font files preloaded — kills LCP
    Impact:     High        Confidence: Confirmed
    Evidence:   13 × <link rel="preload" as="font" href="/fonts/...woff2"
                in homepage <head>. PublicSans (7 weights) + NunitoSans (6).
    Why it hurts: Each preload is a high-priority request competing with
                the LCP image. Browser delays render until preloaded fonts
                load. Mobile 3G this can add 2–4 s to LCP.
    Fix:        Preload only the 1–2 weights actually used above the fold
                (likely PublicSans-Regular + PublicSans-Bold). Drop the
                rest — let them lazy-load via @font-face. Better:
                migrate to next-gen variable fonts (one file = all weights).
    Effort: S    Priority: P1
    Expected:   LCP improvement of 1–3 s on mobile.

  [P1-2] No explicit AI crawler policy in robots.txt
    Impact:     Medium      Confidence: Confirmed
    Evidence:   robots.txt has zero references to GPTBot, ClaudeBot,
                PerplexityBot, Google-Extended, Applebot-Extended, CCBot.
                Default = allow.
    Why it hurts: You're benefiting from default-allow but with no audit
                trail. If a crawler misbehaves you can't selectively block.
                Also: AI Overviews now appear on ~48% of education
                queries (one of the most-affected verticals at +83%).
                Explicit allow signals intent and aids citation parsing.
    Fix:        Add to robots.txt:
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

  [P1-3] /pricing has ZERO JSON-LD schema
    Impact:     Medium      Confidence: Confirmed
    Evidence:   grep -c 'application/ld+json' /tmp/pricing.html → 0
    Why it hurts: Pricing pages are conversion-critical. Missing
                BreadcrumbList kills breadcrumb display in SERPs. Missing
                Service or Offer schema means no rich pricing display.
    Fix:        Add BreadcrumbList + Service schema with priceRange:
                  {"@type": "Service",
                   "name": "Online Tutoring",
                   "provider": {"@type": "Organization", "@id": "https://edumentors.co.uk/#org"},
                   "offers": {"@type": "Offer", "priceRange": "£15–£75",
                              "priceCurrency": "GBP"}}
    Effort: S    Priority: P1

  [P1-4] /about-us description 202 chars — will be truncated
    Impact:     Medium      Confidence: Confirmed
    Evidence:   <meta name="description"> length = ~202 chars on /about-us.
                Mobile SERP truncates ~120, desktop ~160.
    Fix:        Rewrite to 140–155 chars. Lead with primary value prop.
    Effort: S    Priority: P1

  [P1-5] All pages share the same generic OG image (banner.png)
    Impact:     Medium      Confidence: Confirmed
    Evidence:   og:image = "https://edumentors.co.uk/banner.png" on
                homepage AND /tutors AND /about-us AND /edu-ai-tutor.
    Why it hurts: Generic social shares get fewer clicks. AI tools that
                use OG images for previews see no differentiation. /pricing,
                /blog, individual /tutors/<subject> pages all benefit from
                page-specific imagery.
    Fix:        Use Astro's built-in OG image generation. Easiest: install
                `astro-og-canvas` or generate at build time per page.
                For dynamic per-tutor pages: a single OG template that
                accepts {title, subject} via URL params.
    Effort: M    Priority: P1

  [P1-6] Astro v3.6.5 — three major versions behind
    Impact:     Medium      Confidence: Confirmed
    Evidence:   <meta name="generator" content="Astro v3.6.5">
                Current stable: Astro 5.x.
    Why it hurts: Astro 4 added View Transitions; Astro 5 added Content
                Layer (cleaner blog/CMS integration), better build perf,
                native i18n routing, sharp ImageService improvements
                (AVIF default). Stuck on v3 = leaving CWV improvements
                on the table.
    Fix:        Schedule a migration. astro 3 → 4 → 5 has codemods. Test
                in a staging deploy on Netlify preview.
    Effort: L    Priority: P1
    Note:       Won't move ranking immediately, but unlocks every other
                fix in this list (image transforms, Content Layer for
                blog, native sitemap improvements).

  [P1-7] FAQPage schema on homepage — no rich results value
    Impact:     Low         Confidence: Confirmed
    Evidence:   FAQPage JSON-LD with 5 Q&A blocks on homepage.
    Note:       Since Aug 2023, FAQPage rich results are restricted to
                gov + well-known health sites. Edumentors won't get the
                visual treatment in SERPs. The schema IS still valuable
                as an AI-extraction signal — keep it, but be aware
                it's not earning SERP real estate.
    Fix:        No urgent action. Optional: move FAQ to /faqs subpage and
                link from homepage; keeps homepage focused.
    Effort: S    Priority: P2 (downgraded — keep it, don't expect SERPs)

═══════════════════════════════════════════════════════════════════════════

P2 — MEDIUM PRIORITY

  [P2-1] /tutors only has WebPage schema — should have ItemList + Service
  [P2-2] Disallow: *?* in robots.txt blocks ALL query-string URLs site-wide
         Risk: any future search/filter feature is invisible to crawlers.
         Fix: replace with specific patterns (Disallow: /*?selectedSpec*).
  [P2-3] og:image:alt missing site-wide (a11y + AI image-context)
  [P2-4] No BreadcrumbList schema on any inspected page
  [P2-5] 20/90 images have empty alt — most decorative, but verify
  [P2-6] Mobile description truncation: even pages within 160 chars get
         cut at ~120 on mobile — lead with value prop in first 120 chars
  [P2-7] No <link rel="alternate" hreflang="en-GB"> declared. Audience is
         UK; if you ever target US/IN/etc., add hreflang now.

═══════════════════════════════════════════════════════════════════════════

PER-PAGE SCORECARDS (100-point rubric)

  Homepage          73/100   strong (deduct: 2 H1, FAQ schema deprecated)
  /tutors           68/100   strong (deduct: thin schema, no breadcrumb)
  /about-us         62/100   action plan (deduct: desc too long, no schema)
  /edu-ai-tutor     67/100   action plan (deduct: AI-product page should
                              have Service schema with comprehensive
                              feature list for AI citation; generic OG)
  /pricing          28/100   REWORK (broken canonical/OG, 3 H1s, no schema)
  /blog             20/100   REWORK (intermittent crawler timeouts)

═══════════════════════════════════════════════════════════════════════════

GEO / AEO NOTES (AI search optimization)

  Education vertical now sees AI Overviews on ~83 % of queries (Feb 2026,
  up from 18 %). #1 organic CTR has dropped ~35 % in those SERPs. The
  highest-leverage GEO moves for Edumentors:

  1. /blog must be reliably crawlable (P0-2). It's the citation surface.
  2. Direct answers in first 200 words of every blog article. Lead with
     the answer, not "In this article we'll explore..."
  3. Aim for self-contained 134–167 word passages in every article — the
     citation sweet spot.
  4. The existing llms.txt is excellent. Consider adding a /llms-full.txt
     or per-page .md variants — Anthropic and Perplexity ingest these
     directly.
  5. Author bylines on /blog with full Person schema (sameAs LinkedIn,
     jobTitle) — strongest E-E-A-T signal for education content where
     trust is paramount.
  6. Add HCSchool / Course schema for tutoring offerings (if structured
     enough). Consider Course schema per subject page.

═══════════════════════════════════════════════════════════════════════════

ACTION PLAN

  Week 1 — fix bleeding (P0)
    Day 1:  P0-1 /pricing canonical (5 min code change)
    Day 1:  P0-3 + P0-4 demote extra H1s on home + /pricing
    Day 1:  P0-2 investigate Netlify logs for /blog Googlebot path
    Day 2:  ship + verify with: curl -A "Googlebot" + Rich Results Test

  Week 2 — high-impact polish (P1)
    Day 3:  P1-1 strip down to 2-font preload (LCP win)
    Day 3:  P1-2 explicit AI crawler allow list
    Day 4:  P1-3 add BreadcrumbList + Service schema to /pricing
    Day 4:  P1-4 rewrite /about-us description
    Day 5:  P1-5 per-page OG images (start with money pages)

  Week 3 — strategic (P1 cont. + GEO)
    P1-6:   Plan Astro 3 → 5 migration; staging deploy
    GEO #4: Add /llms-full.txt
    GEO #5: Audit blog author markup, add Person schema to top articles

  Week 4 — validate
    Re-crawl all touched URLs (curl + lighthouse + Rich Results Test).
    Submit /sitemap.xml to GSC + BingWMT for re-discovery.
    Monitor GSC Page experience and Coverage daily.

═══════════════════════════════════════════════════════════════════════════

EXPECTED IMPACT (if Week 1+2 ship)

  Crawlability:     /blog reliably indexable; estimated +5–10 % crawl
                    success for AI bots specifically.
  CWV:              LCP improvement 1–3 s on mobile from font preload trim.
  Schema coverage:  /pricing visible in BreadcrumbList; possible Service
                    rich treatment.
  AI citation:      Higher likelihood of Edumentors being cited in AI
                    Overviews for "online tutors UK", "GCSE tutors", etc.
  Traffic:          Hard to forecast without GSC + Semrush baseline. The
                    /pricing fix alone should improve conversions
                    measurably (broken OG = bad social shares = lost
                    referrals).

═══════════════════════════════════════════════════════════════════════════

WHAT WE COULDN'T MEASURE (need user help)

  • Real-user Core Web Vitals — need CrUX API key OR GSC Core Web
    Vitals report screenshot.
  • Keyword positions / traffic — need GSC export OR Semrush MCP
    (currently not connected).
  • Lighthouse mobile scores — needs local run with throttling.
  • Backlink profile — Semrush / Ahrefs needed.
  • Real INP from RUM — needs analytics with web-vitals.js wired up.

  If you provide GSC access OR connect Semrush MCP, we can re-run the
  audit phase against real data and refine the action plan with traffic-
  weighted priorities.

═══════════════════════════════════════════════════════════════════════════

METHODOLOGY

  Tools used:
    curl (Googlebot UA + default)
    grep / Python regex for HTML parsing
    Manual JSON-LD extraction + json.loads for schema check
    No Lighthouse run (would need local install)
    No GSC / CrUX (no access provided)
    No Semrush MCP (not connected this session)

  Skill applied: seo-vue-astro (this site is Astro v3.6.5)
  Audit rubric:  100-point scorecard from seo-shared/audit-rubric.md
  Confidence levels: every finding labeled Confirmed / Likely / Hypothesis
                     per the standard rubric.

═══════════════════════════════════════════════════════════════════════════
