---
name: create_profile
description: One-time onboarding - upload resume, set preferences, and do a work history interview
argument-hint: "'interview' to skip to the interview portion"
---

# Create Profile Skill

> **Priority hierarchy**: See `shared/references/priority-hierarchy.md` for conflict resolution.

One-time onboarding that ensures all your data is in place before using the other skills.

## Quick Start

- `/see:create_profile` - Full onboarding (checks what's missing, does only what's needed)
- `/see:create_profile interview` - Just the work history interview (if resume/prefs are already done)

## File Structure

```
scripts/
  conduct-interview.md    # Work history interview guide
```

The profile template is at `shared/templates/profile.md`.

## Data Directory

Resolve the data directory using `shared/references/data-directory.md`. For setup, if no directory exists this is a fresh install — create it in Step 1.

---

## Workflow

### Step 0: Check What's Already Done

Resolve the data directory, then check which of these exist and have real content (not just templates): resume, preferences, profile.md.

If `$ARGUMENTS` is "interview", skip to Step 3 (but check that a resume exists first). If `$ARGUMENTS` is empty, run Steps 1-3 in order.

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

## Search Sources
- LinkedIn search (Apify, ~$0.08/run): [Yes / No — default No]
```

The education, experience, and skills sections should be pre-filled from the parsed resume. The user only needs to provide the job-specific preferences (roles, salary, location, filters). The LinkedIn search option should only be set to Yes if the user has Apify tools available and explicitly opts in.

If they leave something out, that's fine — save what you have. They can always update later.

### Step 3: Work History Interview

Have a conversational interview to build a comprehensive work history profile. Use the parsed resume data from Step 1 as your starting point — you already know titles, dates, and extracted skills, so don't re-ask for basics.

**Phase 1: Role deep-dives** — Go through each role on the resume, most recent first. For each role, ask four questions:

1. **Context & mandate**: "Tell me about [Company] — what did they do, how big were they, and what were you hired to do?"
2. **Accomplishments**: "What were your biggest wins there? Let's get specific — numbers, scale, what changed because of you."
3. **Motivation**: "What drew you to this role? What about the work kept you energized?"
4. **Transition**: "Why did you move on?"

Spend full time on recent roles (last 5 years). For older roles, skip motivation and transition. After all roles, ask the career narrative questions:
- "What's the thread that connects these roles?"
- "What kind of work makes you lose track of time?"
- "Where do you want to go next, and why?"

Capture their answers in their own words — these feed directly into cover letters.

**Phase 2: Hidden experience discovery** — After covering all resume roles, ask these questions to surface experience the resume doesn't mention:

1. "Is there anything you've done professionally that's *not* on your resume? Side projects, consulting, freelance work, open source contributions, volunteer roles with real responsibilities?"
2. "Have you worked in any industries or domains outside your main career track? Sometimes experience in one field translates to another in ways people don't expect."
3. "Any skills you use regularly that you've never thought to put on a resume? Things like public speaking, data analysis, writing, project management, mentoring?"
4. "Have you built or led anything outside of work — a community, a product, a team?"

For each piece of hidden experience, capture it with the same structure as a resume role: what they did, what the impact was, what skills it demonstrates, and how long they did it.

**Phase 3: Skills reconciliation** — Show the user the skills inventory from the parsed resume and ask:

> "Here are the skills I extracted from your resume. What's missing? What would you add, remove, or adjust the years on?"

Update the skills inventory in `DATA_DIR/resume/parsed.md` and `DATA_DIR/preferences.md` with any corrections.

After the interview, save the profile to `DATA_DIR/profile.md` using the template at `shared/templates/profile.md`. Make sure to populate:
- The **Education** table (from parsed resume)
- The **Skills Inventory** with core vs. secondary and years (from parsed resume + corrections)
- All resume roles with accomplishments and detail from the interview
- A **Non-Resume Experience** section for anything surfaced in Phase 2:

```markdown
---

## Non-Resume Experience

### [Activity/Project/Role]
**Context**: [What it was — freelance, volunteer, side project, etc.]
**Duration**: [timeframe]
**What they did**: [description]
**Skills demonstrated**: [relevant skills]
**Impact**: [outcomes if any]

---
```

Include these in the Cross-Role Patterns section as well — hidden experience often reveals the candidate's real strengths.

### Step 4: Summary

```
You're all set! Here's what we have:

- Resume: [filename] in DATA_DIR/resume/
- Preferences: [summary of target roles and key criteria]
- Work History Profile: [number of roles covered]

You're ready to use:
- /see:find_jobs - Find matching jobs
- /see:tweak_resume [job URL] - Tailor your resume
- /see:generate_cover_letter [job URL] - Write a cover letter

Built with Seek Employment Expeditiously (SEE).
github.com/fb443/seek-employment-expeditiously
```

---

## Response Format

Structure the final summary output with these sections:

1. **Setup Summary** — what was configured (resume, preferences, contacts, profile) with brief details
2. **What's Next** — list available skills the user can now run

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
