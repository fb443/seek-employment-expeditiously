# Resume Tailoring Skill for Claude Code

Create tailored resumes that make you the obvious candidate for any specific job posting. Uses your resume and work history profile to craft compelling, targeted resumes.

## Features

- **Job posting analysis** - fetches and parses job details from any URL
- **Intelligent tailoring** - rearranges, rewrites, and emphasizes the right experience
- **Level-appropriate framing** - calibrates language and emphasis to match the role's seniority
- **Assumption tracking** - flags guesses when no work history profile exists
- **Feedback-informed framing** - reads your search feedback and preferences to emphasize themes you're drawn to
- **Learns from your edits** - after each resume, asks what you changed. Your corrections are applied to future resumes automatically

## Prerequisites

See the [main README](../../README.md) for installation and prerequisites. Resume must be set up via `/see:create_profile`. For much stronger tailored resumes, run `/see:enhance_profile` first.

## Usage

### Tailor resume for a job

```bash
claude "/see:tweak_resume https://example.com/jobs/vp-growth"
```

### General flow

```bash
claude "/see:tweak_resume"
```

This will check prerequisites, then ask for a job URL.

## File Structure

**Plugin files:**
```
tweak_resume/
├── SKILL.md                          # Main skill definition
├── README.md                         # This file
└── scripts/
    └── tailor-resume.md              # Tailoring agent prompt
```

**User data (at `~/.see/`):**
```
~/.see/
├── resume/                           # Your resume PDF/DOCX
├── profile.md                        # Work history from interview
├── preferences.md                    # Job preferences (for context)
└── jobs/
    └── [company-slug]/
        ├── posting.md                # Saved job description
        └── resume.md                 # Tailored resume
```

## How It Works

1. **Checks prerequisites** - resume must exist (via `/see:create_profile`); work history profile from `/see:enhance_profile` is recommended
2. **Loads feedback context** - reads search preferences, liked patterns, and past style corrections
3. **Fetches the job posting** via browser automation
4. **Maps your experience** to the job's requirements, informed by what themes you value
5. **Generates a tailored resume** with reordered bullets, rewritten descriptions, and a targeted summary
6. **Saves the output** for your review and iteration
7. **Captures your edits** - asks what you changed before submitting, stores corrections for next time

## Tips

- The work history interview is the biggest unlock. A 15-minute conversation gives dramatically better tailored resumes.
- You can iterate on any generated resume - ask to adjust tone, emphasis, or specific bullets.
- Tailored resumes are saved with the company name and date, so you can track what you've sent where.
- The skill never fabricates experience - it reorganizes and reframes what's real.
- Your search feedback matters here too: if you consistently like AI/ML roles in find_jobs, your resume will lead with ML-relevant accomplishments.
