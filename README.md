# Engineering Loop

A `CLAUDE.md` workflow that turns an AI coding agent into a structured engineering process — not just a code generator.

## The problem

AI agents are great at writing code. They are bad at managing complexity across a real task: they skip planning, make assumptions, forget what they were doing, and declare success before verifying the goal. The result is fast first drafts and slow debugging.

## What this is

A single `CLAUDE.md` file you drop into your project. It defines a mandatory 9-phase loop that the agent follows for every task — from receiving it to confirming the goal is achieved.

```
Receive → Understand → Decompose → Plan → Align → Execute → Verify → [Gap? → Plan]
```

Key behaviors it enforces:

- Agent confirms understanding before touching any code
- Every task is broken into blocks, each with an explicit plan
- Plans are reviewed for coherence before execution starts
- Execution is verified against the original goal, not just "it runs"
- Infinite retry loops are prevented with a hard escape hatch
- Human confirmation gates can be turned off for autonomous runs

## What it works with

The loop is toolset-agnostic. It works with:

- **Bare Claude Code** — no extra setup required
- **MCP servers** (task trackers, docs, design tools, CI) — the loop tells the agent when and how to use them
- **Superpowers plugin** — skill references (brainstorming, plan-writing, verification) map directly to Superpowers skills; without it, they describe the approach to take manually

## How to integrate

**Option 1 — new project:**
```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/sidan93/claude-eng-loop/main/CLAUDE.md
```

Review the file before using it. Never feed a remotely fetched file directly to an agent without reading it first.

**Option 2 — existing project with its own CLAUDE.md:**

Append the Engineering Loop content after your existing instructions, or add this at the top of your file:

```
# Engineering Loop
[paste CLAUDE.md content here]
```

Your project-specific instructions take priority over the loop if there is a conflict.

**Option 3 — global Claude config:**

Place `CLAUDE.md` in `~/.claude/` to apply the loop to all projects by default. When using this option, **remove the `## Project Context` section** — it has no meaning at the global level. Project-specific context belongs in a per-project `CLAUDE.md` at the repo root.

| Scope | Location | Project Context section |
|-------|----------|------------------------|
| Global (all projects) | `~/.claude/CLAUDE.md` | Remove it |
| Per-project | `<repo-root>/CLAUDE.md` | Fill it in |

Both can coexist: global sets the process, project overrides with specifics. Project-level `CLAUDE.md` takes priority over global when both are present.

## Execution modes

At the start of each task the agent asks once:

- **Interactive** (default) — agent confirms understanding and plan with you before proceeding. Use when requirements are unclear or stakes are high.
- **Autonomous** — agent proceeds without waiting for approvals, stops only on hard blockers. Use when you want to step away and let it run.

**For autonomous runs, also enable Claude's Auto mode** — this allows Claude to accept tool calls without asking for permission on each one. In Claude Code: press `Shift+Tab` to cycle to Auto mode.

> **Security note:** Auto mode gives the agent permission to read and modify files and run shell commands without confirmation. Only use it in isolated environments (a dedicated repo, a VM, a container) or when you understand exactly what the agent will do. Never run it with access to production systems, credentials, or sensitive data.

## The phases

| Phase | What happens |
|-------|-------------|
| 0 — Receive | Read the full task and all linked context before forming any opinion |
| 1 — Understand | Restate the task; confirm with human (interactive) or document and proceed (autonomous) |
| 2 — Decompose | Break into independently plannable blocks |
| 3 — Plan | Write an explicit step-by-step plan for each block |
| 4 — Review | Read all plans together, check coherence against the original goal |
| 5 — Align | Present summary + full plans for complex blocks; wait for approval (interactive) or log and proceed (autonomous) |
| 6 — Execute | Run block by block using available tools and skills |
| 7 — Verify | Tests, core scenario, plan vs. result |
| 8 — Goal check | Did we achieve what was asked? If not — gap analysis and back to Phase 3, with a hard limit of 3 iterations |

## Escape hatch

The loop never runs forever. The agent stops and escalates when:

- The same gap appears twice
- A block fails more than twice
- The root cause is outside its control (third-party bug, missing access, changed requirements)
- More than 3 full iterations have completed without reaching the goal

## Cost & token usage

The Engineering Loop uses explicit multi-phase planning before any execution. This is intentional — catching a wrong approach in Phase 3 is cheaper than unwinding it after Phase 6.

- **Interactive mode** is cost-efficient: you review plans before the model executes, so planning errors are caught early and don't compound.
- **Autonomous mode** can consume significant context on large or ambiguous tasks — iterative gap analysis (Phase 8 → Phase 3 loops) multiplies token usage fast. Use it on tightly scoped, well-defined tasks. Set a budget limit in your Claude Code settings before running autonomously on large codebases.
- **Plan verbosity** is capped at 5–7 steps per block by default. For tasks with many blocks, consider decomposing into smaller sub-tasks rather than running one giant loop.

## Customization

The loop is intentionally generic. You can extend it in your own `CLAUDE.md`:

- Add project-specific tools per phase ("in Phase 6, always run `make lint` before committing")
- Override parallelism rules for your stack
- Add domain-specific escape hatch conditions
- Replace skill references with concrete tool names if your setup is fixed

## License

MIT
