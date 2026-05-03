# Core Web Vitals — 2026 Reference

p75 of real-user (CrUX) data. INP replaced FID on **2024-03-12**. As of **March 2026 core update**, Google scores CWV holistically (composite of LCP/INP/CLS) — passing two and failing one is now penalized harder than before.

## Thresholds

| Metric | Good | Needs Improvement | Poor | Notes |
|---|---|---|---|---|
| **LCP** | ≤ 2.5s | 2.5–4.0s | > 4.0s | Largest Contentful Paint |
| **INP** | ≤ 200ms | 201–500ms | > 500ms | Replaces FID. **~43% of sites fail this.** |
| **CLS** | ≤ 0.1 | 0.1–0.25 | > 0.25 | Cumulative Layout Shift |
| TTFB | ≤ 800ms | 800–1800ms | > 1800ms | Diagnostic, not ranking |

Only ~47% of sites pass all three. INP is the highest-leverage metric in 2026.

## LCP root causes (frequency order)

1. **Unoptimized hero image** (~60% of LCP failures) — `priority` + AVIF/WebP
2. **Slow TTFB** — caching, ISR, edge runtime
3. **Render-blocking JS/CSS** in `<head>` — defer/async or move to body
4. **Web fonts blocking render** — `display: swap`, preload critical subset
5. **Client-side data fetch above the fold** — move to server

## INP root causes

1. **Heavy event handlers** — `useTransition` (React) / debounce
2. **Long task on main thread** — `scheduler.yield()` or `setTimeout` chunking
3. **Hydration cost** — minimize client components / use islands
4. **Third-party scripts** (analytics, chat) — lazy-load with `requestIdleCallback`

## CLS root causes

1. **Images without explicit dimensions** — `width`/`height` or `aspect-ratio`
2. **Web fonts swapping** — preload + `size-adjust` CSS metric
3. **Dynamic content injection above existing content** — reserve space
4. **Ads / embeds without slots** — fixed dimensions

## Profiling commands

```bash
# Lighthouse CLI
npx lighthouse https://site.com/page --view --preset=desktop
npx lighthouse https://site.com/page --view --preset=mobile

# Real user data (CrUX API — needs API key)
curl -X POST "https://chromeuxreport.googleapis.com/v1/records:queryRecord?key=$KEY" \
  -d '{"url":"https://site.com/","metrics":["largest_contentful_paint","interaction_to_next_paint","cumulative_layout_shift"]}'

# Bundle analysis
npx @next/bundle-analyzer  # Next.js
ANALYZE=true npm run build # Generic
```

## Fix priority (highest ROI first)

1. LCP image: `priority` + AVIF + correct `sizes`
2. Server-side data above the fold (kill client fetches)
3. Defer non-critical JS (analytics, widgets)
4. Reserve image/embed space (kills CLS)
5. Code-split heavy components (kills INP)
