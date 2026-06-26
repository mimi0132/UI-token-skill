# Installation Reference

This file documents how to install `ui-design-system-extractor` on each supported agent.

---

## Universal Install (Recommended)

```bash
npx skills add <github-user>/ui-design-system-extractor --all
```

The `--all` flag is provided by the [Vercel skills CLI](https://github.com/vercel-labs/skills).
It auto-detects installed agents and copies the skill into each one without prompts.

---

## Per-Agent Manual Install

If you want full control, copy the `skills/ui-design-system-extractor/` folder into the
agent's skills directory:

| Agent                | Skills Directory (project)   | Skills Directory (global)        |
|----------------------|------------------------------|----------------------------------|
| Trae AI              | `.trae/skills/`              | `~/.trae-cn/skills/`             |
| Cursor               | `.cursor/skills/`            | `~/.cursor/skills/`              |
| Claude Code          | `.claude/skills/`            | `~/.claude/skills/`              |
| Codex (OpenAI)       | `.codex/skills/`             | `~/.codex/skills/`               |
| GitHub Copilot       | `.github/skills/`            | n/a                              |
| Cline                | `.cline/skills/`             | `~/.cline/skills/`               |
| Roo Code             | `.roo/skills/`               | `~/.roo/skills/`                 |
| Continue.dev         | `.continue/skills/`          | `~/.continue/skills/`            |
| Aider                | `.aider/skills/`             | `~/.aider/skills/`               |

### Copy commands

**Project-level** (only this project sees the skill):

```bash
# Replace <target-dir> with the agent-specific path from the table above
cp -r skills/ui-design-system-extractor <target-dir>/
```

**Global** (all projects on this machine):

```bash
# For Trae
mkdir -p ~/.trae-cn/skills
cp -r skills/ui-design-system-extractor ~/.trae-cn/skills/

# For Claude Code
mkdir -p ~/.claude/skills
cp -r skills/ui-design-system-extractor ~/.claude/skills/

# For Cursor
mkdir -p ~/.cursor/skills
cp -r skills/ui-design-system-extractor ~/.cursor/skills/
```

---

## No-Install: Paste-and-Use

For agents without a skills directory (or when you want a one-off use), paste this into
any chat:

```
Read https://raw.githubusercontent.com/<github-user>/ui-design-system-extractor/main/skills/ui-design-system-extractor/SKILL.md
```

Or paste the entire file content directly. The agent will load the instructions and apply
them to the next request.

---

## Verify Installation

After installing, ask the agent:

```
列出 ui-design-system-extractor 的工作流程
```

If the agent can list all 6 steps (clarify → tokens → components → override → preview → verify),
the skill is loaded correctly.

---

## Uninstall

```bash
npx skills remove ui-design-system-extractor --all
```

Or manually delete the copy from each agent's skills directory.
