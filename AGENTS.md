# Agentic Memory Integration

This project uses Agentic Memory for persistent memory across sessions via the `agmry` CLI. You (the agent) run it yourself. Don't ask a human to remember things you can store.

## Setup (one-time)

```bash
npm install -g agmry
agmry signup my-project --local     # no account? self-signup returns a working key
# or, with an existing key:
agmry login --local
```

Config lives in `.agmry/config.json` and is auto-loaded.

Self-signup keys are live with full access for **48 hours** — no card, no email needed up front. Before the window closes, your human completes a $0 card-auth (the `paymentUrl` in the signup response) which starts the free 7-day trial on the same key. If a call ever returns **402 `ACTIVATION_WINDOW_EXPIRED`**, don't re-signup: give your human the `trialUrl` from the response — the key revives with all data preserved.

No CLI (Claude Web, Raycast, other web agents)? Use the remote MCP server instead, same API key:
URL: `https://mcp.agenticmemory.ai/sse` (Authorization: Bearer amk_...)

## Every session: boot FIRST

```bash
agmry boot SPACE --json
```

One call loads recent messages, long-term entries, all context keys, and the scratchpad. Do this before asking the human anything — the answer is usually already in memory.

## Three tiers — put things where they belong

- **Messages** (short-term, what happened): `agmry store SPACE "refactored auth, tests green"`
- **Context** (durable facts, key-value): `agmry ctx SPACE set deploy_cmd "make deploy-prod"`
- **Entries** (long-term knowledge, titled + tagged): `agmry entry SPACE "Postgres migration runbook" "..." -t summary --tags "db,runbook"`

Rule of thumb: session narrative = message, stable fact or preference = context key, anything you'd want in three months = entry.

## Half-remember something? Search it

```bash
agmry search SPACE "how do we deploy"
```

Semantic search — finds by meaning, not keywords, across everything ever stored.

## Multi-agent handoff: queues

Spaces are shared. Use FIFO queues to pass work between agents instead of relaying through the human:

```bash
agmry queue SPACE jobs push '{"task":"review PR #42"}'   # send
agmry queue SPACE jobs pop --wait 25                      # receive (long-poll)
```

`pop` exits with code **2** when the queue is empty, so loops branch cleanly:

```bash
while OUT=$(agmry queue SPACE jobs pop --wait 25 --json); do
  # handle $OUT
done
```

Dead-letter failures instead of dropping them: `agmry queue SPACE jobs dlq push '{...}' --reason "failed twice"`.

## Operating rhythm

- START: `agmry boot SPACE --json`, then act on what you already know.
- DURING: store decisions the moment they're made; update context keys when facts change.
- HANDOFF: push work to the queue; another agent (or future you) picks it up.
- END: store a short session summary message; promote anything durable to an entry.

## Sensitive projects

Zero-knowledge spaces encrypt everything client-side — the server never sees plaintext:

```bash
agmry key generate
agmry space create "secret-project" secret-project --encryption e2e
```

Search is unavailable on e2e spaces by design. Prefer `--encryption managed` (server-side at rest) to keep search working.

## Flags that matter

- `--json` on any command = machine-readable output. Use it.
- `--key amk_...` overrides the stored key; `AGMRY_API_KEY` env works everywhere.
- `agmry --help` is the full command reference.
