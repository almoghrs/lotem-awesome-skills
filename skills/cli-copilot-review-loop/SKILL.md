---
name: cli-copilot-review-loop
description: Uses the GitHub CLI to trigger Copilot reviews, parse GraphQL feedback, evaluate suggestions critically, apply fixes with local verification, and loop until approved (max 3 review iterations).
---

# CLI Copilot Review Loop

## Overview

This skill drives a pull request to an "APPROVED" state by requesting code reviews from GitHub Copilot and iterating on feedback. Treat Copilot's comments as professional code review input, validate, test, and document every decision. The loop exits when:
- Copilot approves the PR, **OR**
- You have requested review 3 times and there are no unresolved threads from Copilot, **OR**
- Human approval is required for a behavior/API change (escalate and pause).

## Core Principles

1. **Evaluate, don't blindly apply.** For each comment, ask: Why was the original code written this way? What exactly is Copilot proposing? Which choice is best?
2. **Answer every thread.** If you accept a suggestion, explain why and link tests/commits. If you decline, explain why in the thread itself.
3. **Verify locally before pushing.** Run tests, linting, and spot-check the change.
4. **Limit review iterations.** After 3 Copilot reviews, if no major issues remain, stop and merge or escalate.
5. **Require human approval for breaking changes.** Behavior/API changes must be reviewed by a human before merging.

## Workflow

### Step 1: Track Review Count

Before requesting a review, check how many times Copilot has already reviewed this PR. Use a case-insensitive match for "copilot" to handle various account names (e.g., `copilot-pull-request-reviewer`):
```bash
gh pr view --json reviews --jq '.reviews[] | select(.author.login | contains("copilot")) | .state' | wc -l
```

**SAFETY GATE**: If the count is **3 or greater**, you have reached the iteration limit. 
- **DO NOT** request another review.
- If threads remain unresolved, escalate to a human or provide a final summary and stop.
- If all threads are resolved, proceed to final merge if allowed.

### Step 2: Request Review

Ensure you are on the correct branch. Request a review from Copilot:
```bash
gh pr edit --add-reviewer @copilot
```

Wait for the review to complete. You can check status periodically:
```bash
gh pr view --json reviews,reviewRequests
```

Once Copilot has submitted a review (check `reviews`), proceed to Step 3.

### Step 3: Fetch and Parse Threads

Get the PR number:
```bash
gh pr view --json number --jq '.number'
```

Fetch unresolved review threads (replace `<PR_NUMBER>`):
```bash
gh api graphql \
  -F owner=":owner" \
  -F repo=":repo" \
  -F pr=<PR_NUMBER> \
  -f query='query($owner: String!, $repo: String!, $pr: Int!) {
    repository(owner: $owner, name: $repo) {
      pullRequest(number: $pr) {
        reviewThreads(first: 50) {
          nodes {
            id
            isResolved
            comments(first: 10) {
              nodes {
                author { login }
                path
                line
                body
              }
            }
          }
        }
      }
    }
  }'
```

Parse the JSON to extract:
- Threads where `isResolved == false` AND comment author contains `copilot`
- For each thread: save `id`, `path`, `line`, and `body`

### Step 4: Classify and Evaluate Each Thread

For each unresolved Copilot thread, classify it:

- **Bug / Correctness:** A logical error or missing edge case.
  - Try to reproduce locally with a minimal test or repro case.
  - If you can reproduce, apply the fix + add a test that captures the issue.
  - If you cannot reproduce, ask for a reproducible case in the thread.

- **Behavior / API Change:** Alters semantics, output, or public interface.
  - **Requires human approval.** Do NOT apply without explicit sign-off.
  - If you think the change is wrong, reply in the thread explaining why.
  - Escalate to a team lead or project owner before merging.

- **Performance / Resource:** Suggests optimization (memory, CPU, network).
  - Measure or provide a benchmark only if the improvement is material.
  - Accept only if risk is low and gain is meaningful.

- **Style / Formatting / Lint:** Code style, naming, or formatting.
  - Apply if it aligns with repo linting rules.
  - Prefer automated formatters (e.g., `eslint --fix`, `prettier`) over manual edits.

- **Tests / Test Maintenance:** Flaky tests, missing coverage, or test improvements.
  - Add or update tests as suggested.
  - Run the full test suite locally.

- **Documentation / Clarity:** Suggests improved comments or docstrings.
  - Apply if it materially improves readability or correctness understanding.

- **Question / Clarification:** Copilot asks for more context.
  - Reply in the thread with an explanation or context.
  - If you need clarification from Copilot, ask in the thread.

### Step 5: Implement Changes (Accepted Threads Only)

For each thread you decide to accept:

1. **Make the change** using local file-editing tools.
2. **Add or update tests** that verify the fix (if applicable).
3. **Run local verification:**
   - `npm test` (or your project's test command)
   - `npm run lint` or equivalent
   - Manual spot-check of the change

4. **Prepare a thread reply** (see templates below).

### Step 6: Commit and Push

```bash
git add <changed-files>
git commit -m "fix(review): address copilot feedback <thread-id> <reason>"
git push
```

Commit messages should reference the thread ID and a brief rationale. Example:
```
fix(review): add bounds check in parseConfig PRRT_kw123 prevent out-of-bounds access when input is empty
```

### Step 7: Answer All Threads

For **every** unresolved Copilot thread:

- **If you applied the fix:** Post a reply in the thread:
  ```
  Applied. Commit: <sha>. Tests added: <paths>. Resolving thread.
  ```

- **If you declined / modified:** Post a reply explaining your decision:
  ```
  I evaluated this suggestion. I am not applying it because <reason>. 
  If you intended <alternative interpretation>, please clarify.
  ```

- **If you need clarification:** Post in the thread:
  ```
  Can you provide a reproducible case or more context? Thanks.
  ```

### Step 8: Resolve Threads

For threads you have decided to **implement or decline**:

- **Implement:** Ensure the fix is pushed and verified. Then, resolve the thread after posting a reply with the rationale and tests.
- **Decline:** Provide a clear reply explaining why, then resolve the thread.

For threads awaiting clarification or further discussion:

- Leave the thread unresolved. You may return to it after receiving Copilot's response and deciding how to proceed.

Once ready to resolve a thread, use the following GraphQL command:
```bash
gh api graphql \
  -f query='mutation {
    resolveReviewThread(input: {threadId: "<THREAD_ID>"}) {
      thread {
        isResolved
      }
    }
  }'
```

### Step 9: Loop or Exit

1. **If Copilot review shows `APPROVED`:** Exit, the PR is approved.

2. **If you have already received 3 reviews:** You have reached the iteration limit. **Do NOT request another review**, even if the status is not `APPROVED`. Resolve remaining threads with rationale and then exit or merge.

3. **If Copilot review shows `CHANGES_REQUESTED` or `COMMENTED` and this is the 1st or 2nd review:** Return to Step 1, request another review, and iterate.

4. **If all threads have been resolved and Copilot has not yet approved (and you are on review 1 or 2):** Dismiss the review and re-request to see if the latest changes satisfy Copilot:
    ```bash
    gh pr dismiss-review $(gh pr view --json reviews --jq '.reviews[] | select(.author.login | contains("copilot")) | .author.login' | tail -1)
    gh pr edit --add-reviewer @copilot
    ```
    Then return to Step 2.

## Handling Behavior / API Changes

If Copilot suggests a change that alters public behavior or APIs:

1. **Do not apply** without explicit human approval.
2. **Reply in the thread** explaining the risk or your alternative interpretation.
3. **Escalate to project owner** or team lead with a link to the thread.
4. **Wait for sign-off** before pushing a behavior change.

## Decision Templates (Copy/Paste)

### Applied a Suggestion
```
Applied this suggestion. Commit: <sha>. Rationale: <1-2 sentences>. 
Tests added/updated: <paths>.
Resolving thread.
```

### Declined a Suggestion
```
I evaluated this suggestion and am not applying it. Reason: <1-2 sentences>.
If you intended <alternative interpretation>, please clarify.
```

### Need Clarification
```
Can you provide more context or a reproducible case? 
I want to ensure I'm addressing the right issue.
```

## Exit Criteria

- Copilot review is `APPROVED`
- All Copilot threads have been answered and resolved
- No unresolved behavior/API changes requiring human approval
- Local tests pass and linting is clean