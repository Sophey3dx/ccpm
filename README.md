# CCPM Suite

![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
![Made for Claude Code](https://img.shields.io/badge/Made%20for-Claude%20Code-D97757)
![Skills: 2](https://img.shields.io/badge/Skills-2-blue)
![Version](https://img.shields.io/badge/Version-2.0.0-green)
![Status](https://img.shields.io/badge/Status-Active-success)
![Stars](https://img.shields.io/github/stars/Sophey3dx/ccpm?style=social)
![Last Commit](https://img.shields.io/github/last-commit/Sophey3dx/ccpm)
![Issues](https://img.shields.io/github/issues/Sophey3dx/ccpm)

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

| Skill | What it does |
|-------|--------------|
| 🏛️ **[CCPM](skills/ccpm/SKILL.md)** | Claude Code Plan Management — annotates every plan step with the right model + effort, plans on Opus 4.7 xHigh so the routing is good |
| 🎯 **[CCGM](skills/ccgm/SKILL.md)** | Claude Code Goal Management — lets you set an active goal (Cost / Quality / Speed / Learning / Balanced) that adapts every task |

You can install one or both. They reference each other but each one stands on its own.

---

## Why?

Claude Code by default uses whatever model the session was started on, with default thinking budgets, and treats every task the same. That's fine — but it leaves money and quality on the table:

- A 6-step feature plan might have 4 steps that Haiku could handle and 1 step that genuinely needs Opus 4.7 + deep thinking. Running the whole thing on Opus is wasteful. Running the whole thing on Sonnet risks getting the architecture step wrong.
- Different sessions have different goals. "I'm prototyping for fun" and "I'm shipping to prod" should not be served identically. CCGM gives the user a knob.

**The TL;DR:**
- CCPM picks the right model+effort *per step*.
- CCGM picks the right defaults *per session*.

---

## Quick Example

You ask:
> "Plan how to add WebSocket reconnection to my client library."

Without CCPM:
```
1. Read current connection logic
2. Design backoff strategy
3. Implement reconnect loop
4. Write tests
```

With CCPM (plan ran on Opus 4.7 xHigh):
```
Context: Goal mode: Balanced. Existing client.ts handles a single connection.
Tradeoff: aggressive reconnect = faster recovery but server load risk.

1. Read current connection logic
   > 🤖 Haiku 4.5 | ⚡ Low — file reading, no decisions

2. Design exponential backoff strategy with jitter
   > 🤖 Opus 4.7 | ⚡ xHigh — concurrency, easy to get subtly wrong

3. Implement the reconnect loop
   > 🤖 Sonnet 4.6 | ⚡ Medium — standard implementation from design

4. Write unit tests covering disconnect → reconnect → resume
   > 🤖 Sonnet 4.6 | ⚡ Medium — test cases need thought

Cost summary: 1× Haiku, 2× Sonnet, 1× Opus 4.7
```

With CCGM in **💰 Cost mode**, the same plan shifts:
- Step 2's Opus might split into "design (Opus xHigh)" and "translate to spec (Sonnet)"
- Steps 3 and 4 drop to Low/Medium effort
- The cost summary skews lighter

With CCGM in **🛡️ Quality mode**, it shifts the other way — more Opus, more thinking, explicit review steps added.

---

## Installation

### Both skills (recommended)

**macOS / Linux:**
```bash
mkdir -p ~/.claude/skills
git clone https://github.com/Sophey3dx/ccpm.git /tmp/ccpm
cp -r /tmp/ccpm/skills/ccpm ~/.claude/skills/
cp -r /tmp/ccpm/skills/ccgm ~/.claude/skills/
rm -rf /tmp/ccpm
```

**Windows (PowerShell):**
```powershell
$skillsDir = "$env:USERPROFILE\.claude\skills"
New-Item -ItemType Directory -Force -Path $skillsDir | Out-Null
git clone https://github.com/Sophey3dx/ccpm.git $env:TEMP\ccpm
Copy-Item -Recurse $env:TEMP\ccpm\skills\ccpm $skillsDir
Copy-Item -Recurse $env:TEMP\ccpm\skills\ccgm $skillsDir
Remove-Item -Recurse -Force $env:TEMP\ccpm
```

After install, the layout should be:
```
~/.claude/skills/
├── ccpm/
│   └── SKILL.md
└── ccgm/
    └── SKILL.md
```

### Just one skill

If you only want one, copy only that folder. They work independently.

### Verify

Start a new Claude Code session.

**CCPM check:**
> "Plan how to add a dark mode toggle to my app."

You should see a context paragraph and `🤖 ... | ⚡ ...` tags under each step.

**CCGM check:**
> "Switch to cost mode."

Claude should confirm `💰 Cost mode active.` and adapt subsequent tasks.

---

## How They Work Together

```
User asks for a plan
         │
         ▼
┌─────────────────┐
│   CCGM checks   │  ← active goal: Cost / Quality / Speed / Learning / Balanced
│ for active goal │
└────────┬────────┘
         │
         ▼
┌─────────────────────┐
│   CCPM produces a   │  ← always Opus 4.7 + xHigh for the plan itself
│  goal-aware plan,   │
│  annotated per step │
└────────┬────────────┘
         │
         ▼
┌─────────────────┐
│  User executes  │  ← runs each step on the suggested model/effort
│  step-by-step   │
└─────────────────┘
```

If only CCPM is installed: plans assume Balanced goal.
If only CCGM is installed: goals affect everything *except* per-step plan routing.

---

## Goal Modes Cheat Sheet (CCGM)

| Goal | What it does |
|------|--------------|
| ⚖️ **Balanced** | Default. Sensible middle. |
| 💰 **Cost** | Haiku-heavy, low thinking, terse output. Cheapest viable. |
| 🛡️ **Quality** | Opus-heavy, high thinking, explicit review steps. Best possible. |
| ⚡ **Speed** | Lean models, low effort, skip polish. Ship fast. |
| 🎓 **Learning** | Walk through reasoning, one piece at a time, check understanding. |

```
/goal cost       → 💰
/goal quality    → 🛡️
/goal speed      → ⚡
/goal learning   → 🎓
/goal balanced   → ⚖️ (or /goal off)
```

CCGM also **actively suggests switches** when context shifts — e.g. a "quick fix" balloons into a deep investigation, or a learning session pivots to a prod deploy.

---

## Model Cheat Sheet (CCPM)

| Model | Use For |
|-------|---------|
| **Haiku 4.5** | Reading, renaming, formatting, boilerplate, mechanical edits |
| **Sonnet 4.6** | Default. Features, debugging, refactors, tests, code review |
| **Opus 4.7** | Architecture, concurrency, security, novel problems — and ALL planning |

| Effort | When |
|--------|------|
| **Low** | Obvious answer, pattern matching |
| **Medium** | Default. Normal feature work |
| **High** | Multiple approaches to weigh, subtle bugs |
| **xHigh** | Architecture, concurrency, security, hard tradeoffs — and ALL planning |

---

## Customizing

Both skills are single Markdown files in `skills/<name>/SKILL.md`. To customize:

1. Fork this repo (or just copy the file)
2. Edit the SKILL.md — tweak the rules, add use cases, change defaults
3. Save to `~/.claude/skills/<name>/SKILL.md`

Common tweaks:
- **Default to Cost mode** for a personal side-project setup
- **Add stack-specific use cases** (Unity/Blender, mobile, etc.)
- **Loosen the Opus criteria** if you have a lot of Opus budget
- **Tighten Speed mode** to drop more polish

---

## License

MIT — see [LICENSE](LICENSE).

---

## Contributing

PRs welcome, especially:
- New use case examples in CCPM for domains not yet covered
- New goal-switch heuristics in CCGM for context shifts not yet captured
- Better trigger phrases in either skill
- Tweaks that survive real-world usage

Open an issue first for bigger changes so we can talk through it.
