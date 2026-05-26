# Prerequisites by Skill

Check that required data files exist before proceeding. If any required file is missing, show the failure message and stop.

## Required Files

| File | create_profile | find_jobs | tweak_resume | generate_cover_letter | apply | track_applications |
|------|:--------------:|:---------:|:------------:|:---------------------:|:-----:|:------------------:|
| `DATA_DIR/resume/*` | — | Required | Required | Required | Required | — |
| `DATA_DIR/preferences.md` | — | Required | — | — | — | — |
| `DATA_DIR/profile.md` | — | — | Recommended | Recommended | — | — |
| `DATA_DIR/linkedin-contacts.csv` | — | — | — | — | — | — |
| `DATA_DIR/application-data.md` | — | — | — | — | Created if missing | — |
| `DATA_DIR/feedback.md` | — | Read if exists | — | — | — | Created if missing |

## Failure Messages

- **Resume missing**: "Run `/see:create_profile` first to upload your resume."
- **Preferences missing**: "Run `/see:create_profile` first to configure your resume and preferences."
- **Profile missing (tweak_resume)**: Warn that the resume will be based only on resume text and may require more corrections. Recommend running `/see:create_profile interview` first. Allow the user to proceed if they choose.
- **Profile missing (generate_cover_letter)**: Warn that the cover letter will be based only on the resume. Recommend running `/see:create_profile interview` first. Proceed anyway.
- **No applications (track_applications)**: Not an error. Show empty dashboard and suggest running `/see:find_jobs` to start.
