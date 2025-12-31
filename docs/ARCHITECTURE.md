# Architecture: Continual Learning System

This document explains the design, architecture, and internal workings of the Continual Learning System. It's intended for developers who want to deeply understand how the system works.

---

## Table of Contents

1. [Design Philosophy](#1-design-philosophy)
2. [System Architecture](#2-system-architecture)
3. [Component Details](#3-component-details)
4. [Data Flow](#4-data-flow)
5. [Storage Design](#5-storage-design)
6. [Integration with Claude Code](#6-integration-with-claude-code)
7. [Design Decisions and Rationale](#7-design-decisions-and-rationale)
8. [Limitations and Constraints](#8-limitations-and-constraints)
9. [Future Extensibility](#9-future-extensibility)

---

## 1. Design Philosophy

### Core Principles

The system was designed with these principles in mind:

#### 1.1 Simplicity Over Complexity

- **No code compilation**: Everything is plain text
- **No external dependencies**: Uses only Claude Code's built-in features
- **No databases**: Flat files for storage
- **No servers**: Runs entirely locally

#### 1.2 Transparency

- **Human-readable files**: All files are markdown or JSON
- **Editable by hand**: Users can modify any file directly
- **Inspectable**: Users can see exactly what's stored

#### 1.3 Progressive Disclosure

- **Minimal initial load**: Only skill names and descriptions load at startup
- **On-demand content**: Full skill content loads when needed
- **Referenced files**: Learnings load only when relevant

#### 1.4 Separation of Concerns

- **Domain separation**: Different skills for different topics
- **Data separation**: Skills vs learnings vs failures
- **Config separation**: Hooks in settings.json, not in skills

---

## 2. System Architecture

### High-Level Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLAUDE CODE RUNTIME                             │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                         SKILL SYSTEM                                  │   │
│  │                                                                       │   │
│  │   ┌─────────────┐  ┌─────────────────┐  ┌───────────────────────┐   │   │
│  │   │ retrospective│  │ coding-workflow │  │ infrastructure-debug │   │   │
│  │   │   SKILL.md  │  │    SKILL.md     │  │      SKILL.md        │   │   │
│  │   └─────────────┘  │  learnings.md   │  │    learnings.md      │   │   │
│  │                    └─────────────────┘  └───────────────────────┘   │   │
│  │                                                                       │   │
│  │   ┌─────────────────────────────────────────────────────────────┐   │   │
│  │   │                    failures-log                              │   │   │
│  │   │                     failures.md                              │   │   │
│  │   └─────────────────────────────────────────────────────────────┘   │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌──────────────────────────┐  ┌──────────────────────────────────────┐    │
│  │      COMMAND SYSTEM      │  │            HOOK SYSTEM               │    │
│  │                          │  │                                      │    │
│  │  /retrospective.md       │  │  Stop Hook (prompt-based)            │    │
│  │                          │  │  [in settings.json]                  │    │
│  └──────────────────────────┘  └──────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       │ reads/writes
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              FILE SYSTEM                                     │
│                                                                              │
│  ~/.claude/                                                                  │
│  ├── settings.json              # Configuration and hooks                    │
│  ├── commands/                  # Slash commands                             │
│  │   └── retrospective.md                                                   │
│  └── skills/                    # All skills                                 │
│      ├── retrospective/                                                     │
│      ├── coding-workflow/                                                   │
│      ├── infrastructure-debugging/                                          │
│      └── failures-log/                                                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Component Relationships

```
                    ┌─────────────────────┐
                    │   User Interaction  │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
    ┌─────────────────┐  ┌──────────┐  ┌──────────────┐
    │  Direct Request │  │ /command │  │  Stop Hook   │
    │ (skill matching)│  │          │  │  (automated) │
    └────────┬────────┘  └────┬─────┘  └──────┬───────┘
             │                │               │
             └────────────────┼───────────────┘
                              │
                              ▼
                    ┌─────────────────────┐
                    │   Skill Activation  │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
    ┌──────────────┐  ┌───────────────┐  ┌─────────────┐
    │   SKILL.md   │  │ learnings.md  │  │ failures.md │
    │   (loaded)   │  │  (read/write) │  │(read/write) │
    └──────────────┘  └───────────────┘  └─────────────┘
```

---

## 3. Component Details

### 3.1 Skills

#### Structure of a Skill

```
skill-name/
├── SKILL.md           # Required: Main skill file
├── learnings.md       # Optional: Accumulated learnings
└── [other files]      # Optional: Referenced resources
```

#### SKILL.md Anatomy

```
┌─────────────────────────────────────────────────────────────┐
│  ---                                                         │ ◄─ YAML frontmatter
│  name: skill-name                                            │    start marker
│  description: When to use this skill                         │ ◄─ Metadata
│  ---                                                         │ ◄─ YAML frontmatter
│                                                              │    end marker
│  # Skill Title                                               │
│                                                              │ ◄─ Markdown body
│  [Instructions and content...]                               │    (loaded on
│                                                              │     activation)
│  See [learnings.md](learnings.md) for accumulated knowledge  │ ◄─ Reference to
│                                                              │    other files
└─────────────────────────────────────────────────────────────┘
```

#### Skill Loading Phases

| Phase | What Loads | When |
|-------|------------|------|
| Phase 1 | Name + Description only | At Claude Code startup |
| Phase 2 | Full SKILL.md content | When skill is activated |
| Phase 3 | Referenced files | When explicitly needed |

### 3.2 The Retrospective Skill

This is the core skill that orchestrates learning capture.

#### Responsibilities

1. **Session Analysis**: Review conversation history
2. **Learning Extraction**: Identify successes, failures, insights
3. **Categorization**: Route learnings to appropriate domains
4. **Deduplication**: Check for existing similar learnings
5. **Persistence**: Write to learnings.md files
6. **Reporting**: Summarize what was captured

#### Interaction with Other Skills

```
                 ┌──────────────────────┐
                 │   retrospective      │
                 │      SKILL.md        │
                 └──────────┬───────────┘
                            │
           ┌────────────────┼────────────────┐
           │ categorizes to │ categorizes to │
           ▼                ▼                ▼
   ┌───────────────┐ ┌────────────────┐ ┌────────────────┐
   │coding-workflow│ │infrastructure- │ │  failures-log  │
   │ learnings.md  │ │ debugging      │ │   failures.md  │
   └───────────────┘ │ learnings.md   │ └────────────────┘
                     └────────────────┘
```

### 3.3 Domain Skills

#### coding-workflow

- **Purpose**: Coding best practices
- **Triggers**: Debugging, refactoring, code review discussions
- **Contains**: SKILL.md (static) + learnings.md (dynamic)

#### infrastructure-debugging

- **Purpose**: Dependency and environment issues
- **Triggers**: npm/pip errors, installation problems, platform issues
- **Contains**: SKILL.md (static) + learnings.md (dynamic)

#### failures-log

- **Purpose**: Cross-domain failure tracking
- **Triggers**: Retrospective identifies significant failures
- **Contains**: failures.md (dynamic only)

### 3.4 Hooks

#### Stop Hook Configuration

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "[Prompt that evaluates if retrospective is needed]",
            "timeout": 15
          }
        ]
      }
    ]
  }
}
```

#### Hook Execution Flow

```
Claude finishes responding
           │
           ▼
┌─────────────────────┐
│  Stop hook fires    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  LLM evaluates      │
│  session context    │
└──────────┬──────────┘
           │
     ┌─────┴─────┐
     │           │
     ▼           ▼
Substantive   Trivial
session       session
     │           │
     ▼           ▼
Block stop,   Allow stop
suggest       (no action)
retrospective
```

### 3.5 Slash Commands

#### /retrospective

- **File**: `~/.claude/commands/retrospective.md`
- **Effect**: Triggers the retrospective skill manually
- **Flow**:
  1. User types `/retrospective`
  2. Claude loads command content
  3. Command invokes retrospective behavior
  4. Learnings are captured

---

## 4. Data Flow

### Learning Capture Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         LEARNING CAPTURE FLOW                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. SESSION ACTIVITY                                                         │
│     ┌─────────────────────────────────────────────────────────────────┐     │
│     │  User: "Help me fix this npm error"                              │     │
│     │  Claude: [Applies infrastructure-debugging skill]               │     │
│     │  Claude: "Try npm install --legacy-peer-deps"                   │     │
│     │  User: "That worked!"                                           │     │
│     └─────────────────────────────────────────────────────────────────┘     │
│                                     │                                        │
│                                     ▼                                        │
│  2. RETROSPECTIVE TRIGGERED                                                  │
│     ┌─────────────────────────────────────────────────────────────────┐     │
│     │  /retrospective  OR  Stop hook reminder                         │     │
│     └─────────────────────────────────────────────────────────────────┘     │
│                                     │                                        │
│                                     ▼                                        │
│  3. SESSION ANALYSIS                                                         │
│     ┌─────────────────────────────────────────────────────────────────┐     │
│     │  Claude reviews entire conversation                             │     │
│     │  Identifies: 1 problem solved, 0 failures, 1 insight           │     │
│     └─────────────────────────────────────────────────────────────────┘     │
│                                     │                                        │
│                                     ▼                                        │
│  4. CATEGORIZATION                                                           │
│     ┌─────────────────────────────────────────────────────────────────┐     │
│     │  npm error → infrastructure-debugging domain                    │     │
│     └─────────────────────────────────────────────────────────────────┘     │
│                                     │                                        │
│                                     ▼                                        │
│  5. DEDUPLICATION CHECK                                                      │
│     ┌─────────────────────────────────────────────────────────────────┐     │
│     │  Read: infrastructure-debugging/learnings.md                    │     │
│     │  Check: Similar entry exists? No → proceed                      │     │
│     └─────────────────────────────────────────────────────────────────┘     │
│                                     │                                        │
│                                     ▼                                        │
│  6. PERSISTENCE                                                              │
│     ┌─────────────────────────────────────────────────────────────────┐     │
│     │  Append to: infrastructure-debugging/learnings.md               │     │
│     │  Format: Date, Context, Outcome, Insight, Solution, Tags       │     │
│     └─────────────────────────────────────────────────────────────────┘     │
│                                     │                                        │
│                                     ▼                                        │
│  7. CONFIRMATION                                                             │
│     ┌─────────────────────────────────────────────────────────────────┐     │
│     │  Claude: "Captured 1 learning. Updated learnings.md"           │     │
│     └─────────────────────────────────────────────────────────────────┘     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Knowledge Retrieval Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         KNOWLEDGE RETRIEVAL FLOW                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. NEW SESSION STARTS                                                       │
│     ┌─────────────────────────────────────────────────────────────────┐     │
│     │  Claude Code loads skill names and descriptions                 │     │
│     │  (Full content not yet loaded)                                  │     │
│     └─────────────────────────────────────────────────────────────────┘     │
│                                     │                                        │
│                                     ▼                                        │
│  2. USER MAKES REQUEST                                                       │
│     ┌─────────────────────────────────────────────────────────────────┐     │
│     │  User: "I have an npm ERESOLVE error"                           │     │
│     └─────────────────────────────────────────────────────────────────┘     │
│                                     │                                        │
│                                     ▼                                        │
│  3. SKILL MATCHING                                                           │
│     ┌─────────────────────────────────────────────────────────────────┐     │
│     │  "npm ERESOLVE error" matches infrastructure-debugging          │     │
│     │  description: "dependencies, npm, installation"                 │     │
│     └─────────────────────────────────────────────────────────────────┘     │
│                                     │                                        │
│                                     ▼                                        │
│  4. SKILL ACTIVATION                                                         │
│     ┌─────────────────────────────────────────────────────────────────┐     │
│     │  Full SKILL.md content loaded into context                      │     │
│     └─────────────────────────────────────────────────────────────────┘     │
│                                     │                                        │
│                                     ▼                                        │
│  5. LEARNINGS REFERENCE                                                      │
│     ┌─────────────────────────────────────────────────────────────────┐     │
│     │  Claude reads: infrastructure-debugging/learnings.md            │     │
│     │  Finds: Previous npm ERESOLVE solution                         │     │
│     └─────────────────────────────────────────────────────────────────┘     │
│                                     │                                        │
│                                     ▼                                        │
│  6. RESPONSE WITH CONTEXT                                                    │
│     ┌─────────────────────────────────────────────────────────────────┐     │
│     │  Claude: "From a previous session, you solved this using       │     │
│     │           package.json overrides. Let me check if that         │     │
│     │           applies here..."                                     │     │
│     └─────────────────────────────────────────────────────────────────┘     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Storage Design

### File Format Choices

| File Type | Format | Rationale |
|-----------|--------|-----------|
| Skills | Markdown + YAML | Human-readable, Claude-native format |
| Learnings | Markdown | Easy to read, append, edit |
| Configuration | JSON | Standard config format, parseable |
| Commands | Markdown | Simple, no special syntax needed |

### Learning Entry Schema

```markdown
## [YYYY-MM-DD] - Title

**Context**: What was happening (string)
**Outcome**: Success | Failure | Partial (enum)
**Insight**: Key lesson learned (string)
**Solution**: How it was resolved (string, optional)
**Tags**: #tag1 #tag2 (hashtag-prefixed strings)
```

### Why Not a Database?

1. **Simplicity**: No setup, no server, no dependencies
2. **Transparency**: Users can open files in any editor
3. **Portability**: Copy files to backup or share
4. **Claude-native**: Claude reads markdown naturally
5. **Version control**: Can track changes with git

---

## 6. Integration with Claude Code

### Claude Code Features Used

| Feature | How We Use It |
|---------|---------------|
| Skills | Core of the learning system |
| Slash Commands | Manual retrospective trigger |
| Hooks | Automated retrospective reminder |
| File Tools | Reading/writing learnings |
| Settings | Hook configuration |

### What Claude Code Provides

1. **Skill Loading**: Automatic discovery and loading
2. **Context Management**: Progressive disclosure
3. **File Access**: Read and write capabilities
4. **Hook System**: Event-driven automation
5. **Command System**: Custom slash commands

### What We Build On Top

1. **Learning Capture Logic**: Retrospective skill instructions
2. **Categorization System**: Domain-based learnings
3. **Persistence Format**: Structured markdown entries
4. **Reminder System**: Stop hook configuration

---

## 7. Design Decisions and Rationale

### Decision 1: User-Level Skills (Not Project-Level)

**Choice**: Skills in `~/.claude/skills/` (user-level)

**Rationale**:
- Learnings apply across all projects
- Personal knowledge accumulates globally
- No duplication across project folders

**Alternative Considered**: Project-level (`.claude/skills/`)
- Would require per-project learning
- Useful for team scenarios
- Can be added as additional layer

### Decision 2: Prompt-Based Stop Hook (Not Command)

**Choice**: Use `type: "prompt"` for the Stop hook

**Rationale**:
- Allows intelligent evaluation of session content
- Can decide if retrospective is actually useful
- Avoids false positives on trivial sessions

**Alternative Considered**: Command-based hook
- Would need external script
- More complex to implement
- Less context-aware

### Decision 3: Separate Learnings Files (Not In SKILL.md)

**Choice**: `learnings.md` separate from `SKILL.md`

**Rationale**:
- SKILL.md stays constant (instructions)
- learnings.md grows over time (data)
- Easier to backup/reset learnings
- Cleaner separation of concerns

**Alternative Considered**: Learnings in SKILL.md
- Would make skills very long
- Harder to maintain
- Mixes instructions with data

### Decision 4: Markdown Format (Not JSON/YAML)

**Choice**: Store learnings as markdown

**Rationale**:
- Human-readable and editable
- Claude writes markdown naturally
- Easy to append new entries
- Can include formatted content

**Alternative Considered**: JSON or YAML
- Harder for Claude to append correctly
- Less readable for humans
- Overkill for this use case

---

## 8. Limitations and Constraints

### Current Limitations

| Limitation | Description | Possible Future Solution |
|------------|-------------|--------------------------|
| Session-only | Can't capture from past sessions | External logging integration |
| Manual categorization | Claude decides domains | User confirmation step |
| No search | Can't search across learnings | Index/search skill |
| No versioning | No history of learning changes | Git integration |
| Single user | Not designed for team sync | Shared repository approach |

### Claude Code Constraints

| Constraint | Impact | Workaround |
|------------|--------|------------|
| Context limits | Large learnings files may be truncated | Keep entries concise |
| Skill activation | Requires user approval | Accept skill prompts |
| Hook latency | Prompt hooks add ~15 sec | Set reasonable timeout |
| File paths | Must be absolute or relative to config | Use `~` expansion |

### Performance Considerations

- **Startup**: More skills = slower startup (but marginal)
- **Context**: Large learnings.md = more tokens used
- **Hooks**: Prompt hooks require LLM call

---

## 9. Future Extensibility

### Potential Enhancements

#### 9.1 Team Sharing

```
┌─────────────────────────────────────────────────────────────┐
│  Individual User          │         Team Repository         │
│  ~/.claude/skills/        │    (git-based)                  │
│                           │                                  │
│  personal-learnings.md ◄──┼── team-learnings.md             │
│                           │                                  │
│  [Merge workflow]         │    [PR-based updates]           │
└─────────────────────────────────────────────────────────────┘
```

#### 9.2 Learning Search

Add a skill that can search across all learnings:

```
/search-learnings "ERESOLVE"
```

#### 9.3 Learning Analytics

Track patterns in learnings:
- Most common tags
- Failure vs success ratio
- Topics over time

#### 9.4 Export/Import

Move learnings between systems:
- Export to JSON
- Import from other users
- Backup to cloud

### Extension Points

| Extension Point | How to Use |
|-----------------|------------|
| New domains | Create new skill directories |
| New triggers | Add hooks for different events |
| New commands | Add markdown files in commands/ |
| New storage | Modify retrospective skill to write elsewhere |

---

## Architecture Diagram Summary

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          CONTINUAL LEARNING SYSTEM                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   USER INPUT                                                                 │
│       │                                                                      │
│       ├─────────────────────────┬────────────────────────────────┐          │
│       │                         │                                │          │
│       ▼                         ▼                                ▼          │
│   ┌────────────┐         ┌─────────────┐                 ┌──────────────┐   │
│   │  Natural   │         │   Slash     │                 │    Hooks     │   │
│   │  Language  │         │  Commands   │                 │  (automated) │   │
│   │  (skills)  │         │  /retro...  │                 │              │   │
│   └─────┬──────┘         └──────┬──────┘                 └───────┬──────┘   │
│         │                       │                                │          │
│         └───────────────────────┴────────────────────────────────┘          │
│                                 │                                            │
│                                 ▼                                            │
│                    ┌─────────────────────────┐                              │
│                    │   RETROSPECTIVE SKILL   │                              │
│                    │   (orchestrates all)    │                              │
│                    └────────────┬────────────┘                              │
│                                 │                                            │
│           ┌─────────────────────┼─────────────────────┐                     │
│           │                     │                     │                     │
│           ▼                     ▼                     ▼                     │
│   ┌───────────────┐    ┌────────────────┐    ┌───────────────┐             │
│   │coding-workflow│    │infrastructure- │    │ failures-log  │             │
│   │               │    │debugging       │    │               │             │
│   │ • SKILL.md    │    │ • SKILL.md     │    │ • failures.md │             │
│   │ • learnings.md│    │ • learnings.md │    │               │             │
│   └───────────────┘    └────────────────┘    └───────────────┘             │
│                                                                              │
│   STORAGE: ~/.claude/skills/                                                │
│   CONFIG:  ~/.claude/settings.json                                          │
│   COMMANDS:~/.claude/commands/                                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```
