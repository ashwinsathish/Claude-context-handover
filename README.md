# context-relay

> A Claude/agentic-CLI skill that monitors token usage and generates a barrier-free **handover prompt** when the context window is about to run out — so you can continue any task in a fresh session with zero progress lost.

## What it does

Instead of silently failing or refusing a task when tokens run low, `context-relay` produces a ready-to-paste handover prompt that includes:

- Project & task overview
- Exact current objective
- What's been completed / decided / in-flight
- Relevant file paths
- Precise "resume here" instructions
- Environment & constraints

Paste it into any AI (Claude Code, Gemini CLI, Cursor, GPT, etc.) and the new session picks up immediately — no re-explaining.

## Behaviour

| Token budget | What happens |
|---|---|
| Plenty left | Silent. Zero overhead. |
| Getting tight | One-line warning, then proceeds with the task |
| About to run out | Outputs the 🔁 Handover Prompt instead of attempting the task |

## Installation

### Claude Code — project level
```bash
cp -r context-relay /path/to/your/project/.claude/skills/
```

### Claude Code — global (all projects)
```bash
cp -r context-relay ~/.claude/skills/
```

### Other agentic CLIs
Drop the `context-relay/` folder wherever your tool reads skills from. Any tool that supports the [Agent Skills](https://agentskills.io) open standard will pick it up automatically.

## Usage

The skill runs **automatically** before long tasks. You can also trigger it explicitly:

- *"token check"*
- *"context relay"*
- *"handover"*
- *"are we close to the limit?"*

## Skill format

Built on the [SKILL.md open standard](https://agentskills.io) — compatible with Claude Code, Cursor, and any agentic CLI that supports the format.

---

MIT License
