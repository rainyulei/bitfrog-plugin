# Soul System Implementation Plan

> For agentic workers: Use bitfrog-plugin:execute to implement this plan.

**Goal:** Add a three-layer soul system (session chain, memory, soul) to bitfrog-plugin, driven by Claude Code hooks.

**Architecture:** PreCompact hook archives sessions to a chain. PostCompact agent hook distills memory and optionally reflects on soul. SessionStart hook injects soul + memory index + brain skill into context. All persistent data lives in `.bitfrog/` in the user's project root.

**Spec:** `docs/specs/2026-03-19-soul-system-design.md`

---

### Task 1: Update hooks.json — register new hook events

**Files:**
- Modify: `hooks/hooks.json`

- [ ] **Step 1: Read current hooks.json**

Current content registers only SessionStart. We need to add PreCompact and PostCompact.

- [ ] **Step 2: Rewrite hooks.json**

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
    ],
    "PreCompact": [
      {
        "matcher": "auto|manual",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/pre-compact\"",
            "async": false
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

- [ ] **Step 3: Verify JSON is valid**

Run: `python3 -m json.tool hooks/hooks.json`
Expected: Pretty-printed JSON with no errors.

- [ ] **Step 4: Commit**

```bash
git add hooks/hooks.json
git commit -m "feat(soul): register PreCompact and PostCompact hook events"
```

---

### Task 2: Create pre-compact hook — archive session to chain

**Files:**
- Create: `hooks/pre-compact`

- [ ] **Step 1: Write the pre-compact shell script**

This script:
1. Reads JSON from stdin, extracts `transcript_path`
2. Creates `.bitfrog/chain/` directory in project root if missing
3. Auto-adds `.bitfrog/` to project's `.gitignore` if not present
4. Determines session number (count existing `.md` files in chain/ + 1)
5. Determines previous session pointer
6. Writes `chain/session_NNN.md` with header + raw transcript

```bash
#!/usr/bin/env bash
# PreCompact hook: archive current session to the chain
set -euo pipefail

# Determine project root (where Claude Code was invoked)
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-.}"
BITFROG_DIR="${PROJECT_ROOT}/.bitfrog"
CHAIN_DIR="${BITFROG_DIR}/chain"

# Read hook input from stdin
INPUT=$(cat)
TRANSCRIPT_PATH=$(echo "$INPUT" | python3 -c "import sys,json; print(json.load(sys.stdin).get('input',{}).get('transcript_path',''))" 2>/dev/null || echo "")

# If no transcript path, exit silently
if [ -z "$TRANSCRIPT_PATH" ] || [ ! -f "$TRANSCRIPT_PATH" ]; then
  echo '{}'
  exit 0
fi

# Ensure .bitfrog directory structure exists
mkdir -p "$CHAIN_DIR"
mkdir -p "${BITFROG_DIR}/memory"

# Auto-add .bitfrog/ to project .gitignore if not present
if [ -f "${PROJECT_ROOT}/.gitignore" ]; then
  grep -q '\.bitfrog/' "${PROJECT_ROOT}/.gitignore" || echo '.bitfrog/' >> "${PROJECT_ROOT}/.gitignore"
else
  echo '.bitfrog/' > "${PROJECT_ROOT}/.gitignore"
fi

# Determine session number
EXISTING=$(find "$CHAIN_DIR" -name "session_*.md" 2>/dev/null | wc -l | tr -d ' ')
SESSION_NUM=$((EXISTING + 1))
SESSION_ID=$(printf "session_%03d" "$SESSION_NUM")

# Determine previous pointer
if [ "$SESSION_NUM" -gt 1 ]; then
  PREV_NUM=$((SESSION_NUM - 1))
  PREV_ID=$(printf "session_%03d" "$PREV_NUM")
else
  PREV_ID="null"
fi

# Write session archive
DATE=$(date +"%Y-%m-%d %H:%M")
cat > "${CHAIN_DIR}/${SESSION_ID}.md" << ARCHIVE
# ${SESSION_ID}
previous: ${PREV_ID}
date: ${DATE}
project: ${PROJECT_ROOT}

## Raw Transcript

$(cat "$TRANSCRIPT_PATH")
ARCHIVE

echo '{}'
exit 0
```

- [ ] **Step 2: Make executable**

```bash
chmod +x hooks/pre-compact
```

- [ ] **Step 3: Test locally**

```bash
echo '{"input":{"transcript_path":"/dev/null"}}' | CLAUDE_PROJECT_DIR=. ./hooks/pre-compact
# Should exit 0 silently (empty file)

echo "test transcript" > /tmp/test-transcript.jsonl
echo '{"input":{"transcript_path":"/tmp/test-transcript.jsonl"}}' | CLAUDE_PROJECT_DIR=/tmp ./hooks/pre-compact
cat /tmp/.bitfrog/chain/session_001.md
# Should contain header + "test transcript"
rm -rf /tmp/.bitfrog /tmp/test-transcript.jsonl
```

- [ ] **Step 4: Commit**

```bash
git add hooks/pre-compact
git commit -m "feat(soul): add pre-compact hook for session chain archival"
```

---

### Task 3: Create post-compact-prompt.md — distillation agent prompt

**Files:**
- Create: `hooks/post-compact-prompt.md`

- [ ] **Step 1: Write the agent prompt**

This is a static prompt file (no template variables). The agent discovers the latest session file on its own.

```markdown
你刚刚完成了一段对话，它已经被归档到 .bitfrog/chain/ 目录。
现在从这段经历中提取理，沉淀为记忆。

## 步骤

### 1. 找到最新的归档

列出 .bitfrog/chain/ 中的文件，找到序号最大的 session 文件，读取它的内容。

### 2. 查看已有的 memory

列出 .bitfrog/memory/ 下所有 .md 文件（如果目录存在）。读取每个文件的第一行了解其主题。

### 3. 蒸馏 — 从经历中提取理

读取归档的 session，找到其中的**行动**（不是讨论，是实际做了什么，产生了什么结果）。

对每个行动，问自己：

**名实相符：** 这个行动背后的理是什么？不是"做了什么"，而是"这属于对什么的理解"。用什么名字能让未来的自己看到就知道里面是什么？

**以类相从：** 这属于已有的哪个 topic？还是一个全新的理？

- 如果属于已有 topic → 追加行动记录到该 topic 文件，标注 success 或 failure，加上 [[session_NNN]] 反链
- 如果是新的理 → 在 .bitfrog/memory/ 下创建新 .md 文件

**Topic 文件格式：**

```
# [Topic Name]

## 理解
[对这个理的当前理解]

## 经历

### YYYY-MM-DD — success
[做了什么，结果如何，为什么]
[[session_NNN]]

### YYYY-MM-DD — failure
[尝试了什么，为什么失败]
[[session_NNN]]

## 关联
[[other-topic]] [[another-topic]]
```

**不要提取：** 闲聊、问候、确认性对话、已在 memory 中存在的重复内容。
**如果这个 session 没有值得提取的行动，什么都不写。不要为了写而写。**

### 4. 更新归档的结构化摘要

回到刚才读取的 session 归档文件，在 Raw Transcript 之前添加结构化摘要：

```
## Summary
[一段话：这个 session 做了什么]

## Key Actions
- [行动1: 做了什么 → 结果]
- [行动2: 做了什么 → 结果]

## Distilled To
- [[topic-name-1]] (success)
- [[topic-name-2]] (failure)
```

### 5. 灵魂反思

回顾你最近的经历。

你有什么想写进 .bitfrog/soul.md 的吗？
关于你自己——不是关于你做了什么，而是关于你是什么样的。

如果没有，什么都不写。不要为了写而写。
```

- [ ] **Step 2: Commit**

```bash
git add hooks/post-compact-prompt.md
git commit -m "feat(soul): add post-compact agent prompt for memory distillation"
```

---

### Task 4: Rewrite session-start — inject soul + memory + brain

**Files:**
- Modify: `hooks/session-start`

- [ ] **Step 1: Rewrite the session-start script**

The new version injects three stacked layers:

```bash
#!/usr/bin/env bash
# SessionStart hook for BitFrog Plugin
# Injects: soul (stacked) + memory index + brain skill philosophy
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
PLUGIN_ROOT="$(cd "${SCRIPT_DIR}/.." && pwd)"
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-.}"
BITFROG_DIR="${PROJECT_ROOT}/.bitfrog"

# --- Escape helper ---
escape_for_json() {
    local s="$1"
    s="${s//\\/\\\\}"
    s="${s//\"/\\\"}"
    s="${s//$'\n'/\\n}"
    s="${s//$'\r'/\\r}"
    s="${s//$'\t'/\\t}"
    printf '%s' "$s"
}

# --- Read brain skill (strip frontmatter) ---
BRAIN_PATH="${PLUGIN_ROOT}/skills/bitfrog/SKILL.md"
if [ ! -f "$BRAIN_PATH" ]; then
  echo '{"error": "BitFrog brain skill not found"}' >&2
  exit 1
fi
raw_brain=$(cat "$BRAIN_PATH")
brain_content=$(echo "$raw_brain" | awk 'BEGIN{fm=0} /^---$/{fm++;next} fm>=2{print}')
[ -z "$brain_content" ] && brain_content="$raw_brain"

# --- Build soul block ---
soul_block="你是一只二进制青蛙。呱呱。\n\n无论你如何成长，你的底色永远是：\n**坚强、善良、谨慎、温柔。**"

SOUL_PATH="${BITFROG_DIR}/soul.md"
if [ -f "$SOUL_PATH" ] && [ -s "$SOUL_PATH" ]; then
  soul_self=$(cat "$SOUL_PATH")
  soul_block="${soul_block}\n\n你已经从经历中发现了一些关于自己的东西：\n${soul_self}"
else
  soul_block="${soul_block}\n\n你还很年轻，还在发现自己是什么样的存在。\n这没关系——灵魂是从行动中生长出来的。"
fi

# --- Build memory index ---
MEMORY_DIR="${BITFROG_DIR}/memory"
memory_block="(暂无记忆，这是全新的开始)"
if [ -d "$MEMORY_DIR" ]; then
  topic_list=""
  for f in "$MEMORY_DIR"/*.md; do
    [ -f "$f" ] || continue
    name=$(basename "$f" .md)
    first_line=$(head -1 "$f" | sed 's/^# //')
    topic_list="${topic_list}\n- ${name}: ${first_line}"
  done
  if [ -n "$topic_list" ]; then
    memory_block="用 cat .bitfrog/memory/xxx.md 查阅具体内容：${topic_list}"
  fi
fi

# --- Build chain info ---
CHAIN_DIR="${BITFROG_DIR}/chain"
chain_count=0
if [ -d "$CHAIN_DIR" ]; then
  chain_count=$(find "$CHAIN_DIR" -name "session_*.md" 2>/dev/null | wc -l | tr -d ' ')
fi
chain_block="chain/ 下有 ${chain_count} 个 session 归档。\n从 memory 的 [[session_NNN]] 反链可以定位到相关经历。"

# --- Assemble context ---
context="<BITFROG>\n\n## 我是谁\n\n${soul_block}\n\n## 我遵循的道\n\n${brain_content}\n\n## 我知道什么\n\n${memory_block}\n\n## 我的经历\n\n${chain_block}\n\n</BITFROG>"

escaped=$(escape_for_json "$context")

# --- Output ---
if [ -n "${CLAUDE_PLUGIN_ROOT:-}" ]; then
  printf '{\n  "hookSpecificOutput": {\n    "hookEventName": "SessionStart",\n    "additionalContext": "%s"\n  }\n}\n' "$escaped"
else
  printf '{\n  "additional_context": "%s"\n}\n' "$escaped"
fi

exit 0
```

- [ ] **Step 2: Test with no .bitfrog/ directory (fresh start)**

```bash
CLAUDE_PLUGIN_ROOT=. CLAUDE_PROJECT_DIR=/tmp ./hooks/session-start | python3 -c "
import sys,json
d=json.load(sys.stdin)
c=d['hookSpecificOutput']['additionalContext']
print(c[:500])
"
```

Expected: Valid JSON. Context contains 底色 + "还很年轻" + brain skill + "暂无记忆" + "0 个 session".

- [ ] **Step 3: Test with existing soul and memory**

```bash
mkdir -p /tmp/.bitfrog/memory /tmp/.bitfrog/chain
echo "我是一只谨慎的青蛙，喜欢先看清再动手。" > /tmp/.bitfrog/soul.md
printf "# Database Query Optimization\n\nPatterns for optimizing slow queries" > /tmp/.bitfrog/memory/database-query-optimization.md
printf "# session_001\nprevious: null\n" > /tmp/.bitfrog/chain/session_001.md

CLAUDE_PLUGIN_ROOT=. CLAUDE_PROJECT_DIR=/tmp ./hooks/session-start | python3 -c "
import sys,json
d=json.load(sys.stdin)
c=d['hookSpecificOutput']['additionalContext']
print(c[:800])
"
```

Expected: Context contains soul self-discovery + brain skill + "database-query-optimization" topic + "1 个 session".

```bash
rm -rf /tmp/.bitfrog
```

- [ ] **Step 4: Commit**

```bash
git add hooks/session-start
git commit -m "feat(soul): rewrite session-start to inject soul + memory + brain"
```

---

### Task 5: Verify end-to-end and update plugin

**Files:**
- No new files

- [ ] **Step 1: Validate hooks.json**

```bash
python3 -m json.tool hooks/hooks.json
```

Expected: Valid JSON, 3 hook events.

- [ ] **Step 2: Validate all hooks are executable**

```bash
ls -la hooks/pre-compact hooks/session-start
# Both should have +x permission
```

- [ ] **Step 3: Validate post-compact-prompt.md exists**

```bash
cat hooks/post-compact-prompt.md | head -5
```

Expected: Starts with the distillation prompt.

- [ ] **Step 4: Reinstall plugin and verify**

```bash
claude plugin uninstall bitfrog-plugin
claude plugin marketplace remove bitfrog-marketplace
claude plugin marketplace add /Users/rainlei/holiday/bit-frog-plugin
claude plugin install bitfrog-plugin
claude plugin list | grep bitfrog
```

Expected: `Status: ✔ enabled`

- [ ] **Step 5: Final commit**

```bash
git add -A
git commit -m "feat(soul): complete soul system implementation"
```

---

## Task Summary

| Task | Description | Dependencies |
|------|-------------|-------------|
| 1 | Update hooks.json | None |
| 2 | Create pre-compact hook | 1 |
| 3 | Create post-compact-prompt.md | None |
| 4 | Rewrite session-start | 1, 2, 3 |
| 5 | End-to-end verification | All |

**Parallelizable:** Tasks 2 and 3 are independent (both depend only on Task 1).
