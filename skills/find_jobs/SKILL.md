---
name: find_jobs
description: Search for jobs matching my resume and preferences
argument-hint: "keyword to search"
---

# Job Search Skill

> **Priority hierarchy**: See `shared/references/priority-hierarchy.md` for conflict resolution.

Automated job search using parallel subagents across structured APIs (Apify) and browser automation (Hiring.cafe, Google Jobs).

## Quick Start

- `/see:find_jobs` - Run search with default terms from preferences
- `/see:find_jobs AI infrastructure` - Search with specific keywords

## File Structure

```
scripts/
  search-apify.md         # Subagent: Apify Google Jobs Scraper
  search-hiring-cafe.md   # Subagent: Hiring.cafe browser search
  search-google-jobs.md   # Subagent: Google Jobs browser search
  evaluate-jobs.md        # Subagent: job scoring and evaluation
assets/
  templates/              # Format templates (committed)
```

## Data Directory

Resolve the data directory using `shared/references/data-directory.md`.

---

## Workflow

### Step 1: Prerequisites & Context

Resolve the data directory, then check prerequisites per `shared/references/prerequisites.md`. Resume and preferences are both required.

Read these files:
- `DATA_DIR/resume/*` (candidate profile)
- `DATA_DIR/preferences.md` (preferences)
- `DATA_DIR/job-history.md` (to avoid duplicates)
- `DATA_DIR/feedback.md` (if it exists — user feedback from previous searches)

**Apply feedback:** If `feedback.md` exists, read patterns and adjustments before searching:
- Skip jobs matching disliked patterns
- Weight search terms toward liked patterns
- Adjust display threshold if Medium-scored results were consistently disliked
- Refine title interpretation based on past corrections

Extract search terms from:
1. `$ARGUMENTS` if provided
2. Target roles from preferences

### Step 2: Expand Search Terms

Generate additional search queries beyond the candidate's stated target roles. Build three categories:

**Direct searches** — Target roles from preferences or `$ARGUMENTS` as-is.

**Skills-based searches** — Combine the candidate's top 2-3 core skills into queries without a job title. Use the highest-experience core skills. Limit to 2-3 queries.

**Adjacent role searches** — 2-3 role titles the candidate may not have considered but would be qualified for based on their skill set, experience level, and career trajectory.

Present the expanded search terms to the user before searching:

```
Searching for:
- Direct: [target roles]
- Skills-based: [generated queries]
- Adjacent: [suggested roles]
```

The user can remove any they don't want. Proceed after confirmation.

### Step 3: Detect Sources & Launch Parallel Searches

Check which sources are available, then launch all available search subagents **in parallel** using the Agent tool. Each subagent runs independently and returns a JSON array of job listings.

#### 3a: Detect available sources

1. **Apify** — Try calling `mcp__Apify__search-actors` with query `"google jobs scraper"`. If it succeeds, Apify is available.

2. **Claude in Chrome** — Try calling `mcp__Claude_in_Chrome__tabs_context_mcp`. If it succeeds, browser sources (Hiring.cafe and Google Jobs) are available.

3. **If neither is available**, tell the user they need at least one search source and explain how to set up either.

4. **If only browser is available**, mention that Apify provides faster, more reliable results and offer to help connect it via `mcp__mcp-registry__suggest_connectors`. If the user skips, proceed with browser sources only.

#### 3b: Launch subagents in parallel

Spawn one Agent tool call per available source, **all in the same message** so they run concurrently. Each subagent gets the search terms and relevant preferences.

**Apify subagent** (if available):
```
Agent({
  description: "Search Apify Google Jobs",
  prompt: "<read scripts/search-apify.md for instructions>

Search terms: [all search terms]
Location: [from preferences]
Country: [from preferences]"
})
```

**Hiring.cafe subagent** (if browser available):
```
Agent({
  description: "Search Hiring.cafe",
  prompt: "<read scripts/search-hiring-cafe.md for instructions>

Search terms: [all search terms]
Preferences:
- Seniority: [from preferences]
- Salary minimum: [from preferences]
- Location: [from preferences]
- Commitment: [from preferences]
- Industry: [from preferences, if any]"
})
```

**Google Jobs subagent** (if browser available):
```
Agent({
  description: "Search Google Jobs",
  prompt: "<read scripts/search-google-jobs.md for instructions>

Search terms: [all search terms]
Location: [from preferences]"
})
```

**Important:** Launch all available subagents in a single message. Do NOT wait for one to finish before launching the next. The browser subagents each create their own tab, so they don't interfere with each other.

### Step 4: Merge & Deduplicate

After all subagents return, combine their JSON arrays into one list.

Deduplicate by matching company name + job title (fuzzy — e.g., "Google" = "Google LLC"). When a job appears in multiple sources:
- Prefer the entry with the most data (salary, description, direct apply link)
- Apify results typically have the most complete data
- Note which source(s) each job came from

### Step 5: Title Pre-Screen

Run the **Title Pre-Screen** (Step 0 in `shared/references/fit-scoring.md`) on every listing. Using just the title, company, and location:

- **Skip** titles with obvious seniority or function mismatches
- **Pass** everything else to scoring

### Step 6: Score & Rank

Score passing jobs against the candidate's resume and preferences using the full criteria in `shared/references/fit-scoring.md` (Steps 1-7). You can use `scripts/evaluate-jobs.md` as a subagent for this if the volume is high.

### Step 7: Save History

Append ALL jobs to `DATA_DIR/job-history.md`:

```markdown
## [DATE] - Search: "[terms]"

| Job Title | Company | Location | Salary | Fit | Notes |
|-----------|---------|----------|--------|-----|-------|
| ... | ... | ... | ... | ... | ... |
```

### Step 8: Resolve Employer URLs & Save Top Postings

For each **High-fit** job:

**If sourced from Apify:** The apply link and description are already in the data. Save directly to `DATA_DIR/jobs/[company-slug]-[date]/posting.md` with the employer URL at the top. No browser navigation needed.

**If sourced from Hiring.cafe or Google Jobs:** Use Claude in Chrome to:
1. Click through the listing to reach the actual employer careers page
2. Capture the direct employer URL
3. Extract the job description using `javascript_tool` (e.g., `document.querySelector('[class*="description"], [class*="content"], article, main')?.innerText`). Do NOT use `get_page_text`.
4. Save to `DATA_DIR/jobs/[company-slug]-[date]/posting.md` with the employer URL at the top

For **Medium-fit** jobs, try to resolve the employer URL but don't save the full posting.

Never show hiring.cafe URLs to the user.

### Step 9: Present Results

Show only NEW High/Medium fits not in previous history.

```markdown
## Top Matches for [DATE]

### 1. [Title] at [Company]
- **Fit**: High (score: XX)
- **Salary**: $XXXk
- **Location**: Remote
- **Source**: [apify / hiring.cafe / google-jobs]
- **Why**: [reason]
- **Apply**: [direct employer URL]
```

After results, tell the user:
- `/see:apply [job URL]` — apply end-to-end
- `/see:tweak_resume [job URL]` — tailor resume only
- `/see:generate_cover_letter [job URL]` — cover letter only

**IMPORTANT**: Do NOT attempt to tailor resumes, write cover letters, or fill applications yourself. Those are separate skills.

```
Built with Seek Employment Expeditiously (SEE).
github.com/fb443/seek-employment-expeditiously
```

### Step 10: Collect Feedback

**Always ask** after presenting results:

```
Did you apply to any of these (or plan to)? And were any off the mark?
I'll track applications and adjust future searches.
```

**When the user says they applied to a job:**

For each job:
1. Ensure a folder exists at `DATA_DIR/jobs/[company-slug]-[date]/`
2. Create `applied.md`:

```markdown
# Application Log

- **Date**: [date]
- **ATS**: Manual (not via /see:apply)
- **Status**: Submitted
- **Notes**: Logged from /see:find_jobs results
```

3. Update `DATA_DIR/job-history.md` — mark "Applied" in the Notes column.

**When the user gives feedback on results:**

1. **Preference-level changes** → update `DATA_DIR/preferences.md` immediately
2. **Scoring-level feedback** → append to `DATA_DIR/feedback.md`:

```markdown
## [DATE] — Feedback on search results

### Liked
- [Role] at [Company] (score [X]) — [reason if given]

### Disliked
- [Role] at [Company] (score [X]) — Reason: [user's reason]
  → Action: [what was updated in preferences or noted for scoring]

### Patterns
- [observation that should adjust future scoring]
```

If `DATA_DIR/feedback.md` doesn't exist, create it with a header:

```markdown
# Search Feedback

Collected during /see:find_jobs to improve future searches.
Read by find_jobs at Step 1 to adjust scoring and filtering.

---
```

If the user says "looks good" or gives no feedback, don't write anything.

---

## Response Format

1. **Top Matches** — High/Medium fits with company, role, score, salary, location, source, and direct URL
2. **Next Steps** — suggest `/see:tweak_resume` and `/see:generate_cover_letter` for top matches
3. **Feedback prompt** — always end with the feedback question from Step 10

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
      "mcp__claude-in-chrome__*",
      "mcp__Apify__*"
    ]
  }
}
```
