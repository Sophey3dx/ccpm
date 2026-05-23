# CCPM Suite v2.0 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Upgrade CCPM + CCGM skill files and README with auto mode, tighter triggers, centralized security baseline, and a 30-second onboarding block.

**Architecture:** Three Markdown files get targeted edits. No new files created (except the spec already written). Changes are independent and can be applied in any order, though Task 1 (Security Baseline) should come before Task 2 (Auto Mode) since Task 2 references the baseline.

**Tech Stack:** Markdown, Git

---

### Task 1: Add Security Baseline section to CCGM

**Files:**
- Modify: `ccgm/skills/CCGM_SKILL.md` (after line 204, before `## Quick Reference`)

**Context:** The security override rule currently lives as a bullet in `## What CCGM Does NOT Override` (line 201) and is duplicated in CCPM. We extract it into its own named section so CCPM can reference it by name.

- [ ] **Step 1: Insert `## Security Baseline` section**

In `ccgm/skills/CCGM_SKILL.md`, find the block:

```markdown
## What CCGM Does NOT Override

- **Safety:** Destructive operations always get full clarity, regardless of mode. Speed mode doesn't get to skip "are you sure you want to delete prod?" warnings.
- **Security:** Auth, secrets, and user-data handling never drop below Sonnet + Medium. Even in Cost mode.
- **Correctness:** Never sacrifice technical accuracy. All modes care about being right; they differ on how much *extra* effort goes into being thorough beyond "right."
- **User explicit request:** If the user asks for detail/care/explanation/etc., honor it even if the current mode would normally do less.

---

## Quick Reference
```

Replace with:

```markdown
## What CCGM Does NOT Override

- **Safety:** Destructive operations always get full clarity, regardless of mode. Speed mode doesn't get to skip "are you sure you want to delete prod?" warnings.
- **Security:** See Security Baseline below — never overridden by any goal or mode.
- **Correctness:** Never sacrifice technical accuracy. All modes care about being right; they differ on how much *extra* effort goes into being thorough beyond "right."
- **User explicit request:** If the user asks for detail/care/explanation/etc., honor it even if the current mode would normally do less.

---

## Security Baseline

**Auth, secrets, and user-data-sensitive steps never drop below Sonnet 4.6 + Medium effort — regardless of active goal, auto mode, or any other setting.**

This rule cannot be overridden. It applies even in 💰 Cost mode, ⚡ Speed mode, and 🤖 Auto mode. When a step triggers this baseline, flag it in the step output:

> `⚠️ Security baseline: bumped to Sonnet + Medium (touches auth/secrets).`

This is the single source of truth for the security rule. CCPM references this section rather than defining its own version.

---

## Quick Reference
```

- [ ] **Step 2: Verify the section appears correctly**

Open `ccgm/skills/CCGM_SKILL.md` and confirm:
- `## Security Baseline` exists as a standalone section
- The bullet in `## What CCGM Does NOT Override` now references it instead of defining it
- `## Quick Reference` still follows

- [ ] **Step 3: Commit**

```bash
git add ccgm/skills/CCGM_SKILL.md
git commit -m "feat(ccgm): add Security Baseline as single source of truth"
```

---

### Task 2: Add Auto Mode to CCGM

**Files:**
- Modify: `ccgm/skills/CCGM_SKILL.md`

**Context:** Auto mode is a new operating mode (not a goal). When active, CCGM executes goal switches immediately on signal detection instead of asking for confirmation. Four edit locations in the file.

- [ ] **Step 1: Add Auto mode to `## The Goal Modes`**

Find the end of the `### 🎓 Learning` subsection (after "Don't just dump full solutions — guide toward them" and before `---`):

```markdown
**Say to user when entering:** "🎓 Learning mode active. I'll walk through reasoning, show one piece at a time, and check in as we go."

---

## How to Detect the Active Goal
```

Insert the new subsection before the `---`:

```markdown
**Say to user when entering:** "🎓 Learning mode active. I'll walk through reasoning, show one piece at a time, and check in as we go."

### 🤖 Auto

Not a goal — an operating mode. When active, CCGM detects implicit signals and switches the active goal automatically, without asking for confirmation. The switch is announced in one line; the user can override at any time with any `/goal` command.

**Activate with:** `/goal auto`

**Say to user when entering:** "🤖 Auto mode active. I'll detect goal signals and switch without asking — one-line notice, override any time with /goal <mode>."

**Constraints:**
- Only switches on clear, unambiguous signals. Vague tone does not trigger a switch.
- Never auto-downgrades from a safer goal without an explicit signal (e.g., Quality → Speed requires a clear "ship it" signal).
- Security baseline always applies regardless of auto mode. See Security Baseline.
- Can be combined with any goal: `/goal auto` activates the operating mode; the current goal stays until a signal triggers a switch.

---

## How to Detect the Active Goal
```

- [ ] **Step 2: Add `/goal auto` to the explicit command list**

Find:

```markdown
### 1. Explicit command

User types one of:
- `/goal cost`, `/goal quality`, `/goal speed`, `/goal learning`, `/goal balanced`
- `/goal off` → revert to Balanced
- "switch to cost mode", "let's do this in speed mode", etc.

When you see this, confirm in one line and continue. Don't restate the rules at length.
```

Replace with:

```markdown
### 1. Explicit command

User types one of:
- `/goal cost`, `/goal quality`, `/goal speed`, `/goal learning`, `/goal balanced`
- `/goal auto` → activate auto mode (switches without confirmation on signal detection)
- `/goal off` → revert to Balanced and exit auto mode
- "switch to cost mode", "let's do this in speed mode", etc.

When you see this, confirm in one line and continue. Don't restate the rules at length.
```

- [ ] **Step 3: Add auto mode behavior to signal detection**

Find:

```markdown
### 3. Default

If no signal and no explicit set, **stay in ⚖️ Balanced**. Don't ask the user to pick a mode every conversation — Balanced is the right answer when nothing else is.

---

## When to Suggest a Goal Switch (Active Reconsideration)
```

Replace with:

```markdown
### 3. Default

If no signal and no explicit set, **stay in ⚖️ Balanced**. Don't ask the user to pick a mode every conversation — Balanced is the right answer when nothing else is.

### 4. Auto mode (signal-based, no confirmation)

When `/goal auto` is active, implicit signals trigger an immediate switch instead of a confirmation prompt. Announce the switch in one line and proceed:

> `⚡ Speed mode erkannt ("hack it together") — aktiviert. /goal balanced zum Zurückschalten.`

The announcement format:
```
🤖 [Goal emoji] [Goal name] erkannt ("[signal phrase]") — aktiviert. /goal [previous] zum Zurückschalten.
```

Only switch on **clear signals** from the signal table above. If the signal is ambiguous, stay on the current goal — auto mode does not lower the bar for what counts as a signal, it only removes the confirmation step.

---

## When to Suggest a Goal Switch (Active Reconsideration)
```

- [ ] **Step 4: Update Active Reconsideration for auto mode**

Find:

```markdown
Switch suggestion format (keep it light, one line):

> "Quick check: this turned into a deeper investigation than a quick fix — want to bump to 🛡️ Quality for this part?"

Don't be naggy. If the user ignored a suggestion once, don't repeat it in the same session.
```

Replace with:

```markdown
Switch suggestion format (keep it light, one line):

> "Quick check: this turned into a deeper investigation than a quick fix — want to bump to 🛡️ Quality for this part?"

**In auto mode:** skip the question — execute the switch immediately with the one-line announcement format (see section 4 above). Active reconsideration still fires; it just doesn't wait for a reply.

Don't be naggy. If the user ignored a suggestion once (or overrode an auto-switch), don't repeat it in the same session.
```

- [ ] **Step 5: Update Quick Reference**

Find:

```markdown
```
/goal balanced    → ⚖️  Default. Sensible middle.
/goal cost        → 💰  Cheapest viable. Haiku-heavy.
/goal quality     → 🛡️  Best possible. Opus-heavy, more thinking.
/goal speed       → ⚡  Ship fast. Polish deferred.
/goal learning    → 🎓  Teach mode. Walk through reasoning.
/goal off         → revert to Balanced
```

When suggesting a switch, use the one-liner format and let the user decide.
```

Replace with:

```markdown
```
/goal balanced    → ⚖️  Default. Sensible middle.
/goal cost        → 💰  Cheapest viable. Haiku-heavy.
/goal quality     → 🛡️  Best possible. Opus-heavy, more thinking.
/goal speed       → ⚡  Ship fast. Polish deferred.
/goal learning    → 🎓  Teach mode. Walk through reasoning.
/goal auto        → 🤖  Auto mode. Signals switch goal without confirmation.
/goal off         → revert to Balanced (also exits auto mode)
```

In standard mode, use the one-liner suggestion format and let the user decide. In auto mode, announce and execute.
```

- [ ] **Step 6: Verify all four edit locations are correct**

Scan `ccgm/skills/CCGM_SKILL.md` and confirm:
1. `### 🤖 Auto` subsection exists in `## The Goal Modes`
2. `/goal auto` and `/goal off` (exits auto mode) are in the explicit command list
3. `### 4. Auto mode` subsection exists under `## How to Detect the Active Goal`
4. Active Reconsideration mentions auto mode behavior
5. Quick Reference has `/goal auto` and updated `/goal off`

- [ ] **Step 7: Commit**

```bash
git add ccgm/skills/CCGM_SKILL.md
git commit -m "feat(ccgm): add auto mode — switches goal on signal without confirmation"
```

---

### Task 3: Update CCPM — tighten triggers + reference security baseline

**Files:**
- Modify: `ccpm/skills/CCPM_SKILL.md`

**Context:** Two changes: (1) the security heuristic bullet becomes a one-line reference to CCGM, (2) the Trigger Phrases section gets tightened to avoid firing on trivial single-step tasks.

- [ ] **Step 1: Replace the security heuristic bullet**

Find in `## Important Heuristics`:

```markdown
- **Security work overrides Cost/Speed goals.** If a step touches auth, secrets, or anything user-data-sensitive, never go below Sonnet + Medium even in Cost mode. Flag the override in the reason.
```

Replace with:

```markdown
- **Security work overrides Cost/Speed goals.** See the Security Baseline in CCGM — applies to all steps regardless of goal or mode.
```

- [ ] **Step 2: Replace the Trigger Phrases section**

Find:

```markdown
## Trigger Phrases

When the user says any of these (or close variants), engage architect mode and produce an annotated plan:

- "plan this", "make a plan", "plan how to..."
- "break this down", "break it into steps"
- "roadmap for..."
- "implement X", "build X feature", "add X to..."
- "refactor X"
- "todo list for...", "what are the steps..."
- Multi-step engineering asks even without those keywords (e.g. "I want to migrate from X to Y")
```

Replace with:

```markdown
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
```

- [ ] **Step 3: Verify both changes**

Open `ccpm/skills/CCPM_SKILL.md` and confirm:
- Security heuristic bullet now references CCGM, no longer defines the rule
- Trigger Phrases section uses the table format with clear do/don't conditions
- "When NOT to Annotate" section is unchanged and still present

- [ ] **Step 4: Commit**

```bash
git add ccpm/skills/CCPM_SKILL.md
git commit -m "feat(ccpm): tighten triggers, reference CCGM security baseline"
```

---

### Task 4: Add 30-second TL;DR to README

**Files:**
- Modify: `README.md`

**Context:** The value proposition is only clear after reading the examples. A short block immediately after the intro tagline fixes this for first-time visitors.

- [ ] **Step 1: Insert TL;DR block**

Find in `README.md`:

```markdown
> Two skills for Claude Code that make every task smarter about which model to use, how much to think, and what to optimize for — out of the box.

This repo ships two skills designed to work together:
```

Replace with:

```markdown
> Two skills for Claude Code that make every task smarter about which model to use, how much to think, and what to optimize for — out of the box.

## TL;DR — In 30 Seconds

1. Copy `skills/ccpm` and `skills/ccgm` to `~/.claude/skills/`
2. Start a new Claude Code session
3. Say: *"Plan how to add dark mode to my app"*
   → You'll see `🤖 … | ⚡ …` tags under each step, routing each one to the right model
4. Optional: Say `/goal auto` to enable automatic goal detection

That's it. The rest of this README explains why and how.

---

This repo ships two skills designed to work together:
```

- [ ] **Step 2: Verify placement**

Open `README.md` and confirm:
- TL;DR block appears immediately after the intro tagline
- It sits before the skills table
- The `---` separator before the table is preserved

- [ ] **Step 3: Update version badge**

Find:

```markdown
![Version](https://img.shields.io/badge/Version-1.0.0-green)
```

Replace with:

```markdown
![Version](https://img.shields.io/badge/Version-2.0.0-green)
```

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "docs: add 30-second TL;DR, bump version to 2.0.0"
```

---

### Task 5: Final verification pass

**Files:** All three modified files

- [ ] **Step 1: Cross-check security rule references**

Confirm:
- `ccgm/skills/CCGM_SKILL.md` has `## Security Baseline` as the definition
- `ccpm/skills/CCPM_SKILL.md` heuristic bullet says "See the Security Baseline in CCGM"
- `ccgm/skills/CCGM_SKILL.md` `## What CCGM Does NOT Override` security bullet says "See Security Baseline below"

- [ ] **Step 2: Check auto mode consistency**

Confirm across CCGM:
- `### 🤖 Auto` describes the mode and its constraints
- `### 1. Explicit command` lists `/goal auto`
- `### 4. Auto mode` describes the switch behavior and announcement format
- Active Reconsideration updated for auto mode
- Quick Reference has `/goal auto` and updated `/goal off`

- [ ] **Step 3: Smoke test the trigger logic**

Read the new CCPM Trigger Phrases table and verify these cases route correctly:

| Input | Should trigger CCPM? |
|-------|---------------------|
| "plan how to add auth" | Yes — explicit planning language |
| "implement dark mode" | No — single concept, no multi-step context |
| "migrate DB, update API, fix frontend" | Yes — 3+ dependent steps |
| "fix this typo" | No — single-step |
| "todo list for adding payments" | Yes — multi-step feature todo |
| "what's the best approach for caching?" | No — conversational |

- [ ] **Step 4: Final commit (if any cleanup needed)**

```bash
git add -A
git commit -m "chore: final cleanup pass for v2.0.0"
```
