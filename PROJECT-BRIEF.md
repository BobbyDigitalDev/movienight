# Movie Night Scheduler — Project Brief

**Project:** Movie Club availability picker
**URL:** `bobbydigitaldev.com/movienight`
**Hosting:** GitHub Pages (static, single `index.html`)
**Date:** July 16, 2026

---

## 1. Problem

Six busy friends want to find evenings when the most people are free to catch a movie in the theater. Group texts are chaos. We need a shared link where each person taps the nights they're available and everyone can see the overlap.

## 2. Core Decisions (already made)

| Decision | Choice | Rationale |
|---|---|---|
| Hosting | GitHub Pages, single HTML file | Free, already own the domain, no build step |
| Shared data | JSONBin.io (one bin) | Simplest possible backend. Free tier: 100 bins, 10MB, 1 req/sec, more than enough for 6 people |
| Identity | Free-text name entry | Group can grow without code changes |
| Name memory | `localStorage` | Persists across visits (better than a session cookie, which dies when the browser closes) |
| Calendar window | Rolling: current month + next month | Zero maintenance; past days grayed out |
| Extras (v1) | Best-night highlight + movie pick field | Adds the "so when are we going?" answer |

## 3. User Flows

### First visit
1. Welcome screen: app title, one-line pitch ("Tap the nights you're free"), text field for name, big "Let's go" button.
2. Name saved to `localStorage`. Returning visitors skip straight to the calendar (small "not Bobby? switch" link to change name).

### Picking availability
1. Two month grids stacked vertically (mobile) or side by side (desktop).
2. Tap a future date to toggle it: your selected days fill with your color.
3. Saves automatically (debounced ~1.5s after last tap to respect the 1 req/sec limit). Subtle "Saved ✓" indicator.

### Seeing the group
1. Each day cell shows a count badge (e.g. "4") when others have picked it.
2. Tap-and-hold or tap an already-glowing day to open a bottom sheet: "Fri Aug 7 — Bobby, Mike, Dre, Ray are free."
3. **Best Night banner** at the top: "🏆 Best night: Fri Aug 7 — 5 of 6 free." Ties show multiple dates.
4. Legend strip of member chips (name + color), tap a chip to highlight just that person's days.

### Movie pick
- A card above the calendars: "Now Showing: ___" with an edit (pencil) icon. Anyone can set the movie title for now; shows who set it and when.

## 4. Data Model (single JSONBin bin)

```json
{
  "movie": { "title": "Dune Part Three", "setBy": "Bobby", "updatedAt": "2026-07-16T14:00:00Z" },
  "availability": {
    "Bobby": ["2026-07-18", "2026-07-24", "2026-08-07"],
    "Mike":  ["2026-08-07", "2026-08-14"]
  },
  "updatedAt": "2026-07-16T14:00:00Z"
}
```

- Dates stored as `YYYY-MM-DD` strings.
- Old dates pruned automatically on each save (keeps the bin tiny forever).
- **Write safety:** before every save, re-fetch the bin, merge in only *your* name's array, then PUT. Two people saving simultaneously can't wipe each other out.

## 5. JSONBin Setup (one-time, ~5 min)

1. Create free account at jsonbin.io.
2. Create one bin with the empty data model above.
3. Generate an **Access Key** scoped to read/update that bin only (do NOT embed the master key). The access key ships in the page source; worst case a stranger scribbles on our calendar, and we can rotate the key.
4. Drop bin ID + access key into the config block at the top of `index.html`.

## 6. Design Direction

- **Mobile-first.** Everyone will open this from a group text on their phone.
- **Vibrant:** dark background with a saturated gradient accent (think cinema-neon: magenta/violet/electric blue), glowing selected days, popcorn/ticket iconography, one display font for headers.
- Each member auto-assigned a color from a fixed 8-color palette (hash of name), used in chips, day fills, and the bottom sheet.
- Big tap targets (min 44px cells), no hover-dependent interactions.
- Subtle micro-animations: day cells pop on tap, Best Night banner shimmer.

## 7. Technical Notes

- One file: `index.html` (inline CSS + JS, no frameworks, no build).
- Repo options: `bobbydigitaldev.com` repo with a `/movienight/` folder, or a `movienight` repo with Pages enabled (gives `bobbydigitaldev.com/movienight/` automatically if the domain is set at the user/org level).
- Poll for updates every ~30s while the page is open (cheap on the free tier), plus refresh on tab focus.
- Handle offline/failed saves gracefully: keep local state, retry, show a "reconnecting" note.

## 8. Out of Scope (v1)

- Accounts, passwords, real auth
- Push/text notifications
- Showtime lookup or theater APIs
- Host-only lock-in of a date (revisit for v2)
- Editing other people's availability

## 9. Milestones

1. **M1:** Static UI complete with fake data (welcome, calendars, bottom sheet, best night).
2. **M2:** JSONBin wired up (load, merge-save, poll, prune).
3. **M3:** Polish pass (animations, empty states, error states) + test on 2 phones.
4. **M4:** Deploy to GitHub Pages, send the link to the group.

## 10. Resolved Questions

- **Time granularity:** v1 stays at whole-evening scope. No early/late show splits.
- **Host mode:** confirmed for v2. Bobby (host) locks the official night; anyone returning to the URL sees a confirmation banner with the locked date and movie. Data model already accommodates this (add a `lockedDate` field alongside `movie`).
