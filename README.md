# scope-guard

Stop Claude from touching files you didn't ask it to touch.

[![GitHub stars](https://img.shields.io/github/stars/grxkun/scope-guard?style=flat)](https://github.com/grxkun/scope-guard/stargazers) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE) [![X](https://img.shields.io/badge/X-@grxkun-black)](https://x.com/grxkun)

A portable Claude Code skill that forces a **Scope Declaration** before any edit — a confirmed list of exactly which files will be touched, what changes, and what stays alone. Anything spotted outside scope goes to a **Notices log**, not a silent fix.


## Quick demo

**Request:** "the submit button on checkout is throwing a console error, fix it"

**Without scope-guard:**

Claude fixes the error, notices the form validation "could be cleaner," refactors it into a custom hook, creates a new `hooks/` directory, updates three other forms that use similar logic, renames `handleSubmit` to `onFormSubmit` across all of them, and adds a `useFormState` utility "that'll be useful later." 340-line diff. 8 files. You asked for one console error fix.

**With scope-guard:**

```
SCOPE DECLARATION
─────────────────
Task:        Fix console error on checkout form submit button
Files I will touch:
  - src/components/CheckoutForm.tsx → fix undefined reference on line 84
Files I will NOT touch (even if I see problems):
  - src/components/ShippingForm.tsx — noticed same pattern, leaving alone
Blast radius: S
─────────────────
Proceed? (yes / adjust scope)
```

You say yes. Claude fixes line 84. Diff is 3 lines.

```
NOTICES (not fixed — outside scope)
─────────────────────────────────────
• src/components/ShippingForm.tsx:61 — same undefined reference pattern, likely same bug
─────────────────────────────────────
Run /scope-guard to address any of these.
```

You know about the ShippingForm bug. You decide when to fix it.

## Why a skill, not just a prompt

A one-shot "don't touch other files" instruction works once. This skill is different:

- **Structured declaration format** — not a vague promise. A specific list of files, with a reason for each one, confirmed before anything is written.
- **Blast radius rating** — S / M / L tells you immediately how contained the change is. L requires explicit confirmation. Claude proposes a smaller scope first.
- **Mid-edit re-declaration** — if Claude discovers mid-edit that it needs an unlisted file, it stops and re-declares instead of silently expanding scope.
- **Notices, not fixes** — problems spotted outside scope get logged in a structured format, not quietly corrected. You decide what to address next.
- **6-entry anti-pattern table** — not vibes-based. Specific prohibited behaviors: "clean up while I'm here," "rename for clarity," "add a helper utility," "fix this related bug," unprompted refactors, silent scope expansion.

## Installation & Usage

### Claude Code

**Option 1: Clone into skills directory**

```bash
git clone https://github.com/grxkun/scope-guard ~/.claude/skills/scope-guard
```

**Option 2: Copy the file directly**

Download `SKILL.md` and place it in any directory Claude Code can read. Reference it in your `CLAUDE.md`:

```
- When fixing bugs or editing code → read `path/to/scope-guard/SKILL.md`
```

**Option 3: Use as a slash command**

Create `~/.claude/commands/scope-guard.md`:

```
---
description: Fix or change code while staying strictly within declared scope
---

$ARGUMENTS

Read and follow the instructions in ~/.claude/skills/scope-guard/SKILL.md
```

Then use `/scope-guard fix the login button color` in Claude Code.

### Triggering the skill

Once installed, the skill auto-triggers on:

- "fix this bug"
- "change this function"
- "refactor this component"
- "update the styles"
- "clean this up"
- Any request to modify existing code or files

Every run returns three sections:

1. **Scope Declaration** — presented before any edits. Requires your confirmation.
2. **Execution** — only what was declared.
3. **Notices log** — what was spotted outside scope, not fixed.

## Blast Radius Reference

| Rating | Meaning |
|---|---|
| **S** | 1–2 files, no shared logic. Diff readable in 30 seconds. Default target. |
| **M** | 3–5 files, or touches shared utilities. Claude names every consumer of shared logic. |
| **L** | 6+ files, or touches core abstractions. Claude proposes S/M alternative first. Requires explicit confirmation to proceed. |

## Anti-pattern table

| What Claude wants to do | What scope-guard makes it do instead |
|---|---|
| "Clean up the formatting while I'm here" | Log it in Notices |
| "Rename this variable to something cleaner" | Only if inside the specific function being changed |
| "Add a helper utility that would make this cleaner" | Do the task directly. No new abstractions. |
| "Fix this related bug I noticed" | Log it in Notices |
| "Refactor this to be more readable" | Only if explicitly asked |
| "Update the tests to match" | Yes — but declare the test files in Scope Declaration first |

## Credits

Pain documented by:

- r/ClaudeCode — "Claude refactored my entire codebase when I asked it to fix a typo" (Sep 2025, 847 upvotes)
- Kelsey Piper, *The Argument* — ["I can't stop yelling at Claude Code"](https://www.theargumentmag.com/p/i-cant-stop-yelling-at-claude-code) (Jan 2026)
- Mikhail Shcheglov, *Corporate Waters* — ["The Ultimate Guide to Claude Code Skills"](https://corpwaters.substack.com/p/the-ultimate-guide-to-claude-code) (2026) — "40 of 47 skills made the output worse"
- [conorbronsdon/avoid-ai-writing](https://github.com/conorbronsdon/avoid-ai-writing) — skill format reference

## License

MIT
