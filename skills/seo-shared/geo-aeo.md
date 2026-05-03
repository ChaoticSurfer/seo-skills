# GEO / AEO — Generative & Answer Engine Optimization (2026)

Optimizing for AI Overviews, ChatGPT, Perplexity, Claude, Gemini citations.

## Why this matters now

- **AI Overviews on ~48% of tracked queries** (Feb 2026), up from ~31% YoY (+58%)
- **#1 organic CTR has dropped ~35%** in AI Overview SERPs
- **Education**: 18% → 83% of queries trigger AIO. **B2B Tech**: 36% → 82%.
- **Only 38% of AI citations come from top-10 organic** (down from 76%) — lower-ranked pages with strong E-E-A-T can still get cited

Traditional SEO is necessary but no longer sufficient.

## Citability — what gets cited

1. **Self-contained passages of 134–167 words** (sweet spot for AI extraction). Each passage answers one question completely without surrounding context.
2. **Direct answer in the first 200 words** — no buildup. Answer first, elaborate after.
3. **Original data, statistics, first-party research** — "information gain" is the strongest citation signal.
4. **Q&A blocks** with question as H2/H3 and answer immediately below.
5. **Entity-rich prose** — name people, products, places, dates explicitly. Avoid pronouns where entities matter.
6. **Brand mentions** are reportedly **3× stronger signal** than raw backlinks for LLM citation.

## Cite-able passage example (134–167 words)

```markdown
## How long does an INP measurement take?

INP measures the latency between a user interaction (click, tap, keypress) and 
the next paint. Google takes the **75th percentile** of all interactions across 
a 28-day window from real users via the Chrome User Experience Report. A page 
needs at least 50 sampled interactions in that window to receive an INP score.

Threshold: ≤ 200ms is "Good", 201–500ms is "Needs Improvement", > 500ms is 
"Poor". INP replaced FID on March 12, 2024 because INP captures the latency 
of every interaction during the page lifetime, while FID only captured the 
first.
```

Direct answer, specific numbers, no fluff, entity-rich (Google, CrUX, FID).

## llms.txt — the honest take

**Google does NOT use llms.txt for ranking or AI Overview citations** (John Mueller, 2025).

It IS useful for:
- Anthropic, Stripe, Cloudflare, Hugging Face, Perplexity, Mintlify, Cursor — these pull `/llms.txt` for docs ingestion
- Internal docs sites that want LLM-friendly Markdown alongside HTML
- ~10% adoption industry-wide

Implement it as a **one-time low-effort signal**, not a strategy.

```markdown
# /llms.txt
# Site Name

> One-paragraph site summary including primary keywords and entity names.

## Documentation
- [Getting Started](https://example.com/docs/start.md): one-line description
- [API Reference](https://example.com/docs/api.md): one-line description

## Blog
- [Latest Post](https://example.com/blog/post.md): one-line description
```

Plus per-page `.md` versions (or `application/markdown` content negotiation).

## AI crawler robots.txt — explicit allow/block

Decide explicitly. Each crawler:

| Crawler | Bot UA | Operator | Use |
|---|---|---|---|
| GPTBot | `GPTBot` | OpenAI | Training + ChatGPT browsing |
| ChatGPT-User | `ChatGPT-User` | OpenAI | On-demand ChatGPT browsing |
| OAI-SearchBot | `OAI-SearchBot` | OpenAI | ChatGPT Search |
| ClaudeBot | `ClaudeBot` | Anthropic | Training |
| Claude-Web | `Claude-Web` | Anthropic | Browsing |
| anthropic-ai | `anthropic-ai` | Anthropic | Older UA, still seen |
| PerplexityBot | `PerplexityBot` | Perplexity | Search index |
| Perplexity-User | `Perplexity-User` | Perplexity | On-demand |
| Google-Extended | `Google-Extended` | Google | Bard/Gemini training (separate from Googlebot) |
| Applebot-Extended | `Applebot-Extended` | Apple | Apple Intelligence training |
| CCBot | `CCBot` | Common Crawl | Public dataset, used by many LLMs |
| Bytespider | `Bytespider` | ByteDance | Training |
| Meta-ExternalAgent | `Meta-ExternalAgent` | Meta | Training |

**Maximum citation reach** (recommended for marketing sites):
```
User-agent: *
Allow: /

User-agent: GPTBot
Allow: /
User-agent: ClaudeBot
Allow: /
User-agent: PerplexityBot
Allow: /
User-agent: Google-Extended
Allow: /
```

**Block training, allow on-demand citation**:
```
User-agent: GPTBot
Disallow: /
User-agent: ClaudeBot
Disallow: /
User-agent: Google-Extended
Disallow: /

User-agent: ChatGPT-User
Allow: /
User-agent: Perplexity-User
Allow: /
User-agent: OAI-SearchBot
Allow: /
```

## Topic clusters → AI citations

Hub-and-spoke architecture reportedly raises AI citation rates from **~12% → ~41%** for pillar topics. Pillars 3–5k words covering the entire topic, clusters 800–2000 words on subtopics, all bidirectionally linked.

## E-E-A-T — what AI looks for

- **Experience**: first-hand examples, screenshots from your own usage
- **Expertise**: author credentials, jobTitle in schema, sameAs links to LinkedIn/GitHub
- **Authoritativeness**: original data, citations from authoritative sources
- **Trust**: clear about page, contact info, security (HTTPS), accurate dates

## Anti-patterns

- ❌ AI-generated thin content — Helpful Content System (now in core ranking) demotes this
- ❌ Stuffing the first 200 words with keywords — write a real answer
- ❌ Treating llms.txt as a Google ranking factor
- ❌ Fake author bios — must be verifiable real people
- ❌ Burying the answer 800 words deep "for engagement" — kills citation chance
