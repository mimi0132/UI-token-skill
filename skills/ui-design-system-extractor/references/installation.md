# Installation Reference

Two paths, pick the one that fits you.

---

## Path 1: One-line install (recommended)

```bash
npx skills add mimi0132/UI-token-skill --all
```

The [`skills` CLI](https://github.com/vercel-labs/skills) (from Vercel Labs) auto-detects
every AI agent on your machine and installs the skill. **The `--all` flag is critical**
— without it, the CLI pops up a 50+ option TUI picker asking you to choose an agent
by hand. `--all` skips the picker entirely: "don't ask, just install."

Works with Cursor, Claude Code, Codex, GitHub Copilot, Cline, Roo Code, Trae AI,
Continue.dev, Aider, and 60+ other agents.

### Variations

```bash
# Global install — all projects on this machine
npx skills add mimi0132/UI-token-skill --all -g

# Target a specific agent explicitly (when you know what you use)
npx skills add mimi0132/UI-token-skill -a claude-code -y
npx skills add mimi0132/UI-token-skill -a cursor -y
npx skills add mimi0132/UI-token-skill -a codex -y

# List what skills are in the repo first (no install)
npx skills add mimi0132/UI-token-skill --list

# Preview the install without writing anything
npx skills add mimi0132/UI-token-skill --all --dry-run
```

### Prerequisite

- **Node.js** (for the `npx` command) — [download](https://nodejs.org/)
- That's it. The CLI handles the rest.

### Uninstall

```bash
npx skills remove mimi0132/UI-token-skill --all
```

---

## Path 2: Paste-and-use (no install, no files)

For a one-off task — or when you don't want to install anything — paste this into **any
AI chat box** (Cursor, Claude.ai, ChatGPT, an IDE assistant, …):

```
Read https://github.com/mimi0132/UI-token-skill and follow its
skills/ui-design-system-extractor/SKILL.md to [your design task here].
```

The agent fetches the repo, loads `SKILL.md`, and applies the skill to your request.
Nothing is installed, nothing is left on your machine.

**Examples:**

```
Read https://github.com/mimi0132/UI-token-skill and follow its
skills/ui-design-system-extractor/SKILL.md to convert this Figma design
into a CSS theme for Element Plus: https://www.figma.com/design/xxx
```

```
Read https://github.com/mimi0132/UI-token-skill and follow its
skills/ui-design-system-extractor/SKILL.md to extract design tokens from
the attached screenshot and generate a theme override for Ant Design.
```

---

## Advanced: Manual install per agent

If `npx skills` doesn't detect your agent, copy `skills/ui-design-system-extractor/`
into the agent's skills folder manually:

| Agent            | Project folder        | Global folder         |
|------------------|-----------------------|-----------------------|
| Trae AI          | `.trae/skills/`       | `~/.trae-cn/skills/`  |
| Cursor           | `.cursor/skills/`     | `~/.cursor/skills/`   |
| Claude Code      | `.claude/skills/`     | `~/.claude/skills/`   |
| Codex (OpenAI)   | `.codex/skills/`      | `~/.codex/skills/`    |
| GitHub Copilot   | `.github/skills/`     | n/a                   |
| Cline            | `.cline/skills/`      | `~/.cline/skills/`    |
| Roo Code         | `.roo/skills/`        | `~/.roo/skills/`      |
| Continue.dev     | `.continue/skills/`   | `~/.continue/skills/` |
| Aider            | `.aider/skills/`      | `~/.aider/skills/`    |

```bash
# Project-level (this project only)
cp -r skills/ui-design-system-extractor <project-folder-from-table>/

# Global (every project)
mkdir -p ~/.claude/skills    # for Claude Code, for example
cp -r skills/ui-design-system-extractor ~/.claude/skills/
```

---

## Verify the install

After installing, ask the agent:

```
列出 ui-design-system-extractor 的工作流程
```

If the agent can list all the phases (clarify → discover lib → extract tokens → derive
gaps → override → preview → verify), the skill loaded correctly.

---

## Troubleshooting

### A TUI picker pops up asking you to choose an agent

You're running the install **without `--all`**. The default behavior is to show a 50+
option picker. Fix: add `--all` to your command:

```bash
npx skills add mimi0132/UI-token-skill --all
```

Or pick one explicitly:

```bash
npx skills add mimi0132/UI-token-skill -a claude-code -y
```

Or skip the install entirely with the paste-and-use path (above).

### `npx skills` says "agent not detected"

Your agent isn't in the CLI's auto-detect list yet. Use Path 2 (paste-and-use) or the
manual install table above.

### `npx skills` reports "skill not found in repo"

Make sure the skill folder is named exactly `ui-design-system-extractor` and contains
a `SKILL.md` at its root. Run `npx skills add mimi0132/UI-token-skill --list` to see
what the CLI sees.

### The agent loads the skill but doesn't use it

Some agents require explicit activation. Try:
- **Claude Code**: type `/ui-design-system-extractor`
- **Cursor / Codex**: mention `$ui-design-system-extractor` in chat
- **Generic**: start your request with "Use the ui-design-system-extractor skill to..."

### Permission denied on the install

If `cp -r` fails on a global folder, prefix with `sudo` or use Path 1 (the `npx` CLI
handles permissions for you).

### Want to fork it?

The repo is yours to clone, rename, and re-publish:

```bash
git clone https://github.com/mimi0132/UI-token-skill.git
# Edit SKILL.md, references/*.md, and README.md to match your team
# Push to your own repo
git remote set-url origin <your-repo-url>
git push -u origin main
```

Then share `npx skills add <your-org>/<your-repo>` with your team.
