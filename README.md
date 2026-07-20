# MICA Schedule → iPhone Calendar Sync

Automatically filters the MICA PGP-2 Term-3 schedule down to 10 course codes
and publishes a live-updating calendar feed you can subscribe to on iPhone.

Runs on a 15-minute schedule via GitHub Actions — free, no server needed.

## Course codes currently filtered for (edit in `build_ics.py` → `TARGET_CODES`)
- F1-BDB, F1-OBM, F1-MAPC, F1-CAA, F1-TBO:CP
- F4-MTAI:TNOSM, F4-PMG:EFF, F4-BSCIAIW, F4-CI:CRMDGP, F4-MTAIP:EPF

## One-time setup

1. **Create a new GitHub repo** (public — GitHub Pages on the free tier
   requires a public repo unless you're on GitHub Pro/Team) and upload
   these files, keeping the folder structure:
   - `build_ics.py`
   - `requirements.txt`
   - `.github/workflows/update-calendar.yml`

2. **Add the SharePoint URL as a secret** (keeps it out of the public repo
   code, even though the link itself is "anyone with the link"):
   - Repo → Settings → Secrets and variables → Actions → New repository secret
   - Name: `SHAREPOINT_URL`
   - Value: the full SharePoint link to the .xlsx file (the one you tested
     opens without a sign-in prompt in an incognito window)

3. **Confirm the sheet name matches.** The workflow currently targets the
   tab named `Schedule Term-3 phase-1`. If MICA renames the tab (e.g. for a
   new phase or term), update the `SHEET_NAME` line in
   `.github/workflows/update-calendar.yml` to match exactly.

4. **Enable GitHub Pages:**
   - Repo → Settings → Pages
   - Source: "Deploy from a branch"
   - Branch: `main`, folder: `/docs`
   - Save. GitHub will give you a URL like:
     `https://<your-username>.github.io/<repo-name>/schedule.ics`

5. **Run the workflow once manually** to generate the first `schedule.ics`:
   - Repo → Actions tab → "Update MICA schedule calendar" → Run workflow
   - Check the Actions log — if `SHAREPOINT_URL` or `SHEET_NAME` is wrong,
     the error will tell you exactly what's missing (e.g. lists the actual
     sheet names found in the workbook).

6. **Subscribe on iPhone:**
   - Take the GitHub Pages URL from step 4, replace `https://` with `webcal://`
   - Settings → Calendar → Accounts → Add Subscribed Calendar → paste it in
   - iOS will poll this URL periodically (typically every few hours — this
     is an Apple-side setting you don't control precisely, but it's far
     better than manual redownloading)

## If the download URL doesn't work (403 error in the Actions log)

Some SharePoint "anyone with the link" shares don't respond to the
`&download=1` trick directly. If that happens:
1. Open the file in your browser (incognito, to confirm it's truly public)
2. Click the "..." menu on the file → Download, and watch the Network tab
   in browser dev tools for the actual file URL it hits
3. Use that exact URL as your `SHAREPOINT_URL` secret instead

## Notes on data quality

- Session start/end times are hardcoded (`FIXED_SESSION_TIMES` in
  `build_ics.py`) because the sheet doesn't mark AM/PM and is ambiguous to
  auto-parse around midday. If MICA changes the time slots, update that list.
- Multi-classroom days (two rows per date, second row's date cell blank) are
  handled — the continuation row inherits the date above it.
- Course titles in the calendar are copied verbatim from the cell, including
  faculty initials and session numbers, so events stay identifiable even
  when a session is rescheduled.
