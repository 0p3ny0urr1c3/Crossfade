# Crossfade

Find where a friend group's music taste overlaps, and get new track suggestions built from that overlap — powered by [Last.fm](https://www.last.fm) listening data.

Crossfade is a single self-contained HTML file. There's no build step, no backend, and no server — it runs entirely in your browser and talks to the Last.fm API directly.

## What it does

Give it a Last.fm username for each person in a group, and it will show:

- **Shared genres** — pulled from the community tags on each member's most-played artists, ranked by how many members share them.
- **Tracks the group already has in common** — songs that show up in at least 80% of the group's top tracks. You can choose to look within each person's top 10, 20, 50, or 100.
- **New track recommendations** — 10 songs nobody in the group has in their listening history yet, built from Last.fm's "similar tracks" and "similar artists" data around the group's shared taste. Hit **Re-spin** for a fresh batch.

## Getting started

1. Open `crossfade.html` in a browser (or visit the hosted site).
2. Set the group size (2–10 people) and enter each person's Last.fm username.
3. Click **Analyze the group**.

That's it — no installation, no accounts, no sign-in. 

### Running it with your own API key instead

The app calls the Last.fm API with a key hardcoded near the top of the `<script>` block:

```js
const LASTFM_API_KEY = "...";
```

If you fork this project, swap in your own free key from [last.fm/api/account/create](https://www.last.fm/api/account/create). A couple of things worth knowing before you deploy:

- This is a static, client-side-only site (e.g. GitHub Pages), so **anything in the HTML/JS is publicly visible** — including the API key. Last.fm keys are read-only by default (no write/scrobble access without a session + shared secret, which this app never uses), so exposing one is low-risk, but all traffic through it shares one rate limit.
- If the key ever gets rate-limited or you want to rotate it, generate a new one from your Last.fm API account page and swap it into `LASTFM_API_KEY` — no other code changes needed.

## How the analysis works

| Output | Source data | Method |
|---|---|---|
| Shared genres | `user.getTopArtists` + `artist.getTopTags` | Each member's top ~12 artists are tagged, weighted by chart rank, and rolled up into a per-person genre profile. Genres are "shared" if they appear in at least 2 members' top tags. |
| Common tracks | `user.getTopTracks` | Tracks appearing in ≥80% of the group's top-N tracks (N chosen by the depth selector), ranked by combined play count. |
| Recommendations | `track.getSimilar`, `artist.getSimilar`, `artist.getTopTracks` | Seeded from the group's shared tracks/artists, scored by Last.fm's match strength and how many members' taste a seed represents. Anything already in *any* member's top 100 is excluded, so suggestions are new to the whole group. Capped at 2 tracks per artist for variety. |

## A note on "tempo"

Last.fm doesn't expose tempo, key, or other audio-feature data (that used to be available through Spotify's API, but that endpoint has since been restricted). Matching here is based on genre tags, listening rank, and Last.fm's own similarity graph rather than BPM — this is called out in the app itself.

## Limitations

- Last.fm profiles must be **public** — private listening history won't return data.
- Larger groups mean more API calls (tag lookups + similarity lookups per member), so analysis can take 10–30 seconds.
- Genre quality depends on how well-tagged an artist is on Last.fm; very niche or new artists may return few or no tags.
- Nothing is persisted — refreshing the page clears the API key, usernames, and results. There's no localStorage or backend, by design.

## Tech

Plain HTML, CSS, and vanilla JavaScript in one file (`crossfade.html`). No frameworks, no dependencies beyond Google Fonts (Oswald, Inter, IBM Plex Mono) and the Last.fm REST API.
