# Installing BitFrog Plugin for Codex

Enable BitFrog's philosophy-driven development skills in Codex via native skill discovery.

## Prerequisites

- Git

## Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/rainyulei/bitfrog-plugin.git ~/.codex/bitfrog
   ```

2. **Create the skills symlink:**
   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/bitfrog/skills ~/.agents/skills/bitfrog
   ```

   **Windows (PowerShell):**
   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
   cmd /c mklink /J "$env:USERPROFILE\.agents\skills\bitfrog" "$env:USERPROFILE\.codex\bitfrog\skills"
   ```

3. **Restart Codex** to discover the skills.

## Verify

```bash
ls -la ~/.agents/skills/bitfrog
```

You should see a symlink pointing to your BitFrog skills directory.

## Tool Mapping

When skills reference Claude Code tools, Codex equivalents apply:
- `TodoWrite` → `todowrite`
- `Skill` → native `skill` tool
- File operations → your native tools

## Updating

```bash
cd ~/.codex/bitfrog && git pull
```

Skills update instantly through the symlink.

## Uninstalling

```bash
rm ~/.agents/skills/bitfrog
rm -rf ~/.codex/bitfrog
```
