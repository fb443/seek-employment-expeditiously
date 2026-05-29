# Job Search Skill for Claude Code

An automated job search skill that finds and evaluates job listings using parallel subagents across structured APIs (Apify) and browser automation (Hiring.cafe, Google Jobs). Automatically detects which sources are available and searches them simultaneously.

## Features

- **Parallel search** — runs all available sources concurrently via subagents, significantly faster than sequential search
- **Multi-source coverage** across Apify Google Jobs Scraper, Hiring.cafe, and Google Jobs
- **Smart source detection** — uses Apify for fast structured results when available, falls back to browser automation, offers to connect Apify if missing
- **Expanded search terms** — suggests skills-based and adjacent role queries beyond your stated targets
- **Fit scoring** with numeric scores based on seniority, skills, experience, and preferences
- **Smart filtering** based on salary, location, dealbreakers, and learned patterns
- **Job history tracking** to avoid showing duplicates
- **Feedback loop** — after every search, asks what you thought. Your feedback updates preferences and adjusts future scoring automatically.

## Prerequisites

See the [main README](../../README.md) for installation and prerequisites.

## Installation

Install via Claude Code CLI:

```bash
claude plugin marketplace add https://github.com/fb443/seek-employment-expeditiously.git
claude plugin install see@see
```

Then run setup:

```bash
claude "/see:create_profile"
```

This will configure your resume, preferences, and work history profile.

## Usage

### Daily search (manual)
```bash
claude "/see:find_jobs"
```

### Search with specific keywords
```bash
claude "/see:find_jobs AI infrastructure"
claude "/see:find_jobs remote startup"
```

### Run headless (for cron)
```bash
claude -p "/see:find_jobs"
```

## Architecture

The skill uses **source-level parallelism** — each search source runs as an independent subagent:

```
SKILL.md (orchestrator)
  ├── Load context & expand search terms
  ├── Detect available sources
  ├── Launch subagents in parallel:
  │     ├── search-apify.md        (Apify Google Jobs Scraper)
  │     ├── search-hiring-cafe.md  (browser: hiring.cafe)
  │     └── search-google-jobs.md  (browser: Google Jobs)
  ├── Merge & deduplicate results
  ├── Score & rank (evaluate-jobs.md)
  ├── Resolve employer URLs for top matches
  └── Present results & collect feedback
```

Each subagent returns a standardized JSON array, making merge/dedup straightforward.

## File Structure

**Plugin files:**
```
find_jobs/
├── SKILL.md                      # Orchestrator
├── README.md                     # This file
├── assets/
│   └── templates/                # Format templates (committed)
│       └── job-entry.md          # Format for history entries
└── scripts/
    ├── search-apify.md           # Apify search subagent
    ├── search-hiring-cafe.md     # Hiring.cafe browser subagent
    ├── search-google-jobs.md     # Google Jobs browser subagent
    └── evaluate-jobs.md          # Job scoring subagent
```

**User data (at `~/.see/`):**
```
~/.see/
├── resume/                       # Your resume PDF/DOCX
├── preferences.md                # Job matching rules
├── job-history.md                # Log of all jobs found
├── feedback.md                   # Your feedback — adjusts future scoring
└── jobs/                         # Per-job application folders
```

## Configuration

### Matching Rules (`~/.see/preferences.md`)

Customize your job preferences:

```markdown
## Target Roles
- VP Growth
- Head of Growth
- Director of Marketing

## Must-Have Criteria
- Remote or hybrid OK
- Minimum $250k+ total comp

## Dealbreakers
- Marketing agencies
- Crypto/blockchain
- >25% travel required

## Nice-to-Have
- Series B+ startup
- AI/ML focus
- B2B SaaS
```

### Updating Preferences

Just tell Claude what you want:
- *"Add fintech to my nice-to-haves"*
- *"I don't want any roles requiring relocation"*
- *"Bump my minimum salary to $300k"*

The skill will update `~/.see/preferences.md` automatically.

## Job History

All jobs found are logged to `~/.see/job-history.md` with:
- Date and search terms
- Job details (title, company, location, salary)
- Fit score (High/Medium/Low/Skip)
- Notes explaining the rating

This prevents showing you the same jobs twice and creates a searchable archive.

## Cron Setup

To run daily at 9am:

```bash
# Add to crontab
(crontab -l 2>/dev/null; echo "0 9 * * * cd ~ && claude -p '/see:find_jobs' >> ~/.see/logs/find_jobs.log 2>&1") | crontab -
```

**Note**: Requires Chrome to be running with Claude in Chrome extension active.

## Troubleshooting

### Permission prompts interrupting cron
Ensure all permissions are in `~/.claude/settings.json` (see Installation step 3).

### Browser not responding
Make sure Chrome is running and Claude in Chrome extension is active.

### No jobs found
- If using Apify: check your Apify account has available compute units
- If using browser sources: check that hiring.cafe is accessible and Chrome is running
- Try different search terms
- Verify your matching rules aren't too restrictive

## License

MIT
