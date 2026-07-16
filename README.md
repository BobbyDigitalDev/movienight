# 🍿 Movie Night

A tiny, zero-backend scheduling app for movie groups. Friends open one shared link, tap the evenings they're free, and the app highlights the night the most people can make it to the theater.

The whole thing is one `index.html` — no framework, no build step, no server. Host it free on GitHub Pages, store the group's picks in a free [JSONBin.io](https://jsonbin.io) bin.

## Features

- **Two-month rolling calendar** — always shows this month and next; past days prune themselves automatically, so it rolls forward after every movie night with zero maintenance
- **No accounts** — everyone just types a name once; it's remembered on their device ("mike" and "Mike" are matched, so no duplicate people)
- **Per-person colors** — your picks glow in your color; everyone else shows up as count badges and colored dots
- **🏆 Best Night card** — automatically surfaces the date(s) with the most overlap
- **Who's in?** — hold any day to see the names; tap a name chip to spotlight one person's availability
- **Now Showing card** — anyone can set the movie you're all going to see
- **Safe concurrent saves** — saving re-fetches and merges, so two people saving at once can't wipe each other out

## Setup (~15 minutes)

### 1. Create the JSON store

1. Make a free account at [jsonbin.io](https://jsonbin.io)
2. Create a bin containing exactly:
   ```json
   { "movie": { "title": "", "setBy": "", "updatedAt": "" }, "availability": {}, "updatedAt": "" }
   ```
3. Note the **Bin ID** (shown in the bin's URL/header)
4. Under API Keys, create an **Access Key** with only **Bins: Read + Update** permissions. Use an access key, *not* your master key — this key ships in your page source, so it should only be able to read/update this one thing. You can rotate it from the dashboard anytime.

### 2. Get the code

Fork or clone this repo, then open `index.html` and find the `CONFIG` block at the top of the `<script>`:

```js
const CONFIG = {
  BIN_ID: '…',      // ← replace with your Bin ID
  ACCESS_KEY: '…',  // ← replace with your Access Key
  ...
};
```

Replace both values with your own. (The values you'll find there belong to the original author's movie club — the app won't work for your group until you swap them.)

### 3. Deploy

1. Push to a GitHub repo (name it `movienight` and the URL path is `/movienight/`)
2. Repo **Settings → Pages** → Source: **Deploy from a branch** → `main` / root → Save
3. Your app is live at `https://<username>.github.io/movienight/` — or under your custom domain if your GitHub Pages user site has one

Text the link to your group. Done.

## How your group uses it

1. Open the link, enter your name once
2. Tap the evenings you're free, then hit **💾 Save my nights**
3. Hold a day to see who's in; watch the **Best Night** card find your date
4. Set the movie with the ✏️ on the Now Showing card

## Data model

```json
{
  "movie":        { "title": "…", "setBy": "…", "updatedAt": "…" },
  "availability": { "alex": ["2026-07-24", "2026-08-07"] },
  "updatedAt":    "…"
}
```

## Resetting the board

Normally unnecessary — past dates prune themselves and the movie title just gets overwritten. For a full wipe, open your bin in the JSONBin dashboard and restore the starter JSON from Setup step 1.

## Local development

Serve over http (fetch can be flaky from `file://`):

```bash
python3 -m http.server 8000
# open http://localhost:8000
```

## Notes on the free tier

JSONBin's free tier allows 1 request/second, which is why saves space their read/write calls ~1.1s apart and the page polls others' picks every 45s. For a group of friends this is far more capacity than you'll ever use.

## Roadmap

- **Host mode** — the host locks in the official night; everyone sees a confirmation banner
- **"Start next movie night"** — one-tap board reset for the host

## License

MIT — see [LICENSE](LICENSE). Clone it, remix it, take your friends to the movies.
