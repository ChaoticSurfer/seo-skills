---
name: seo-nextjs
description: SEO audit and optimization for Next.js 15 App Router (TypeScript). Covers technical SEO, metadata API, structured data, Core Web Vitals 2026, and GEO/AEO (AI search). Trigger on requests to "audit SEO", "improve rankings", "fix Core Web Vitals", "add structured data", or any Next.js SEO question.
---

# Next.js SEO Skill (App Router, TypeScript, 2026)

> **Trigger:** Next.js project + any SEO/performance/visibility question.
> **Output:** Evidence-backed audit + per-page fixes + validation plan.
> **Stance:** Data-driven. Cite specific URLs, line numbers, metric values. Confidence-labeled findings.

---

## Skill contract

| | |
|---|---|
| **Inputs** | Domain, optional GSC/CrUX/GA4 access, target keywords, scope (full site / page list) |
| **Phases** | 1. Detect → 2. Audit → 3. Score → 4. Fix → 5. Validate |
| **Outputs** | Findings with evidence + confidence, top-10 prioritized action plan, per-page code patches |
| **Reference files** | `reference/checklist.md`, `../seo-shared/core-web-vitals.md`, `../seo-shared/structured-data.md`, `../seo-shared/geo-aeo.md`, `../seo-shared/audit-rubric.md`, `../seo-shared/tools.md`, `../seo-shared/platforms.md` |
| **Optional MCPs** | Semrush MCP (`mcp__semrush__*`). If connected, run the augmented audit module in `../seo-shared/semrush-mcp.md`. The skill works fully without it — Semrush adds competitor & keyword-volume evidence on top. |

Read shared references when you need depth on CWV thresholds, JSON-LD templates, GEO patterns, or scoring rubric. Don't recite — link.

---

## Phase 1 — Detect

Before recommending anything, verify:

```bash
# Confirm App Router (not Pages)
test -d app && echo "App Router" || echo "Pages Router (skip this skill, ask user)"

# Confirm Next.js 15+
node -e "console.log(require('./package.json').dependencies.next)"

# Confirm TS
test -f tsconfig.json && echo "TS" || echo "JS"

# Find existing metadata, sitemap, robots
fd -e ts -e tsx 'page|layout|sitemap|robots|opengraph-image' app/
```

If Pages Router → tell user this skill is App Router-only.
If Next < 15 → flag known differences (params not async, etc.) but proceed.

---

## Phase 2 — Audit

> **Optional**: if Semrush MCP is connected, before Phase 2.1 run the full Semrush audit module in `../seo-shared/semrush-mcp.md` to seed competitor data, keyword inventory, and position-tracking evidence. Skip if unavailable — the rest of the audit stands alone.

### 2.1 Crawl

```bash
# Status codes + redirect chains for top 50 URLs
xargs -I{} curl -s -o /dev/null -w "%{http_code} %{redirect_url} {}\n" < urls.txt

# Sitemap reachable + valid
curl -s https://site.com/sitemap.xml | xmllint --noout - && echo "OK"

# robots.txt
curl -s https://site.com/robots.txt
```

Flag: 5xx, 4xx on linked pages, redirect chains > 1, non-canonical URLs in sitemap, `Disallow` on important paths, missing `Sitemap:` directive.

### 2.2 Per-page extraction (script-driven, no LLM guessing)

> Optional: if Semrush MCP is connected, also pull `mcp__semrush__url_organic_keywords` per page — see `../seo-shared/semrush-mcp.md`.


For each URL, parse the **rendered HTML** (use `curl` with `User-Agent: Googlebot` and a JS-renderer if needed):

- `<title>` (length 30–65)
- `<meta name="description">` (length 70–160)
- `<link rel="canonical">` count and target
- `<h1>` count (must be 1) and length
- All headings hierarchy (no skips H2→H4)
- `<script type="application/ld+json">` blocks → parse + validate
- `<meta property="og:*">` completeness
- `<img>` count, missing alt count, missing dimensions count

Then ask the LLM to judge: keyword targeting, content depth, E-E-A-T signals.

### 2.3 Metadata audit (Next.js 15 specifics)

**Critical pitfalls to detect:**

```
[ ] generateMetadata is in a Client Component → silently dropped
    → Move to Server Component or static `export const metadata`

[ ] No metadataBase set in root layout
    → OG/Twitter image URLs are relative → broken in crawlers
    → Add: metadataBase: new URL('https://site.com')

[ ] Next 15: params destructured without await
    → Runtime error. Must be: const { slug } = await params

[ ] notFound() called after metadata return
    → Metadata still ships. Call notFound() before returning metadata.

[ ] Same metadata across dynamic pages
    → Duplicate-title issue. generateMetadata must vary per param.
```

**Required pattern (root layout):**

```tsx
// app/layout.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  metadataBase: new URL('https://site.com'),  // CRITICAL
  title: { default: 'Site Name', template: '%s | Site Name' },
  description: 'Default site description (70–160 chars).',
  openGraph: {
    type: 'website',
    siteName: 'Site Name',
    images: ['/og-default.png'],
  },
  twitter: { card: 'summary_large_image' },
  robots: { index: true, follow: true },
  alternates: { canonical: '/' },
}
```

**Required pattern (dynamic page):**

```tsx
// app/blog/[slug]/page.tsx
export async function generateMetadata(
  { params }: { params: Promise<{ slug: string }> }
): Promise<Metadata> {
  const { slug } = await params
  const post = await getPost(slug)
  if (!post) return {}  // 404 handled separately
  
  return {
    title: post.title,                          // template adds " | Site Name"
    description: post.excerpt,
    alternates: { canonical: `/blog/${slug}` },
    openGraph: {
      type: 'article',
      title: post.title,
      description: post.excerpt,
      publishedTime: post.publishedAt,
      modifiedTime: post.updatedAt,
      authors: [post.author.name],
      images: [post.image],
    },
    twitter: { title: post.title, description: post.excerpt, images: [post.image] },
  }
}
```

### 2.4 Sitemap + robots audit

**Required:** `app/sitemap.ts` + `app/robots.ts` (file conventions, NOT `public/`).

```tsx
// app/sitemap.ts
import type { MetadataRoute } from 'next'

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const posts = await getAllPosts()
  return [
    { url: 'https://site.com', lastModified: new Date(), changeFrequency: 'daily', priority: 1 },
    ...posts.map(p => ({
      url: `https://site.com/blog/${p.slug}`,
      lastModified: p.updatedAt,
      changeFrequency: 'weekly' as const,
      priority: 0.8,
    })),
  ]
}
```

For >50k URLs, split via `generateSitemaps`:

```tsx
export async function generateSitemaps() {
  const total = await getPostCount()
  return Array.from({ length: Math.ceil(total / 50000) }, (_, id) => ({ id }))
}

export default async function sitemap({ id }: { id: number }): Promise<MetadataRoute.Sitemap> {
  const posts = await getPostsPage(id, 50000)
  return posts.map(p => ({ url: `https://site.com/blog/${p.slug}`, lastModified: p.updatedAt }))
}
```

```tsx
// app/robots.ts
import type { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      { userAgent: '*', allow: '/', disallow: ['/api/', '/admin/'] },
      // AI crawlers — see ../seo-shared/geo-aeo.md for full list and policy
      { userAgent: 'GPTBot', allow: '/' },
      { userAgent: 'ClaudeBot', allow: '/' },
      { userAgent: 'PerplexityBot', allow: '/' },
      { userAgent: 'Google-Extended', allow: '/' },
    ],
    sitemap: 'https://site.com/sitemap.xml',
    host: 'https://site.com',
  }
}
```

### 2.5 Dynamic OG images

Use `app/<route>/opengraph-image.tsx` (file convention). Default size 1200×630.

```tsx
// app/blog/[slug]/opengraph-image.tsx
import { ImageResponse } from 'next/og'

export const runtime = 'edge'
export const size = { width: 1200, height: 630 }
export const contentType = 'image/png'

export default async function Image({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug)
  return new ImageResponse(
    (
      <div style={{ display: 'flex', width: '100%', height: '100%', background: '#0b0b0b', color: 'white', padding: 80, fontSize: 64 }}>
        {post.title}
      </div>
    ),
    { ...size }
  )
}
```

Plus `twitter-image.tsx` for the Twitter-card variant (or symlink to opengraph-image).

### 2.6 Structured data (JSON-LD)

Inline in the Server Component returning the page. **XSS-escape `<`**:

```tsx
// app/blog/[slug]/page.tsx
import { JsonLd } from '@/components/JsonLd'  // tiny wrapper

export default async function Page({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params
  const post = await getPost(slug)
  
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: post.title,
    description: post.excerpt,
    image: [post.image],
    datePublished: post.publishedAt,
    dateModified: post.updatedAt,
    author: [{
      '@type': 'Person',
      name: post.author.name,
      url: `https://site.com/authors/${post.author.slug}`,
      jobTitle: post.author.role,
      sameAs: post.author.social,
    }],
    publisher: {
      '@type': 'Organization',
      name: 'Site Name',
      logo: { '@type': 'ImageObject', url: 'https://site.com/logo.png' },
    },
    mainEntityOfPage: `https://site.com/blog/${slug}`,
  }
  
  return (
    <>
      <JsonLd data={jsonLd} />
      <article>{/* … */}</article>
    </>
  )
}
```

```tsx
// components/JsonLd.tsx
export function JsonLd({ data }: { data: object }) {
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{
        __html: JSON.stringify(data).replace(/</g, '\\u003c'),
      }}
    />
  )
}
```

See `../seo-shared/structured-data.md` for type-by-type templates and 2026 deprecations (HowTo dead, FAQ restricted).

### 2.7 Core Web Vitals

See `../seo-shared/core-web-vitals.md` for thresholds and root-cause taxonomy.

**Next.js-specific fixes:**

```tsx
// LCP: hero image
import Image from 'next/image'
<Image src="/hero.jpg" alt="..." width={1920} height={1080}
       priority sizes="100vw"
       placeholder="blur" blurDataURL="data:image/..." />

// LCP: above-the-fold data — keep server-side, no client fetch
export default async function Page() {
  const data = await fetchHero()  // server, blocks initial HTML — that's correct
  return <Hero data={data} />
}

// INP: heavy click handler
'use client'
import { useTransition } from 'react'
const [pending, start] = useTransition()
const onClick = () => start(() => router.push('/heavy'))

// CLS: explicit image dimensions or aspect-ratio
<div style={{ aspectRatio: '16/9' }}>
  <Image src={src} alt="" fill sizes="(max-width: 768px) 100vw, 50vw" />
</div>

// Font: next/font + display swap (kills layout shift)
import { Inter } from 'next/font/google'
const inter = Inter({ subsets: ['latin'], display: 'swap' })

// Render strategy
export const revalidate = 3600  // ISR — best for content pages
// or: export const dynamic = 'force-static'  // pure SSG
// or: export const dynamic = 'force-dynamic'  // only when truly needed
```

### 2.8 Rendering strategy

| Use case | Choice | Why |
|---|---|---|
| Marketing landing pages | SSG (default) | Fastest TTFB, fully cacheable |
| Blog/articles | ISR (`revalidate = 3600`) | Static + scheduled refresh |
| Product catalog | ISR + `generateStaticParams` | Pre-render top, ISR rest |
| User dashboard | SSR (`force-dynamic`) | Personal data |
| Search results | SSR | Query-driven |

`force-dynamic` is the slowest for SEO and CWV — use only when needed.

---

## Phase 3 — Score

Apply the 100-point rubric (`../seo-shared/audit-rubric.md`) per page. Report distribution:

```
Page score distribution (115 pages):
  90–100  ████████░░░░░░░░░░░░  22 pages (19%)
  75–89   █████████████░░░░░░░  41 pages (36%)
  60–74   ██████████░░░░░░░░░░  31 pages (27%)
  <60     ███████░░░░░░░░░░░░░  21 pages (18%)
```

The bottom 18% drives most of the recoverable traffic.

---

## Phase 4 — Fix

Surface findings using the Evidence/Confidence rubric. Prioritize via `Priority Score = (Impact × Traffic) / Effort`. Generate code patches, not advice.

Pattern:

```
[P0-2] /  Hero image causing LCP 4.2s
  Impact: High        Confidence: Confirmed
  Evidence: Lighthouse mobile run @ 2026-05-03 → LCP 4.2s; CrUX p75 LCP 3.8s
  Fix:
    --- a/app/page.tsx
    +++ b/app/page.tsx
    @@
    -<img src="/hero.jpg" alt="Hero" />
    +<Image src="/hero.jpg" alt="Hero" width={1920} height={1080}
    +       priority sizes="100vw"
    +       placeholder="blur" blurDataURL="..." />
  Effort: S    Priority: P0
  Expected: LCP 4.2s → ~1.6s (CrUX projection from similar fixes)
```

---

## Phase 5 — Validate

After deploy:

```bash
# 1. Re-crawl key URLs — 200 + correct canonical + valid JSON-LD
./scripts/seo-recrawl.sh urls.txt

# 2. Lighthouse re-run (mobile, throttled)
npx lighthouse https://site.com/ --view --preset=mobile --throttling-method=simulate

# 3. Validate structured data
npx structured-data-testing-tool https://site.com/blog/foo

# 4. CrUX query (real-user, takes 28 days to update fully)
curl -X POST "https://chromeuxreport.googleapis.com/v1/records:queryRecord?key=$KEY" -d '...'

# 5. GSC: Coverage, Page experience, Performance — check daily for 2 weeks
```

> Optional: if Semrush MCP is connected, run the validate-phase steps in `../seo-shared/semrush-mcp.md` (position-tracking + per-URL keyword re-pull, ~2 weeks post-deploy).

Set monitoring:
- Weekly Lighthouse run on top 20 URLs (alert if CWV regresses)
- GSC coverage delta > 5% triggers review
- Schema validation on every deploy (CI check)

---

## Anti-patterns specific to Next.js

| ❌ Don't | ✅ Do |
|---|---|
| Metadata in `'use client'` file | Server Component or `export const metadata` |
| Forget `metadataBase` | Set it once in root layout |
| `<img>` for hero | `<Image priority sizes="100vw" />` |
| `force-dynamic` for marketing | SSG/ISR |
| Schema in `<Head>` from `next/head` | App Router uses inline `<script>` JSON-LD |
| Multi-page same metadata | `generateMetadata` returns per-param values |
| Robots/sitemap in `public/` | Use `app/robots.ts` and `app/sitemap.ts` (TypeScript, dynamic) |
| `params` synchronous in Next 15 | `params: Promise<...>` then `await params` |

---

## Quick checklist (paste into PR description)

```
[ ] metadataBase set in root layout
[ ] Every page has unique title (30–65 chars) + description (70–160)
[ ] Every page has alternates.canonical
[ ] app/sitemap.ts exists and includes all indexable URLs
[ ] app/robots.ts exists with explicit AI crawler policy
[ ] OG image: opengraph-image.tsx OR /og-default.png referenced
[ ] JSON-LD: Article on posts, Product on PDPs, Organization in root
[ ] Hero <Image priority sizes="100vw" /> (kills LCP)
[ ] All images have alt + width/height (kills CLS + a11y)
[ ] No client-side data fetch for above-the-fold content
[ ] Lighthouse mobile: LCP ≤2.5s, INP ≤200ms, CLS ≤0.1
[ ] Schema validates: validator.schema.org + Rich Results Test
[ ] No /api/* or /admin/* in sitemap
```

## Reference files

- `reference/checklist.md` — the long form of the checklist above, with explanations
- `../seo-shared/core-web-vitals.md` — thresholds + root causes
- `../seo-shared/structured-data.md` — JSON-LD templates (2026)
- `../seo-shared/geo-aeo.md` — AI Overview / LLM citation tactics
- `../seo-shared/audit-rubric.md` — 100-point scorecard + finding format
- `../seo-shared/tools.md` — free vs paid tool catalogue, what each one gives you, recommended stack by budget
- `../seo-shared/platforms.md` — every external platform that needs registration, with verification method + setup time + recommended priority order

## Sources

- Next.js generateMetadata: https://nextjs.org/docs/app/api-reference/functions/generate-metadata
- Next.js sitemap convention: https://nextjs.org/docs/app/api-reference/file-conventions/metadata/sitemap
- Next.js robots convention: https://nextjs.org/docs/app/api-reference/file-conventions/metadata/robots
- Next.js next/og: https://nextjs.org/docs/app/api-reference/functions/image-response
- Next.js JSON-LD guide: https://nextjs.org/docs/app/guides/json-ld
- Core Web Vitals: https://developers.google.com/search/docs/appearance/core-web-vitals
- Schema.org validator: https://validator.schema.org/
- Rich Results Test: https://search.google.com/test/rich-results
