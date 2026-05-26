# Seek Employment Expeditiously (SEE)

A Claude Code plugin for AI-powered job searching, resume tailoring, and cover letter writing.

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| [Create Profile](./skills/create_profile/) | `/see:create_profile` | One-time onboarding: resume, preferences, and work history interview |
| [Find Jobs](./skills/find_jobs/) | `/see:find_jobs` | Automated job search with smart filtering |
| [Tweak Resume](./skills/tweak_resume/) | `/see:tweak_resume` | Create tailored resumes for specific job postings |
| [Generate Cover Letter](./skills/generate_cover_letter/) | `/see:generate_cover_letter` | Write natural, persuasive cover letters |
| [Apply](./skills/apply/) | `/see:apply` | Fill out job applications on Greenhouse, Lever, and Workday |

## How They Work Together

1. **`/see:create_profile`** uploads your resume, configures preferences, and conducts a work history interview (one-time)
2. **`/see:find_jobs`** finds jobs that match your preferences and resume
3. **`/see:tweak_resume`** rewrites your resume for a specific job posting, saves the job posting and tailored resume together
4. **`/see:generate_cover_letter last`** writes a cover letter using the most recent job's posting and tailored resume
5. **`/see:apply last`** fills out the application form on Greenhouse, Lever, or Workday using your tailored resume and cover letter

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

Setup will create `~/.see/`, prompt you for your resume, configure your job preferences, and conduct a work history interview.

You can also add your resume manually first:

```bash
mkdir -p ~/.see/resume
cp /path/to/your/resume.pdf ~/.see/resume/
```

## Prerequisites

- [Claude Cowork](https://claude.com/product/cowork) desktop app **or** [Claude Code CLI](https://claude.ai/code)
- [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome) extension (for browser automation)
- Chrome browser running with the extension active

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
│   └── apply/
│       ├── SKILL.md
│       └── scripts/
└── README.md
```

**User data (created by `/see:create_profile`, persists across plugin updates):**
```
~/.see/
├── resume/                             # Your resume PDF/DOCX
├── profile.md                          # Work history from interview
├── preferences.md                      # Job matching rules
├── job-history.md                      # Running log from find_jobs
├── application-data.md                # Reusable form field answers
└── jobs/                               # One folder per application
    ├── google-lead-gpm-2026-02-11/
    │   ├── posting.md                  # Saved job description
    │   ├── resume.md                   # Tailored resume
    │   ├── cover-letter.md             # Cover letter
    │   └── applied.md                  # Application log (date, ATS, status)
    └── ...
```

## About

This plugin is free and open source.

## License

MIT
