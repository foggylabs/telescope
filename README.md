# Foggy Observe

AI-powered analytics and observability skill library for vibe-coded products.

## What is this?

A set of composable AI coding skills (for Claude Code, Cursor, etc.) that scan your codebase, understand your business, and generate a **personalized semantic layer** — tracking code + metric descriptions + business context — committed directly to your repo.

## Skills

| Skill | What it does | Status |
|-------|-------------|--------|
| `/observe-setup` | Install PostHog SDK, configure auto-capture. Zero to tracking. | Planned |
| `/observe-plan` | Scan codebase + ask business questions. Generate `observability-plan.md`. | Planned |
| `/observe-track` | Read the plan. Generate + commit PostHog tracking code. | Planned |
| `/observe-improve` | Audit existing tracking vs the plan. Fill gaps. | Planned |
| `/observe-dashboard` | Generate a single-page dashboard from the plan. | Planned |
| `/observe-diagnose` | AI data analyst. Reads plan + queries PostHog. | Planned |
| `/observe-infra` | OTel + Grafana. Infrastructure observability. | Planned |
| `/observe-alert` | Set up alerts from plan thresholds. | Planned |

## Shared Artifact: `observability-plan.md`

Every skill reads and/or writes this file. It's the semantic layer — a machine-readable AND human-readable plan that describes what each metric means, why it matters, what's normal, and what to do when something breaks.

## Architecture

```
User runs skill in Claude Code / Cursor
      │
      ▼
┌─────────────────┐
│  /observe-setup  │ → Install PostHog SDK, configure auto-capture
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  /observe-plan   │ → Scan codebase + questions → observability-plan.md
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  /observe-track  │ → Read plan → generate + commit tracking code
└────────┬────────┘
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

Skills are SKILL.md files — prompt engineering + instructions for AI coding tools.

```
foggy-observe/
  observe-setup/SKILL.md
  observe-plan/SKILL.md
  observe-track/SKILL.md
  ...
  docs/
    observability-plan-spec.md    # Plan format specification
    design-doc.md                 # Product design document
  BACKLOG.md                      # Skill backlog and priorities
```
