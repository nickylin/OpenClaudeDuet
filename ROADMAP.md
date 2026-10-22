# Roadmap

OpenClaudeDuet is built as "Duet mode" on the [OpenGrokBot](https://github.com/nickylin/OpenGrokBot) kernel — one codebase, two product shapes. Items land in whichever repo owns the layer; this file tracks the Duet-specific line.

## P0 — Position (done, 2026-10-22)

- This repo: name, positioning README, connector-layer design draft.
- Kernel building block shipped in OpenGrokBot: interactive form cards + secret fields that never reach the model (PR #1) — the mechanism Duet mode uses both to collect connector credentials and to ask for approvals.

## P1 — Duet mode in the kernel

- A first-run profile that seeds a single Bot named "Duet" (instead of the starter team roster), proactive defaults on.
- Attention inbox: storage, dedupe index, and inbox cards in the Duet thread (see docs/connectors-design.md §5).
- Morning brief: a default scheduled routine that reads the inbox and posts one summary at a user-chosen time.
- Budget guard + quiet hours settings for all background model calls.

## P2 — First connectors

- Gmail watcher (hand-written adapter, historyId checkpoint, OAuth token in the vault).
- MCP client (stdio) + poll wrapper, proven against one filesystem and one Gmail MCP server.
- Local folder watcher as the zero-account dogfood path.

## P3 — More sources, more reach

- Calendar and Slack adapters (or via MCP where the servers are good enough).
- Drafts prepared for approval (reply drafts, payment reminders) — always behind approval cards.
- Windows/Linux packaging parity with the macOS desktop app.

## Explicit non-goals

- No cloud relay: if your machine is asleep, your Duet is asleep. (A tiny always-on box you own is the intended workaround, not our servers.)
- No sending/ paying / signing / deleting without an approval card. Ever.
- No telemetry.
