# Testing

There are no automated tests. Test by hand in a phone-sized viewport (about 390 x 844) and, before a release that changes interaction, on a real phone.

## Use a test bin for anything that writes

`CONFIG.BIN_ID` in `index.html` points at the **live** group bin, even when you run the page locally. In a fresh clone it points at the **original author's** bin, so swap in your own before testing anything (see README setup). To test saves, locks, removals, or resets without touching real data:

1. In the JSONBin dashboard, create a second bin with the starter JSON:
   ```json
   { "outings": [], "history": [], "updatedAt": "" }
   ```
2. Give the existing Access Key access to it (or make a test key with Bins Read + Update).
3. Temporarily change `CONFIG.BIN_ID` (and `ACCESS_KEY` if different) in your local copy.
4. **Change them back before committing.** Check with `git diff index.html` that only intended lines changed.

To test the automatic reset, put a `lockedDate` in the past in the test bin through the dashboard, then load the page.

Two browser profiles (or one normal + one private window) give you two users. Give one of them the `CONFIG.HOST_NAME` name to see host controls.

## Checklist

### Welcome and identity
- [ ] First visit shows the welcome screen; entering a name opens the app
- [ ] Entering an existing name in different case (e.g. all caps) joins that person rather than creating a duplicate
- [ ] Reload keeps you signed in; the name chip "switch" asks to confirm, then returns to welcome

### Picking and saving
- [ ] Tapping a future day toggles it in your color and shows the Save bar with a count
- [ ] The calendar shows `CONFIG.MONTHS` months starting with the current one (2x2 grid on desktop, stacked on phones)
- [ ] Past days are grayed and not tappable
- [ ] Save shows "Saving…" then "Saved ✓"; the other user sees the pick after reload or within 45 s
- [ ] Two users saving at nearly the same time both keep their picks
- [ ] Clear my nights empties your picks after confirm (still needs Save)

### Group view
- [ ] Count badges and colored dots appear on days others picked
- [ ] Holding a day opens the sheet with who's free and who isn't
- [ ] Best Night shows the top date(s) once 2+ people overlap; ties listed
- [ ] Tapping a legend chip dims days that person isn't free; tapping again clears
- [ ] No two members share a color (up to 8)

### Movie card
- [ ] Pencil edits title, poster URL, theater, showtime; byline shows "picked by"
- [ ] A broken poster URL doesn't break the layout

### Multiple outings
- [ ] An old single-outing bin shows as one outing with its movie, picks, and lock intact, and a plain page load doesn't write
- [ ] ➕ Add another movie outing creates a second section below with its movie form open; "Outing 1 of 2" headers appear
- [ ] Picks, movie details, Best Night, and lock are independent per outing; one Save button saves picks from several outings
- [ ] The legend is shared; filtering by a person dims days in every outing
- [ ] Creator sees Cancel on their own outing but not on others; host sees it on all; a lone blank outing shows no Cancel
- [ ] Cancelling asks twice, removes the outing, and does not add it to History; cancelling the last outing leaves a blank one
- [ ] If someone else cancels an outing while you have unsaved picks in it, saving doesn't bring it back and tells you
- [ ] An open movie edit form keeps what you typed through a background refresh

### Host (name = `CONFIG.HOST_NAME`)
- [ ] Best Night shows 🔓 pills; tapping one opens the lock sheet; Lock it in shows the official banner for everyone
- [ ] Holding a future day offers Lock; holding the locked day offers Unlock
- [ ] ✕ on another member's chip removes them from every outing after confirm
- [ ] Non-host users see none of these controls

### Official night
- [ ] Banner shows date, showtime, movie, theater, who's going, who can't
- [ ] Add to calendar downloads a working .ics (check time, title, location)
- [ ] Google Calendar link opens with the right date and time

### Resets and history
- [ ] With a past `lockedDate` on one outing, loading the page archives that outing to History (with attendees) and removes it; other outings are untouched
- [ ] If that was the only outing, a fresh blank outing appears
- [ ] History entries show year, poster or 🎬 placeholder, alphabetical attendees

### Resilience
- [ ] With network off, load shows "Can't reach the calendar — retrying…" and recovers when back online
- [ ] A failed save shows "Save failed — tap to try again" and the retry works

### After deploy
- [ ] Your live URL shows the change (hard refresh; Pages can take a minute or two)
