# Telescope

Add analytics to your product with context built for AI agents.

Telescope scans your codebase, generates a `tracking-plan.md` with full context — what each metric means, why it matters, what's normal, what's broken — and commits PostHog tracking code. The result: AI agents can answer "why did activation drop this week?" instead of staring at raw numbers.

Works with Claude Code, Cursor, Codex, and any AI coding tool.

## Install — 30 seconds

Open your AI coding tool and paste this:

**Claude Code:**

> Install telescope: run `git clone --single-branch --depth 1 https://github.com/foggylabs/telescope.git ~/.claude/skills/telescope && cd ~/.claude/skills/telescope && ./setup` then add a "Telescope" section to CLAUDE.md that lists the available skills: `/telescope-explore` — scan and understand your codebase, `/telescope-plan` — generate a tracking plan, `/telescope-review` — validate the plan, `/telescope-execute` — generate tracking code, `/telescope-add-feature-plan` — add tracking for a new feature. Start with `/telescope-explore` for initial setup or `/telescope-add-feature-plan` for existing projects.

**Cursor / Codex / Other:**

> Install telescope: run `git clone --single-branch --depth 1 https://github.com/foggylabs/telescope.git ~/.telescope && cd ~/.telescope && ./setup` then add a "Telescope" section to your instructions listing: `/telescope-explore`, `/telescope-plan`, `/telescope-review`, `/telescope-execute`, `/telescope-add-feature-plan`. Start with `/telescope-explore`.

## Skills

| Skill | What it does |
|-------|-------------|
| `/telescope-explore` | Scan your codebase — stack, routes, auth, payments, existing analytics |
| `/telescope-plan` | Generate `tracking-plan.md` — funnel metrics, marketing attribution, event properties, all with context for AI agents |
| `/telescope-review` | Validate the plan against actual code paths, fix issues |
| `/telescope-execute` | Generate PostHog tracking code and commit |
| `/telescope-add-feature-plan` | Add tracking for a new feature to an existing plan |

**Initial setup:** `/telescope-explore` → auto-chains through plan → review → execute

**New feature:** `/telescope-add-feature-plan`

## What it generates

**`tracking-plan.md`** — analytics context that AI agents can read:
- Funnel metrics with normal ranges and red flags
- Marketing attribution (channels, revenue per visitor)
- Event properties (schema for each event)

**Tracking code** — placed in the correct files, client-side for UI interactions, server-side for state changes.

## Supported stacks

Next.js, React + Vite, Express, Django, Flask, Rails, Go. Payment providers: Stripe, LemonSqueezy, Paddle, Polar.

## Updating

```bash
cd ~/.claude/skills/telescope && git pull && ./setup
```

## Uninstall

```bash
rm -rf ~/.claude/skills/telescope ~/.claude/skills/telescope-{explore,plan,review,execute,add-feature-plan}
```

## License

MIT
