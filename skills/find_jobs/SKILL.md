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
- `DATA_DIR/linkedin-contacts.csv` (if it exists — for network matching)
- `DATA_DIR/feedback.md` (if it exists — user feedback from `/see:track_applications`)

**Apply feedback:** If `feedback.md` exists, read the patterns and adjustments before searching. Use these to:
- Skip jobs matching disliked patterns (e.g., if user flagged "healthcare" as unwanted, treat it as a dealbreaker even before it's formally added to preferences)
- Weight search terms toward liked patterns (e.g., if user liked product-oriented roles, prioritize those keywords)
- Adjust the display threshold if feedback shows Medium-scored results were consistently disliked
- Refine title interpretation (e.g., if user said "Growth Lead is really marketing," filter those more carefully by reading descriptions before scoring)

Extract search terms from:
1. `$ARGUMENTS` if provided
2. Target roles from preferences

### Step 1b: Expand Search Terms

Generate additional search queries beyond the candidate's stated target roles. Build three categories of search terms:

**Direct searches** — The target roles from preferences or `$ARGUMENTS` as-is.

**Skills-based searches** — Combine the candidate's top 2-3 core skills (from the profile's skills inventory) into queries without a job title. For example, if the candidate's core skills are Python (8yr), Kubernetes (5yr), and distributed systems (6yr), generate queries like:
- `Python Kubernetes distributed systems`
- `Python infrastructure`
- `Kubernetes platform`

Use the highest-experience core skills. Limit to 2-3 skills-based queries to avoid noise.

**Adjacent role searches** — Based on the candidate's skill set, experience level, and career trajectory, infer 2-3 role titles they may not have considered but would be qualified for. Think about:
- What roles combine this person's skills in a non-obvious way?
- What roles do people with this background commonly transition into?
- Are there emerging roles that map well to this skill set?

For example, a senior backend engineer with strong data skills might get: "ML Infrastructure Engineer", "Developer Experience Engineer", "Platform Reliability Engineer".

Present the expanded search terms to the user before searching:

```
Searching for:
- Direct: [target roles]
- Skills-based: [generated queries]
- Adjacent: [suggested roles]
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

If LinkedIn contacts were loaded, cross-reference each result's company name against the "Company" column in the CSV. Use fuzzy matching (e.g. "Google" matches "Google LLC", "Alphabet/Google"). If there's a match, include the contact's name and title.

```markdown
## Top Matches for [DATE]

### 1. [Title] at [Company]
- **Fit**: High
- **Salary**: $XXXk
- **Location**: Remote
- **Why**: [reason]
- **Network**: You know [First Last] ([Position]) at [Company]
- **Apply**: [direct employer URL]
```

Omit the "Network" line if there are no contacts at that company.

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

### Step 9: Learn from Feedback

If the user provides immediate feedback on results, update `DATA_DIR/preferences.md`:
- "No agencies" → add to dealbreakers
- "Prefer AI companies" → add to nice-to-haves
- "Minimum $350k" → update salary threshold

Also check `DATA_DIR/feedback.md` if it exists. This file is maintained by `/see:track_applications feedback` and contains structured feedback from past searches — patterns the user liked/disliked, scoring adjustments, and search term refinements. Apply these when filtering and scoring results.

After presenting results, remind the user:
- "Run `/see:track_applications feedback` later to review these results and help me find better matches next time."

---

## Response Format

Structure user-facing output with these sections:

1. **Top Matches** — table or list of High/Medium fits with company, role, fit rating, salary, location, network contacts, and direct URL
2. **Next Steps** — suggest `/see:tweak_resume` and `/see:generate_cover_letter` for top matches

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
