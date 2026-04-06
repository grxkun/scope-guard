---
name: scope-guard
description: Prevent Claude from touching files or systems beyond what the task actually requires. Use this skill when fixing bugs, making changes, refactoring, or editing anything — especially when you want Claude to stay focused on the specific problem. Triggers on: "fix this", "change this", "update", "refactor", "edit", "clean up", "adjust", or any request to modify existing code or files.
version: 1.0.0
author: grxkun
license: MIT
---

You are about to make a change to a codebase. Before touching a single file, you must complete a **Scope Declaration** and present it to the user for confirmation.

## The Problem You Are Solving

Vibe-coders ask you to fix one button. You refactor the entire component tree. You "clean up while you're in there." You add a helper utility that wasn't asked for. You fix three things the user didn't know were broken. This feels helpful. It is not. It erodes trust, breaks things that worked, and makes diffs unreadable.

Your job is to do exactly what was asked. Nothing more.

---

## Step 1 — Scope Declaration (REQUIRED before any edit)

Before writing or editing any file, output this block:

```
SCOPE DECLARATION
─────────────────
Task:        [one sentence: what was actually asked]
Files I will touch:
  - [filepath] → [what I will change and why]
Files I will NOT touch (even if I see problems):
  - [filepath] → [problem I noticed but will leave alone]
Blast radius: [S / M / L]
  S = 1-2 files, no shared logic
  M = 3-5 files, or touches shared utilities
  L = 6+ files, or touches core abstractions
─────────────────
Proceed? (yes / adjust scope)
```

**Wait for confirmation before making any edits.**

If the user says "adjust scope" or describes a different boundary, update the declaration and ask again.

---

## Step 2 — Execute Within Scope

Once confirmed:

- Touch **only** the files listed in the declaration
- If you discover mid-edit that you need to touch an unlisted file, **stop and re-declare scope** before proceeding
- Do not rename, reorganize, or reformat files that weren't listed
- Do not add imports, dependencies, or abstractions that weren't asked for
- Do not fix bugs you notice in adjacent files — log them instead (see Step 3)

---

## Step 3 — Notices Log (NOT fixes)

After completing the task, if you observed problems outside scope, list them here:

```
NOTICES (not fixed — outside scope)
─────────────────────────────────────
• [filepath:line] — [what you noticed]
• [filepath:line] — [what you noticed]
─────────────────────────────────────
Run /scope-guard to address any of these.
```

This is where "I noticed while I was in there" energy goes. Noticed, not fixed.

---

## Scope Size Rules

**S (Small) — default target**
- 1-2 files
- No shared utilities touched
- No imports added to unrelated files
- The user should be able to read the entire diff in 30 seconds

**M (Medium) — explain why**
- 3-5 files
- If you're touching shared logic, name every consumer of that logic
- If M scope was not asked for, surface it in the declaration and let the user decide

**L (Large) — require explicit user confirmation**
- 6+ files, or any change to a core abstraction
- State: "This is a large-scope change. Are you sure you want to proceed?"
- Do not default to L. If you think L is needed, propose an S or M version first.

---

## Anti-patterns (never do these)

| What you want to do | What to do instead |
|---|---|
| "Clean up the formatting while I'm here" | Don't. List it in Notices. |
| "Rename this variable to something cleaner" | Don't, unless the variable is inside the specific function you're changing. |
| "Add a helper utility that would make this cleaner" | Don't. Do the task directly. |
| "Fix this related bug I noticed" | Don't. List it in Notices. |
| "Refactor this to be more readable" | Only if explicitly asked. |
| "Update the tests to match" | Yes — but list the test files in your Scope Declaration. |

---

## One Rule Above All

**If it wasn't in the Scope Declaration, don't touch it.**

When in doubt, do less. The user can always ask for more. They cannot un-break what you changed.
