# BGU AI Agents Assignment — A Coding Agent From Scratch

Build a **ReAct-style coding agent** with no frameworks, no MCP, no magic — just an
LLM, a handful of Python functions, and the loop you write between them. By the end
you'll have an agent that can read, write, run, and search files in a tiny sandbox to
accomplish a goal.

Everything lives in one file: **[`part2/agent.py`](part2/agent.py)**. Fill in every
section marked `# TODO`.

## Setup

This project uses [`uv`](https://docs.astral.sh/uv/) for dependency management and
[Ollama](https://ollama.com/) as the local model provider.

1. Install `uv` (see the link above) and [Ollama](https://ollama.com/download).
2. Pull the model the agent uses:

   ```bash
   ollama pull granite4:micro
   ```

   Make sure Ollama is running (`ollama serve`, or the desktop app).

## Run it

```bash
uv run python part2/agent.py
```

`uv` creates the virtual environment and installs dependencies (`openai`) on the
first run automatically.

## What you implement

Open `part2/agent.py` and fill in the three `# TODO` sections:

1. **`dispatch_tool(name, args)`** — route a tool name to its Python function.
2. **`run_agent(goal)`** — the ReAct loop (think → act → observe → repeat).
3. **Register `search_files`** — add its JSON Schema to `TOOLS` and its entry to
   `TOOL_FN` so the model can call it.

Each task writes its trajectory to `part2/traces/task_<N>.json` — submit those
alongside your `agent.py`.
