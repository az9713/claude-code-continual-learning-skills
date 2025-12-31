# Developer Guide: Continual Learning System

This guide is for developers who want to understand, modify, or extend the Continual Learning System. It assumes you have programming experience but may be new to Claude Code's skill system.

---

## Table of Contents

1. [Understanding the Technology Stack](#1-understanding-the-technology-stack)
2. [Project Structure](#2-project-structure)
3. [How Claude Code Skills Work](#3-how-claude-code-skills-work)
4. [File Format Reference](#4-file-format-reference)
5. [Modifying Existing Skills](#5-modifying-existing-skills)
6. [Creating New Skills](#6-creating-new-skills)
7. [Working with Hooks](#7-working-with-hooks)
8. [Working with Slash Commands](#8-working-with-slash-commands)
9. [Testing Your Changes](#9-testing-your-changes)
10. [Common Development Tasks](#10-common-development-tasks)
11. [Debugging](#11-debugging)
12. [Contributing Guidelines](#12-contributing-guidelines)

---

## 1. Understanding the Technology Stack

### What Technologies Are Used?

This project is simpler than typical software projects because it uses Claude Code's built-in systems:

| Component | Technology | Description |
|-----------|------------|-------------|
| Skills | Markdown + YAML | Text files that Claude reads |
| Hooks | JSON configuration | Settings that trigger scripts |
| Commands | Markdown | Custom slash commands |
| Storage | Plain text files | Learnings stored as markdown |

### No Code Compilation Required

Unlike C, C++, or Java projects, there is:
- No compilation step
- No build system (no Makefiles, no Maven, no npm)
- No dependencies to install
- No runtime environment to configure

Everything is plain text files that Claude Code reads directly.

### Key Concepts for Traditional Programmers

If you're coming from C/C++/Java, here's a translation:

| Traditional Concept | Equivalent Here |
|---------------------|-----------------|
| Header files | SKILL.md frontmatter (YAML) |
| Implementation files | SKILL.md body (Markdown) |
| Libraries | Other skills (referenced in markdown) |
| Configuration files | settings.json |
| Log files | learnings.md, failures.md |
| Environment variables | `~/.claude/` directory location |

---

## 2. Project Structure

### Complete File Layout

```
~/.claude/                              # Claude Code configuration directory
├── settings.json                       # Main configuration (includes hooks)
├── commands/                           # Custom slash commands
│   └── retrospective.md               # /retrospective command
└── skills/                            # All skills
    ├── retrospective/                 # Core learning skill
    │   └── SKILL.md
    ├── coding-workflow/               # Coding best practices
    │   ├── SKILL.md
    │   └── learnings.md               # Accumulated learnings
    ├── infrastructure-debugging/       # Infrastructure troubleshooting
    │   ├── SKILL.md
    │   └── learnings.md               # Accumulated learnings
    └── failures-log/                  # Failure tracking
        └── failures.md                # Failure entries
```

### File Purposes

| File | Purpose | Modified By |
|------|---------|-------------|
| `settings.json` | Hook configurations | Developer (manually) |
| `retrospective.md` | Slash command definition | Developer (manually) |
| `*/SKILL.md` | Skill definitions | Developer (manually) |
| `*/learnings.md` | Accumulated learnings | Claude (via retrospective) |
| `failures.md` | Failure log | Claude (via retrospective) |

---

## 3. How Claude Code Skills Work

### The Skill Lifecycle

```
┌─────────────────────────────────────────────────────────────────────┐
│ 1. STARTUP: Claude Code loads skill names and descriptions          │
│    (Only frontmatter is loaded - keeps context small)               │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│ 2. USER INPUT: User asks a question or makes a request              │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│ 3. MATCHING: Claude compares request to skill descriptions          │
│    - Semantic matching (not just keywords)                          │
│    - Multiple skills can match                                       │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│ 4. ACTIVATION: If match found, Claude asks to load the skill        │
│    (User can approve or deny)                                       │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│ 5. EXECUTION: Full SKILL.md content is loaded into context          │
│    - Claude follows the instructions                                │
│    - May read referenced files (progressive disclosure)             │
└─────────────────────────────────────────────────────────────────────┘
```

### Progressive Disclosure

This is a key concept. Instead of loading everything upfront:

1. **Initial load**: Only skill name + description (small)
2. **On activation**: Full SKILL.md content (medium)
3. **On demand**: Referenced files like learnings.md (as needed)

This keeps Claude's context window efficient.

---

## 4. File Format Reference

### SKILL.md Format

Every skill must have a `SKILL.md` file with this structure:

```markdown
---
name: skill-name
description: A clear description of when Claude should use this skill.
---

# Skill Title

[Skill content in markdown...]
```

#### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Lowercase, hyphens allowed, no spaces. Max 64 chars. |
| `description` | Yes | What the skill does AND when to use it. Max 1024 chars. |
| `allowed-tools` | No | Restrict which tools Claude can use |
| `model` | No | Specific model to use for this skill |

#### Example with All Fields

```markdown
---
name: database-patterns
description: SQL and database best practices. Use when working with databases, writing queries, or optimizing database performance.
allowed-tools: Read, Grep, Glob
---

# Database Patterns

[Content here...]
```

### learnings.md Format

Learnings files have a simple structure:

```markdown
# [Category] Learnings

This file contains learnings captured from sessions.

---

<!-- Entries appended below -->

## [YYYY-MM-DD] - Title

**Context**: What was happening
**Outcome**: Success | Failure | Partial
**Insight**: The key lesson
**Solution**: How it was resolved (if applicable)
**Tags**: #tag1 #tag2
```

### settings.json Format

The settings file is JSON:

```json
{
  "hooks": {
    "HookEventName": [
      {
        "hooks": [
          {
            "type": "command" | "prompt",
            "command": "shell command (for type: command)",
            "prompt": "LLM prompt (for type: prompt)",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### Slash Command Format

Slash commands are markdown files in `~/.claude/commands/`:

```markdown
[Instructions for Claude when this command is invoked]

Any markdown content here will be used as the prompt.
```

The filename becomes the command name: `retrospective.md` → `/retrospective`

---

## 5. Modifying Existing Skills

### Step-by-Step: Editing a Skill

**Goal**: Add a new debugging tip to the coding-workflow skill.

**Step 1**: Open the skill file

```bash
# On Windows (Git Bash)
code ~/.claude/skills/coding-workflow/SKILL.md

# Or use any text editor
notepad ~/.claude/skills/coding-workflow/SKILL.md
```

**Step 2**: Make your changes

```markdown
## Debugging Strategy

1. Reproduce first
2. Isolate the problem
3. Read error messages carefully
4. Check recent changes
5. Verify assumptions
6. **NEW: Check log files** ← Add your new tip
```

**Step 3**: Save the file

**Step 4**: Restart Claude Code

```bash
# Exit current session
exit

# Start new session
claude
```

**Step 5**: Verify the change

```
Show me the coding-workflow skill
```

### What Can Be Modified

| Component | What to Change | Effect |
|-----------|----------------|--------|
| `description` | Trigger conditions | When skill activates |
| Body content | Instructions | How Claude behaves |
| Referenced files | Additional context | What Claude can access |

---

## 6. Creating New Skills

### Step-by-Step: Creating a New Skill

**Goal**: Create a "testing-patterns" skill for unit testing best practices.

**Step 1**: Create the directory

```bash
mkdir ~/.claude/skills/testing-patterns
```

**Step 2**: Create SKILL.md

Create `~/.claude/skills/testing-patterns/SKILL.md`:

```markdown
---
name: testing-patterns
description: Unit testing best practices and patterns. Use when writing tests, debugging test failures, or discussing testing strategies.
---

# Testing Patterns

## Core Principles

1. **Test behavior, not implementation**
2. **One assertion per test** (when reasonable)
3. **Arrange-Act-Assert pattern**
4. **Descriptive test names**

## Common Patterns

### Testing Async Code
[Content...]

### Mocking Dependencies
[Content...]

## Accumulated Learnings

See [learnings.md](learnings.md) for session-learned testing insights.
```

**Step 3**: Create learnings.md (optional but recommended)

Create `~/.claude/skills/testing-patterns/learnings.md`:

```markdown
# Testing Patterns Learnings

This file contains learnings from testing sessions.

---

<!-- Entries appended below -->
```

**Step 4**: Update retrospective skill to include new domain

Edit `~/.claude/skills/retrospective/SKILL.md` to categorize testing learnings to the new skill.

**Step 5**: Restart Claude Code and verify

```bash
exit
claude
```

Then:
```
What skills are available?
```

### Skill Creation Checklist

- [ ] Created directory under `~/.claude/skills/`
- [ ] Created `SKILL.md` with valid frontmatter
- [ ] Name is lowercase with hyphens only
- [ ] Description clearly states when to use
- [ ] Content provides useful instructions
- [ ] Created `learnings.md` for accumulated knowledge
- [ ] Updated retrospective to include new domain
- [ ] Tested by restarting Claude Code

---

## 7. Working with Hooks

### Understanding Hooks

Hooks are automated triggers that run at specific events. This system uses a `Stop` hook to remind about retrospectives.

### Hook Events Available

| Event | When It Fires |
|-------|---------------|
| `PreToolUse` | Before Claude uses a tool |
| `PostToolUse` | After Claude uses a tool |
| `Stop` | When Claude finishes responding |
| `SubagentStop` | When a subagent finishes |
| `SessionStart` | When Claude Code starts |
| `SessionEnd` | When Claude Code exits |
| `UserPromptSubmit` | When user submits a prompt |

### Current Hook Configuration

In `~/.claude/settings.json`:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Analyze the session context: $ARGUMENTS\n\nDetermine if this session contains substantive learnings...",
            "timeout": 15
          }
        ]
      }
    ]
  }
}
```

### Hook Types

| Type | Description | Use Case |
|------|-------------|----------|
| `command` | Runs a shell command | Scripts, external tools |
| `prompt` | Sends prompt to LLM | Intelligent decisions |

### Modifying the Stop Hook

**To make it less aggressive**:

Change the prompt to be more lenient about what constitutes "substantive learnings."

**To disable entirely**:

Remove the `Stop` section from `settings.json`.

**To add additional hooks**:

Add new event sections following the same pattern.

### Creating a Custom Hook

**Example**: Log all file writes

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$(date): File modified\" >> ~/.claude/file-log.txt"
          }
        ]
      }
    ]
  }
}
```

---

## 8. Working with Slash Commands

### Understanding Slash Commands

Slash commands are custom prompts triggered by `/commandname`.

### Current Commands

| Command | File | Purpose |
|---------|------|---------|
| `/retrospective` | `~/.claude/commands/retrospective.md` | Capture session learnings |

### Creating a New Command

**Goal**: Create `/review-learnings` to display accumulated learnings.

**Step 1**: Create the file

Create `~/.claude/commands/review-learnings.md`:

```markdown
Review and display all accumulated learnings from my skills.

## Instructions

1. Read these files:
   - ~/.claude/skills/coding-workflow/learnings.md
   - ~/.claude/skills/infrastructure-debugging/learnings.md
   - ~/.claude/skills/failures-log/failures.md

2. For each file:
   - Count the number of entries
   - List the titles and dates
   - Highlight any patterns across multiple entries

3. Provide a summary:
   - Total learnings captured
   - Most common tags
   - Suggestions for what might be outdated
```

**Step 2**: Restart Claude Code

**Step 3**: Test

```
/review-learnings
```

### Command Naming Rules

- Filename = command name (without .md)
- Use lowercase and hyphens
- Keep names short and descriptive
- Avoid conflicts with built-in commands

---

## 9. Testing Your Changes

### Manual Testing Workflow

```
1. Make your changes to skill/command/hook files
2. Exit Claude Code: exit
3. Start Claude Code: claude
4. Test the change:
   - For skills: Ask about the topic
   - For commands: Run /commandname
   - For hooks: Trigger the event
5. Verify expected behavior
6. If issues, check debug mode (see Debugging section)
```

### Testing Skills

```
# Test if skill is loaded
You: What skills are available?
Expected: Your skill appears in the list

# Test if skill triggers correctly
You: [Ask something matching the description]
Expected: Claude uses the skill

# Test skill content
You: Show me the [skill-name] skill content
Expected: See your changes
```

### Testing Commands

```
# Run the command
You: /your-command-name

Expected: Claude executes the command instructions
```

### Testing Hooks

Hooks are trickier to test because they're event-driven:

1. For `Stop` hooks: Try ending a session
2. For `PostToolUse` hooks: Have Claude use the relevant tool
3. Check for expected behavior or output

---

## 10. Common Development Tasks

### Task: Add a New Learning Domain

1. Create skill directory: `mkdir ~/.claude/skills/new-domain/`
2. Create `SKILL.md` with frontmatter and content
3. Create `learnings.md` file
4. Edit retrospective skill to include new domain
5. Restart and test

### Task: Change When a Skill Triggers

Edit the `description` field in the skill's frontmatter:

```yaml
---
name: my-skill
description: Changed description with new trigger keywords
---
```

### Task: Merge Learnings from Another User

1. Get their `learnings.md` file
2. Open your corresponding file
3. Copy entries (avoiding duplicates)
4. Check for conflicting insights

### Task: Backup the Entire System

```bash
# Create timestamped backup
cp -r ~/.claude/ ~/claude-backup-$(date +%Y%m%d)/
```

### Task: Reset to Initial State

```bash
# Warning: This deletes all learnings!
rm ~/.claude/skills/coding-workflow/learnings.md
rm ~/.claude/skills/infrastructure-debugging/learnings.md
rm ~/.claude/skills/failures-log/failures.md

# Recreate empty files
echo "# Coding Workflow Learnings\n\n---\n" > ~/.claude/skills/coding-workflow/learnings.md
echo "# Infrastructure Debugging Learnings\n\n---\n" > ~/.claude/skills/infrastructure-debugging/learnings.md
echo "# Failures Log\n\n---\n" > ~/.claude/skills/failures-log/failures.md
```

---

## 11. Debugging

### Debug Mode

Run Claude Code with debug flag:

```bash
claude --debug
```

This shows:
- Skill loading messages
- Hook execution details
- Tool usage information

### Common Issues and Solutions

#### Skill Not Loading

**Symptoms**: Skill doesn't appear in "What skills are available?"

**Checks**:
1. File is named exactly `SKILL.md` (case-sensitive)
2. File is in `~/.claude/skills/skillname/SKILL.md`
3. YAML frontmatter is valid (starts and ends with `---`)
4. Name field uses only lowercase and hyphens

**Debug**:
```bash
claude --debug
# Look for skill loading errors
```

#### Skill Not Triggering

**Symptoms**: Skill is loaded but doesn't activate when expected

**Checks**:
1. Description matches user request semantically
2. Description is specific enough
3. Description includes trigger phrases

**Test**:
```
You: [Phrase that exactly matches description keywords]
```

#### Hook Not Firing

**Symptoms**: Expected hook behavior doesn't occur

**Checks**:
1. `settings.json` is valid JSON (use a JSON validator)
2. Hook event name is correct and properly capitalized
3. For command hooks: command is accessible from shell
4. For prompt hooks: prompt is well-formed

**Debug**:
```bash
claude --debug
# Look for hook execution messages
```

#### Learnings Not Persisting

**Symptoms**: Retrospective runs but file isn't updated

**Checks**:
1. File exists and is writable
2. Path in retrospective skill is correct
3. File permissions allow writing

**Test**:
```bash
# Check write permissions
touch ~/.claude/skills/coding-workflow/learnings.md
```

---

## 12. Contributing Guidelines

### Making Changes

1. **Document your changes**: Update relevant documentation
2. **Test thoroughly**: Verify in a real session
3. **Keep it simple**: Minimal changes that solve the problem
4. **Maintain compatibility**: Don't break existing learnings

### Code Style for Skills

- Use clear, concise language
- Structure with headers (##, ###)
- Use bullet points for lists
- Include examples where helpful
- Keep instructions actionable

### Submitting Changes

If this were a shared project:

1. Create a branch
2. Make and test changes
3. Update documentation
4. Submit for review

### Documentation Standards

When adding new features:

1. Update CLAUDE.md (project context)
2. Update README.md if user-facing
3. Update USER_GUIDE.md for usage instructions
4. Update this DEVELOPER_GUIDE.md for technical details
5. Add to TROUBLESHOOTING.md if new issues possible

---

## Quick Reference: Developer Cheatsheet

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEVELOPER QUICK REFERENCE                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SKILL LOCATION:     ~/.claude/skills/[name]/SKILL.md          │
│  COMMAND LOCATION:   ~/.claude/commands/[name].md              │
│  SETTINGS:           ~/.claude/settings.json                   │
│                                                                 │
│  RESTART REQUIRED:   Yes, after any file change                │
│  DEBUG MODE:         claude --debug                            │
│                                                                 │
│  SKILL FRONTMATTER:                                            │
│    ---                                                          │
│    name: lowercase-with-hyphens                                 │
│    description: When to use this skill                          │
│    ---                                                          │
│                                                                 │
│  HOOK EVENTS: PreToolUse, PostToolUse, Stop,                   │
│               SessionStart, SessionEnd, UserPromptSubmit        │
│                                                                 │
│  HOOK TYPES:  command (run shell), prompt (ask LLM)            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## See Also

- [Skill Mechanics](SKILL_MECHANICS.md) - Deep dive into how skills are invoked, matched, and executed
- [Architecture](ARCHITECTURE.md) - System design and data flow diagrams
- [User Guide](USER_GUIDE.md) - End-user documentation
- [Troubleshooting](TROUBLESHOOTING.md) - Solutions to common problems
