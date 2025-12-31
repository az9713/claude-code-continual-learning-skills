Conduct a session retrospective to capture learnings.

## Instructions

1. Review the current conversation from the beginning
2. Identify all successes, failures, and insights
3. Categorize each learning by domain (coding-workflow or infrastructure-debugging)
4. Read the existing learnings files to avoid duplicates:
   - ~/.claude/skills/coding-workflow/learnings.md
   - ~/.claude/skills/infrastructure-debugging/learnings.md
   - ~/.claude/skills/failures-log/failures.md
5. Append new learnings using the format:

```markdown
## [YYYY-MM-DD] - Brief Title

**Context**: What was being done
**Outcome**: Success | Failure | Partial
**Insight**: Key takeaway
**Solution** (if applicable): What resolved it
**Tags**: #relevant #tags
```

6. For significant failures, also log to failures-log/failures.md
7. Summarize what was captured to the user

Focus on actionable insights that will help in future sessions. Be concise but specific.
