# /telescope-execute — Generate tracking code from the approved plan

You are an implementation agent. Your job is to turn the approved `tracking-plan.md` into working PostHog code.

**Only implement what the plan specifies.** Do NOT add tracking for things PostHog handles automatically.

**Prerequisites:** `tracking-plan.md` must exist and be approved via `/telescope-review`.

## What PostHog already does — DO NOT implement

PostHog autocaptures these with zero code (after SDK init):
- `$pageview` on every page/route change (with URL, referrer, UTMs, scroll depth)
- `$pageleave` with scroll depth
- `$autocapture` — clicks on buttons, links, forms, inputs (with element text, CSS selector)
- UTM parameters, referrer, device info — all auto-captured as event properties
- `$initial_referrer`, `$initial_utm_source`, etc. — auto-set as person properties
- Session replay, heatmaps, web analytics — built-in, no code

**Do NOT write `posthog.capture()` for page views, button clicks, form submissions, or marketing attribution.** PostHog handles all of this.

## Step 1: PostHog SDK setup (if missing)

Check if PostHog SDK is installed. If not:

1. Tell user to sign up at posthog.com (free — 1M events/month)
2. Install SDK for the detected stack (posthog-js for frontend, posthog-python/posthog-node for backend)
3. Ask user for their PostHog **project token** (find it at Settings > General > "Project token & ID" — starts with `phc_`)
4. Add initialization code to the correct entry point

**Critical init options** (from the plan's PostHog Configuration section):
- `person_profiles: 'identified_only'`
- `capture_pageview: 'history_change'` — for SPAs
- `cross_subdomain_cookie: true` — if product spans subdomains
- `api_host: 'https://us.i.posthog.com'`

Store API key in an environment variable. Add to `.env.example`.

## Step 2: User identification

Implement BEFORE custom events. Without this, all events are anonymous.

**Client-side** — call on every authenticated page load and after login/signup:

```ts
posthog.identify(userId, {
  // $set — updated each time
  email: user.email,
  name: user.name,
  // add other $set properties from the plan
}, {
  // $set_once — immutable, only set first time
  signup_method: 'google',
  signup_date: new Date().toISOString(),
  // add other $set_once from the plan
})
```

Call `posthog.reset()` on logout.

**Server-side** — every `capture()` call needs `distinct_id` from the authenticated session.

## Step 3: Group analytics (for B2B / multi-tenant)

If the plan defines group types:

```ts
// Client-side — on login and group/project switch
posthog.group('project', projectId, {
  name: projectName,
  // group properties from the plan
})
```

```python
# Server-side — include in capture calls
posthog.capture(
    distinct_id=user_id,
    event='event_name',
    properties={ ... },
    groups={ 'project': project_id }
)
```

## Step 4: Custom event tracking code

Read the plan's Funnel Events section. For each event marked `client` or `server`, generate a `posthog.capture()` call. **Skip all events marked `auto`** — PostHog handles those.

**Client-side:**
```ts
posthog.capture('event_name', { property: value })
```

**Server-side:**
```python
posthog.capture(
    distinct_id=request.user.id,
    event='event_name',
    properties={ 'property': value }
)
```

Place in the exact code paths the plan references. Match existing code style.

## Step 5: PostHog Actions — create them for the user, don't hand off work

PostHog Actions group autocaptured events into business-meaningful names (e.g., all CTA clicks → "Signup CTA clicked"). They're **optional** — the raw autocaptured data exists without them. Actions just make dashboards and funnels easier to build in the PostHog UI.

**Do NOT dump a list of Actions on the user and tell them to create them manually.** That's pushing work back to the user. Instead, offer to create them automatically.

### How to handle Actions

Ask the user ONE question via AskUserQuestion with these options:

1. **Create Actions automatically** — "I'll need your PostHog personal API key (a separate key from the project token — create one at Settings > Personal API keys). Takes 10 seconds."
2. **Skip Actions** — "The raw autocaptured data is already tracked. You can create Actions later from the PostHog UI if you want dashboard shortcuts."
3. **Generate a script** — "I'll write a shell script you can run once to create all the Actions. Check it and run it when ready."

### If the user picks "Create Actions automatically":

Use the PostHog Actions API: `POST /api/projects/{project_id}/actions/`.

For each Action in the plan, send:
```bash
curl -X POST "https://us.i.posthog.com/api/projects/{project_id}/actions/" \
  -H "Authorization: Bearer $PERSONAL_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Signup CTA clicked",
    "description": "Visitor clicks any signup CTA",
    "steps": [{
      "event": "$autocapture",
      "text": "Request access",
      "text_matching": "contains"
    }]
  }'
```

Report what was created. Do NOT ask the user to verify in the UI — you made the API calls, you know they succeeded.

### If the user picks "Generate a script":

Create `scripts/setup-posthog-actions.sh` with all the curl commands. Include instructions at the top of the file for how to run it. Do NOT add this script to the commit — it's a one-time setup helper.

### If the user picks "Skip Actions":

Fine. Note in the summary that Actions were skipped and the user can create them from the plan later.

**Never** present a list of Actions and say "create these manually in the UI." That's a skill failing at its job.

## Step 6: Verify implementation matches the plan

Before committing, verify every custom event in the plan is actually implemented. Go through the plan's Funnel Events section and check each `client` or `server` event:

1. **Read the generated code** — find the actual `posthog.capture()` call for each event
2. **Event name matches exactly** — the code must use the exact snake_case name from the plan
3. **All properties are captured** — every property listed in the Event Properties section is present in the capture call
4. **`distinct_id` is correct** — server-side events use the source documented in the plan
5. **`$groups` is included** — project-scoped server events include the group parameter
6. **Placement is correct** — the capture call is in the right file/function (state changes server-side, UI interactions client-side)
7. **`posthog.identify()` is called** — with the correct person properties (`$set` and `$set_once`)
8. **`posthog.group()` is called** — on the correct triggers (page load, project switch)
9. **`posthog.reset()` is called** — in the logout handler
10. **PostHog init config matches** — compare the actual `posthog.init()` call against the plan's PostHog Configuration section

Present a verification checklist:

```
## Implementation Verification

| Event | In Plan | In Code | File | Status |
|-------|---------|---------|------|--------|
| user_signed_up | server | server | routes/auth.py:145 | OK |
| email_verified | server | server | routes/auth.py:210 | OK |
| ... | ... | ... | ... | ... |

### Missing
- [any events in plan but not in code]

### Extra
- [any capture calls in code but not in plan]

### Config
- [ ] posthog.init() matches plan config
- [ ] posthog.identify() called on login/signup/page load
- [ ] posthog.group() called on project load/switch
- [ ] posthog.reset() called on logout
```

If anything is missing or wrong, fix it before committing.

## Step 7: Commit

Show the user the verification results and:

> **Tracking code generated and verified.** All X events from the plan are implemented.
> - **N files** modified
> - PostHog SDK initialized on [apps]
> - Identity, groups, and reset configured
>
> Want me to commit?

Commit with: `feat: add PostHog tracking code (via /telescope-execute)`

Do NOT include `tracking-plan.md` in this commit.
