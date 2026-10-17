# Architecture

Everything is in `index.html`. Line numbers drift, so this map uses the section banner comments in the script (`/* ================= NAME ================= */`). Search for the banner to jump to a section.

## File layout

| Part | Where | What |
|---|---|---|
| `<head>` | top | Meta, Google Fonts (Unbounded for display, Outfit for body), `<title>Movie Night 🍿</title>` |
| `<style>` | `<head>` | All CSS. Theme tokens on `:root` (`--bg`, `--panel`, `--magenta`, `--violet`, `--cyan`, `--grad`, ...). Dark only. A global `[hidden] { display: none !important; }` rule makes the `hidden` attribute win over class display rules. |
| Markup | `<body>` | `#welcome` (name entry), `#app` (main UI), `<template id="outingTpl">` (one outing's markup), then overlays: `#saveBar`, `#status`, `#scrim`, `#sheet` (who's free), `#lockSheet` (host lock confirm) |
| `<script>` | end of body | All JS, sections below |

### Page, top to bottom (`#app`)

1. `header.app`: logo + `#userChip` (your name and color; tap to switch name)
2. Instructions line
3. `#legend`: shared member chips for all outings (tap to filter; host sees ✕ to remove someone from every outing)
4. `#outings`: one `<section class="outing">` per outing, stacked. Each section, cloned from `#outingTpl`:
   - `.outing-head`: "Outing 2 of 3 · Fri, Nov 20" (hidden when there is only one outing)
   - `.movie-card`: Now Showing. Pencil opens `.movie-edit` (title, poster URL, theater, showtime)
   - `.locked-card`: "It's Official" banner when the outing has a `lockedDate`; pencil edits theater/showtime; .ics and Google Calendar buttons
   - `.best-card`: Best Night for this outing (hidden once locked). Host gets lock pills
   - `.clear-btn`: clear my nights for this outing (only when you have picks in it)
   - `.months`: `CONFIG.MONTHS` month grids (default 4). One column on phones, two at 860px+
   - `.outing-cancel-btn`: shown to the host or the outing's creator (hidden for a lone blank outing)
5. `#addOutingBtn`: ➕ Add another movie outing (anyone)
6. `#historyCard`: Movie Night History

## Script sections

| Banner | Contents |
|---|---|
| `CONFIG` | `BIN_ID`, `ACCESS_KEY`, `HOST_NAME`, `API`, `POLL_MS` (45000), `PUT_DELAY` (1100), `MONTHS` (4). Also `PALETTE` (8 member colors). |
| `STATE` | `myName` (from `localStorage['mn_name']`), `data` (last known record), `loaded`, `myDates` (outingId → Set of your picks, local), `dirty` (Set of outingIds with unsaved picks), `saving`, `filterName`, `longPressed`, `sections` (outingId → section element) |
| `HELPERS` | `$`, `esc`, `isoOf`/`todayIso`, `blankOuting`, `newId`, colors (`assignColors`/`colorOf`), `prettyDate`, `outingById`, `members`, `isHost`, `canCancel`, `mySet`, `picksOf`, `freeOn`, `outingLabel` |
| `DATA SHAPE` | `normalize` (migration and defaults), `pruneOld`, `attendeesOf`, `archiveInto` |
| `API` | `apiGet()` (GET `/latest`, returns `normalize(record)`), `apiPut(record)`, `goneError` |
| `SYNC` | `syncMine`, `load`, `save`, `saveOuting`, `addOuting`, `cancelOuting`, `removeMember`, save bar helpers |
| `RENDER` | `render()` → `renderUser`, `renderLegend`, `renderOutings` (→ `renderOuting` → movie, locked, best, months), `renderHistory`; calendar export (`eventTimes`, `gcalLink`, `downloadICS`) |
| `OUTING SECTION` | `buildSection(id)`: clones the template and wires its buttons; `openMovieEdit` |
| `DAY INTERACTION` | `attachDayEvents(cell, outingId, iso)`: tap toggles a pick, hold 450 ms opens the who's-free sheet |
| `SHEETS` | `openSheet(outingId, iso)` (host gets Lock / Unlock), `openLockSheet(outingId, iso)`, `closeSheet` |
| `STATUS` | `showStatus`, `flashStatus` (toast) |
| `BOOT` | `enterApp`, global listeners, polling, `init()` |

## Data model (one JSONBin bin)

```json
{
  "outings": [
    {
      "id": "o1",
      "createdBy": "Alex",
      "createdAt": "2026-10-17T22:00:00.000Z",
      "movie": { "title": "", "poster": "", "setBy": "", "updatedAt": "" },
      "availability": { "Alex": ["2026-10-23", "2026-10-30"], "Sam": ["2026-10-30"] },
      "lockedDate": "2026-10-30",
      "theater": "AMC Empire 25",
      "showtime": "7:45 PM"
    }
  ],
  "history": [
    { "date": "2026-08-07", "movie": "", "poster": "", "theater": "", "showtime": "", "attendees": ["Alex", "Sam"] }
  ],
  "updatedAt": "2026-10-17T21:30:00.000Z"
}
```

- Outing `id`s are random (`newId()`), except the migrated or fallback first outing, which is always `"o1"` so every device converts old data identically.
- `createdBy` is the name of whoever added the outing; `""` for the migrated outing and auto-created blank ones (so only the host can cancel those).
- Dates are local-time `YYYY-MM-DD` strings compared as strings.
- `availability` keys are display names exactly as first entered; name entry matches case-insensitively against everyone in any outing.
- **Members are derived**: everyone with a row in any outing, plus you (`members()`). There is no separate member list.
- `history` is newest first and is never pruned.

### Migration (`normalize`)
Every read goes through `normalize()`. A record without an `outings` array (the pre-October-2026 shape: top-level `movie`, `availability`, `lockedDate`, `theater`, `showtime`) becomes `outings: [{ id: "o1", ...those fields }]`, and the old top-level keys are deleted. An empty `outings` array gets one blank `"o1"` outing. Missing fields are defaulted. The converted shape is written on the next save; a plain page load doesn't write.

**Old cached pages:** `apiPut()` merges `LEGACY_STUB` into every write: top-level `availability: {}`, empty `lockedDate`/`theater`/`showtime`, and a `movie` whose title says "Movie Night was updated, refresh this page". A tab still running the pre-outings page reads that cleanly, shows the refresh message as its movie title, and polls normally. Its own writes re-read the whole record, so `outings` and `history` survive them. Without the stub, the old page threw on every read and its retry loops multiplied into thousands of requests (found in review, 2026-10-17). The new page ignores the stub keys.

## Flows

### Boot
`init()` does one `apiGet()` (so the welcome form can match an existing name), then `enterApp()` if a name is stored. `enterApp()` renders and calls `load()`.

### Load / poll
`load()` = `apiGet()` → `archiveInto()` (PUT only if something was archived) → `pruneOld()` → `data` → `syncMine()` → `render()`. Polls every 45 s and on tab focus, only while nothing is unsaved (`dirty` empty) or saving. A load whose read started before a save finished is discarded (`writeSeq`), so an old read can't overwrite newer state. On failure, `scheduleRetry()` keeps a single pending retry with backoff (8 s doubling to 2 min). `syncMine()` refreshes your local picks for every outing you have no unsaved changes in, and forgets outings that no longer exist.

### Rendering and open forms
Sections are created once per outing id and then only updated, never rebuilt, so an open edit form keeps what you typed through re-renders and background polls. Handlers look the outing up by id at click time.

### Picking nights
Tapping a day changes `myDates[outingId]` and marks that outing dirty. One **Save my nights** button saves all dirty outings together.

### Save (the only write path)
`save(mutate, okMsg)`:
0. If another save is running, wait for it (saves are queued, never dropped)
1. `apiGet()` fresh, `archiveInto()`, `pruneOld()`
2. For each dirty outing still present, replace **only your own** `availability[myName]`. If a dirty outing is gone (someone cancelled it), those picks are dropped and the toast says so.
3. Run `mutate(fresh)` for any other edit
4. Wait `PUT_DELAY`, `apiPut()`

`saveOuting(id, edit)` applies an edit to one outing locally (instant screen update) and inside the save. If the outing was cancelled meanwhile it throws `goneError()`, and the page refreshes instead of writing. Movie edits, detail edits, lock, and unlock all use it. `addOuting`, `cancelOuting`, and `removeMember` call `save()` with their own `mutate`.

Edit forms (movie, details, lock sheet) snapshot their values when opened, and `applyChanges()` writes only the fields the person changed, so a form left open doesn't put back someone else's newer values. If an edit's save fails, the toast says "That change didn't save" and a reload puts the screen back.

### Locking (host only, per outing)
`isHost()` compares `myName` to `CONFIG.HOST_NAME` case-insensitively (no auth). Either Best Night pill → `openLockSheet()` → `#lockConfirm` (can set movie, poster, theater, showtime too), or hold any future day → `openSheet()` → Lock / Unlock.

### Wrapping a finished outing (`archiveInto`)
Runs inside every `load()` and `save()`. "Over" uses `cutoffIso()`, which is **yesterday** on this device, so a member in a later time zone (e.g. traveling in Japan) can't wrap tonight's outing early. The outing stays on the page through the day after its night. Every outing whose `lockedDate` is before the cutoff becomes a history entry (attendees = who had that date picked in that outing) and is removed. If none remain, a blank outing is added with a deterministic id (`'n' + removed id`) so two devices wrapping at once agree. It must run **before** `pruneOld()`. `pruneOld()` drops picks before the cutoff but always keeps each outing's own `lockedDate`, so the attendee list survives until archiving.

### Adding and cancelling outings
- **Add:** anyone. Pushes `blankOuting(myName)`, saves, scrolls to it, and opens its movie form.
- **Cancel:** `canCancel()` = host, or `createdBy` matches you. Two confirms. Removes the outing without archiving. Cancelling the last one leaves a fresh blank outing.

### Colors
`assignColors()` over `members()`: sorted names take their hash slot, probing to the next free one. Same map on every device; distinct for up to 8 people.

### Calendar export
Per outing. `eventTimes(o)` parses `showtime` loosely (`7:45 PM`, `7pm`, `19:30`); without am/pm, 1 to 8 means PM; default 7:00 PM. 3-hour events in floating local time.

## Gotchas

- **Local preview writes to the live bin.** See `TESTING.md`.
- **Removed members come back** if they still have the page open and save again. Intended ("they can rejoin anytime").
- **`confirm()` dialogs** are used for switch name, clear my nights, remove member, and cancel outing (twice). Browser automation can hang on these unless dialogs are auto-accepted.
- **Any save also saves your unsaved picks** in every outing you changed.
- **Two saves from different people within about 1.5 s:** the later one wins. That comes with a single JSONBin record and is rare for a small group.
- **Picks are per outing.** Someone free on the same night for two outings taps it in both.
- **JSONBin keeps no history for us.** Keep a copy of the bin JSON before risky changes.
