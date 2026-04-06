# scope-guard

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that stops Claude from touching files you didn't ask it to touch.

You ask Claude to fix one bug. It refactors six files, renames three variables "for clarity," adds a helper utility you didn't need, and fixes two things you didn't know were broken. The diff is 400 lines. You asked for 12.

This skill fixes that.

## What it does

Before Claude edits anything, it produces a **Scope Declaration** — a structured list of exactly which files it will touch, what it will change in each one, and what it noticed but will leave alone. You confirm. Then it executes, stays inside the boundary, and logs anything it spotted outside scope as **Notices** (not fixes).

Three outcomes every time:
1. **Scope Declaration** — what Claude plans to do, before it does anything
2. **Execution within scope** — only the files that were declared
3. **Notices log** — problems spotted but not touched

## Why a skill, not just a prompt

A one-shot "don't touch other files" instruction works once. This skill is different:

- **Structured declaration format** — not a vague promise, a specific list of files with reasons
- **Blast radius rating** — S / M / L tells you immediately whether the change is contained
- **Mid-edit re-declaration** — if Claude discovers it needs an unlisted file, it stops and re-declares instead of silently expanding scope
- **Notices, not fixes** — problems outside scope get logged, not quietly corrected
- **Anti-pattern table** — 6 specific behaviors Claude is explicitly prohibited from doing ("clean up while I'm here," "rename for clarity," etc.)

## Installation

### Option 1: Clone into skills directory

```bash
git clone https://github.com/grxkun/scope-guard ~/.claude/skills/scope-guard
```

### Option 2: Copy the file directly

Download `SKILL.md` and place it in any directory Claude Code can read. Reference it in your `CLAUDE.md`:

```
- When fixing bugs or editing code → read `path/to/scope-guard/SKILL.md`
```

### Option 3: Use as a slash command

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

## What you get

Every run returns three sections:

**Scope Declaration** — presented before any edits, requires your confirmation

```
SCOPE DECLARATION
─────────────────
Task:        Fix the login button not responding on mobile
Files I will touch:
  - src/components/LoginButton.tsx → update onClick handler to use touchstart
Files I will NOT touch (even if I see problems):
  - src/components/AuthForm.tsx → noticed deprecated prop usage, leaving alone
Blast radius: S
─────────────────
Proceed? (yes / adjust scope)
```

**Execution** — only what was declared

**Notices log** — what was spotted outside scope, not fixed

```
NOTICES (not fixed — outside scope)
─────────────────────────────────────
• src/components/AuthForm.tsx:47 — deprecated `onSubmit` prop, should be `onFormSubmit`
• src/utils/auth.ts:12 — console.log left in production path
─────────────────────────────────────
Run /scope-guard to address any of these.
```

## Full Example

**Request:** "The submit button on the checkout form is throwing a console error, can you fix it"

---

**Without scope-guard:**

Claude fixes the console error, then notices the form validation logic is "a bit messy," refactors it into a custom hook, moves the hook to a new `hooks/` directory it creates, updates three other forms that use similar logic, renames `handleSubmit` to `onFormSubmit` across all of them, and adds a `useFormState` utility "that'll be useful later."

You asked for a console error fix. You got a 340-line diff across 8 files.

---

**With scope-guard:**

```
SCOPE DECLARATION
─────────────────
Task:        Fix console error thrown by submit button in checkout form
Files I will touch:
  - src/components/CheckoutForm.tsx → fix undefined reference on line 84
Files I will NOT touch (even if I see problems):
  - src/components/ShippingForm.tsx — noticed similar pattern, leaving alone
  - src/hooks/ — no new files
Blast radius: S
─────────────────
Proceed? (yes / adjust scope)
```

You say yes. Claude fixes line 84. The diff is 3 lines.

```
NOTICES (not fixed — outside scope)
─────────────────────────────────────
• src/components/ShippingForm.tsx:61 — same undefined reference pattern, likely same bug
─────────────────────────────────────
Run /scope-guard to address any of these.
```

You now know about the ShippingForm bug. You decide when to fix it.

---

**What scope-guard caught:** unprompted refactor, new directory creation, cross-file renames, speculative utility creation, silent scope expansion.

## Blast Radius Reference

| Rating | Meaning |
|---|---|
| **S** | 1-2 files, no shared logic. Diff readable in 30 seconds. |
| **M** | 3-5 files, or touches shared utilities. Claude names every consumer. |
| **L** | 6+ files, or touches core abstractions. Requires explicit confirmation. Claude proposes S/M alternative first. |

## Anti-pattern table

| What Claude wants to do | What scope-guard makes it do instead |
|---|---|
| "Clean up the formatting while I'm here" | Log it in Notices |
| "Rename this variable to something cleaner" | Only if inside the specific function being changed |
| "Add a helper utility that would make this cleaner" | Don't. Do the task directly. |
| "Fix this related bug I noticed" | Log it in Notices |
| "Refactor this to be more readable" | Only if explicitly asked |
| "Update the tests to match" | Yes — but declare the test files in Scope Declaration |

## License

MIT
