# TPACK Activity Blueprint

A web app for planning one classroom activity with the TPACK framework (Technology, Pedagogy, and Content Knowledge).

Each activity has two steps:

1. **Card sort.** Teachers type their own cards (the pieces of the activity), tag each one T, P, and/or CK, and watch the cards move into the matching part of a TPACK diagram. **Remove technology** lists the technology cards and asks whether each is altered or not possible without the technology. **Use cards in the blueprint** sends every tagged card into the matching section of the blueprint.
2. **Blueprint.** A planning form organized by TPACK, with a diagram that fills in as the teacher writes.

Teachers sign in with their district Google account, fill out the blueprint, and save it. They can come back later to edit a saved activity or start a new one. Admins can open every saved activity and see who wrote it. Any activity can be turned into a Google Doc.

## How it works

The app is a **Google Apps Script web app**. This repository holds the source code. You push it to Apps Script with [`clasp`](https://github.com/google/clasp) and deploy it from there.

| Piece | Where it lives |
|---|---|
| Source code | This GitHub repository (`src/`) |
| Running app | Apps Script web app, restricted to your Google Workspace domain |
| Saved activities | A Google Sheet in the deployer’s Drive, one row per activity |
| Google Docs | A Drive folder in the deployer’s Drive, each Doc shared with the person who made it |

**Why Apps Script and not GitHub Pages:** GitHub Pages only serves static files, so it can’t sign people in or save their work. Apps Script signs people in with their district Google account automatically, keeps the data in district Drive, and costs nothing.

### Access rules

These are enforced on the server in `Code.gs`, not just hidden in the page.

- Every user sees, edits, and deletes **only their own** activities.
- Admins see **all** activities, with the author’s email. Other people’s activities open read-only.
- Only the deployer can open the data sheet directly. The app runs as the deployer, so teachers never need access to the sheet.

## Files

```
src/
  appsscript.json   Manifest: web app settings and permissions
  Code.gs           Server: storage, access checks, admin list, Google Doc export, prompt text
  Index.html        Page markup
  Styles.html       Styles (light and dark mode)
  CardSort.html     Card sort: tagging, the moving diagram, and the without-technology reflection
  Script.html       Page logic: views, tabs, form, diagram, saving
.clasp.json.example Template for linking this repo to an Apps Script project
```

All prompt wording lives in the `INFO` and `PARTS` lists at the top of `Code.gs`. The form and the Google Doc are both built from those lists, so you only change wording in one place. If you add or rename a field, the data sheet’s columns change too. Run `setup()` again to update the header row. Existing rows won’t move to match, so plan field changes before teachers start saving.

## Setup

You need Node.js and a Google account in your district domain.

### 1. Install clasp and sign in

```bash
npm install -g @google/clasp
clasp login
```

Turn on the Apps Script API for your account at <https://script.google.com/home/usersettings>.

### 2. Create the Apps Script project from this repo

```bash
git clone <your-repo-url>
cd tpack-activity-blueprint
clasp create --type standalone --title "TPACK activity blueprint" --rootDir src
```

This writes a `.clasp.json` file with your script ID. It’s in `.gitignore`. Each person who deploys keeps their own copy. If `clasp create` leaves an `appsscript.json` in the project root, delete it so the one in `src/` is used.

### 3. Push the code

```bash
clasp push
```

### 4. Run setup once

```bash
clasp open
```

In the Apps Script editor, open `Code.gs`, choose `setup` from the function menu, and select **Run**. Approve the permissions when asked.

`setup()` does three things:

- creates the data spreadsheet, **TPACK activity blueprints (data)**
- creates the Drive folder, **TPACK activity blueprints (Google Docs)**
- makes you the first admin

The execution log shows the links to both.

### 5. Add admins

In the editor, go to **Project Settings → Script properties** and edit `ADMIN_EMAILS`. Use a comma-separated list:

```
you@menifeeusd.org, coordinator@menifeeusd.org
```

### 6. Deploy

In the editor, select **Deploy → New deployment → Web app**, then set:

- **Execute as:** Me
- **Who has access:** Anyone within your domain

Copy the web app URL and share it with teachers.

## Updating the app

```bash
git pull           # or make your changes
clasp push
```

Then, in the editor, go to **Deploy → Manage deployments**, edit the existing deployment, and choose **New version**. The URL stays the same.

If you create a *new* deployment instead, you’ll get a new URL.

## Notes

- **Where cards land in the blueprint.** Cards tagged T go to the tool prompt, P to “What will students do?”, CK to the learning intention, and each overlap (TPK, TCK, PCK, TPACK) to its own prompt. Cards are added as bullet lines and never overwrite what’s already written. Teachers then move each line to the prompt where it fits. Change the mapping in `CARD_TARGETS` in `Script.html`.
- **The card sort is saved with the activity** in the `cards` column of the data sheet, including the altered and not possible decisions. It also appears in the Google Doc. Each activity holds up to 40 cards of up to 200 characters.

- **Saving is manual.** Teachers select **Save** or press Ctrl+S / ⌘S. The page warns them before they leave with unsaved changes.
- **Each field holds up to 5,000 characters.**
- **Google Docs are owned by the deployer** and shared with the person who made them. They show up under **Shared with me**. Each select of **Make a Google Doc** creates a fresh copy reflecting the current saved version.
- **Entries are stored as plain text,** so anything that starts with `=` or `+` is never treated as a spreadsheet formula.
- **The app runs as the deployer.** If that person leaves the district, transfer the script, the sheet, and the folder to another account, then redeploy.
