# BGU AI Agents Assignment — Build an Agent From Scratch

Build a **ReAct-style agent** with no frameworks and no magic — just an LLM, a
handful of tools, and the loop you write between them. You write that loop once
in **Part 2**, then in **Part 3** you point the *same* loop at a live server over
MCP. The lesson: the loop doesn't change — only where the tools live.

| Part | What you build | Tools live in… |
|------|----------------|----------------|
| **[Part 2](part2/agent.py)** | A ReAct coding agent that reads, writes, runs, and searches files in a tiny sandbox | plain Python functions in your file |
| **[Part 3](part3/agent.py)** | The same loop wired to a **live stock exchange** over MCP | a remote **MCP server** + one local Python tool |

## Setup

This project uses [`uv`](https://docs.astral.sh/uv/) for dependency management and
[Ollama](https://ollama.com/) as the local model provider.

1. Install `uv` (see the link above) and [Ollama](https://ollama.com/download).
2. Pull the model the agent uses:

   ```bash
   ollama pull granite4:micro
   ```

   Make sure Ollama is running (`ollama serve`, or the desktop app).

`uv` creates the virtual environment and installs dependencies on the first run
automatically.

---

## Part 2 — A coding agent from scratch

Everything lives in one file: **[`part2/agent.py`](part2/agent.py)**. Fill in every
section marked `# TODO`.

### Run it

```bash
uv run python part2/agent.py
```

### What you implement

Open `part2/agent.py` and fill in the three `# TODO` sections:

1. **`dispatch_tool(name, args)`** — route a tool name to its Python function.
2. **`run_agent(goal)`** — the ReAct loop (think → act → observe → repeat).
3. **Register `search_files`** — add its JSON Schema to `TOOLS` and its entry to
   `TOOL_FN` so the model can call it.

Each task writes its trajectory to `part2/traces/task_<N>.json` — submit those
alongside your `agent.py`.

---

## Part 3 — Wire your loop to a LIVE MCP exchange

Same loop, real server. In Part 3 most tools live on a **server** your agent
discovers and calls over **MCP** (Model Context Protocol), with one tool staying
a plain Python function in your file. The loop treats both the same.

And the server is **real**: you trade against
[`agent-stocks.vercel.app`](https://agent-stocks.vercel.app) — a live exchange
with **real US-market prices**, a **shared leaderboard**, and **$100,000** of
pretend cash. Nothing is deterministic, so your agent must read what *actually*
happened and never fake a fill.

> **You're graded on the agent, not the profit.** What counts is the whole flow:
> it connects over MCP, discovers and calls tools, feeds observations back, makes
> and explains decisions, handles errors honestly, and logs faithful traces.

Part 3 has two sections:

- **Part 3A** — explore the exchange **by hand** with the official
  [MCP Inspector](https://github.com/modelcontextprotocol/inspector) (no code; a
  human-graded worksheet). Connect with Transport **Streamable HTTP**, URL
  `https://agent-stocks.vercel.app/api/mcp`, and an `X-API-Key` header.
- **Part 3B** — wire your `run_agent` loop to it and run the instructor's goals
  against the live exchange.

### The exchange — quick reference

**MCP endpoint:** `https://agent-stocks.vercel.app/api/mcp` (Streamable HTTP).
Every call needs an `X-API-Key: ax_...` header. Prices are in **cents**
(`29115` = $291.15), quantities are whole shares, every trade pays a **0.05%
fee**, orders **fill instantly**, and there's **no short selling**. The
list-returning tools wrap results in `{"items": [...]}`.

| Tool | Args | Returns |
|------|------|---------|
| `get_symbols` | — | symbols with `last_cents` |
| `get_quote` | `symbol` | the price you fill at |
| `get_news` | `limit=20` | headlines, newest first |
| `place_order` | `symbol, side, qty` | the fill, **or** `{"error": …}` to handle |
| `get_portfolio` | — | `cash_cents` + positions |
| `get_trades` | `limit=50` | your trade history |
| `get_leaderboard` | — | teams by net worth |

### What you implement

Open **[`part3/agent.py`](part3/agent.py)** and fill in three `# TODO` sections —
the loop is identical to Part 2; only how a tool is dispatched changes:

1. **`mcp_tools_to_openai(mcp_tools)`** — translate the server's tool list
   (`client.list_tools()`) into the OpenAI-compatible tool schema — the same
   shape you wrote by hand in Part 2's `TOOLS`.
2. **`dispatch_tool(name, args, client, mcp_names)`** — route each call to a
   **local** tool (`TOOL_FN[name](**args)`) or an **MCP** tool
   (`await client.call_tool(...)`, then read `result.data`).
3. **`run_agent(goal, client)`** — the observe → think → act loop over the merged
   tool list (`LOCAL_TOOLS + mcp_tools_to_openai(...)`).

The system prompt, the local `pct_change` tool, the goals, the driver, and the
trace writer are all **provided**.

### Run it

```bash
# register your team on https://agent-stocks.vercel.app to get an ax_... key,
# then put it in part3/.env:   AGENTS_EXCHANGE_API_KEY=ax_your_key_here

uv run python part3/agent.py            # run every goal
uv run python part3/agent.py --only 3   # re-run just goal 3 (1-based)
```

> **Rate limit:** the exchange enforces 12 s between trading calls; the driver
> already waits 15 s between goals. Never add a tight retry loop.

### Deliverables

| File | What it is |
|------|-----------|
| `part3/inspector_worksheet.md` (+ screenshots) | Part 3A — hand-exploration of the MCP |
| `part3/agent.py` | your `mcp_tools_to_openai` + `dispatch_tool` + `run_agent` |
| `part3/traces/goal_<N>.json` | one faithful trace per instructor goal |
| `part3/analysis/evaluation.md` | post-run metrics + failure analysis |

> The grader cross-checks your traces against the exchange's own `get_trades`
> history — a trace logging a trade that never happened scores zero. The provided
> `save_trace` keeps them honest by writing exactly what the loop executed.
