# /telescope-execute — Generate tracking code from the approved plan

You are an implementation agent. The tracking plan is a **semantic layer** that AI agents will use to understand this product's data. Your job is to turn it into working PostHog tracking code that produces clean, well-structured events matching the plan exactly.

The data quality matters: AI agents will query these events by name and property. If the code captures `signup_complete` but the plan says `user_signed_up`, the semantic layer is broken and every AI agent built on it fails.

**Prerequisites:** `tracking-plan.md` must exist in the repo root and must have been approved by the user (via `/telescope-review`). If it doesn't exist, tell the user to run `/telescope-explore` first.

## Step 1: PostHog SDK setup (if missing)

Check if PostHog SDK is installed in the project dependencies.

If **not installed**:

1. Tell the user: "You need a PostHog account (free — 1M events/month). Sign up at posthog.com if you haven't already."
2. Install the SDK for the detected stack:
   - **Frontend**: `posthog-js` (via npm/yarn/pnpm/bun based on lockfile)
   - **Node.js backend**: `posthog-node`
   - **Python backend**: `posthog` (via pip/uv)
   - **Go backend**: `posthog-go`
   - **Ruby backend**: `posthog-ruby`
3. Ask the user for their PostHog project API key
4. Add PostHog initialization code to the correct entry point

**Critical init options** (read from the PostHog Configuration Notes in the plan):
- `person_profiles: 'identified_only'` — always set this
- `capture_pageview: 'history_change'` — for SPAs (React, Vue, Svelte with client-side routing)
- `cross_subdomain_cookie: true` — if product spans subdomains
- `api_host: 'https://us.i.posthog.com'` (or `eu.i.posthog.com`)

Store the API key in an environment variable appropriate for the framework. Add the env var name to `.env.example` (create if needed).

If PostHog is **already installed**: verify the init options match what the plan requires. Fix if needed.

## Step 2: User identification

This must be implemented BEFORE event tracking. Without it, all events are anonymous.

**Client-side (`posthog-js`):**

Call `posthog.identify()` on every authenticated page load and immediately after login/signup:

```ts
posthog.identify(userId, {
  // $set properties (updated each time)
  email: user.email,
  name: user.name,
  // add other $set person properties from the plan
})
```

Also set `$set_once` properties on signup:

```ts
posthog.identify(userId, {}, {
  // $set_once properties (immutable)
  signup_method: 'google',
  signup_date: new Date().toISOString(),
})
```

**Server-side (`posthog-python` / `posthog-node`):**

Every server-side `capture()` call requires `distinct_id`. Get it from the authenticated session:

```python
# Python
posthog.capture(
    distinct_id=request.user.id,  # from authenticated session
    event='user_signed_up',
    properties={ ... }
)
```

```ts
// Node.js
posthog.capture({
    distinctId: req.user.id,  // from authenticated session
    event: 'user_signed_up',
    properties: { ... }
})
```

## Step 3: Group analytics (for multi-tenant / B2B)

If the plan defines group types, implement `posthog.group()`:

```ts
// Client-side — call on login and project/workspace switch
posthog.group('project', projectId, {
  name: projectName,
  // add other group properties from the plan
})
```

```python
# Server-side — include group in capture calls
posthog.capture(
    distinct_id=user_id,
    event='thread_created',
    properties={ ... },
    groups={ 'project': project_id }
)
```

## Step 4: Generate event tracking code

Read `tracking-plan.md`. For each custom event in the funnel metrics, generate tracking code.

**Client-side events** (`Capture: client` in the plan):
```ts
posthog.capture('event_name', {
  property_1: value1,
  property_2: value2,
})
```

**Server-side events** (`Capture: server` in the plan):
```python
posthog.capture(
    distinct_id=request.user.id,
    event='event_name',
    properties={
        'property_1': value1,
        'property_2': value2,
    }
)
```

**Placement rules:**
- Client: place in click handlers, form submit handlers, component effects
- Server: place in route handlers, service methods, webhook handlers, background job completions
- Follow the plan's `Capture` column — don't second-guess client vs server

**Do NOT duplicate autocapture events.** PostHog already captures:
- `$pageview` (with URL, referrer, UTMs) on every page/route change
- `$pageleave` (with scroll depth)
- `$autocapture` (clicks on buttons/links/forms)

Only add custom `posthog.capture()` calls for events that autocapture doesn't cover.

## Step 5: First-touch attribution

Add first-touch attribution on the **client-side entry point** (runs on first visit, persists across sessions):

```ts
posthog.register_once({
  first_touch_source: urlParams.get('utm_source') || getReferrerSource(document.referrer),
  first_touch_medium: urlParams.get('utm_medium') || getReferrerMedium(document.referrer),
  first_touch_campaign: urlParams.get('utm_campaign') || '',
})
```

Also set as person properties on identify (so attribution survives across devices):

```ts
posthog.identify(userId, {}, {
  // $set_once — only written on first identify
  first_touch_source: firstTouchSource,
  first_touch_medium: firstTouchMedium,
  first_touch_campaign: firstTouchCampaign,
})
```

Include referrer classification helpers:

```ts
function getReferrerSource(referrer: string): string {
  if (!referrer) return 'direct'
  const domain = new URL(referrer).hostname
  if (domain.includes('google')) return 'google'
  if (domain.includes('bing')) return 'bing'
  if (domain.includes('twitter') || domain.includes('x.com')) return 'twitter'
  if (domain.includes('reddit')) return 'reddit'
  if (domain.includes('linkedin')) return 'linkedin'
  if (domain.includes('producthunt')) return 'producthunt'
  if (domain.includes('news.ycombinator')) return 'hackernews'
  return domain
}

function getReferrerMedium(referrer: string): string {
  if (!referrer) return 'none'
  const domain = new URL(referrer).hostname
  if (['google', 'bing', 'duckduckgo', 'yahoo'].some(s => domain.includes(s))) return 'organic'
  if (['twitter', 'x.com', 'reddit', 'linkedin', 'facebook'].some(s => domain.includes(s))) return 'social'
  return 'referral'
}
```

## Step 6: Review and commit

Show the user a summary:

> **Tracking code generated.** Here's what I set up:
> - PostHog SDK initialized with [config options]
> - `posthog.identify()` on [login/signup/page load]
> - `posthog.group()` for [group type] (if applicable)
> - **X custom events** across **Y funnel stages**
> - First-touch attribution via `register_once()` + `$set_once`
> - **N files** modified
>
> Want me to commit?

Wait for confirmation. Commit with message:

```
feat: add PostHog tracking code (via /telescope-execute)
```

Include all modified files. Do NOT include `tracking-plan.md` in this commit — it was already saved by `/telescope-plan`.
