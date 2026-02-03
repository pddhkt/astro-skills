---
name: reflect
description: Push improvements back to Astro Skills source repository. Use after encountering issues with Astro, content, or islands skills.
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - AskUserQuestion
---

# Reflect — Astro Skills

Capture learnings from the current session and push improvements back to the `astro-skills` source repository.

## When to Use

Run `/astro:reflect` when you've:
- Encountered API mismatches (wrong Astro APIs, deprecated patterns)
- Discovered undocumented Astro patterns
- Found workflow improvements for content or islands architecture
- Fixed common error patterns that should be documented

## Workflow

```
┌─────────────────────────────────────────────────────────────┐
│  Current Session                                            │
│                                                             │
│  1. Analyze conversation for improvements                   │
│  2. Pre-check: Compare installed vs latest version          │
│     - If outdated → prompt to update first                  │
│     - If current → continue                                 │
│  3. Locate astro-skills repo                                │
│  4. Show proposed changes for approval                      │
│  5. Apply changes to skill files                            │
│  6. Commit and push to origin                               │
│  7. Update plugin in current project                        │
│     - Run: /plugin marketplace update astro-skills          │
│  8. Verify new version is installed                         │
└─────────────────────────────────────────────────────────────┘
```

## Process

### Step 1: Version Check

Before analyzing improvements, check if the installed plugin is up-to-date:

```bash
# Compare installed version with latest in source repo
# If outdated, prompt user to update first to avoid duplicate changes
```

If outdated:
```
The installed astro plugin is outdated. Please update first:

  /plugin marketplace update astro-skills

Then run /astro:reflect again.
```

### Step 2: Analyze Context

Review the current conversation for:

| Category | Look For |
|----------|----------|
| **API Mismatches** | Wrong function signatures, deprecated APIs used |
| **Missing Documentation** | Undocumented patterns discovered during work |
| **Workflow Improvements** | Better approaches found for common tasks |
| **Error Patterns** | Common mistakes that should have warnings |
| **Template Updates** | Outdated code templates that needed fixes |

### Step 3: Locate Source Repository

Find the astro-skills repository:

1. Check `ASTRO_SKILLS_REPO` environment variable
2. Default to `~/Projects/personal/astro-skills`
3. Clone from GitHub if not present locally:
   ```bash
   git clone git@github.com:pddhkt/astro-skills.git
   ```

### Step 4: Propose Changes

Show the user what will be updated:

```markdown
## Proposed Skill Improvements

### astro/reference/components.md
- Add ViewTransitions API usage pattern
- Update slot forwarding example for Astro 4.x

### content/SKILL.md
- Fix content collection schema example

### Reasoning
During this session, we encountered incorrect slot forwarding syntax
because the skill referenced Astro 3.x patterns.

---
Proceed with these changes? (y/n)
```

### Step 5: Apply Changes

After user approval:

1. Navigate to source repo
2. Edit the relevant skill files under `plugins/astro/`
3. Create a descriptive commit:
   ```bash
   git commit -m "reflect: update component patterns for Astro 4.x

   - astro/reference/components.md: Add ViewTransitions, fix slots
   - content/SKILL.md: Fix collection schema example

   Source: Session reflection"
   ```

### Step 6: Push to Origin

```bash
git push origin main
```

### Step 7: Update Current Project

After pushing, update the plugin in the current project:

```
/plugin marketplace update astro-skills
```

Verify the new version is installed and the changes are reflected.

### Step 8: Notify Team

Output a summary:

```markdown
## Skills Updated Successfully

The following changes have been pushed to astro-skills:
- astro/reference/components.md: Updated for Astro 4.x patterns
- content/SKILL.md: Fixed collection schema example

Your local installation has been updated.

**For other team members:**
Run `/plugin marketplace update astro-skills` to get these improvements.
```

## Configuration

Set a custom repository location:

```bash
# In your shell profile (~/.bashrc, ~/.zshrc)
export ASTRO_SKILLS_REPO="/custom/path/to/astro-skills"
```

## What Gets Captured

### Good Candidates for Reflection

- API signature corrections
- Missing configuration steps discovered
- Better Astro patterns found
- Common errors and their fixes
- Updated content collection schemas
- New component or island patterns

### Not Captured

- Project-specific code
- Credentials or secrets
- Personal preferences (unless universally beneficial)
- Temporary workarounds
