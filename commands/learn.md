---
description: Extract reusable patterns from the session and save them as a Cursor skill (folder + SKILL.md).
---

# /learn - Extract Reusable Patterns

Analyze the current session and extract any patterns worth saving as skills.

## Trigger

Run `/learn` at any point during a session when you've solved a non-trivial problem.

## What to Extract

Look for:

1. **Error Resolution Patterns**
   - What error occurred?
   - What was the root cause?
   - What fixed it?
   - Is this reusable for similar errors?

2. **Debugging Techniques**
   - Non-obvious debugging steps
   - Tool combinations that worked
   - Diagnostic patterns

3. **Workarounds**
   - Library quirks
   - API limitations
   - Version-specific fixes

4. **Project-Specific Patterns**
   - Codebase conventions discovered
   - Architecture decisions made
   - Integration patterns

## Output format (Cursor skills)

Cursor expects each skill as a **directory** containing **`SKILL.md`** (not a bare `.md` at the skill root).

**Primary location (global, learned patterns):** create:

`~/.cursor/skills/learned/<pattern-name>/SKILL.md`

Use a short, kebab-case `pattern-name` (folder name = skill identifier).

### SKILL.md template

```markdown
---
name: learned-<pattern-name>
description: When this pattern applies and what it does (one or two sentences).
---

# [Descriptive Pattern Name]

**Extracted:** [Date]
**Context:** [Brief description of when this applies]

## Problem

[What problem this solves - be specific]

## Solution

[The pattern/technique/workaround]

## Example

[Code example if applicable]

## When to Use

[Trigger conditions - what should activate this skill]
```

## Process

1. Review the session for extractable patterns
2. Identify the most valuable/reusable insight
3. Draft `SKILL.md` using the template above
4. Ask the user to confirm before saving
5. Write the file under `learned/<pattern-name>/SKILL.md` at the chosen location

## Notes

- Don't extract trivial fixes (typos, simple syntax errors)
- Don't extract one-time issues (specific API outages, etc.)
- Focus on patterns that will save time in future sessions
- Keep skills focused — one pattern per skill folder
