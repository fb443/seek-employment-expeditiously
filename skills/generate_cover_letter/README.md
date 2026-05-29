# Cover Letter Skill for Claude Code

Write natural, persuasive cover letters tailored to specific job postings. Works alongside the [tweak_resume](../tweak_resume/) and [find_jobs](../find_jobs/) skills.

## Features

- **Authentic voice** - sounds like a real professional, not AI
- **Achievement-focused** - connects 2-3 measurable results to the employer's specific needs
- **Strict honesty** - never fabricates or exaggerates any detail from the resume
- **Works with tailored resumes** - leverages existing match analysis when available

## Prerequisites

See the [main README](../../README.md) for installation and prerequisites. Resume must be set up via `/see:create_profile`. For stronger cover letters, run `/see:enhance_profile` first.

## Usage

### Write a cover letter for a job

```bash
claude "/see:generate_cover_letter https://example.com/jobs/vp-growth"
```

### Use the most recent tailored resume

```bash
claude "/see:generate_cover_letter last"
```

### General flow

```bash
claude "/see:generate_cover_letter"
```

## How It Works

1. Reads your resume and work history profile
2. Gets the job posting (from URL or most recent tailored resume)
3. Identifies 2-3 achievements that directly address the employer's needs
4. Writes a 250-350 word cover letter in a natural, conversational tone
5. Saves to `~/.see/jobs/[company-slug]/` for your review

## Tips

- Run `/see:tweak_resume` first, then `/see:generate_cover_letter last` to get a cover letter that matches your tailored resume
- The work history interview makes cover letters significantly better since there's more material to draw from
- Every claim in the cover letter is verified against your actual resume and work history
