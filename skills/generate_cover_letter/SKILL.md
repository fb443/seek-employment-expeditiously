---
name: generate_cover_letter
description: Write a tailored cover letter for a specific job posting
argument-hint: "job URL, or 'last' to use the most recent job"
---

# Cover Letter Skill

> **Priority hierarchy**: See `shared/references/priority-hierarchy.md` for conflict resolution.

Write natural, persuasive cover letters that sound like a real professional wrote them.

## Quick Start

- `/see:generate_cover_letter` - Start the flow (will ask for a job URL)
- `/see:generate_cover_letter https://...` - Write a cover letter for a specific job posting
- `/see:generate_cover_letter last` - Write a cover letter for the most recent job

## File Structure

```
scripts/
  write-cover-letter.md       # Cover letter writing agent prompt
```

## Data Directory

Resolve the data directory using `shared/references/data-directory.md`.

---

## Workflow

### Step 0: Check Prerequisites

Resolve the data directory, then check prerequisites per `shared/references/prerequisites.md`. Resume is required; profile is recommended but not blocking.

Also read these optional files if they exist:
- `DATA_DIR/preferences.md` — liked patterns and nice-to-haves inform which themes and motivations to emphasize
- `DATA_DIR/feedback.md` — search feedback reveals what the user values (e.g., "likes AI/ML roles" → open with genuine enthusiasm for the AI space)
- `DATA_DIR/style-feedback.md` — corrections from past edits (e.g., "user always makes the tone more casual" → write in a more conversational register)

### Step 1: Get Job Details

**If `$ARGUMENTS` is "last" or empty:**
- Check `DATA_DIR/jobs/` for the most recently modified folder
- If found, read `posting.md` and `resume.md` from that folder
- Confirm with the user which job this is for
- If no job folders exist, ask the user for a job URL

**If `$ARGUMENTS` is a URL:**
- Check if a job folder already exists for this company in `DATA_DIR/jobs/`
- If yes, read the existing `posting.md` and `resume.md`
- If no, use Claude in Chrome MCP tools to fetch the job posting per `shared/references/browser-setup.md`
- Save the posting to `DATA_DIR/jobs/[company-slug]-[date]/posting.md` if not already saved

If the page can't be loaded, ask the user to paste the job description directly.

### Step 2: Gather Materials

For the target job folder, check what exists:
- `posting.md` - the job description (required)
- `resume.md` - a tailored resume (optional, improves quality significantly)

If no tailored resume exists, use the original resume and work history profile directly.

### Step 3: Write the Cover Letter

Follow the framework in `scripts/write-cover-letter.md`. Use:
- The work history profile (or original resume if no profile)
- The tailored resume for this role (if available)
- The job posting

The cover letter must:
- Be 250-350 words
- Start with "Dear Hiring Manager,"
- End with "Regards, [Name]"
- Use ONLY hyphens, never em dashes
- Sound like a real human wrote it
- Never fabricate or exaggerate any detail
- Connect 2-3 specific, measurable achievements to the employer's needs

**Feedback-informed writing**: If `feedback.md` or `preferences.md` were loaded, use them to shape the letter's angle:
- If liked patterns reveal themes the user is drawn to (e.g., "product-oriented", "mission-driven", "AI/ML"), weave genuine enthusiasm for those themes into the opening and motivation paragraphs
- If nice-to-haves from preferences match this company's attributes (e.g., "Series B+ startup" and this is a Series C company), mention that alignment naturally
- If `style-feedback.md` exists, apply its corrections preemptively — respect tone preferences, paragraph length preferences, and any phrasing patterns the user has consistently corrected

### Step 4: Present and Save

Save to `DATA_DIR/jobs/[company-slug]-[date]/cover-letter.md`

Present the cover letter to the user with:
- The full text
- A brief note on which achievements were highlighted and why
- The file path where it's saved

### Step 5: Iterate

Ask if the user wants to adjust:
- Tone (more formal, more casual, more technical)
- Which achievements to highlight
- Specific phrasing
- Length

Apply changes and re-save.

After the user is satisfied with the cover letter, include:

```
Built with Seek Employment Expeditiously (SEE).
github.com/fb443/seek-employment-expeditiously
```

### Step 6: Capture Style Feedback

After the user is satisfied (or if they mention making edits before sending), ask:

```
Did you change anything in the cover letter before sending it?
Even small tweaks help me write better ones next time.
```

If the user describes changes, categorize each edit and append to `DATA_DIR/style-feedback.md`:

```markdown
## [DATE] — Cover letter for [Role] at [Company]

### Edits
- [What they changed]: [What it was → what they changed it to]
  → Pattern: [What this tells us about their preferences]

### Applies to
- generate_cover_letter
```

Common categories:
- **Tone**: "Made it less formal" / "Too enthusiastic" / "More direct"
- **Opening**: "Rewrote the first paragraph" / "Removed the generic opener"
- **Achievement selection**: "Swapped in a different accomplishment" / "Removed the third example"
- **Length**: "Cut it down" / "Added more detail"
- **Closing**: "Changed the sign-off" / "Made the call-to-action softer"

If `DATA_DIR/style-feedback.md` doesn't exist, create it with:

```markdown
# Style Feedback

Corrections and preferences captured from user edits to generated resumes and cover letters.
Read by /see:tweak_resume and /see:generate_cover_letter to avoid repeating the same mistakes.

---
```

If they say no changes or skip, don't write anything.

---

## Response Format

Structure user-facing output with these sections:

1. **Cover Letter** — the full cover letter text
2. **Writing Notes** — which achievements were highlighted and why, any tradeoffs made
3. **What's Next** — suggest iterating on tone/emphasis, or using other skills

---

## Permissions Required

Add to `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Read(~/.claude/plugins/**)",
      "Read(~/.see/**)",
      "Write(~/.see/**)",
      "Edit(~/.see/**)",
      "mcp__claude-in-chrome__*"
    ]
  }
}
```
