---
name: cli-copilot-review-loop
description: >
  Runs an iterative, agent-driven code review loop entirely in the terminal.
  Use when you want to repeatedly review, receive structured feedback, apply
  fixes, and re-review until the code meets quality criteria — all without
  leaving the CLI. Triggers on phrases like "review loop", "iterative review",
  "copilot review cycle", or "keep reviewing until it passes".
license: MIT
---

# CLI Copilot Review Loop

This skill guides an AI copilot through a structured, repeating review cycle:
**review → feedback → fix → re-review**, continuing until a stop condition is met
(all issues resolved, a pass threshold is reached, or a maximum iteration count).

## When to Use

- You want automated, multi-pass code review without manual intervention
- You need the agent to apply its own review feedback and verify the fix
- You are working entirely in the terminal and want an end-to-end loop
- You want a traceable record of what changed between review passes

## Loop Overview

```
┌──────────────────────────────────────────┐
│  1. Review target files / diff            │
│  2. Emit structured feedback (issues list)│
│  3. Apply fixes for each issue            │
│  4. Re-run review on changed files        │
│  5. Repeat until stop condition met       │
└──────────────────────────────────────────┘
```

## Steps

<!-- TODO: flesh out each step with concrete agent instructions -->

1. **Initialize** — Identify the target scope: a file glob, a git diff, or an
   explicit list of paths. Confirm the stop condition (e.g. zero issues, ≤N
   warnings, or max M iterations).

2. **Review pass** — Read each target file and emit a structured issue list:
   ```
   [ISSUE] <severity> | <file>:<line> | <category> | <description>
   ```
   Severities: `ERROR`, `WARNING`, `INFO`.

3. **Triage** — Filter issues by severity. Stop the loop if the stop condition
   is already satisfied. Otherwise proceed to the fix pass.

4. **Fix pass** — For each `ERROR` (and optionally `WARNING`), apply the
   minimal change that resolves the issue. Do not refactor beyond the issue scope.

5. **Verify** — Re-read the changed sections and confirm the fix resolves the
   original issue without introducing new ones.

6. **Loop** — Return to step 2 with the updated files. Increment the iteration
   counter. If the maximum iteration count is reached, halt and report remaining
   open issues.

7. **Report** — Emit a final summary:
   - Total iterations run
   - Issues found / fixed / remaining
   - Files changed

## Stop Conditions

| Condition          | Description                                      |
|--------------------|--------------------------------------------------|
| `all-clear`        | Zero issues remain (default)                     |
| `no-errors`        | Zero `ERROR`-severity issues remain              |
| `max-iterations:N` | Stop after N loops regardless of issue count     |

## Configuration (optional frontmatter in calling context)

```yaml
# Example: override defaults when invoking this skill
stop-condition: no-errors
max-iterations: 5
fix-severities: [ERROR, WARNING]
```

## References

<!-- TODO: add references/issue-categories.md with a taxonomy of review categories -->
<!-- TODO: add scripts/review.py for deterministic static-analysis integration -->
