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

```markdown
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

```markdown
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
