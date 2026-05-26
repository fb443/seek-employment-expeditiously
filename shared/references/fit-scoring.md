# Fit Scoring Criteria

Standard scoring used across all SEE skills for evaluating job-candidate fit.

## Evaluation Process

### 0. Title Pre-Screen (fast pass)

Before reading the full job description, evaluate the title alone against the candidate's profile. This saves time by filtering out obvious mismatches.

**Check seniority signal in title:**

| Title keyword | Implied level |
|---|---|
| Intern, Junior, Associate, Entry | Entry / Junior |
| Mid, (no level modifier) | Mid |
| Senior, Lead | Senior IC |
| Staff, Principal, Distinguished | Staff+ IC |
| Manager, Senior Manager | Manager |
| Director, Senior Director, Head of | Director |
| VP, SVP, EVP, C-level | Executive |

If the implied level is **more than one band** from the candidate's target → **Skip** without reading the description.

**Check function match:**
Compare the role function in the title (e.g., "Engineering", "Marketing", "Sales", "Data Science") against the candidate's target roles and skill set. If the function is completely unrelated (e.g., candidate targets engineering roles, title is "Director of Sales") → **Skip**.

**Pass forward:** Titles that are ambiguous, close enough, or clearly aligned proceed to full evaluation starting at Step 1.

### 1. Check Dealbreakers First

Immediately mark as **Skip** if ANY dealbreaker from `preferences.md` is present:
- Company type (agency, crypto, etc.)
- Location requirements
- Travel requirements
- Clearance requirements
- Salary below minimum threshold

### 2. Check Seniority Band

Compare the job's level against the candidate's target level from their profile.

| Candidate Target | Acceptable Job Levels |
|---|---|
| VP / Head of | VP, Head of, Senior Director |
| Director | Director, Senior Director, Head of |
| Senior Manager | Senior Manager, Manager, Director |
| Senior IC | Senior, Staff, Principal |
| Mid IC | Mid, Senior |

If the job level falls **more than one band away** from the candidate's target (e.g., VP candidate → IC role, or junior role → Director candidate), mark as **Skip** with note "seniority mismatch."

### 3. Check Education & Experience Requirements

Compare the job's stated requirements against the candidate's actual credentials:

- **Degree match**: Does the candidate meet the degree requirement (or exceed it)?
  - Required PhD and candidate has PhD → full credit
  - Required Master's and candidate has Master's or PhD → full credit
  - Required Bachelor's and candidate has any degree → full credit
  - Degree preferred but not required → full credit regardless
  - Degree required but candidate lacks it → **-20 points** (may still pass on experience)
- **Years of experience**: Compare required years to candidate's relevant experience
  - Meets or exceeds → full credit
  - Within 2 years below → **-10 points**
  - More than 2 years below → **-20 points**
  - More than 2 years above → note "overqualified" (not penalized but flagged)
- **Domain experience**: Has the candidate worked in this industry before?
  - Direct industry match in last 5 years → **+10 bonus**
  - Adjacent industry → no adjustment
  - No overlap → **-5 points**

### 4. Score Skills Alignment

Compare the job's required and preferred skills against the candidate's skill set. Use the years-of-experience data from the candidate's profile to gauge depth, and weight recent experience (last 5 years) more heavily than older experience.

**Required skills overlap:**

| Overlap | Points |
|---|---|
| 80%+ of required skills matched | 30 |
| 60-79% matched | 20 |
| 40-59% matched | 10 |
| Below 40% matched | 0 |

**Experience depth modifier:** When a job specifies minimum years for a skill (e.g., "5+ years Python"), compare against the candidate's years for that skill. Apply per-skill:
- Meets or exceeds required years → no adjustment
- Within 2 years below → **-2 points** per skill
- More than 2 years below → **-5 points** per skill

**Preferred skills bonus:** +1 point per preferred skill matched, up to +10.

When matching skills, treat equivalent technologies as matches (e.g., "PostgreSQL" matches "relational databases"; "React" matches "modern frontend frameworks"). Use judgment — the goal is to assess real capability, not keyword bingo.

### 5. Score Must-Haves

Count how many remaining must-have criteria from `preferences.md` are met:
- All met → **20 points**
- Most met (75%+) → **15 points**
- Some met (50-74%) → **10 points**
- Few met (<50%) → **0 points**

### 6. Score Nice-to-Haves

For jobs passing must-haves, evaluate nice-to-have criteria:
- Company stage/funding
- Industry alignment
- Growth potential
- Role scope

Score: **+3 points** per nice-to-have met, up to **+15**.

### 7. Apply Feedback Adjustments

If `DATA_DIR/feedback.md` exists, apply learned adjustments before computing the final score:

- **Disliked patterns**: If the job matches a pattern the user previously flagged as unwanted (e.g., "healthcare companies", "marketing-disguised-as-growth roles"), apply a **-15 penalty** or treat as a soft dealbreaker.
- **Liked patterns**: If the job matches patterns from thumbs-up feedback (e.g., "product-oriented roles", "Series B+ startups"), apply a **+5 bonus**.
- **Title reinterpretation**: If feedback notes that certain titles don't mean what they seem (e.g., "Growth Lead at agencies = marketing"), read the description more carefully before scoring rather than relying on the title pre-screen.

These adjustments are cumulative with the base score.

### 8. Compute Final Score & Assign Rating

Sum all points (max ~105, before feedback adjustments):

| Rating | Score | Criteria |
|---|---|---|
| **High** | 70+ | Strong alignment across seniority, skills, experience, and preferences |
| **Medium** | 45-69 | Solid match with some gaps — worth reviewing |
| **Low** | 25-44 | Significant gaps in skills, experience, or preferences |
| **Skip** | <25 or any dealbreaker | Dealbreaker present or too many gaps to be viable |

When presenting results, include the numeric score alongside the rating so the user can compare within tiers (e.g., a 92 High vs. a 71 High).
