# OpenClaudeDuet

**An open-source alternative to Claude Duet — a proactive AI agent that watches your apps, briefs you every morning, and asks before it acts. On YOUR machine, with any model.**

Claude Duet launched on 2026-10-21 as a research preview: US-only, waitlist-gated, limited to Claude Pro/Max plans, and it runs on Anthropic's cloud computer with Anthropic's model. If you are outside the US, on the waitlist, or simply want your agent's memory and credentials to stay on your own machine, there is no Duet for you.

OpenClaudeDuet is that Duet — open, local, and model-agnostic.

> Status: **early build**. This repo starts as the product spec and roadmap. The runtime is being built as "Duet mode" inside [OpenGrokBot](https://github.com/nickylin/OpenGrokBot), so the two projects share one kernel instead of drifting into two codebases. First building block already shipped there: interactive form cards + secret fields that never reach the model ([OpenGrokBot PR #1](https://github.com/nickylin/OpenGrokBot/pull/1)).

## What Duet does (the official list)

- Connects to your apps (Gmail, Slack, calendars, Stripe, …) and **monitors them for you**
- Sends a **short morning brief** on what needs your attention
- **Checks in during the day** only when something actually needs you
- Does real work in the background — scheduling, drafts, forecasts, even building web apps
- **Asks for approval** before sending a message, making a payment, signing, or deleting anything
- Keeps working when you close the laptop (cloud computer)

## What OpenClaudeDuet keeps, and what it changes

| | Claude Duet | OpenClaudeDuet |
|---|---|---|
| Availability | US, Pro/Max, waitlist | Anyone, anywhere |
| Runs on | Anthropic's cloud computer | Your own machine |
| Model | Opus 5.5 only | Any model — Claude API, OpenAI-compatible, DeepSeek, Codex/Cursor CLI, Ollama |
| Connectors | 6,000+ via Claude Marketplace | MCP servers + local adapters (Gmail first) |
| Memory | Anthropic account | Markdown files on your disk, per Bot |
| Secrets | Held by the platform | A local vault (0600); the model only ever sees masked references |
| Background checks | Count against your plan limits, their rules | Your own daily budget guard, your rules |
| Price | Bundled in Pro/Max | Free, MIT |

## The proactive layer (what makes it a Duet, not a chatbot)

1. **Watchers** poll each connected app on an interval you set, with per-connector checkpoints so nothing is read twice.
2. **An attention inbox** collects what the watchers find. Rules triage first; the model only classifies what rules can't — under a daily background-token budget with quiet hours, so the agent never silently burns your quota (the official product counts background checks against your plan; here the budget is explicit and yours).
3. **A morning brief** routine folds the inbox into one short summary at the time you choose.
4. **Approval & form cards** are the only way anything leaves the machine: send, pay, sign, delete — each waits for your tap, using the same card system as OpenGrokBot's shell approvals.
5. **Secrets stay local.** Connector credentials are collected through secret form fields into the local vault; neither your chat history nor the model ever sees them.

Design details: [docs/connectors-design.md](docs/connectors-design.md) · Roadmap: [ROADMAP.md](ROADMAP.md)

## Relationship to OpenGrokBot

[OpenGrokBot](https://github.com/nickylin/OpenGrokBot) is the open-source Grok Bot alternative from the same author: a roster of named Bots with memory, a shared workspace, shell approvals, routines, and a local web/desktop UI. Grok Bot's shape is a *team*; Duet's shape is a *single proactive partner*. OpenClaudeDuet is that partner mode on the same kernel:

- Same runtime, same data directory (`~/.opengrokbot/`)
- Default roster of one ("Duet"), proactive defaults on
- Same harness switcher — bring whatever model you already pay for, or run one locally

## Disclaimer

OpenClaudeDuet is an independent open-source project, not affiliated with Anthropic. "Claude" and "Claude Duet" are trademarks of Anthropic; this project is an alternative implementation of a similar idea, in the same spirit as OpenGrokBot is to Grok Bot.

License: [MIT](LICENSE).
