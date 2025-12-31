# Claude Code Continual Learning System

## Project Overview

This project implements a **Continual Learning System** for Claude Code using Anthropic's Skills framework. It allows Claude to learn from each coding session and accumulate knowledge over time, creating a persistent memory that improves future interactions.

## What This Project Does

1. **Captures Learnings**: Automatically extracts insights from coding sessions
2. **Persists Knowledge**: Stores learnings in markdown files that Claude reads in future sessions
3. **Categorizes by Domain**: Separates coding workflow learnings from infrastructure/debugging learnings
4. **Tracks Failures**: Maintains a failure log to avoid repeating mistakes

## Key Components

### Skills (located at `~/.claude/skills/`)

| Skill | Purpose |
|-------|---------|
| `retrospective/` | Analyzes sessions and updates learning files |
| `coding-workflow/` | Best practices for coding, debugging, refactoring |
| `infrastructure-debugging/` | Dependency and installation troubleshooting |
| `failures-log/` | Cross-domain failure documentation |

### Slash Commands (located at `~/.claude/commands/`)

| Command | Purpose |
|---------|---------|
| `/retrospective` | Manually trigger learning capture |

### Hooks (configured in `~/.claude/settings.json`)

| Hook | Purpose |
|------|---------|
| `Stop` | Reminds user to run retrospective before ending substantive sessions |

## Project Structure

```
claude_code_continual_learning_developers_digest/
├── CLAUDE.md                  # This file (project context for Claude Code)
├── README.md                  # Main documentation entry point
├── settings.example.json      # Example hook configuration
├── skills/                    # Skills to install at ~/.claude/skills/
│   ├── retrospective/
│   │   └── SKILL.md           # Core retrospective skill
│   ├── coding-workflow/
│   │   ├── SKILL.md           # Coding best practices
│   │   └── learnings.md       # Accumulated coding learnings
│   ├── infrastructure-debugging/
│   │   ├── SKILL.md           # Infrastructure troubleshooting
│   │   └── learnings.md       # Accumulated infra learnings
│   └── failures-log/
│       └── failures.md        # Failure documentation
├── commands/                  # Commands to install at ~/.claude/commands/
│   └── retrospective.md       # /retrospective slash command
└── docs/
    ├── QUICKSTART.md          # Quick start with 10 use cases
    ├── USER_GUIDE.md          # Detailed user guide
    ├── SKILL_MECHANICS.md     # How skills work under the hood
    ├── DEVELOPER_GUIDE.md     # Developer documentation
    ├── ARCHITECTURE.md        # System architecture
    └── TROUBLESHOOTING.md     # Common issues and solutions
```

### Installation Target Structure

After installation, files are copied to:

```
~/.claude/
├── skills/
│   ├── retrospective/
│   ├── coding-workflow/
│   ├── infrastructure-debugging/
│   └── failures-log/
├── commands/
│   └── retrospective.md
└── settings.json              # Merge hooks from settings.example.json
```

## How to Use

### For Users
1. Work normally with Claude Code on any project
2. When you solve problems or learn something, run `/retrospective`
3. Claude will automatically suggest retrospective when ending substantive sessions
4. Future sessions benefit from accumulated learnings

### For Developers
1. Read `docs/DEVELOPER_GUIDE.md` for complete development guide
2. Skills are markdown files - edit them directly to modify behavior
3. Add new skills by creating new directories under `~/.claude/skills/`
4. Test changes by restarting Claude Code

## Important Files to Know

- **SKILL.md**: Required file in each skill directory. Contains YAML frontmatter (name, description) and markdown instructions.
- **learnings.md**: Accumulated learnings from sessions. Updated by the retrospective skill.
- **failures.md**: Cross-domain failure log. Helps avoid repeating mistakes.
- **settings.json**: Hook configurations. Handles automated retrospective reminders.

## Conventions

- Learnings use date-prefixed entries: `## [YYYY-MM-DD] - Title`
- Tags use hashtags: `#category #subcategory`
- Skills follow Anthropic's naming convention: lowercase with hyphens

## Related Documentation

- [Quick Start Guide](docs/QUICKSTART.md) - Get started in 5 minutes
- [User Guide](docs/USER_GUIDE.md) - Complete user documentation
- [Skill Mechanics](docs/SKILL_MECHANICS.md) - How skills work under the hood
- [Developer Guide](docs/DEVELOPER_GUIDE.md) - For future development
- [Architecture](docs/ARCHITECTURE.md) - System design and flow
- [Troubleshooting](docs/TROUBLESHOOTING.md) - Common issues and solutions

## Credits & Acknowledgements

### Inspiration

This project was inspired by:
- **[Agent Experts: Finally Agents That ACTUALLY Learn](https://www.youtube.com/watch?v=zTcDwqopvKE)** by IndyDevDan
- **[Continual Learning in Claude Code](https://www.youtube.com/watch?v=sWbsD-cP4rI)** by Developers Digest

### Implementation

All code and documentation generated by **Claude Code** with **Anthropic's Claude Opus 4.5** model.

### Official Anthropic References

- [Skills Documentation](https://code.claude.com/docs/en/skills)
- [Skills Repository](https://github.com/anthropics/skills/tree/main/skills)
- [Claude Code Docs Map](https://code.claude.com/docs/en/claude_code_docs_map.md)
