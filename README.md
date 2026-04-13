# Foggy Observe

AI-powered analytics and observability skill library for vibe-coded products.

## What is this?

A set of composable AI coding skills (for Claude Code, Cursor, etc.) that scan your codebase, understand your business, and generate a **personalized semantic layer** — tracking code + metric descriptions + business context — committed directly to your repo.

## Skills

| Skill | What it does | Status |
|-------|-------------|--------|
| `/observe` | Full flow: detect stack, set up PostHog, scan codebase, ask business questions, generate `tracking-plan.md`, commit tracking code | Done |
| `/observe-improve` | Audit existing tracking vs the plan. Fill gaps. | Planned |
| `/observe-dashboard` | Generate a single-page dashboard from the plan. | Planned |
| `/observe-diagnose` | AI data analyst. Reads plan + queries PostHog. | Planned |
| `/observe-infra` | OTel + Grafana. Infrastructure observability. | Planned |
| `/observe-alert` | Set up alerts from plan thresholds. | Planned |

## Shared Artifact: `tracking-plan.md`

Every skill reads and/or writes this file. It's the semantic layer — a machine-readable AND human-readable plan that describes what each metric means, why it matters, what's normal, and what to do when something breaks. Includes funnel metrics, infrastructure health, marketing attribution, runbook, and event properties.

## Architecture

```
User runs /observe in Claude Code / Cursor
      │
      ▼
┌─────────────────────────────────────────────┐
│  /observe (full flow)                       │
│  1. Detect stack (framework, deployment)    │
│  2. Set up PostHog (if missing)             │
│  3. Scan codebase (routes, auth, payments)  │
│  4. Ask 5 business questions                │
│  5. Generate tracking-plan.md               │
│  6. Generate + commit tracking code         │
│     (events, first-touch attribution,       │
│      revenue tracking, user identification) │
└────────┬────────────────────────────────────┘
         │
         ▼
┌──────────────────┐
│ /observe-improve  │ → Audit tracking vs plan → fill gaps
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ /observe-dashboard│ → Generate dashboard from plan + PostHog API
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ /observe-diagnose │ → AI analyst: reads plan + queries PostHog
└──────────────────┘
```

## Development

Skills are markdown files with instructions for AI coding tools.

```
foggy-observe/
  .claude/skills/
    observe.md                    # /observe skill (full flow)
  docs/
    tracking-plan-spec.md         # Plan format specification v0.1
    design-doc.md                 # Product design document
    deep-research.md              # Market research
  BACKLOG.md                      # Skill backlog and priorities
```
