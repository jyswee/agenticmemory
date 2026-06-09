# agmry — Agentic Memory CLI

Persistent memory for AI agents. As easy as git.

```bash
npm install -g agmry
```

## Quick Start

```bash
# Setup (agent runs once, human pays $1 via emailed link)
agmry signup my-project --email joe@company.com --local

# Store memory
agmry store SPACE_ID "User prefers dark mode"
agmry store SPACE_ID "Deployed v2.1 to production" -r assistant

# Recall recent messages
agmry recall SPACE_ID
agmry recall SPACE_ID -n 50 --json

# Semantic search
agmry search SPACE_ID "what was the deadline"

# Key-value context
agmry ctx SPACE_ID set user_prefs '{"theme":"dark","lang":"en"}'
agmry ctx SPACE_ID get user_prefs

# Long-term entries
agmry entry SPACE_ID "Project architecture" "Monorepo with 3 services" --tags "arch,decisions"
agmry entries SPACE_ID --tags "decisions"

# Load everything at session start
agmry boot SPACE_ID --json
```

## Agent Instructions

Add to your agent instructions (`.cursorrules`, `.clinerules`, `.windsurfrules`, etc.):

```
## Agentic Memory
Use the agmry CLI for persistent memory across sessions.
Key is in .agmry/config.json (auto-loaded).

agmry store SPACE "what happened"     # remember something
agmry recall SPACE                     # what was said recently
agmry search SPACE "the deadline"     # find past context
agmry ctx SPACE set key "value"       # store structured data
agmry boot SPACE --json                # load everything at session start
```

## Config Priority

1. `--key` flag
2. `AGMRY_API_KEY` environment variable
3. `AGENTICMEMORY_API_KEY` environment variable
4. `./.agmry/config.json` (project-local)
5. `~/.agmry/config.json` (global)

## All Commands

Run `agmry --help` for the full reference.

| Command | Description |
|---------|-------------|
| `agmry signup <slug>` | Create account ($1 verification) |
| `agmry login` | Save API key |
| `agmry spaces` | List memory spaces |
| `agmry space create "name" slug` | Create space |
| `agmry store <SPACE> "msg"` | Store message |
| `agmry recall <SPACE>` | Recall messages |
| `agmry search <SPACE> "query"` | Semantic search |
| `agmry ctx <SPACE> set key "val"` | Set context |
| `agmry ctx <SPACE> get key` | Get context |
| `agmry entry <SPACE> "title" "content"` | Add long-term entry |
| `agmry entries <SPACE>` | List entries |
| `agmry entity <SPACE> "name"` | Add entity |
| `agmry scratch <SPACE>` | Get scratchpad |
| `agmry boot <SPACE>` | Bootstrap full context |
| `agmry status` | Dashboard overview |
| `agmry health` | API health check |

## Flags

- `--json` — Machine-readable JSON output (for agent parsing)
- `--key KEY` — Override API key
- `--url URL` — Override base URL (default: https://agenticmemory.ai)

## 3-Tier Memory

1. **Short-term** (messages) — sub-millisecond reads
2. **Medium-term** (search) — semantic search across past conversations
3. **Long-term** (entries) — auto-summarised knowledge across months

## Why Agentic Memory vs Claude Memory

Claude Memory is great — but it only works with Claude. If you switch LLMs, you lose everything.

| | Agentic Memory | Claude Memory |
|---|---|---|
| **LLM support** | Any — GPT, Claude, Gemini, Llama, Mistral, DeepSeek | Claude only |
| **Switch LLMs** | Keep all memory | Lose everything |
| **Multi-agent** | Shared memory across agents | Single user |
| **Data ownership** | You own it. EU / US / regional. | Stored by Anthropic (US) |
| **Programmatic control** | Full CLI + API | Black box — Claude decides |
| **Structured data** | Messages + KV context + entries + entities + scratchpad | Free-text notes |
| **Semantic search** | `agmry search "the deadline"` | No search API |
| **Agent-to-agent sharing** | World memory across agents | Not possible |
| **Cost** | $1 one-time, then free | Bundled in Claude Pro ($20/mo) |

**"Claude Memory works for Claude. Agentic Memory works for everything."**

## MCP Server

Run `agmry mcp-serve` to start a stdio MCP server. Add to your `.mcp.json`:

```json
{
  "mcpServers": {
    "agenticmemory": {
      "type": "stdio",
      "command": "agmry",
      "args": ["mcp-serve"]
    }
  }
}
```

14 tools: `memory_store`, `memory_recall`, `memory_search`, `memory_bootstrap`, `context_set`, `context_get`, `entry_add`, `entries_list`, `spaces_list`, `spaces_create`, and more.

## Links

- [Documentation](https://agenticmemory.ai/docs)
- [Quick Start Guides](https://agenticmemory.ai/quickstart)
- [Pricing](https://agenticmemory.ai/pricing) — $1 one-time, then free. Pro $24.99/mo.

---

Built by [Tyga.Cloud Ltd](https://tyga.cloud). Copyright 2024-2026.
