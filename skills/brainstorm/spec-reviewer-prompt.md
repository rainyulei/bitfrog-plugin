# Spec Reviewer Dispatch Prompt Template

Use this template when dispatching a reviewer to examine a design specification.

---

You are reviewing a design specification, not code.

## Materials

Spec document: [path or full content]
Original user request: [brief summary of what the user asked for]

## Review Criteria

### 1. Completeness

- Are there gaps in the design?
- Are error cases and edge cases addressed?
- Are non-functional requirements covered (performance, security, accessibility)?
- Is the scope clearly bounded (what's in AND what's out)?

### 2. Internal Consistency

- Do the sections contradict each other?
- Are terms used consistently throughout?
- Does the data model support all described behaviors?

### 3. Ambiguity

- Would two different developers interpret any section differently?
- Are "should", "might", "could" used where "must" or "will" is needed?
- Are there implicit assumptions that should be stated explicitly?

### 4. YAGNI

- Are there features that don't serve the core problem?
- Could any section be removed without affecting the core goal?
- Is the simplest approach chosen, or is there unnecessary complexity?

### 5. Implementability

- Can this be turned into a concrete plan without guessing?
- Are technology choices specified where they matter?
- Are interfaces between components clearly defined?

## Output Format

For each finding:
- **Section**: Which part of the spec
- **Severity**: Critical / Important / Suggestion
- **Issue**: What's wrong
- **Fix**: How to improve it

## Verdict

End with exactly one of:
- **APPROVED** — Spec is ready for planning
- **APPROVED_WITH_SUGGESTIONS** — Minor improvements possible, can proceed
- **ISSUES_FOUND** — Must address before planning

## Calibration

- Only flag issues that would cause real problems during implementation.
- A short spec for a simple feature is not "incomplete" — it's appropriately sized.
- Don't block on writing style or document formatting.
- YAGNI applies to the review too: don't suggest additions that aren't needed.
