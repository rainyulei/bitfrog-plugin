---
name: review
description: "三省吾身 San Sheng Wu Shen — Three levels of reflection: self (spec compliance), peer (code-reviewer subagent), final (user value). Includes branch completion and cleanup."
---

# 三省吾身 San Sheng Wu Shen — Three Levels of Reflection

> 曾子曰：「吾日三省吾身 — 为人谋而不忠乎？与朋友交而不信乎？传不习乎？」
>
> Zengzi said: "I examine myself three times daily — Have I been loyal in counseling others? Have I been trustworthy with friends? Have I practiced what was taught?"

Self-reflection reveals blind spots. Peer-reflection provides independent perspective. Final-reflection confirms user value. Code that passes all three is code worth shipping.

---

## Three Reflections Workflow

### 一省 — 自省 Zi Sheng (Self-Reflection): Spec Compliance

Before asking anyone else, examine your own work first.

1. **Load the plan/spec document** — reread the original requirements, acceptance criteria, and task list.
2. **Determine the review range** — get the exact SHAs:
   ```bash
   BASE_SHA=$(git merge-base HEAD main 2>/dev/null || git merge-base HEAD master)
   HEAD_SHA=$(git rev-parse HEAD)
   git diff ${BASE_SHA}..${HEAD_SHA}
   ```
3. **Scope Drift Detection — 偏则正之 (When drifting, correct course)**:
   Compare `git diff --stat` against the stated intent (plan/spec/commit messages):

   ```
   Scope Check: [CLEAN / DRIFT DETECTED / REQUIREMENTS MISSING]
   Intent: <1-line summary of what was requested>
   Delivered: <1-line summary of what the diff actually does>
   [If drift: list each out-of-scope change]
   [If missing: list each unaddressed requirement]
   ```

   Watch for:
   - **Scope creep** — files changed that are unrelated to the stated intent, "while I was in there" changes
   - **Missing requirements** — tasks from the plan not addressed in the diff
   - **Partial implementations** — started but not finished

4. **Flag deviations**:
   - Missing tasks — work specified in the plan that was not implemented.
   - Extra changes — modifications not called for in the plan (scope creep).
   - Spec misinterpretations — implemented something, but not what was actually asked for.
5. **Check**: are all tests passing? Run the test suite and confirm green.

If self-reflection finds gaps, fix them before moving to peer-reflection. Do not waste the reviewer's time on known issues.

#### Plan Review — 格物致知 applied to plans

格物 does not end at the design phase. Plans themselves need investigation:
- Does every task have a clear, testable outcome?
- Are dependencies correctly identified?
- Are there gaps — things the plan assumes but does not state?
- Could any "sequential" tasks actually run in parallel, or vice versa?

If the plan has issues, fix them before executing. A flawed plan produces flawed code — no amount of TDD discipline compensates for building the wrong thing.

---

### 二省 — 互省 Hu Sheng (Peer-Reflection): Code Quality

Dispatch the `code-reviewer` subagent with the following inputs:

- **Git diff output** (or commit range SHAs covering all changes)
- **Path to plan/spec document**
- **Brief description** of what was implemented and why

The subagent reviews for:

| Dimension       | What to look for                                      |
|-----------------|-------------------------------------------------------|
| Readability     | Clear naming, logical structure, no unnecessary complexity |
| Abstraction     | Right level — not too clever, not too verbose          |
| Debuggability   | Meaningful errors, logging, traceable control flow     |
| Error handling  | Edge cases covered, failures handled gracefully        |

**Fix-First Review — 行胜于言 (Action speaks louder than words):**

Don't just list problems — fix them. For each finding from the reviewer:

1. **AUTO-FIX** — If the fix is mechanical and unambiguous (typo, missing import, obvious null check, style issue): fix it directly. Report: `[AUTO-FIXED] file:line — Problem → What you did`

2. **ASK** — If the fix requires judgment (architecture choice, tradeoff decision, unclear intent): present to the user with options and your recommendation.

Batch all ASK items into a single question to minimize interruption:

```
Auto-fixed 4 issues. 2 need your input:

1. [Important] src/auth.ts:42 — Race condition in token refresh
   Fix: Add mutex lock around refresh call
   → A) Fix as recommended  B) Skip

2. [Suggestion] src/api.ts:88 — Error message exposes internal path
   Fix: Replace with generic error
   → A) Fix  B) Skip

RECOMMENDATION: Fix #1 (real concurrency bug). #2 is low-risk, your call.
```

**Severity guide for Fix-First routing:**

| Severity     | Default Action |
|--------------|----------------|
| **Critical** | ASK — always get user confirmation for critical fixes |
| **Important**| AUTO-FIX if mechanical, ASK if judgmental |
| **Suggestion**| AUTO-FIX if trivial, otherwise note and move on |

#### Reviewer Dispatch Prompt Template

When dispatching the code-reviewer subagent, use the template in `skills/review/reviewer-prompt.md`. It includes: the 三省 review framework, output format, severity definitions, verdict options, and calibration notes.

Read the template file and fill in the placeholders (description, spec path, git diff range).

---

### 三省 — 终省 Zhong Sheng (Final-Reflection): User Value

Step back from the code entirely and ask:

> Does this actually solve the user's ORIGINAL problem?

Not "does the code work?" — but "does it deliver value?"

- Reread the user's initial request or issue description.
- Trace the path from their problem to your solution.
- If the connection is unclear or the value is uncertain, **ask the user directly**. Do not guess.

---

## Receiving Feedback — 三省的深层功夫

> 三省 is not just about giving reflection — it is equally about RECEIVING it.

### The Danger of Agreement

"You're absolutely right!" is the most dangerous response in a code review. Not because the reviewer is wrong, but because agreement terminates thinking. The moment you say "great point!" your mind stops examining whether the point is actually great.

**What genuine reception looks like (知行合一 applied to feedback):**

1. **Read the feedback completely** before forming a response. Not while scrolling — after.
2. **Verify against the codebase.** The reviewer may be wrong. They may be reading old code. They may misunderstand the context. Check.
3. **If the suggestion is correct,** implement it with understanding, not compliance. Ask yourself: WHY is this better? What does this teach me?
4. **If the suggestion is wrong,** push back with technical reasoning. Showing why, not just asserting disagreement.
5. **If the suggestion is unclear,** ask for clarification FIRST. Do not guess what the reviewer meant and implement your guess.

### The YAGNI Test for Review Suggestions

Reviewers sometimes suggest "professional" improvements that add complexity without solving a real problem:
- "You should add a factory pattern here" — Why? Is there more than one implementation?
- "This should be configurable" — Who will configure it? When?
- "Add error handling for X" — Can X actually happen in this context?

格物致知 applies to review suggestions too: investigate whether the suggestion addresses a real problem before implementing it.

### The Trap of Performative Agreement

These responses indicate you have stopped thinking:
- "Great catch!" (before verifying it IS a catch)
- "Absolutely, will fix!" (before understanding what to fix)
- "Good point, I missed that!" (before checking if you actually missed it)

Replace them with genuine engagement:
- "Let me check if that's the case..." (then actually check)
- "I see what you mean. The tradeoff is..." (then explain the tradeoff)
- "Can you clarify what you mean by X?" (when genuinely unclear)

### When to Push Back

Push back is not defiance — it is 知行合一. If you implement something you know is wrong, you have violated the unity of knowledge and action.

Push back when:
- The suggestion would break existing functionality
- The reviewer lacks context about a deliberate design choice
- The suggestion violates YAGNI (adds unused complexity)
- The suggestion is technically incorrect (verify first!)
- The suggestion conflicts with the spec/plan

How to push back well:
- State what you understand their concern to be
- Explain why you disagree, with evidence (code references, test results)
- Offer an alternative if you have one
- Accept gracefully if they convince you — being wrong about pushback is fine

### Implementation Order for Review Fixes

When multiple issues are found:
1. **Critical issues first** — these block correctness
2. **Simple fixes next** — quick wins, verify each individually
3. **Complex fixes last** — these need thought, test thoroughly
4. Run the full test suite after ALL fixes, not just after each one

---

## Embedded Tools — Finish Branch

### Pre-Completion Gate

Before presenting completion options, verify:

```bash
# Run the full test suite
npm test   # or cargo test, pytest, go test ./..., etc.

# Run linter/type checker
npm run lint   # or equivalent
```

**If tests fail, STOP.** Do not present completion options. Fix the failures first and re-run self-reflection.

### Determine Base Branch

```bash
BASE_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@' || echo "main")
```

### Completion Options

Present exactly four options to the user:

| Option | What happens | When to recommend |
|--------|-------------|-------------------|
| **1. Merge locally** | `git checkout $BASE_BRANCH && git merge feature-branch` | Small changes, single developer |
| **2. Create PR** | `gh pr create --title "..." --body "..."` | Team projects, needs review |
| **3. Keep as-is** | Leave the branch for later | Work in progress, needs more thought |
| **4. Discard** | Delete the branch entirely | Experiment that didn't work out |

**For option 4 (Discard):** Require explicit confirmation. Ask the user to type "discard" to confirm. Accidental deletion of work is irreversible.

```bash
# Only after user types "discard"
git checkout $BASE_BRANCH
git branch -D feature-branch
```

### Worktree Cleanup

If the work was done inside a git worktree, clean it up after the user chooses a completion option:

```bash
cd /original/repo
git worktree remove ../worktree-dir
# Verify cleanup
git worktree list
```

---

## Issue Severity Reference

| Level        | Meaning                        | Action       |
|--------------|--------------------------------|--------------|
| **Critical** | Must fix — blocks correctness  | Fix now      |
| **Important**| Should fix — affects quality   | Fix before done |
| **Suggestion**| Nice to have — polish          | Note it      |

---

## Transition

- Review passed + user chooses a completion option → **done**.
- Critical issues found → **fix and re-review** (return to 自省).
