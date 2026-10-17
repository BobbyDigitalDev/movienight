# AGENTS.md: start here

Guide for any AI assistant (or person) working on this project, whether you are the original maintainer's assistant or you cloned the repo to run it for your own group. Read this file first, then follow the reading order below.

## What this is

Movie Night is a single-page scheduling app for a small group of friends. The group can run several movie outings at once; each outing is its own board on the page with its own movie details and rolling four-month calendar. Each person enters a name once, taps the evenings they are free for an outing, and the app highlights the night the most people can make. A host locks in each outing's night; everyone sees the date, movie, theater, and showtime and can add it to their calendar. The morning after, that outing moves to a history log.

- **Stack:** one `index.html` (inline CSS + vanilla JS, no framework, no build step). Shared data lives in one JSONBin.io bin.
- **Hosting:** GitHub Pages, deploy from branch `main` / root. See `README.md` for setup.

## First: is this a fresh clone?

The `CONFIG` block at the top of the `<script>` in `index.html` ships with the **original author's** `BIN_ID`, `ACCESS_KEY`, and `HOST_NAME`. If you or your user cloned this repo for a different group:

1. Stop before running anything that saves. With the original values, every Save writes to someone else's group calendar.
2. Follow `README.md` "Setup" to create your own bin and access key.
3. Replace `BIN_ID`, `ACCESS_KEY`, and `HOST_NAME` with your own.

## Reading order

1. `AGENTS.md` (this file): rules and workflow
2. `STATUS.md`, **if present**: the maintainer's private status, deployment details, and change log. It is gitignored, so a fresh clone won't have it. If you are maintaining your own copy, create one (template below).
3. `ARCHITECTURE.md`: how `index.html` is organized, data model, sync and reset logic, gotchas
4. `DECISIONS.md`: why things are the way they are
5. `TESTING.md`: manual test checklist (there are no automated tests)
6. `README.md`: user-facing setup and feature guide

`PROJECT-BRIEF.md`, if present, is the original author's local planning doc. It is outdated; `DECISIONS.md` lists where the code differs.

## Hard rules

1. **One file.** All app code stays in `index.html`. No frameworks, bundlers, npm packages, or extra JS/CSS files unless the owner explicitly approves.
2. **Mobile first.** People open this from a group text on a phone. Tap targets at least 44px, nothing that depends on hover.
3. **Never embed the JSONBin master key.** The page ships an *Access Key* scoped to Bins Read + Update only. It is visible in public source by design (see `DECISIONS.md`). Do not add other secrets to the repo.
4. **Every write is read-merge-write.** Any code that changes the bin must `apiGet()` fresh, apply only its own change, then `apiPut()`. Never PUT the in-memory `data` object directly. This is what stops two people from wiping each other out.
5. **Respect the free tier: 1 request/second.** Keep the `CONFIG.PUT_DELAY` (1100 ms) gap between the GET and PUT of a write. Do not shorten `POLL_MS` (45 s) without reason.
6. **The live bin is real group data.** Do not write test data to it, reset it, or change its shape without the owner's approval. For destructive or schema changes, show what will change first, make one test change, verify it, and keep a before/after copy of the bin JSON so it can be restored. See `TESTING.md` for using a separate test bin.
7. **Schema changes must be backward compatible.** Existing bins may be in the older single-outing shape or missing newer fields. All reads go through `normalize()`; extend it rather than reading raw fields, and update the data model in `README.md` and `ARCHITECTURE.md`.
8. **Escape user text.** Names, titles, theater, showtime, and poster URLs are free text from anyone. Use `esc()` for anything put in `innerHTML`, or use `textContent`.
9. **Keep committed docs generic.** The repo is public and meant to be cloned. Anything specific to one deployment (people, URLs, plans, status) goes in the gitignored `STATUS.md`, not in `README.md` or these docs.
10. **Pushing to `main` is deploying.** GitHub Pages publishes `main` within a minute or two. Only push when the owner asks.
11. **Verify before reporting.** Do not say something is deployed, saved, or working without checking (re-read the file, load the live page, etc.), and say what you checked.

## Workflow

```bash
# local preview (fetch can misbehave from file://)
python3 -m http.server 8000
# open http://localhost:8000
```

A local preview still talks to whatever bin `CONFIG.BIN_ID` points at. Read-only browsing is fine; Save, lock, add or cancel an outing, and remove member all change real data. Use a test bin for anything that writes (see `TESTING.md`).

1. Read `STATUS.md` (if present) and confirm with the owner what this session is for.
2. Make the change in `index.html`, following the patterns in `ARCHITECTURE.md`.
3. Walk the relevant parts of `TESTING.md` on a phone-sized viewport.
4. Commit with a descriptive message (history uses a one-line summary plus a short "why" paragraph for non-trivial changes).
5. Push only when the owner asks, then load the live URL and confirm the change is there.

## End of session (required)

Before finishing any session that changed something:

- Add a dated entry to the top of the log in `STATUS.md`: what changed, why, commit hash, deployed or not, anything half-done.
- Update its "Current state" and "Backlog" sections if they changed.
- If you made a design decision, add it to `DECISIONS.md`.
- If code structure or the data model changed, update `ARCHITECTURE.md` and the data model in `README.md`.
- If a user-visible feature changed, update the feature list in `README.md`.

## STATUS.md template (for your own copy)

```markdown
# Status

Last updated: YYYY-MM-DD

## This deployment
- Owner / host name:
- Live URL:
- Repo:
- Bin: (where the ID lives; never write the master key here)

## Current state

## Log
### YYYY-MM-DD: short title
- what changed, why, commit, deployed?

## Known issues

## Backlog / ideas

## Open questions
```
