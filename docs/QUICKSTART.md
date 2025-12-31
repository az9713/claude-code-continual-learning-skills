# Quick Start Guide: 10 Hands-On Use Cases

Welcome! This guide will get you comfortable with the Continual Learning System through 10 practical exercises. Each use case builds on the previous one, starting simple and gradually introducing more features.

**Time Required**: About 30 minutes for all 10 use cases

**Prerequisites**: Claude Code installed and running

---

## Before You Begin: Verify Setup

Open a terminal and start Claude Code:

```bash
claude
```

Then ask:

```
What skills are available?
```

You should see at least these skills:
- `retrospective`
- `coding-workflow`
- `infrastructure-debugging`

If you don't see them, check the [Troubleshooting Guide](TROUBLESHOOTING.md).

---

## Use Case 1: Your First Retrospective (The Basics)

**Goal**: Learn how to manually capture learnings

**Scenario**: You just learned something useful and want to save it.

### Steps:

1. **Have a short conversation with Claude**:
   ```
   You: What's the difference between == and === in JavaScript?

   Claude: [Explains the difference - == does type coercion, === is strict equality]
   ```

2. **Run the retrospective command**:
   ```
   /retrospective
   ```

3. **Observe what happens**:
   ```
   Claude: Analyzing session for learnings...

   Found:
   - 1 coding insight (JavaScript equality operators)

   This session was primarily informational. No substantive learnings
   to persist (simple Q&A doesn't need to be stored).
   ```

**What You Learned**: The retrospective skill is smart - it only captures substantive learnings, not basic Q&A.

---

## Use Case 2: Capturing a Debugging Victory

**Goal**: Capture a real problem-solving experience

**Scenario**: You debug a problem and want to remember the solution.

### Steps:

1. **Simulate a debugging session**:
   ```
   You: I have a JavaScript function that returns undefined sometimes:

   async function getData() {
     const result = fetch('/api/data');
     return result.json();
   }

   Claude: I see the issue! You're missing 'await' before fetch().
   Without await, fetch() returns a Promise, not the response.

   Fix:
   async function getData() {
     const result = await fetch('/api/data');
     return result.json();
   }

   You: That fixed it! Thanks!
   ```

2. **Capture the learning**:
   ```
   /retrospective
   ```

3. **See what was captured**:
   ```
   Claude: Retrospective complete!

   Captured 1 learning:
   - Coding: Missing await causing undefined returns

   Updated files:
   - ~/.claude/skills/coding-workflow/learnings.md
   ```

4. **Verify the file was updated**:
   ```
   You: Show me what's in the coding-workflow learnings file

   Claude: [Reads and displays the file with your new learning]
   ```

**What You Learned**: Real problem-solving sessions create persistent learnings that Claude can reference later.

---

## Use Case 3: Infrastructure Problem Capture

**Goal**: Capture an infrastructure/dependency learning

**Scenario**: You solve an npm installation problem.

### Steps:

1. **Simulate an npm problem**:
   ```
   You: I'm getting this error when installing packages:

   npm ERR! ERESOLVE unable to resolve dependency tree
   npm ERR! peer react@"^17.0.0" from some-package@2.0.0

   Claude: This is a peer dependency conflict. Your project likely uses
   React 18, but some-package expects React 17.

   Solutions (in order of preference):
   1. Check if some-package has a newer version supporting React 18
   2. Add an override in package.json:
      {
        "overrides": {
          "some-package": {
            "react": "^18.0.0"
          }
        }
      }
   3. Use: npm install --legacy-peer-deps

   You: The override worked perfectly!
   ```

2. **Run retrospective**:
   ```
   /retrospective
   ```

3. **Observe the categorization**:
   ```
   Claude: Retrospective complete!

   Captured 1 learning:
   - Infrastructure: npm ERESOLVE resolution with package.json overrides

   Updated files:
   - ~/.claude/skills/infrastructure-debugging/learnings.md
   ```

**What You Learned**: Learnings are automatically categorized - coding issues go to coding-workflow, infrastructure issues go to infrastructure-debugging.

---

## Use Case 4: Seeing Skills in Action

**Goal**: Observe how Claude automatically applies skills

**Scenario**: Ask about a topic covered by a skill.

### Steps:

1. **Ask about debugging**:
   ```
   You: What's your approach to debugging code?
   ```

2. **Notice Claude's response structure**:
   ```
   Claude: [Claude will structure the response based on the coding-workflow
   skill's debugging strategy section]

   My debugging approach:
   1. Reproduce first - Confirm the issue before fixing
   2. Isolate the problem - Narrow to smallest reproducible case
   3. Read error messages carefully - They often contain the solution
   4. Check recent changes - What changed since it last worked?
   5. Verify assumptions - Test each assumption individually
   ```

3. **Ask Claude to confirm skill usage**:
   ```
   You: Did you use any skills for that answer?

   Claude: Yes, I applied the coding-workflow skill which contains
   debugging strategies and best practices.
   ```

**What You Learned**: Skills are applied automatically when Claude detects relevant topics.

---

## Use Case 5: Viewing Your Accumulated Learnings

**Goal**: See all the learnings you've captured

**Scenario**: Review what the system has learned from your sessions.

### Steps:

1. **Ask Claude to show learnings**:
   ```
   You: Show me all the learnings captured so far
   ```

2. **Claude reads the files**:
   ```
   Claude: Let me read your learning files...

   === Coding Workflow Learnings ===
   ## [2024-12-30] - Missing await causing undefined returns
   **Context**: Async function returning undefined intermittently
   **Outcome**: Success
   **Insight**: When async functions return undefined sometimes,
   check for missing await keywords
   ...

   === Infrastructure Debugging Learnings ===
   ## [2024-12-30] - npm ERESOLVE with React 18 peer dependency
   **Context**: Installing packages in React 18 project
   **Outcome**: Success
   **Insight**: Use overrides in package.json instead of --legacy-peer-deps
   ...
   ```

3. **Check the failures log**:
   ```
   You: What's in the failures log?

   Claude: The failures log is currently empty - no significant
   failures have been recorded yet.
   ```

**What You Learned**: You can review accumulated learnings at any time by asking Claude.

---

## Use Case 6: Recording a Failure

**Goal**: Capture what NOT to do

**Scenario**: You tried something that didn't work.

### Steps:

1. **Simulate a failed approach**:
   ```
   You: I tried to fix a memory leak by adding garbage collection
   hints with global.gc(), but it made things worse.

   Claude: That's a common misconception. In Node.js, manually calling
   gc() can actually hurt performance because:
   1. It forces a full GC cycle, blocking the event loop
   2. The V8 engine's automatic GC is highly optimized
   3. It can cause longer pause times

   Instead, focus on:
   - Finding the actual leak source
   - Using --inspect with Chrome DevTools
   - Checking for event listener cleanup

   You: Good to know! I won't try that again.
   ```

2. **Capture the failure**:
   ```
   /retrospective
   ```

3. **See the failure recorded**:
   ```
   Claude: Retrospective complete!

   Captured 1 learning (failure):
   - Coding: Avoid manual gc() calls for memory leaks

   Updated files:
   - ~/.claude/skills/coding-workflow/learnings.md
   - ~/.claude/skills/failures-log/failures.md
   ```

**What You Learned**: Failures are valuable learnings too! The system captures what NOT to do.

---

## Use Case 7: The Automatic Reminder

**Goal**: Experience the Stop hook reminder

**Scenario**: Try to end a session with uncaptured learnings.

### Steps:

1. **Have a substantive debugging session** (don't run retrospective yet):
   ```
   You: My Python script is slow. Help me optimize it.

   Claude: [Provides optimization advice]

   You: [Apply the suggestions and they work]
   ```

2. **Try to end the session**:
   ```
   You: Thanks, that's all!
   ```

3. **Notice the reminder**:
   ```
   Claude: Before ending, consider running /retrospective to capture
   learnings from this session. Key topics: Python optimization.

   If you prefer to skip, just say 'stop' again.
   ```

4. **Choose to capture or skip**:
   ```
   Option A - Capture:
   You: /retrospective

   Option B - Skip:
   You: stop
   ```

**What You Learned**: The system reminds you to capture learnings so nothing is lost.

---

## Use Case 8: Benefiting from Past Learnings

**Goal**: See how past learnings help in new sessions

**Scenario**: Encounter a similar problem to one you've solved before.

### Steps:

1. **Start a new Claude Code session**:
   ```bash
   # Exit current session
   exit

   # Start fresh
   claude
   ```

2. **Encounter a "familiar" problem**:
   ```
   You: My async function sometimes returns undefined. Any ideas?
   ```

3. **Watch Claude reference past learnings**:
   ```
   Claude: Looking at this issue...

   From my learnings, a common cause of intermittent undefined returns
   in async functions is a missing 'await' keyword. Can you share your
   code so I can check for this pattern?
   ```

**What You Learned**: Knowledge persists across sessions! Claude remembers past solutions.

---

## Use Case 9: Adding Context to Skills

**Goal**: Enhance a skill with your own best practices

**Scenario**: You have personal coding conventions you want Claude to follow.

### Steps:

1. **View the current skill**:
   ```
   You: Show me the coding-workflow SKILL.md file

   Claude: [Displays the skill content]
   ```

2. **Ask Claude to add your convention**:
   ```
   You: I want to add a personal convention: "Always use TypeScript
   strict mode for new projects." Can you add this to the coding
   workflow skill?
   ```

3. **Claude updates the skill**:
   ```
   Claude: I'll add that to the coding-workflow skill.

   [Updates ~/.claude/skills/coding-workflow/SKILL.md]

   Added under "Common Pitfalls":
   - Not using TypeScript strict mode for new projects
   ```

4. **Verify the update**:
   ```
   You: Show me the updated skill
   ```

**What You Learned**: Skills are editable - you can customize them with your own practices.

---

## Use Case 10: Creating a New Skill Domain

**Goal**: Extend the system with a new skill area

**Scenario**: You work with databases frequently and want a dedicated skill.

### Steps:

1. **Ask Claude to create a new skill**:
   ```
   You: Create a new skill called "database-patterns" for SQL and
   database best practices
   ```

2. **Claude creates the skill structure**:
   ```
   Claude: I'll create a new skill for database patterns.

   Creating:
   - ~/.claude/skills/database-patterns/SKILL.md
   - ~/.claude/skills/database-patterns/learnings.md

   [Creates the files with initial content]
   ```

3. **Verify the new skill**:
   ```
   You: What skills are available now?

   Claude: [Lists skills including new database-patterns]
   ```

4. **Update retrospective to include the new domain**:
   ```
   You: Update the retrospective skill to also categorize database-related
   learnings into the new database-patterns skill
   ```

**What You Learned**: The system is extensible - add new skill domains for your specific needs.

---

## Bonus: A Real-Life Example

This is an actual retrospective captured during the creation of this documentation system:

### The Session

While writing documentation, a user added reference links using a table format with raw URLs:

```markdown
| Resource | URL |
|----------|-----|
| Skills Documentation | https://code.claude.com/docs/en/skills |
```

The user requested a cleaner format with URLs embedded in titles.

### Running the Retrospective

```
User: /retrospective

Claude: **Session Analysis**
Reviewing this session...

**Topics covered:**
- Updating documentation with credits/acknowledgements
- Formatting fix: embedding URLs in titles instead of raw URLs

**Learnings identified:**
1. **Coding Workflow**: Documentation best practice - always embed URLs
   in descriptive titles using markdown `[Title](URL)` format.

---

**Retrospective Complete**

| Metric | Value |
|--------|-------|
| Learnings captured | 1 |
| Files updated | `~/.claude/skills/coding-workflow/learnings.md` |
```

### The Captured Learning

This entry was added to `~/.claude/skills/coding-workflow/learnings.md`:

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

### Why This Matters

In future sessions, when working on any documentation, Claude will remember this best practice and automatically use embedded links instead of displaying raw URLs.

---

## Summary: What You've Learned

| Use Case | Key Skill |
|----------|-----------|
| 1 | Running `/retrospective` command |
| 2 | Capturing debugging victories |
| 3 | Categorizing infrastructure learnings |
| 4 | Skills apply automatically |
| 5 | Viewing accumulated learnings |
| 6 | Recording failures (what NOT to do) |
| 7 | Automatic end-of-session reminders |
| 8 | Benefiting from past learnings |
| 9 | Customizing skills |
| 10 | Creating new skill domains |

---

## Next Steps

Now that you understand the basics:

1. **Use it daily**: Run `/retrospective` at the end of coding sessions
2. **Review periodically**: Check your learnings.md files monthly
3. **Customize**: Add your own conventions and best practices
4. **Extend**: Create skills for your specific domains

For more details, see:
- [User Guide](USER_GUIDE.md) - Complete documentation
- [Developer Guide](DEVELOPER_GUIDE.md) - How to modify the system
- [Troubleshooting](TROUBLESHOOTING.md) - If something isn't working

---

## Quick Reference Card

```
┌────────────────────────────────────────────────────────────┐
│              CONTINUAL LEARNING QUICK REFERENCE            │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  CAPTURE LEARNINGS:        /retrospective                  │
│                                                            │
│  VIEW SKILLS:              "What skills are available?"    │
│                                                            │
│  VIEW LEARNINGS:           "Show me my learnings"          │
│                                                            │
│  FILES LOCATION:           ~/.claude/skills/               │
│                                                            │
│  LEARNING FILES:                                           │
│    - ~/.claude/skills/coding-workflow/learnings.md         │
│    - ~/.claude/skills/infrastructure-debugging/learnings.md│
│    - ~/.claude/skills/failures-log/failures.md             │
│                                                            │
└────────────────────────────────────────────────────────────┘
```
