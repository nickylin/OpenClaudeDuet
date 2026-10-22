# Connector layer design (draft, 2026-10-22)

How OpenDuet watches your apps without becoming a quota bonfire or a credential leak. Written against the OpenGrokBot kernel (scheduler, cards, secrets vault), which Duet mode inherits.

## 1. Shape

```
connector adapter ──poll──▶ delta events ──▶ triage (rules → model) ──▶ attention inbox
                                                                          │
                        morning brief routine ◀── aggregates ───────────┤
                                                                          ▼
                                              approval / form cards ──▶ user decision
```

A connector is a poll-based adapter. Push/webhook sources can be added later behind the same delta interface, but polling is the lowest common denominator every source supports (IMAP, Gmail API, RSS, MCP servers, local folders).

## 2. Connector contract

```ts
type ConnectorAdapter = {
  id: string;                    // "gmail", "calendar", ...
  displayName: string;
  defaultIntervalMin: number;    // user-overridable
  // Read the vault key names this connector needs (never the values at this layer).
  requiredSecrets: string[];     // e.g. ["connector.gmail.oauth_refresh"]
  poll(ctx: { checkpoint: unknown, secrets: SecretResolver }): Promise<{
    events: AttentionEvent[];
    checkpoint: unknown;         // opaque, persisted verbatim (last id / timestamp / etag)
  }>;
};

type AttentionEvent = {
  connectorId: string;
  externalId: string;            // dedupe key, together with connectorId
  title: string;
  summary: string;               // plain text, already secret-scrubbed by the adapter
  url?: string;
  receivedAt: string;
  // Set by triage, not by the adapter:
  urgency?: "now" | "brief" | "drop";
};
```

Rules:

- Adapters return deltas only, keyed off their checkpoint. The checkpoint is stored per connector in `~/.opengrokbot/connectors/<id>.json` next to the events log.
- Adapters never write to chat directly. Everything goes through the attention inbox, so dedupe, budgets and quiet hours apply uniformly.
- Event text must be scrubbed by the adapter before it leaves (no auth headers, no tokens, no full email bodies beyond the summary the user opted into).

## 3. Sources, in order

1. **MCP servers (stdio)** — the strategic bet. Claude's connector ecosystem is MCP-shaped; a local MCP client lets OpenDuet reuse community servers (Gmail, Calendar, Slack, GitHub…) instead of hand-writing OAuth flows per service. The poll wrapper calls the server's read tools on the connector's interval.
2. **Gmail via API + OAuth** — the one hand-written adapter, because email is the highest-value watcher and deserves a first-class checkpoint (historyId) instead of polling a generic MCP list endpoint. OAuth refresh token lives in the vault.
3. **Local sources** — a watched folder, an RSS feed, a webhook inbox file. Cheap, private, and good dogfood for the pipeline before any account is connected.

## 4. Triage and the budget guard

The official Duet counts background checks against your plan limits. OpenDuet makes the cost explicit instead:

- Stage 1, rules (free): sender/domain allow- and block-lists, keyword rules, "only events newer than X", per-connector mute. Most noise dies here.
- Stage 2, model (metered): only events that pass rules get classified into now / brief / drop, with a one-line reason. Every classification is charged to a **daily background budget** (default: a token cap the user sets); when the budget is exhausted, events queue for the morning brief unclassified instead of failing or silently spending.
- Quiet hours: no "now" notifications outside the user's window; they fold into the brief.

## 5. Attention inbox

- Storage: append-only JSONL per day under `~/.opengrokbot/attention/`, plus a small index of seen (connectorId, externalId) pairs for dedupe.
- In chat, the inbox surfaces as cards in the Duet thread: "now" items arrive as they are found (with approval/form cards when action is needed); "brief" items appear only inside the morning brief.
- The morning brief is a scheduled routine (the kernel's scheduler already exists) whose prompt reads the inbox — no separate brief engine.

## 6. Credentials

- Collected only through secret form fields (shipped in OpenGrokBot PR #1) into `~/.opengrokbot/secrets.json` (mode 0600). OAuth flows: the user pastes the resulting refresh token into a secret field, or completes the flow in the browser and the local server catches the loopback redirect — the token still lands only in the vault.
- Adapters receive a `SecretResolver` that hands a value to the exact outbound call that needs it. Values are never placed in prompts, event text, logs, or the transcript — the same redaction contract as form secrets, enforced in the same places.

## 7. Open questions

- MCP polling semantics differ per server (some expose search, some only list). The adapter layer may need a per-server "query template" config; prototype with the Gmail and filesystem servers first.
- Multi-account (two Gmail accounts) — connector instances (`gmail:work`, `gmail:personal`) rather than multi-account adapters; checkpoints and secrets are per instance.
- Whether "now" items may auto-draft replies (draft only, never send) as part of triage — default no; sending always waits for an approval card.
