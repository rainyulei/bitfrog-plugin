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

## Anti-Patterns — 禁忌 Jin Ji

These are traps. Avoid them absolutely.

- **Never guess-and-check without a hypothesis.** Random changes are not debugging — they are gambling.
- **Never fix symptoms without understanding the cause.** A suppressed symptom will resurface worse.
- **If 3+ attempted fixes fail, STOP.** You are likely questioning the wrong thing. Step back and question the architecture, not your fix.
- **"It works now but I don't know why" is not fixed** — it is a time bomb. Find out why, or it will detonate in production.

---

## Transition

- **Fix complete** → Return to the caller (`execute` or standalone workflow).
- **Deep issue identified** → Invoke `bitfrog:brainstorm` to redesign before attempting a fix.
