# User Guide: Continual Learning System

This comprehensive guide covers everything you need to know to use the Continual Learning System effectively.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [System Overview](#2-system-overview)
3. [Getting Started](#3-getting-started)
4. [Using Skills](#4-using-skills)
5. [Running Retrospectives](#5-running-retrospectives)
6. [Understanding Learnings](#6-understanding-learnings)
7. [The Automatic Reminder System](#7-the-automatic-reminder-system)
8. [Best Practices](#8-best-practices)
9. [Simulation Tutorial](#9-simulation-tutorial)
10. [Advanced Usage](#10-advanced-usage)
11. [Frequently Asked Questions](#11-frequently-asked-questions)

---

## 1. Introduction

### What is the Continual Learning System?

The Continual Learning System is an add-on for Claude Code that enables Claude to learn from your coding sessions. Instead of starting fresh every time, Claude accumulates knowledge about what works, what doesn't, and how to approach problems in your specific context.

### Who Is This For?

- **Developers** who use Claude Code regularly
- **Teams** who want shared knowledge across members
- **Anyone** who repeatedly solves similar problems

### What You'll Gain

- Faster problem resolution (Claude remembers past solutions)
- Consistent approaches (patterns become documented)
- Reduced mistakes (failures are tracked and avoided)
- Personalized assistance (Claude learns your preferences)

---

## 2. System Overview

### Components

The system consists of four main parts:

#### 2.1 Skills

Skills are markdown files that give Claude specialized knowledge. This system includes:

| Skill | Location | Purpose |
|-------|----------|---------|
| Retrospective | `~/.claude/skills/retrospective/` | Analyzes sessions, extracts learnings |
| Coding Workflow | `~/.claude/skills/coding-workflow/` | Coding best practices |
| Infrastructure Debugging | `~/.claude/skills/infrastructure-debugging/` | Dependency/installation help |
| Failures Log | `~/.claude/skills/failures-log/` | Tracks what NOT to do |

#### 2.2 Learning Files

Each skill domain has a `learnings.md` file that accumulates session insights:

- `~/.claude/skills/coding-workflow/learnings.md`
- `~/.claude/skills/infrastructure-debugging/learnings.md`

#### 2.3 Slash Commands

Custom commands you can type:

| Command | Purpose |
|---------|---------|
| `/retrospective` | Manually trigger learning capture |

#### 2.4 Hooks

Automated triggers:

| Hook | Trigger | Action |
|------|---------|--------|
| Stop Hook | When session ends | Reminds to run retrospective |

### How Data Flows

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Your Coding    │ ──▶ │  /retrospective  │ ──▶ │  learnings.md   │
│    Session      │     │     command      │     │     files       │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                                                         │
                                                         ▼
                        ┌──────────────────┐     ┌─────────────────┐
                        │  Future Session  │ ◀── │  Claude reads   │
                        │   (you benefit)  │     │   learnings     │
                        └──────────────────┘     └─────────────────┘
```

---

## 3. Getting Started

### 3.1 Prerequisites

Before using this system, you need:

1. **Claude Code installed**: The CLI tool from Anthropic
2. **A terminal application**: Any of these work:
   - Windows: Command Prompt, PowerShell, Git Bash, Windows Terminal
   - macOS: Terminal, iTerm2
   - Linux: Any terminal emulator

### 3.2 Installation Verification

The system should already be installed. Verify by checking for the skills directory:

**On Windows (Command Prompt):**
```cmd
dir %USERPROFILE%\.claude\skills
```

**On Windows (Git Bash / PowerShell):**
```bash
ls ~/.claude/skills/
```

**On macOS / Linux:**
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

### 3.3 Starting Claude Code

Open your terminal and type:

```bash
claude
```

Claude Code will start and load the skills automatically.

### 3.4 Verifying Skills Are Loaded

Once Claude Code is running, type:

```
What skills are available?
```

Claude should list the installed skills.

---

## 4. Using Skills

### 4.1 How Skills Work

Skills are **automatically applied** by Claude when relevant. You don't need to explicitly call them.

**Example:**
```
You: How do I debug this Python error?

Claude: [Automatically applies coding-workflow skill]
        Let me walk through the debugging process...
        1. Reproduce the issue first...
        2. Check error messages...
        [etc.]
```

### 4.2 Skill Domains

#### Coding Workflow Skill

Covers:
- Code review approaches
- Debugging strategies
- Refactoring guidelines
- Common pitfalls

**Triggered when you discuss:**
- Bugs, errors, debugging
- Code review, refactoring
- Best practices, patterns

#### Infrastructure Debugging Skill

Covers:
- Package manager issues (npm, pip, etc.)
- Dependency resolution
- Environment setup
- Platform-specific problems

**Triggered when you discuss:**
- Installation errors
- Dependency conflicts
- Environment variables
- OS-specific issues

### 4.3 Viewing Skill Content

To see what's in a skill:

```
Show me the coding-workflow skill
```

or

```
What does the infrastructure-debugging skill contain?
```

---

## 5. Running Retrospectives

### 5.1 What is a Retrospective?

A retrospective is a review of your current session to extract learnings. Claude analyzes the conversation and saves valuable insights for future use.

### 5.2 When to Run a Retrospective

Run `/retrospective` after:

- Solving a tricky bug
- Discovering a new approach
- Making a mistake you want to avoid
- Learning something non-obvious
- Completing a significant task

### 5.3 How to Run a Retrospective

Type the command:

```
/retrospective
```

### 5.4 What Happens During a Retrospective

1. **Analysis**: Claude reviews the entire conversation
2. **Extraction**: Identifies successes, failures, and insights
3. **Categorization**: Sorts learnings by domain
4. **Deduplication**: Checks for existing similar learnings
5. **Storage**: Appends new learnings to appropriate files
6. **Summary**: Reports what was captured

### 5.5 Example Retrospective Output

```
Claude: Analyzing session for learnings...

Session Analysis:
- Duration: ~15 minutes
- Topics: npm dependency resolution, React 18 compatibility
- Problems solved: 1
- Failures encountered: 0

Captured Learnings:
1. [Infrastructure] npm ERESOLVE resolution using package.json overrides

Updated files:
- ~/.claude/skills/infrastructure-debugging/learnings.md (1 new entry)

Summary: This session's insight about using overrides instead of
--legacy-peer-deps for React 18 compatibility has been preserved
for future sessions.
```

---

## 6. Understanding Learnings

### 6.1 Learning Entry Format

Each learning is stored in a structured format:

```markdown
## [YYYY-MM-DD] - Brief Title

**Context**: What was being done when this was learned
**Outcome**: Success | Failure | Partial
**Insight**: The key takeaway or lesson
**Solution** (if applicable): What resolved the issue
**Tags**: #relevant #tags
```

### 6.2 Example Learning Entry

```markdown
## [2024-12-30] - Missing await causing undefined returns

**Context**: Debugging async function that intermittently returned undefined
**Outcome**: Success
**Insight**: When async functions return undefined sometimes, the most
common cause is a missing 'await' keyword before an async call
**Solution**: Add await before fetch() and other async operations
**Tags**: #javascript #async #await #debugging
```

### 6.3 Viewing Your Learnings

Ask Claude:

```
Show me my coding workflow learnings
```

or

```
What have I learned about infrastructure debugging?
```

### 6.4 Learning Categories

| Category | File | Contains |
|----------|------|----------|
| Coding | `coding-workflow/learnings.md` | Bug fixes, patterns, best practices |
| Infrastructure | `infrastructure-debugging/learnings.md` | Dependencies, installations, environments |
| Failures | `failures-log/failures.md` | What NOT to do |

### 6.5 Real-Life Example: A Captured Learning

This is an actual learning captured during a real session while creating this documentation:

**The Session:**
- User was updating README.md with reference documentation links
- Initial implementation used a table with raw URLs
- User requested URLs be embedded in titles for better readability

**The Retrospective:**
```
User: /retrospective

Claude: **Session Analysis**
Reviewing this session...

**Topics covered:**
- Updating documentation with credits/acknowledgements
- Formatting fix: embedding URLs in titles instead of raw URLs

**Learnings identified:**
1. **Coding Workflow**: Documentation best practice - always embed URLs
   in descriptive titles using markdown `[Title](URL)` format rather
   than displaying raw URLs in tables or lists.

---

**Retrospective Complete**

| Metric | Value |
|--------|-------|
| Learnings captured | 1 |
| Files updated | `~/.claude/skills/coding-workflow/learnings.md` |
```

**The Resulting Learning Entry** (in `coding-workflow/learnings.md`):

```markdown
## [2025-12-30] - Embed URLs in markdown titles for better readability

**Context**: Updating README.md with reference documentation links
**Outcome**: Success
**Insight**: When adding URLs to documentation (especially credits, references,
or resource sections), always use markdown embedded link format
`[Descriptive Title](URL)` rather than displaying raw URLs in tables or inline.
This improves readability in both rendered and raw markdown views.
**Solution**: Changed from table with raw URLs to bullet list with embedded links:
- Before: `| Skills Documentation | https://code.claude.com/docs/en/skills |`
- After: `- [Skills Documentation](https://code.claude.com/docs/en/skills)`
**Tags**: #documentation #markdown #best-practices #readability
```

**Why This Matters:**
In future sessions, when working on documentation, Claude will remember this best practice and automatically use embedded links instead of raw URLs.

---

## 7. The Automatic Reminder System

### 7.1 How It Works

When you try to end a session that contained substantive work, Claude will remind you to capture learnings.

### 7.2 What Triggers the Reminder

The reminder appears when:
- The session included problem-solving
- No retrospective has been run yet
- You indicate you're done (e.g., "Thanks, that's all")

### 7.3 Example Reminder

```
You: Thanks, I'm done for now!

Claude: Before ending, consider running /retrospective to capture
learnings from this session. Key topics: async debugging, error handling.

If you prefer to skip, just say 'stop' again.
```

### 7.4 Responding to the Reminder

**To capture learnings:**
```
/retrospective
```

**To skip and end:**
```
stop
```

---

## 8. Best Practices

### 8.1 Daily Workflow

1. **Start normally**: Begin your Claude Code session as usual
2. **Work on problems**: Debug, code, ask questions
3. **Run retrospective**: Before ending, type `/retrospective`
4. **Review weekly**: Check your learnings.md files periodically

### 8.2 What to Capture

**Good candidates for capture:**
- Solutions to problems that took time
- Approaches that didn't work (failures)
- Non-obvious configurations
- Platform-specific gotchas
- Personal preferences and conventions

**Skip capturing:**
- Basic Q&A (what is X?)
- Trivial fixes (typos)
- One-off tasks unlikely to recur

### 8.3 Keeping Learnings Clean

Periodically review and clean up:

```
Show me my coding learnings

[Review the list]

Please remove the learning about [outdated topic] - it's no longer relevant
```

### 8.4 Team Usage

If using in a team:
1. Keep skills in a shared repository
2. Use project-level skills (`.claude/skills/`) instead of user-level
3. Review learnings in team retrospectives
4. Merge valuable personal learnings into shared skills

---

## 9. Simulation Tutorial

This section provides a complete walkthrough of a realistic session demonstrating all features.

### Scenario: Debugging a Node.js Dependency Issue

#### Part 1: The Problem Occurs

Imagine you start a session and encounter this error:

```
$ npm install
npm ERR! ERESOLVE unable to resolve dependency tree
npm ERR! Could not resolve dependency:
npm ERR! peer react@"^17.0.0" from some-package@2.0.0
npm ERR! node_modules/some-package
npm ERR!   some-package@"^2.0.0" from the root project
```

You ask Claude for help. Claude automatically detects this matches the **infrastructure-debugging** skill and applies its knowledge:

```
Claude: I see an ERESOLVE peer dependency conflict. Based on my
infrastructure-debugging knowledge, here are approaches in order
of preference:

1. First, check if you can update React: npm ls react
2. Try: npm install --legacy-peer-deps
3. Or update the conflicting package to a compatible version

Let me check your package.json...
```

#### Part 2: Solving the Problem

After some back-and-forth, you discover that:
- The issue was a mismatch between React 18 and an older package
- The solution was to add an override in package.json

```json
{
  "overrides": {
    "some-package": {
      "react": "^18.0.0"
    }
  }
}
```

#### Part 3: Running the Retrospective

Now you want to capture this learning. You type:

```
/retrospective
```

Claude responds:

**Step 1 - Analyze the session:**
```
Claude: Analyzing session for learnings...

Found:
- 1 infrastructure issue (npm ERESOLVE error)
- 1 successful resolution (package.json overrides)
- 1 insight (React 18 compatibility with legacy packages)
```

**Step 2 - Check existing learnings:**
```
Claude: Reading existing learnings to avoid duplicates...
- ~/.claude/skills/infrastructure-debugging/learnings.md: 0 entries
- ~/.claude/skills/coding-workflow/learnings.md: 0 entries
```

**Step 3 - Update the files:**

Claude appends to `~/.claude/skills/infrastructure-debugging/learnings.md`:

```markdown
## [2024-12-30] - npm ERESOLVE with React 18 peer dependency

**Context**: Installing packages in a React 18 project with legacy dependencies
**Outcome**: Success
**Insight**: When packages have peer deps on React 17 but you're using React 18,
use `overrides` in package.json instead of --legacy-peer-deps for a cleaner solution
**Solution**:
```json
{
  "overrides": {
    "package-name": {
      "react": "^18.0.0"
    }
  }
}
```
**Tags**: #npm #react #peer-dependencies #eresolve
```

**Step 4 - Report to user:**
```
Claude: Retrospective complete!

Captured 1 learning:
- Infrastructure: npm ERESOLVE resolution with React 18 overrides

Updated files:
- ~/.claude/skills/infrastructure-debugging/learnings.md

This insight will be available in future sessions when similar issues occur.
```

#### Part 4: Future Session Benefits

In a **future session**, when you encounter a similar error:

```
$ npm install
npm ERR! ERESOLVE unable to resolve dependency tree
```

Claude now has your accumulated knowledge:

```
Claude: I see an ERESOLVE error. Checking my learnings...

From a previous session (2024-12-30), you resolved a similar React 18
peer dependency issue using package.json overrides instead of
--legacy-peer-deps.

Would you like me to check if that approach applies here?
```

#### Part 5: The Automatic Stop Hook

When you try to end a substantive session without running retrospective:

```
You: Thanks, that's all for now.

[Stop hook triggers]

Claude: Before ending, consider running /retrospective to capture
learnings from this session. Key topics: dependency resolution,
React compatibility.

If you prefer to skip, just say 'stop' again.
```

---

## 10. Advanced Usage

### 10.1 Creating Custom Skill Domains

Add skills for your specific needs:

```
Create a new skill called "database-patterns" for SQL best practices
```

### 10.2 Editing Skills Directly

Skills are just markdown files. You can edit them:

**Location:** `~/.claude/skills/[skill-name]/SKILL.md`

**Format:**
```markdown
---
name: skill-name
description: When Claude should use this skill
---

# Skill Title

Instructions and content here...
```

### 10.3 Importing Team Learnings

If a teammate has useful learnings:

1. Get their `learnings.md` file
2. Append entries to your own file
3. Or create a shared project-level skill

### 10.4 Backing Up Learnings

Periodically backup your learnings:

```bash
# Copy to a safe location
cp -r ~/.claude/skills/ ~/backup/claude-skills-$(date +%Y%m%d)/
```

---

## 11. Frequently Asked Questions

### General Questions

**Q: Does Claude send my learnings to Anthropic?**
A: No. Learnings are stored locally in files on your computer.

**Q: Will learnings work across different computers?**
A: Not automatically. Copy the `~/.claude/skills/` directory to sync.

**Q: Can I use this with a team?**
A: Yes! Use project-level skills (`.claude/skills/` in your repo) that get committed to git.

### Usage Questions

**Q: How often should I run retrospective?**
A: After any session with problem-solving. The reminder system helps.

**Q: What if I forget to run retrospective?**
A: The Stop hook reminds you. If you skip it, the learning is lost.

**Q: Can I capture learnings from past sessions?**
A: No. Retrospective only works on the current session in context.

### Technical Questions

**Q: Where are the files stored?**
A: In `~/.claude/skills/` (your home directory).

**Q: Can I edit learnings.md manually?**
A: Yes, they're plain markdown files.

**Q: How do I remove a learning?**
A: Edit the learnings.md file and delete the entry.

### Troubleshooting Questions

**Q: Skills aren't loading. What do I do?**
A: Restart Claude Code. Skills load at startup.

**Q: Retrospective says "no learnings found".**
A: The session may have been informational rather than problem-solving.

**Q: The reminder is annoying. Can I disable it?**
A: Edit `~/.claude/settings.json` and remove the Stop hook section.

---

## Next Steps

- [Skill Mechanics](SKILL_MECHANICS.md) - Deep dive into how skills work under the hood
- [Developer Guide](DEVELOPER_GUIDE.md) - Learn to modify and extend the system
- [Architecture](ARCHITECTURE.md) - Understand how it works internally
- [Troubleshooting](TROUBLESHOOTING.md) - Solutions to common problems
