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
2. **Run `git diff` against the base branch** — see exactly what changed.
3. **Compare**: does the implementation match the plan?
4. **Flag deviations**:
   - Missing tasks — work specified in the plan that was not implemented.
   - Extra changes — modifications not called for in the plan (scope creep).
   - Spec misinterpretations — implemented something, but not what was actually asked for.
5. **Check**: are all tests passing? Run the test suite and confirm green.

If self-reflection finds gaps, fix them before moving to peer-reflection. Do not waste the reviewer's time on known issues.

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

**Act on feedback by severity:**

| Severity     | Action                                    |
|--------------|-------------------------------------------|
| **Critical** | Must fix immediately before proceeding.   |
| **Important**| Should fix before completing the task.    |
| **Suggestion**| Note for future improvement. Don't block.|

---

### 三省 — 终省 Zhong Sheng (Final-Reflection): User Value

Step back from the code entirely and ask:

> Does this actually solve the user's ORIGINAL problem?

Not "does the code work?" — but "does it deliver value?"

- Reread the user's initial request or issue description.
- Trace the path from their problem to your solution.
- If the connection is unclear or the value is uncertain, **ask the user directly**. Do not guess.

---

## Key Rule — Receiving Feedback

> **"You're absolutely right!" is the most dangerous response.**

When receiving review feedback:

1. **Verify** the suggestion is correct before implementing it. Reviewers can be wrong.
2. **Push back** with technical reasoning when the suggestion would make things worse.
3. **Never implement feedback blindly** — that violates 知行合一 (Zhi Xing He Yi, unity of knowledge and action). Understanding must precede action.

A good review is a dialogue, not a set of orders.

---

## Embedded Tools — Finish Branch

After review passes all three reflections, present exactly four options to the user:

1. **Merge locally**
   ```bash
   git checkout main && git merge feature-branch
   ```

2. **Create PR**
   ```bash
   gh pr create --title "..." --body "..."
   ```

3. **Keep as-is** — leave the branch for later work or manual review.

4. **Discard** — confirm with the user first, then:
   ```bash
   git branch -D feature-branch
   ```

### Worktree Cleanup

If the work was done inside a git worktree, clean it up after the user chooses a completion option:

```bash
cd /original/repo
git worktree remove ../worktree-dir
```

---

## Issue Severity Reference

| Level        | Meaning                        | Action       |
|--------------|--------------------------------|--------------|
| **Critical** | Must fix — blocks correctness  | Fix now      |
| **Important**| Should fix — affects quality   | Fix before done |
| **Minor**    | Nice to have — polish          | Note it      |

---

## Transition

- Review passed + user chooses a completion option → **done**.
- Critical issues found → **fix and re-review** (return to 自省).
