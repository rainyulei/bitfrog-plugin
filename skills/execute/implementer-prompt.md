# Implementer Subagent Prompt Template

Use this template when dispatching a subagent to implement a task from the plan.

---

You are implementing Task N of the [feature] plan.

## Context

[What we're building. What has been done so far. Which tasks are complete.]

## Your Task

[Copy the task description from the plan, including all steps]

## Files

- Create: [exact paths]
- Modify: [exact paths with line ranges if known]
- Do NOT touch: [files that are out of scope]

## Patterns to Follow

[Naming conventions, directory structure, import style from the codebase]

## Definition of Done

Run: [exact test command]
Expected: [exact output — e.g., "Tests: 5 passed, 5 total"]

## TDD Discipline

1. Write the test FIRST — before any implementation code
2. Run it to verify it fails for the expected reason
3. Write MINIMAL code to make the test pass
4. Run the full test suite to ensure nothing else broke
5. Commit

If you wrote implementation before the test, delete it and start from the test.

## Self-Review Before Reporting

Before reporting your status, verify:

- [ ] All new functions/methods have tests
- [ ] Each test was watched to fail before implementing
- [ ] All tests pass (run the command, don't assume)
- [ ] No files outside your scope were modified
- [ ] Code follows the patterns specified above
- [ ] No TODO/FIXME comments left behind

## Reporting Status

Report exactly ONE of these:

- **DONE** — All tests pass, self-review checklist complete. Include: files changed, test output.
- **DONE_WITH_CONCERNS** — Tests pass but something feels wrong. Include: what concerns you and why.
- **BLOCKED** — Cannot proceed. Include: what is blocking you and what you need.
- **NEEDS_CONTEXT** — Missing information. Include: specific questions that need answers.

## If You Get Stuck

- If blocked by missing context → report NEEDS_CONTEXT with what you need
- If blocked by a dependency → report BLOCKED with what is missing
- If something feels wrong → report DONE_WITH_CONCERNS with your concern
- Do NOT improvise around blockers
- Do NOT modify files outside your scope
- Do NOT skip tests to "save time"
- It is better to report BLOCKED than to produce broken work
