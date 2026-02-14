# Polar Integration Skill

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill for integrating [Polar](https://polar.sh) payments into web projects.

## What it does

Guides you through the full Polar integration: checkout, webhooks, subscriptions, customer portal, and more. Works with Next.js, SvelteKit, Supabase, Express, and any JS/TS or Python project.

## Install

```
/install-skill https://github.com/bbssppllvv/Polar-Integration-Skill
```

## Triggers

The skill activates when you ask things like:

- "Add Polar payments to my project"
- "Set up Polar checkout"
- "Configure Polar webhooks"
- "Integrate Polar subscriptions"

## Structure

```
SKILL.md              — Main skill definition and integration workflow
references/           — 23 detailed docs covering the full Polar platform
  ├── checkout-*.md   — Checkout links, API, embedded
  ├── webhooks-*.md   — Setup, local dev, delivery handling
  ├── nextjs.md       — Next.js integration
  ├── sveltekit.md    — SvelteKit integration
  ├── supabase.md     — Supabase integration
  ├── products.md     — Products, pricing, benefits
  └── ...             — Auth, OAuth, trials, discounts, sandbox, etc.
```
