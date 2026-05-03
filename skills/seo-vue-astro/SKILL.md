---
name: seo-vue-astro
description: SEO audit and optimization for Astro 5 and Vue 3 / Nuxt 3 (TypeScript). Covers technical SEO, head metadata, structured data, Core Web Vitals 2026, and GEO/AEO (AI search). Trigger on requests to "audit SEO", "improve rankings", "fix Core Web Vitals", "add structured data", or any Astro/Vue/Nuxt SEO question.
---

# Astro & Vue/Nuxt SEO Skill (TypeScript, 2026)

> **Trigger:** Astro 5 OR Nuxt 3 (Vue 3 + TS) project + any SEO/performance/visibility question.
> **Output:** Evidence-backed audit + per-page fixes + validation plan.
> **Stance:** Data-driven. Cite specific URLs, line numbers, metric values. Confidence-labeled findings.

---

## Skill contract

| | |
|---|---|
| **Inputs** | Domain, optional GSC/CrUX/GA4 access, target keywords, scope |
| **Phases** | 1. Detect → 2. Audit → 3. Score → 4. Fix → 5. Validate |
| **Outputs** | Findings with evidence + confidence, top-10 prioritized action plan, per-page code patches |
| **Reference files** | `reference/astro.md`, `reference/nuxt.md`, `reference/checklist.md`, `../seo-shared/*` |
| **Optional MCPs** | Semrush MCP (`mcp__semrush__*`). If connected, run the augmented audit module in `../seo-shared/semrush-mcp.md`. The skill works fully without it — Semrush adds competitor & keyword-volume evidence on top. |

Read shared references for CWV thresholds, JSON-LD templates, GEO patterns, scoring rubric.

---

## Phase 1 — Detect

```bash
# Astro?
test -f astro.config.mjs -o -f astro.config.ts -o -f astro.config.js && echo "Astro"

# Nuxt?
test -f nuxt.config.ts -o -f nuxt.config.js && echo "Nuxt"

# Versions
node -e "const p=require('./package.json').dependencies; console.log({astro:p.astro, nuxt:p.nuxt, vue:p.vue})"

# TypeScript
test -f tsconfig.json && echo "TS"

# Existing SEO surface
fd -e astro -e vue -e ts 'BaseLayout|head|seo|sitemap|robots' src/
```

Branch from here:
- **Astro detected** → use `reference/astro.md` patterns
- **Nuxt detected** → use `reference/nuxt.md` patterns
- **Both** (monorepo) → audit each separately

---

## Phase 2 — Audit (framework-agnostic crawl)

> **Optional**: if Semrush MCP is connected, before Phase 2.1 run the full Semrush audit module in `../seo-shared/semrush-mcp.md` to seed competitor data, keyword inventory, and position-tracking evidence. Skip if unavailable — the rest of the audit stands alone.

### 2.1 Crawl

Framework-agnostic crawl-level checks:

```bash
# Status codes for top 50 URLs
xargs -I{} curl -s -o /dev/null -w "%{http_code} %{redirect_url} {}\n" < urls.txt

# Sitemap reachable + valid
curl -s https://site.com/sitemap-index.xml | xmllint --noout - && echo "OK"

# robots.txt
curl -s https://site.com/robots.txt
```

Per-page extraction parses **rendered HTML** (Googlebot UA). For Astro this is straight HTML. For Nuxt SSR-rendered HTML — but verify the page isn't accidentally CSR-only.

Extract per page:
- `<title>` (length 30–65)
- `<meta name="description">` (70–160)
- `<link rel="canonical">` count + target
- `<h1>` count and length
- All headings (no H2→H4 skips)
- `<script type="application/ld+json">` blocks
- OG/Twitter completeness
- Image audit (alt, dimensions)

---

## Phase 3 — Astro 5 patterns

### 3.1 Astro head/metadata via Layout

Astro has no built-in metadata API — meta tags live in a layout's `<head>`. Centralize.

```astro
---
// src/layouts/BaseLayout.astro
import '@/styles/global.css'

interface Props {
  title: string
  description?: string
  ogImage?: string
  ogType?: 'website' | 'article' | 'product'
  noindex?: boolean
  publishedAt?: string
  modifiedAt?: string
}

const {
  title,
  description = 'Default site description.',
  ogImage = '/og-default.png',
  ogType = 'website',
  noindex = false,
  publishedAt,
  modifiedAt,
} = Astro.props

// site MUST be set in astro.config or canonical breaks
const canonical = new URL(Astro.url.pathname, Astro.site).toString()
const ogUrl = new URL(ogImage, Astro.site).toString()
---
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>{title}</title>
  <meta name="description" content={description} />
  <link rel="canonical" href={canonical} />
  {noindex && <meta name="robots" content="noindex, nofollow" />}

  <meta property="og:type" content={ogType} />
  <meta property="og:title" content={title} />
  <meta property="og:description" content={description} />
  <meta property="og:url" content={canonical} />
  <meta property="og:image" content={ogUrl} />
  <meta property="og:image:width" content="1200" />
  <meta property="og:image:height" content="630" />
  {publishedAt && <meta property="article:published_time" content={publishedAt} />}
  {modifiedAt && <meta property="article:modified_time" content={modifiedAt} />}

  <meta name="twitter:card" content="summary_large_image" />
  <meta name="twitter:title" content={title} />
  <meta name="twitter:description" content={description} />
  <meta name="twitter:image" content={ogUrl} />
  
  <slot name="head" />
</head>
<body>
  <slot />
</body>
</html>
```

**Critical**: `astro.config` must set `site: 'https://example.com'` or sitemap and `Astro.site` silently break.

```ts
// astro.config.ts
import { defineConfig } from 'astro/config'
import sitemap from '@astrojs/sitemap'

export default defineConfig({
  site: 'https://example.com',  // CRITICAL
  trailingSlash: 'never',         // pick one and stick
  integrations: [
    sitemap({
      filter: (page) => !page.includes('/admin/'),
      changefreq: 'weekly',
      lastmod: new Date(),
    }),
  ],
})
```

### 3.2 Sitemap

Use `@astrojs/sitemap` integration. Auto-generates `sitemap-index.xml` + chunked `sitemap-N.xml`. Filter out non-indexable.

For per-collection split (advisable when blog + docs + products coexist):

```ts
sitemap({
  filter: (page) => !page.includes('/admin/'),
  serialize: (item) => {
    if (item.url.includes('/blog/')) return { ...item, changefreq: 'weekly', priority: 0.8 }
    if (item.url.includes('/docs/')) return { ...item, changefreq: 'monthly', priority: 0.6 }
    return item
  },
})
```

### 3.3 Robots

No first-party Astro module. Two options:

```ts
// src/pages/robots.txt.ts (recommended — typed, dynamic)
import type { APIRoute } from 'astro'

const SITE = 'https://example.com'

export const GET: APIRoute = () => {
  const body = `
User-agent: *
Allow: /
Disallow: /admin/

# AI crawlers — see ../seo-shared/geo-aeo.md for full list
User-agent: GPTBot
Allow: /
User-agent: ClaudeBot
Allow: /
User-agent: PerplexityBot
Allow: /
User-agent: Google-Extended
Allow: /

Sitemap: ${SITE}/sitemap-index.xml
`.trim()
  return new Response(body, { headers: { 'Content-Type': 'text/plain' } })
}
```

### 3.4 Content collections (typed blog SEO)

```ts
// src/content.config.ts
import { defineCollection, z } from 'astro:content'
import { glob } from 'astro/loaders'

const blog = defineCollection({
  loader: glob({ pattern: '**/*.md', base: './src/content/blog' }),
  schema: z.object({
    title: z.string().min(20).max(70),
    description: z.string().min(70).max(160),
    publishedAt: z.coerce.date(),
    modifiedAt: z.coerce.date().optional(),
    heroImage: z.string(),
    author: z.object({
      name: z.string(),
      slug: z.string(),
      jobTitle: z.string(),
      sameAs: z.array(z.string().url()),
    }),
    tags: z.array(z.string()).default([]),
    draft: z.boolean().default(false),
  }),
})

export const collections = { blog }
```

Now SEO frontmatter is typed at build time. The post page reads it:

```astro
---
// src/pages/blog/[...slug].astro
import { getCollection, render } from 'astro:content'
import BaseLayout from '@/layouts/BaseLayout.astro'

export async function getStaticPaths() {
  const posts = (await getCollection('blog')).filter(p => !p.data.draft)
  return posts.map(p => ({ params: { slug: p.id }, props: { post: p } }))
}

const { post } = Astro.props
const { Content } = await render(post)

const jsonLd = {
  '@context': 'https://schema.org',
  '@type': 'Article',
  headline: post.data.title,
  description: post.data.description,
  image: [new URL(post.data.heroImage, Astro.site).toString()],
  datePublished: post.data.publishedAt.toISOString(),
  dateModified: (post.data.modifiedAt ?? post.data.publishedAt).toISOString(),
  author: [{
    '@type': 'Person',
    name: post.data.author.name,
    url: new URL(`/authors/${post.data.author.slug}`, Astro.site).toString(),
    jobTitle: post.data.author.jobTitle,
    sameAs: post.data.author.sameAs,
  }],
  publisher: {
    '@type': 'Organization',
    name: 'Site Name',
    logo: { '@type': 'ImageObject', url: new URL('/logo.png', Astro.site).toString() },
  },
}
---
<BaseLayout
  title={post.data.title}
  description={post.data.description}
  ogImage={post.data.heroImage}
  ogType="article"
  publishedAt={post.data.publishedAt.toISOString()}
  modifiedAt={post.data.modifiedAt?.toISOString()}
>
  <script type="application/ld+json" set:html={JSON.stringify(jsonLd).replace(/</g, '\\u003c')} />
  <article>
    <h1>{post.data.title}</h1>
    <Content />
  </article>
</BaseLayout>
```

`set:html` is Astro's escape hatch — XSS-guard `<` to `<`.

### 3.5 Astro images

```astro
---
import { Image, Picture } from 'astro:assets'
import hero from '@/assets/hero.jpg'  // build-time optimized
---
<!-- LCP image: eager, modern format -->
<Image src={hero} alt="Hero" widths={[400, 800, 1200]} sizes="100vw" loading="eager" />

<!-- Below the fold: lazy (default) + AVIF preferred -->
<Picture src={someImage} alt="..." formats={['avif', 'webp']} widths={[400, 800]} sizes="(max-width: 768px) 100vw, 50vw" />
```

`<Image>` requires `width`/`height` (or static import infers them). Required to prevent CLS.

### 3.6 View Transitions impact on SEO

`<ClientRouter />` adds SPA-style navigation. Each route still ships full SSR HTML (crawler-safe), BUT:

- Per-page meta needs to be re-rendered on each navigation (Astro handles by default, verify with DevTools).
- Don't apply `transition:persist` to anything in `<head>` — meta will get stuck.
- Test with `User-Agent: Googlebot` to confirm initial HTML still has full meta.

---

## Phase 4 — Nuxt 3 patterns

### 4.1 Use `useSeoMeta` (typed, XSS-safe)

```vue
<script setup lang="ts">
const route = useRoute()
const { data: post } = await useFetch(`/api/posts/${route.params.slug}`)

useSeoMeta({
  title: () => post.value?.title,
  description: () => post.value?.excerpt,
  ogTitle: () => post.value?.title,
  ogDescription: () => post.value?.excerpt,
  ogType: 'article',
  ogImage: () => post.value?.image,
  ogUrl: () => `https://example.com/blog/${route.params.slug}`,
  twitterCard: 'summary_large_image',
  articlePublishedTime: () => post.value?.publishedAt,
  articleModifiedTime: () => post.value?.updatedAt,
  robots: 'index, follow',
})

useHead({
  link: [{ rel: 'canonical', href: () => `https://example.com/blog/${route.params.slug}` }],
})
</script>
```

**Pitfalls**:
- Keys are camelCase (`ogImage`, not `og:image`).
- Reactive values MUST be functions or refs (`title: () => ...`), not `.value`.
- Don't mix `useHead({ meta: [...] })` and `useSeoMeta` for the same tag — last write wins, debugging is painful.
- Use `useServerSeoMeta` if metadata never needs to be reactive after hydration (smaller client bundle).

### 4.2 Root defaults via `nuxt.config.ts` + app.head

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxtjs/seo'],
  site: {
    url: 'https://example.com',
    name: 'Site Name',
    description: 'Default site description.',
    defaultLocale: 'en',
  },
  app: {
    head: {
      htmlAttrs: { lang: 'en' },
      meta: [
        { charset: 'utf-8' },
        { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      ],
      link: [{ rel: 'icon', href: '/favicon.ico' }],
    },
  },
  routeRules: {
    '/': { prerender: true },
    '/blog/**': { isr: 3600 },              // SEO-friendly hybrid
    '/admin/**': { ssr: false, robots: false },
    '/api/**': { robots: false },
  },
  nitro: {
    prerender: { routes: ['/sitemap.xml', '/robots.txt'] },
  },
})
```

### 4.3 `@nuxtjs/seo` meta-module

Single install bundles:
- `@nuxtjs/sitemap` — auto from routes + crawl
- `@nuxtjs/robots` — programmatic robots
- `nuxt-schema-org` — typed JSON-LD components
- `nuxt-og-image` — Vue-template OG images
- `nuxt-link-checker` — broken link detection
- `nuxt-seo-utils` — sane defaults (canonical, etc.)

```bash
npx nuxi module add @nuxtjs/seo
```

### 4.4 Schema.org via `nuxt-schema-org`

```vue
<script setup lang="ts">
const route = useRoute()
const { data: post } = await useFetch(`/api/posts/${route.params.slug}`)

useSchemaOrg([
  defineArticle({
    headline: () => post.value?.title,
    description: () => post.value?.excerpt,
    image: () => post.value?.image,
    datePublished: () => post.value?.publishedAt,
    dateModified: () => post.value?.updatedAt,
    author: [{
      name: () => post.value?.author.name,
      url: () => `https://example.com/authors/${post.value?.author.slug}`,
      jobTitle: () => post.value?.author.jobTitle,
      sameAs: () => post.value?.author.social,
    }],
  }),
  defineBreadcrumb({
    itemListElement: [
      { name: 'Home', item: '/' },
      { name: 'Blog', item: '/blog' },
      { name: () => post.value?.title },
    ],
  }),
])
</script>
```

In root layout, define site-wide:

```vue
<script setup lang="ts">
useSchemaOrg([
  defineOrganization({
    name: 'Site Name',
    logo: '/logo.png',
    sameAs: ['https://twitter.com/site', 'https://linkedin.com/company/site'],
  }),
  defineWebSite({
    name: 'Site Name',
    potentialAction: {
      '@type': 'SearchAction',
      target: 'https://example.com/search?q={search_term_string}',
      'query-input': 'required name=search_term_string',
    },
  }),
])
</script>
```

### 4.5 OG images via `nuxt-og-image`

```vue
<script setup lang="ts">
defineOgImage({
  component: 'BlogPostOG',
  props: { title: post.value?.title, author: post.value?.author.name },
})
</script>
```

```vue
<!-- components/OgImage/BlogPostOG.vue -->
<template>
  <div class="w-full h-full bg-zinc-900 text-white flex items-center justify-center p-20 text-6xl">
    {{ title }}
  </div>
</template>
```

### 4.6 Nuxt images via `@nuxt/image`

```vue
<!-- LCP -->
<NuxtImg src="/hero.jpg" alt="Hero" width="1920" height="1080"
  preload sizes="100vw" format="avif,webp" densities="x1 x2" />

<!-- Below fold -->
<NuxtPicture src="/feature.jpg" alt="Feature" width="800" height="600"
  loading="lazy" sizes="(max-width: 768px) 100vw, 50vw" />
```

### 4.7 Rendering strategy

| Use case | `routeRules` | Why |
|---|---|---|
| Marketing pages | `prerender: true` | SSG, fastest TTFB |
| Blog/articles | `isr: 3600` | Static + scheduled refresh |
| Product catalog | `swr: true` | Stale-while-revalidate |
| User dashboard | `ssr: true` (default) | Personal data |
| Admin | `ssr: false, robots: false` | Client-only, blocked from crawl |

**`ssr: false` (full SPA) is SEO-fatal** — crawlers without JS execution see an empty shell. AI crawlers (ChatGPT/Perplexity bots) rarely execute JS. Only acceptable behind auth.

---

## Phase 5 — Score (same rubric, framework-agnostic)

Apply the 100-point rubric (`../seo-shared/audit-rubric.md`) per page.

---

## Phase 6 — Validate

```bash
# Status + canonical + JSON-LD on key URLs
./scripts/seo-recrawl.sh urls.txt

# Lighthouse mobile, throttled
npx lighthouse https://site.com/ --view --preset=mobile --throttling-method=simulate

# Validate structured data
npx structured-data-testing-tool https://site.com/blog/foo

# CrUX (real users)
curl -X POST "https://chromeuxreport.googleapis.com/v1/records:queryRecord?key=$KEY" -d '...'

# Astro: verify sitemap chunks
curl -s https://site.com/sitemap-index.xml
curl -s https://site.com/sitemap-0.xml | xmllint --noout -

# Nuxt: verify @nuxtjs/sitemap output
curl -s https://site.com/sitemap.xml
```

---

## Anti-patterns specific to Astro & Nuxt

### Astro
| ❌ Don't | ✅ Do |
|---|---|
| Forget `site:` in `astro.config` | Set it — sitemap and `Astro.site` depend on it |
| `<img>` in `.astro` files | `<Image>` from `astro:assets` |
| Build with `output: 'static'` then expect dynamic OG | Use `output: 'hybrid'` or pre-render OG |
| Mix `trailingSlash` settings | Pick one in config; canonical follows |
| Multiple JSON-LD blocks for same entity | Combine into a `@graph` array |
| `transition:persist` on `<head>` content | Never |

### Nuxt
| ❌ Don't | ✅ Do |
|---|---|
| `ssr: false` for indexable content | `ssr: true` or `prerender: true` |
| `useSeoMeta({ title: post.title })` | `useSeoMeta({ title: () => post.value?.title })` |
| Mix `useHead` and `useSeoMeta` for same tag | Pick one |
| Forget `nitro.prerender.routes` for `/sitemap.xml` `/robots.txt` | Add them |
| `<img>` in `.vue` files | `<NuxtImg>` |
| Hand-roll JSON-LD strings | `useSchemaOrg([defineArticle(...)])` |

---

## Quick checklist (paste into PR)

```
General:
[ ] site URL set (Astro: site config; Nuxt: site.url config)
[ ] Every page has unique title (30–65) + description (70–160)
[ ] Every page has canonical
[ ] Sitemap reachable + valid XML
[ ] Robots references sitemap, has explicit AI crawler policy

Astro:
[ ] BaseLayout used everywhere; canonical via Astro.site
[ ] Content collections schema enforces SEO frontmatter
[ ] @astrojs/sitemap configured with filter
[ ] src/pages/robots.txt.ts exists
[ ] All images use <Image>/<Picture> from astro:assets
[ ] LCP image: loading="eager"

Nuxt:
[ ] @nuxtjs/seo module installed
[ ] useSeoMeta on every page (with reactive functions)
[ ] useSchemaOrg in root layout (Organization + WebSite)
[ ] routeRules: prerender or isr for indexable; ssr:false only behind auth
[ ] All images use <NuxtImg>/<NuxtPicture>
[ ] LCP image: preload prop
[ ] nitro.prerender.routes includes /sitemap.xml /robots.txt

Performance (both):
[ ] Lighthouse mobile: LCP ≤2.5s, INP ≤200ms, CLS ≤0.1
[ ] Schema validates: validator.schema.org + Rich Results Test
```

## Reference files

- `reference/astro.md` — extra Astro patterns (i18n, trailingSlash, hybrid output)
- `reference/nuxt.md` — extra Nuxt patterns (i18n, hybrid rendering, edge)
- `reference/checklist.md` — long-form combined checklist with explanations
- `../seo-shared/core-web-vitals.md`
- `../seo-shared/structured-data.md`
- `../seo-shared/geo-aeo.md`
- `../seo-shared/audit-rubric.md`

## Sources

- Astro sitemap: https://docs.astro.build/en/guides/integrations-guide/sitemap/
- Astro images: https://docs.astro.build/en/guides/images/
- Astro view transitions: https://docs.astro.build/en/guides/view-transitions/
- astro-seo (community): https://github.com/jonasmerlin/astro-seo
- Nuxt SEO meta: https://nuxt.com/docs/getting-started/seo-meta
- Nuxt SEO module: https://nuxtseo.com
- Nuxt image: https://image.nuxt.com
- Schema.org validator: https://validator.schema.org/
- Rich Results Test: https://search.google.com/test/rich-results
