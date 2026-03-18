---
name: plan
description: "格物致知 Ge Wu Zhi Zhi — Map dependencies and decompose into bite-sized TDD tasks. Creates isolated worktrees and detailed implementation plans."
---

# Plan — 格物致知 Ge Wu Zhi Zhi

## 哲学 — Philosophy

Planning follows a two-phase approach drawn from 《大学》 (The Great Learning):

- **格物 Ge Wu (Investigating Things)** — First map the terrain. Understand what exists, what connects to what, and where the risks hide. Do not propose until you have observed.
- **致知 Zhi Zhi (Deriving Knowledge)** — Then derive the path. From thorough investigation, the correct sequence of changes becomes clear. The plan writes itself when the territory is known.

One cannot skip 格物 and jump to 致知. A plan built without investigation is a house built on sand.

## Phase 1 — 格物 Ge Wu (Dependency Mapping)

Before writing a single task, map the full landscape:

### 1. Identify Primary Files

- List every file that will be **created** or **modified**
- Use absolute paths — no ambiguity

### 2. Trace Dependencies

- **Inbound dependencies**: What imports or calls the files you will change? These are your blast radius.
- **Outbound dependencies**: What do your target files depend on? These are your prerequisites.

### 3. Discover Existing Patterns

- Examine git history for conventions:
  ```bash
  git log --oneline -20                    # recent commit style
  git log --diff-filter=A -- '*.ts'        # how new files are introduced
  git log --all --oneline -- <file>        # history of specific files
  ```
- Study coding conventions: naming, directory structure, test placement, export patterns
- Identify recurring patterns the new code must follow

### 4. Determine Modification Order

Follow the dependency-safe sequence:

```
types → utils → core → consumers → tests
```

- Types and interfaces first (they break nothing)
- Utilities and helpers next (leaf nodes)
- Core logic (depends on types + utils)
- Consumers of core logic (UI, routes, commands)
- Tests last (they depend on everything)

### 5. Produce Context Map

Present a concise Context Map to the user for confirmation:

```
Files to create:  [list]
Files to modify:  [list]
Blast radius:     [list of affected dependents]
Patterns found:   [key conventions to follow]
Modification order: [sequence]
```

**Stop here. Wait for user confirmation before proceeding to Phase 2.**

## Phase 2 — 致知 Zhi Zhi (Task Decomposition)

With the terrain mapped and confirmed, decompose the work:

### 1. Bite-Sized Tasks

- Each task should take **2-5 minutes** to complete
- A task that takes longer is too large — split it further
- Each task must be independently verifiable

### 2. TDD Rhythm Per Task

Every task follows the red-green-commit cycle:

1. **Write failing test** — Define expected behavior before implementation
2. **Verify fail** — Run the test, confirm it fails for the right reason
3. **Implement** — Write the minimum code to make the test pass
4. **Verify pass** — Run the test, confirm green
5. **Commit** — Lock in the progress with a meaningful message

### 3. Precision Requirements

Each task must include:

- **Exact file paths** — Absolute, no guessing
- **Complete code snippets** — Copy-pasteable, not pseudocode
- **Precise commands** — With expected output so the agent can verify success
- **Dependencies** — Which prior tasks must complete before this one starts

### 4. Parallel Marking

- Mark tasks that have **no dependency on each other** with `[parallel]`
- The execute phase can run independent tasks concurrently
- Tasks sharing a file or depending on a prior task's output must be sequential

### 5. Save the Plan

Save the completed plan document to:

```
docs/plans/YYYY-MM-DD-<topic>-plan.md
```

Create the `docs/plans/` directory if it does not exist.

### 6. Review the Plan — 格物不止于设计

格物致知 does not end when the spec is written. The plan itself is an artifact that deserves investigation.

After saving the plan, dispatch a review subagent:

**Dispatch the code-reviewer agent with:**

```
You are reviewing an implementation plan, not code.

Plan document: [path]
Spec document: [path]

Review for:
1. Spec alignment — Does every spec requirement have a corresponding task?
2. Task completeness — Does each task have clear files, code, commands, and expected output?
3. Dependency correctness — Are sequential/parallel markings accurate?
4. TDD coverage — Does every task include a test-first step?
5. Gap detection — Are there implicit assumptions not stated as tasks?

For each finding: task number, severity (Critical/Important/Suggestion), issue, suggested fix.
End with verdict: APPROVED / APPROVED_WITH_SUGGESTIONS / ISSUES_FOUND.
```

**Loop rules:** Fix issues and re-dispatch, max 3 iterations. Then present to user for confirmation.

## 嵌入工具 — Embedded Tools: Git Worktree

Before any code changes begin, create an isolated worktree to protect the main branch. 格物 applied to workspace: investigate the safest place to work before working.

### Directory Selection

Check these locations in priority order:
1. `.worktrees/` directory (if exists in project root)
2. `worktrees/` directory (if exists in project root)
3. Parent directory of the project (`../`)
4. Ask the user if none of the above are suitable

### Setup Steps

1. **Verify gitignore** — Ensure the worktree directory pattern is ignored:
   ```bash
   grep -qE '\.?worktrees/' .gitignore || echo '.worktrees/' >> .gitignore
   ```

2. **Create worktree**:
   ```bash
   git worktree add <dir>/bitfrog-<feature-name> -b feature/<feature-name>
   ```

3. **Auto-detect and run project setup**:
   ```bash
   # Detect project type and install dependencies
   [ -f package.json ] && npm install
   [ -f Cargo.toml ] && cargo build
   [ -f requirements.txt ] && pip install -r requirements.txt
   [ -f go.mod ] && go mod download
   [ -f Gemfile ] && bundle install
   ```

4. **Run baseline tests** — Confirm the worktree starts green:
   ```bash
   # Use project's test command
   [ -f package.json ] && npm test
   [ -f Cargo.toml ] && cargo test
   [ -f pytest.ini ] || [ -f setup.py ] && pytest
   [ -f go.mod ] && go test ./...
   ```

If baseline tests fail, stop and diagnose before proceeding. Never start implementation on a red baseline.

## 计划文档格式 — Plan Document Format

Every plan follows this structure:

```markdown
# [Feature] Implementation Plan

> For agentic workers: Use bitfrog:execute to implement this plan.

**Goal:** [one sentence]
**Architecture:** [2-3 sentences]

---

### Task N: [Component]

**Files:** Create/Modify/Test paths

- [ ] Step 1: Write failing test
- [ ] Step 2: Verify fail
- [ ] Step 3: Implement
- [ ] Step 4: Verify pass
- [ ] Step 5: Commit
```

## 流转 — Transition

When the plan is complete and confirmed by the user:

- Auto-invoke `bitfrog:execute` to begin implementation
- Pass the plan document path so the execute phase can track progress
- The execute phase will work through tasks sequentially (and in parallel where marked)
