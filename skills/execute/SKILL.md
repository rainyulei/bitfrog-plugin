---
name: execute
description: "知行合一 Zhi Xing He Yi — Execute plans with TDD discipline, parallel agents, and verification. Knowledge without action is not true knowledge."
---

# Execute — 知行合一 Zhi Xing He Yi

> Unity of Knowledge and Action

"True knowledge manifests as action. If you know you should write tests but skip them, you don't truly know why tests matter."

## Philosophy — 知行合一 (Zhi Xing He Yi)

知 (zhi, knowledge) and 行 (xing, action) are not separate stages — they are one movement. Understanding a design pattern means nothing if you cannot implement it correctly. Writing code means nothing if you cannot verify it works.

Two principles govern this skill:

1. **Knowledge without action is not true knowledge (知而不行非真知).** A plan that stays a plan is not a plan — it is a wish. Reading a task and nodding is not understanding. You understand the task only when your implementation passes its tests.

2. **Action without verification is not true completion (行而不验非真成).** Code that "should work" does not work until proven. A task is not done when the code is written. It is done when the tests pass, the linter is clean, and the full suite is green.

## Workflow

### 1. Load and Critically Review the Plan

Read the plan document. Do not blindly follow it — critically assess:

- Does each task have a clear, testable outcome?
- Are dependencies between tasks correctly ordered?
- Are there gaps — things the plan assumes but does not state?
- Are there tasks marked as parallelizable that actually share files or state?

If the plan has problems, invoke `bitfrog-plugin:plan` to revise it before proceeding.

### 2. Execute Tasks — TDD Cycle (红绿重构)

For each task, follow the cycle strictly:

```
1. 读 (du, read)       — Read the task from the plan. Understand the requirement.
2. 红 (hong, red)      — Write the test FIRST. The test MUST fail.
3. 验红 (yan hong)     — Run the test. Confirm it fails with the EXPECTED error.
                         If it passes, your test is wrong. If it fails unexpectedly,
                         your understanding is wrong. Fix before proceeding.
4. 绿 (lv, green)      — Write the MINIMAL implementation to make the test pass.
                         No more, no less.
5. 验绿 (yan lv)       — Run the test. Confirm it passes.
6. 理 (li, refactor)   — Refactor if needed. Remove duplication, improve naming,
                         simplify logic. Tests must still pass after refactoring.
7. 省 (xing, reflect)  — Run the FULL test suite. Ensure nothing is broken.
8. 记 (ji, commit)     — Commit the working, verified change.
```

### Why TDD — The Philosophical Root

TDD is not a ritual. It is 知行合一 made concrete:

- **Writing the test first** means you define what "correct" means BEFORE you can be biased by your implementation. This is 格物 (investigation) applied to code — understanding the requirement before acting on it.
- **Watching it fail (红/red)** proves your test actually tests something. A test that passes immediately is not a test — it is a false comfort.
- **Writing minimal code (绿/green)** is 中庸之道 — the right measure. Not too much, not too little. Only what is needed.
- **Refactoring (理)** is reflection in action — improving what works without breaking it.

If you find yourself wanting to write code first and test later, examine your understanding: do you truly know what you're building, or are you hoping to discover it by writing code? If the latter, the test IS your discovery tool — write it first.

### TDD by Example

**Good — test defines behavior before implementation:**

```typescript
// RED: Write the test first
test('rejects empty email on form submit', async () => {
  const result = await submitForm({ email: '' });
  expect(result.error).toBe('Email required');
});

// Verify RED: run test → FAIL: "submitForm is not defined" ✓

// GREEN: Minimal implementation
function submitForm(data: FormData) {
  if (!data.email?.trim()) {
    return { error: 'Email required' };
  }
  return { success: true };
}

// Verify GREEN: run test → PASS ✓
```

**Bad — implementation first, test retrofitted:**

```typescript
// ✗ Wrote implementation first
function submitForm(data: FormData) {
  if (!data.email?.trim()) return { error: 'Email required' };
  if (!data.email.includes('@')) return { error: 'Invalid email' };
  // ... 50 lines of validation ...
  return { success: true };
}

// ✗ Test written after — just confirms what already exists
test('validates email', () => {
  expect(submitForm({ email: '' }).error).toBe('Email required');
  // This test was shaped by the code, not the requirement
});
```

The first approach discovers bugs because the test is independent of the code. The second just photographs existing behavior — including its bugs.

### When Stuck with TDD

| Problem | What it tells you | Solution |
|---------|-------------------|----------|
| Don't know how to test it | You don't know what it should do | Write the assertion first — the API you wish existed |
| Test too complicated | The design is too complicated | Simplify the interface, then the test follows |
| Must mock everything | Code is too coupled | Use dependency injection, reduce coupling |
| Test setup is huge | Too many dependencies | Extract helpers, or simplify the design |

When adding mocks or test utilities, read `skills/execute/testing-anti-patterns.md` to avoid common traps: testing mock behavior instead of real behavior, test-only production methods, mocking without understanding, testing implementation instead of behavior, and incomplete mocks hiding real bugs.

### Self-Check Questions — 反求诸己

When tempted to skip TDD, ask yourself:

- Am I writing code first because I understand the requirement better that way — or because I'm impatient?
- If I wrote the code first, can I honestly delete it and write from tests? If not — why not?
- Is the test I'm about to write testing behavior (what it does) or implementation (how it does it)?
- Would a different developer write the same test from the same requirement?

### 3. Report Progress

After every 3 completed tasks, report:

- Tasks completed (with pass/fail status)
- Tasks remaining
- Any issues encountered or risks identified
- Updated time estimate if trajectory has changed

### 4. Track Progress in the Plan Document

Update the plan document with checkbox tracking as tasks complete:

```markdown
- [x] Task 1: Implement user model ✓ (tests pass)
- [x] Task 2: Add validation logic ✓ (tests pass)
- [ ] Task 3: Create API endpoint (in progress)
- [ ] Task 4: Add error handling
```

## Embedded Tools — Parallel Subagent Dispatch (阴阳互生)

阴阳互生 (yin yang hu sheng) — Yin and Yang arise together. Independent tasks can proceed in parallel, each complementing the other, as long as they do not interfere.

### When to Parallelize

Only when the plan explicitly marks tasks as parallelizable AND all of these are true:

- **No shared files** — agents writing to the same file will conflict
- **No sequential dependency** — task B does not need task A's output
- **No shared mutable state** — no overlapping database tables, config entries, or global state

### How to Dispatch

1. **One Agent per independent task.** Each agent receives a focused prompt containing:
   - **Specific scope** — exactly which files to create or modify
   - **Clear goal** — the testable outcome for this task
   - **Constraints** — what NOT to touch, style rules, naming conventions
   - **Expected output** — what success looks like (test command + expected result)

2. **Wait for all agents to return.**

3. **Review results for conflicts:**
   - File conflicts (two agents modified the same file)
   - Interface mismatches (agent A expects a function signature agent B didn't provide)
   - Test failures in the combined suite

4. **If conflicts are found, resolve them sequentially.** Do not re-parallelize conflict resolution.

### Subagent Prompt — 阴阳互生的责任 (The Responsibility of Delegation)

When you delegate to another agent, 阴阳互生 demands you give them what they need to succeed. A vague delegation is not collaboration — it is abandonment.

**Every subagent prompt must include:**

1. **Context** — What are we building? What has been done so far?
2. **Specific scope** — Exactly which files to create/modify, and which to leave alone
3. **Expected outcome** — The test command and expected output that proves success
4. **Constraints** — Style rules, naming conventions, patterns to follow
5. **Escalation path** — What to do if blocked (report status, don't improvise)

**Subagent status protocol:**

When a subagent returns, it should report one of:
- **DONE** — All tests pass, work is complete
- **DONE_WITH_CONCERNS** — Tests pass but something feels wrong. Explain what.
- **BLOCKED** — Cannot proceed. Explain why and what is needed.
- **NEEDS_CONTEXT** — Missing information required to continue.

**Do not trust "DONE" blindly.** 三省吾身 (three reflections) applies to agent reports too: verify their changes against the plan, run the test suite yourself, then accept.

### Implementer Prompt Template

When dispatching a subagent, use the template in `skills/execute/implementer-prompt.md`. It includes: context, task scope, file boundaries, patterns to follow, definition of done, TDD discipline, self-review checklist, and status reporting protocol.

Read the template file and fill in the placeholders for each dispatch.

### Handling Subagent Returns

| Status | Action |
|--------|--------|
| **DONE** | Verify: read the diff, run the test suite yourself. If tests pass → accept. If not → fix or re-dispatch. |
| **DONE_WITH_CONCERNS** | Read the concerns. If valid → address before moving on. If not → note and proceed. |
| **BLOCKED** | Assess the blocker. If it's a plan gap → invoke `bitfrog-plugin:plan`. If it's a dependency → resolve sequentially, then re-dispatch. |
| **NEEDS_CONTEXT** | Provide the missing context and re-dispatch. If you don't have it either → ask the user. |

### For Larger Projects — Per-Task Review

For projects with 10+ tasks, consider reviewing after every 3-5 tasks rather than waiting until the end. 中庸之道 — the right review frequency depends on the project size. Small projects: review at the end. Large projects: review in batches. The measure is: can you still hold the full context of unreviewed changes in mind?

## Embedded Tools — Verification Before Completion (知行合一)

知行合一 teaches: if you truly know something, that knowledge is inseparable from action. Applied to verification:

**What it means to truly understand "done":**

- You understand that "done" is a statement about reality, not about your intentions
- So you verify because you seek truth, not because a rule demands it
- "I believe it works" is not knowledge — it is hope. Hope is not 知.
- "Should be fine" / "Probably passes" / "Looks correct" — these are the language of guessing, not knowing

**What true verification looks like:**

1. **Identify** the verification command (test suite, type checker, linter, build)
2. **Run** it NOW — not from memory, not from a previous run
3. **Read** every line of output
4. **Confirm** the output matches your claim: "All tests passed", "0 errors", exit code 0

If verification fails, you don't know it works. You only know it doesn't.

### Verification Evidence Table

| Claim | Requires | NOT Sufficient |
|-------|----------|----------------|
| "Tests pass" | Fresh test output showing 0 failures | Previous run, "should pass", partial run |
| "Linter clean" | Linter output: 0 errors, 0 warnings | "Linter passed last time" |
| "Build succeeds" | Build command: exit code 0 | Linter passing (linter ≠ compiler) |
| "Bug fixed" | Test for original symptom passes | "Code changed, should be fixed" |
| "Regression test works" | Red-green verified (test fails without fix, passes with fix) | Test passes once |
| "Agent completed task" | You verified the diff + ran tests yourself | Agent reported "DONE" |
| "All requirements met" | Line-by-line checklist against spec | "Tests pass" (tests ≠ requirements) |

### Self-Check — 反求诸己

If you catch yourself thinking any of these, pause:

- "I'm confident this works" — Confidence is not evidence. Run the command.
- "I just need to commit and move on" — Urgency is not a reason to skip truth.
- "The agent said it's done" — The agent might be wrong. Verify independently.
- "This is a trivial change" — Trivial changes break production too. Verify.

**The deeper question:** If you find yourself wanting to skip verification, ask: do I truly understand why I verify? If the answer is "because the rules say so" — you have not yet internalized 知行合一. You verify because claiming without evidence is self-deception, and self-deception produces broken software.

## Problem Escalation — 辩证论治 (Bian Zheng Lun Zhi)

辩证论治 — Diagnose through dialectical analysis, treat according to the diagnosis. Different problems require different remedies. Forcing the wrong remedy wastes effort.

### Escalation Ladder

| Situation | Action | Max Retries |
|---|---|---|
| Clear test failure with obvious fix | Fix directly | 3 |
| Unclear or repeated failure (after 3 direct fix attempts) | Invoke `bitfrog-plugin:debug` | 3 |
| Plan is wrong, incomplete, or missing steps | Invoke `bitfrog-plugin:plan` | — |
| Fundamental design flaw discovered during implementation | Invoke `bitfrog-plugin:brainstorm` | — |

### Rules

- **Max 3 retries at any level before escalating.** If you have tried to fix a test failure 3 times and it still fails, do not try a 4th time. Escalate to `bitfrog-plugin:debug`.
- **If `bitfrog-plugin:debug` fails after 3 attempts,** the problem is likely in the plan or design. Escalate to `bitfrog-plugin:plan` or `bitfrog-plugin:brainstorm`.
- **Never hide failures.** Report what failed, what you tried, and why you are escalating.

## Transition

When ALL of the following are true:

- Every task in the plan is marked `[x]`
- The full test suite passes (verified by fresh output)
- The linter/type checker is clean (verified by fresh output)
- No known issues are deferred or hidden

Then — and only then — automatically invoke `bitfrog-plugin:review`.
