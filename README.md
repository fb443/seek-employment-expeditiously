# Seek Employment eXpeditiously (SEE)

The job search agent that does everything but show up to the interview. SEE searches multiple sources in parallel, tailors your resume and cover letter for every role, auto-fills applications on Greenhouse, Lever, and Workday, and tracks your full pipeline from first search to offer. It learns from your feedback on every run - your preferences sharpen search results, your edits refine future resumes and cover letters, and your pipeline data reveals what's working and what to fix.

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| [Create Profile](./skills/create_profile/) | `/see:create_profile` | Quick setup: upload resume and set preferences (~5 min) |
| [Enhance Profile](./skills/enhance_profile/) | `/see:enhance_profile` | Deep work history interview for better resumes and cover letters (~15 min) |
| [Find Jobs](./skills/find_jobs/) | `/see:find_jobs` | Search online aggregators and company sites, score and rank matches, save top postings |
| [Tweak Resume](./skills/tweak_resume/) | `/see:tweak_resume` | Rewrite your resume for a specific job posting using your work history |
| [Generate Cover Letter](./skills/generate_cover_letter/) | `/see:generate_cover_letter` | Write a cover letter that connects your achievements to the employer's needs |
| [Apply](./skills/apply/) | `/see:apply` | Tailor resume, generate cover letter, and fill the application form end-to-end autonomously |
| [Track Applications](./skills/track_applications/) | `/see:track_applications` | Dashboard, pipeline analysis, and feedback to improve future searches |

## How They Work Together

```
create_profile (~5 min, required)
       ↓
   find_jobs ←────────────────┐
       ↓                      │
  enhance_profile (optional,  │ feedback improves
  unlocks better materials)   │ future searches
       ↓                      │
  tweak_resume ←──────────────│ pipeline results inform 
       ↓                      │ resume + cover letter generation
 generate_cover_letter ←──────│
       ↓                      │
     apply                    │
       ↓                      │
 track_applications ──────────┘
```

1. **`/see:create_profile`** uploads your resume and configures preferences — takes about 5 minutes, and you're ready to search
2. **`/see:find_jobs`** searches multiple sources, suggests adjacent roles you might not have considered, scores each match, and saves the best postings. After showing results, it asks what you thought — your feedback is saved and applied to future searches automatically.
3. **`/see:enhance_profile`** (optional but recommended) — a 15-minute interview that captures the accomplishments, motivations, and career narrative behind your resume. This is what makes tailored resumes and cover letters genuinely compelling instead of generic. You can do this anytime.
4. **`/see:tweak_resume`** fetches the job posting, maps your experience to the requirements, fills gaps with you, and generates a tailored resume
5. **`/see:generate_cover_letter last`** writes a cover letter connecting 2-3 of your achievements to the employer's specific needs
6. **`/see:apply last`** generates any missing materials (resume, cover letter), then fills out the application form on Greenhouse, Lever, or Workday
7. **`/see:track_applications`** shows where every application stands, computes your funnel (found → applied → response → interview → offer), diagnoses where things are stuck, and suggests what to change. You can also review past search results you didn't comment on at the time.

**Everything gets smarter over time.** Your feedback flows across skills: when you tell `find_jobs` which results were off the mark, it updates preferences and adjusts scoring. Those same preferences inform how `tweak_resume` and `generate_cover_letter` frame your experience — emphasizing the themes you're actually drawn to. When you edit a generated resume or cover letter before submitting, those corrections are captured and applied to future materials automatically. `track_applications` adds pipeline-level insight on top: if your response rate is low, it can identify possible causes and suggest fixes.

All skills share a `~/.see/` directory for personal files. Each job application gets its own folder containing the posting, tailored resume, and cover letter.

## Installation

### Option A: Claude Cowork (desktop app)

1. Download [Claude Cowork](https://claude.com/product/cowork) if you haven't already
2. Download the plugin as a zip from GitHub: [Download ZIP](https://github.com/fb443/seek-employment-expeditiously/archive/refs/heads/main.zip)
3. In Cowork, go to **Plugins** (left sidebar) and click the **+** button
4. Select **Upload plugin**
5. Drag and drop the downloaded zip file, then click **Upload**
6. Run `/see:create_profile` to get started

### Option B: Claude Code CLI

First, add the repository as a marketplace:

```bash
claude plugin marketplace add https://github.com/fb443/seek-employment-expeditiously.git
```

Then install the plugin:

```bash
claude plugin install see@see
```

Then run setup:

```
/see:create_profile
```

### After installing

Setup will create `~/.see/`, prompt you for your resume, and configure your job preferences. You'll be searching in under 5 minutes.

When you're ready to get the most out of resume tailoring and cover letters, run `/see:enhance_profile` for a deeper interview.

You can also add your resume manually first:

```bash
mkdir -p ~/.see/resume
cp /path/to/your/resume.pdf ~/.see/resume/
```

## Prerequisites

- [Claude Cowork](https://claude.com/product/cowork) desktop app **or** [Claude Code CLI](https://claude.ai/code)
- [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome) extension (for browser automation and applying to jobs)
- Chrome browser running with the extension active
- **Optional but recommended:** [Apify](https://apify.com) connector for faster, more reliable job search results. The plugin will offer to help you connect it on first run if it's not already set up.

## File Structure

**Plugin (installed via marketplace):**
```
seek-employment-expeditiously/
├── .claude-plugin/
│   └── plugin.json                     # Plugin manifest
├── shared/
│   ├── templates/
│   │   └── profile.md                  # Work history profile template
│   └── references/
│       ├── fit-scoring.md              # Canonical fit scoring criteria
│       ├── data-directory.md           # Data directory resolution algorithm
│       ├── prerequisites.md            # Prerequisites checking by skill
│       ├── browser-setup.md            # Browser automation setup sequence
│       ├── ats-patterns.md            # ATS navigation patterns (Greenhouse, Lever, Workday)
│       └── priority-hierarchy.md       # Instruction priority hierarchy
├── skills/
│   ├── create_profile/
│   │   └── SKILL.md
│   ├── enhance_profile/
│   │   ├── SKILL.md
│   │   └── scripts/
│   ├── find_jobs/
│   │   ├── SKILL.md
│   │   ├── assets/templates/
│   │   └── scripts/
│   ├── tweak_resume/
│   │   ├── SKILL.md
│   │   └── scripts/
│   ├── generate_cover_letter/
│   │   ├── SKILL.md
│   │   └── scripts/
│   ├── apply/
│   │   ├── SKILL.md
│   │   └── scripts/
│   └── track_applications/
│       └── SKILL.md
└── README.md
```

**User data (created by `/see:create_profile`, persists across plugin updates):**
```
~/.see/
├── resume/                             # Your resume PDF/DOCX
├── profile.md                          # Work history from /see:enhance_profile
├── preferences.md                      # Job matching rules
├── job-history.md                      # Running log from find_jobs
├── application-data.md                # Reusable form field answers
├── applications.md                    # Aggregated application tracker
├── feedback.md                        # Search feedback for improving future results
├── style-feedback.md                  # Writing corrections from user edits to resumes/cover letters
└── jobs/                               # One folder per application
    ├── google-lead-gpm-2026-02-11/
    │   ├── posting.md                  # Saved job description
    │   ├── resume.md                   # Tailored resume
    │   ├── cover-letter.md             # Cover letter
    │   └── applied.md                  # Application log (date, ATS, status)
    └── ...
```

## Support

If this repo is helping your job search, star it on GitHub! It helps others find it. Click the **Star** button in the top-right corner of the main repository page (you'll need a free GitHub account).

Found a bug or have a feature idea? Click **Issues** at the top of the page, then **New issue**.

## About

This plugin is free and open source.

## License

MIT
