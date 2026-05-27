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

Use Claude in Chrome MCP tools per `shared/references/browser-setup.md`. Hiring.cafe uses Next.js SSR — structured job data is embedded in `window.__NEXT_DATA__` on every search page, so extract from there instead of scraping the DOM.

**Building the search URL:** Construct URLs with these parameters (the only ones that actually filter server-side):

```
https://hiring.cafe/?searchState={"searchQuery":"[query]","departments":["[dept]"],"roleTypes":["[type]"]}
```

- `searchQuery` — the search term
- `departments` — array of department filters. Common values: `"Software Development"`, `"Data & Analytics"`, `"Product"`, `"Design"`, `"Sales"`, `"Marketing"`, `"Operations"`, `"Finance"`
- `roleTypes` — array: `"Individual Contributor"`, `"Manager"`, `"Executive"`

Map the candidate's preferences to `departments` and `roleTypes`. Other filters (salary, seniority, commitment, location) are **not supported via URL** — apply those in our scoring logic instead.

**For each search term:** Navigate to the constructed URL and wait for the page to load.

**Extracting results** with `javascript_tool`:

```javascript
const hits = (window.__NEXT_DATA__?.props?.pageProps?.ssrHits || []);
hits.filter(h => !h.is_hc_pinned).slice(0, 25).map(h => {
  const v5 = h.v5_processed_job_data || {};
  const co = h.enriched_company_data || {};
  const ji = h.job_information || {};
  return {
    t: ji.title || h.hc_title,
    co: co.name,
    sMin: v5['yearly_min_compensation'],
    sMax: v5['yearly_max_compensation'],
    sen: v5['seniority_level'],
    wp: v5['workplace_type'],
    city: v5['workplace_city'],
    st: v5['workplace_state'],
    emp: co['num_employees'],
    dept: v5['department']
  };
});
```

This returns structured data directly — no DOM scraping needed. Promoted listings (`is_hc_pinned: true`) are ads; filter them out.

**Resolving employer URLs:** Extract apply domains separately (full URLs trigger a security filter):

```javascript
hits.filter(h => !h.is_hc_pinned).slice(0, 25).map(h => {
  const raw = h.v5_processed_job_data?.apply_url || h.job_information?.url || '';
  try { return new URL(raw).hostname; } catch { return ''; }
});
```

Use these domains in Step 6 to navigate directly to employer career pages.

**Fallback:** If `__NEXT_DATA__` is empty or the page doesn't load within 15 seconds, take a screenshot to diagnose. If hiring.cafe is down, skip to Source B — don't waste time retrying.

**Note:** Hiring.cafe is just a search tool. Don't share hiring.cafe links with the user — resolve direct employer URLs in Step 6.

#### Source B: Work at a Startup (YC)

Covers early-stage YC startups that typically don't appear on major job boards. Uses browser automation — the site is a Rails + Algolia app with Tailwind classes.

**Building the search URL:**

```
https://www.workatastartup.com/jobs?role=[role_code]&type=any&hasEquity=any&industry=any&minSalary=0
```

Role codes: `eng` (Engineering), `design` (Design), `product` (Product), `sales` (Sales), `marketing` (Marketing), `ops` (Operations), `data` (Data Science). Map the candidate's target roles to the appropriate code. For multiple roles, use separate searches.

**For each search term:** Navigate to the URL and wait for the page to load (3-4 seconds).

**Extracting results** with `javascript_tool` — the site renders up to 30 job cards without sign-in:

```javascript
const cards = document.querySelectorAll('div[class*="cursor-pointer"][class*="rounded"][class*="border"]');
Array.from(cards).map(card => {
  const company = card.querySelector('.font-bold')?.innerText?.trim() || '';
  const detailsText = card.querySelector('.job-details')?.innerText?.trim() || '';
  const salaryMatch = detailsText.match(/\$[\d,]+K?\s*-\s*\$[\d,]+K?/);
  const isRemote = detailsText.includes('Remote');
  // Title: text between company info and job-details
  const companyDesc = card.querySelector('.text-sm.text-gray-700')?.innerText?.replace(company, '')?.trim() || '';
  const allText = card.innerText;
  const compEnd = allText.indexOf(companyDesc) + companyDesc.length;
  const detStart = allText.indexOf(detailsText);
  const title = allText.substring(compEnd, detStart).trim();
  return { company, title, salary: salaryMatch?.[0] || '', remote: isRemote, details: detailsText.substring(0, 120) };
}).filter(j => j.title);
```

This returns structured data with company (including YC batch), title, salary range, and location. Nearly every listing includes salary data.

**Why this source matters:** YC startups — especially recent batches (W26, F25, P26) — often post *only* here. Roles like "Founding Engineer," "Forward Deployed Engineer," and "Product Engineer" at 5-person companies won't appear on hiring.cafe.

**Fallback:** If the page doesn't load or returns 0 cards, skip — the site occasionally has downtime.

#### Source C: LinkedIn via Apify (Optional)

LinkedIn is the largest pool of exclusive job postings that hiring.cafe cannot access. This source uses the Apify `curious_coder/linkedin-jobs-scraper` actor and costs ~$0.001/result.

**Only run this source if:**
- The user has Apify MCP tools available in their environment
- The candidate's preferences include `linkedin_search: true` (or the user explicitly asks for LinkedIn results)

**Running the actor:** Use `call-actor` with the actor name `curious_coder/linkedin-jobs-scraper`. First fetch the input schema with `fetch-actor-details`, then call with:

```json
{
  "searchQueries": ["[role] [location]"],
  "location": "[city or 'Remote']",
  "jobType": ["Full-time"],
  "experienceLevel": ["Entry level", "Associate", "Mid-Senior level"],
  "limit": 25
}
```

Map the candidate's seniority preference to LinkedIn's experience levels. Run one query per direct search term (skip wildcards — LinkedIn's search is already broad).

**Processing results:** The actor returns structured JSON with title, company, location, salary (when listed), description, and apply URL. No DOM scraping needed.

**Cost awareness:** At $0.001/result, a typical search (3-4 queries × 25 results) costs ~$0.08. Mention the cost to the user the first time this source runs.

#### Deduplication

After collecting from all sources, deduplicate by matching company name + job title (fuzzy — e.g., "Google" = "Google LLC"). Keep the entry with more data (salary, description). Note which source each job came from.

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
1. **Resolve the direct employer URL:**
   - **Hiring.cafe jobs** — use the apply domain extracted in Source A to navigate to the employer careers page. Never show hiring.cafe URLs to the user.
   - **WaaS jobs** — click through the listing on workatastartup.com to reach the startup's application page. WaaS links often go directly to the company.
   - **LinkedIn jobs** — the Apify results include direct apply URLs. Use those.
2. Capture the direct employer URL for the job posting
3. Extract the job description using `javascript_tool` to pull the posting content (e.g. `document.querySelector('[class*="description"], [class*="content"], article, main')?.innerText`). Do NOT use `get_page_text` — employer pages often have huge footers, navs, and related listings that bloat the output and can blow out the context window.
4. Save to `DATA_DIR/jobs/[company-slug]-[date]/posting.md` with the employer URL at the top

For **Medium-fit** jobs, try to resolve the employer URL but don't save the full posting.

If you can't resolve the direct link for a job, note the company name so the user can find it themselves.

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
      "mcp__claude-in-chrome__*",
      "mcp__Apify__*"
    ]
  }
}
```
