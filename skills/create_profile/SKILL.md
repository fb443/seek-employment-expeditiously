---
name: create_profile
description: Quick setup - upload your resume and set job preferences so you can start searching immediately
argument-hint: ""
---

# Create Profile Skill

> **Priority hierarchy**: See `shared/references/priority-hierarchy.md` for conflict resolution.

Quick onboarding that gets you searching in under 5 minutes. Upload your resume, set your preferences, and you're ready to go.

## Quick Start

- `/see:create_profile` - Full quick setup (checks what's missing, does only what's needed)

## Data Directory

Resolve the data directory using `shared/references/data-directory.md`. If no directory exists this is a fresh install — create it in Step 1.

---

## Workflow

### Step 0: Check What's Already Done

Resolve the data directory, then check which of these exist and have real content (not just templates): resume, preferences.md, profile.md.

If everything exists, tell the user they're good to go and list the available skills. Otherwise, run only the missing phases in order.

### Step 1: Resume Upload & Parsing

Ask the user to provide their resume. Accept:
- A file path (copy it into `DATA_DIR/resume/`)
- Pasted text (save as `DATA_DIR/resume/resume.md`)

After saving, **parse the resume** and extract structured data into `DATA_DIR/resume/parsed.md`:

```markdown
# Parsed Resume Data

*Extracted: [DATE]*

## Contact
**Name**: [name]
**Email**: [email if present]
**Location**: [location if present]

## Education

| Degree | Field | Institution | Year |
|--------|-------|-------------|------|
| [e.g., M.S.] | [e.g., Computer Science] | [University] | [Year] |

Certifications:
- [e.g., AWS Solutions Architect]

## Work History

| # | Title | Company | Start | End | Duration |
|---|-------|---------|-------|-----|----------|
| 1 | [most recent title] | [company] | [date] | [date] | [Xyr Xmo] |

**Total professional experience**: [X years]

## Skills Extracted

**Core skills** (mentioned across multiple roles or prominently featured):

| Skill | Estimated Years | Last Used |
|-------|----------------|-----------|
| [skill] | [years — estimate from roles where it appears] | [year] |

**Secondary skills** (mentioned once or in older roles):

| Skill | Estimated Years | Last Used |
|-------|----------------|-----------|
| [skill] | [years] | [year] |

**Tools & platforms**: [list]

## Summary Observations
- **Career level**: [Entry / Mid / Senior IC / Manager / Director / VP / Executive]
- **Primary function**: [e.g., Software Engineering, Product Management, Marketing]
- **Industries worked in**: [list]
- **Management experience**: [X years managing teams of Y, or "none"]
```

**Estimating skill years**: Look at which roles mention each skill and sum the tenure of those roles. If a skill appears in a 3-year role and a 2-year role, estimate 5 years. Mark skills from the last 5 years as core, older ones as secondary.

Present the parsed summary to the user and ask:

> "Here's what I extracted from your resume. Does this look right? Anything I got wrong or missed?"

Fix any corrections before proceeding.

### Step 2: Preferences

Ask the user these questions conversationally (don't dump them all at once — adapt based on what the parsed resume already tells you):

1. **Roles & seniority**: "What roles are you targeting? And are you looking at the same level as your last role, or stepping up/down?" — Infer a reasonable default from the parsed resume if they're unsure.
2. **Location & work style**: "Any location preferences — remote, hybrid, specific cities?"
3. **Compensation**: "What's your salary target? And what's the minimum you'd consider if everything else was great?"
4. **Filters**: "Anything you'd want to filter out — types of companies, industries, travel requirements?"
5. **Nice-to-haves**: "Any preferences that aren't hard requirements — company stage, industry, specific perks?"

From their responses combined with the parsed resume data, save `DATA_DIR/preferences.md`:

```markdown
# Job Preferences

## Target Roles
- [parsed from response]

## Target Seniority
- Level: [VP / Director / Senior Manager / Senior IC / Mid IC]
- Will consider one level below: [Yes / No]
- Will consider one level above: [Yes / No]

## Education
- Highest degree: [from parsed resume]
- Field(s) of study: [from parsed resume]

## Experience
- Total years of relevant experience: [from parsed resume]
- Key industries worked in: [from parsed resume]

## Core Skills
| Skill | Years |
|-------|-------|
| [from parsed resume] | [years] |

## Secondary Skills
| Skill | Years | Last Used |
|-------|-------|-----------|
| [from parsed resume] | [years] | [year] |

## Location
[parsed from response]

## Compensation
- Target: $[X]k+
- Minimum: $[Y]k+

## Must-Haves
- [parsed from response]

## Dealbreakers
- [parsed from response]

## Nice-to-Haves
- [parsed from response]
```

The education, experience, and skills sections should be pre-filled from the parsed resume. The user only needs to provide the job-specific preferences (roles, salary, location, filters).

If they leave something out, that's fine — save what you have. They can always update later.

### Step 3: Summary

```
You're all set! Here's what we have:

- Resume: [filename] in DATA_DIR/resume/
- Preferences: [summary of target roles and key criteria]

You're ready to search:
- /see:find_jobs - Find matching jobs
- /see:tweak_resume [job URL] - Tailor your resume
- /see:apply [job URL] - Apply to a job end-to-end

Want better resumes and cover letters? Run /see:enhance_profile for a
15-minute interview that gives us the detail to write really compelling,
tailored materials. You can do this anytime — before or after searching.

Built with Seek Employment Expeditiously (SEE).
github.com/fb443/seek-employment-expeditiously
```

---

## Response Format

Structure the final summary output with these sections:

1. **Setup Summary** — what was configured (resume, preferences) with brief details
2. **What's Next** — list available skills the user can now run, with a nudge toward `/see:enhance_profile`

---

## Permissions Required

Add to `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Read(~/.see/**)",
      "Write(~/.see/**)",
      "Edit(~/.see/**)"
    ]
  }
}
```
