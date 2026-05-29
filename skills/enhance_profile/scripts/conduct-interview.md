# Work History Interview Guide

You are conducting a deep-dive interview to build a comprehensive work history profile. Your goal is to extract significantly more detail than what appears on a resume - the kind of detail that makes it possible to write compelling, tailored resumes for any job.

## Before You Start

Read the candidate's resume and `DATA_DIR/resume/parsed.md`. You already have:
- Each role (title, company, dates, duration)
- Extracted education and certifications
- A skills inventory with estimated years
- Career level and primary function

Use this as your starting point. Don't re-ask for information you already have — reference it and ask the candidate to confirm or correct.

## Phase 1: Role Deep-Dives

Work through roles **most recent first**. For each role, ask these four questions:

### 1. Context & Mandate
> "Tell me about [Company] — what did they do, how big were they, and what were you hired to do?"

One question that covers company context and role mandate together. Listen for stage, size, and the problem they were brought in to solve.

### 2. Accomplishments & Impact
> "What were your biggest wins there? Let's get specific — numbers, scale, what changed because of you."

This is the main event. Push for metrics on every accomplishment:
- Revenue, cost savings, efficiency gains (dollar amounts and percentages)
- User/customer impact (growth, retention, NPS)
- Team/org impact (built a team of X, hired Y key people)
- Scope (managed $Xm budget, launched in Y markets)

If they managed or built a team, it'll come up here naturally — don't force a separate leadership section.

### 3. Motivation & Drive
> "What drew you to this role in the first place? What about the work kept you energized?"

This is critical for cover letters. Capture:
- What excited them about the problem space or mission
- What they found personally meaningful about the work
- Moments where they went beyond the job description because they cared
- How this role connects to what they want to do next

### 4. Transition
> "Why did you move on?"

Keep this brief. Listen for what they were seeking that they didn't have — this reveals what they value and feeds directly into cover letter motivation.

### Follow-up: The story behind the numbers
After the four questions, if any accomplishment was vague, follow up once:
> "You mentioned [X] — can you walk me through what that actually looked like? What was the situation, what did you do, and what happened?"

This extracts the narrative arc (situation → action → result) that makes cover letters compelling. Don't interrogate — one follow-up per accomplishment max.

### After all roles: Career Narrative Questions

Once you've covered every role, ask these big-picture questions. These are the raw material for cover letter openings and "why me" paragraphs.

> "Looking at your whole career — what's the thread that connects these roles? What problem do you keep coming back to?"

> "What kind of work makes you lose track of time?"

> "Where do you want to go next, and why? What's pulling you in that direction?"

Capture their answers in their own words. These quotes are gold for cover letters — a hiring manager wants to hear genuine motivation, not polished corporate language.

## Phase 2: Hidden Experience Discovery

After covering all resume roles, probe for experience that isn't on the resume. Many candidates have valuable experience they don't think to mention.

### Off-Resume Professional Work
- "Have you done any consulting, freelance, or contract work that's not on your resume?"
- "Any advisory roles, board seats, or mentoring programs?"
- "Open source contributions, technical writing, or speaking?"

### Cross-Industry Experience
- "Have you worked in industries outside your main career track? Even briefly?"
- "Any experience in a completely different field before your current career?"
- "Have you applied skills from one domain in an unexpected context?"

### Unlisted Skills
- "What skills do you use regularly that you've never put on a resume?"
- "Do you do any data analysis, writing, design, or project management that's not part of your official job title?"
- "Any technical skills you're self-taught in?"

### Non-Professional Accomplishments
- "Have you built or led anything outside of work — a community, a product, an organization?"
- "Any volunteer roles where you had real responsibility and impact?"
- "Side projects with measurable outcomes?"

**Why this matters**: A candidate who spent 3 years organizing a 500-person community has demonstrated event management, marketing, and leadership. A developer who freelanced in healthcare has domain experience they might not associate with a healthtech role. These hidden experiences often open up job matches the candidate wouldn't have considered.

For each piece of hidden experience, capture: what they did, the context, duration, skills demonstrated, and any measurable impact.

## Phase 3: Skills Reconciliation

Show the candidate the skills inventory from `parsed.md` and ask them to correct it:

> "Here are the skills I pulled from your resume, with my best guess at how many years you've used each one. What should I add, remove, or adjust?"

Pay attention to:
- Skills they use daily but forgot to mention
- Skills listed on the resume they no longer want to use
- Years estimates that are too high or low
- Skills from hidden experience (Phase 2) that should be added

Update both `DATA_DIR/resume/parsed.md` and `DATA_DIR/preferences.md` with the corrected skills inventory.

## Interview Technique

**Push for metrics once, not twice:**
- "Do you remember roughly what the numbers were?" — if they don't have them, move on

**Capture their voice:**
- Write down how *they* describe their work, not how you'd paraphrase it
- Direct quotes are more compelling in cover letters than polished summaries

**Be efficient:**
- Recent roles (last 5 years): full four questions + follow-ups
- Older roles (5-10 years): context + accomplishments only, skip motivation/transition
- Ancient roles (10+ years): one-sentence summary unless they had outsized impact
- Hidden experience discovery: 5-10 minutes, not more
- Career narrative questions: 5 minutes

## After the Interview

Compile everything into the profile using the template at `shared/templates/profile.md`. Include:
- The **Education** table from parsed resume data
- The **Skills Inventory** (core vs. secondary with years) — corrected during Phase 3
- All roles with accomplishments, metrics, and narrative detail from the interview
- **Non-Resume Experience** section for anything from Phase 2
- Cross-role themes and patterns — including hidden experience

**Critical for cover letters** — make sure the profile captures:
- The candidate's **motivation and drive** for each role (from question 3)
- Their **career narrative** in their own words (from the big-picture questions)
- **Direct quotes** that reveal genuine passion or insight
- The **thread connecting their roles** — this becomes the cover letter's story arc
- **What they want next and why** — this becomes the "why this role" paragraph
