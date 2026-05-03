# Audit Rubric — Evidence, Confidence, Scoring

Every audit finding follows this structured format. Don't ship a finding without evidence.

## Finding format

```
[ID] Issue title
  Impact:     High | Medium | Low
  Confidence: Confirmed | Likely | Hypothesis
  Evidence:   <specific URL, line, screenshot, or metric value>
  Fix:        <code snippet or specific action>
  Effort:     S (<1h) | M (1–4h) | L (>4h)
  Priority:   P0 | P1 | P2  (P0 = ship within 24h)
```

## Confidence tiers

- **Confirmed** — directly observed in HTML/network/CrUX. E.g. "GET /products → 500" or "LCP=4.2s in Lighthouse".
- **Likely** — strong inference from observable signals. E.g. "Title 89 chars likely truncated in SERP".
- **Hypothesis** — pattern match without direct verification. E.g. "Probably keyword cannibalization based on overlapping titles".

Never present a Hypothesis as Confirmed.

## 100-point scorecard (per page)

| Dimension | Points | What's measured |
|---|---|---|
| **Metadata** | 20 | Title (5), description (5), canonical (3), OG (4), Twitter (3) |
| **Structured data** | 15 | Required schema present + valid (10), entity completeness (5) |
| **Content** | 25 | Word count vs competitors (5), heading hierarchy (5), keyword presence (5), internal links (5), original data/E-E-A-T (5) |
| **E-E-A-T** | 20 | Author identity + credentials (5), pubDate (3), modDate (3), citations (5), about/contact (4) |
| **Strategy** | 10 | Keyword targeting clear (5), no cannibalization (5) |
| **Technical** | 10 | Core Web Vitals pass (5), mobile parity (3), crawlable (2) |

Bands: **90–100** ship · **75–89** strong · **60–74** action plan · **<60** rework.

### Per-element bounds (auto-flag if outside)

| Element | Min | Max | Notes |
|---|---|---|---|
| Title | 30 | 65 chars | Pixel-truncation around 580px ≈ 65 chars |
| Meta description | 70 | 160 chars | Mobile ~120, desktop ~160 |
| H1 count | 1 | 1 | Multiple = auto-fail |
| H1 length | 20 | 70 chars | |
| Word count (article) | 800 | — | <800 = thin content flag |
| Internal links per page | 3 | 100 | <3 = orphan; >100 = noise |
| Image alt | 5 | 125 chars | Empty = a11y + AI fail |
| Canonical | 1 | 1 | Multiple/chained = auto-fail |

## Issue prioritization

```
Priority Score = (Impact_weight × Page_traffic) / Effort_hours
Impact_weight: High=10, Medium=3, Low=1
Effort_hours:  S=1, M=2, L=4
```

Top 10 by Priority Score = action plan. Everything else = backlog.

## Output structure (final report)

```
═══════════════════════════════════════════════════════════════════
  SEO AUDIT — site.com — 2026-05-03
═══════════════════════════════════════════════════════════════════

EXECUTIVE SUMMARY
  Pages crawled:      127
  Pages indexable:    115 (90%)
  Avg page score:     68 / 100
  Critical issues:    3 (P0)
  High priority:      12 (P1)
  Estimated traffic loss: ~450 clicks/mo

═══════════════════════════════════════════════════════════════════

P0 — CRITICAL (fix within 24h)

  [P0-1] /products returns 500
    Impact:     High        Confidence: Confirmed
    Evidence:   curl -I https://site.com/products → HTTP/1.1 500
    Fix:        Check server logs; redeploy; add health check
    Effort:     S
    
  [P0-2] Hero LCP = 4.2s on /
    Impact:     High        Confidence: Confirmed
    Evidence:   Lighthouse mobile: LCP 4.2s (target ≤2.5s)
    Fix:        <Image src="/hero.jpg" priority sizes="100vw" />
    Effort:     S

P1 — HIGH PRIORITY (fix within 2 weeks)
  [P1-1] ...

═══════════════════════════════════════════════════════════════════

ACTION PLAN
  Week 1: P0-1, P0-2, P0-3 → re-audit
  Week 2: P1-1, P1-2, P1-3
  Week 3: P1-4, P1-5
  Week 4: Validation crawl + ranking check

EXPECTED IMPACT (if all P0+P1 shipped)
  CWV pass rate: 47% → 92%
  Traffic estimate (8wk): +280 clicks/mo (CrUX-based projection)
═══════════════════════════════════════════════════════════════════
```

## What you do NOT do

- Don't suggest fixes without showing current state (Evidence first)
- Don't list 50 issues — surface top 10 by Priority Score
- Don't claim ranking gains; claim *visibility* and *technical* gains
- Don't conflate Confirmed with Hypothesis
