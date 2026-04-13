# Tracking Plan Specification v0.1

The tracking plan (`tracking-plan.md`) is a standardized markdown file that describes what a product tracks, why, and what to do when something breaks. It is designed for two consumers:

1. **Humans** — PMs, founders, and engineers read it to understand the product's metrics
2. **AI agents** — data analysts, SRE agents, and MCP servers parse it to query and analyze data

## File Location

`tracking-plan.md` in the repository root (same level as README.md).

## Format

The file consists of YAML frontmatter followed by markdown sections.

### Frontmatter (required)

```yaml
---
version: "0.1"
generated: "2026-04-13T14:00:00Z"
generator: "foggy-observe/0.1.0"
product:
  name: "Product Name"
  type: "saas"           # saas | marketplace | api | mobile | ecommerce | other
  business_model: "subscription"  # subscription | freemium | transactional | ad_supported | free
  stage: "pre_revenue"   # pre_revenue | has_users | has_revenue
analytics:
  provider: "posthog"    # posthog | mixpanel | amplitude | custom
  project_id: ""         # optional, for MCP/API access
stack:
  framework: "next.js"   # detected from codebase
  language: "typescript"
  deployment: "vercel"   # vercel | netlify | fly | render | aws | other
---
```

**Required fields:** version, product.name, product.type, analytics.provider
**Optional fields:** everything else (auto-detected or user-provided)

### Section 1: Funnel Metrics (required)

Organized by funnel stage. Each stage has a table with these columns:

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| Metric | string | yes | Human-readable name (e.g., "Signup completed") |
| Event | string | yes | Analytics event name (e.g., `user_signed_up`). Must be valid for the configured analytics provider. |
| Description | string | yes | One sentence explaining what this metric measures |
| Why | string | yes | Why this metric matters for the business |
| Normal | string | yes | Expected range or value (e.g., "3-8% of visitors", ">100/day") |
| Red Flag | string | yes | Condition that indicates a problem (e.g., "<2% sustained for 3 days") |

#### Required Funnel Stages

```markdown
## Funnel Metrics

### Acquisition
| Metric | Event | Description | Why | Normal | Red Flag |
|--------|-------|-------------|-----|--------|----------|
| ... | ... | ... | ... | ... | ... |

### Activation
| Metric | Event | Description | Why | Normal | Red Flag |
|--------|-------|-------------|-----|--------|----------|
| ... | ... | ... | ... | ... | ... |

### Engagement
| Metric | Event | Description | Why | Normal | Red Flag |
|--------|-------|-------------|-----|--------|----------|
| ... | ... | ... | ... | ... | ... |

### Retention
| Metric | Event | Description | Why | Normal | Red Flag |
|--------|-------|-------------|-----|--------|----------|
| ... | ... | ... | ... | ... | ... |

### Revenue
| Metric | Event | Description | Why | Normal | Red Flag |
|--------|-------|-------------|-----|--------|----------|
| ... | ... | ... | ... | ... | ... |
```

**Rules:**
- Every stage must have at least 1 metric
- Event names must follow the analytics provider's naming convention (PostHog: snake_case)
- "Revenue" section can be omitted if `product.stage` is `pre_revenue`

### Section 2: Infrastructure Health (optional in v1)

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| Metric | string | yes | Human-readable name |
| Source | string | yes | Where this metric comes from (e.g., "server logs", "Vercel metrics") |
| Description | string | yes | What it measures |
| Threshold | string | yes | Normal operating range |
| Action | string | yes | What to do when threshold is breached |

```markdown
## Infrastructure Health

| Metric | Source | Description | Threshold | Action |
|--------|--------|-------------|-----------|--------|
| Error rate (5xx) | server framework | % of requests returning 500+ | <1% | Investigate recent deploys |
| P95 latency | hosting platform | 95th percentile response time | <2s web, <500ms API | Profile slow endpoints |
| Uptime | hosting platform | % of time the app is reachable | >99.5% | Check hosting status |
```

### Section 3: Runbook (required)

Actionable guidance for each red flag in the funnel metrics. One subsection per metric that has a red flag.

```markdown
## When Something Breaks

### Signup rate dropped below 2%
1. Check if the signup page is loading (open it, verify)
2. Check PostHog for errors on the signup page
3. Review recent deploys for changes to the signup flow
4. Check if a third-party auth provider (Google, GitHub) is down
```

**Rules:**
- Every red flag in Section 1 should have a corresponding runbook entry
- Steps should be actionable by someone who can use AI coding tools but isn't a DevOps engineer
- Include specific URLs, commands, or PostHog queries where possible

### Section 4: Event Properties (optional, recommended)

Detailed schema for each event, useful for tracking code generation and data analysis.

```markdown
## Event Properties

### user_signed_up
| Property | Type | Description | Example |
|----------|------|-------------|---------|
| method | string | Signup method used | "email", "google", "github" |
| referrer | string | Traffic source | "producthunt", "organic", "direct" |
| plan | string | Selected plan (if applicable) | "free", "pro" |
```

### Section 5: Marketing Attribution (recommended)

Tracks where users come from and which sources drive revenue.

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| Channel | string | yes | Marketing channel name (e.g., "Organic Search", "Twitter/X", "Reddit") |
| Tracking Method | string | yes | How this channel is identified (e.g., "UTM params", "referrer header", "document.referrer") |
| Key Metrics | string | yes | What to measure for this channel (e.g., "visitors, signups, revenue, revenue/visitor") |
| Attribution Event | string | yes | PostHog event that captures the source (e.g., `page_viewed` with `$referrer` property) |

#### Standard Channels to Track

| Channel | Tracking Method | Example |
|---------|----------------|---------|
| Organic Search | `$referrer` contains google/bing/duckduckgo | SEO traffic |
| Paid Ads | UTM params (`utm_source`, `utm_medium`, `utm_campaign`) | Google Ads, Meta Ads |
| Social Media | `$referrer` contains twitter/reddit/linkedin/hackernews | Organic social |
| Direct | No referrer | Bookmarks, typed URL |
| Referral | `$referrer` from other sites | Blog mentions, partner links |
| Email | UTM params with `utm_medium=email` | Newsletter, drip campaigns |
| Product Hunt | `$referrer` contains producthunt.com | Launch traffic |

#### Revenue Attribution Properties

These properties should be captured on conversion/payment events to enable revenue-per-channel analysis:

| Property | Type | Description | Example |
|----------|------|-------------|---------|
| first_touch_source | string | Original traffic source (first visit) | "google", "twitter", "direct" |
| first_touch_medium | string | Original medium | "organic", "social", "paid" |
| first_touch_campaign | string | Original campaign (if UTM) | "launch-week", "blog-post-1" |
| last_touch_source | string | Most recent source before conversion | "direct" |
| revenue_amount | number | Transaction amount in cents | 2900 |
| payment_provider | string | Payment processor | "stripe", "lemonsqueezy" |

PostHog already captures `$referrer`, `$referring_domain`, and UTM parameters automatically via `posthog-js`. The skill should ensure these are flowing and add `first_touch_*` properties via `posthog.register()` for attribution across sessions.

## Machine Parsing

The plan is designed to be parseable by simple markdown parsers or AI agents:

- **Frontmatter** is standard YAML, parseable by any YAML library
- **Tables** use standard GitHub-flavored markdown table format
- **Section headers** (##, ###) define the hierarchy
- **Funnel stage names** (Acquisition, Activation, Engagement, Retention, Revenue) are fixed keywords
- **Event names** in the "Event" column are the exact strings to use in `posthog.capture()` or equivalent

An AI agent reading this plan can:
1. Parse the frontmatter to understand the product and analytics provider
2. Find a metric by scanning table rows
3. Get the event name to query from the "Event" column
4. Understand what's normal from the "Normal" column
5. Know when to alert from the "Red Flag" column
6. Look up the runbook for diagnosis

## Versioning

The `version` field in frontmatter tracks the spec version. Current: `0.1`.

Breaking changes increment the minor version (0.1 → 0.2).
Non-breaking additions are compatible within the same minor version.

## Quality Criteria

A plan meets the quality bar when:
1. Every metric has all required columns filled with specific, non-generic content
2. Event names are valid for the configured analytics provider
3. "Normal" ranges are specific to the product type (not "varies" or "depends")
4. "Red Flag" conditions include a time dimension (e.g., "for 3+ days", "week-over-week")
5. The runbook has actionable steps for every red flag
6. A PM reading it says "this covers what I would track"
7. An AI agent can answer "what happened to activation this week?" by querying events from the plan
