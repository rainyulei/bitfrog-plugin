---
name: brainstorm
description: "格物致知 Ge Wu Zhi Zhi — Explore ideas and design solutions by investigating the essence before proposing. Probes root causes, challenges assumptions, writes specs."
---

# Brainstorm Skill

## Philosophy — 格物致知 Ge Wu Zhi Zhi (Investigate Things to Attain Knowledge)

The ancient principle 格物致知 teaches: before you can act wisely, you must investigate the essence of things. In software, this means:

> **X is the user's proposed solution, not their problem. Dig for the real problem.**

When a user says "add caching," they are presenting a solution. Your job is to uncover the problem behind it. Apply three lenses:

1. **追根溯源 Zhuī Gēn Sù Yuán (Trace to the Root)** — Probe root causes by asking "why" iteratively until you reach the real constraint or pain point.
2. **反向思维 Fǎn Xiàng Sī Wéi (Reverse Thinking)** — Ask "What happens if we don't build this at all?" to test whether the problem is real and the solution is necessary.
3. **旁通曲鉴 Páng Tōng Qū Jiàn (Explore Alternatives)** — Never settle on the first approach. Generate at least two alternatives and compare them honestly.

Do not propose until you understand. Do not build until you have designed.

---

## Workflow

Follow this checklist in order. Do not skip steps.

- [ ] **Explore project context**
  - Read relevant source files, documentation, and recent commits.
  - Understand the current architecture and constraints before asking anything.

- [ ] **Ask clarifying questions — ONE AT A TIME**
  - Prefer multiple-choice format to reduce cognitive load on the user.
  - Example: "Which of these best describes the issue? (A) Slow page load (B) High memory usage (C) Timeout errors (D) Something else"

- [ ] **Probe root causes — 追根溯源**
  - Ask "why" iteratively. Do not accept the first answer.
  - Example chain:
    - User: "Add caching for the API."
    - You: "Which specific scenario is slow?"
    - User: "The dashboard load."
    - You: "Have you checked the query plan for the dashboard query?"
    - User: "No..."
    - You: "Let me check that first — the fix might be an index, not a cache."

- [ ] **Apply reverse thinking — 反向思维**
  - Ask: "What happens if we don't build this at all?"
  - Identify the true cost of inaction. If the cost is low, challenge whether the work is needed.

- [ ] **Propose 2-3 approaches**
  - For each approach, state:
    - **Benefits** — what it solves, why it is good.
    - **Costs** — complexity, maintenance burden, risks.
    - **Recommendation** — which approach you favor and why.

- [ ] **Present design in sections, confirm each with user**
  - Break the design into logical sections (e.g., data model, API, UI).
  - Present one section at a time. Wait for user confirmation before moving to the next.

- [ ] **Apply YAGNI ruthlessly — 删繁就简 Shān Fán Jiù Jiǎn (Cut Complexity, Keep Simplicity)**
  - For every proposed feature, ask: "Do we need this in the first version?"
  - Remove anything that is not essential to solving the core problem.

- [ ] **Assess scope — 大而化之不如分而治之 (Better to divide and conquer than to oversimplify)**
  - Before diving into detailed questions, assess: does this request describe multiple independent subsystems?
  - If yes, help the user decompose into sub-projects first. Don't spend questions refining details of a project that needs decomposition.
  - Each sub-project gets its own spec → plan → execute → review cycle.
  - Brainstorm the first sub-project through the normal flow, then return for the next.

---

## Working in Existing Codebases — 入乡随俗 Rù Xiāng Suí Sú (When in Rome)

When brainstorming within an existing project, 格物 starts with what already exists:

1. **Explore the current structure first** — read source files, directory layout, existing patterns, before proposing anything new.
2. **Follow existing patterns** — if the codebase uses factories, use factories. If it uses flat modules, use flat modules. Consistency beats "better" patterns.
3. **Do not propose unrelated refactoring** — stay focused on the current goal. If existing code has problems that affect your work, include targeted improvements. Otherwise, leave it.
4. **Respect existing naming conventions** — check git history for how new files/functions are typically named in this project.
5. **Identify the blast radius** — before proposing changes, understand what depends on the code you want to modify.

The principle: your design should feel like it belongs in this codebase, not like it was dropped in from a different project.

---

## Embedded Tools

### Spec Document Writer

When the design is agreed upon, write the specification to:

```
docs/specs/YYYY-MM-DD-<topic>-spec.md
```

The spec must include:
- Problem statement (the real problem, not the original request)
- Chosen approach with rationale
- Scope and non-goals
- Technical design (broken into sections)
- Open questions (if any remain)

### Spec Reviewer — Subagent Dispatch Loop

After writing the spec, dispatch a review subagent to examine it. 格物致知 applies to our own output — investigate the spec before treating it as truth.

**Dispatch the code-reviewer agent using the template in `skills/brainstorm/spec-reviewer-prompt.md`.** It includes: 5 review criteria (completeness, consistency, ambiguity, YAGNI, implementability), output format, verdict options, and calibration notes.

Read the template file and fill in the placeholders (spec path, user request summary).

**Loop rules:**
1. If issues found → fix them in the spec, re-dispatch reviewer
2. Maximum 3 iterations
3. If still unresolved after 3 rounds, flag remaining concerns to the user

### User Review Gate

After the spec passes automated review (or exhausts the 3-iteration limit):

- Present the final spec to the user.
- Ask the user to review and confirm before proceeding.
- Do NOT proceed until the user explicitly approves.

---

## Hard Gate

> **Do NOT write any code or invoke any implementation skill until the design is presented and the user has approved.**

This is non-negotiable. No exceptions. If the user asks to skip design, explain why investigation matters (格物致知) and offer to accelerate the process — but never skip it entirely.

---

## Transition

When the user approves the design:

1. Confirm approval explicitly: "Design approved. Proceeding to planning."
2. Auto-invoke `bitfrog-plugin:plan` to begin implementation planning based on the approved spec.
