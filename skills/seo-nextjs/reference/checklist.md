# Next.js SEO Checklist (long form)

## 1. Foundations

- [ ] **`metadataBase`** set in `app/layout.tsx` to absolute production URL.
  - Without it, OG/Twitter image URLs become relative → broken in crawlers.
- [ ] **Title template** in root: `title: { default: '...', template: '%s | Brand' }`.
- [ ] **Default description** (70–160 chars) — distinct, not corporate fluff.
- [ ] **Default OG**: site name, type, default image (1200×630).

## 2. Per-page metadata

- [ ] Every page exports `metadata` (static) or `generateMetadata` (dynamic).
- [ ] **Title 30–65 chars**, includes primary keyword + brand.
- [ ] **Description 70–160 chars**, distinct, includes primary keyword once naturally.
- [ ] **`alternates.canonical`** set — even on the canonical itself.
- [ ] **OG type** correct (`website`, `article`, `product`).
- [ ] **OG image** — page-specific via `opengraph-image.tsx` or absolute URL in metadata.
- [ ] No client component is exporting metadata (silent failure).

## 3. Sitemap & robots

- [ ] `app/sitemap.ts` exists, includes all indexable URLs with accurate `lastModified`.
- [ ] If > 50k URLs, split via `generateSitemaps`.
- [ ] No `/api/*`, `/admin/*`, or query-stringed URLs in sitemap.
- [ ] All sitemap URLs return 200 + canonical to themselves.
- [ ] `app/robots.ts` references the sitemap.
- [ ] Explicit AI crawler policy (allow or block, don't leave to default).
- [ ] Submit sitemap to Google Search Console + Bing Webmaster Tools.

## 4. Structured data (JSON-LD)

- [ ] Organization schema in root layout (site-wide).
- [ ] WebSite + SearchAction in root layout (sitelinks searchbox).
- [ ] Article on every blog post with full author block (sameAs, jobTitle).
- [ ] Product on every product page with offers, aggregateRating if real.
- [ ] BreadcrumbList on non-root pages.
- [ ] LocalBusiness if applicable.
- [ ] All JSON-LD passes Rich Results Test (no errors AND no warnings).
- [ ] XSS guard: `JSON.stringify(...).replace(/</g, '\\u003c')`.
- [ ] No HowTo schema (deprecated). No fabricated FAQPage.

## 5. Performance — Core Web Vitals

- [ ] **LCP ≤ 2.5s** (p75 mobile). Hero image uses `<Image priority>`.
- [ ] **INP ≤ 200ms**. Heavy click handlers use `useTransition`. Third-party scripts lazy-loaded.
- [ ] **CLS ≤ 0.1**. All images have explicit width/height. Fonts use `display: swap`.
- [ ] **TTFB ≤ 800ms**. ISR or SSG for marketing pages; edge runtime for dynamic.
- [ ] Above-the-fold data fetched server-side (not in `useEffect`).
- [ ] Bundle analyzed with `@next/bundle-analyzer`; no unexpected client deps.

## 6. Images

- [ ] Every image uses `next/image` (never `<img>` for content).
- [ ] LCP image has `priority`.
- [ ] Below-fold images have `loading="lazy"` (default) and `decoding="async"` (default).
- [ ] `sizes` attribute set correctly (responsive).
- [ ] Modern formats configured: AVIF + WebP fallback (Next default ships WebP; AVIF via `images.formats: ['image/avif', 'image/webp']` in `next.config`).
- [ ] All images have meaningful `alt` (5–125 chars). Decorative images use `alt=""`.
- [ ] Remote images: `images.remotePatterns` configured.

## 7. Fonts

- [ ] Use `next/font/google` or `next/font/local`.
- [ ] `display: 'swap'` (default in next/font).
- [ ] Subsets specified (`['latin']` minimum).
- [ ] Variable fonts preferred over multiple weight files.
- [ ] No `@import` from Google Fonts CSS (loses optimization).

## 8. Content & E-E-A-T

- [ ] H1 present, exactly one, 20–70 chars, includes primary keyword.
- [ ] Heading hierarchy correct (no H2→H4 skips).
- [ ] Article ≥ 800 words; if competing for high-volume keyword, match competitor depth (often 2000+).
- [ ] Author block on every article: name, photo, bio, link to author page, social links.
- [ ] Publication date AND last-modified date visible.
- [ ] Citations / external sources linked (authoritative outbound links improve E-E-A-T).
- [ ] About page + Contact page exist and are linked from footer.
- [ ] HTTPS everywhere; mixed content fixed.

## 9. Internal linking

- [ ] Every indexable page is reachable in ≤ 3 clicks from home.
- [ ] Topic-cluster architecture: pillars (3–5k words) + spokes (800–2000 words) bidirectionally linked.
- [ ] Anchor text varied: ~15–25% exact match, 30–40% partial, 25–35% semantic.
- [ ] No orphan pages (page with 0 internal inlinks).
- [ ] Broken internal links = 0.

## 10. AI search (GEO/AEO)

- [ ] First 200 words contain a direct answer to the page's primary question.
- [ ] Article includes self-contained 134–167 word passages answering specific sub-questions.
- [ ] Original data, statistics, or first-party research present.
- [ ] Entity-rich prose (named people, products, dates) — limit pronouns where entities matter.
- [ ] AI crawler policy explicit in `robots.ts`.
- [ ] Optional: `/llms.txt` published (no Google ranking benefit, but useful for Anthropic/Perplexity ingestion).

## 11. International SEO (if applicable)

- [ ] `alternates.languages` set with all locale variants.
- [ ] Hreflang reciprocal (every variant points back to all others).
- [ ] `x-default` hreflang for the language selector / default locale.
- [ ] Locale-specific URLs (`/en/...`, `/fr/...`) not `?lang=` parameters.

## 12. Validation gates (CI)

- [ ] `npx structured-data-testing-tool` on key URLs.
- [ ] Lighthouse CI on PRs (fail if LCP/INP/CLS regress).
- [ ] Broken-link check on staging.
- [ ] Sitemap validation: all URLs return 200, all canonical to self.
