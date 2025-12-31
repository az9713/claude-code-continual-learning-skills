# Claude Code Continual Learning System

## What Is This?

This is a **learning system for Claude Code** that makes Claude smarter over time. Every time you solve a problem, debug an issue, or discover something useful, this system captures that knowledge and stores it. In future sessions, Claude automatically applies these learnings.

Think of it like building a personal knowledge base that Claude can read and update.

---

## Why Do I Need This?

Without this system:
- Every new Claude Code session starts fresh
- You might solve the same problem multiple ways
- Insights from debugging sessions are lost
- You repeat mistakes you've already learned from

With this system:
- Knowledge accumulates across sessions
- Claude remembers what worked (and what didn't)
- Solutions compound over time
- Your workflow becomes more efficient

---

## How Does It Work? (The Simple Version)

```
┌─────────────────────────────────────────────────────────────────┐
│                    YOUR CODING SESSION                          │
│                                                                 │
│   You: "Help me fix this npm error"                            │
│   Claude: [Uses infrastructure-debugging skill]                │
│   Claude: "Try npm install --legacy-peer-deps"                 │
│   You: "That worked!"                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    RUN RETROSPECTIVE                            │
│                                                                 │
│   You: "/retrospective"                                         │
│   Claude: "Analyzing session... Found 1 learning"              │
│   Claude: [Writes to learnings.md]                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    FUTURE SESSION                               │
│                                                                 │
│   You: "I have an npm ERESOLVE error"                          │
│   Claude: "Based on previous learnings, you solved this        │
│            before using --legacy-peer-deps. Let me check       │
│            if that applies here..."                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Quick Start (5 Minutes)

### Prerequisites

- **Claude Code installed** - The CLI tool from Anthropic
- **A terminal** - Command Prompt, PowerShell, Git Bash, or any terminal

### Step 1: Install Skills from This Repository

Copy the skills and commands to your Claude Code configuration:

**On macOS/Linux/Git Bash:**
```bash
# Clone or download this repository first, then:
cp -r skills/* ~/.claude/skills/
cp -r commands/* ~/.claude/commands/

# If ~/.claude/skills/ doesn't exist, create it:
mkdir -p ~/.claude/skills ~/.claude/commands
```

**On Windows (PowerShell):**
```powershell
# Clone or download this repository first, then:
Copy-Item -Recurse skills\* $env:USERPROFILE\.claude\skills\
Copy-Item -Recurse commands\* $env:USERPROFILE\.claude\commands\
```

**Configure the Stop Hook** (optional but recommended):

Merge the contents of `settings.example.json` into your `~/.claude/settings.json`:
- If `settings.json` doesn't exist, copy `settings.example.json` as `settings.json`
- If it exists, add the `"hooks"` section from the example file

### Step 2: Verify Installation

```bash
ls ~/.claude/skills/
```

You should see:
```
coding-workflow/
failures-log/
infrastructure-debugging/
retrospective/
```

### Step 3: Restart Claude Code

Skills are loaded when Claude Code starts. If Claude Code is running, exit and restart it:

```bash
# Exit current session (type 'exit' or press Ctrl+C)
# Then start a new session
claude
```

### Step 4: Verify Skills Are Loaded

Ask Claude:

```
What skills are available?
```

You should see the retrospective, coding-workflow, and infrastructure-debugging skills listed.

### Step 5: Try Your First Retrospective

After doing some work, type:

```
/retrospective
```

Claude will analyze the session and capture any learnings.

---

## Documentation

| Document | Description | Who Should Read |
|----------|-------------|-----------------|
| [Quick Start Guide](docs/QUICKSTART.md) | 10 hands-on use cases | Everyone (start here!) |
| [User Guide](docs/USER_GUIDE.md) | Complete user documentation | Users who want full details |
| [Skill Mechanics](docs/SKILL_MECHANICS.md) | How skills work under the hood | Curious users, Developers |
| [Developer Guide](docs/DEVELOPER_GUIDE.md) | How to modify and extend | Developers |
| [Architecture](docs/ARCHITECTURE.md) | System design and internals | Developers |
| [Troubleshooting](docs/TROUBLESHOOTING.md) | Common problems and solutions | Everyone |

---

## Features at a Glance

| Feature | Description |
|---------|-------------|
| **Automatic Skill Matching** | Claude detects when skills are relevant |
| **Manual Retrospective** | Run `/retrospective` anytime |
| **Automated Reminders** | Prompted before ending substantive sessions |
| **Categorized Learning** | Coding vs Infrastructure learnings |
| **Failure Tracking** | Dedicated log for what didn't work |
| **Progressive Disclosure** | Only loads context when needed |

---

## File Locations

All files are stored in your home directory under `.claude/`:

| Path | Purpose |
|------|---------|
| `~/.claude/skills/` | All skill definitions |
| `~/.claude/commands/` | Custom slash commands |
| `~/.claude/settings.json` | Configuration and hooks |

On Windows, `~` means `C:\Users\YourUsername\`.

---

## Getting Help

1. **Check Troubleshooting**: See [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)
2. **Ask Claude**: Type your question - Claude knows about this system
3. **Read the Guides**: The documentation covers most scenarios

---

## License

This project is provided as-is for educational purposes.

---

## Credits & Acknowledgements

### Inspiration

This project was inspired by the following YouTube videos:

- **[Agent Experts: Finally Agents That ACTUALLY Learn](https://www.youtube.com/watch?v=zTcDwqopvKE)** by IndyDevDan
- **[Continual Learning in Claude Code](https://www.youtube.com/watch?v=sWbsD-cP4rI)** by Developers Digest

### Implementation

All code and documentation in this project were generated by **Claude Code** powered by **Anthropic's Claude Opus 4.5** model.

### Framework

Built on **Anthropic's Claude Code Skills system**.

---

## Reference Documentation

For deeper understanding of Claude Code Skills, refer to these official Anthropic resources:

- [Skills Documentation](https://code.claude.com/docs/en/skills)
- [Skills Repository](https://github.com/anthropics/skills/tree/main/skills)
- [Claude Code Docs Map](https://code.claude.com/docs/en/claude_code_docs_map.md)
