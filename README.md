# agent-config

My agent configuration for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) and Codex.

## Quick start

```bash
git clone https://github.com/jcanton/agent-config.git
cd agent-config
./install.sh
```

Usually cloned by the dotfiles' `bootstrap.sh`, which also installs git hooks that re-run `install.sh --force` after every pull. `install.sh` removes links to items that are no longer in the repo.

## What's included

### Settings
- `settings.json` - Global permissions and preferences
- `statusline.sh` - Custom statusline showing token usage

### Skills
Reusable capabilities that your coding agents can invoke.

| Skill | Description |
|-------|-------------|
| `cscs-ci` | CSCS CI pipelines: status, logs, triggering (extends [msimberg/agent-skills](https://github.com/msimberg/agent-skills)) |
| `deslop` | Remove AI-generated code slop |
| `design-principles` | Review taste distilled from real review comments ([msimberg/agent-skills](https://github.com/msimberg/agent-skills)) |
| `find-skills` | Discover installable skills ([vercel-labs/skills](https://github.com/vercel-labs/skills)) |
| `grill-me`, `grilling` | Interview to stress-test a plan ([mattpocock/skills](https://github.com/mattpocock/skills)) |
| `reclaude` | Refactor CLAUDE.md files for progressive disclosure |
| `working-principles` | Design principles and a red-flag checklist for diffs (copied from gt4py_knowledge) |

## Managing your config

```bash
# See what's synced vs local-only
./sync.sh

# Preview what install would do
./install.sh --dry-run

# Add a local skill to the repo
./sync.sh add skill my-skill
./sync.sh push

# Pull changes on another machine
./sync.sh pull

# Remove a skill from repo (keeps local copy)
./sync.sh remove skill my-skill
./sync.sh push
```

### Safe operations with backups

All destructive operations create timestamped backups:

```bash
# List available backups
./sync.sh backups

# Restore from last backup
./sync.sh undo
```

### Validate skills

```bash
./sync.sh validate
```

Skills must have a `SKILL.md` with frontmatter containing `name` and `description`.

## Testing

Tests use [Bats](https://github.com/bats-core/bats-core) (Bash Automated Testing System).

```bash
# Install bats (one-time)
brew install bats-core

# Run all tests
bats tests/

# Run specific test file
bats tests/install.bats
bats tests/sync.bats
bats tests/validation.bats
```

Tests run in isolated temp directories and don't affect your actual `~/.claude` config.
Tests also cover Codex skills syncing in `~/.codex/skills`.

## Local-only config

Not everything needs to be synced. The install script only creates symlinks for what's in this repo - it won't delete your local-only skills.

Machine-specific permissions accumulate in `~/.claude/settings.local.json` (auto-created by Claude, not synced).
Codex skills are also linked from this repo into `~/.codex/skills`.

## Creating your own

Fork this repo and customize! The structure is simple:

```
agent-config/
├── settings.json      # Claude Code settings
├── statusline.sh      # Optional statusline script
├── skills/            # Skills (subdirectories with SKILL.md)
├── agents/            # Subagent definitions
├── rules/             # Rule files
└── tests/             # Bats tests
```

## See also

- [Claude Code docs](https://docs.anthropic.com/en/docs/claude-code)
- [My dotfiles](https://github.com/brianlovin/dotfiles) - Shell, git, SSH config
