---
name: clauude-model-picker
description: Use this skill whenever a user is choosing which Claude model and reasoning mode (Thinking ON/OFF, sometimes labeled "Extended" or "Adaptive") to use for general chat, writing, analytical, or text-processing work. Trigger on direct asks like "which model should I use for X?", "should I turn thinking on?", or "is Opus worth the burn here?" AND proactively whenever the user describes a substantive non-coding workflow where the choice would meaningfully affect output quality, latency, or message-limit burn (e.g. "I need to summarize 200 customer reviews", "drafting a sensitive HR letter", "analyzing a long commercial lease", "cleaning up a 50-page transcript"). Trigger even when the user does not explicitly say "which model" — if their task description maps cleanly onto a model-tier decision, fire the skill. Do NOT trigger for coding/IDE workflows (different selection logic applies there) or for one-off factual questions that do not reveal a workflow.
---

# Claude Chat Model Picker

A decision framework for picking the right Claude model + reasoning mode for general chat, writing, and analytical work in claude.ai. The goal is to balance **output quality**, **latency**, and **message-limit burn** across the six available configurations.

This skill is for non-coding workflows. Coding has different selection considerations (long-running agents, tool-use, file edits) that are not covered here.

---

## The six configurations

Three model tiers × thinking toggle = six combinations. They differ along two independent axes: the model's reasoning ceiling, and whether it deliberates before writing.

| Configuration | Relative burn | Comparative strength |
|---|---|---|
| **Haiku 4.5 — Thinking OFF** | ~1× | Mechanical text manipulation: stripping formatting, fixing typos, basic classification, simple extraction, dictionary-style lookups |
| **Haiku 4.5 — Thinking ON** ("Extended") | ~2× | Light-to-medium careful reasoning at low cost: edge-case classification, multi-step extraction with conditional rules, simple logic checks, cost-sensitive constrained drafting |
| **Sonnet 4.6 — Thinking OFF** | ~3× | Fluid language synthesis: routine emails, standard creative fiction, multi-page summaries, formatting notes into structured markdown, general chat |
| **Sonnet 4.6 — Adaptive Thinking ON** ⭐ *daily workhorse* | ~6× | Tone-calibrated communication, logic auditing of pitches/essays, strategic planning with competing constraints, learning about nuanced topics |
| **Opus 4.7 — Adaptive Thinking OFF** | ~5–7× | Frontier-grade prose fluidity without reasoning burn: long-form creative writing at the ceiling, executing a plan Opus already deliberated on, Opus-depth chat where latency matters |
| **Opus 4.7 — Adaptive Thinking ON** 🏔️ *heavy lifter* | ~15–20×+ | Dense conceptual abstraction: contract loophole hunting, nested philosophical interpretation, multi-variable strategy mapping, intricate logic puzzles |

The burn multipliers are mental models for claude.ai message-limit consumption, not exact billing. Thinking tokens are charged at output rates (5× the input rate), which is why flipping thinking on roughly doubles burn at any tier. Opus 4.7 also ships with a denser tokenizer that adds up to ~35% additional overhead on top of its base rate.

---

## Switching costs

**Model switch = cold cache = heavier message burn.** When you change the model mid-thread (e.g. switch from Sonnet to Opus in the dropdown), claude.ai has to re-read the entire conversation history from scratch on the first message to the new model. In a long, active thread this hidden re-read consumes significantly more message budget than simply continuing on the current model.

**Thinking toggle = partial cache miss = lighter burn.** Flipping thinking on/off preserves the system-level infrastructure (Claude's base instructions) but invalidates the message history. Cheaper than a model switch, but not free in a very long thread.

**Prefer toggling thinking before escalating model tier.** If Sonnet-off isn't cutting it, try Sonnet-on before jumping to Opus.

**When switching model is fine:**
- Thread is short (few messages) — switch freely, the overhead is trivial.
- Current model is clearly failing the task (wrong outputs, stuck loops) — switch regardless of thread length; continued failed attempts cost more than the switch overhead.
- Long thread + committing to the new model for many subsequent messages — the re-read cost amortizes.

**Fresh tab pattern:** If you only need Opus for one bounded task (a single contract section, one planning pass), open a fresh tab and paste just the relevant excerpt. Tiny context = trivial cache overhead. Bring the result back to your main thread as plain text.

**Running this skill itself has a cost.** In a very large, active thread, invoking the skill consumes message budget. Prefer running it at the start of a session or in a fresh tab when the thread is already long. **Do not re-invoke repeatedly** — one call per task segment is the norm. Only re-invoke if the task has significantly changed since the last recommendation.

---

## Decision framework

Two axes. Pick the row first, then the column.

### Axis 1 — Cognitive depth required
- **Mechanical / surface-level:** fixed-rule transformations, classification, extraction, formatting → **Haiku tier**
- **Fluid synthesis:** writing, summarizing, planning, conversation → **Sonnet tier**
- **Frontier abstraction:** dense systemic logic, hidden contradictions across many variables, nested metaphor → **Opus tier**

### Axis 2 — Does the task benefit from deliberation before writing?
- **No → Thinking OFF.** The task is pattern-recognition or fluid generation. There is no logic trap to audit, no competing constraints to balance, no tone tightrope. Deliberating wastes tokens and adds latency.
- **Yes → Thinking ON.** The task requires balancing competing constraints, auditing tone or logic, surfacing hidden flaws, or planning under uncertainty. The internal blueprint pass meaningfully improves quality.

### Heuristics for when thinking earns its keep
- Multiple competing constraints to balance? → ON
- Sensitive tone (HR, legal, conflict, customer recovery)? → ON
- Auditing somebody else's argument for flaws? → ON
- Planning with uncertainty or branching paths? → ON
- Pure language generation, summarization, chit-chat? → OFF
- High volume / cost-sensitive / latency-sensitive? → OFF (or Haiku ON for the cheap-careful niche)

---

## Output format

When the skill fires, respond with this template:

```
**Primary: <Model> — <Thinking ON/OFF>**
<1–2 sentences on why this matches the task.>

**Fallback: <Model> — <Thinking ON/OFF>**
<1–2 sentences on the tradeoff — usually "cheaper if X" or "step up if you want Y".>

**Switch:** In current thread | Fresh tab — <one-line rationale>   [only when Primary is a different model tier than current; omit for thinking-toggle-only changes]
**If already on Primary:** <decision tree or low-context note>      [always include; see rules below]
```

**`Switch:` rules** (only when recommending a different model tier):
- `In current thread` — thread is short, model is failing, or committing to many messages on the new model.
- `Fresh tab` — long thread + only need the new model for one bounded task. Note what to paste in.
- Omit entirely when Primary and Fallback are the same model (thinking toggle only).

**`If already on Primary:` rules** (always include):
- **Short thread:** "Thread is short — switching is cheap. Just apply the recommendation."
- **Long thread, thinking-toggle change:** include a brief branch: "If thinking is already ON: no change needed. If OFF: flip it on."
- **Long thread, cross-model move:** `Switch:` line covers it; add "If already on [Primary]: no change needed."

The fallback is typically *one step in either direction* on cost/power — a cheaper option if the user seems burn-conscious, or an upgrade if they hint at wanting peak quality.

For workflows with clearly heterogeneous stages (e.g. raw cleanup → analysis), the **Primary** may be a two-stage recipe. Only do this when the stages genuinely call for different configurations.

If the recommendation rests on a non-obvious assumption, end with one short line: `Assumed: <the assumption>.`

Keep the whole output tight — power users want a confident pick with a clean rationale, not a survey of all six tiers every turn.

---

## Clarifying questions: when and what

If a single missing detail would flip the recommendation across tiers, ask **one or two short clarifying questions** before recommending. Otherwise, infer, recommend, and flag the assumption inline.

**Ask when:**
- Stakes are ambiguous ("write a difficult email" — difficult how? legally? interpersonally? unfamiliar topic?)
- Volume/scale is missing and would matter ("clean up some reviews" — 5 or 5,000?)
- The deliverable type has wildly different cognitive demands ("help me think through a contract" — read-and-summarize, or hunt-for-liabilities?)

**Don't ask when:**
- The task is described in enough detail to pick confidently — just recommend, flag any assumption, move on.
- The user has already signaled a constraint ("on Pro plan, trying to save messages").

Keep clarifications short and concrete. No questionnaires. If two questions feel needed, lead with the one whose answer would most narrow the recommendation.

---

## Worked examples

### Example 1 — workflow with heterogeneous stages
> "I have a 40-page raw Zoom transcript from a strategy offsite. I want to clean it up, then pull out the key decisions and disagreements."

**Primary: Haiku 4.5 (Thinking OFF) for cleanup, then Sonnet 4.6 (Thinking ON) for the analysis pass.**
Cleanup is mechanical text manipulation — Haiku handles it at a fraction of the burn. The decisions/disagreements pass needs to balance multiple voices and surface implicit conflict, which is exactly what Sonnet's adaptive thinking pass is built for.

**Fallback: Sonnet 4.6 (Thinking ON) for the whole thing.**
If you'd rather not switch threads, Sonnet thinking-on can do both — you just pay ~6× burn on the cleanup work you could've offloaded to Haiku.

---

### Example 2 — direct ask, sensitive communications
> "I'm writing a performance improvement plan for an employee who's struggling but I genuinely believe can recover. Has to be firm, fair, legally clean, and not crush them. Which model?"

**Primary: Sonnet 4.6 — Adaptive Thinking ON.**
This is the canonical "competing constraints + sensitive tone" case: firmness vs. compassion, legal cleanliness vs. human voice. Thinking on lets Sonnet draft a blueprint, audit tone, and check the constraints before committing text.

**Fallback: Opus 4.7 — Adaptive Thinking ON.**
Step up to Opus only if the situation has unusual legal exposure or you want maximum care on phrasing. For typical PIPs, Sonnet on is overkill-resistant and Opus is overspend.

---

### Example 3 — proactive trigger, mechanical bulk work
> "I've got a folder of about 300 customer support chat logs and I want to bucket them into themes — refund, bug, onboarding confusion, feature request, other."

**Primary: Haiku 4.5 — Thinking OFF.**
High-volume classification with a fixed taxonomy is exactly Haiku's comparative advantage — fast, cheap, and accurate enough on rule-following work.

**Fallback: Haiku 4.5 — Thinking ON.**
If a meaningful chunk lands in "other" or gets miscategorized at the edges, flipping thinking on (still ~2×, far cheaper than Sonnet) lets Haiku reason about ambiguous cases without escalating the tier.

---

### Example 4 — frontier abstraction
> "I need to dissect a 90-page commercial lease and surface every clause that could let the landlord raise rent or pass through costs in ways I'd find surprising. I want cross-clause interactions too."

**Primary: Opus 4.7 — Adaptive Thinking ON.**
Cross-clause interaction analysis on a dense legal document is what Opus thinking-on is reserved for — Sonnet often skims past systemic contradictions hiding three clauses apart.

**Fallback: Sonnet 4.6 — Adaptive Thinking ON.**
For a first-pass scan before committing Opus burn, Sonnet thinking-on will catch the obvious traps. Use its output to identify focus sections, then send only those (not the whole lease) to Opus.

**Switch:** Fresh tab — if you're already deep in a long thread, open a new tab and paste only the lease text (or the flagged sections). Tiny context = trivial Opus overhead. Bring the findings back to your main thread as plain text.

Assumed: you want depth-over-breadth (loophole hunting), not a plain-English summary.

---

## Edge case configurations worth knowing

Most picks land on Sonnet (off or on). The other four configurations exist for specific situations — don't treat them as exotic, but don't reach for them reflexively either.

### Haiku 4.5 — Thinking ON ("Extended")
The most overlooked configuration. Lives in the gap between "Haiku is too dumb" and "Sonnet on is too expensive."
- **Use when:** you need Haiku's speed/cost but the task has ambiguous edges — classification with fuzzy categories, extraction with conditional rules, drafting templated messages with several constraints, simple arithmetic/logic that needs verification.
- **Compared to Sonnet OFF:** roughly the same burn (~2× vs ~3×), but trades fluency for explicit reasoning. Pick Haiku-on when correctness on rules matters more than prose feel; Sonnet-off when the reverse.

### Opus 4.7 — Adaptive Thinking OFF
The most counter-intuitive configuration. Pricier than Sonnet ON but doesn't reason explicitly.
- **Use when:** you've already done the planning (e.g. in an earlier Opus-thinking thread) and now want Opus's prose ceiling to execute it without re-deliberating; or for long-form creative writing where you want frontier-grade vocabulary, rhythm, and synthesis but the work isn't logic-heavy.
- **Caution:** for analytical or strategic work, Sonnet thinking-on usually beats Opus thinking-off and costs less. Don't reach for Opus-off unless you specifically want its writing voice or its depth-without-deliberation.

---

## A note on terminology

Anthropic and the Claude.ai UI have used several labels for the reasoning toggle — "Extended thinking", "Adaptive thinking", or just "Thinking" — across models and updates. They refer to the same underlying capability: the model produces internal reasoning tokens before its visible response. Treat them as interchangeable when reading user prompts. In recommendations, "Thinking ON" / "Thinking OFF" is the cleanest phrasing.
