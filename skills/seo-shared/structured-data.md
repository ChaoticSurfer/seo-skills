# Structured Data — 2026 Reference

JSON-LD only (microdata/RDFa are legacy). Schema is now primarily an **AI/entity trust signal**, not a SERP-display trigger — many rich-result types were deprecated.

## Status by type (2026)

| Type | Status | Use |
|---|---|---|
| **Article / BlogPosting / NewsArticle** | ✅ Active | Every article. Required for Top Stories. |
| **Product** | ✅ Active | Every product page. Drives shopping carousels. |
| **Organization** | ✅ Active | Site-wide. Knowledge Panel signal. |
| **WebSite** + `SearchAction` | ✅ Active | Sitelinks searchbox. One per site. |
| **BreadcrumbList** | ✅ Active | Every non-root page. |
| **Review / AggregateRating** | ✅ Active | Strict policies — must be on the reviewed item. |
| **Event** | ✅ Active | Real events only. |
| **Recipe** | ✅ Active | Niche-specific. |
| **VideoObject** | ✅ Active | All embedded video. |
| **LocalBusiness** | ✅ Active | Critical for local SEO. |
| **FAQPage** | ⚠️ Restricted | Rich results limited to gov + well-known health (Aug 2023). Still valid as AI signal — DO NOT fabricate FAQs. |
| **HowTo** | ❌ Dead | Removed entirely 2023–2024. Don't ship. |
| **QAPage** | ⚠️ Restricted | Same as FAQ. |

## Required validation

1. **Google Rich Results Test**: https://search.google.com/test/rich-results
2. **Schema.org validator**: https://validator.schema.org/
3. **CLI**: `npx structured-data-testing-tool https://site.com/page`

Both warnings AND errors must be resolved.

## Article template (most common)

```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Title (max 110 chars)",
  "description": "Excerpt",
  "image": [
    "https://example.com/og-1x1.jpg",
    "https://example.com/og-4x3.jpg",
    "https://example.com/og-16x9.jpg"
  ],
  "datePublished": "2026-05-03T08:00:00+00:00",
  "dateModified": "2026-05-03T09:30:00+00:00",
  "author": [{
    "@type": "Person",
    "name": "Author Name",
    "url": "https://example.com/authors/jane",
    "jobTitle": "Senior Engineer",
    "sameAs": ["https://twitter.com/jane", "https://linkedin.com/in/jane"]
  }],
  "publisher": {
    "@type": "Organization",
    "name": "Site Name",
    "logo": { "@type": "ImageObject", "url": "https://example.com/logo.png" }
  },
  "mainEntityOfPage": "https://example.com/blog/slug"
}
```

Author `sameAs` and `jobTitle` are **E-E-A-T signals** — required in 2026.

## Organization (site-wide, in root layout)

```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Brand",
  "url": "https://example.com",
  "logo": "https://example.com/logo.png",
  "sameAs": [
    "https://twitter.com/brand",
    "https://linkedin.com/company/brand",
    "https://github.com/brand"
  ],
  "contactPoint": {
    "@type": "ContactPoint",
    "telephone": "+1-555-0100",
    "contactType": "customer service"
  }
}
```

## WebSite + SearchAction (sitelinks searchbox)

```json
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "url": "https://example.com",
  "potentialAction": {
    "@type": "SearchAction",
    "target": "https://example.com/search?q={search_term_string}",
    "query-input": "required name=search_term_string"
  }
}
```

## BreadcrumbList

```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://example.com" },
    { "@type": "ListItem", "position": 2, "name": "Blog", "item": "https://example.com/blog" },
    { "@type": "ListItem", "position": 3, "name": "Post Title" }
  ]
}
```

## Product

```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Product Name",
  "image": ["https://example.com/p1.jpg"],
  "description": "Product description",
  "sku": "SKU-123",
  "brand": { "@type": "Brand", "name": "Brand" },
  "offers": {
    "@type": "Offer",
    "url": "https://example.com/p/sku-123",
    "priceCurrency": "USD",
    "price": "29.99",
    "availability": "https://schema.org/InStock"
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.6",
    "reviewCount": "248"
  }
}
```

## XSS guard for inline JSON-LD

When inlining via `dangerouslySetInnerHTML` (React) or `set:html` (Astro), escape `<`:

```ts
const safe = JSON.stringify(jsonLd).replace(/</g, '\\u003c')
```

## Anti-patterns

- ❌ FAQPage on marketing pages hoping for rich results — won't show.
- ❌ HowTo schema — deprecated, ignored.
- ❌ Schema for content not visible on the page — Google penalizes this.
- ❌ AggregateRating on category pages — must be per-item.
- ❌ Multiple conflicting `@type: Organization` blocks — one per site.
