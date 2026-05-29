# Data Directory Resolution

All user data lives in a `.see/` folder. Follow these steps to find it:

## Resolution Algorithm

1. Check the current working directory for `.see/` — use it if found
2. Check `~/.see/` — use it if found
3. If neither exists:
   - **create_profile skill**: this is a fresh setup — create it in Step 1
   - **all other skills**: tell the user to run `/see:create_profile` first, then stop

## Ephemeral Session Warning

If no folder is selected (i.e. the working directory looks like an ephemeral session path such as `/sessions/...`), stop and tell the user:

> "Before we start, you need to select a folder so your data persists between sessions. Click 'Work in a folder' and select your home directory, then try again."

Do NOT proceed without a persistent folder.

## DATA_DIR Tree

All paths in skill instructions use `DATA_DIR` to mean whichever `.see/` directory was found or created.

```
DATA_DIR/
  resume/              # Your resume PDF/DOCX
  preferences.md       # Job matching rules
  profile.md           # Work history from /see:enhance_profile interview
  jobs/                # Per-job application folders
  job-history.md       # Running log from find_jobs
  applications.md      # Aggregated application tracker
  feedback.md          # Search feedback for improving future results
  style-feedback.md    # Writing style corrections from user edits to resumes/cover letters
```
