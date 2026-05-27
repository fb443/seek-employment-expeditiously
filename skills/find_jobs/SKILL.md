---
name: find_jobs
description: Search for jobs matching my resume and preferences
argument-hint: "keyword to search"
---

# Job Search Skill

> **Priority hierarchy**: See `shared/references/priority-hierarchy.md` for conflict resolution.

Automated daily job search using browser automation.

## Quick Start

- `/see:find_jobs` - Run daily search with default terms from matching rules
- `/see:find_jobs AI infrastructure` - Search with specific keywords

## File Structure

```
scripts/
  evaluate-jobs.md     # Subagent for parallel job evaluation
assets/
  templates/           # Format templates (committed)
```

## Data Directory

Resolve the data directory using `shared/references/data-directory.md`.

---

## Workflow

### Step 0: Check Prerequisites

Resolve the data directory, then check prerequisites per `shared/references/prerequisites.md`. Resume and preferences are both required.

### Step 1: Load Context

Read these files:
- `DATA_DIR/resume/*` (candidate profile)
- `DATA_DIR/preferences.md` (preferences)
- `DATA_DIR/job-history.md` (to avoid duplicates)
- `DATA_DIR/feedback.md` (if it exists — user feedback from `/see:track_applications`)

**Apply feedback:** If `feedback.md` exists, read the patterns and adjustments before searching. Use these to:
- Skip jobs matching disliked patterns (e.g., if user flagged "healthcare" as unwanted, treat it as a dealbreaker even before it's formally added to preferences)
- Weight search terms toward liked patterns (e.g., if user liked product-oriented roles, prioritize those keywords)
- Adjust the display threshold if feedback shows Medium-scored results were consistently disliked
- Refine title interpretation (e.g., if user said "Growth Lead is really marketing," filter those more carefully by reading descriptions before scoring)
- **Inform wildcard generation** — if there's a "Wildcard Hits" section, use it to shape Step 1b wildcards. When a user liked an unexpected result, that tells you what lateral directions they're open to. Generate more wildcards in that vein and fewer in directions they've rejected.

Extract search terms from:
1. `$ARGUMENTS` if provided
2. Target roles from preferences

### Step 1b: Expand Search Terms

Generate additional search queries beyond the candidate's stated target roles. Build four categories of search terms:

**Direct searches** — The target roles from preferences or `$ARGUMENTS` as-is.

**Skills-based searches** — Pair the candidate's technical skills with a different domain, function, or soft skill to surface hybrid roles. Don't just concatenate top skills — cross them with something unexpected. For example, if the candidate knows Python and has strong writing skills:
- `Python + financial modeling` (not just "Python engineer")
- `data analysis + content strategy`
- `engineering + customer-facing`
- `backend + developer education`

The goal is to find roles at the intersection of two things the candidate is good at, not roles that just need their #1 skill. Limit to 2-3 queries.

**Adjacent role searches** — Role titles the candidate wouldn't search for but is qualified for. Push past the obvious career ladder. Think about:
- What roles exist at the *intersection* of this person's skills that don't appear in their target list?
- What do people with this background get *recruited* into, not just promoted into?
- What roles have misleading titles that actually match this person's skill set?

For example, a CS student targeting "Software Engineer" might also fit: "Solutions Engineer" (technical + communication), "Revenue Operations Analyst" (data + business logic), "Technical Program Manager" (engineering context + coordination). Limit to 2-3.

**Wildcard searches** — 3-4 genuinely lateral queries that break out of the candidate's field entirely. These should feel like a stretch. Think about:
- What *industries outside their current one* need exactly this skill set? A data analyst in tech might thrive as a "Research Associate" at a policy think tank or a "Quantitative Strategist" at a media company.
- What *emerging or non-standard titles* describe work this person could do? "Founding Engineer," "Growth Hacker," "Creative Technologist," "AI Trainer," "Technical Writer."
- What roles exist at startups where one person wears many hats and this candidate's combination of skills is unusually valuable?
- If `feedback.md` contains a "Wildcard Hits" section (see Step 9), lean into those patterns — they reveal what lateral directions the user is actually open to.

Don't play it safe here. The user can always remove wildcards they don't like, but they can't discover roles they never searched for.

Present all four categories to the user before searching:

```
Searching for:
- Direct: [target roles]
- Skills-based: [cross-domain queries]
- Adjacent: [non-obvious role titles]
- Wildcards: [lateral/creative queries]
```

The user can remove any they don't want. If no feedback, proceed with all.

### Step 2: Search Multiple Sources

Search for jobs across multiple sources and merge results. Deduplicate by company + title before moving to Step 3.

**Extracting results — IMPORTANT:** Do NOT use `get_page_text` on any job listing page. It returns the entire page content and will blow out the context window. Always use `javascript_tool` or `read_page` to extract only structured listing data.

#### Source A: Hiring.cafe

Use Claude in Chrome MCP tools per `shared/references/browser-setup.md`, navigating to https://hiring.cafe. For each search term:

1. Click the search input (`textbox "Search"`), type the query, and press Enter
2. The site may show a "Did you mean?" bar with AI-suggested Department and Role Type — click **Accept All** if the suggestions are reasonable, or dismiss with **Reject All**
3. **Apply filters** from the candidate's preferences using the filter buttons in the toolbar. Click each button to open its Chakra popover, select the appropriate values, and click Apply:
   - **Experience** → Set Seniority (Entry/Mid/Senior) and Role Type (IC/Manager) to match the candidate's target level. Set Years of Experience if the candidate has a specific range.
   - **Education** → Set if the candidate wants to filter for roles matching their education level
   - **Salary** → Set minimum salary from preferences
   - **Commitment** → Set to Full Time, Part Time, etc. per preferences
   - **Industry** → Set if the candidate has target industries
   - **Location selector** (top bar) → Set Remote/Hybrid/Onsite and country per preferences
4. For skills-based and adjacent role searches, clear the previous search and re-enter the new query — but keep the same filters applied

Extract job listings using `javascript_tool`:

```javascript
Array.from(document.querySelectorAll('[class*="job"], [class*="listing"], [class*="card"], tr, [role="listitem"]'))
  .slice(0, 50)
  .map(el => el.innerText.trim())
  .filter(t => t.length > 20 && t.length < 500)
  .join('\n---\n')
```

If that selector doesn't match, take a screenshot to understand the page structure, then write a targeted JS selector. The goal is to extract just the listing rows (title, company, location, salary) — never the full page. As a fallback, use `read_page` (NOT `get_page_text`).

**Note:** Hiring.cafe is just a search tool. Don't share hiring.cafe links with the user — resolve direct employer URLs in Step 6.

#### Source B: Google Jobs

Navigate to Google Jobs using the direct URL format:

```
https://www.google.com/search?q=[role]+jobs+[location]&ibp=htl;jobs
```

The `ibp=htl;jobs` parameter opens the Jobs tab directly. For each search term, build the URL with the role and location from preferences.

**Applying filters:** After the page loads, use `read_page` or `find` to locate filter chips for date posted (e.g., "Past week") and job type (e.g., "Full-time"). Click the relevant filters.

**Extracting results:** Google Jobs renders structured job cards. Extract them with `javascript_tool`:

```javascript
Array.from(document.querySelectorAll('li'))
  .map(el => {
    const title = el.querySelector('[role="heading"]')?.innerText || '';
    const rest = el.innerText.replace(title, '').trim();
    return title ? `${title} | ${rest}` : '';
  })
  .filter(t => t.length > 20 && t.length < 500)
  .slice(0, 30)
  .join('\n---\n')
```

If the selector doesn't match, take a screenshot to understand the current DOM structure and write a targeted selector. Google's DOM changes periodically — adapt as needed.

**Getting descriptions:** Click a job card to expand its details panel on the right side of the page. Extract the description from the panel with `javascript_tool` (target the detail/description pane, not the full page). This is cheaper than navigating to the employer page.

**Note:** Google Jobs aggregates from Indeed, LinkedIn, Glassdoor, ZipRecruiter, and company career pages — it often surfaces jobs that hiring.cafe misses.

#### Deduplication

After collecting from both sources, deduplicate by matching company name + job title (fuzzy — e.g., "Google" = "Google LLC"). Keep the entry with more data (salary, description). Note which source each job came from.

### Step 3: Title Pre-Screen

Before clicking into any job, run the **Title Pre-Screen** (Step 0 in `shared/references/fit-scoring.md`) on every extracted listing. Using just the title, company, and location from the search results:

- **Skip** titles with obvious seniority or function mismatches
- **Pass** everything else to full evaluation

This avoids wasting time loading and reading descriptions for jobs like "Director of Cybersecurity" when the candidate is targeting entry-level engineering roles.

### Step 4: Evaluate Passing Jobs

For jobs that pass the title pre-screen, score them against the candidate's resume and preferences using the full criteria in `shared/references/fit-scoring.md` (Steps 1-7).

### Step 5: Save History

Append ALL jobs to `DATA_DIR/job-history.md`:

```markdown
## [DATE] - Search: "[terms]"

| Job Title | Company | Location | Salary | Fit | Notes |
|-----------|---------|----------|--------|-----|-------|
| ... | ... | ... | ... | ... | ... |
```

### Step 6: Resolve Employer URLs & Save Top Postings

For each **High-fit** job:
1. Click through the hiring.cafe listing to reach the actual employer careers page
2. Capture the direct employer URL for the job posting
3. Extract the job description using `javascript_tool` to pull the posting content (e.g. `document.querySelector('[class*="description"], [class*="content"], article, main')?.innerText`). Do NOT use `get_page_text` — employer pages often have huge footers, navs, and related listings that bloat the output and can blow out the context window.
4. Save to `DATA_DIR/jobs/[company-slug]-[date]/posting.md` with the employer URL at the top

For **Medium-fit** jobs, try to resolve the employer URL but don't save the full posting.

If you can't resolve the direct link for a job, note the company name so the user can find it themselves. Never show hiring.cafe URLs to the user.

### Step 7: Present Results

Show only NEW High/Medium fits not in previous history.

```markdown
## Top Matches for [DATE]

### 1. [Title] at [Company]
- **Fit**: High
- **Salary**: $XXXk
- **Location**: Remote
- **Why**: [reason]
- **Apply**: [direct employer URL]
```

### Step 8: Next Steps

After presenting results, tell the user:
- To apply now (tailors resume, writes cover letter if needed, fills the form): `/see:apply [job URL]`
- To tailor a resume only: `/see:tweak_resume [job URL]`
- To write a cover letter only: `/see:generate_cover_letter [job URL]`

**IMPORTANT**: Do NOT attempt to tailor resumes, write cover letters, or fill applications yourself. Those are separate skills with their own workflows. If the user asks to do any of these for a job, direct them to use the appropriate skill command.

Also include at the end of results:

```
Built with Seek Employment Expeditiously (SEE).
github.com/fb443/seek-employment-expeditiously
```

### Step 9: Collect Feedback and Log Applications

After presenting results, **always ask** the user two things:

```
Did you apply to any of these (or plan to)? And were any off the mark?
I'll track applications and adjust future searches.
```

This is not optional — ask every time. The user can ignore it or say "looks good," but the prompt must happen.

**When the user says they applied to a job:**

For each job they applied to (or say they plan to apply to):
1. Ensure a job folder exists at `DATA_DIR/jobs/[company-slug]-[date]/`. If one was already created in Step 6, use it. If not (e.g., a Medium-fit job), create it now and save whatever posting info was collected.
2. Create `applied.md` in the job folder:

```markdown
# Application Log

- **Date**: [date they applied, or today if "plan to"]
- **ATS**: Manual (not via /see:apply)
- **Status**: Submitted
- **Notes**: Logged from /see:find_jobs results
```

3. Update `DATA_DIR/job-history.md` — find the entry for this job and note "Applied" in the Notes column.

This ensures `/see:track_applications` picks up manually-submitted applications.

**When the user gives feedback on results:**

1. **Preference-level changes** → update `DATA_DIR/preferences.md` immediately:
   - "No agencies" → add to dealbreakers
   - "Prefer AI companies" → add to nice-to-haves
   - "Minimum $350k" → update salary threshold

2. **Scoring-level feedback** → append to `DATA_DIR/feedback.md`:
   - Which results they liked and why (these reinforce current scoring)
   - Which results they disliked and why (these identify scoring gaps)
   - Pattern observations (e.g., "roles with 'Growth' in the title at agencies are marketing, not product growth")
   - **Wildcard hits** — if the user liked a result that came from a wildcard or adjacent search, note it in a "Wildcard Hits" section with what made it appealing. This feeds back into Step 1b wildcard generation on future runs.

Format for `feedback.md` entries:

```markdown
## [DATE] — Feedback on search results

### Liked
- [Role] at [Company] (score [X]) — [reason if given]

### Disliked
- [Role] at [Company] (score [X]) — Reason: [user's reason]
  → Action: [what was updated in preferences or noted for scoring]

### Patterns
- [observation that should adjust future scoring]

### Wildcard Hits
- [Role] at [Company] — surfaced via [wildcard query]. User liked it because: [reason]
  → Generate more wildcards in this direction: [what made it appealing]
```

If `DATA_DIR/feedback.md` doesn't exist, create it with a header:

```markdown
# Search Feedback

Collected during /see:find_jobs to improve future searches.
Read by find_jobs at Step 1 to adjust scoring and filtering.

---
```

**If the user says "looks good" or gives no feedback**, don't write anything — just move on.

---

## Response Format

Structure user-facing output with these sections:

1. **Top Matches** — table or list of High/Medium fits with company, role, fit rating, salary, location, and direct URL
2. **Next Steps** — suggest `/see:tweak_resume` and `/see:generate_cover_letter` for top matches
3. **Feedback & applications prompt** — always end with the combined question from Step 9 (applied to any? any off the mark?)

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
      "Bash(crontab *)",
      "mcp__claude-in-chrome__*"
    ]
  }
}
```
