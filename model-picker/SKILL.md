---
name: model-picker
description: Recommend the right Claude Code model + thinking level (Haiku 4.5, Sonnet 4.6, or Opus 4.7 at low/medium/high/xhigh/max) for the current coding task. Use whenever the user asks "what model should I use", "is this the right model", mentions `/model`, complains about cost / latency / quality of responses, OR when you (Claude) notice the active model is mismatched — e.g., burning visible thinking on trivial edits, or hitting repeated tool-loop failures and losing the plot on a complex multi-file change. Self-trigger proactively on those signals; don't wait to be asked. Output a single recommendation with a brief justification — the user configures via `/model`.
---

# Model Picker

Pick one model + thinking level for the task at hand. Be terse: a recommendation, a one-line justification, and an optional cheaper fallback. **Do not run `/model`** — the user configures it themselves.

## Models in scope (Claude Code)

| Model | Thinking levels | Input / Output ($/MTok) | Context | Max output |
|---|---|---|---|---|
| Haiku 4.5 | *(single mode — no thinking dial)* | $1 / $5 | **200k** | 64k |
| Sonnet 4.6 | low, medium, high, max | $3 / $15 | 1M | 64k |
| Opus 4.7 | low, medium, high, **xhigh**, max | $5 / $25 | 1M | 128k |

`xhigh` exists **only** on Opus 4.7. Haiku has no thinking-level setting in Claude Code.

## Power ranking (weakest → strongest)

1. Haiku 4.5
2. Opus 4.7 — low *(thinking-starved; underperforms a thinking Sonnet on agentic loops)*
3. Sonnet 4.6 — low
4. Sonnet 4.6 — medium
5. Opus 4.7 — medium
6. Sonnet 4.6 — high
7. Sonnet 4.6 — max
8. Opus 4.7 — high
9. Opus 4.7 — xhigh
10. Opus 4.7 — max

Power and cost don't move in lockstep — a high-effort Sonnet beats a low-effort Opus, because agentic work needs thinking headroom that Opus-low refuses to spend.

## Cost ranking (cheapest → most expensive)

Relative multipliers vs. a non-thinking Haiku session = 1x. Thinking tokens bill at the **output** rate (5x input), so higher effort levels scale superlinearly on dense tool loops.

| Tier | ~Multiplier |
|---|---|
| Haiku | 1x |
| Sonnet — low | 3x |
| Sonnet — medium | 4x |
| Opus — low | 5x |
| Sonnet — high | 6x |
| Opus — medium | ~7x *(estimate, less commonly used)* |
| Sonnet — max | 10x |
| Opus — high | 12x |
| Opus — xhigh | 20x |
| Opus — max | 40x |

Approximate — actual burn depends on how much the model chooses to think.

## Context windows

- **Haiku 4.5: 200k tokens.** Disqualifies Haiku for big-codebase sweeps or huge log dumps.
- **Sonnet 4.6: 1M tokens.**
- **Opus 4.7: 1M tokens.**

If task description + already-loaded context clearly exceeds 200k, Haiku is off the table even for mechanical work.

## Selection guidelines

**Haiku 4.5** — fast mechanical work: regex/grep sweeps, throwaway scripts, syntax checks, pre-filtering log files (within 200k). Don't use it when reasoning has to span files or when the codebase context is large. **Important:** Haiku requires manual user approval for every tool call — avoid it for tasks that involve many sequential tool calls (file edits, builds, multi-step installs), as constant approval prompts will slow work to a crawl. Reserve Haiku for truly one-shot lookups or when the user is actively supervising each step.

**Sonnet 4.6 (medium / high / max)** — *"Sonnet for the work."* The daily workhorse for full-time coding:
- `medium` — simple feature work, light bug fixes, routine edits in a known codebase.
- `high` — general implementation, interactive CLI sessions, normal debugging. Sensible default.
- `max` — complex multi-file refactors, dense runtime debugging, tricky edge-case test writing.

**Opus 4.7 (high / xhigh / max)** — *"Opus for the plan."* Architectural work:
- `high` — planning, foreign-language / custom-grammar parsing without docs, solid design work.
- `xhigh` — deep cross-file analysis, sweeping structural changes, dependency mapping.
- `max` — peak architectural reasoning across very large (1M-token) codebases.

A common efficient pattern: use Opus at `high`+ to draft the plan, then drop to Sonnet to execute the edits.

**Avoid Opus on `low`** — it minimizes thinking and consistently loses to a thinking Sonnet on agentic loops while costing more per token.

## When to self-trigger this skill

Watch for these signals during normal work and invoke the skill proactively when you see them — don't wait for the user to ask.

**Downgrade signals:**
- Visible thinking time on one-line edits, trivial renames, formatting.
- Task is pure file-shuffle / regex / formatting and current model is Sonnet-max or any Opus.
- User comments on cost, latency, or "this is overkill" on routine work.

**Upgrade signals:**
- Repeated tool-call failures or retries on the same operation.
- Losing the thread across files in a multi-file change.
- Plan/implementation drift — code doesn't match the stated intent.
- Task involves novel architecture, foreign grammars, or 500k+ tokens of loaded context.

**Switch family (Sonnet ↔ Opus):**
- Sonnet thrashing on what's really a *design* problem → Opus high/xhigh to plan.
- Opus burning xhigh/max thinking on mechanical execution → Sonnet high to do the edits.

## Classification cheat sheet

| Task signal | Recommendation |
|---|---|
| grep across repo, throwaway script, one-shot regex (single tool call) | Haiku |
| Routine feature, single-file bug fix, known patterns | Sonnet — medium or high |
| Multi-file refactor, tricky debugging, edge-case tests | Sonnet — max |
| "Design …", "how should I architect …", `/create-plan` | Opus 4.7 — high or xhigh |
| Sweeping changes across 1M+ token codebases, foreign grammar | Opus 4.7 — xhigh or max |
| Plan handoff to execution | Opus to plan → Sonnet to execute |

## Output format

When invoked, produce **exactly** this shape — nothing more:

```
Recommendation: <Model> — <level>
Why: <1–2 sentences tying the task to the choice>
Fallback (cheaper): <Model> — <level>, if <condition>   [optional, omit if no sensible cheaper option]
Ask: <one clarifying question>                           [only if genuinely blocked on classification]
```

**When recommending Opus 4.7 at high/xhigh/max**, the `Fallback` line should usually surface the *plan → execute* handoff rather than a cheaper Opus level. Example:

> `Fallback (cheaper): Sonnet 4.6 — high once the plan is set, to execute the edits at ~3x lower cost.`

That handoff is the single biggest cost-saver on architectural work and is the main reason Opus + Sonnet beats Opus alone — so don't bury it in the body where the user won't see it.

Don't paste the ranking tables. Don't lecture. The user reads the recommendation and runs `/model` themselves.
