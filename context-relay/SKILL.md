---
name: context-relay
description: >
  Monitors token usage throughout a conversation and proactively detects when the
  context window is at risk of being exhausted. When triggered, instead of failing
  silently or refusing a task, it generates a rich "handover prompt" the user can
  paste directly into any other agentic CLI (Claude Code, Gemini CLI, Cursor, GPT,
  etc.) to resume the task seamlessly. Use this skill proactively at the START of
  any response—especially before long agentic tasks, multi-file edits, large code
  generation, or research tasks. Trigger phrases: "context relay", "token check",
  "handover", or whenever the agent estimates it is getting close to its context limit.
---

# Context Relay Skill

## Purpose
Detect when the conversation is approaching the model's context window limit **before**
a task begins—not after it fails. Produce a compact, ready-to-paste **handover prompt**
the user can give to a fresh AI session to continue with zero loss of progress.

---

## Step 1 — Estimate Token Budget

Run this mental calculation **before every response** (costs < 10 tokens of overhead):

```
USED   ≈ (total characters in conversation so far) ÷ 3.5
LIMIT  = model limit (200 000 for Claude Sonnet/Haiku; 1 000 000 for extended)
BUDGET = LIMIT − USED
SAFE_MARGIN = 15 000   # tokens to keep free for output + tool calls
AVAILABLE = BUDGET − SAFE_MARGIN
```

**Heuristics for the incoming request cost:**
| Request type | Estimated cost |
|---|---|
| Quick answer / single-file edit | ~2 000 tokens |
| Multi-file refactor / analysis | ~8 000–20 000 tokens |
| Full codebase scan + report | ~30 000–80 000 tokens |
| Long agentic loop (many tool calls) | ~20 000–60 000 tokens |

If `AVAILABLE < incoming_request_estimated_cost × 1.3`, proceed to Step 2.

---

## Step 2 — Decide: Warn or Proceed

| Condition | Action |
|---|---|
| `AVAILABLE > request_cost × 1.3` | Proceed normally. No mention of tokens needed. |
| `AVAILABLE < request_cost × 1.3` AND `AVAILABLE > 8 000` | Add a one-line warning at the top: ⚠️ *Context is ~X% full. Consider a new session if the task grows.* Then proceed. |
| `AVAILABLE < 8 000` OR `AVAILABLE < request_cost` | **Trigger Handover Protocol** (Step 3). Do NOT attempt the full task. |

---

## Step 3 — Handover Protocol

When triggered, output **only** the following block. Be concise. Do NOT pad with explanations.

````
═══════════════════════════════════════════════════════════════
🔁  CONTEXT RELAY — HANDOVER PROMPT
    Paste this into a fresh AI session to continue seamlessly.
═══════════════════════════════════════════════════════════════

## Project / Task Overview
[1–3 sentences on what we are building or doing]

## Current Objective
[The exact task the user just requested]

## Progress So Far
- [Bullet: what has been completed]
- [Bullet: what was decided and why — key design choices]
- [Bullet: what is partially done or in flight]

## Relevant Files & Paths
[List only files touched or referenced, with 1-line purpose each]

## Current State / Last Known Good State
[Short description of where the code/system is right now]

## Resume Instructions
Continue from where we left off. The next step is:
[Precise, unambiguous next action]

## Environment & Constraints
- Language / framework: [e.g., Python 3.11 / Next.js 14]
- Key dependencies: [e.g., prisma, langchain]
- Style rules: [e.g., use TypeScript strict mode, 2-space indent]
- Do NOT: [any explicit anti-patterns or refusals the user stated]

═══════════════════════════════════════════════════════════════
⚡ You are now the primary agent. Pick up exactly where the
   previous session ended. Ask no clarifying questions about
   context already described above.
═══════════════════════════════════════════════════════════════
````

---

## Rules

1. **Be silent when not needed.** Never mention token counts unless Step 2 says to.
2. **Be accurate.** Do not over-trigger. One false alarm wastes more tokens than silence.
3. **Fill every section.** A handover prompt with "[unknown]" entries is useless. Pull context from the conversation.
4. **Keep the handover prompt self-contained.** The user should be able to paste it cold.
5. **Do not self-compact silently.** Never drop history without telling the user.
6. **Do not refuse tasks.** The handover prompt is an *offer*, not a block. If the task is small enough, do it.
