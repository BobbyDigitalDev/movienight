# Decisions

Why things are the way they are. Ask the project owner before reversing any of these. If you cloned this for your own group, you're the owner: change what you like, and record it here. Add new decisions at the bottom of the relevant section with a date.

## Platform

| Decision | Choice | Why | When |
|---|---|---|---|
| Hosting | GitHub Pages, single `index.html` | Free, works with a custom domain already set on the GitHub user site, no build step | 2026-07-16 |
| Backend | One JSONBin.io bin | Simplest shared store. Free tier (1 req/sec, 10 MB) is plenty for a friend group | 2026-07-16 |
| Key in page source | JSONBin **Access Key** (Bins Read + Update only), never the master key | It has to ship to the browser. Worst case a stranger edits the calendar, and the key can be rotated in the JSONBin dashboard | 2026-07-16 |
| Identity | Free-text name, remembered in `localStorage` (`mn_name`) | No accounts. The group can grow without code changes. `localStorage` survives browser restarts, unlike a session cookie | 2026-07-16 |
| Name matching | Case-insensitive against existing names | "mike" and "Mike" are the same person, no duplicates | 2026-07-16 |
| Calendar window | Rolling: current month + next month | Zero maintenance. Past days are grayed out and pruned from the bin on every save | 2026-07-16 |
| Granularity | Whole evenings only, no early/late show split | Keep v1 simple | 2026-07-16 |
| Repo is public and generic | README written for strangers, MIT license, the original author's personal notes gitignored | So others can clone it for their own group | 2026-07-16 (commit 97751a3) |

## Behavior

| Decision | Choice | Why | When |
|---|---|---|---|
| Concurrent writes | Re-fetch, change only your own row (plus the specific field being edited), then PUT | Two people saving at once can't wipe each other out | 2026-07-16 |
| Member colors | Group-wide assignment: sorted names, hash slot, probe to next free slot | Earlier pure-hash version gave two people the same color | 2026-07-16 (3afdc96) |
| Host | Single host by name (`CONFIG.HOST_NAME`), no auth | Originally planned for v2; built on day one. No auth is acceptable for a friend group | 2026-07-16 |
| Theater/showtime | Anyone can edit, not just the host | Whoever buys the tickets fills them in | 2026-07-16 (ae43867) |
| Calendar export | .ics download + Google Calendar link, generated in the browser | No server needed | 2026-07-16 (ae43867) |
| Auto-reset | Morning after the locked night, the first visitor wipes the board | Zero maintenance between rounds | 2026-07-16 |
| Clear my nights | One tap with a confirm | Sitting a round out shouldn't take many taps | 2026-07-16 (3c11696) |
| Member removal | Host only, via ✕ on the chip, with confirm. Removed people can rejoin by entering their name | Clean up stray testers or people who left | 2026-07-16 (cbe2632) |
| New Movie Night | **Anyone** can wipe the board on demand, behind **two** confirms | Whoever is organizing this time can start the next round early | 2026-08-10 (b5c8ab5) |
| History | Every locked night is archived (movie, poster, date, theater, showtime, attendees) before any wipe | Never lose the group's history | 2026-08-10 (3cf1b84) |
| Posters | Optional pasted image URL, no movie API | No API keys, no lookup service (showtime and theater APIs are out of scope) | 2026-08-10 (3cf1b84) |
| History dates | Show the year | History persists across years; other dates stay inside the two-month window and don't need it | 2026-08-10 (d6d3255) |
| History attendee order | Alphabetical at render time | Nobody reads as first or last, however the entry was created | 2026-08-10 (ac5092a) |

## Out of scope (from the original plan, still true unless the owner decides otherwise)

- Accounts, passwords, real auth
- Push or text notifications
- Showtime lookup or theater APIs
- Editing other people's availability

## Where the code differs from the original plan

The original plan (July 16, 2026) is not in the repo. Where it and the code disagree, the code is the truth. This table exists so nobody "fixes" the code back to the plan.

| Original plan | Code does |
|---|---|
| Picks save automatically, debounced ~1.5 s | Explicit **💾 Save my nights** button. Taps are local until saved. (Reason not recorded in git; likely to avoid accidental writes and to make "saved" obvious. Confirm with the owner before switching to auto-save.) |
| Poll every ~30 s | Polls every 45 s (`POLL_MS`), plus on tab focus |
| Host lock-in is v2 | Built in v1, along with theater, showtime, calendar export |
| Tap a glowing day to see who's free | Hold a day (450 ms) to see who's free; tap always toggles your pick |
| Data model: `movie`, `availability`, `updatedAt` | Adds `lockedDate`, `theater`, `showtime`, `history`, `movie.poster` |
