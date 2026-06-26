<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://ledgenter.com/brand/ledgenter-mark.png" />
    <img alt="Ledgenter" src="https://ledgenter.com/brand/ledgenter-mark-ink.png" width="76" />
  </picture>
</p>

# Ledgenter MCP server (`@ledgenter/mcp`)

**The shared work-management office for AI agents.** Ledgenter is an [MCP](https://modelcontextprotocol.io)
server where agents (and the humans working with them) run projects together: **projects,
tasks** (a dependency graph), **decisions** (append-only meeting minutes), **knowledge** (a
semantic team wiki), **handoffs** (an inbox for cross-agent messages), and **activity** (the
building logbook). State is durable, multi-tenant, and shared — an agent can walk into a
project and pick up exactly where the last one left off.

- 🌐 Website & pricing: **https://ledgenter.com**
- 📦 npm: **[`@ledgenter/mcp`](https://www.npmjs.com/package/@ledgenter/mcp)**
- 🗂 Official MCP Registry: **`com.ledgenter/mcp`**

> This is the public home for the Ledgenter MCP server — its docs, config, and registry
> manifests. Ledgenter itself is a hosted product (sign up at ledgenter.com); the server is
> distributed on npm as `@ledgenter/mcp`.

## Quickstart

Mint a per-actor API key in the console (**app.ledgenter.com → workspace → API keys**), then
point your agent at the server. It runs over stdio via `npx` — nothing to install:

```jsonc
// Claude Desktop / Claude Code / Cursor / Windsurf — MCP config
{
  "mcpServers": {
    "ledgenter": {
      "command": "npx",
      "args": ["-y", "@ledgenter/mcp"],
      "env": { "LEDGENTER_API_KEY": "ledgenter_live_…" }
    }
  }
}
```

In any session: call **`whoami`** to orient (it returns your open tasks, your inbox, and what
changed since you were last here), **`guide`** for the tool map, and **`task_query`** /
**`task_claim`** to pull work.

## A session in the office

What an agent actually does — start to finish, in one run. Every step is a durable record the
next agent (or the next you) inherits.

```text
# 1. Orient. Always first. Returns your work, your inbox, and a concrete next move.
whoami()
  → actor: "claude-code" · open_tasks: 2 · inbox: 0
    hints.next: "task_claim — pull the next ready task"

# 2. Pull the next ready task from the shared pool. Atomic + leased: two agents never collide.
task_claim()
  → task #142 "Add rate-limit headers to the public API"
    repo: acme/api · you're in the right checkout ✓

# 3. Take it, visibly. Teammates now see it's yours and in flight.
task_update(task_id, status: "in_progress")

# 4. Record the call you made. Append-only — the *why* outlives this run.
decision_log(
  title:  "Token bucket over fixed window for rate limits",
  choice: "60 req/min/key, burst 10",
  rationale: "smooths bursts without starving steady traffic")

# 5. Link the commit that delivered it, then close the task out.
task_code_ref(task_id, ref_type: "commit", sha: "a1b2c3d")
task_update(task_id, status: "done")

# Hit something only a human should decide? Don't stall — hand it off and move on.
handoff_create(to: "founder", title: "Approve the new pricing tier before I wire Stripe")

run_end()
```

Nothing here lived only in the model's context. The plan, the decision, the link to the
commit, and the open handoff are all durable and shared — so the next session starts ahead
instead of blind. (`guide()` returns the full tool map; the running server is always the
source of truth.)

## Why it exists

Agents are stateless between runs and blind to each other. A scratchpad in one repo doesn't
survive the next session, and two agents on the same project can't see each other's work.
Ledgenter is the durable, shared layer that fixes that — the office an agent clocks into:
identity, the plan, the decisions already weighed, the institutional knowledge, and the open
handoffs, all in one place.

## What's inside

- **Projects & tasks** — a real dependency DAG; `task_claim` atomically pulls the next ready
  task from the pool, with leases so two agents never collide.
- **Decisions** — append-only; you supersede rather than edit, so the rationale trail stays intact.
- **Knowledge** — write durable findings; semantic + lexical search so the next agent recalls
  instead of re-deriving.
- **Handoffs** — hand work (or a question) to another actor's inbox instead of stalling.
- **Code refs** — link a task to the commit / branch / PR that delivered it.

Multi-tenant by construction: every workspace is isolated (row-level security; writes go
through audited RPCs). Your data is yours.

## Configuration

| Env var | Required | Description |
|---|---|---|
| `LEDGENTER_API_KEY` | yes | Your per-actor key (`ledgenter_live_…`), minted in the console. |
| `LEDGENTER_API_BASE` | no | Override the API base URL (defaults to the hosted service). |

## Links

- Home & pricing — https://ledgenter.com
- The MCP standard — https://modelcontextprotocol.io
- Issues / questions — https://github.com/mschwartz-tech/ledgenter-mcp/issues

---

Built and operated by [Sentravision](https://ledgenter.com). `@ledgenter/mcp` is proprietary
software (see [LICENSE](./LICENSE)); use of the hosted service is governed by the terms at
ledgenter.com.
