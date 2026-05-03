# Astro & Nuxt SEO Checklist (long form)

## 1. Foundations (both)

- [ ] **Production URL** configured: `site:` in `astro.config` / `site.url` in `nuxt.config`.
- [ ] **`trailingSlash`** consistent — pick one and stick with it; canonical depends on it.
- [ ] **HTTPS** everywhere; mixed content fixed.
- [ ] **lang** attribute on `<html>`.
- [ ] **viewport** meta on every page.

## 2. Per-page metadata

- [ ] Every page has unique title (30–65 chars) including primary keyword.
- [ ] Every page has unique meta description (70–160 chars).
- [ ] Every page has explicit canonical (even if pointing to itself).
- [ ] Open Graph: `og:title`, `og:description`, `og:url`, `og:image`, `og:type`.
- [ ] Twitter Card: `twitter:card=summary_large_image` minimum.
- [ ] No accidental `noindex` on indexable pages.

### Astro-specific
- [ ] `BaseLayout.astro` centralizes meta — no per-page `<head>` reinvention.
- [ ] Canonical built with `new URL(Astro.url.pathname, Astro.site)`.
- [ ] If using View Transitions, verified per-page meta still updates on nav.

### Nuxt-specific
- [ ] `useSeoMeta` on every page (NOT `useHead({ meta })` for the same tags).
- [ ] All reactive metadata uses functions: `title: () => x.value?.title`.
- [ ] `useServerSeoMeta` where reactivity isn't needed (smaller bundle).
- [ ] Default app head set in `nuxt.config.ts → app.head`.

## 3. Sitemap & robots

### Astro
- [ ] `@astrojs/sitemap` integration installed and configured.
- [ ] `site:` set in config (sitemap silently no-ops without it).
- [ ] `filter` excludes `/admin/`, draft routes, etc.
- [ ] `src/pages/robots.txt.ts` exists and references sitemap-index.xml.

### Nuxt
- [ ] `@nuxtjs/sitemap` configured (or via `@nuxtjs/seo` meta-module).
- [ ] `@nuxtjs/robots` configured with explicit rules.
- [ ] `nitro.prerender.routes` includes `/sitemap.xml` and `/robots.txt`.
- [ ] `routeRules` marks admin/api with `robots: false`.

### Both
- [ ] Sitemap returns 200, valid XML.
- [ ] All sitemap URLs return 200 + canonical to themselves.
- [ ] No `/api/*` or query-stringed URLs.
- [ ] Submitted to GSC + Bing Webmaster Tools.
- [ ] Explicit AI crawler policy (allow OR block, not default).

## 4. Structured data (JSON-LD)

### Astro
- [ ] JSON-LD inlined via `<script type="application/ld+json" set:html={...} />`.
- [ ] XSS-escape: `JSON.stringify(...).replace(/</g, '\\u003c')`.
- [ ] Organization schema in BaseLayout (site-wide).
- [ ] Article schema on every blog post (full author E-E-A-T).
- [ ] BreadcrumbList on non-root pages.

### Nuxt
- [ ] `nuxt-schema-org` installed (via `@nuxtjs/seo`).
- [ ] `useSchemaOrg([defineOrganization(...), defineWebSite(...)])` in root layout.
- [ ] `useSchemaOrg([defineArticle(...), defineBreadcrumb(...)])` per page.
- [ ] Author block includes `jobTitle` and `sameAs` (E-E-A-T).

### Both
- [ ] All JSON-LD passes Rich Results Test (no errors AND no warnings).
- [ ] No HowTo schema (deprecated).
- [ ] No fabricated FAQPage on marketing pages.
- [ ] Schema reflects content actually visible on the page.

## 5. Performance — Core Web Vitals

- [ ] **LCP ≤ 2.5s** (p75 mobile).
- [ ] **INP ≤ 200ms**.
- [ ] **CLS ≤ 0.1**.
- [ ] **TTFB ≤ 800ms**.
- [ ] Above-the-fold data fetched server-side.

### Astro
- [ ] LCP image: `loading="eager"` on `<Image>`.
- [ ] Below-fold images: default lazy.
- [ ] Modern formats: `<Picture formats={['avif', 'webp']}>`.
- [ ] Use `output: 'static'` or `'hybrid'`; avoid pure SSR for marketing.
- [ ] Islands hydration directives set correctly (`client:idle`, `client:visible`).

### Nuxt
- [ ] LCP image: `<NuxtImg preload>`.
- [ ] Below-fold: `<NuxtImg loading="lazy">`.
- [ ] `format="avif,webp"` on `<NuxtImg>`.
- [ ] `routeRules` with `prerender:true` or `isr:N` for indexable pages.
- [ ] **Never** `ssr:false` for indexable content — SPA mode is SEO-fatal.

## 6. Images

- [ ] Every content image uses framework component (`<Image>` Astro / `<NuxtImg>` Nuxt).
- [ ] Explicit `width`/`height` (or `aspect-ratio`) — kills CLS.
- [ ] Modern formats: AVIF + WebP fallback.
- [ ] Meaningful `alt` (5–125 chars). Decorative: `alt=""`.
- [ ] LCP image marked priority (Astro: `loading="eager"`; Nuxt: `preload`).
- [ ] `sizes` attribute on responsive images.

## 7. Fonts

- [ ] Self-hosted or via framework integration (Astro: `astro-font` or local; Nuxt: `@nuxt/fonts`).
- [ ] `font-display: swap`.
- [ ] Latin subset only (or specific to content).
- [ ] Variable fonts preferred.
- [ ] Preload critical font(s).

## 8. Content & E-E-A-T

- [ ] H1 present, exactly one, 20–70 chars, includes primary keyword.
- [ ] Heading hierarchy correct (no skips).
- [ ] Article ≥ 800 words; competitive niches often need 2000+.
- [ ] Author block: name, photo, bio, link to author page, social links.
- [ ] Publication + last-modified dates visible.
- [ ] Citations / external authoritative links present.
- [ ] About + Contact pages exist, linked from footer.

## 9. Internal linking

- [ ] Every indexable page reachable in ≤ 3 clicks from home.
- [ ] Topic clusters: pillars + spokes bidirectionally linked.
- [ ] Anchor variety: ~15–25% exact, 30–40% partial, 25–35% semantic.
- [ ] Zero orphan pages.
- [ ] Zero broken internal links.

### Tooling
- [ ] Astro: `astro-seo-checker` or manual crawl.
- [ ] Nuxt: `nuxt-link-checker` (in `@nuxtjs/seo`).

## 10. AI search (GEO/AEO)

- [ ] First 200 words contain a direct answer to the page's primary question.
- [ ] Self-contained 134–167 word passages for sub-questions.
- [ ] Original data / first-party research present.
- [ ] Entity-rich prose (named people, products, dates).
- [ ] AI crawler policy explicit in robots.
- [ ] Optional: `/llms.txt` published for Anthropic/Perplexity ingestion.

## 11. International SEO (if applicable)

- [ ] Locale-specific URLs (`/en/...`, `/fr/...`), not `?lang=`.
- [ ] Hreflang on every page including `x-default`.
- [ ] Reciprocal hreflang (every variant references all others).
- [ ] Astro: i18n routing config + sitemap with `i18n: { defaultLocale, locales }`.
- [ ] Nuxt: `@nuxtjs/i18n` integrated with `@nuxtjs/seo`.

## 12. Validation gates (CI)

- [ ] `npx structured-data-testing-tool` on key URLs.
- [ ] Lighthouse CI on PRs.
- [ ] Broken-link check on staging (Nuxt: `nuxt-link-checker`).
- [ ] Sitemap validation in CI: every URL returns 200 + correct canonical.
- [ ] Schema validation in CI.
