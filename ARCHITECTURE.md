# Architecture

Everything is in `index.html` (about 1,140 lines as of Oct 17, 2026). Line numbers drift, so this map uses the section banner comments in the script (`/* ================= NAME ================= */`). Search for the banner to jump to a section.

## File layout

| Part | Where | What |
|---|---|---|
| `<head>` | top | Meta, Google Fonts (Unbounded for display, Outfit for body), `<title>Movie Night 🍿</title>` |
| `<style>` | `<head>` | All CSS. Theme tokens on `:root` (`--bg`, `--panel`, `--magenta`, `--violet`, `--cyan`, `--grad`, ...). Dark only; no light theme. |
| Markup | `<body>` | `#welcome` (name entry), `#app` (main UI), then overlays: `#saveBar`, `#status`, `#scrim`, `#sheet` (who's free), `#lockSheet` (host lock confirm) |
| `<script>` | end of body | All JS, sections below |

### Main UI, top to bottom (`#app`)

1. `header.app`: logo + `#userChip` (your name and color; tap to switch name)
2. `#movieCard`: Now Showing. Poster thumb, title, byline (picked by, theater, showtime). Pencil opens `#movieEdit` with title, poster URL, theater, showtime.
3. `#lockedCard`: "It's Official" banner, only when `lockedDate` is set. Who's going, who can't, pencil to edit theater/showtime, `#calRow` with .ics download and Google Calendar link.
4. `#bestCard`: Best Night, hidden once a night is locked. Host sees lock buttons ("pills") per tied date; others tap to see who's in.
5. Instructions line, `#legend` (member chips, tap to filter; host sees ✕ to remove), `#clearBtn` (clear my nights)
6. `#months`: two month grids (current + next)
7. `#historyCard`: Movie Night History
8. `#newRoundBtn`: New Movie Night (wipes the board, two confirms)

## Script sections

| Banner | Contents |
|---|---|
| `CONFIG` | `BIN_ID`, `ACCESS_KEY`, `HOST_NAME` (the host's name, case-insensitive), `API`, `POLL_MS` (45000), `PUT_DELAY` (1100). Also `PALETTE` (8 member colors). |
| `STATE` | Globals: `myName` (from `localStorage['mn_name']`), `data` (last known bin record), `myDates` (Set of your picks, local), `dirty` (unsaved picks), `saving`, `filterName` (legend filter), `longPressed` |
| `HELPERS` | `$`, `pad`, `isoOf`, `todayIso`, `esc` (HTML escape), `hashIdx`, `assignColors` / `colorOf`, `prettyDate`, `members`, `isHost`, `freeOn`, `attendeesForRecord` |
| `API` | `apiGet()` (GET `/latest`, returns `.record`), `apiPut(record)`, `describeError`, `pruneOld(record)` |
| `SYNC` | `maybeReset`, `load`, `markDirty`, `save(extra)`, `startNewRound` |
| `RENDER` | `render()` calls every `render*` function. Plus calendar export (`eventTimes`, `gcalLink`, `downloadICS`), `bestNights`, `openLockSheet` |
| `DAY INTERACTION` | `attachDayEvents`: tap toggles a pick, hold 450 ms opens the who's-free sheet |
| `SHEET` | `openSheet(iso)` (who's free; host gets Lock / Unlock button), `closeSheet()` |
| `STATUS` | `showStatus`, `flashStatus` (toast at bottom) |
| `BOOT` | `enterApp`, all event listeners, `removeMember`, polling interval, `visibilitychange` refresh, `init()` |

## Data model (one JSONBin bin)

```json
{
  "movie":        { "title": "", "poster": "", "setBy": "", "updatedAt": "" },
  "availability": { "Alex": ["2026-10-23", "2026-10-30"], "Sam": ["2026-10-30"] },
  "lockedDate":   "2026-10-30",
  "theater":      "AMC Empire 25",
  "showtime":     "7:45 PM",
  "history": [
    { "date": "2026-08-07", "movie": "", "poster": "", "theater": "", "showtime": "", "attendees": ["Alex", "Sam"] }
  ],
  "updatedAt":    "2026-10-17T21:30:00.000Z"
}
```

- Dates are local-time `YYYY-MM-DD` strings. String comparison is used for ordering (`d >= todayIso()`).
- `availability` keys are display names exactly as first entered. Matching on entry is case-insensitive, so "mike" joins an existing "Mike".
- `history` is newest first (`unshift`). It grows forever; nothing prunes it.
- `lockedDate`, `theater`, `showtime`, `history`, and `movie.poster` were added after v1. Older bins may lack them; code treats missing as empty.

## Flows

### Boot
`init()` does one `apiGet()` (so the welcome form can match an existing name), then `enterApp()` if a name is in `localStorage`. `enterApp()` renders and calls `load()`.

### Load / poll
`load()` = `apiGet()` → `maybeReset()` → `pruneOld()` → set `data` → render. If you have unsaved picks (`dirty`), your local `myDates` win over the server copy of your row. Runs every 45 s while the tab is visible and nothing is unsaved or saving, and when the tab becomes visible. On failure it shows a toast and retries every 8 s.

### Picking nights
Tapping a day only changes local `myDates` and shows the `#saveBar`. Nothing is sent until **Save my nights** is tapped. (The original plan was auto-save; the code uses an explicit button. See `DECISIONS.md`.)

### Save (the only general write path)
`save(extra)`:
1. `apiGet()` fresh and `pruneOld()` it
2. Optionally delete a member (`extra.remove`)
3. Replace **only your own** `availability[myName]` with `myDates` (future dates only)
4. Apply optional `extra.movie`, `extra.lock`, `extra.theater`, `extra.showtime`
5. Wait `PUT_DELAY`, then `apiPut()`

Movie edits, lock/unlock, detail edits, and member removal all go through `save()` with an `extra`. Note that every one of these also writes your current picks, saved or not.

UI is updated optimistically before the write. If the write fails the toast says so, but the optimistic in-memory change is not rolled back until the next `load()`.

### Locking (host only)
`isHost()` compares `myName` to `CONFIG.HOST_NAME` case-insensitively. There is no auth; anyone who enters the `HOST_NAME` is the host. Two entry points:
- Best Night pill → `openLockSheet()` → `#lockConfirm` (can set movie, poster, theater, showtime at the same time)
- Hold any future day → `openSheet()` → Lock / Unlock button (sets the date only)

### Automatic reset (archive then wipe)
`maybeReset(record)` runs inside every `load()`. If `lockedDate` is set and earlier than today, it pushes a history entry (attendees = who had that date picked), clears availability, movie, lock, theater, showtime, and PUTs. It must run **before** `pruneOld()`, because pruning would delete the past locked date from everyone's picks and the attendee list would come out empty.

### New Movie Night (manual reset)
`startNewRound()`: any user, after two `confirm()` dialogs. Reads the raw record, archives the locked night to history **only if one is locked**, then wipes the same fields as `maybeReset()`. If nothing was locked, the round is wiped with no history entry.

### Colors
`assignColors()` sorts every member name, gives each its hash slot (`hashIdx`), and probes forward to the next free slot on collision. Every device computes the same map. Distinct for up to 8 people; past 8, colors repeat.

### Calendar export
`eventTimes()` parses `showtime` loosely (`7:45 PM`, `7pm`, `19:30`). With no am/pm, hours 1 to 8 are treated as PM. No showtime defaults to 7:00 PM. Events are 3 hours long and written in floating local time (no timezone).

## Gotchas

- **Local preview writes to the live bin.** See `TESTING.md`.
- **Removed members come back** if they still have the page open or their name in `localStorage` and they save again. This is intended ("they can rejoin anytime").
- **`confirm()` dialogs** are used for switch name, clear my nights, remove member, and New Movie Night. Browser automation tools can hang on these.
- **Any write also saves your picks.** Editing the movie while you have unsaved taps will save those taps too.
- **JSONBin keeps no history for us.** A bad write is only recoverable if someone kept a copy of the bin JSON (or from JSONBin's own version history, if the plan provides it; not verified).
