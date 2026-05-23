# CCPM Suite v2.0 — Design Spec

**Date:** 2026-05-23  
**Status:** Approved  
**Scope:** Three improvements to the CCPM + CCGM skill pair

---

## Context

The CCPM Suite ships two Claude Code skills that work together:
- **CCPM** — annotates plan steps with model + effort tags
- **CCGM** — sets an active optimization goal (Cost / Quality / Speed / Learning / Balanced)

v2.0 addresses three issues identified in review:
1. CCGM always asks before switching goals — friction for experienced users
2. CCPM triggers too broadly — fires on trivial single-step tasks
3. Security override rule is duplicated across both skills; README buries the value proposition

---

## Change 1: CCGM Auto Mode

### What

A new goal state: `auto`. When active, CCGM detects implicit signals and switches the goal automatically — without asking for confirmation. Instead it announces the switch in one line and proceeds. The user can override at any time.

### Activation

```
/goal auto   → 🤖 Auto mode active. I'll detect goal signals and switch without asking.
```

Auto mode is an *operating mode*, not a goal itself. The active goal can still be any of the five (Cost / Quality / Speed / Learning / Balanced). `/goal auto` just means "don't ask, just switch."

### Behavior

When a signal is detected in auto mode:

1. Switch the goal immediately
2. Announce in one line:
   > `⚡ Speed mode erkannt ("hack it together") — aktiviert. /goal balanced zum Zurückschalten.`
3. Continue with the task under the new goal

No waiting for confirmation.

### Constraints

- **Only switch on clear signals.** Vague language does not trigger a switch.
- **Never auto-downgrade from a safer goal without an explicit signal.** Quality → Speed requires a clear "ship it / just make it work" signal, not just a casual tone.
- **Security baseline always applies**, regardless of auto mode or active goal. See Change 3.
- **Active reconsideration still fires** — the existing "suggest a switch" logic runs, but in auto mode it executes the switch instead of asking.

### Quick Reference addition

```
/goal auto     → 🤖  Auto mode. Signals detected, switches run without confirmation.
/goal off      → revert to Balanced (also exits auto mode)
```

---

## Change 2: CCPM Trigger Tightening

### Problem

Current trigger phrases include "implement X", "build X feature", "add X to..." — these are too broad. A single-step task like "add a console.log here" should not produce a full annotated plan.

### New Trigger Rule

CCPM activates when **at least one** of these is true:

| Condition | Example |
|-----------|---------|
| User uses explicit planning language | "plan this", "roadmap for", "break this down", "steps for" |
| Task has 3+ recognizably dependent steps | "migrate the DB, update the API, then fix the frontend" |
| User asks for a todo list for a feature | "what are the steps to implement auth?" |

### No longer triggers on

- "implement X" or "build X" alone, without multi-step context
- "fix this", "add this" for single-file or single-concept changes
- Conversational exploration ("what's the best way to...")

### Unchanged

The "When NOT to Annotate" section in CCPM already covers single-step asks and Q&A. This change tightens the *trigger detection* to match that section, not the output behavior.

---

## Change 3: Security Baseline Centralization + README

### Security Baseline

**Single source of truth: CCGM.**

CCGM gets a dedicated `## Security Baseline` section:

> Auth, secrets, and user-data-sensitive steps never drop below **Sonnet 4.6 + Medium effort**, regardless of active goal or auto mode. This rule cannot be overridden by any goal setting.

CCPM removes its duplicated security text and replaces it with a one-line reference:

> *Security baseline: see CCGM — applies to all steps regardless of goal.*

### README: 30-Second Block

A new section inserted immediately after the intro tagline, before the current "Why?" section:

```markdown
## TL;DR — In 30 Sekunden

1. Kopiere `skills/ccpm` und `skills/ccgm` nach `~/.claude/skills/`
2. Starte eine neue Claude Code Session
3. Sag: "Plan how to add dark mode to my app"
   → Du siehst 🤖/⚡-Tags unter jedem Schritt
4. Optional: Sag "/goal auto" für automatische Goal-Erkennung

Das war's. Der Rest der README erklärt warum und wie.
```

---

## Files Changed

| File | Change |
|------|--------|
| `ccgm/skills/CCGM_SKILL.md` | Add `## Security Baseline` section; add auto mode to goal states, quick reference, detection logic, active reconsideration |
| `ccpm/skills/CCPM_SKILL.md` | Tighten trigger phrases; replace duplicated security text with CCGM reference |
| `README.md` | Insert 30-second TL;DR block before "Why?" section |

---

## Out of Scope

- New goal modes beyond the existing five
- Persistence of goal state across sessions
- Integration with other skills beyond current CCPM ↔ CCGM relationship
