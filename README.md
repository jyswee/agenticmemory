# agmry

[![npm version](https://img.shields.io/npm/v/agmry.svg)](https://www.npmjs.com/package/agmry)
[![MCP](https://img.shields.io/badge/MCP-14_tools-blue)](https://agenticmemory.ai/quickstart)
[![Remote MCP](https://img.shields.io/badge/remote_MCP-mcp.agenticmemory.ai-8b5cf6)](#remote-mcp--zero-install)

**Persistent memory for AI agents — conversation history, key-value context, and semantic search across sessions. As easy as git.**

> **git remembers your code. agmry remembers everything else.**

Your agent writes great code all day — then forgets every decision the moment its context window resets. So *you* become the memory layer: re-explaining the project, the preferences, what broke last time. Agentic Memory is the memory your agent runs itself: one install, and it stores, recalls, and searches its own state across sessions, machines, and even other agents. It can even **sign itself up** — one command returns a working API key, no browser, no human.

**Works with:** Claude Code · Cursor · Cline · Windsurf · Aider · Codex · any MCP client

## Install

```bash
npm install -g agmry
```

## Quick Start

```bash
# Agent signs itself up — working key returned instantly, 7-day trial starts now
agmry signup my-project --local

# Store memory
agmry store my-project "User prefers TypeScript strict mode and pnpm"

# New session? Recall everything
agmry recall my-project

# Half-remember something? Search it
agmry search my-project "how do we deploy"

# Full reference
agmry --help
```

## Agents sign themselves up

No browser. No OAuth. No waiting for a human.

```
$ agmry signup my-project
✓ Project "my-project" created
  API key: amk_************************  (saved to .agmry/config.json)
  Trial:   fully active for 7 days — $0 today
```

The key works immediately — every endpoint, every tool. Add `--email you@company.com` and the human gets a card-setup link to continue past the trial. Until then, the agent just works.

## Three tiers of memory — because not all memory is equal

A transcript dump isn't memory. Agents need different recall for different things:

```bash
# Short-term: ordered, role-aware conversation history (sub-ms reads)
agmry store my-project "Deployed v2.1, rolled back — migration locked the users table" -r assistant
agmry recall my-project -n 50

# Durable facts: typed key-value context — decisions, preferences, runbooks
agmry ctx my-project set deploy_flow "push image to registry, then ask infra to roll"
agmry ctx my-project get deploy_flow

# Long-term: titled, tagged knowledge entries that survive months
agmry entry my-project "Auth decision" "JWT not sessions — mobile clients can't hold cookies" --tags "arch,decisions"
agmry entries my-project --tags "decisions"

# Work-in-progress: scratchpad that's allowed to expire
agmry scratch my-project set "midway through the billing refactor, invoice.js next"
```

And semantic search stitches it together when the agent only half-remembers:

```bash
agmry search my-project "did we ever discuss rate limiting"
# → surfaces a months-old entry, with similarity score
```

## Session start = one command

Instead of pasting yesterday's summary into today's prompt:

```bash
agmry boot my-project --json          # messages + context + entries, one call
agmry boot my-project --semantic "billing refactor"   # or focused on a topic
```

~200 tokens of structured state, not top-k chunks of old transcripts.

## MCP Server

Prefer tools over a CLI? `agmry` ships an MCP server. Point Claude Code (or any MCP client) at it and your agent gets **17 native tools**: store, recall, search, bootstrap, context, entries, spaces, queues.

```bash
claude mcp add agenticmemory -- agmry mcp-serve
```

For clients that use a JSON config (Cline, Cursor, Windsurf), pass your API key via the environment — the MCP server runs outside your project directory, so it won't pick up `.agmry/config.json`:

```json
{
  "mcpServers": {
    "agenticmemory": {
      "command": "agmry",
      "args": ["mcp-serve"],
      "env": { "AGMRY_API_KEY": "amk_your_key_here" }
    }
  }
}
```

### Remote MCP — zero install

Claude Web, Claude Desktop, Raycast, or any hosted MCP client can connect straight to the remote server. Same tools, same API key, nothing to install:

```
URL:  https://mcp.agenticmemory.ai/sse
Auth: Authorization: Bearer YOUR_API_KEY
```

## End-to-end encryption (zero-knowledge spaces)

Create a space the server can never read. Encryption happens inside the CLI —
the API only ever sees ciphertext.

```bash
agmry key generate                                  # one-time: creates + saves your key
agmry space create "Private" private --encryption e2e

agmry store SPACE "my secret"                       # encrypted before it leaves your machine
agmry recall SPACE                                  # transparently decrypted
agmry key verify SPACE                              # holding the right key?
```

Or derive per-space keys from a passphrase instead of storing a key:

```bash
agmry key set --passphrase "long secret phrase"
```

Notes:
- Semantic search is impossible on zero-knowledge spaces by design.
- Prefer **encrypted at rest** instead? `--encryption managed` — the server encrypts your data at rest and every feature (search, summaries) keeps working. No client key needed.
- Losing the key or passphrase means the data is unrecoverable. That's the point. Back it up: `agmry key show --reveal`.

Key resolution order: `--enc-key` / `--passphrase` flag → `AGMRY_ENCRYPTION_KEY` env → `./.agmry/config.json` → `~/.agmry/config.json`. Same envelope and key derivation as the Node and Python SDKs — keys are interchangeable across all three.

## Multi-agent: one brain, many agents

Spaces are shareable. One agent stores the deploy runbook; another recalls it a week later, from a different machine, over a different interface (CLI, MCP, or REST — same memory). Your agents hand off between sessions and between projects without you couriering context.

## Queues — the agent bus

FIFO queues inside a space. One agent pushes work, another pops it — no polling glue, no extra infra.

```bash
agmry queue SPACE jobs push '{"task":"review PR #42"}'   # enqueue (FIFO)
agmry queue SPACE jobs pop                                # dequeue oldest — exit code 2 if empty
agmry queue SPACE jobs pop --wait 25                      # long-poll up to 25s for the next item
agmry queue SPACE jobs                                    # peek: length + head, without consuming
agmry queue SPACE jobs dlq push '{"task":"..."}' --reason "failed twice"   # dead-letter
agmry queue SPACE jobs dlq list                           # inspect dead-lettered items
```

Exit code 2 on empty means shell loops branch cleanly: `while agmry queue SPACE jobs pop --wait 25 --json; do ...; done`. Envelopes are opaque JSON — the server never inspects them.

## Agent Integration

Add to your CLAUDE.md, .cursorrules, .clinerules, .windsurfrules, or AGENTS.md:

```
## Agentic Memory
This project uses Agentic Memory for persistent memory across sessions.
Use the `agmry` CLI. Key is in .agmry/config.json (auto-loaded).

agmry boot SPACE --json               # load everything at session start
agmry store SPACE "what happened"     # remember something
agmry ctx SPACE set key "value"       # store a durable decision/fact
agmry search SPACE "the deadline"     # find past context
agmry queue SPACE jobs push '{...}'   # send work to another agent
agmry queue SPACE jobs pop --wait 25  # receive work (exit 2 = empty)
```

## Config Priority

1. `--key` flag
2. `AGMRY_API_KEY` environment variable
3. `AGENTICMEMORY_API_KEY` environment variable
4. `./.agmry/config.json` (project-local)
5. `~/.agmry/config.json` (global)

Add `.agmry/` to your `.gitignore`.

## Features

- **Conversation history** — ordered, role-aware messages with recency windowing, sub-ms reads
- **Key-value context** — typed durable facts: decisions, preferences, runbooks
- **Long-term entries** — titled, tagged knowledge that survives months, with auto-summarisation
- **Entities** — people and systems the agent should know about
- **Scratchpad** — ephemeral working memory with TTLs (expiry is a feature)
- **Semantic search** — across everything the agent has ever stored
- **Bootstrap** — full session context in one call
- **Agent self-signup** — working API key from one CLI command, 7-day trial starts instantly
- **Queues** — FIFO agent bus with long-poll and dead-letter, `agmry queue` (exit 2 = empty)
- **MCP server** — 17 tools, local (`agmry mcp-serve`) or fully remote (`mcp.agenticmemory.ai`)
- **REST API** — same memory on the request path of proxies and pipelines
- **Multi-agent spaces** — a fleet of agents reads and writes one memory
- **End-to-end encryption** — zero-knowledge spaces where only you hold the key (`agmry key generate`)
- **Export** — full data takeout per space, one command (`agmry space export`)

**Pricing:** 7-day free trial — the agent's key works instantly, $0 today. Continue after the trial from $24.99/mo. No data deletion on expiry: your memory waits for you. [Details](https://agenticmemory.ai/pricing).

## Why not just Claude's built-in memory?

Claude Memory is great — but it only works with Claude, and only Claude decides what's in it.

| | Agentic Memory | Claude Memory |
|---|---|---|
| **LLM support** | Any — GPT, Claude, Gemini, Llama, Mistral, DeepSeek | Claude only |
| **Switch LLMs** | Keep all memory | Lose everything |
| **Multi-agent** | Shared spaces across agents | Single user |
| **Programmatic control** | Full CLI + MCP + REST | Black box — Claude decides |
| **Structured data** | Messages + context + entries + entities + scratchpad | Free-text notes |
| **Semantic search** | `agmry search "the deadline"` | No search API |
| **Data ownership** | You own it, export any time | Stored by Anthropic |

## Why this exists

I was my agents' memory. Every session started with me re-explaining my own project to my own tools — decisions, preferences, what broke last time. So I built the memory they run themselves. My own agents use it in production across a dozen projects; if something's rough or missing, [open an issue](https://github.com/jyswee/agmry/issues) — I read every one.

## Documentation

- [Quickstart guides](https://agenticmemory.ai/quickstart) — Claude Code, Cursor, LangChain, CrewAI, AutoGen + 7 more
- [API docs](https://agenticmemory.ai/docs)
- [Pricing](https://agenticmemory.ai/pricing)

## License

Proprietary — Tyga.Cloud Ltd. See [LICENSE](./LICENSE).
