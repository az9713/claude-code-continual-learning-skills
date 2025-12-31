# Troubleshooting Guide

This guide helps you diagnose and fix common issues with the Continual Learning System. Issues are organized by symptom for easy lookup.

---

## Table of Contents

1. [Quick Diagnostics](#1-quick-diagnostics)
2. [Installation Issues](#2-installation-issues)
3. [Skill Issues](#3-skill-issues)
4. [Retrospective Issues](#4-retrospective-issues)
5. [Hook Issues](#5-hook-issues)
6. [File and Permission Issues](#6-file-and-permission-issues)
7. [Performance Issues](#7-performance-issues)
8. [Recovery Procedures](#8-recovery-procedures)
9. [Getting Help](#9-getting-help)

---

## 1. Quick Diagnostics

### Run These Checks First

Before diving into specific issues, run these diagnostic steps:

#### Check 1: Verify Skills Directory Exists

**Windows (Command Prompt):**
```cmd
dir %USERPROFILE%\.claude\skills
```

**Windows (Git Bash) / macOS / Linux:**
```bash
ls -la ~/.claude/skills/
```

**Expected Output:**
```
coding-workflow/
failures-log/
infrastructure-debugging/
retrospective/
```

#### Check 2: Verify SKILL.md Files Exist

```bash
ls ~/.claude/skills/*/SKILL.md
```

**Expected Output:**
```
/home/user/.claude/skills/coding-workflow/SKILL.md
/home/user/.claude/skills/infrastructure-debugging/SKILL.md
/home/user/.claude/skills/retrospective/SKILL.md
```

#### Check 3: Verify Settings File

```bash
cat ~/.claude/settings.json
```

**Should contain**: A valid JSON object with `hooks` section.

#### Check 4: Verify Skills Are Loaded in Claude Code

Start Claude Code and ask:
```
What skills are available?
```

**Expected**: List includes retrospective, coding-workflow, infrastructure-debugging.

---

## 2. Installation Issues

### Issue: Skills Directory Doesn't Exist

**Symptom**: `ls ~/.claude/skills/` shows "No such file or directory"

**Cause**: The skills were never installed, or installed to wrong location.

**Solution**:

1. Create the directory structure:
   ```bash
   mkdir -p ~/.claude/skills/retrospective
   mkdir -p ~/.claude/skills/coding-workflow
   mkdir -p ~/.claude/skills/infrastructure-debugging
   mkdir -p ~/.claude/skills/failures-log
   mkdir -p ~/.claude/commands
   ```

2. Re-run the installation process, or manually copy the SKILL.md files.

---

### Issue: Wrong Home Directory on Windows

**Symptom**: Files exist but Claude Code doesn't find them.

**Cause**: Windows has multiple "home" directory concepts.

**Solution**:

Check what `~` means in your environment:

**In Git Bash:**
```bash
echo ~
# Usually: /c/Users/YourName
```

**In Command Prompt:**
```cmd
echo %USERPROFILE%
# Usually: C:\Users\YourName
```

**In PowerShell:**
```powershell
echo $HOME
# Usually: C:\Users\YourName
```

Ensure your `.claude` folder is at `C:\Users\YourName\.claude\`.

---

### Issue: Claude Code Not Installed

**Symptom**: `claude` command not found.

**Cause**: Claude Code CLI not installed or not in PATH.

**Solution**:

1. Install Claude Code from Anthropic
2. Ensure it's in your system PATH
3. Restart your terminal after installation

---

## 3. Skill Issues

### Issue: Skills Don't Appear in List

**Symptom**: "What skills are available?" doesn't show your skills.

**Possible Causes and Solutions**:

#### Cause A: Wrong Directory Structure

**Check**: Each skill must be in its own directory.

**Wrong:**
```
~/.claude/skills/SKILL.md          ← Won't work!
```

**Right:**
```
~/.claude/skills/my-skill/SKILL.md  ← Correct!
```

#### Cause B: File Named Incorrectly

**Check**: File must be exactly `SKILL.md` (case-sensitive).

**Wrong:**
```
skill.md
Skill.md
SKILL.MD
skill.MD
```

**Right:**
```
SKILL.md
```

#### Cause C: Invalid YAML Frontmatter

**Check**: The frontmatter must be valid YAML.

**Wrong:**
```markdown
---
name: my-skill        ← Tab character (use spaces!)
description: A skill
---
```

**Wrong:**
```markdown

---                   ← Blank line before first ---
name: my-skill
description: A skill
---
```

**Right:**
```markdown
---
name: my-skill
description: A skill that does something useful
---
```

#### Cause D: Need to Restart Claude Code

**Solution**: Skills are loaded at startup. Exit and restart:
```bash
exit
claude
```

---

### Issue: Skill Doesn't Trigger When Expected

**Symptom**: You talk about a topic but the skill doesn't activate.

**Cause**: Description doesn't match your request semantically.

**Solution**: Improve the description to include trigger phrases.

**Before:**
```yaml
description: Helps with coding
```

**After:**
```yaml
description: Coding best practices and debugging. Use when reviewing code, debugging issues, refactoring, or discussing coding patterns.
```

**Test**: Use exact words from the description in your request.

---

### Issue: Skill Shows Error When Loading

**Symptom**: Claude mentions an error when trying to use the skill.

**Cause**: Usually malformed YAML or invalid content.

**Solution**:

1. Validate YAML using an online validator
2. Check for special characters that need escaping
3. Verify all required fields are present

**Required fields:**
```yaml
---
name: required-field
description: required-field
---
```

---

## 4. Retrospective Issues

### Issue: /retrospective Command Not Found

**Symptom**: Typing `/retrospective` doesn't work.

**Cause**: Command file missing or in wrong location.

**Solution**:

1. Check file exists:
   ```bash
   ls ~/.claude/commands/retrospective.md
   ```

2. If missing, create it:
   ```bash
   mkdir -p ~/.claude/commands
   # Then create the file with proper content
   ```

3. Restart Claude Code.

---

### Issue: Retrospective Says "No Learnings Found"

**Symptom**: Running `/retrospective` reports nothing to capture.

**Cause**: The session was informational rather than problem-solving.

**This is normal when**:
- You only asked simple questions
- No debugging or problem-solving occurred
- No new insights were generated

**If you think there should be learnings**:
- Be more explicit: "I just learned that X works better than Y. Please capture this."
- The retrospective skill looks for problems solved, not just information exchanged.

---

### Issue: Learnings Not Being Written to Files

**Symptom**: Retrospective runs but files aren't updated.

**Possible Causes**:

#### Cause A: File Path Issues

**Check**: Verify the learnings.md files exist:
```bash
ls ~/.claude/skills/coding-workflow/learnings.md
ls ~/.claude/skills/infrastructure-debugging/learnings.md
```

If missing, create them:
```bash
echo "# Learnings\n\n---\n" > ~/.claude/skills/coding-workflow/learnings.md
```

#### Cause B: Permission Issues

**Check**: Verify write permissions:
```bash
touch ~/.claude/skills/coding-workflow/learnings.md
```

If you get "Permission denied", fix permissions:
```bash
chmod 644 ~/.claude/skills/coding-workflow/learnings.md
```

#### Cause C: Claude Didn't Actually Write

**Check**: Sometimes Claude says it will write but hits an issue.

Ask directly:
```
Read the file ~/.claude/skills/coding-workflow/learnings.md and show me its contents
```

---

### Issue: Duplicate Learnings Being Created

**Symptom**: Same learning appears multiple times.

**Cause**: Deduplication check not working perfectly.

**Solution**:
1. Manually edit the file to remove duplicates
2. When running retrospective, remind Claude: "Check for existing similar entries before adding"

---

## 5. Hook Issues

### Issue: Stop Hook Not Triggering

**Symptom**: Sessions end without retrospective reminder.

**Possible Causes**:

#### Cause A: Hook Not Configured

**Check**: Verify settings.json contains the hook:
```bash
cat ~/.claude/settings.json | grep -A 20 "hooks"
```

Should show `"Stop"` section.

#### Cause B: Invalid JSON

**Check**: Validate JSON syntax:
```bash
cat ~/.claude/settings.json | python -m json.tool
```

If you see errors, fix the JSON syntax.

#### Cause C: Hook Timed Out

**Check**: The hook has a 15-second timeout. If the LLM takes too long, it fails silently.

**Solution**: Increase timeout in settings.json:
```json
"timeout": 30
```

---

### Issue: Stop Hook Always Blocks

**Symptom**: Can never end a session without the reminder.

**Cause**: Hook prompt is too aggressive.

**Solution**: Edit the prompt in settings.json to be more lenient, or remove the hook:

To disable:
```json
{
  "hooks": {}
}
```

---

### Issue: Hook Causes Errors

**Symptom**: Error messages when hook runs.

**Check**: Run Claude Code in debug mode:
```bash
claude --debug
```

Look for hook-related error messages.

---

## 6. File and Permission Issues

### Issue: "Permission Denied" Errors

**Symptom**: Claude can't read or write files.

**Cause**: File permissions don't allow Claude's operations.

**Solution (Unix/macOS/Linux/Git Bash):**
```bash
chmod -R 755 ~/.claude/skills/
chmod 644 ~/.claude/skills/*/*.md
chmod 644 ~/.claude/settings.json
```

**Solution (Windows):**
Right-click the `.claude` folder → Properties → Security → Edit → Allow full control.

---

### Issue: Files Are Empty

**Symptom**: SKILL.md or learnings.md files exist but are empty.

**Cause**: File creation failed or content was deleted.

**Solution**: Re-create the file content. See [Recovery Procedures](#8-recovery-procedures).

---

### Issue: Settings.json Is Corrupted

**Symptom**: Claude Code won't start or hooks don't work.

**Check**: Validate JSON:
```bash
python -m json.tool ~/.claude/settings.json
```

**Solution**: Fix JSON syntax or restore from backup.

---

## 7. Performance Issues

### Issue: Claude Code Starts Slowly

**Symptom**: Takes a long time to start.

**Cause**: Too many skills or very large skill files.

**Solution**:
1. Keep SKILL.md files under 500 lines
2. Move detailed content to referenced files
3. Remove unused skills

---

### Issue: Context Window Fills Up

**Symptom**: Claude mentions context limits or truncates responses.

**Cause**: Learnings.md files have grown too large.

**Solution**:
1. Archive old learnings to a separate file
2. Summarize and condense entries
3. Remove outdated learnings

---

### Issue: Hook Delays

**Symptom**: Noticeable pause when hooks run.

**Cause**: Prompt-based hooks require LLM calls.

**Solution**:
1. Reduce hook timeout
2. Simplify hook prompt
3. Consider using command-based hooks instead

---

## 8. Recovery Procedures

### Procedure: Restore Default SKILL.md Files

If skill files are corrupted or missing, recreate them:

#### retrospective/SKILL.md
```markdown
---
name: retrospective
description: Captures learnings from the current session. Use when ending a session, after solving problems, or when asked to capture learnings.
---

# Session Retrospective

Review the conversation and extract learnings. Categorize by domain and write to appropriate learnings.md files.

See the full content in the original installation.
```

#### coding-workflow/SKILL.md
```markdown
---
name: coding-workflow
description: Best practices for coding, debugging, and refactoring. Use when reviewing code, debugging issues, or discussing coding patterns.
---

# Coding Workflow Best Practices

[Restore from backup or see original installation]
```

#### infrastructure-debugging/SKILL.md
```markdown
---
name: infrastructure-debugging
description: Troubleshooting dependencies, installations, and environment issues. Use when facing npm/pip errors, installation failures, or platform issues.
---

# Infrastructure Debugging

[Restore from backup or see original installation]
```

---

### Procedure: Reset Learnings Files

If you want to start fresh:

```bash
# Backup first!
cp ~/.claude/skills/coding-workflow/learnings.md ~/learnings-backup-coding.md
cp ~/.claude/skills/infrastructure-debugging/learnings.md ~/learnings-backup-infra.md

# Reset
echo "# Coding Workflow Learnings

This file contains learnings from coding sessions.

---

<!-- Entries appended below -->" > ~/.claude/skills/coding-workflow/learnings.md

echo "# Infrastructure Debugging Learnings

This file contains learnings from infrastructure sessions.

---

<!-- Entries appended below -->" > ~/.claude/skills/infrastructure-debugging/learnings.md

echo "# Failures Log

This file documents significant failures.

---

<!-- Entries appended below -->" > ~/.claude/skills/failures-log/failures.md
```

---

### Procedure: Fix Corrupted settings.json

If settings.json is broken:

```bash
# Backup the broken file
cp ~/.claude/settings.json ~/.claude/settings.json.broken

# Create minimal working settings
echo '{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "If this session had substantive problem-solving, suggest running /retrospective. Otherwise, allow stopping.",
            "timeout": 15
          }
        ]
      }
    ]
  }
}' > ~/.claude/settings.json
```

---

### Procedure: Complete System Reset

To completely reset the continual learning system:

```bash
# Backup everything first!
cp -r ~/.claude/skills/ ~/claude-skills-backup-$(date +%Y%m%d)/

# Remove and recreate
rm -rf ~/.claude/skills/
rm -rf ~/.claude/commands/

# Then re-run the installation process
```

---

## 9. Getting Help

### Self-Help Resources

1. **This Troubleshooting Guide**: Check all relevant sections
2. **User Guide**: [USER_GUIDE.md](USER_GUIDE.md)
3. **Developer Guide**: [DEVELOPER_GUIDE.md](DEVELOPER_GUIDE.md)
4. **Architecture**: [ARCHITECTURE.md](ARCHITECTURE.md)

### Ask Claude

Claude knows about this system! Try asking:
```
I'm having trouble with [describe issue]. What should I check?
```

### Debug Mode

Run with debugging enabled:
```bash
claude --debug
```

This shows detailed information about:
- Skill loading
- Hook execution
- Tool usage

### Check Claude Code Documentation

For issues with Claude Code itself (not this learning system):
- Official docs: https://code.claude.com/docs/
- Skill docs: https://code.claude.com/docs/en/skills

---

## Quick Reference: Symptom → Solution

| Symptom | Likely Cause | Quick Solution |
|---------|--------------|----------------|
| Skills not listed | Wrong directory | Check `~/.claude/skills/[name]/SKILL.md` |
| Skill not triggering | Poor description | Add trigger keywords to description |
| /retrospective not found | Missing command | Check `~/.claude/commands/retrospective.md` |
| No learnings captured | Trivial session | Run on problem-solving sessions |
| Files not updating | Permission issue | Check file write permissions |
| Stop hook not firing | Missing config | Check settings.json hooks section |
| Slow startup | Too many skills | Keep SKILL.md files small |
| JSON errors | Syntax issues | Validate with json tool |
| Need fresh start | Corrupted files | Follow reset procedure |

---

## Diagnostic Commands Cheatsheet

```bash
# Check skills directory
ls -la ~/.claude/skills/

# Check each skill has SKILL.md
ls ~/.claude/skills/*/SKILL.md

# Validate settings.json
cat ~/.claude/settings.json | python -m json.tool

# Check file permissions
ls -la ~/.claude/skills/*/*.md

# View recent learnings
tail -50 ~/.claude/skills/coding-workflow/learnings.md

# Run Claude in debug mode
claude --debug

# Check command exists
ls ~/.claude/commands/

# Verify home directory
echo ~
```
