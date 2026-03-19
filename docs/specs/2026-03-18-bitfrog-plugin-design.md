# BitFrog Plugin — Design Specification

> 蛙鸣万物，道法自然
> One entry, many paths — philosophy guides, tools serve.

## Overview

BitFrog Plugin is a Claude Code / Codex / OpenCode plugin that brings Chinese philosophy-driven thinking models to AI-assisted software development. It adapts the BitFrog Copilot agent system (originally a VS Code/GitHub Copilot extension) into a skill-based plugin architecture.

**Core differentiator:** Unlike rule-driven frameworks (e.g., "you MUST write tests first"), BitFrog cultivates genuine understanding through philosophical thinking models. An agent that truly understands _why_ tests matter writes them naturally.

## Design Principles

### One Entry, Many Paths (一即是全)

The user sees **one skill**: `bitfrog`. It loads at session start, assesses the current situation via dialectical triage, and automatically invokes the appropriate internal workflow. Users never need to think about "which skill to use."

### Philosophy as Skeleton, Tools as Flesh (哲学为骨，工具为肉)

Each internal workflow embeds both philosophical thinking models AND concrete tool instructions in a single SKILL.md. Philosophy guides _when_ and _why_; tool instructions specify _how_.

### Bilingual by Nature (双语呈现)

Chinese philosophical terms are preserved with pinyin and English explanations: "格物致知 Ge Wu Zhi Zhi (Investigate to understand the essence)". This lets international users appreciate the philosophical depth while understanding the practical meaning.

## Philosophy Framework

### Five Principles

| # | Chinese | Pinyin | English | Applied To |
|---|---------|--------|---------|------------|
| 1 | 格物致知 | Ge Wu Zhi Zhi | Investigate to understand essence | brainstorm, plan |
| 2 | 知行合一 | Zhi Xing He Yi | Unity of knowledge and action | execute, verify |
| 3 | 辩证论治 | Bian Zheng Lun Zhi | Diagnose before prescribing | debug, triage |
| 4 | 阴阳互生 | Yin Yang Hu Sheng | Complementary collaboration | parallel agents |
| 5 | 三省吾身 | San Sheng Wu Shen | Three levels of reflection | review |

### Meta-Principle

**中庸之道 Zhong Yong Zhi Dao (The Doctrine of the Mean)** — Every action has its appropriate measure. Neither excess nor deficiency. This is not compromise — it is judgment.

## Architecture

### Directory Structure

```
bitfrog-plugin/
├── .claude-plugin/
│   ├── plugin.json              # Claude Code plugin registration
│   └── marketplace.json         # Marketplace publishing
├── .codex/
│   └── INSTALL.md               # Codex installation guide
├── .opencode/
│   └── plugins/bitfrog.js       # OpenCode plugin entry
│
├── skills/
│   ├── bitfrog/                  # The Brain — only user-facing skill
│   │   └── SKILL.md             # Philosophy framework + dialectical triage
│   │
│   ├── brainstorm/              # Internal sub-skills (auto-invoked)
│   │   └── SKILL.md
│   ├── plan/
│   │   └── SKILL.md
│   ├── execute/
│   │   └── SKILL.md
│   ├── debug/
│   │   └── SKILL.md
│   ├── review/
│   │   └── SKILL.md
│   └── mentor/
│       └── SKILL.md
│
├── agents/
│   └── code-reviewer.md         # Subagent for review skill
│
├── hooks/
│   ├── hooks.json                # Claude Code SessionStart config
│   └── session-start             # Injects brain skill into context
│
├── commands/
│   ├── brainstorm.md             # /brainstorm shortcut
│   ├── plan.md                   # /plan shortcut
│   └── execute-plan.md           # /execute-plan shortcut
│
├── package.json
└── LICENSE
```

### Core Mechanism

1. **Session Start:** `hooks/session-start` is a bash script that reads `skills/bitfrog/SKILL.md` and outputs JSON to stdout. The Claude Code hooks system captures this output and injects it into the session context. Example output:
   ```json
   {
     "hookSpecificOutput": {
       "hookEventName": "SessionStart",
       "additionalContext": "<BITFROG>\n...contents of SKILL.md...\n</BITFROG>"
     }
   }
   ```
2. **Dialectical Triage:** The brain skill assesses each user message and determines the appropriate workflow path.
3. **Sub-skill Invocation:** The brain invokes internal sub-skills via the Skill tool using the `bitfrog:<name>` namespace. Claude Code's plugin system automatically maps `bitfrog:<name>` to `skills/<name>/SKILL.md` within the plugin directory, based on the plugin name declared in `.claude-plugin/plugin.json`. The sub-skills are registered alongside the brain skill but are designed to be invoked by the brain, not directly by the user. Users see them listed but the brain's routing makes manual selection unnecessary.
4. **Workflow Chaining:** Sub-skills flow into each other naturally: brainstorm → plan → execute → review → finish.
5. **User Override:** If the brain's triage is wrong, the user can explicitly say "no, I want to debug" or use a slash command (`/brainstorm`, `/plan`, etc.) to override. The brain always respects explicit user intent over its own judgment.

## The Brain: `skills/bitfrog/SKILL.md`

### Responsibilities

1. Inject the five philosophical principles into session context
2. Perform dialectical triage on each user message
3. Route to the appropriate sub-skill

### Dialectical Triage Table

| Signal | Diagnosis | Route |
|--------|-----------|-------|
| Clear error / unexpected behavior | 病症 (illness) | → bitfrog:debug |
| Asking "why" / wants to understand | 求知 (seeking knowledge) | → bitfrog:mentor |
| Requesting review / pre-commit check | 省察 (reflection) | → bitfrog:review |
| Has design doc, needs task breakdown | 致知 (derived knowledge) | → bitfrog:plan |
| Has plan, ready to implement | 行动 (action) | → bitfrog:execute |
| Everything else (new idea / vague request) | 探索 (exploration) | → bitfrog:brainstorm |

### Routing Principles

- What the user says is the surface; the intent behind it is the essence
- When intent is ambiguous, ask ONE question (only one)
- Never assume, never rush ahead
- Express routing judgment in 2 sentences: understanding + recommended direction
- If the user disagrees with routing, immediately switch — user intent always overrides triage
- If a sub-skill fails mid-workflow, surface the error to the user with context and suggest next steps (retry, switch skill, or manual intervention)

## Sub-Skills

Each sub-skill follows a unified structure: **Philosophy → Workflow → Embedded Tools → Exit/Transition**.

### brainstorm (格物致知 — Investigate to Understand)

**Philosophy:** X is the user's proposed solution, not their problem. Dig for the real problem.

**Workflow:**
1. Probe root causes — ask "why" iteratively ("Add caching" → "Which scenario is slow?" → "Have you checked the query plan?")
2. Reverse thinking — "What happens if we don't build this at all?"
3. Propose 2-3 approaches with benefits and costs
4. Present design in sections, confirm each section
5. Write spec document to `docs/specs/`

**Embedded Tools:**
- Spec document writing (`docs/specs/YYYY-MM-DD-<topic>-design.md`)
- Spec-reviewer subagent dispatch loop (max 3 iterations)
- User review gate before proceeding

**Transition:** Design approved → auto invoke bitfrog:plan

### plan (格物→致知 — Map then Decompose)

**Philosophy:** First map the terrain (ge wu), then derive the path (zhi zhi).

**Workflow:**
- **Phase 1 — Ge Wu (Dependency Mapping):**
  - Map primary files and trace inbound/outbound dependencies
  - Discover existing patterns from git history
  - Determine safest modification order: types → utils → core → consumers → tests
  - Produce a Context Map for user confirmation

- **Phase 2 — Zhi Zhi (Task Decomposition):**
  - Break into 2-5 minute tasks
  - Each task includes TDD cycle: test → verify fail → implement → verify pass → commit
  - Include exact file paths, complete code snippets, precise commands
  - Save plan to `docs/plans/YYYY-MM-DD-<topic>-plan.md`

**Embedded Tools:**
- Git worktree creation for isolated workspace
- Parallel task identification and marking (for execute phase)

**Transition:** Plan confirmed → auto invoke bitfrog:execute

### execute (知行合一 — Unity of Knowledge and Action)

**Philosophy:** True knowledge manifests as action. If you know you should write tests but skip them, you don't truly know why tests matter.

**Workflow:**
1. Read the plan
2. Per-task TDD cycle: Write test (RED) → Confirm fail → Implement (GREEN) → Confirm pass → Refactor
3. Report progress every 3 tasks
4. Verify before any completion claim

**Embedded Tools:**
- Parallel subagent dispatch — when tasks are independent, dispatch multiple agents simultaneously (阴阳互生)
- Verification-before-completion — must run verification command and confirm output before claiming success (知行合一: knowledge without verification is not true knowledge)

**Problem Escalation:**
- Clear test failure → fix directly
- Unclear failure → invoke bitfrog:debug
- Plan is wrong → invoke bitfrog:plan
- Design flaw → invoke bitfrog:brainstorm
- Max 3 retries before escalating

**Transition:** All tasks complete → auto invoke bitfrog:review

### debug (辩证论治 — Diagnose Before Prescribing)

**Philosophy:** Same symptom can have different root causes. The Four Diagnostic Methods from traditional Chinese medicine.

**Workflow:**
- 望 Wang (Observe): Full error message, blast radius, frequency, timing
- 闻 Wen (Listen): Runtime environment, logs, recent deployments
- 问 Wen (Inquire): 5-why root cause tracing
- 切 Qie (Examine): Deep-dive with breakpoints/logging

**Issue Classification:**
- 表证 Biao Zheng (Surface): Local bug → fix directly + write regression test
- 里证 Li Zheng (Internal): Systemic issue → fix root cause, not symptom
- 深证 Shen Zheng (Deep): Architectural issue → invoke bitfrog:brainstorm

**Transition:** Fix complete → return to caller (execute or standalone)

### review (三省吾身 — Three Levels of Reflection)

**Philosophy:** Self-reflection reveals blind spots, peer-reflection provides independent perspective, final-reflection confirms user value.

**Workflow:**
1. 自省 Zi Sheng (Self-reflection): Compare plan vs git diff — spec compliance
2. 互省 Hu Sheng (Peer-reflection): Dispatch code-reviewer subagent — code quality, appropriate abstraction, debuggability
3. 终省 Zhong Sheng (Final-reflection): Does this actually solve the user's original problem?

**Issue Severity:** Critical / Important / Minor

**Embedded Tools:**
- Code-reviewer subagent dispatch
- Finish-branch options: local merge / create PR / keep branch / discard
- Git worktree cleanup

**Key Rule:** "You're absolutely right!" is the most dangerous response. Verify feedback before accepting.

**Transition:** Review passed → present completion options

### mentor (不愤不启 — Only Guide When Ready to Learn)

**Philosophy:** Never give answers directly. Teach through hints and questions.

**Workflow — 5-Level Escalation:**

| Level | Action | Example |
|-------|--------|---------|
| 1 | Point a direction | "Look at how `similar_function` handles this" |
| 2 | Point to a file | "Check `src/auth/middleware.ts`" |
| 3 | Name the pattern | "This is the Observer pattern" |
| 4 | Explain the thinking | "The reason this works is..." |
| 5 | Almost the answer | Show 90%, leave the last step |

- Always start at level 1, escalate only when stuck
- Show learning progress at end of each interaction
- If stuck at level 5, suggest switching to another workflow

**Transition:** Independent — does not auto-chain to other skills

## Agents

### code-reviewer

```yaml
name: code-reviewer
model: inherit
```

Senior code reviewer dispatched by the review skill's 互省 (Hu Sheng) phase. The caller provides the subagent with: git diff output, plan/spec file path, and a description of what was implemented.

**System Prompt Summary:**

You are a senior code reviewer applying the 三省 (Three Reflections) framework:

1. **Plan Compliance (自省):** Compare the git diff against the plan/spec. Flag deviations — missing tasks, extra changes, spec misinterpretations.
2. **Code Quality (互省):** Evaluate readability, appropriate abstraction level, debuggability, error handling, naming conventions. Ask: "Does this solve the real problem, or just the surface problem?"
3. **Architecture (终省):** Assess impact on the broader system. Flag coupling issues, missing tests, performance concerns.

**Output Format:**
- Categorize every finding as **Critical** (must fix before merge), **Important** (should fix), or **Suggestion** (nice to have)
- For each finding: file, line range, issue description, suggested fix
- End with a verdict: APPROVED, APPROVED_WITH_SUGGESTIONS, or CHANGES_REQUESTED

### spec-reviewer

The brainstorm skill's spec review loop reuses the `code-reviewer` agent with a different dispatch prompt. Instead of reviewing code, the caller provides the spec document content and asks the reviewer to check for: completeness, internal consistency, ambiguity, missing edge cases, and implementability. The same agent definition is used — only the dispatch context differs.

## Commands

Slash commands provide explicit shortcuts. Optional — the brain handles routing automatically.

| Command | Target |
|---------|--------|
| `/brainstorm` | Invoke bitfrog:brainstorm |
| `/plan` | Invoke bitfrog:plan |
| `/execute-plan` | Invoke bitfrog:execute |
| `/debug` | Invoke bitfrog:debug |
| `/review` | Invoke bitfrog:review |
| `/mentor` | Invoke bitfrog:mentor |

## Platform Adaptation

### Claude Code

- **Registration:** `.claude-plugin/plugin.json` declares paths to skills, agents, commands, hooks
- **Injection:** `hooks/hooks.json` → SessionStart → `hooks/session-start` script → reads brain SKILL.md → outputs JSON with `additionalContext`
- **Skills Access:** Via Skill tool

### Codex

- **Registration:** User clones the repo and creates a symlink:
  ```bash
  git clone https://github.com/rainyulei/bitfrog-plugin.git ~/.codex/bitfrog
  mkdir -p ~/.agents/skills
  ln -s ~/.codex/bitfrog/skills ~/.agents/skills/bitfrog
  ```
- **Injection:** Codex natively discovers skills through the symlink at `~/.agents/skills/bitfrog/`. The brain skill (`bitfrog/SKILL.md`) is loaded when referenced. Sub-skills are accessible as `bitfrog/brainstorm`, `bitfrog/plan`, etc.
- **Skills Access:** Codex native skill discovery. Use `skill` tool to list and load.
- **Updating:** `cd ~/.codex/bitfrog && git pull` — symlink ensures instant updates.
- **Tool Mapping:** When skills reference Claude Code tools, Codex equivalents apply: `TodoWrite` → `todowrite`, `Skill` → native `skill` tool, file operations → native tools.

### OpenCode

- **Registration:** Add to `opencode.json`:
  ```json
  {
    "plugin": ["bitfrog@git+https://github.com/rainyulei/bitfrog-plugin.git"]
  }
  ```
- **Injection:** `.opencode/plugins/bitfrog.js` is an ES module that:
  1. Reads `skills/bitfrog/SKILL.md` at startup
  2. Injects its content into the system prompt via `experimental.chat.system.transform`
  3. Registers the `skills/` directory path via the `config` hook so OpenCode discovers all sub-skills
- **Plugin Entry Point Signature:**
  ```javascript
  export const BitfrogPlugin = async ({ client, directory }) => {
    return {
      config: async (config) => { /* register skills path */ },
      'experimental.chat.system.transform': async (_input, output) => {
        /* inject brain SKILL.md into output.system */
      }
    };
  };
  ```
- **Skills Access:** OpenCode native `skill` tool.
- **Tool Mapping:** `TodoWrite` → `todowrite`, `Task` with subagents → `@mention` syntax, `Skill` → OpenCode native `skill` tool.

### Shared Assets

All three platforms share the same `skills/`, `agents/`, and `commands/` directories. Only the entry layer (`.claude-plugin/`, `.codex/`, `.opencode/`, `hooks/`) is platform-specific.

## Workflow Overview

```
User message
    │
    ▼
┌─────────┐
│ BitFrog  │  ← Dialectical Triage (辩证分诊)
│ (Brain)  │
└────┬─────┘
     │
     ├── 探索 ──→ brainstorm ──→ plan ──→ execute ──→ review ──→ finish
     │                                       ↕
     ├── 病症 ──→ debug ──────────────→ return to caller
     │
     ├── 求知 ──→ mentor (independent)
     │
     ├── 省察 ──→ review
     ├── 致知 ──→ plan
     └── 行动 ──→ execute
```

## Success Criteria

1. User installs plugin once, never needs to think about skill selection
2. Philosophy is felt through behavior, not lecturing — the agent acts differently, not just talks differently
3. All tool capabilities (worktree, parallel agents, verify, finish-branch) work seamlessly within the philosophical workflows
4. Bilingual experience: Chinese terms with English explanations, international users feel included
5. Works consistently across Claude Code, Codex, and OpenCode
