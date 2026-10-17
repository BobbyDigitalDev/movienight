# 🍿 Movie Night

A tiny, zero-backend scheduling app for movie groups. Friends open one shared link, tap the evenings they're free, and the app highlights the night the most people can make it to the theater.

The whole thing is one `index.html` — no framework, no build step, no server. Host it free on GitHub Pages, store the group's picks in a free [JSONBin.io](https://jsonbin.io) bin.

## Features

- **Several movie outings at once** — tap **➕ Add another movie outing** and a second board appears below the first, with its own Now Showing details, calendar, Best Night, and lock-in. Each outing is self-contained, so nobody has to untangle two movies on one calendar
- **Four-month rolling calendar** — always shows this month and the next three (change `MONTHS` in `CONFIG`); past days prune themselves automatically, so it rolls forward after every movie night with zero maintenance
- **No accounts** — everyone just types a name once; it's remembered on their device ("mike" and "Mike" are matched, so no duplicate people)
- **Per-person colors** — your picks glow in your color; everyone else shows up as count badges and colored dots
- **🏆 Best Night card** — automatically surfaces the date(s) with the most overlap
- **Who's in?** — hold any day to see the names; tap a name chip to spotlight one person's availability
- **Now Showing card** — anyone can set the movie for each outing
- **Host lock-in** — the host taps an outing's best night and locks it; everyone sees an "🎬 It's Official" banner with the date, movie, theater, showtime, and who's going
- **Theater & showtime** — anyone can set or update them (whoever buys the tickets fills them in)
- **Add to calendar** — once locked, everyone gets an .ics download and a Google Calendar link generated right in the browser
- **Auto-wrap** — the morning after an outing's locked night, that outing moves to History and disappears from the page; the others stay put. If it was the only one, a fresh blank outing takes its place
- **Cancel an outing** — the host, or whoever added the outing, can delete it (two confirmations). Cancelled outings never happened, so they don't go into History
- **🎞️ Movie Night History** — every locked night gets automatically archived (movie, poster, date, theater, showtime, who was free that night) when it wraps, so you never lose the group's history; optionally add a poster image URL when you set the movie
- **Sit one out** — a "clear my nights" link empties all your picks for one outing in one tap (with confirmation) if you can't make it
- **Host member removal** — the host sees a ✕ on each name chip to remove someone from every outing (stray testers, departed members); anyone removed can rejoin just by entering their name again
- **Safe concurrent saves** — saving re-fetches and merges, so two people saving at once can't wipe each other out

## Setup (~15 minutes)

### 1. Create the JSON store

1. Make a free account at [jsonbin.io](https://jsonbin.io)
2. Create a bin containing exactly:
   ```json
   { "outings": [], "history": [], "updatedAt": "" }
   ```
   (The app fills in the first outing itself. A bin made with the older starter JSON, with top-level `movie` and `availability`, also works.)
3. Note the **Bin ID** (shown in the bin's URL/header)
4. Under API Keys, create an **Access Key** with only **Bins: Read + Update** permissions. Use an access key, *not* your master key — this key ships in your page source, so it should only be able to read/update this one thing. You can rotate it from the dashboard anytime.

### 2. Get the code

Fork or clone this repo, then open `index.html` and find the `CONFIG` block at the top of the `<script>`:

```js
const CONFIG = {
  BIN_ID: '…',      // ← replace with your Bin ID
  ACCESS_KEY: '…',  // ← replace with your Access Key
  HOST_NAME: '…',   // ← the name (as entered in the app) of your group's host
  ...
};
```

Replace the values with your own. (The bin values you'll find there belong to the original author's movie club — the app won't work for your group until you swap them.) `MONTHS` (default 4) sets how many months the calendar shows. `HOST_NAME` decides who sees the lock-in button; it matches case-insensitively against the name that person enters on the welcome screen.

### 3. Deploy

1. Push to a GitHub repo (name it `movienight` and the URL path is `/movienight/`)
2. Repo **Settings → Pages** → Source: **Deploy from a branch** → `main` / root → Save
3. Your app is live at `https://<username>.github.io/movienight/` — or under your custom domain if your GitHub Pages user site has one

Text the link to your group. Done.

## How your group uses it

1. Open the link, enter your name once
2. Tap the evenings you're free, then hit **💾 Save my nights**
3. Hold a day to see who's in; watch the **Best Night** card find your date
4. Set the movie, poster URL, theater, and showtime with the ✏️ on the Now Showing card (anyone can, anytime)
5. The host taps a best night to lock it in — everyone gets the official banner and can add it to their calendar
6. Want to see a second movie too? Tap **➕ Add another movie outing** at the bottom, set its movie, and pick your nights on its own calendar
7. The morning after an outing's night, it moves to the **Movie Night History** log automatically

## Data model

```json
{
  "outings": [
    {
      "id":           "o1",
      "createdBy":    "alex",
      "createdAt":    "…",
      "movie":        { "title": "…", "poster": "…", "setBy": "…", "updatedAt": "…" },
      "availability": { "alex": ["2026-07-24", "2026-08-07"] },
      "lockedDate":   "2026-08-07",
      "theater":      "AMC Empire 25",
      "showtime":     "7:45 PM"
    }
  ],
  "history": [
    { "date": "2026-07-10", "movie": "…", "poster": "…", "theater": "…", "showtime": "…", "attendees": ["alex", "sam"] }
  ],
  "updatedAt":    "…"
}
```

Each outing is independent. When an outing's `lockedDate` is set, everyone sees the official-night banner on that outing. Once that date passes, the first person to open the page pushes that night (movie, poster, theater, showtime, and who was free that day) onto `history` and removes the outing; if no outings are left, a blank one is created. Bins in the older single-outing shape (top-level `movie`, `availability`, `lockedDate`, `theater`, `showtime`) are converted to `outings[0]` automatically the first time the new version saves. `poster` is just a URL you paste in when you set the movie; leave it blank and the history entry shows a plain placeholder instead.

## Resetting the board

Normally unnecessary — past dates prune themselves and finished outings move to History on their own. For a full wipe, open your bin in the JSONBin dashboard and restore the starter JSON from Setup step 1.

## Customizing it (or working on it with an AI assistant)

Start with [AGENTS.md](AGENTS.md). It points to [ARCHITECTURE.md](ARCHITECTURE.md) (how the code is laid out and how saving/syncing works), [DECISIONS.md](DECISIONS.md) (why it's built this way), and [TESTING.md](TESTING.md) (a manual test checklist, including how to test without touching your group's real data).

## Local development

Serve over http (fetch can be flaky from `file://`):

```bash
python3 -m http.server 8000
# open http://localhost:8000
```

## Notes on the free tier

JSONBin's free tier allows 1 request/second, which is why saves space their read/write calls ~1.1s apart and the page polls others' picks every 45s. For a group of friends this is far more capacity than you'll ever use.

## License

MIT — see [LICENSE](LICENSE). Clone it, remix it, take your friends to the movies.
