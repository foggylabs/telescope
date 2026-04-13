# Foggy Observe — Skill Backlog

## P1: Ship First (Week 1-2)

| ID | Skill | Description | Status |
|----|-------|-------------|--------|
| OBS-1 | `/observe-setup` | Install PostHog SDK, configure auto-capture. Detect framework (React, Next.js, Svelte, etc.), install posthog-js, add provider wrapper, configure project API key. | TODO |
| OBS-2 | `/observe-plan` | Scan codebase, ask 5 business questions, generate observability-plan.md. The semantic layer — metric descriptions, thresholds, actions, organized by funnel stage. | TODO |

## P2: Make It Actionable (Week 2-3)

| ID | Skill | Description | Status |
|----|-------|-------------|--------|
| OBS-3 | `/observe-track` | Read observability-plan.md, generate PostHog tracking code (posthog.capture calls), commit to repo. Maps each plan metric to a concrete event. | TODO |

## P3: Iterate and Visualize (Week 3-4)

| ID | Skill | Description | Status |
|----|-------|-------------|--------|
| OBS-4 | `/observe-improve` | Audit existing tracking code against the plan. Find gaps (metrics in plan but not tracked), noise (events tracked but not in plan), quality issues (events missing properties). Suggest and apply fixes. | TODO |
| OBS-5 | `/observe-dashboard` | Generate a single-page HTML dashboard that reads from PostHog API. Personalized to the plan. Deployed as static file or in the repo. The "whoa" moment. | TODO |

## P4: The AI Colleague (Paid Product)

| ID | Skill | Description | Status |
|----|-------|-------------|--------|
| OBS-6 | `/observe-diagnose` | AI data analyst agent. Reads observability-plan.md + queries PostHog data. Answers: "Why did activation drop this week?", "Which channel has the best conversion?", "Is the app healthy?" Daily digest via Slack/email. | TODO |

## P5: Cross-Tool Moat

| ID | Skill | Description | Status |
|----|-------|-------------|--------|
| OBS-7 | `/observe-infra` | OTel + Grafana infrastructure observability. Health checks, error rates, response times, uptime. Extends observability-plan.md with infra metrics. | TODO |
| OBS-8 | `/observe-alert` | Set up alerts based on plan thresholds. PostHog alerts for product metrics, Grafana alerting rules for infra metrics. | TODO |

## Dependencies

- OBS-2 depends on OBS-1 (plan references PostHog event schemas)
- OBS-3 depends on OBS-2 (reads the plan to generate code)
- OBS-4 depends on OBS-3 (audits existing tracking)
- OBS-5 depends on OBS-2 (reads the plan for dashboard structure)
- OBS-6 depends on OBS-2 (reads the plan for context)
- OBS-7 is independent (can be built in parallel)
- OBS-8 depends on OBS-2 + OBS-7 (reads plan thresholds)

## Key Decision Log

- 2026-04-13: Chose skill library architecture over monolithic skill (CEO review)
- 2026-04-13: Tracking code generation included in v1 scope (moved from fast-follow)
- 2026-04-13: Open source for free skills (P1-P5), paid for AI colleague (P4)
- 2026-04-13: Repo is private for now, will open source when skills are ready
