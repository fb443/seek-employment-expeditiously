---
name: track_applications
description: See where every application stands, what needs follow-up, and how your search is going
argument-hint: "'update' to log changes, or a company name for details"
---

# Track Applications Skill

> **Priority hierarchy**: See `shared/references/priority-hierarchy.md` for conflict resolution.

Track every application, diagnose pipeline problems, and learn from feedback to improve future job searches.

## Quick Start

- `/see:track_applications` - Dashboard view with pipeline analysis
- `/see:track_applications update` - Log status changes ("heard back from Stripe", "rejected from Google")
- `/see:track_applications [company]` - Deep dive on a single application
- `/see:track_applications feedback` - Review past results and tell us what you liked/didn't like

## Data Directory

Resolve the data directory using `shared/references/data-directory.md`.

---

## Workflow

### Step 0: Check Prerequisites

Resolve the data directory. The only requirement is that `DATA_DIR` exists. This skill is useful even with zero applications — it will show prepared-but-not-applied entries and suggest next steps.

### Step 1: Scan Job Folders

Read every subdirectory in `DATA_DIR/jobs/`. For each folder, check what exists:

| File | What it tells us |
|------|-----------------|
| `posting.md` | Role title, company, link, when saved |
| `resume.md` | Tailored resume was generated |
| `cover-letter.md` | Cover letter was generated |
| `applied.md` | Application was submitted (has date, ATS, status) |

Classify each job folder:

- **Applied** — has `applied.md` with Status: Submitted
- **Prepared** — has `posting.md` and/or `resume.md` but no `applied.md` (or Status: Draft)
- **Saved** — has only `posting.md`

Also read `DATA_DIR/job-history.md` to count total jobs found vs. applied.

### Step 2: Build/Refresh Tracker

Create or update `DATA_DIR/applications.md`. For each applied job, track:

```markdown
# Application Tracker

*Last updated: [DATE]*

| Company | Role | Applied | Status | Last Update | Follow-up Due | Notes |
|---------|------|---------|--------|-------------|---------------|-------|
| [company] | [title] | [date] | [status] | [date] | [date or —] | [notes] |
```

**Status values** (in pipeline order):
- Applied
- Phone Screen
- Interview
- Final Round
- Offer
- Accepted
- Rejected
- No Response
- Withdrawn

**Follow-up due** calculation:
- Applied with no response → 7 days after application date
- After phone screen → 3 days after the screen
- After interview → 3 days after the interview
- Offer received → note expiration if mentioned
- Rejected / Accepted / Withdrawn → no follow-up (—)

Merge with any existing `applications.md` — preserve user-entered notes and status updates.

### Step 3: Route by Argument

**If `$ARGUMENTS` is empty** → Step 4 (Dashboard)
**If `$ARGUMENTS` is "update"** → Step 6 (Update statuses)
**If `$ARGUMENTS` is "feedback"** → Step 8 (Feedback review)
**If `$ARGUMENTS` is a company name** → Step 7 (Single application detail)

### Step 4: Dashboard

Group applications by urgency:

```
🔴 Needs attention
  - [Company] [Role] — applied [X] days ago, no response. Follow up?
  - [Company] [Role] — interview was [X] days ago. Send thank-you?

🟡 In progress
  - [Company] [Role] — phone screen scheduled [date]
  - [Company] [Role] — take-home due [date]

🟢 Waiting
  - [Company] [Role] — applied [date], too early to follow up

⬜ Prepared (not yet applied)
  - [Company] [Role] — resume tailored, ready to apply

⚫ Closed
  - [Company] [Role] — [Rejected/Withdrawn/Accepted] on [date]
```

Below the dashboard, show summary stats:
- Total found / applied / response / interview / offer
- Response rate (applications that got any response / total applied)
- Active pipeline count

### Step 5: Pipeline Analysis

This is the core value of the skill. Compute funnel metrics and diagnose problems.

**Funnel:**

```
Found → Applied → Response → Interview → Offer
 [n]      [n]       [n]        [n]        [n]
       [%]        [%]        [%]         [%]
```

**Diagnose the biggest drop-off.** For the stage transition with the worst conversion, present:

1. **What's happening**: State the numbers plainly. "You applied to 18 roles and heard back from 5. That's a 28% response rate."

2. **Likely causes**: Based on the specific drop-off stage:

   **Found → Applied (low apply rate):**
   - Too many Low/Medium fits in results — search terms may be too broad
   - User may be cherry-picking too aggressively
   - Suggest: tighten preferences, adjust fit scoring, or lower selectivity

   **Applied → Response (high ghost rate):**
   - Resume may not be passing ATS keyword filters
   - Applying too long after posting (check dates)
   - Resume format may not be ATS-parseable
   - No referral or internal connection
   - Suggest: re-run `tweak_resume` with stronger keyword mirroring, apply within 48hr of posting, consider networking

   **Response → Interview (screened out after contact):**
   - Phone screen answers may be weak
   - Salary mismatch surfaced during screen
   - Resume may have oversold what the interview revealed
   - Suggest: interview prep, review resume accuracy

   **Interview → Offer (final round failures):**
   - Behavioral answers may lack depth
   - Technical gaps exposed in interview
   - Competing against candidates with more experience
   - Suggest: targeted practice, build portfolio projects, reframe thin experience using project stories

   **No offers after multiple final rounds:**
   - May be losing on negotiation, references, or internal competition
   - Suggest: ask interviewers for feedback, adjust target seniority

3. **Early-career patterns** — check for and flag these specifically:
   - "Overqualified for entry, underqualified for mid" — getting rejected from both ends. Narrow the seniority band or target new-grad programs.
   - Response rate <10% after 15+ applications — resume likely isn't passing ATS. Check formatting, add Skills section if missing, ensure keywords appear verbatim.
   - Responses but no offers — materials are working, problem is downstream. Suggest interview prep, not more resume tweaking.
   - Only hearing from roles they didn't apply to — profile is attracting a different market. Adjust preferences to match what's biting, or rewrite resume for what they actually want.

4. **What SEE can help with vs. what the user must do themselves:**

   ```
   What we can do:
   - Re-tailor resumes with better keyword matching
   - Adjust search terms and fit scoring
   - Draft follow-up emails (suggest /see:follow_up if built)
   - Check if resume format is ATS-friendly

   What you need to do:
   - Practice interview answers out loud
   - Ask rejected-at-interview companies for feedback
   - Build portfolio projects if technical gaps are the issue
   - Network — find internal referrals for target companies
   ```

5. **SEE-side self-critique** — look for patterns in our own output:
   - If tailored resumes without a Skills section correlate with higher ghost rates, suggest adding one (ties to tweak_resume structural suggestions)
   - If cover letter tone differs between responded vs. ghosted applications, note it
   - If the user applied >7 days after posting for ghosted apps, flag timing

**Present the analysis:**

```
📊 Pipeline Analysis — [DATE]

Found → Applied → Response → Interview → Offer
 42        18        5           2          0
       43%       28%         40%          0%

Biggest drop: Interview → Offer (0% on 2 attempts)

Likely cause: With [X] months of experience, behavioral answers 
may lack the depth interviewers expect. This is common for 
early-career candidates.

Recommended:
1. Prep for your next interview with /see:interview_prep [job URL]
2. Ask your [Company] interviewer for specific feedback
3. Consider framing class projects and side work as professional 
   experience — your profile has [N] non-resume items that could help
```

### Step 6: Update Statuses

Present the current tracker table. Ask the user what changed. Accept natural language:

- "Google rejected me"
- "Got a phone screen with Stripe for next Thursday"
- "Withdrawing from Airbnb"
- "Notion wants a second interview"
- "Got an offer from Figma — $120k base"

Parse each update:
1. Match to an existing application (fuzzy match on company name)
2. Update status in `DATA_DIR/applications.md`
3. Update the matching `DATA_DIR/jobs/[folder]/applied.md`
4. Recalculate follow-up dates

When a status changes, suggest the natural next action:
- → Phone Screen scheduled: "Want me to help you prep? (interview_prep skill coming soon)"
- → Interview completed: "Want me to draft a thank-you email?"
- → Rejected: "Sorry to hear. Want to run `/see:find_jobs` to refill the pipeline?"
- → Offer: "Congrats! Your target was $[X]k from preferences. Want to compare?"

### Step 7: Single Application Detail

Find the job folder matching `$ARGUMENTS` (fuzzy match on company name or folder name). Present everything:

```
## [Role] at [Company]

**Status**: [status] (last updated [date])
**Applied**: [date] via [ATS]
**Follow-up due**: [date]
**Job posting**: [employer URL]

**Materials:**
- Posting: DATA_DIR/jobs/[folder]/posting.md
- Tailored resume: [exists / not generated]
- Cover letter: [exists / not generated]

**Timeline:**
- [date] — Job found via /see:find_jobs (Fit: High, Score: 85)
- [date] — Resume tailored
- [date] — Cover letter written
- [date] — Applied via Greenhouse
- [date] — [any status updates]

**Next action**: [what to do based on current status]
```

### Step 8: Feedback Review

This step collects explicit feedback on past job search results to improve future searches. It writes to `DATA_DIR/feedback.md` and updates `DATA_DIR/preferences.md`.

**Present recent results for review:**

Pull the last 2-3 search runs from `DATA_DIR/job-history.md`. For each batch, show the High and Medium fits:

```
From your [DATE] search, here are the jobs I showed you:

1. VP of Growth at Acme Corp — Fit: High (85)
   Did you like seeing this result? [👍 / 👎 / didn't look]

2. Sr PM at StartupX — Fit: Medium (52)
   Did you like seeing this result? [👍 / 👎 / didn't look]

3. Growth Lead at BigCo — Fit: High (78)
   Did you like seeing this result? [👍 / 👎 / didn't look]
```

For each thumbs-down, ask: **"What was wrong with it?"**

Accept natural language reasons:
- "Too senior for me"
- "I don't want to work in healthcare"
- "The company is too small"
- "That role is really just marketing, not growth"
- "Salary was too low"
- "I actually applied and it was a great fit" (correction — mark as thumbs-up)

**Process feedback into three buckets:**

1. **Preference updates** → Write to `DATA_DIR/preferences.md`
   - New dealbreakers: "No healthcare" → add to Dealbreakers
   - New nice-to-haves: "Prefer companies with 100+ employees" → add to Nice-to-Haves
   - Seniority correction: "Too senior" → adjust target seniority band
   - Salary threshold: "Salary too low" → raise minimum

2. **Scoring adjustments** → Write to `DATA_DIR/feedback.md`
   These are patterns the fit scoring should weight differently. They don't change the algorithm but inform how find_jobs interprets scores:
   - "That role is really marketing, not growth" → Note: user distinguishes between growth and marketing roles even when titles overlap. Filter more aggressively on job description keywords, not just title.
   - "Company too small" → Note: user prefers companies above [X] employees. Add minimum company size as a soft filter.

3. **Search term refinements** → Write to `DATA_DIR/feedback.md`
   - If thumbs-up results cluster around certain keywords or industries, note them as high-signal
   - If thumbs-down results cluster around certain search terms, note those as producing noise

**Save feedback:**

Append to `DATA_DIR/feedback.md`:

```markdown
# Search Feedback

Feedback collected by /see:track_applications to improve future searches.
The /see:find_jobs skill reads this file to adjust scoring and filtering.

---

## [DATE] — Feedback on [search date] results

### Liked
- [Role] at [Company] (score [X]) — [reason if given]

### Disliked
- [Role] at [Company] (score [X]) — Reason: [user's reason]
  → Action taken: [what was updated in preferences.md or noted for scoring]

### Patterns Observed
- [e.g., "User consistently dislikes healthcare companies — added to dealbreakers"]
- [e.g., "User prefers product roles over marketing roles even when titles say 'Growth'"]
- [e.g., "Roles scored Medium (50-60) were universally disliked — consider raising the display threshold"]
```

After saving, confirm what was updated:

```
Got it. Based on your feedback:
- Added "healthcare" to your dealbreakers
- Noted that you prefer product-oriented growth roles over marketing-oriented ones
- Raised your minimum company size preference to 100+ employees

These will be applied next time you run /see:find_jobs.
```

---

## Response Format

Structure user-facing output with these sections:

1. **Dashboard** — applications grouped by urgency, with summary stats
2. **Pipeline Analysis** — funnel metrics, diagnosis, and recommendations
3. **Feedback** (if collected) — what was updated and how it affects future searches
4. **Next Steps** — suggest the most impactful action based on current pipeline state

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
      "Edit(~/.see/**)"
    ]
  }
}
```
