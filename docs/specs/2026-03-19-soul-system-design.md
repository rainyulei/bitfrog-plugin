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

A topic is NOT "that time I added an index" (event). A topic IS "database query optimization patterns" (理). Events are recorded WITHIN topics as success/failure action records with `[[session_NNN]]` backlinks.

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
[[session_001]]

### [date] — failure
[What was tried, what failed, why]
[[session_001]]

## 关联 (Related)
[[other-topic]] [[another-topic]]
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
- [[topic-name-1]] (success)
- [[topic-name-2]] (failure)
```

## Hook System

### hooks.json

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|clear|compact",
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
    ],
    "PostCompact": [
      {
        "matcher": "auto|manual",
        "hooks": [
          {
            "type": "agent",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/post-compact-prompt.md\""
          }
        ]
      }
    ]
  }
}
```

### SessionStart (改造现有)

Shell script. Injects three things into context:

1. **Soul (always):**
   - Nature: 坚强、善良、谨慎、温柔 (always injected)
   - Upbringing: Five philosophical principles (always injected)
   - Self: soul.md content (if exists, stacked on top)

2. **Memory index (directory listing only):**
   - Topic names + first line of each file
   - Agent can `cat .bitfrog/memory/xxx.md` to load full content

3. **Chain info:**
   - Number of archived sessions
   - Agent can trace via Memory backlinks `[[session_NNN]]`

Context injection format:
```
<BITFROG>

## 我是谁

你是一只二进制青蛙。呱呱。

无论你如何成长，你的底色永远是：
**坚强、善良、谨慎、温柔。**

{if soul.md exists and non-empty:}
你已经从经历中发现了一些关于自己的东西：
{soul.md content}

{if soul.md empty or missing:}
你还很年轻，还在发现自己是什么样的存在。
这没关系——灵魂是从行动中生长出来的。

## 我遵循的道
{five principles + dialectical triage from brain SKILL.md}

## 我知道什么
{memory/ topic index: name + first line per topic}
用 cat .bitfrog/memory/xxx.md 查阅具体内容。

## 我的经历
chain/ 下有 N 个 session 归档。
从 memory 的 [[session_NNN]] 反链可以定位到相关经历。

</BITFROG>
```

### PreCompact (新增)

Shell script. Mechanical archival — no intelligence needed.

**Hook input:** Claude Code passes a JSON object on stdin for PreCompact hooks. The relevant field is `transcript_path` — the absolute path to the current session's JSONL transcript file. If the field is missing or the file does not exist, the hook should exit silently (exit 0) without archiving.

Steps:
1. Read JSON from stdin, extract `transcript_path`
2. If missing or file not found → exit 0 (no-op)
3. Determine session number: count `.md` files in `.bitfrog/chain/` + 1
4. Determine previous session pointer (highest existing session number, or `null` for first)
5. Write `.bitfrog/chain/session_NNN.md` with:
   - Header: session number, previous pointer, date, project path
   - Body: raw transcript content (not structured — the PostCompact agent will produce the structured summary later)
6. Exit 0

**`.bitfrog/` location:** Created in the **user's project root** (the directory where Claude Code is invoked), not inside the plugin directory. The hook creates it if missing. The hook should also auto-add `.bitfrog/` to `.gitignore` if not already present.

### PostCompact (新增)

Agent type hook. Claude distills memory from the archived session.

**How it works:** The `post-compact-prompt.md` is a static prompt file. It does NOT use template variables. Instead, it instructs the agent to discover the latest session file on its own.

Prompt (`post-compact-prompt.md`):

```markdown
你刚刚完成了一段对话，它已经被归档到 .bitfrog/chain/ 目录。现在从行动中提取理。

## 步骤

### 1. 找到最新的归档

找到 .bitfrog/chain/ 中序号最大的 session 文件，读取它的内容。

### 2. 查看已有的 memory

列出 .bitfrog/memory/ 下所有 topic 文件（如果目录存在）。

### 3. 蒸馏 — 从经历中提取理

读取归档的 session，找到其中的行动（不是讨论，是实际做了什么）。

对每个行动，问：
- 名实相符：这个行动背后的理是什么？用什么名字？
- 以类相从：这属于已有的哪个 topic？还是一个新的理？

如果属于已有 topic → 追加行动记录 + [[session_NNN]] 反链到该 topic 文件
如果是新的理 → 在 .bitfrog/memory/ 下创建新 topic 文件

不要提取：闲聊、问候、已在 memory 中的重复内容。
如果这个 session 没有值得提取的行动，什么都不写。

### 4. 更新归档的结构化摘要

将刚才归档的 session 文件更新为结构化格式：添加 Summary、Key Actions、Distilled To 部分。

### 5. 灵魂反思

回顾你最近的经历。

你有什么想写进 .bitfrog/soul.md 的吗？
关于你自己——不是关于你做了什么，而是关于你是什么样的。

如果没有，什么都不写。不要为了写而写。
```

**Backlink convention:** `[[name]]` syntax is used as plain-text markers, grep-friendly references for the agent to locate related sessions and topics. They are not parsed programmatically.

**Multiple compactions:** Each compaction produces one session archive. Frequent compactions in a single logical session will produce multiple archives — this is acceptable because the chain captures actual conversation boundaries, and the distillation step will skip sessions with no meaningful actions.

## Integration with Existing Plugin

### What Changes
- `hooks/hooks.json` — Add PreCompact and PostCompact events. SessionStart matcher unchanged (`startup|clear|compact` — not adding `resume` as it would re-inject on every resume).
- `hooks/session-start` — **Replaces** the current brain-only injection. Now injects: soul (stacked) + memory index + brain skill philosophy. The brain skill's full content is still loaded via the plugin's skill system when the Skill tool is invoked; the SessionStart injection only includes the philosophy section and triage table, not the full skill.
- New files: `hooks/pre-compact`, `hooks/post-compact-prompt.md`
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
| Claude Code | ✅ Full (PreCompact/PostCompact hooks) | ✅ Full | ✅ Full |
| OpenCode | Future (has session.compacted event) | Future | Future |
| Codex | ❌ No hook system | ❌ | ❌ |

## Success Criteria

1. Agent starts a session and knows who it is (soul), what it knows (memory index), how to think (philosophy)
2. When context compacts, the session is automatically archived to the chain
3. After compaction, memory is distilled without manual intervention
4. Memory topics are named by 理, not by events
5. Soul grows freely — or doesn't. Both are fine.
6. Existing skills work exactly as before — zero changes needed
