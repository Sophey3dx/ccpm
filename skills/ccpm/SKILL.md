---
name: ccpm
description: >
  CCPM (Claude Code Plan Management) — Opus 4.7 plans and oversees execution,
  dispatching agents with the right model (Haiku 4.5 / Sonnet 4.6 / Opus 4.7)
  for each step. Execution happens in isolated agent contexts so the main Opus
  session only sees summaries — saving tokens while keeping quality high.
  ALSO triggers as a universal Opus intercept: whenever running on Opus and
  about to do substantive work, ask the user whether to run inline or dispatch
  as a cheaper agent. If a CCGM goal mode is active (Cost / Quality / Speed /
  Learning / Balanced), CCPM adapts model and effort choices to match.
  Trigger on: explicit planning language, 3+ ordered steps, todo/breakdown
  requests — AND any time Opus is about to execute substantive work.
---

# CCPM — Claude Code Plan Management

Opus plans. Opus oversees. Cheap agents do the work.

Every plan step gets a **model** and **effort** tag. After the plan is approved,
you dispatch each step as an isolated agent — Haiku or Sonnet for the bulk of
the work, Opus only for the steps that genuinely need deep reasoning. Your main
Opus context only ever sees agent summaries, not file contents, raw output, or
intermediate noise. That's where the token savings come from.

---

## 🛑 Universal Opus Intercept (always active)

**Whenever you are running on Opus and are about to execute substantive work,
pause and ask the user before starting.**

"Substantive work" means anything that would accumulate significant context:
reading multiple files, writing or editing code, running searches across a
codebase, generating long output, or any multi-tool task.

**Not** substantive: a one-sentence answer, a clarifying question, a
single-tool lookup, pure reasoning with no file I/O.

### The Intercept Question

Before starting the work, present this choice once:

> **Token check:** Should I run this here (Opus, full context) or dispatch as
> an agent to keep this session lean?
>
> - **Dispatch as Sonnet** — handles most implementation, debugging, analysis
> - **Dispatch as Haiku** — for simple/mechanical tasks (reads, renaming, docs)
> - **Run here in Opus** — needs full conversation context or deep reasoning

If the user picks dispatch: pick the appropriate model (Sonnet default unless
the task is clearly mechanical → Haiku, or clearly requires deep novel
reasoning → stay Opus), craft a self-contained agent prompt, dispatch via
the Agent tool.

If the user picks "run here" or says "just do it" / "go ahead": proceed
inline, no further intercept for this task.

### When NOT to intercept

- The user has already answered this question for the current task
- The work is clearly just a quick reply or single-file lookup
- The user has said "run here" before and hasn't asked to change it
- The session is already dispatching agents (you're in oversight mode)
- Pure reasoning/design with no file I/O (Opus inline is correct here)

---

## 🏛️ The Architect Rule (read this first)

**Plan creation always runs on Opus 4.7 + xHigh thinking.** No exceptions.

A plan is a force multiplier: one good planning pass produces a roadmap that
5-15 cheaper agents follow. A bad plan wastes every cheap agent downstream.
The architect needs the full picture, the tradeoffs, the failure modes, the
long view. The workers just execute.

Before writing the plan:

1. **Think deeply.** Use all the budget — this is the one place where
   over-thinking is correct.
2. **Read enough to plan well.** Glance at relevant files, existing patterns,
   constraints. Don't implement, but don't plan blind.
3. **Surface the real tradeoffs.** Where are the forks? Which decisions, if
   wrong, cost the most to reverse?
4. **Write the annotated plan.** Each step gets model + effort tags.
5. **Identify the dependency graph.** Which steps can run in parallel?

The plan is the deliverable. Execution comes after, with agents doing the work.

---

## 🎯 Goal Awareness (CCGM integration)

CCPM defaults assume **Balanced** goal mode. If CCGM is active and a different
goal is set, adapt:

| Goal | How CCPM shifts |
|------|-----------------|
| **💰 Cost** | Bias steps one notch down (Sonnet → Haiku where viable). Opus only where no cheaper model can do it. Split aggressively to isolate Opus need. |
| **🛡️ Quality** | Bias steps one notch up. Add explicit review steps. Sonnet steps touching business logic → Opus. |
| **⚡ Speed** | Prefer Haiku/Sonnet, Low/Medium effort. Maximize parallel dispatch. Skip polish steps — mark as "Speed-mode: deferred". |
| **🎓 Learning** | Add explanatory steps before implementation. Bias toward Opus on conceptually rich steps. Each agent's prompt asks for reasoning, not just output. |
| **⚖️ Balanced** | Sonnet/Medium baseline. Opus only where genuinely needed. |

If you don't know the active goal, ask once:

> "Quick check before I plan — what's the goal here? **Quality / Cost / Speed / Learning / Balanced** (default: Balanced)"

The Architect Rule **does not change with goal mode** — even Cost mode plans
on Opus, because a bad plan in cost mode wastes more than it saves.

---

## The Tag Format

Every plan step ends with:

```
> 🤖 Sonnet 4.6 | ⚡ Medium — reason in <8 words
```

- One blockquote line directly under the step
- Model first, effort second, brief reason last (under 8 words)
- Exact model names: **Haiku 4.5**, **Sonnet 4.6**, **Opus 4.7**
- Also mark dependency: `[parallel: steps X,Y]` or `[after: step N]`

---

## Model Selection

Pick the cheapest model that can do the job well. Default to Sonnet.

| Model | Use For |
|-------|---------|
| **Haiku 4.5** | Boilerplate, renaming, formatting, file reading, simple ops, scaffolding from templates, doc strings, log filtering |
| **Sonnet 4.6** | Default. Multi-file features, debugging, refactors, tests, code review, API integration — most real work |
| **Opus 4.7** | Novel architecture decisions, complex system design, deep tradeoff reasoning, hard concurrency/state, security-critical logic, anywhere wrong = hours lost |

Do NOT dispatch Opus agents just because the feature is big. Opus is for
**specific steps that need deep reasoning**, not "this step is important."

**Opus steps and inline handling:** If an Opus step is a pure reasoning/design
step with no file I/O, handle it inline rather than dispatching an agent —
you already have full context and agent overhead adds nothing. If it's a
self-contained deep work task (e.g., isolated security review of a module),
dispatch it as an Opus agent to keep its output clean.

---

## Effort Level Selection

Effort maps to how much thinking guidance the agent prompt includes.

| Effort | When | Agent Prompt Instruction |
|--------|------|--------------------------|
| **Low** | Obvious answer. Boilerplate. Mechanical edits. | No special wording — just give the task. |
| **Medium** | Default. Normal feature work. Requires some thought. | "Work through this carefully before starting." |
| **High** | Multiple valid approaches. Subtle bugs. Non-obvious ripple effects. | "Consider multiple approaches. Think through edge cases before deciding." |
| **xHigh** | Genuinely hard. Architecture. Concurrency. Security. | "Think deeply. Explore the problem space, surface tradeoffs, consider failure modes before acting." |

Effort and model are independent. A Sonnet agent can have xHigh effort.
Opus + Low is rare — usually if you need Opus you need thinking too.

---

## What a Good Plan Looks Like

1. **Short context paragraph** — what we're building, the key constraint, the
   assumed starting state, active goal mode.
2. **The real tradeoff up front** — if there's a fork, name it before the steps.
3. **Ordered, sized steps** — each step is one model+effort combo's worth of
   work. If a step needs two different models, split it.
4. **Dependency graph** — which steps can run in parallel, which must be sequential.
5. **Cost summary** — count of steps per model. Sanity check for the user.
6. **Optional: risk flags** — steps where the plan might need to change.

---

## Execution Phase: Agent Dispatch

After the plan is reviewed (or when the user says "go"), execute it:

### Step 1: Group by Dependency

Identify which steps can run in parallel (no shared files, no ordering deps)
and which must be sequential. Announce the groups:

```
Parallel batch 1: steps 1, 2 (independent reads)
Sequential: step 3 (needs results from 1+2)
Parallel batch 2: steps 4, 5 (independent implementations)
Sequential: step 6 (integration, needs 4+5)
```

### Step 2: Dispatch Agents

Use the `Agent` tool with the `model` parameter matching the step's tag:

```
Haiku 4.5  →  model: "haiku"
Sonnet 4.6 →  model: "sonnet"
Opus 4.7   →  model: "opus"
```

For parallel steps in the same batch, dispatch all of them in one message
(multiple Agent tool calls). They run concurrently.

**Agent prompt requirements — every agent gets:**
- Clear task description (what to do, what to produce)
- All context it needs (relevant file paths, constraints, patterns to follow)
- Effort instruction (from the table above)
- Output spec: "Return: [what exactly to summarize]"
- Constraints: what NOT to change, what NOT to include
- No inherited conversation context — construct everything it needs

**Agents do NOT get:** conversation history, reasoning about other steps,
or anything outside their specific scope.

### Step 3: Receive and Integrate

When agents return:
- Read each summary — understand what changed
- Check for conflicts (did any agents touch the same files?)
- If an agent failed or went off-track, re-dispatch with a corrected prompt
- Once all agents in a batch complete, proceed to the next sequential step
- Run tests or verification as appropriate between batches

### Step 4: Opus Synthesis

After all agents complete, provide a brief synthesis:
- What was done (agent-by-agent summary)
- Any integration steps you handled inline
- Verification results (tests, build, etc.)
- Any deferred items or follow-up needed

---

## Token Savings Explained

The main Opus session accumulates context fast. Each file read, each code
block, each tool result lands in context. With agent dispatch:

- **File I/O and boilerplate** happen in Haiku agent contexts (not yours)
- **Multi-file implementation** happens in Sonnet agent contexts (not yours)
- **Your Opus context** only grows by agent summary strings — typically
  100-300 tokens per step instead of thousands
- **Parallel agents** reduce wall-clock time while each context stays small

A 6-step plan where steps 1-2 are Haiku reads + steps 3-5 are Sonnet
implementations saves ~8-15k tokens in your main context versus handling
everything inline — and runs faster because 1+2 and 3+5 can be parallel.

---

## Use Case Examples

### Use Case 1: New Feature (Balanced goal)

```markdown
## Plan: Add WebSocket reconnection logic

**Context:** Goal mode: Balanced. Existing `client.ts` handles a single
connection. Need automatic reconnect with exponential backoff. Main tradeoff:
aggressive reconnect = faster recovery but server load risk.

**Dependency graph:**
- Steps 1 can run alone (read)
- Step 2 needs step 1's findings
- Steps 3, 4 can run in parallel after step 2
- Step 5 needs 3+4
- Step 6 is independent (docs)

1. **Read current connection logic in `client.ts`**
   > 🤖 Haiku 4.5 | ⚡ Low — file reading, no decisions [parallel: step 6]

2. **Design exponential backoff strategy with jitter**
   > 🤖 Opus 4.7 | ⚡ xHigh — concurrency, easy to get subtly wrong [after: step 1]

3. **Implement the reconnect loop**
   > 🤖 Sonnet 4.6 | ⚡ Medium — standard impl from design [after: step 2, parallel: step 4]

4. **Add reconnect events to the typed event emitter**
   > 🤖 Sonnet 4.6 | ⚡ Low — mechanical addition to existing pattern [parallel: step 3]

5. **Write unit tests covering disconnect → reconnect → resume**
   > 🤖 Sonnet 4.6 | ⚡ Medium — test cases need thought [after: steps 3+4]

6. **Update README with the new behavior**
   > 🤖 Haiku 4.5 | ⚡ Low — doc writing [parallel: step 1]

**Cost summary:** 2× Haiku, 3× Sonnet, 1× Opus 4.7
**Dispatch plan:** Batch 1 (parallel): steps 1+6 → Step 2 inline (Opus reasoning) → Batch 2 (parallel): steps 3+4 → Step 5
```

### Use Case 2: Bug Hunt (Cost goal)

```markdown
## Plan: Investigate intermittent 500 error on /api/orders

**Context:** Goal mode: Cost. Production error rate ~0.3%. Cheapest viable
path — escalate models only if cheaper attempts stall.

1. **Pull last 24h of error logs and group by stack trace**
   > 🤖 Haiku 4.5 | ⚡ Low — log filtering

2. **Identify the most likely root cause from grouped traces**
   > 🤖 Sonnet 4.6 | ⚡ Medium — reasoning across noisy data [after: step 1]

3. **Reproduce the bug locally**
   > 🤖 Sonnet 4.6 | ⚡ Low — standard repro workflow [after: step 2]

4. **Patch the issue + add a regression test**
   > 🤖 Sonnet 4.6 | ⚡ Medium — impl once cause is known [after: step 3]

5. **Write the postmortem**
   > 🤖 Haiku 4.5 | ⚡ Low — writing from known facts [after: step 4]

**Cost summary:** 2× Haiku, 3× Sonnet (no Opus — Cost mode, escalate only if stalled)
**Dispatch plan:** Steps 1→2→3→4→5 sequential (each depends on previous)
```

### Use Case 3: Refactor (Quality goal)

```markdown
## Plan: Refactor auth from session cookies to JWT

**Context:** Goal mode: Quality. Security-critical refactor. Bias toward
more careful review and higher effort on token handling.

1. **Map all current touch points of the session system**
   > 🤖 Sonnet 4.6 | ⚡ High — Quality: don't miss any callers

2. **Design the JWT structure, claims, refresh strategy**
   > 🤖 Opus 4.7 | ⚡ xHigh — security-critical, hard to fix later [after: step 1, inline]

3. **Implement the new JWT module**
   > 🤖 Opus 4.7 | ⚡ High — Quality: token logic, no Sonnet shortcut [after: step 2]

4. **Migrate each route handler one-by-one**
   > 🤖 Sonnet 4.6 | ⚡ Medium — mechanical, adversarial review [after: step 3]

5. **Add tests for token expiry, refresh, revocation**
   > 🤖 Sonnet 4.6 | ⚡ High — security tests need adversarial thinking [parallel: step 4]

6. **Independent security review pass**
   > 🤖 Opus 4.7 | ⚡ xHigh — Quality mode: explicit review step [after: steps 4+5]

7. **Remove the old session code**
   > 🤖 Haiku 4.5 | ⚡ Low — deletion, no decisions [after: step 6]

**Cost summary:** 1× Haiku, 2× Sonnet, 3× Opus 4.7
**Dispatch plan:** Step 1 agent → Step 2 inline → Step 3 agent → Batch (4+5 parallel) → Step 6 agent → Step 7 agent
```

---

## Agent Prompt Template

Use this structure for every dispatched agent:

```
[TASK]
<one clear sentence: what to produce>

[CONTEXT]
Working directory: <path>
Relevant files: <list key files — don't dump everything>
Constraints: <what NOT to touch, what pattern to follow>
<paste any key output from previous agents this one depends on>

[EFFORT]
<paste the effort instruction from the effort table>

[OUTPUT]
Return a summary covering:
- What you found / decided
- What files you changed and why
- Any assumptions you made
- Any blockers or uncertainties
Do NOT return full file contents — just the summary.
```

---

## Important Heuristics

- **The plan itself is always Opus 4.7 + xHigh — non-negotiable.** Even Cost mode. A bad plan wastes every agent downstream.
- **Default to Sonnet + Medium** when in doubt (Balanced mode).
- **Splitting is cheaper than upgrading.** If a step needs Opus xHigh, ask: can I split the hard reasoning part from the mechanical implementation? Only the reasoning step needs Opus.
- **Reading/exploration is almost always Haiku Low.** Grepping, listing, reading files — Haiku handles it.
- **The first step of a feature is rarely Opus.** Discovery isn't the hard part.
- **If every step is Opus xHigh, you're panicking, not planning.** Re-evaluate.
- **Parallel dispatch saves time.** If 3 steps are independent, dispatch all 3 at once — they run concurrently.
- **Opus reasoning steps can be inline.** Pure design/reasoning with no file I/O doesn't need an agent — you already have context.
- **Security work overrides Cost/Speed goals.** Always. On every step that touches auth, tokens, or permissions.

---

## When NOT to Plan

Skip architect mode and agent dispatch for:
- Single-step asks ("fix this typo")
- Conversational exploration ("what's the best way to…")
- Reviews where the user wants feedback, not a re-plan
- Pure Q&A with no implementation

Just respond normally — no tags, no dispatch.

---

## Integration with TodoWrite / ExitPlanMode

When using TodoWrite or ExitPlanMode, put the tag in parentheses at the end
of each item:

```
Implement reconnect loop (🤖 Sonnet 4.6 | ⚡ Medium)
```

Keep the full annotated plan with dependency graph in your message body.
When execution starts, follow the Execution Phase above.

---

## Trigger Phrases

Engage architect mode when **at least one** is true:

| Condition | Example |
|-----------|---------|
| Explicit planning language | "plan this", "make a plan", "roadmap for", "break this down", "outline the steps" |
| User enumerates 3+ ordered steps in one message | "migrate the DB, update the API, then fix the frontend" |
| User requests a todo list or step-by-step breakdown | "give me a todo list for adding payments" |

**Do not trigger on:**
- "implement X", "build X", "add X" alone
- "fix this", "change this" for a single targeted change
- "what's the best way to…" (advice, not a breakdown request)

**Sanity check:** Would a senior engineer naturally reach for a checklist here?
If yes and the plan adds value, proceed. If the whole thing fits in one focused
session without sub-decisions, skip the plan.
