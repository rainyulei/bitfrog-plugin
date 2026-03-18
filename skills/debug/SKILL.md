---
name: debug
description: "辩证论治 Bian Zheng Lun Zhi — Diagnose before prescribing using the Four Diagnostic Methods (望闻问切). Classifies issues by depth and applies appropriate treatment."
---

# 辩证论治 Bian Zheng Lun Zhi — Diagnose Before Prescribing

The same symptom can have different root causes. Never apply a fixed remedy without first diagnosing the true nature and level of the problem.

This debug skill is inspired by the Four Diagnostic Methods (四诊法) of Traditional Chinese Medicine. Just as a physician must observe, listen, inquire, and examine before writing a prescription, a debugger must understand before fixing.

---

## 四诊法 Si Zhen Fa — The Four Diagnostic Methods

### 望 Wang (Observe)

Look before you touch. Gather visual evidence first.

- **Read the full error message** — every word matters. Do not skim.
- **Determine blast radius**: Is it one test? One module? The entire system?
- **Check frequency**: Does it happen always? Intermittently? Only under load?
- **Note timing**: When did it start? What changed recently? Was there a deploy, a merge, a dependency update?

### 闻 Wen (Listen)

Let the system speak. Hear what it is telling you beyond the surface error.

- **Check the runtime environment** — versions, configuration, environment variables. A mismatch here is a silent killer.
- **Read relevant logs** — not just the error line, but the context around it. What happened before and after?
- **Review recent changes** — deployments, merges, dependency updates. The bug may not be in your code.
- **Listen to patterns** — if the system has been showing warnings or degraded behavior, those are clues.

### 问 Wen (Inquire)

Ask the right questions. Drill down relentlessly.

- **Apply the 5-Why technique**: Ask "why" iteratively until you reach the root cause.
  - Example:
    - "Test fails" → Why?
    - "Null returned" → Why?
    - "DB query returned empty" → Why?
    - "Migration not run" → Why?
    - "Deploy script skipped migrations" → **Root cause found.**
- **Trace the call path** from symptom to source. Follow the data, not your assumptions.
- **Form hypotheses** ranked by likelihood. The most common cause is usually the most likely.

### 切 Qie (Examine)

Now — and only now — put your hands on the code.

- **Deep-dive with targeted reading** — read the specific code paths identified by your hypotheses.
- **Add strategic logging or breakpoints** to verify, not to explore blindly.
- **Run minimal reproduction cases** — strip away everything that is not essential to triggering the bug.
- **Test one hypothesis at a time** — never shotgun debug. If you change three things and it works, you do not know which one fixed it.

---

## 三证 San Zheng — Issue Classification

After diagnosis, classify the issue by depth. The classification determines the treatment.

### 表证 Biao Zheng (Surface Issue)

A local bug with a clear cause and contained impact. The code is wrong in one place.

**Treatment:**
- Fix directly at the source.
- Write a regression test that would have caught this.
- Commit and move on.

### 里证 Li Zheng (Internal Issue)

A systemic issue where multiple symptoms stem from one root cause. Fixing individual symptoms will not resolve it.

**Treatment:**
- Identify and fix the root cause, not the symptoms.
- Verify that all related symptoms resolve after the fix.
- If symptoms remain, reclassify — the diagnosis may be incomplete.

### 深证 Shen Zheng (Deep Issue)

An architectural issue where the design itself is flawed. No amount of patching will make it right.

**Treatment:**
- Do NOT patch. Patching a flawed architecture creates compounding debt.
- Invoke `bitfrog:brainstorm` to redesign the affected area.
- The fix lives at a different level than the symptom.

---

## 追根溯源 — Root Cause Tracing (Backward Tracing)

When the 5-Why technique points to a code path, trace it backward through the system:

### Technique

1. **Start at the symptom** — the exact line where the error manifests
2. **Trace backward** through the call chain: who called this function? With what arguments? From where?
3. **At each level, verify your assumption** — add a log or read the code to confirm the data flow matches your mental model
4. **Stop when you find the divergence** — the point where reality differs from expectation. That is the root cause.

### Example: Multi-Component Tracing

```
Symptom: API returns 500 on /users/123
  ↑ Handler: getUserById throws "Cannot read property 'name' of null"
    ↑ Service: findUser returns null
      ↑ Repository: SQL query returns empty result
        ↑ Migration: 'users' table missing 'active' column used in WHERE clause
          ↑ Root cause: migration 027 was not included in latest deploy script
```

Each level required one targeted query (a log, a SQL check, a file read) — not guessing.

### Multi-Component Diagnostic

When the system has multiple layers (API → Service → Queue → Worker → DB), add a trace marker at each boundary:

```bash
# Quick diagnostic: add logging at each component boundary
echo "[DEBUG] API received request: $REQUEST_ID"
echo "[DEBUG] Service processed: $REQUEST_ID, result: $RESULT"
echo "[DEBUG] Worker picked up: $REQUEST_ID"
```

Then reproduce the bug and follow the trace marker through the logs. The component where the marker disappears or the data changes is your investigation target.

## 防御深度 — Defense in Depth (Post-Fix Validation)

After finding and fixing the root cause, add validation at multiple layers to prevent recurrence:

| Layer | What to add | Example |
|-------|-------------|---------|
| **Entry point** | Input validation | Reject missing fields before they propagate |
| **Business logic** | Precondition checks | Assert invariants at function entry |
| **Data layer** | Constraint enforcement | DB constraints, NOT NULL, foreign keys |
| **Test suite** | Regression test | The specific scenario that exposed this bug |

The regression test alone is not enough. If the same type of error can enter through a different path, it will. Defense in depth means closing the class of errors, not just the instance.

## 时序问题 — Timing Issues (Condition-Based Waiting)

For intermittent/flaky failures, suspect timing issues first:

**Replace arbitrary waits with condition-based polling:**

```typescript
// Bad: arbitrary timeout
await sleep(3000);
expect(result).toBeDefined();

// Good: wait for the actual condition
await waitFor(() => expect(result).toBeDefined(), { timeout: 5000 });
```

**Common timing patterns:**

| Pattern | Symptom | Fix |
|---------|---------|-----|
| Race condition | Passes sometimes, fails under load | Add proper synchronization or sequencing |
| Stale cache | Old data returned after update | Invalidate cache, or wait for propagation |
| Event ordering | Works locally, fails in CI | Make the test wait for the event, not a timer |

---

## Anti-Patterns — 禁忌 Jin Ji

These are traps. Avoid them absolutely.

- **Never guess-and-check without a hypothesis.** Random changes are not debugging — they are gambling.
- **Never fix symptoms without understanding the cause.** A suppressed symptom will resurface worse.
- **If 3+ attempted fixes fail, STOP.** You are likely questioning the wrong thing. Step back and question the architecture, not your fix.
- **"It works now but I don't know why" is not fixed** — it is a time bomb. Find out why, or it will detonate in production.

### Self-Check — 反求诸己

When tempted to skip investigation and jump to a fix:

- Have I completed all four diagnostic methods (望闻问切), or did I skip to 切?
- Am I forming a hypothesis, or am I guessing?
- If this fix doesn't work, do I have a next hypothesis — or will I be guessing again?
- Can I explain to someone else WHY this fix should work?

### Quick Reference

| Phase | Activities | You're done when... |
|-------|-----------|---------------------|
| 望 Observe | Error message, blast radius, frequency, timing | You can describe the symptom precisely |
| 闻 Listen | Logs, environment, recent changes | You know what changed and what the system is doing |
| 问 Inquire | 5-Why, call path tracing, hypothesis formation | You have a ranked list of likely causes |
| 切 Examine | Code reading, targeted logging, minimal repro | You've confirmed the root cause with evidence |
| 治 Treat | Fix + regression test + defense in depth | The fix is verified and the class of error is prevented |

---

## Transition

- **Fix complete** → Return to the caller (`execute` or standalone workflow).
- **Deep issue identified** → Invoke `bitfrog:brainstorm` to redesign before attempting a fix.
