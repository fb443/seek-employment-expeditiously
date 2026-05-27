# Application Tracker for Claude Code

Track every application, diagnose pipeline problems, and improve future searches with feedback.

## Prerequisites

See the [main README](../../README.md) for installation and prerequisites.

## Usage

### View your dashboard
```bash
claude "/see:track_applications"
```

### Log a status change
```bash
claude "/see:track_applications update"
```
Then tell it what happened: "Google rejected me", "phone screen with Stripe next Tuesday", "got an offer from Figma."

### Review a single application
```bash
claude "/see:track_applications Stripe"
```

### Give feedback on past search results
```bash
claude "/see:track_applications feedback"
```
Rate past results as thumbs-up or thumbs-down. Your feedback adjusts preferences and scoring for future `/see:find_jobs` runs.

## What It Does

1. **Scans** all job folders in `~/.see/jobs/` to build a unified tracker
2. **Groups** applications by urgency — what needs follow-up, what's in progress, what's closed
3. **Analyzes** your funnel (found → applied → response → interview → offer) and diagnoses where things are stuck
4. **Recommends** specific actions based on where your pipeline is weakest
5. **Collects feedback** on past search results to improve future job matching

## Pipeline Analysis

The skill computes conversion rates between each stage and flags the biggest drop-off. It then suggests likely causes and what to do — distinguishing between things SEE can fix (resume keywords, search terms, application timing) and things you need to handle yourself (interview practice, networking, asking for feedback).

## Feedback Loop

Most feedback happens automatically — `/see:find_jobs` asks what you thought after every search and saves your responses to `feedback.md`. That file is read on future searches to adjust scoring and filtering.

`/see:track_applications feedback` is for going back and reviewing results you didn't comment on at the time, or revisiting after you've had time to think.

## License

MIT
