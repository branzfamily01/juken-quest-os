# Juken Quest OS

A single-file, offline, gamified study app for Japanese middle-school
entrance exam prep (中学受験). Drop in a study-material JSON pack, and it
turns that material into a quest-based learning game — no login, no build
step, no server.

## What it is

- **One HTML file** (`index.html`) — the whole app. Works fully offline
  once opened; all data lives in the browser's local storage.
- **Study packs are external JSON**, pasted into the app's data screen —
  the app ships with no bundled sample content by design (see
  `docs/AI_HANDOFF.md` for why).
- Built for iterative development with AI assistants (Claude, ChatGPT,
  etc.), so the codebase is deliberately documented for easy handoff
  between AI sessions.

## Features (v2)

- 6 question formats: multiple choice (shuffled), true/false, select-all,
  ordering, flashcards, and short text input
- 10 selectable visual themes, each with its own mascot, color palette,
  and gacha collection
- Parent-set difficulty/format ranges, with kid-facing daily quest
  selection within those ranges
- Spaced-repetition review queue (Leitner-style), "mistake monster"
  tracking per weak topic, XP/leveling, badges, gacha rewards
- Text-to-speech "listening mode" (per-question read-aloud + auto-advancing
  flashcard playback) with adjustable speed
- Local backup/restore (file download + JSON export/import) with a
  3-day backup reminder
- A `config` pack that switches the whole app between three editions
  (`play` / `custom` / `platform`) for potential distribution — see docs

## Repo structure

```
juken-quest-os/
├── index.html            the app itself — must stay named index.html
│                          and stay at the repo root for GitHub Pages
└── docs/
    ├── SPEC.md            full spec (source of truth)
    ├── AI_HANDOFF.md       invariants, code map, how to modify
    ├── ROADMAP.md          staged plan for future work
    └── PROMPTS_FOR_NEXT_AI.md   copy-paste prompts for future AI sessions
```

File names are ASCII on purpose (Japanese file *names* can get mangled by
some tools when zipping/unzipping on Windows — the *contents* are UTF-8
Japanese throughout and display correctly).

## Getting started (just running it locally)

1. Open `index.html` in a browser (double-click works; no server needed).
2. Create a profile through the setup wizard.
3. Go to the data screen and paste in a study-material JSON pack
   (`juken-pack@1` schema — see `docs/SPEC.md` for the format, and
   `docs/PROMPTS_FOR_NEXT_AI.md` for a ready-made prompt that generates
   one from any source material).
4. Back up regularly via the data screen (file download recommended).

## Publishing to GitHub Pages (optional)

Because the app is a single `index.html` at the repo root, GitHub Pages
can serve it with no build step:

1. Push this whole folder's contents to a GitHub repository (see below).
2. On GitHub: **Settings → Pages → Source → Deploy from a branch →
   branch: `main`, folder: `/ (root)`** → Save.
3. After a minute or two, the app is live at
   `https://<your-username>.github.io/<repo-name>/`.

If you'd rather it live under a sub-path (e.g. to match a pattern like
`branzfamily01.github.io/juken-quest-os/`), just make sure the repo name
matches the path you want — no code changes needed.

## Uploading this to GitHub

You need to upload **everything inside this zip** (i.e. `index.html`,
the `docs/` folder, and `.gitignore`), keeping the folder structure —
not just the zip file itself. Two ways to do it:

**Via the GitHub website (no command line):**
1. Create a new repository on GitHub (don't add a README there — you
   already have one).
2. On the new repo's page, click **"uploading an existing file"**.
3. Unzip this archive on your computer first, then drag the *contents*
   of the unzipped folder (not the zip itself, and not the outer folder —
   drag `index.html`, `docs/`, `.gitignore`, `README.md` individually or
   all selected together) into the upload area.
4. Commit.

**Via the command line, from the unzipped folder:**
```
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main
```

## Working with AI on this codebase

This app is meant to be handed off between AI sessions safely. Before
asking any AI to modify it, have it read `docs/AI_HANDOFF.md` first — it
lists the invariants that must never be broken (storage key name, schema
names, single-file structure, etc.) and a map of where each feature lives
in the code. `docs/PROMPTS_FOR_NEXT_AI.md` has ready-to-paste prompts for
common requests (add a theme, add a question format, generate a study
pack, fix a bug, and so on).

## Status / license

Personal / family project, with a possible future paid distribution
(see `docs/ROADMAP.md`, stage 6). **All rights reserved** — no open-source
license is granted at this time. Keep this repository private unless and
until that changes.
