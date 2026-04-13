# CLAUDE.md — Foggy Observe

## What is this?

Foggy Observe is a skill library for AI coding tools (Claude Code, Cursor, etc.) that generates personalized analytics and observability for vibe-coded products.

Each skill is a SKILL.md file in its own directory. Skills share a common artifact: `observability-plan.md` (the semantic layer).

## Repository Structure

```
foggy-observe/
  observe-setup/SKILL.md      # Install PostHog SDK
  observe-plan/SKILL.md       # Generate semantic layer
  observe-track/SKILL.md      # Generate tracking code from plan
  observe-improve/SKILL.md    # Audit and improve tracking
  observe-dashboard/SKILL.md  # Generate dashboard from plan
  observe-diagnose/SKILL.md   # AI data analyst (paid)
  observe-infra/SKILL.md      # OTel + Grafana (later)
  observe-alert/SKILL.md      # Alert setup (later)
  docs/                       # Design docs, research, specs
  BACKLOG.md                  # Skill backlog and priorities
```

## Development

Skills are prompt engineering. Each SKILL.md contains:
- YAML frontmatter (name, description, triggers)
- Instructions for the AI coding tool
- Templates for generated output

## Conventions

- Skill names: `observe-{action}` (lowercase, hyphenated)
- Shared artifact: `observability-plan.md` in the user's repo root
- Plan format spec: `docs/observability-plan-spec.md`
- All free skills are open source. `/observe-diagnose` is the paid product.

## Backlog

Read BACKLOG.md for current priorities. All new work items go there.
