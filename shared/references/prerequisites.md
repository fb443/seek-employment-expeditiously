# Prerequisites by Skill

Check that required data files exist before proceeding. If any required file is missing, show the failure message and stop.

## Required Files

| File | create_profile | enhance_profile | find_jobs | tweak_resume | generate_cover_letter | apply | track_applications |
|------|:--------------:|:---------------:|:---------:|:------------:|:---------------------:|:-----:|:------------------:|
| `DATA_DIR/resume/*` | — | Required | Required | Required | Required | Required | — |
| `DATA_DIR/preferences.md` | — | — | Required | Read if exists | Read if exists | — | — |
| `DATA_DIR/profile.md` | — | — | — | Recommended | Recommended | — | — |
| `DATA_DIR/application-data.md` | — | — | — | — | — | Created if missing | — |
| `DATA_DIR/feedback.md` | — | — | Read if exists | Read if exists | Read if exists | — | Created if missing |
| `DATA_DIR/style-feedback.md` | — | — | — | Read if exists | Read if exists | — | — |

## Failure Messages

- **Resume missing**: "Run `/see:create_profile` first to upload your resume."
- **Preferences missing**: "Run `/see:create_profile` first to configure your resume and preferences."
- **Profile missing (tweak_resume)**: Warn that the resume will be based only on resume text and may require more corrections. Recommend running `/see:enhance_profile` first for a 15-minute interview that makes tailored resumes much stronger. Allow the user to proceed if they choose.
- **Profile missing (generate_cover_letter)**: Warn that the cover letter will be based only on the resume. Recommend running `/see:enhance_profile` first for better results. Proceed anyway.
- **No applications (track_applications)**: Not an error. Show empty dashboard and suggest running `/see:find_jobs` to start.
