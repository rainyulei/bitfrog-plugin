# BitFrog Plugin 🐸

> 蛙鸣万物，道法自然
> One entry, many paths — philosophy guides, tools serve.

Chinese philosophy-driven development workflows for **Claude Code**, **Codex**, and **OpenCode**.

## What is BitFrog?

BitFrog is not a rule engine. It doesn't enforce "you MUST write tests first" — it cultivates understanding. An agent that truly comprehends _why_ tests matter writes them naturally.

Rooted in Chinese classical wisdom, BitFrog guides AI agents through structured development workflows using five philosophical principles:

| Principle | Applied To |
|-----------|------------|
| 格物致知 Ge Wu Zhi Zhi — Investigate to understand essence | Brainstorming, Planning |
| 知行合一 Zhi Xing He Yi — Unity of knowledge and action | Execution, Verification |
| 辩证论治 Bian Zheng Lun Zhi — Diagnose before prescribing | Debugging |
| 阴阳互生 Yin Yang Hu Sheng — Complementary collaboration | Parallel execution |
| 三省吾身 San Sheng Wu Shen — Three levels of reflection | Code review |

## How It Works

You interact with **one brain** — BitFrog. It automatically assesses your intent and routes to the right workflow:

```
User message
    │
    ▼
┌─────────┐
│ BitFrog  │  ← Dialectical Triage (辩证分诊)
│ (Brain)  │
└────┬─────┘
     │
     ├── New idea        → brainstorm → plan → execute → review → finish
     ├── Bug/error       → debug
     ├── Want to learn   → mentor
     ├── Need a plan     → plan
     ├── Ready to code   → execute
     └── Check quality   → review
```

You never need to think about "which skill to use." Just describe your need.

## Installation

### Claude Code

```bash
claude plugin add bitfrog-plugin
```

Or from this repository:

```bash
claude plugin add /path/to/bitfrog-plugin
```

### Codex

```bash
git clone https://github.com/rainyulei/bitfrog-plugin.git ~/.codex/bitfrog
mkdir -p ~/.agents/skills
ln -s ~/.codex/bitfrog/skills ~/.agents/skills/bitfrog
```

### OpenCode

Add to your `opencode.json`:

```json
{
  "plugin": ["bitfrog@git+https://github.com/rainyulei/bitfrog-plugin.git"]
}
```

## Slash Commands

Optional shortcuts — the brain handles routing automatically, but you can also be explicit:

| Command | Workflow |
|---------|----------|
| `/brainstorm` | Explore ideas — 格物致知 |
| `/plan` | Create implementation plan — 格物→致知 |
| `/execute-plan` | Execute with TDD — 知行合一 |
| `/debug` | Diagnose issues — 辩证论治 |
| `/review` | Three-level reflection — 三省吾身 |
| `/mentor` | Guided learning — 不愤不启 |

## Workflows

### Brainstorm (格物致知)
Probes root causes, challenges assumptions, proposes 2-3 approaches, writes spec documents.

### Plan (格物→致知)
Maps dependencies, discovers patterns from git history, decomposes into bite-sized TDD tasks, creates isolated git worktrees.

### Execute (知行合一)
TDD cycle (RED→GREEN→REFACTOR), parallel subagent dispatch for independent tasks, verification-before-completion.

### Debug (辩证论治)
Four Diagnostic Methods from Traditional Chinese Medicine: 望(Observe), 闻(Listen), 问(Inquire), 切(Examine). Classifies issues as surface/internal/deep.

### Review (三省吾身)
Three reflections: self (spec compliance), peer (code-reviewer subagent), final (user value). Includes branch completion and cleanup.

### Mentor (不愤不启)
5-level guided learning from hints to near-solutions. Never gives answers directly.

## Credits

BitFrog Plugin builds upon:
- [BitFrog Copilot](https://github.com/rainyulei/bitfrog-copilot) — the original VS Code extension with Chinese philosophy-driven agents
- [Superpowers](https://github.com/obra/superpowers) by Jesse Vincent — the skill architecture and workflow framework

## License

MIT
