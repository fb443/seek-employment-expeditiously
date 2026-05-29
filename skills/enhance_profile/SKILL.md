---
name: enhance_profile
description: Deep work history interview that unlocks better resumes and cover letters
argument-hint: ""
---

# Enhance Profile Skill

> **Priority hierarchy**: See `shared/references/priority-hierarchy.md` for conflict resolution.

A 15-minute conversational interview that captures the detail behind your resume — accomplishments with numbers, motivations, career narrative, and hidden experience. This is what makes tailored resumes and cover letters genuinely compelling instead of generic.

You can run this anytime. Before your first job search, or after you've already applied to a few roles and want stronger materials going forward.

## Quick Start

- `/see:enhance_profile` - Start the work history interview

## File Structure

```
scripts/
  conduct-interview.md    # Work history interview guide
```

The profile template is at `shared/templates/profile.md`.

## Data Directory

Resolve the data directory using `shared/references/data-directory.md`.

---

## Workflow

### Step 0: Check Prerequisites

Resolve the data directory. A resume must exist in `DATA_DIR/resume/` — if not, tell the user to run `/see:create_profile` first.

Check if `DATA_DIR/profile.md` already exists with real content. If it does, tell the user:

> "You already have a work history profile. Want to update it (I'll ask about anything that's changed), or start fresh?"

If starting fresh, back up the existing profile to `DATA_DIR/profile.md.bak` before overwriting.

If updating, read the existing profile and skip questions where the answers are already detailed and complete. Focus on roles or areas that are thin.

### Step 1: Work History Interview

Read the candidate's resume and `DATA_DIR/resume/parsed.md`. You already know titles, dates, and extracted skills — don't re-ask for basics.

Follow the interview framework in `scripts/conduct-interview.md`. The interview has three phases:

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

### Step 2: Save Profile

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

### Step 3: Summary

```
Profile complete! Here's what we captured:

- [number] roles covered in detail
- [number] non-resume experiences surfaced
- Skills inventory updated ([number] skills tracked)
- Career narrative and motivation captured

This will make your tailored resumes and cover letters significantly
stronger. Your existing materials won't change — the profile kicks in
next time you run:
- /see:tweak_resume [job URL] - Tailor your resume
- /see:generate_cover_letter [job URL] - Write a cover letter
- /see:apply [job URL] - Full application flow

Built with Seek Employment Expeditiously (SEE).
github.com/fb443/seek-employment-expeditiously
```

---

## Response Format

Structure the final summary output with these sections:

1. **Profile Summary** — what was captured (roles, hidden experience, skills corrections)
2. **What's Next** — how the profile improves other skills, suggest next actions

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
