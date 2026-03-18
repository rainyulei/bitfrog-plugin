# BitFrog Plugin Implementation Plan

> **For agentic workers:** Use this plan to implement the BitFrog Plugin step by step. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a Claude Code / Codex / OpenCode plugin that brings Chinese philosophy-driven development workflows via a single brain skill with auto-routing to internal sub-skills.

**Architecture:** One brain skill (`skills/bitfrog/SKILL.md`) injected at session start handles dialectical triage and routes to 6 internal sub-skills. Platform-specific entry layers (`.claude-plugin/`, `.codex/`, `.opencode/`, `hooks/`) share the same skill/agent/command assets.

**Tech Stack:** Markdown (skills/agents/commands), Bash (hooks), JavaScript ES module (OpenCode plugin)

**Spec:** `docs/specs/2026-03-18-bitfrog-plugin-design.md`

---

## Task 1: Project Scaffolding

**Files:**
- Create: `package.json`
- Create: `LICENSE`
- Create: `.gitignore`

- [ ] **Step 1: Initialize git repo**

```bash
cd /Users/rainlei/holiday/bit-frog-plugin
git init
```

- [ ] **Step 2: Create package.json**

```json
{
  "name": "bitfrog-plugin",
  "version": "1.0.0",
  "description": "Chinese philosophy-driven development workflows for Claude Code, Codex, and OpenCode",
  "author": {
    "name": "rainlei"
  },
  "homepage": "https://github.com/rainyulei/bitfrog-plugin",
  "repository": "https://github.com/rainyulei/bitfrog-plugin",
  "license": "MIT",
  "keywords": ["skills", "chinese-philosophy", "tdd", "debugging", "brainstorm", "bitfrog", "workflows"]
}
```

- [ ] **Step 3: Create .gitignore**

```
node_modules/
.DS_Store
*.log
```

- [ ] **Step 4: Create LICENSE (MIT)**

- [ ] **Step 5: Commit**

```bash
git add package.json LICENSE .gitignore
git commit -m "chore: initialize bitfrog-plugin project"
```

---

## Task 2: Claude Code Plugin Registration

**Files:**
- Create: `.claude-plugin/plugin.json`
- Create: `.claude-plugin/marketplace.json`

- [ ] **Step 1: Create plugin.json**

```json
{
  "name": "bitfrog-plugin",
  "description": "Chinese philosophy-driven development workflows — one brain, many paths",
  "version": "1.0.0",
  "author": {
    "name": "rainlei"
  },
  "homepage": "https://github.com/rainyulei/bitfrog-plugin",
  "repository": "https://github.com/rainyulei/bitfrog-plugin",
  "license": "MIT",
  "keywords": ["skills", "chinese-philosophy", "tdd", "debugging", "brainstorm", "bitfrog"],
  "skills": "./skills/",
  "agents": "./agents/",
  "commands": "./commands/",
  "hooks": "./hooks/hooks.json"
}
```

- [ ] **Step 2: Create marketplace.json**

```json
{
  "name": "bitfrog-marketplace",
  "description": "BitFrog Plugin marketplace",
  "owner": {
    "name": "rainlei"
  },
  "plugins": [
    {
      "name": "bitfrog-plugin",
      "description": "Chinese philosophy-driven development workflows — one brain, many paths",
      "version": "1.0.0",
      "source": "./",
      "author": {
        "name": "rainlei"
      }
    }
  ]
}
```

- [ ] **Step 3: Commit**

```bash
git add .claude-plugin/
git commit -m "feat: add Claude Code plugin registration"
```

---

## Task 3: SessionStart Hook

**Files:**
- Create: `hooks/hooks.json`
- Create: `hooks/session-start`

- [ ] **Step 1: Create hooks.json**

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/session-start\"",
            "async": false
          }
        ]
      }
    ]
  }
}
```

- [ ] **Step 2: Create session-start bash script**

The script reads `skills/bitfrog/SKILL.md`, strips frontmatter, escapes for JSON, and outputs the hook-specific JSON format. Reference superpowers' `hooks/session-start` for the escape logic and platform detection (Claude Code vs others).

Key behaviors:
- Read brain SKILL.md content
- Escape for JSON (backslashes, quotes, newlines, tabs)
- Output `hookSpecificOutput.additionalContext` for Claude Code
- Output `additional_context` for other platforms as fallback
- Wrap content in `<BITFROG>...</BITFROG>` tags

- [ ] **Step 3: Make session-start executable**

```bash
chmod +x hooks/session-start
```

- [ ] **Step 4: Commit**

```bash
git add hooks/
git commit -m "feat: add SessionStart hook for brain skill injection"
```

---

## Task 4: Brain Skill — `skills/bitfrog/SKILL.md`

**Files:**
- Create: `skills/bitfrog/SKILL.md`

This is the core of the entire plugin. It must contain:

- [ ] **Step 1: Write the brain skill**

Structure:
```markdown
---
name: bitfrog
description: "Chinese philosophy-driven development — one brain that assesses context and auto-routes to the right workflow. 蛙鸣万物，道法自然。"
---

# BitFrog — 蛙鸣万物，道法自然

[Philosophy framework: 5 principles + meta-principle with Chinese/Pinyin/English]

[Dialectical Triage table with signals → diagnosis → route]

[Routing principles: 2-sentence response, ask ONE question when ambiguous, user override]

[Instruction to invoke sub-skills via Skill tool: bitfrog:brainstorm, bitfrog:plan, etc.]
```

Content must be adapted from:
- BitFrog Copilot's `bitfrog-philosophy.md` (five principles)
- BitFrog Copilot's `bitfrog.agent.md` (router logic)
- Design spec triage table and routing principles

- [ ] **Step 2: Verify hook loads it**

Test manually: run `hooks/session-start` and confirm JSON output contains the SKILL.md content.

```bash
CLAUDE_PLUGIN_ROOT=. ./hooks/session-start | python3 -m json.tool
```

- [ ] **Step 3: Commit**

```bash
git add skills/bitfrog/
git commit -m "feat: add brain skill with philosophy framework and dialectical triage"
```

---

## Task 5: Brainstorm Sub-Skill

**Files:**
- Create: `skills/brainstorm/SKILL.md`

- [ ] **Step 1: Write the brainstorm skill**

Structure: Philosophy (格物致知) → Workflow (probe → reverse think → propose → present → write spec) → Embedded Tools (spec writing, spec-reviewer dispatch loop, user review gate) → Transition (→ bitfrog:plan)

Adapt from:
- BitFrog Copilot's `bitfrog-brainstorm.agent.md` (philosophy and probing approach)
- Superpowers' `brainstorming/SKILL.md` (spec writing, review loop, checklist)

Key: Merge BitFrog's philosophical probing with superpowers' spec-review-loop mechanics.

- [ ] **Step 2: Commit**

```bash
git add skills/brainstorm/
git commit -m "feat: add brainstorm skill — 格物致知"
```

---

## Task 6: Plan Sub-Skill

**Files:**
- Create: `skills/plan/SKILL.md`

- [ ] **Step 1: Write the plan skill**

Structure: Philosophy (格物→致知) → Phase 1 Ge Wu (dependency mapping, git history pattern discovery, modification order) → Phase 2 Zhi Zhi (task decomposition, TDD per task, exact paths/code/commands) → Embedded Tools (git worktree creation, parallel task marking) → Transition (→ bitfrog:execute)

Adapt from:
- BitFrog Copilot's `bitfrog-plan.agent.md` (two-phase approach, context map)
- Superpowers' `writing-plans/SKILL.md` (bite-sized tasks, plan format, file structure mapping)
- Superpowers' `using-git-worktrees/SKILL.md` (worktree creation steps)

Key: BitFrog's dependency mapping philosophy + superpowers' concrete task format.

- [ ] **Step 2: Commit**

```bash
git add skills/plan/
git commit -m "feat: add plan skill — 格物致知"
```

---

## Task 7: Execute Sub-Skill

**Files:**
- Create: `skills/execute/SKILL.md`

- [ ] **Step 1: Write the execute skill**

Structure: Philosophy (知行合一) → Workflow (read plan → TDD per task → progress reports) → Embedded Tools (parallel subagent dispatch, verification-before-completion) → Problem Escalation (3 retries → escalate) → Transition (→ bitfrog:review)

Adapt from:
- BitFrog Copilot's `bitfrog-execute.agent.md` (TDD cycle, progress reporting, escalation ladder)
- Superpowers' `executing-plans/SKILL.md` (plan loading, task tracking)
- Superpowers' `subagent-driven-development/SKILL.md` (parallel dispatch, two-stage review)
- Superpowers' `dispatching-parallel-agents/SKILL.md` (agent dispatch patterns)
- Superpowers' `verification-before-completion/SKILL.md` (verification gate)

Key: BitFrog's escalation philosophy + superpowers' parallel execution and verification mechanics.

- [ ] **Step 2: Commit**

```bash
git add skills/execute/
git commit -m "feat: add execute skill — 知行合一"
```

---

## Task 8: Debug Sub-Skill

**Files:**
- Create: `skills/debug/SKILL.md`

- [ ] **Step 1: Write the debug skill**

Structure: Philosophy (辩证论治) → Four Diagnostics (望闻问切) → Issue Classification (表证/里证/深证) → Transition (return to caller or escalate)

Adapt from:
- BitFrog Copilot's `bitfrog-debug.agent.md` (four diagnostic methods, three-level classification)
- Superpowers' `systematic-debugging/SKILL.md` (4-phase root cause analysis, hypothesis testing)

Key: BitFrog's TCM diagnostic framework + superpowers' systematic hypothesis-test methodology.

- [ ] **Step 2: Commit**

```bash
git add skills/debug/
git commit -m "feat: add debug skill — 辩证论治"
```

---

## Task 9: Review Sub-Skill

**Files:**
- Create: `skills/review/SKILL.md`

- [ ] **Step 1: Write the review skill**

Structure: Philosophy (三省吾身) → Three Reflections (自省: spec compliance, 互省: code-reviewer subagent, 终省: user value) → Issue Severity → Embedded Tools (subagent dispatch, finish-branch options, worktree cleanup) → Key Rule ("You're right!" is dangerous)

Adapt from:
- BitFrog Copilot's `bitfrog-review.agent.md` (three reflections, dangerous agreement)
- Superpowers' `requesting-code-review/SKILL.md` (subagent dispatch with git SHAs)
- Superpowers' `receiving-code-review/SKILL.md` (verify before implementing feedback)
- Superpowers' `finishing-a-development-branch/SKILL.md` (4 completion options, worktree cleanup)

Key: BitFrog's three-reflection philosophy + superpowers' review dispatch and branch completion mechanics.

- [ ] **Step 2: Commit**

```bash
git add skills/review/
git commit -m "feat: add review skill — 三省吾身"
```

---

## Task 10: Mentor Sub-Skill

**Files:**
- Create: `skills/mentor/SKILL.md`

- [ ] **Step 1: Write the mentor skill**

Structure: Philosophy (不愤不启) → 5-Level Escalation → Learning Progress Display → Transition (independent, suggest switch at level 5)

Adapt directly from BitFrog Copilot's `bitfrog-mentor.agent.md`. This skill has no superpowers equivalent — it's purely BitFrog's contribution.

- [ ] **Step 2: Commit**

```bash
git add skills/mentor/
git commit -m "feat: add mentor skill — 不愤不启"
```

---

## Task 11: Code Reviewer Agent

**Files:**
- Create: `agents/code-reviewer.md`

- [ ] **Step 1: Write the code-reviewer agent definition**

Based on spec section "Agents > code-reviewer". Include:
- YAML frontmatter (name, model: inherit)
- Full system prompt with 三省 framework
- Output format (Critical/Important/Suggestion + verdict)
- Can be dispatched for both code review and spec review with different context

Adapt from:
- Design spec agent section
- Superpowers' `agents/code-reviewer.md`

- [ ] **Step 2: Commit**

```bash
git add agents/
git commit -m "feat: add code-reviewer subagent"
```

---

## Task 12: Slash Commands

**Files:**
- Create: `commands/brainstorm.md`
- Create: `commands/plan.md`
- Create: `commands/execute-plan.md`
- Create: `commands/debug.md`
- Create: `commands/review.md`
- Create: `commands/mentor.md`

- [ ] **Step 1: Create all 6 command files**

Each is a simple skill invocation redirect:

```markdown
---
name: <command-name>
description: <one-line description>
---
Invoke the bitfrog:<skill-name> skill.
```

- [ ] **Step 2: Commit**

```bash
git add commands/
git commit -m "feat: add slash commands for all 6 workflows"
```

---

## Task 13: Codex Platform Support

**Files:**
- Create: `.codex/INSTALL.md`

- [ ] **Step 1: Write Codex installation guide**

Include:
- Prerequisites (git)
- Clone + symlink instructions (Linux/Mac + Windows PowerShell)
- Verification command
- Update instructions
- Uninstall instructions
- Tool mapping notes

Reference superpowers' `.codex/INSTALL.md` for format.

- [ ] **Step 2: Commit**

```bash
git add .codex/
git commit -m "feat: add Codex platform support"
```

---

## Task 14: OpenCode Platform Support

**Files:**
- Create: `.opencode/plugins/bitfrog.js`

- [ ] **Step 1: Write the OpenCode plugin**

ES module that:
1. Reads `skills/bitfrog/SKILL.md` at startup
2. Strips frontmatter
3. Injects content into system prompt via `experimental.chat.system.transform`
4. Registers `skills/` directory via `config` hook
5. Includes tool mapping note for OpenCode equivalents

Reference superpowers' `.opencode/plugins/superpowers.js` for the plugin API.

- [ ] **Step 2: Commit**

```bash
git add .opencode/
git commit -m "feat: add OpenCode platform support"
```

---

## Task 15: Final Verification & README

**Files:**
- Create: `README.md`

- [ ] **Step 1: Test Claude Code hook**

```bash
cd /Users/rainlei/holiday/bit-frog-plugin
CLAUDE_PLUGIN_ROOT=. ./hooks/session-start | python3 -m json.tool
```

Expected: Valid JSON with brain skill content in `hookSpecificOutput.additionalContext`.

- [ ] **Step 2: Verify all files exist**

```bash
find . -name "*.md" -o -name "*.json" -o -name "*.js" -o -name "session-start" | sort
```

Expected: All files from the directory structure in the spec.

- [ ] **Step 3: Write README.md**

Include:
- Project description (bilingual)
- Installation for all 3 platforms
- How it works (one brain, auto-routing)
- Philosophy overview
- Workflow diagram
- Credits (BitFrog Copilot + Superpowers)
- License

- [ ] **Step 4: Final commit**

```bash
git add README.md
git commit -m "docs: add README with installation and usage guide"
```

---

## Task Summary

| Task | Description | Dependencies |
|------|-------------|-------------|
| 1 | Project scaffolding | None |
| 2 | Claude Code plugin registration | 1 |
| 3 | SessionStart hook | 1 |
| 4 | Brain skill (bitfrog) | 1 |
| 5 | Brainstorm sub-skill | 4 |
| 6 | Plan sub-skill | 4 |
| 7 | Execute sub-skill | 4 |
| 8 | Debug sub-skill | 4 |
| 9 | Review sub-skill | 4 |
| 10 | Mentor sub-skill | 4 |
| 11 | Code reviewer agent | 4 |
| 12 | Slash commands | 4 |
| 13 | Codex platform support | 1 |
| 14 | OpenCode platform support | 4 |
| 15 | Final verification & README | All |

**Parallelizable:** Tasks 5-12 are independent of each other (all depend only on Task 4). Tasks 2, 3, 13 are independent of each other (all depend only on Task 1).
