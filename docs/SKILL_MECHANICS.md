# Skill Mechanics: The Inner Plumbing of Claude Code Skills

This document provides a detailed look at exactly how Claude Code skills work under the hood. It explains the what, when, and how of skill invocation so you understand the complete picture.

---

## Table of Contents

1. [Skill Invocation Overview](#1-skill-invocation-overview)
2. [The Skill Lifecycle in Detail](#2-the-skill-lifecycle-in-detail)
3. [Phase 1: Discovery (Startup)](#3-phase-1-discovery-startup)
4. [Phase 2: Matching (Request Analysis)](#4-phase-2-matching-request-analysis)
5. [Phase 3: Activation (Loading)](#5-phase-3-activation-loading)
6. [Phase 4: Execution (Using the Skill)](#6-phase-4-execution-using-the-skill)
7. [Phase 5: Progressive Disclosure (Loading References)](#7-phase-5-progressive-disclosure-loading-references)
8. [Our Skills: Detailed Invocation Analysis](#8-our-skills-detailed-invocation-analysis)
9. [Skill Invocation Debugging](#9-skill-invocation-debugging)
10. [Common Invocation Patterns](#10-common-invocation-patterns)

---

## 1. Skill Invocation Overview

### What Is Skill Invocation?

Skill invocation is the process by which Claude Code:
1. **Discovers** available skills at startup
2. **Matches** user requests to relevant skills
3. **Activates** matched skills by loading their content
4. **Executes** the skill's instructions
5. **Discloses** additional referenced content as needed

### The Key Insight

Skills are **model-invoked**, not user-invoked. This means:
- You don't type `/skill-name` to use a skill
- Claude automatically detects when a skill is relevant
- Claude asks for permission before loading the full skill
- The skill's `description` field is the trigger mechanism

### Visual Overview

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                         SKILL INVOCATION TIMELINE                            │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  STARTUP                    USER INPUT                   RESPONSE            │
│     │                           │                           │               │
│     ▼                           ▼                           ▼               │
│  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐      │
│  │DISCO-│  │MATCH-│  │MATCH │  │ACTI- │  │EXECU-│  │DISCLO│  │RESP- │      │
│  │VERY  │──│ING   │──│FOUND │──│VATION│──│TION  │──│SURE  │──│ONSE  │      │
│  │      │  │      │  │      │  │      │  │      │  │      │  │      │      │
│  └──────┘  └──────┘  └──────┘  └──────┘  └──────┘  └──────┘  └──────┘      │
│     │         │         │         │         │         │         │           │
│     │         │         │         │         │         │         │           │
│  Load      Compare   Skill     Load      Follow    Load      Generate      │
│  names &   request   matches   full      skill     files     response      │
│  descrip-  to skill  user's    SKILL.md  instruc-  referen-  using         │
│  tions     descrip-  intent    content   tions     ced in    skill         │
│  only      tions               into                skill     knowledge     │
│                                context                                      │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. The Skill Lifecycle in Detail

### Complete Lifecycle Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                        ┌─────────────────────┐                              │
│                        │   CLAUDE CODE       │                              │
│                        │   STARTS            │                              │
│                        └──────────┬──────────┘                              │
│                                   │                                         │
│                                   ▼                                         │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │ PHASE 1: DISCOVERY                                                     │ │
│  │                                                                        │ │
│  │ For each directory in ~/.claude/skills/:                              │ │
│  │   1. Check if SKILL.md exists                                         │ │
│  │   2. Parse YAML frontmatter only                                      │ │
│  │   3. Extract: name, description                                       │ │
│  │   4. Store in skill registry                                          │ │
│  │                                                                        │ │
│  │ Result: Skill registry with names + descriptions (lightweight)        │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                   │                                         │
│                                   ▼                                         │
│                        ┌─────────────────────┐                              │
│                        │   WAITING FOR       │◄─────────────────┐          │
│                        │   USER INPUT        │                  │          │
│                        └──────────┬──────────┘                  │          │
│                                   │                             │          │
│                                   ▼                             │          │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │ PHASE 2: MATCHING                                                      │ │
│  │                                                                        │ │
│  │ When user sends a message:                                            │ │
│  │   1. Analyze user's intent                                            │ │
│  │   2. Compare intent to each skill's description                       │ │
│  │   3. Use semantic matching (not just keywords)                        │ │
│  │   4. Rank potential matches by relevance                              │ │
│  │                                                                        │ │
│  │ Decision: Match found? → Proceed to activation                        │ │
│  │           No match?    → Respond without skills                       │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                   │                                         │
│                        ┌──────────┴──────────┐                             │
│                        │                     │                             │
│                   Match Found           No Match                           │
│                        │                     │                             │
│                        ▼                     ▼                             │
│  ┌────────────────────────────────┐  ┌────────────────────────┐           │
│  │ PHASE 3: ACTIVATION            │  │ Normal response        │           │
│  │                                │  │ (no skill used)        │           │
│  │ 1. Ask user for permission     │  └────────────────────────┘           │
│  │ 2. If approved:                │                                       │
│  │    - Load full SKILL.md        │                                       │
│  │    - Add to current context    │                                       │
│  │ 3. If denied:                  │                                       │
│  │    - Continue without skill    │                                       │
│  └───────────────┬────────────────┘                                       │
│                  │                                                         │
│                  ▼                                                         │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │ PHASE 4: EXECUTION                                                     │ │
│  │                                                                        │ │
│  │ Claude now has skill instructions in context:                         │ │
│  │   1. Follow the skill's instructions                                  │ │
│  │   2. Apply the skill's patterns and knowledge                        │ │
│  │   3. Use any tools the skill specifies                               │ │
│  │                                                                        │ │
│  └───────────────┬────────────────────────────────────────────────────────┘ │
│                  │                                                         │
│                  ▼                                                         │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │ PHASE 5: PROGRESSIVE DISCLOSURE                                        │ │
│  │                                                                        │ │
│  │ If skill references other files (e.g., learnings.md):                 │ │
│  │   1. Claude reads the referenced file                                 │ │
│  │   2. Content is loaded into context                                   │ │
│  │   3. Claude uses this additional knowledge                            │ │
│  │                                                                        │ │
│  │ This happens on-demand, not upfront                                   │ │
│  └───────────────┬────────────────────────────────────────────────────────┘ │
│                  │                                                         │
│                  ▼                                                         │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │ RESPONSE GENERATED                                                     │ │
│  │                                                                        │ │
│  │ Claude responds using:                                                │ │
│  │   - Skill instructions                                                │ │
│  │   - Disclosed reference content                                       │ │
│  │   - General knowledge                                                 │ │
│  └───────────────┬────────────────────────────────────────────────────────┘ │
│                  │                                                         │
│                  └──────────────────────────────────────────────┐          │
│                                                                 │          │
│                                                       Back to waiting      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Phase 1: Discovery (Startup)

### What Happens at Startup

When Claude Code starts, it scans for skills:

```
STARTUP SEQUENCE
================

1. Locate skill directories:
   ~/.claude/skills/           ← User skills
   .claude/skills/             ← Project skills (if in a project)
   [plugin directories]        ← Plugin skills

2. For EACH directory found:

   ~/.claude/skills/retrospective/
   │
   ├── Check: Does SKILL.md exist?
   │   │
   │   ├── YES → Parse frontmatter
   │   │         Extract: name = "retrospective"
   │   │         Extract: description = "Captures learnings..."
   │   │         Add to registry
   │   │
   │   └── NO  → Skip this directory
   │
   └── (Do NOT load the full content yet)

3. Build skill registry:

   ┌─────────────────────────────────────────────────────────────┐
   │ SKILL REGISTRY (in memory)                                  │
   ├─────────────────────────────────────────────────────────────┤
   │ Name                      │ Description                     │
   ├───────────────────────────┼─────────────────────────────────┤
   │ retrospective             │ "Captures learnings from..."    │
   │ coding-workflow           │ "Best practices for coding..."  │
   │ infrastructure-debugging  │ "Troubleshooting dependencies.."│
   └───────────────────────────┴─────────────────────────────────┘
```

### What Gets Loaded (And What Doesn't)

| Loaded at Startup | NOT Loaded at Startup |
|-------------------|----------------------|
| Skill names | Full SKILL.md body |
| Skill descriptions | Referenced files (learnings.md) |
| Skill paths | Scripts or assets |

### Why This Design?

**Token Efficiency**: Loading only names and descriptions uses minimal tokens.

```
Example token usage:

Full skill content:     ~2000 tokens per skill
Name + description:     ~50 tokens per skill

With 10 skills:
Full load:              20,000 tokens (wasteful!)
Discovery only:         500 tokens (efficient!)
```

---

## 4. Phase 2: Matching (Request Analysis)

### How Matching Works

When you send a message, Claude analyzes whether any skill is relevant:

```
USER MESSAGE: "I'm getting an npm ERESOLVE error"

MATCHING PROCESS
================

Step 1: Extract intent
        Intent: "User has an npm dependency error and needs help"

Step 2: Compare to each skill's description

        Skill: retrospective
        Description: "Captures learnings from the current session..."
        Match score: LOW (not about capturing learnings)

        Skill: coding-workflow
        Description: "Best practices for coding, debugging..."
        Match score: MEDIUM (debugging is mentioned)

        Skill: infrastructure-debugging
        Description: "Troubleshooting dependencies, npm errors..."
        Match score: HIGH (npm, dependencies explicitly mentioned)

Step 3: Select best match
        Winner: infrastructure-debugging (highest relevance)

Step 4: Proceed to activation
```

### Semantic Matching (Not Just Keywords)

Claude uses semantic understanding, not simple keyword matching:

```
SEMANTIC MATCHING EXAMPLES
==========================

User says: "My package won't install"

Keyword matching would check:
  ✗ "install" in description? Maybe
  ✗ "package" in description? Maybe
  Result: Uncertain

Semantic matching understands:
  ✓ "package won't install" = dependency/installation issue
  ✓ Maps to: infrastructure-debugging skill
  Result: Confident match

---

User says: "How do I make this code better?"

Keyword matching:
  ✗ "better" not in any description
  Result: No match

Semantic matching:
  ✓ "make code better" = refactoring
  ✓ Maps to: coding-workflow skill
  Result: Confident match
```

### Match Confidence Levels

| Confidence | Behavior |
|------------|----------|
| HIGH | Claude asks to use the skill |
| MEDIUM | Claude may mention the skill, asks if relevant |
| LOW | Skill is not suggested |
| NONE | No skills considered |

---

## 5. Phase 3: Activation (Loading)

### The Activation Dialog

When a match is found, Claude asks permission:

```
USER: "I have an npm ERESOLVE error"

CLAUDE: I can help with that. I have an "infrastructure-debugging"
        skill that covers dependency troubleshooting.

        Would you like me to use it?

        [Yes] [No]

USER: [Clicks Yes or says "yes"]

ACTIVATION: Skill content is now loaded into context
```

### What Happens During Activation

```
ACTIVATION SEQUENCE
===================

1. User approves skill usage

2. Claude reads the full SKILL.md file:

   File: ~/.claude/skills/infrastructure-debugging/SKILL.md

   ┌─────────────────────────────────────────────────────────────┐
   │ ---                                                         │
   │ name: infrastructure-debugging                              │
   │ description: Troubleshooting dependencies...                │
   │ ---                                                         │
   │                                                             │
   │ # Infrastructure Debugging                                  │
   │                                                             │
   │ ## Dependency Resolution Strategy                           │
   │                                                             │
   │ ### When Facing Package Errors                              │
   │ 1. Read the full error...                                  │
   │ 2. Check version compatibility...                          │
   │ [... full content ...]                                      │
   │                                                             │
   │ ## Accumulated Learnings                                    │
   │ See [learnings.md](learnings.md) for insights.             │
   │                                                             │
   └─────────────────────────────────────────────────────────────┘

3. Content is added to Claude's context window:

   CONTEXT WINDOW
   ┌────────────────────────────────────────────────────┐
   │ [System prompt]                                    │
   │ [Conversation history]                             │
   │ [SKILL CONTENT NOW HERE] ◄── Newly added          │
   │ [User's current message]                           │
   └────────────────────────────────────────────────────┘

4. Claude can now follow the skill's instructions
```

### Activation Scope

Skills remain active for the current conversation. They don't persist across sessions.

```
SESSION 1                          SESSION 2
─────────                          ─────────
Skill activated                    Fresh start
Skill in context                   No skills in context
...                                (must be activated again)
Session ends
```

---

## 6. Phase 4: Execution (Using the Skill)

### How Claude Uses Skill Instructions

Once activated, Claude follows the skill's instructions:

```
SKILL CONTENT (in context):
┌─────────────────────────────────────────────────────────────┐
│ ## Dependency Resolution Strategy                           │
│                                                             │
│ ### When Facing Package Errors                              │
│ 1. Read the full error message                              │
│ 2. Check version compatibility                              │
│ 3. Clear caches first                                       │
│ 4. Try fresh install                                        │
│ 5. Check peer dependencies                                  │
└─────────────────────────────────────────────────────────────┘

USER: "I have an npm ERESOLVE error"

CLAUDE'S INTERNAL PROCESS:
1. I have infrastructure-debugging skill active
2. Skill says: "Read the full error message" first
3. I should ask the user for the full error
4. Then follow steps 2-5 as applicable

CLAUDE'S RESPONSE:
"I see you're having an ERESOLVE error. Can you share the full
error message? I'll need to see the complete output to identify
which packages are conflicting."
```

### Tool Usage from Skills

Skills can specify which tools to use:

```yaml
---
name: example-skill
allowed-tools: Read, Grep, Glob
---
```

When this skill is active:
- Claude can use Read, Grep, Glob without asking
- Other tools still require permission

---

## 7. Phase 5: Progressive Disclosure (Loading References)

### What is Progressive Disclosure?

Skills can reference other files that are loaded on-demand:

```
SKILL.md content:
┌─────────────────────────────────────────────────────────────┐
│ ## Infrastructure Debugging                                 │
│                                                             │
│ [Main instructions here...]                                 │
│                                                             │
│ ## Accumulated Learnings                                    │
│                                                             │
│ See [learnings.md](learnings.md) for past insights.        │ ◄── REFERENCE
│                                                             │
└─────────────────────────────────────────────────────────────┘

When does learnings.md get loaded?

NOT at skill activation (would waste tokens if not needed)
ONLY when Claude decides it's relevant to the current task
```

### Progressive Disclosure in Action

```
SCENARIO: User has npm error, skill is active

STEP 1: Claude reads skill instructions
        (learnings.md NOT loaded yet)

STEP 2: Claude provides initial guidance from skill
        "Try clearing npm cache first..."

STEP 3: User says "That didn't work, any other ideas?"

STEP 4: Claude decides to check learnings

        INTERNAL: "The skill mentions learnings.md
                   might have relevant past solutions.
                   Let me read it."

        ACTION: Claude reads learnings.md

        ┌─────────────────────────────────────────────────┐
        │ ## [2024-12-30] - npm ERESOLVE with React 18    │
        │                                                 │
        │ **Insight**: Use package.json overrides         │
        │ **Solution**: Add overrides section...          │
        └─────────────────────────────────────────────────┘

STEP 5: Claude uses the learning
        "Based on a previous session, you solved a similar
         issue using package.json overrides. Let's try that..."
```

### Why Progressive Disclosure?

```
TOKEN EFFICIENCY COMPARISON
===========================

Without progressive disclosure:
- Load ALL skill content upfront
- Load ALL learnings upfront
- Load ALL referenced files upfront
- Result: 5000+ tokens used immediately

With progressive disclosure:
- Load skill body: ~500 tokens
- Load learnings only if needed: ~200 tokens
- Load other files only if needed: variable
- Result: Only pay for what you use
```

---

## 8. Our Skills: Detailed Invocation Analysis

### Skill 1: retrospective

```
SKILL: retrospective
====================

DISCOVERY (Startup):
  Name: "retrospective"
  Description: "Captures learnings from the current session and
               updates skill files. Use when ending a session,
               after solving problems, debugging issues, or when
               the user says 'retrospective', 'capture learnings',
               or 'what did we learn'."

MATCHING TRIGGERS:
  ┌─────────────────────────────────────────────────────────────┐
  │ HIGH CONFIDENCE TRIGGERS:                                   │
  │   • "Let's do a retrospective"                             │
  │   • "Capture what we learned"                              │
  │   • "/retrospective" (slash command)                       │
  │   • "What did we learn today?"                             │
  │   • "Save these learnings"                                 │
  │                                                             │
  │ MEDIUM CONFIDENCE TRIGGERS:                                 │
  │   • "I want to remember this solution"                     │
  │   • "This was useful, let's save it"                       │
  │   • "Before we end..."                                     │
  │                                                             │
  │ LOW/NO MATCH:                                               │
  │   • General coding questions                                │
  │   • Debugging requests                                      │
  │   • Any request not about capturing learnings              │
  └─────────────────────────────────────────────────────────────┘

ACTIVATION:
  When matched, loads full SKILL.md with:
  - Session analysis instructions
  - Categorization rules
  - Learning entry format
  - File update procedures

EXECUTION:
  1. Claude analyzes the conversation
  2. Identifies successes, failures, insights
  3. Categorizes by domain
  4. Reads existing learnings files (progressive disclosure)
  5. Appends new entries
  6. Reports to user

PROGRESSIVE DISCLOSURE:
  References these files (loaded on-demand):
  - ~/.claude/skills/coding-workflow/learnings.md
  - ~/.claude/skills/infrastructure-debugging/learnings.md
  - ~/.claude/skills/failures-log/failures.md
```

### Skill 2: coding-workflow

```
SKILL: coding-workflow
======================

DISCOVERY (Startup):
  Name: "coding-workflow"
  Description: "Best practices for coding, debugging, and
               refactoring. Use when reviewing code, debugging
               issues, refactoring, or establishing coding patterns."

MATCHING TRIGGERS:
  ┌─────────────────────────────────────────────────────────────┐
  │ HIGH CONFIDENCE TRIGGERS:                                   │
  │   • "Help me debug this code"                              │
  │   • "Review this function"                                 │
  │   • "How should I refactor this?"                          │
  │   • "What's wrong with this code?"                         │
  │   • "Best practice for..."                                 │
  │                                                             │
  │ MEDIUM CONFIDENCE TRIGGERS:                                 │
  │   • "My code isn't working"                                │
  │   • "This function is slow"                                │
  │   • "Clean up this code"                                   │
  │                                                             │
  │ LOW/NO MATCH:                                               │
  │   • npm/pip installation errors                             │
  │   • Environment setup questions                             │
  │   • Retrospective requests                                  │
  └─────────────────────────────────────────────────────────────┘

ACTIVATION:
  When matched, loads SKILL.md with:
  - Code review approach
  - Debugging strategy (5 steps)
  - Refactoring guidelines
  - Common pitfalls

EXECUTION:
  Claude applies the debugging/refactoring strategies
  from the skill when helping with code issues.

PROGRESSIVE DISCLOSURE:
  - learnings.md: Loaded when Claude wants past coding insights
```

### Skill 3: infrastructure-debugging

```
SKILL: infrastructure-debugging
================================

DISCOVERY (Startup):
  Name: "infrastructure-debugging"
  Description: "Troubleshooting dependencies, installations,
               environment issues, and platform-specific problems.
               Use when facing npm/pip/package errors, installation
               failures, environment setup issues, or platform
               compatibility problems."

MATCHING TRIGGERS:
  ┌─────────────────────────────────────────────────────────────┐
  │ HIGH CONFIDENCE TRIGGERS:                                   │
  │   • "npm install failed"                                   │
  │   • "ERESOLVE error"                                       │
  │   • "pip install not working"                              │
  │   • "Dependency conflict"                                  │
  │   • "Package won't install"                                │
  │   • "Environment variable issue"                           │
  │                                                             │
  │ MEDIUM CONFIDENCE TRIGGERS:                                 │
  │   • "My build is failing"                                  │
  │   • "Something's wrong with my setup"                      │
  │   • "Works on my machine but not..."                       │
  │                                                             │
  │ LOW/NO MATCH:                                               │
  │   • Logic bugs in code                                      │
  │   • Code review requests                                    │
  │   • Retrospective requests                                  │
  └─────────────────────────────────────────────────────────────┘

ACTIVATION:
  When matched, loads SKILL.md with:
  - Dependency resolution strategy
  - Package manager specific guidance
  - Platform-specific issues (Windows, macOS, Linux)
  - Troubleshooting checklist

EXECUTION:
  Claude follows the troubleshooting checklist
  and applies platform-specific knowledge.

PROGRESSIVE DISCLOSURE:
  - learnings.md: Loaded for past infrastructure solutions
```

### Invocation Decision Tree

```
                         USER INPUT
                             │
                             ▼
              ┌──────────────────────────────┐
              │ Does input mention:          │
              │ retrospective, learnings,    │
              │ capture, what did we learn?  │
              └──────────────┬───────────────┘
                    │                │
                   YES              NO
                    │                │
                    ▼                ▼
           ┌───────────────┐  ┌──────────────────────────────┐
           │ retrospective │  │ Does input mention:          │
           │ skill         │  │ npm, pip, install, package,  │
           └───────────────┘  │ dependency, environment?     │
                              └──────────────┬───────────────┘
                                    │                │
                                   YES              NO
                                    │                │
                                    ▼                ▼
                           ┌───────────────┐  ┌──────────────────────────────┐
                           │infrastructure-│  │ Does input mention:          │
                           │debugging skill│  │ debug, code, refactor,       │
                           └───────────────┘  │ review, function, bug?       │
                                              └──────────────┬───────────────┘
                                                    │                │
                                                   YES              NO
                                                    │                │
                                                    ▼                ▼
                                           ┌───────────────┐  ┌───────────────┐
                                           │coding-workflow│  │ No skill      │
                                           │skill          │  │ matched       │
                                           └───────────────┘  └───────────────┘
```

---

## 9. Skill Invocation Debugging

### How to See Skill Activity

#### Method 1: Ask Claude

```
You: What skills are currently active?

Claude: Currently I have the "infrastructure-debugging" skill
        active from earlier in our conversation.
```

#### Method 2: Debug Mode

```bash
claude --debug
```

This shows:
```
[DEBUG] Loading skills from ~/.claude/skills/
[DEBUG] Found skill: retrospective
[DEBUG] Found skill: coding-workflow
[DEBUG] Found skill: infrastructure-debugging
[DEBUG] Skill registry built: 3 skills

[USER] I have an npm error

[DEBUG] Matching request to skills...
[DEBUG] Match found: infrastructure-debugging (score: 0.87)
[DEBUG] Requesting skill activation...

[USER] yes

[DEBUG] Loading skill content: infrastructure-debugging/SKILL.md
[DEBUG] Skill activated: infrastructure-debugging
```

#### Method 3: Observe Behavior

Signs a skill is active:
- Claude mentions the skill by name
- Response follows skill's structure
- Claude references skill-specific concepts

### Common Invocation Issues

| Issue | Symptom | Cause | Solution |
|-------|---------|-------|----------|
| Skill never triggers | Claude doesn't mention skill | Description doesn't match user language | Improve description with trigger phrases |
| Wrong skill triggers | Different skill activates | Multiple skills have overlapping descriptions | Make descriptions more specific |
| Skill triggers too often | Skill activates on unrelated requests | Description is too broad | Narrow the description |
| Learnings not used | Claude doesn't reference past learnings | Progressive disclosure not happening | Explicitly ask about past learnings |

---

## 10. Common Invocation Patterns

### Pattern 1: Direct Request

```
User: "Help me debug this function"
      ↓
Matching: coding-workflow skill matches "debug"
      ↓
Activation: Claude asks to use skill
      ↓
User: "Yes"
      ↓
Execution: Claude follows debugging strategy
```

### Pattern 2: Indirect Match

```
User: "Why doesn't this work?" [shows code]
      ↓
Matching: Claude infers this is a debugging request
      ↓
Matching: coding-workflow skill matches inferred intent
      ↓
[Same flow continues...]
```

### Pattern 3: Multi-Skill Session

```
Turn 1:
  User: "Fix this npm error"
  → infrastructure-debugging skill activated

Turn 2:
  User: "Now there's a bug in my code"
  → coding-workflow skill might also activate
  (both skills can be active simultaneously)

Turn 3:
  User: "Let's capture what we learned"
  → retrospective skill activated
  → Reviews solutions from both domains
```

### Pattern 4: Progressive Disclosure Chain

```
Turn 1:
  Skill activated (SKILL.md loaded)

Turn 2:
  Claude references learnings.md
  → learnings.md content loaded

Turn 3:
  User asks about a specific past failure
  → failures.md content loaded

Each file loads only when needed
```

### Pattern 5: Slash Command Override

```
User: "/retrospective"
      ↓
Command system intercepts
      ↓
retrospective skill forcefully activated
      ↓
No matching needed - direct invocation
```

---

## Summary: The Complete Picture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SKILL INVOCATION: THE COMPLETE PICTURE                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ STARTUP                                                              │    │
│  │ • Skills discovered (names + descriptions only)                     │    │
│  │ • Skill registry built                                              │    │
│  │ • ~50 tokens per skill used                                         │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                     │                                        │
│                                     ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ USER INPUT                                                           │    │
│  │ • User sends message                                                 │    │
│  │ • Intent extracted                                                   │    │
│  │ • Compared to skill descriptions                                    │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                     │                                        │
│                         ┌───────────┴───────────┐                           │
│                         ▼                       ▼                           │
│  ┌─────────────────────────────┐  ┌─────────────────────────────┐          │
│  │ MATCH FOUND                 │  │ NO MATCH                     │          │
│  │ • Ask user permission       │  │ • Respond normally           │          │
│  │ • Load full SKILL.md        │  │ • No skill context          │          │
│  │ • ~500 tokens added         │  └─────────────────────────────┘          │
│  └──────────────┬──────────────┘                                           │
│                 │                                                           │
│                 ▼                                                           │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ EXECUTION                                                            │    │
│  │ • Claude follows skill instructions                                 │    │
│  │ • Applies skill knowledge                                           │    │
│  │ • May read referenced files (progressive disclosure)               │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                     │                                        │
│                                     ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ RESPONSE                                                             │    │
│  │ • Skill-informed response generated                                 │    │
│  │ • Learning captured (if retrospective)                              │    │
│  │ • Files updated (if needed)                                         │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```
