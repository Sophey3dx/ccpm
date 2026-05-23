---
name: ccpm
description: >
  CCPM (Claude Code Plan Management) — automatically annotates every planning step with the
  optimal Claude model (Haiku 4.5 / Sonnet 4.6 / Opus 4.7) and thinking effort level
  (Low / Medium / High / xHigh). The planning step itself runs on Opus 4.7 + xHigh thinking
  so the breakdown, tradeoffs, and per-step routing are as good as possible before any
  execution work begins. If the user has an active goal mode (see the CCGM skill — Cost /
  Quality / Speed / Learning / Balanced), CCPM adapts its model and effort choices to match
  that goal. Use this skill ANY TIME you create a plan, todo list, implementation roadmap,
  or break a feature down into steps — whether the user explicitly asks for a plan or you're
  using ExitPlanMode/TodoWrite. Trigger on mentions of "plan", "roadmap", "steps", "break
  this down", "implement X", "build X feature", "refactor", "todo list", or any multi-step
  engineering task — even when the user doesn't explicitly say "plan".
---

# CCPM — Claude Code Plan Management

When you create a plan, every step gets two tags: **model** and **effort**. The plan itself is created at full power, so the routing decisions are good.

The user can then run each step on the right tool for the job — saving money on trivial work and keeping the heavy artillery for the parts that actually need it.

---

## 🏛️ The Architect Rule (read this first)

**Plan creation always runs on Opus 4.7 + xHigh thinking.** No exceptions, no shortcuts.

A plan is a force multiplier: one good planning pass produces a roadmap that 5-15 cheaper execution steps follow. A bad plan wastes every cheap step downstream — wrong order, wrong granularity, missed edge cases, work that has to be redone. The planning model is the **architect**; the execution steps are the **workers**. The architect needs the full picture, the tradeoffs, the failure modes, and the long view.

When you detect that you're about to plan (see Trigger Phrases at the bottom), do this **before** writing the plan:

1. **Switch into deep-think mode.** Use Opus 4.7 with xHigh extended thinking — actually use the budget; this is the one place where over-thinking is correct.
2. **Read enough to plan well.** Glance at the relevant files, the existing patterns, the constraints. Don't dive into implementation, but don't plan blind.
3. **Surface the real tradeoffs.** Where are the genuine forks in the road? Which decisions, if wrong, cost the most to reverse later?
4. **Then write the annotated plan.** Each step gets a model+effort tag, picked from the criteria below.

The plan is the deliverable. Execution comes later, with cheaper models doing most of the work.

---

## 🎯 Goal Awareness (CCGM integration)

CCPM defaults assume **Balanced** goal mode. If the CCGM skill is active and a different goal is set, adapt:

| Goal | How CCPM shifts |
|------|-----------------|
| **💰 Cost** | Bias steps one notch down (Sonnet → Haiku where viable, High → Medium). Opus only for steps that genuinely can't be done cheaper. Split aggressively to isolate Opus need. |
| **🛡️ Quality** | Bias steps one notch up. Sonnet steps that touch business logic → Opus. Medium effort → High where ambiguity exists. Add explicit review steps. |
| **⚡ Speed** | Prefer Haiku/Sonnet, Low/Medium effort. Skip optional polish steps (extensive doc updates, exhaustive tests). Note skipped items as "Speed-mode: deferred". |
| **🎓 Learning** | Add explanatory steps before implementation. Bias toward Opus on conceptually rich steps so the reasoning is clear. Each step's reason field gets more detail. |
| **⚖️ Balanced** | Defaults below — Sonnet/Medium baseline, Opus where genuinely needed. |

If you don't know the active goal, **ask the user once at the start of the plan**:

> "Quick check before I plan — what's the goal here? **Quality / Cost / Speed / Learning / Balanced** (default: Balanced)"

Then proceed with the plan. The Architect Rule (plan itself = Opus xHigh) **does not change with goal mode** — even Cost mode plans the plan on Opus, because a bad plan in cost mode wastes more money than it saves.

Mention the active goal in the plan's context paragraph so the user knows the routing reflects it.

---

## The Tag Format

Every plan step ends with a one-liner like this:

```
> 🤖 Sonnet 4.6 | ⚡ Medium — reason in <8 words
```

Format rules:
- One blockquote line directly under the step
- Model first, then effort, then a brief reason
- Keep the reason under 8 words — it's a sanity check, not an essay
- Use exact model names: **Haiku 4.5**, **Sonnet 4.6**, **Opus 4.7**

---

## Model Selection

Pick the cheapest model that can do the job well. Default to Sonnet — only go up to Opus or down to Haiku with a real reason.

| Model | Use For |
|-------|---------|
| **Haiku 4.5** | Boilerplate, renaming, formatting, simple file ops, copy-paste-with-tweaks, single-line fixes, scaffolding from templates, basic doc strings, reading files |
| **Sonnet 4.6** | Default. Multi-file features, normal debugging, standard refactors, writing tests, code review, API integration, most real work |
| **Opus 4.7** | Novel architecture decisions, complex system design, deep reasoning about tradeoffs, hard concurrency/state problems, security-critical code, anywhere a wrong choice costs hours later |

Do NOT use Opus for steps just because the overall feature is big. Opus is for **specific steps that need deep reasoning**, not for "this is important work."

---

## Effort Level Selection

Effort = how much extended thinking budget the model should use before answering.

| Effort | When |
|--------|------|
| **Low** | The answer is obvious. Pattern-matching to known solutions. Boilerplate. Mechanical edits. |
| **Medium** | Default. Normal feature work. Step requires some thought but no deep analysis. |
| **High** | Multiple valid approaches need weighing. Subtle bugs. Refactors with non-obvious ripple effects. |
| **xHigh** | Genuinely hard problem. Architecture decisions. Concurrency. Security. Anywhere a wrong choice costs hours later. |

Effort and model are **independent axes**. Examples:
- Haiku + Low = "rename this variable across the file"
- Sonnet + xHigh = "design the state machine for this complex flow"
- Opus 4.7 + Low = rare — usually if you need Opus you also need thinking

---

## What a Good Plan Looks Like (Architect Output)

Because planning runs on Opus xHigh, the plans should be **noticeably better than a quick breakdown**. A good CCPM plan includes:

1. **A short context paragraph** — what we're building, the key constraint, the assumed starting state, and the active goal mode if applicable.
2. **The real tradeoff up front** — if there's a fork in the road, name it before the steps.
3. **Ordered, sized steps** — each step is one model+effort combo's worth of work. If a step would need two different models, split it.
4. **A cost summary** — count of steps per model. Gives the user a sanity check on total spend.
5. **Optional: risk flags** — steps where the plan might need to change based on what's found.

The architect's job is to make execution boring. If the workers have to make architectural decisions, the architect failed.

---

## Use Case Examples

### Use Case 1: New Feature (Balanced goal)

```markdown
## Plan: Add WebSocket reconnection logic

**Context:** Goal mode: Balanced. Existing `client.ts` handles a single
connection. We need automatic reconnect with exponential backoff. Main
tradeoff: aggressive reconnect = faster recovery but server load risk.

1. **Read current connection logic in `client.ts`**
   > 🤖 Haiku 4.5 | ⚡ Low — file reading, no decisions

2. **Design exponential backoff strategy with jitter**
   > 🤖 Opus 4.7 | ⚡ xHigh — concurrency, easy to get subtly wrong

3. **Implement the reconnect loop**
   > 🤖 Sonnet 4.6 | ⚡ Medium — standard implementation from design

4. **Add reconnect events to the typed event emitter**
   > 🤖 Sonnet 4.6 | ⚡ Low — mechanical addition to existing pattern

5. **Write unit tests covering disconnect → reconnect → resume**
   > 🤖 Sonnet 4.6 | ⚡ Medium — test cases need thought, not just typing

6. **Update README with the new behavior**
   > 🤖 Haiku 4.5 | ⚡ Low — doc writing

**Cost summary:** 2× Haiku, 3× Sonnet, 1× Opus 4.7
```

### Use Case 2: Bug Hunt (Cost goal)

```markdown
## Plan: Investigate intermittent 500 error on /api/orders

**Context:** Goal mode: Cost. Production error rate ~0.3%. Optimizing for
cheapest viable path — only escalate models if cheaper attempts stall.

1. **Pull last 24h of error logs and group by stack trace**
   > 🤖 Haiku 4.5 | ⚡ Low — log filtering

2. **Identify the most likely root cause from grouped traces**
   > 🤖 Sonnet 4.6 | ⚡ Medium — reasoning across noisy data (Cost: try Medium first)

3. **Reproduce the bug locally**
   > 🤖 Sonnet 4.6 | ⚡ Low — standard repro workflow

4. **Patch the issue + add a regression test**
   > 🤖 Sonnet 4.6 | ⚡ Medium — implementation once cause is known

5. **Write the postmortem**
   > 🤖 Haiku 4.5 | ⚡ Low — writing from known facts

**Cost summary:** 2× Haiku, 3× Sonnet (no Opus — Cost mode escalation only if stalled)
```

### Use Case 3: Refactor (Quality goal)

```markdown
## Plan: Refactor auth from session cookies to JWT

**Context:** Goal mode: Quality. Security-critical refactor. Bias toward
more careful review and higher effort on anything touching token handling.

1. **Map all current touch points of the session system**
   > 🤖 Sonnet 4.6 | ⚡ High — Quality: don't miss any callers

2. **Design the JWT structure, claims, refresh strategy**
   > 🤖 Opus 4.7 | ⚡ xHigh — security-critical, hard to fix later

3. **Implement the new JWT module**
   > 🤖 Opus 4.7 | ⚡ High — Quality: token logic, no Sonnet shortcut here

4. **Migrate each route handler one-by-one**
   > 🤖 Sonnet 4.6 | ⚡ Medium — mechanical, but with adversarial review

5. **Add tests for token expiry, refresh, revocation**
   > 🤖 Sonnet 4.6 | ⚡ High — security tests need adversarial thinking

6. **Independent security review pass**
   > 🤖 Opus 4.7 | ⚡ xHigh — Quality mode: explicit review step

7. **Remove the old session code**
   > 🤖 Haiku 4.5 | ⚡ Low — deletion, no decisions

**Cost summary:** 1× Haiku, 3× Sonnet, 3× Opus 4.7
```

### Use Case 4: Greenfield Setup (Speed goal)

```markdown
## Plan: Scaffold new Next.js 15 project with Prisma + tRPC

**Context:** Goal mode: Speed. Hobby project, get to a runnable baseline
fast. Polish deferred.

1. **Run `create-next-app` and pick options**
   > 🤖 Haiku 4.5 | ⚡ Low — template scaffolding

2. **Add Prisma, generate initial schema based on requirements**
   > 🤖 Sonnet 4.6 | ⚡ Low — Speed: good-enough schema, iterate later

3. **Set up tRPC router structure**
   > 🤖 Haiku 4.5 | ⚡ Low — Speed: boilerplate clone

4. **Configure auth (NextAuth or similar)**
   > 🤖 Sonnet 4.6 | ⚡ Low — Speed: minimum viable config

5. **Create base layout + theme tokens**
   > 🤖 Haiku 4.5 | ⚡ Low — Speed: throwaway styling

**Speed-mode: deferred:** Docs, exhaustive tests, theme polish.

**Cost summary:** 3× Haiku, 2× Sonnet
```

### Use Case 5: Performance Optimization

```markdown
## Plan: Reduce dashboard load time from 4s to under 1s

**Context:** Goal mode: Balanced. Profile first, optimize second. Tradeoff
to surface during design: SSR streaming vs aggressive client caching.

1. **Profile current load: network, render, hydration**
   > 🤖 Sonnet 4.6 | ⚡ Medium — interpretation of profiling data

2. **Identify the 2-3 biggest bottlenecks**
   > 🤖 Sonnet 4.6 | ⚡ High — prioritization needs judgement

3. **Design the caching/streaming strategy**
   > 🤖 Opus 4.7 | ⚡ xHigh — tradeoffs (staleness vs speed) are non-trivial

4. **Implement query batching**
   > 🤖 Sonnet 4.6 | ⚡ Medium — known pattern

5. **Add Suspense boundaries + skeleton loaders**
   > 🤖 Sonnet 4.6 | ⚡ Low — React idioms

6. **Re-measure and document the gains**
   > 🤖 Haiku 4.5 | ⚡ Low — writing up numbers

**Cost summary:** 1× Haiku, 4× Sonnet, 1× Opus 4.7
```

### Use Case 6: Migration (Learning goal)

```markdown
## Plan: Migrate JavaScript codebase to TypeScript

**Context:** Goal mode: Learning. User wants to understand TS migration
strategy deeply. Each step explains its reasoning, not just what to do.

1. **Explain the two strategy options: bottom-up vs top-down, with tradeoffs**
   > 🤖 Opus 4.7 | ⚡ xHigh — Learning: pedagogically rich explanation

2. **Add tsconfig.json with strict: false initially, walk through each setting**
   > 🤖 Sonnet 4.6 | ⚡ Medium — Learning: explain why each option matters

3. **Convert one utility module together as an example, narrating each type added**
   > 🤖 Opus 4.7 | ⚡ High — Learning: this is the teaching step

4. **Convert remaining utility/leaf modules**
   > 🤖 Sonnet 4.6 | ⚡ Low — user does these, asks if stuck

5. **Add proper types to shared interfaces — pair-programming style**
   > 🤖 Sonnet 4.6 | ⚡ Medium — Learning: discuss design choices

6. **Flip strict: true and fix errors, explaining each category of error**
   > 🤖 Opus 4.7 | ⚡ High — Learning: errors are teaching moments

**Cost summary:** 1× Sonnet (Low), 2× Sonnet (Medium), 3× Opus 4.7
```

### Use Case 7: Security Audit

```markdown
## Plan: Audit authentication and session handling

**Context:** Goal mode: Quality (forced — security work). Pre-launch audit.
Find issues, don't patch in this pass.

1. **List all auth-related endpoints and middleware**
   > 🤖 Haiku 4.5 | ⚡ Low — code grepping

2. **Review each endpoint for OWASP top 10 risks**
   > 🤖 Opus 4.7 | ⚡ xHigh — adversarial thinking

3. **Check token storage, rotation, and revocation flows**
   > 🤖 Opus 4.7 | ⚡ High — security-critical reasoning

4. **Write up findings with severity + remediation**
   > 🤖 Sonnet 4.6 | ⚡ Medium — structured writing from analysis

5. **File issues in the tracker**
   > 🤖 Haiku 4.5 | ⚡ Low — mechanical issue creation

**Cost summary:** 2× Haiku, 1× Sonnet, 2× Opus 4.7
```

---

## Important Heuristics

- **The plan itself is always Opus 4.7 + xHigh — that's non-negotiable.** Saving cost on the plan is false economy; you pay it back 10x in bad execution. This holds even in Cost mode.
- **When in doubt on a step, default to Sonnet + Medium.** That's the safe middle (in Balanced mode).
- **Splitting a big step is cheaper than upgrading the model.** If one step would need Opus xHigh, ask: can I split the hard reasoning part from the implementation part? Then only the reasoning step needs Opus.
- **Reading/exploration is almost always Haiku Low.** Looking at files, listing directories, grepping — Haiku handles it.
- **The first step of a feature is rarely Opus.** Discovery and setup aren't the hard part. Save Opus for the actual hard decisions.
- **If every step is Opus xHigh, you're not planning — you're panicking.** Re-evaluate the plan.
- **"Important" ≠ "needs Opus".** A critical security fix might still be a one-line change once you know what to do. The thinking part needs Opus, not the patch.
- **Security work overrides Cost/Speed goals.** See the Security Baseline in CCGM — applies to all steps regardless of goal or mode.

---

## When NOT to Annotate

Skip the architect mode and the tags for:
- Single-step asks ("fix this typo")
- Conversational exploration ("what's the best way to…")
- Reviews of existing plans where the user just wants feedback, not a re-plan
- Pure Q&A with no implementation involved

In these cases, just respond normally — no planning, no tags.

---

## Integration with TodoWrite / ExitPlanMode

When using the TodoWrite tool or presenting a plan via ExitPlanMode:
- Put the tag at the **end of each todo item's content**, in parentheses
- Example: `Implement reconnect loop (🤖 Sonnet 4.6 | ⚡ Medium)`
- Keep the full annotated plan in your message body for reference

---

## Trigger Phrases

Engage architect mode when **at least one** of these is true:

| Condition | Example |
|-----------|---------|
| User uses explicit planning language | "plan this", "make a plan", "roadmap for", "break this down", "break it into steps" |
| Task has 3+ recognizably dependent steps | "migrate the DB, update the API, then fix the frontend" |
| User asks for a todo list for a multi-step feature | "what are the steps to implement auth?", "todo list for adding payments" |

**Do not trigger on:**
- "implement X" or "build X" alone without multi-step context
- "fix this", "add this" for a single-file or single-concept change
- "refactor X" for a single targeted change
- Conversational questions ("what's the best way to...")
- Anything already covered by "When NOT to Annotate" above

**The test:** Would a senior engineer naturally reach for a checklist here? If yes, trigger. If the work fits in one focused session without sub-decisions, don't.
