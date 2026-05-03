# Semrush MCP — Optional Audit Module

> **Optional, opt-in.** The base SEO skills (`seo-nextjs`, `seo-vue-astro`) work fully without Semrush. This file is a standalone augmentation that runs ONLY when the Semrush MCP is connected.
>
> **Precondition check** (run first):
> ```
> ToolSearch({ query: "semrush", max_results: 30 })
> ```
> If no `mcp__semrush__*` tools appear → skip this entire module, continue with the base skill.

The Semrush MCP exposes Semrush's API as tools callable from Claude Code. When available, it replaces manual GSC scraping, competitor lookups, and keyword guessing during audit.

## Setup

```bash
# 1. Get a Semrush API key (Business plan or higher)
#    Account → API Units → Generate key

# 2. Install / connect the official Semrush MCP server (config in ~/.claude/settings.json or .mcp.json)
{
  "mcpServers": {
    "semrush": {
      "command": "npx",
      "args": ["-y", "@semrush/mcp-server"],
      "env": { "SEMRUSH_API_KEY": "${SEMRUSH_API_KEY}" }
    }
  }
}

# 3. Verify
echo $SEMRUSH_API_KEY  # should be set
```

After connecting, tools appear as `mcp__semrush__*`. Discover with `ToolSearch({ query: "semrush", max_results: 30 })`.

## When to use during audit

| Audit phase | Without Semrush | With Semrush MCP |
|---|---|---|
| **Keyword discovery** | Ask user, guess | `domain_organic_keywords` → real ranked queries |
| **Competitor gap** | Manual search | `domain_vs_domain` → keywords competitor ranks for that we don't |
| **Backlink audit** | Skip | `backlinks_overview` + `referring_domains` |
| **Position tracking** | Manual SERP checks | `position_tracking` |
| **Keyword volume / KD** | "Probably medium" | Actual numbers |
| **Content gap on a page** | Compare two URLs by eye | `url_organic_keywords` for both |
| **Cannibalization** | Heuristic title overlap | Two URLs ranking for same query → confirmed |

## Core tool patterns

> Exact tool names depend on the MCP server build. Discover via `ToolSearch`. Common names below.

### 1. Domain overview — start of every audit

```
mcp__semrush__domain_overview({
  domain: "example.com",
  database: "us"            // country code
})
→ { organic_traffic, organic_keywords, paid_traffic, authority_score }
```

Pulls a snapshot. Use to set the audit baseline.

### 2. Top organic keywords for the domain

```
mcp__semrush__domain_organic_keywords({
  domain: "example.com",
  database: "us",
  limit: 100,
  filter: { position_max: 20 }   // top 20 SERP only
})
→ [{ keyword, position, search_volume, cpc, url, traffic_pct }, ...]
```

This is the **primary input** for Phase 2 audit. Joins keyword → ranking URL → traffic share.

### 3. Top organic keywords for a single URL

```
mcp__semrush__url_organic_keywords({
  url: "https://example.com/blog/post-slug",
  database: "us",
  limit: 50
})
```

Use during per-page review — confirms what each page actually ranks for. Catches cannibalization (same keyword across two URLs).

### 4. Competitor gap

```
mcp__semrush__domain_vs_domain({
  domains: [
    { domain: "example.com", type: "organic" },
    { domain: "competitor.com", type: "organic" }
  ],
  database: "us",
  limit: 200
})
→ keywords competitor ranks for + your position (or none)
```

Drives "missing topics" recommendations in the audit report.

### 5. Keyword research (for new content briefs)

```
mcp__semrush__keyword_overview({
  keyword: "next.js seo audit",
  database: "us"
})
→ { volume, kd_pct, cpc, intent, serp_features, related }

mcp__semrush__keyword_magic({
  keyword: "next.js seo",
  database: "us",
  limit: 100
})
→ long-tail expansion
```

### 6. Backlinks

```
mcp__semrush__backlinks_overview({ target: "example.com", target_type: "root_domain" })
mcp__semrush__referring_domains({ target: "example.com", limit: 50, filter: { authority_score_min: 30 } })
```

Use sparingly — backlinks are slow signals. Pull once per audit cycle.

### 7. Site audit (technical issues)

```
mcp__semrush__site_audit_issues({ project_id: "12345" })
→ list of crawl errors, broken links, redirect chains, etc.
```

Requires a Semrush project pre-configured for the domain. If available, treat output as **Confirmed** evidence in the audit rubric.

### 8. Position tracking

```
mcp__semrush__position_tracking({
  project_id: "12345",
  date: "2026-05-03"
})
```

Use as evidence for ranking-change findings.

## Standalone audit workflow (when Semrush is available)

Run AFTER detecting `mcp__semrush__*` tools. Outputs feed into the base skill's audit + report.

### Step 1 — Baseline (call before the base skill's Phase 2.1 crawl)

```
mcp__semrush__domain_overview({ domain, database: "us" })
mcp__semrush__domain_organic_keywords({
  domain, database: "us", limit: 200,
  filter: { position_max: 20 }
})
mcp__semrush__site_audit_issues({ project_id })       # if project exists
mcp__semrush__domain_vs_domain({                      # competitor gap
  domains: [{ domain: ours, type: "organic" },
            { domain: competitor, type: "organic" }],
  database: "us", limit: 200
})
```

Cache to `.claude/cache/semrush/<tool>_<domain>_<date>.json`. Cap `limit` on every call.

### Step 2 — Per-page (during base skill's Phase 2.2)

For each top page (sort by Semrush `traffic_pct`):

```
mcp__semrush__url_organic_keywords({ url, database: "us", limit: 50 })
```

Cross-check vs on-page H1/title/copy → flag mismatches. Detect cannibalization (same keyword on 2+ URLs from Step 1).

### Step 3 — Validate (2 weeks post-deploy)

```
mcp__semrush__position_tracking({ project_id, date: "<today>" })
mcp__semrush__url_organic_keywords({ url: "<fixed url>", database: "us" })
→ compare to Step 1 baseline
```

---

## Integration with audit phase (legacy notes)

### Phase 2.1 (crawl) — Semrush adds:

```
1. mcp__semrush__domain_overview → baseline
2. mcp__semrush__domain_organic_keywords (top 200, position ≤ 20) → keyword inventory
3. mcp__semrush__site_audit_issues (if project exists) → technical issues
```

Output: a CSV-like table joining `URL × keyword × position × volume × traffic` — the working set for the rest of the audit.

### Phase 2.2 (per-page) — Semrush adds:

For each top page (by traffic):

```
1. mcp__semrush__url_organic_keywords → what this page actually ranks for
2. Cross-check with on-page H1, title, copy → confirm targeting matches reality
3. Detect cannibalization: same keyword appearing for 2+ URLs from step 1
```

### Phase 2.3 (competitive) — Semrush adds:

```
1. mcp__semrush__domain_vs_domain (you vs top 3 competitors)
2. Filter to keywords with volume ≥ 100 and KD ≤ 60
3. Output: "Missing topics" list — sorted by (volume × intent_match)
```

This becomes the content-strategy section of the report.

## Evidence labeling

Tag findings with their Semrush source:

```
[P1-3] /blog/seo-tools is cannibalizing /tools for "best seo tools"
  Impact: High        Confidence: Confirmed
  Evidence: Semrush domain_organic_keywords (2026-05-03):
            "best seo tools" → /blog/seo-tools position 8
            "best seo tools" → /tools position 14
            volume: 2,400/mo, both URLs receiving traffic
  Fix:      Pick canonical (recommend /blog/seo-tools — better topic match);
            301 /tools → /blog/seo-tools OR rewrite /tools for distinct intent
```

Always cite the MCP tool + date pulled. Semrush data is a snapshot.

## API unit cost (be deliberate)

Each call consumes API units. Rough cost (verify in your plan):

| Call | API units |
|---|---|
| `domain_overview` | 10 |
| `domain_organic_keywords` (per row returned) | 10 |
| `url_organic_keywords` (per row) | 10 |
| `domain_vs_domain` (per row) | 40 |
| `keyword_overview` | 100 |
| `backlinks_overview` | 40 |
| `referring_domains` (per row) | 40 |

**Audit budget rule of thumb**: a 10-page audit ≈ 5–15k units. A full keyword research session ≈ 50–100k units. Cap rows on every call (`limit: N`) — don't pull 10k keywords when 200 will do.

## Caching

Semrush results don't change minute-by-minute. Cache locally during a session:

```bash
# Save first response, reuse within audit session
mkdir -p .claude/cache/semrush
# Naming: <tool>_<domain>_<date>.json
```

Re-pull only when you need a fresh snapshot for validation phase.

## What Semrush does NOT replace

- **Lighthouse / CrUX** — Semrush has no real-user CWV data
- **Google Search Console** — Semrush estimates traffic, GSC has actual click data
- **Schema validation** — use Rich Results Test
- **Crawl your site** — Semrush site audit is good but rate-limited; for ad-hoc deep crawl use Screaming Frog or your own script

Run Semrush + GSC + Lighthouse together. Semrush is competitive intelligence; GSC is your own data; Lighthouse is technical truth.

## Anti-patterns

- ❌ Pulling 10k keywords with no `limit` filter — burns API units
- ❌ Treating Semrush keyword volume as exact — it's a model, ±20–40% vs actual
- ❌ Calling `keyword_overview` for every long-tail variation — use `keyword_magic` once with broad seed
- ❌ Ignoring `database` parameter — defaults to US; must match the user's target market
- ❌ Citing Semrush data without the pull date — it's a snapshot

## Sources

- Semrush API docs: https://developer.semrush.com/api/
- Semrush MCP announcement: https://www.semrush.com/blog/mcp-server/
- API unit pricing: https://www.semrush.com/api-analytics/
