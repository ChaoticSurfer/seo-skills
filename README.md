# seo-skills

Production-grade SEO audit & optimization skills for **Next.js 15**, **Astro 5**, and **Nuxt 3** (TypeScript). Reflects 2026 Google guidance — Core Web Vitals (INP), AI Overviews / GEO/AEO, structured-data deprecations, and explicit AI-crawler policy. Optional Semrush MCP integration for competitor data.

## What's inside

- **`seo-nextjs`** — Next.js 15 App Router (TS): `generateMetadata`, `app/sitemap.ts`, `app/robots.ts`, `next/og`, JSON-LD, ISR/SSG/SSR for SEO.
- **`seo-vue-astro`** — Astro 5 + Nuxt 3 (TS): `BaseLayout` patterns, content collections, `@astrojs/sitemap`, `useSeoMeta`, `useSchemaOrg`, `routeRules`, `nuxt-og-image`.
- **`seo-shared/`** — referenced by both skills:
  - `core-web-vitals.md` — 2026 thresholds (LCP ≤2.5s · INP ≤200ms · CLS ≤0.1) + root-cause taxonomy
  - `structured-data.md` — JSON-LD templates with 2026 status (HowTo dead, FAQ restricted)
  - `geo-aeo.md` — AI Overview / LLM citation tactics, AI-crawler robots.txt, llms.txt honest take
  - `audit-rubric.md` — Evidence + Confidence framework + 100-point scorecard
  - `semrush-mcp.md` — **optional** Semrush MCP audit module (gated on `ToolSearch`)

## Install

### Via Claude Code plugin marketplace

```bash
/plugin marketplace add ChaoticSurfer/seo-skills
/plugin install seo-skills@chaoticsurfer-seo
```

### Manual (clone)

```bash
git clone https://github.com/ChaoticSurfer/seo-skills ~/.claude/plugins/seo-skills
```

Or for a single project:
```bash
mkdir -p .claude/skills && cp -r seo-skills/skills/* .claude/skills/
```

## Use

Once installed, the skills auto-trigger when you ask Claude Code to:
- "Audit this site's SEO"
- "Improve our Core Web Vitals"
- "Add structured data to the blog"
- "Why aren't we ranking for X?"
- Any Next.js / Astro / Nuxt SEO question

Each skill follows a 5-phase workflow: **Detect → Audit → Score → Fix → Validate**, and produces evidence-backed findings with priority ranking.

## Optional: Semrush MCP

If you have the [Semrush MCP server](https://www.semrush.com/blog/mcp-server/) connected, both skills automatically detect it (`ToolSearch({ query: "semrush" })`) and run the additional Semrush audit module — competitor gap, keyword inventory, position tracking. **The skills work fully without it.**

Connect Semrush MCP:

```json
{
  "mcpServers": {
    "semrush": {
      "command": "npx",
      "args": ["-y", "@semrush/mcp-server"],
      "env": { "SEMRUSH_API_KEY": "${SEMRUSH_API_KEY}" }
    }
  }
}
```

## What this skill does NOT do

- Replace **Lighthouse / CrUX** for real-user CWV data
- Replace **Google Search Console** for first-party click data
- Schema validation — use [Rich Results Test](https://search.google.com/test/rich-results) and [validator.schema.org](https://validator.schema.org/)
- Deep crawls of huge sites — use Screaming Frog or a dedicated crawler

This skill is the **brain**: it directs the audit, generates code patches, and writes the report. It assumes you have basic web tools (`curl`, `lighthouse`, `xmllint`) available.

## 2026 facts baked in

- INP replaced FID 2024-03-12 (200ms threshold; ~43% of sites fail)
- HowTo schema deprecated (removed entirely)
- FAQPage rich results restricted to gov / well-known health
- AI Overviews on ~48% of queries (Feb 2026); #1 organic CTR -35%
- Only ~38% of AI citations come from top-10 organic
- llms.txt is **not** used by Google for ranking (still useful for Anthropic/Perplexity ingestion)
- AI-crawler policy must be explicit: GPTBot, ClaudeBot, PerplexityBot, Google-Extended, Applebot-Extended, etc.
- Citability sweet spot: 134–167 word self-contained passages
- AVIF > WebP > JPEG

## Patterns borrowed from prior art

- Phase-graph workflow (aaron-he-zhu/seo-geo-claude-skills)
- Script + LLM two-layer audit (JeffLi1993/seo-audit-skill)
- Evidence + Confidence rubric (Bhanunamikaze/Agentic-SEO-Skill)
- 100-point weighted scorecard (huifer/claude-code-seo)
- AI-crawler robots.txt audit (zubair-trabzada/geo-seo-claude)
- Per-element auto-bounds: title 30–65, description 70–160

## Layout

```
seo-skills/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── skills/
│   ├── seo-nextjs/
│   │   ├── SKILL.md
│   │   └── reference/checklist.md
│   ├── seo-vue-astro/
│   │   ├── SKILL.md
│   │   └── reference/checklist.md
│   └── seo-shared/
│       ├── core-web-vitals.md
│       ├── structured-data.md
│       ├── geo-aeo.md
│       ├── audit-rubric.md
│       └── semrush-mcp.md
├── README.md
└── LICENSE
```

## License

MIT — see [LICENSE](./LICENSE).

## Contributing

Issues + PRs welcome. Especially:
- New framework support (SvelteKit, Remix, Qwik)
- Additional optional MCP integrations (GSC, Lighthouse, DataForSEO)
- Real-world audit reports as case studies

## Author

[Anri Kezeroti](https://github.com/ChaoticSurfer) — `anri.k@edumentors.co.uk`
