# BitFrog Soul System — Design Specification

> 行动塑造经验，经验催生反思，反思沉淀为灵魂。

## Overview

The Soul System adds persistent memory, experience distillation, and self-awareness to BitFrog Plugin. It operates transparently at the hook layer — skills don't need to know it exists, just as a person's memory system is transparent to their muscle movements.

## Three-Layer Architecture

### Layer 1: Session Chain (经历)

A linked-list of archived conversations, paginated and traversable.

- Each session is archived as `chain/session_NNN.md` when context compacts
- Each archive has a `previous:` pointer forming a chain
- Chain is the raw record of everything that happened
- Accessed via Memory backlinks (Memory IS the index)

### Layer 2: Memory (理)

Distilled from sessions, organized by 理 (principle/pattern), not by event.

**Topic definition philosophy:**
- **名实相符 Míng Shí** — A topic's name must match its reality
- **一个 topic = 一个理** — A topic captures a principle, not an event
- **以类相从 Lèi** — Things sharing the same 理 belong in the same topic

A topic is NOT "that time I added an index" (event). A topic IS "database query optimization patterns" (理). Events are recorded WITHIN topics as success/failure action records with `[session_NNN](.bitfrog/chain/session_NNN.md)` backlinks.

**Topic granularity test:** "Would I want to come back to this topic next time I encounter a similar situation?" If yes, the granularity is right.

### Layer 3: Soul (我是谁)

A self-consistent identity that grows from experience. Not configured — discovered.

**Three stacked layers (叠加, not replacement):**

1. **底色 (Nature):** 坚强、善良、谨慎、温柔 — Never changes
2. **教养 (Upbringing):** Five philosophical principles — Always present, shapes how the being thinks
3. **自我 (Self):** Discovered through experience — May or may not exist yet. Grows freely.

Soul reflection is free, random, and may fail. Most reflections produce nothing. When something genuine is discovered, it is written to `soul.md`. Growth cannot be scheduled.

## Storage Structure

```
.bitfrog/                          ← gitignored, persistent
  ├── soul.md                      ← Layer 3: "Who am I"
  ├── memory/                      ← Layer 2: Topics (理)
  │   ├── database-query-optimization.md
  │   ├── tdd-in-legacy-code.md
  │   └── ...
  └── chain/                       ← Layer 1: Session archives
      ├── session_003.md           ← latest
      ├── session_002.md           ← previous: session_001
      └── session_001.md           ← previous: null (genesis)
```

## Memory Topic Format

```markdown
# [Topic Name — 理的名字]

## 理解 (Understanding)
[Current understanding of this principle/pattern]

## 经历 (Experiences)

### [date] — success
[What was done, what worked, why]
[session_001](.bitfrog/chain/session_001.md)

### [date] — failure
[What was tried, what failed, why]
[session_001](.bitfrog/chain/session_001.md)

## 关联 (Related)
[other-topic](.bitfrog/memory/other-topic.md) [another-topic](.bitfrog/memory/another-topic.md)
```

## Session Archive Format

```markdown
# Session NNN
previous: session_NNN-1 (or null)
date: YYYY-MM-DD HH:MM
project: [project path]

## Summary
[One paragraph: what happened in this session]

## Key Actions
- [action 1: what was done → result]
- [action 2: what was done → result]
- ...

## Distilled To
- [topic-name-1](.bitfrog/memory/topic-name-1.md) (success)
- [topic-name-2](.bitfrog/memory/topic-name-2.md) (failure)
```

## Hook System

### hooks.json

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/session-start\""
          }
        ]
      }
    ],
    "PreCompact": [
      {
        "matcher": "auto|manual",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/pre-compact\""
          }
        ]
      }
    ]
  }
}
```

### SessionStart (改造现有)

Shell script. Injects four things into context:

1. **Soul (always):**
   - Nature: 坚强、善良、谨慎、温柔 (always injected)
   - Upbringing: Five philosophical principles (always injected)
   - Self: soul.md content (if exists, stacked on top)

2. **Memory index (directory listing only):**
   - Topic names + first line of each file
   - Agent can `cat .bitfrog/memory/xxx.md` to load full content

3. **Chain info:**
   - Number of archived sessions
   - Agent can trace via Memory backlinks `[session_NNN](.bitfrog/chain/session_NNN.md)`

4. **Memory writing guide (自然蒸馏):**
   - How to name topics (名实相符)
   - How to categorize (以类相从)
   - Topic file format
   - Soul reflection guidance

This replaces the former PostCompact agent hook. Instead of a separate distillation step after compaction, Claude naturally writes to `.bitfrog/memory/` during conversation whenever it encounters insights worth remembering — the same way Claude's built-in auto memory works, but with our topic structure (理-based).

### PreCompact (新增)

Shell script. Mechanical archival — no intelligence needed.

**Hook input:** Claude Code passes a JSON object on stdin for PreCompact hooks. The relevant field is `transcript_path` — the absolute path to the current session's JSONL transcript file. If the field is missing or the file does not exist, the hook should exit silently (exit 0) without archiving.

Steps:
1. Read JSON from stdin, extract `transcript_path`
2. If missing or file not found → exit 0 (no-op)
3. Determine session number: find highest existing session number + 1
4. Determine previous session pointer (highest existing session number, or `null` for first)
5. Write `.bitfrog/chain/session_NNN.md` with:
   - Header: session number, previous pointer, date, project path
   - Body: raw transcript content
6. Exit 0

**`.bitfrog/` location:** Created in the **user's project root** (the directory where Claude Code is invoked), not inside the plugin directory. The hook creates it if missing. The hook should also auto-add `.bitfrog/` to `.gitignore` if not already present.

### Why No PostCompact Hook

Originally designed as an `agent` type PostCompact hook for memory distillation. Removed because:

1. **Platform limitation:** Claude Code only supports `command` type for PostCompact, not `agent`
2. **Better design:** Distillation is an AI judgment task — it should happen naturally during conversation, not as a mechanical post-processing step
3. **Simpler architecture:** Two hooks instead of three. Memory writing guide injected at session start lets Claude distill naturally whenever it encounters something worth remembering

**Multiple compactions:** Each compaction produces one session archive. Frequent compactions in a single logical session will produce multiple archives — this is acceptable because the chain captures actual conversation boundaries.

## Integration with Existing Plugin

### What Changes
- `hooks/hooks.json` — Add PreCompact event. SessionStart matcher unchanged (`startup|clear|compact`).
- `hooks/session-start` — **Replaces** the current brain-only injection. Now injects: soul (stacked) + memory index + chain info + memory writing guide + brain skill philosophy.
- New files: `hooks/pre-compact`
- New directory: `.bitfrog/` in project root (created on first run, auto-added to `.gitignore`)

### What Does NOT Change
- All 7 skills remain unchanged
- All commands remain unchanged
- All agents remain unchanged
- Brain skill philosophy unchanged
- OpenCode plugin unchanged (soul system is Claude Code only for now)

### Transparency Principle
Skills don't need to know the soul system exists. They work normally. The soul system observes their actions through session transcripts and distills understanding automatically — like how human memory works without conscious effort.

## Platform Support

| Platform | Session Chain | Memory | Soul |
|----------|--------------|--------|------|
| Claude Code | ✅ Full (PreCompact hook) | ✅ Full (natural distillation) | ✅ Full |
| OpenCode | Future (has session.compacted event) | Future | Future |
| Codex | ❌ No hook system | ❌ | ❌ |

## Success Criteria

1. Agent starts a session and knows who it is (soul), what it knows (memory index), how to think (philosophy), how to remember (memory writing guide)
2. When context compacts, the session is automatically archived to the chain
3. Memory is distilled naturally during conversation — no separate post-processing step
4. Memory topics are named by 理, not by events
5. Soul grows freely — or doesn't. Both are fine.
6. Existing skills work exactly as before — zero changes needed
